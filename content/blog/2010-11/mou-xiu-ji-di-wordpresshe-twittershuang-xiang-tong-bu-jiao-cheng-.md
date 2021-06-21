---
title: "某⑨秀吉滴Wordpress和Twitter双向同步教程"
date: 2010-11-28T09:24:06+08:00
lastmod: 2010-11-28T09:24:06+08:00
draft: false
isCJKLanguage: true
---

<p>首先要做滴素在乃滴Wordpress空间里搜索安装以下两个插件：Twitter Tools和WP to Twitter ，它们就素我们这里需要滴主角了喵~</p>  <p>Twitter Tools 滴主要作用就是将乃滴推特条目同步到Wordpress数据库，然后根据需要转换成博文或者仅仅只是在侧边栏进行实时显示什么滴，顺带会给文章分栏里面添加一个Tweet滴窗口，方便直接发推~</p>  <p>而WP to Twitter 滴主要作用就素在乃发布/修改/清理博文滴时候，将这些内容同步发布到推特上去，同时它附带了N多插件例如 Bit.ly 等等，可以根据需要进行安装。</p>  <p>(截图什么滴最讨厌了喵！)</p>  <p>这些插件之前都能好好滴和Twitter交互工作直到Twitter强制要求进行OAuth认证。本来没什么大问题的结果……这俩货竟然都不支持直接OAuth而是要求直接填入以下4个内容：</p>  <p>Consumer Key, Consumer Secret: 这两个就是<a href="http://twitter.com/apps">在推特上注册APP</a>滴时候给出来滴公私键值对，只要注册完毕即可获得；</p>  <p>Access Token, Access Token Secret:这两个是进行OAuth登陆成功后计算出来的一对键值对，没有找到直接办法可以获得……根据以往编写Bot滴经验，登陆后的Token可以用GAE直接输出，于是写了一堆代码然后使用GAE的控制台直接输出了出来……感觉各种不爽……</p>  <p>在此公开一个支持登陆后查看Access Token 滴GTAP API代理：<a title="https://kouga-jt.appspot.com/" href="https://kouga-jt.appspot.com/">https://kouga-jt.appspot.com/</a> ，使用它OAuth成功后就能在 <a href="https://kouga-jt.appspot.com/oauth/change">change/view your key here</a> 里面使用该API的username/key来查看上述4个键了喵~什么？该页面无法访问？请自行前往 <a href="http://just-ping.com/">http://just-ping.com/</a> Ping 一下，然后修改Hosts什么滴……</p>  <p>另外友情提醒，如果需要将推特同步为日志，请一并安装mCatFilter这个插件，它可以将每天产生滴话痨博文自动从首页和全文RSS中过滤掉，方便大家订阅喵~</p>  <p align="left">具体做法：新增加一种文章分类，例如“<a href="http://kouga.us/?cat=9">话痨滴日志</a>”，然后在Twitter Tools里面设置发布文章所属类别到该分类，接下来在mCatFilter里面设置该分类永远不在首页出现即可，否则铺天盖地的话痨日志会让博客质量缩水滴喵……</p>