---
title: "Howto: PostgreSQL 查询分析和优化"
date: 2016-05-14T17:03:58+08:00
lastmod: 2016-05-14T17:03:58+08:00
draft: false
isCJKLanguage: true
tags:
  - "入门"
---

故事开始：公司的服务在周末高峰期，跪了喵……

通过分析日志，发现数据库是瓶颈所在，导致了API服务和Nginx都被阻塞以至于无法提供服务……

于是，一边通过启动更大的RDS实例来承载负载，一边开始着手分析慢查询并调优。

---

### 第一步：打开 PostgreSQL 的 查询统计扩展

pg 带有一个 `pg_stat_statements` 扩展，但是默认并没有启用，而启用该扩展需要重启数据库。参照了[这篇文档](https://pganalyze.com/docs/install/amazon_rds/01_configure_rds_instance) 在 Amazon RDS 上开启了该扩展：

1. 新建一个数据库参数组；
2. 修改该参数组设置：
    <table><thead><tr><th>参数</th><th>pg95</th><th>default.postgres9.5</th></tr></thead><tbody><tr><td >pg_stat_statements.max</div></td><td>5000</div></td><td>&lt;engine-default&gt;</div></td></tr><tr><td>pg_stat_statements.save</div></td><td>1</div></td><td>&lt;engine-default&gt;</div></td></tr><tr><td >pg_stat_statements.track</div></td><td>TOP</div></td><td>&lt;engine-default&gt;</div></td></tr><tr><td>pg_stat_statements.track_utility</div></td><td>1</div></td><td>&lt;engine-default&gt;</div></td></tr><tr><td >shared_preload_libraries</div></td><td>pg_stat_statements</div></td><td>&lt;system&gt;</div></td></tr></tbody></table>

3. 选中需要分析的RDS实例，操作->修改->应用刚刚创建的参数组；
4. 重启RDS实例，注意这个会导致服务暂时不可用，请小心操作；
5. 等待重启完毕后，连接到数据库，选中需要分析的数据库，执行 `CREATE EXTENSION IF NOT EXISTS pg_stat_statements;` 来启用扩展；

### 第二步：查询&分析统计数据

`pg_stat_statements` 扩展启用后，保持服务运作或执行压力测试的情况下，过一段时间执行查询:

```SQL
SELECT calls, total_time,max_time, mean_time, rows, 100.0 * shared_blks_hit /nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent,query FROM pg_stat_statements ORDER BY total_time DESC LIMIT 20;
```

即可获得查询时间最长的相关SQL语句，同时可以调整 ORDER BY 为 `mean_time` 和 `max_time` 来查看平均时间最长和单次爆发最长的相关查询，将这些得分最高的QUERY都记录下来喵～

### 第三步：优化查询

使用 pg 自带的 EXPLAIN 查询来分析查询到底使用了什么方式执行，有没有使用索引，有没有扫表操作等等，来分析这条语句为什么这么慢喵～

比如，在上一步发现了这么一条语句查询次数&耗时都非常高： 

`SELECT  "contributions".* FROM "contributions" WHERE (created_at > ?)  ORDER BY like_times desc LIMIT ?;`

我们使用 EXPLAIN 并构造一个查询来看它到底发生了什么：

```SQL
db=> EXPLAIN SELECT  "contributions".* FROM "contributions" WHERE (created_at > '2016-01-01')  ORDER BY like_times desc LIMIT 1000;                                                                                    
                                       QUERY PLAN                                        
-----------------------------------------------------------------------------------------
 Limit  (cost=19020.45..19022.95 rows=1000 width=99)
   ->  Sort  (cost=19020.45..19113.68 rows=37291 width=99)
         Sort Key: like_times DESC
         ->  Seq Scan on contributions  (cost=0.00..16975.83 rows=37291 width=99)
               Filter: (created_at > '2016-01-01 00:00:00'::timestamp without time zone)
(5 rows)
```

发现原来在执行筛选的时候，查询执行了扫表操作，那么进一步查看这张表已有索引情况：

`SELECT * FROM pg_indexes WHERE tablename='contributions';`

原来这张表的 created_at 列没有任何索引，于是创建索引：

`CREATE INDEX index_contributions_on_created_at ON contributions USING btree (created_at);`

然后再次执行 EXPLAIN :

```SQL
db=> EXPLAIN SELECT "contributions".* FROM "contributions" WHERE (created_at > '2016-01-01')  ORDER BY like_times desc LIMIT 1000;
                                                               QUERY PLAN                                                                
-----------------------------------------------------------------------------------------------------------------------------------------
 Limit  (cost=5469.78..5472.28 rows=1000 width=99)
   ->  Sort  (cost=5469.78..5563.54 rows=37505 width=99)
         Sort Key: like_times DESC
         ->  Index Scan using index_contributions_on_created_at on contributions  (cost=0.42..3413.42 rows=37505 width=99)
               Index Cond: (created_at > '2016-01-01 00:00:00'::timestamp without time zone)
(5 rows)

```

发现执行该SQL的时候已经自动启用了 index 并大大缩短了预计时间，此时我们需要重置统计数据来观察优化结果： `SELECT pg_stat_statements_reset();`

重复步骤二和步骤三，直到所有的常用查询基本都在 2ms 以下，然后进行下一步操作：

### 第四步：关闭查询统计扩展

一定时间的分析和优化之后，我们可以选择关闭扩展来释放统计带来的额外数据库消耗：

`DROP EXTENSION IF EXISTS pg_stat_statements;`

同时将 RDS 参数组设置为未开启该扩展的参数组，并重启 pg 实例;

---

备注：本文所有操作都基于 PostgreSQL 9.5，基本可以适用于 9.x 的数据库版本，但是更高版本不保证一定可行喵～