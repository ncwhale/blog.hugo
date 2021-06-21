---
title: "为什么这个服务器会设置如此蛋疼的认证流程"
date: 2012-04-25T04:07:57+08:00
lastmod: 2012-04-25T04:07:57+08:00
draft: false
isCJKLanguage: true
tags:
  - "游戏"
---

<p>咱不做任何评论喵。</p>  <p>服务器日志：

```
kouga@mc:~/db/cb-1.2.5$ tail server.log
2012-04-25 11:23:02 [INFO] Disconnecting ### [/183.94.14.8:61363]: Outdated client!
2012-04-25 11:23:02 [INFO] Reached end of stream    
2012-04-25 11:24:42 [INFO] Disconnecting yooooku [/183.94.14.8:62116]: Outdated client!
2012-04-25 11:24:43 [INFO] Reached end of stream
2012-04-25 11:24:50 [INFO] Disconnecting yooooku [/183.94.14.8:62172]: Outdated client!
2012-04-25 11:24:50 [INFO] Reached end of stream
```

邮件截图：
![邮件截图](http://i.imgur.com/fecUJrT.png)