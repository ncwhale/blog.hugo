---
title: "简单解决WordPress遗留下来的图片问题喵～"
date: 2016-05-06T14:10:34+08:00
lastmod: 2016-05-06T14:10:34+08:00
draft: false
isCJKLanguage: true
tags:
  - "入门"
---

由于将博客系统迁移到EC2上了，发现[之前一些博文](https://kouga.us/tui-jian-yi-kuan-xiao-qiao-hao-yong-de-gu-ge-drivetong-bu-gong-ju-miao--/)里面的图片都跪了喵～ ~~因为当时懒直接用了WordPress的上传~~

解决办法是下载一份之前WP里面的 `/wp-content/uploads/` 文件夹，然后 S3 开个存储桶，开启网站托管，设置所有人可读，然后nginx配置一个跳转即可喵～

```
location /wp-content/ {
  return 301 http://your-s3-endpoint-name.amazonaws.com$request_uri;
}
```

当然别忘了把你下载下来的upload文件夹再上传到S3桶里去喵～
