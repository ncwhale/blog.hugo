---
title: 使用 Nuxt.js 静态构建 Blog
date: 2021-06-04T08:09:15+09:00
tags: ["Nuxt", "Blog", "教程"]
categories: ["教程"]
isCJKLanguage: true
toc:
  enable: true
  auto: true
---

## 前置需求

最近又想要重新开放个人博客，因为很多东西写微博上写不下喵……

比如，有个很好的 UE 相关实现；又比如，自己折腾了一些小项目和小玩意什么的喵~

~~再或者看到了什么不得了的色图喵~~

## 为什么选用全静态站点 （SSG）生成模式？

* 我的博客编辑只需要在 VSCode/EMACS/树莓派/iPad …… 等地方选用自己喜欢的编辑器编辑好就行了喵~
* 静态文件，没地方进行服务端注入操作喵~
* 没有 Cookie ，也没有跟踪 Tag ，天然纯净喵（X
* 部署到 CDN 自动伸缩， DDOS 与我无关喵~
* 就算我编译服务炸了，也仍旧能浏览，而不会影响已经发布的内容喵~

## 预先调研的工具们

首先 [Google 了一下 static site generator 2021](https://www.google.com/search?q=static+site+generator+2021&oq=static+site+generator+2021&ie=UTF-8) 主要参考了 [Choosing the Best Static Site Generator for 2021](https://snipcart.com/blog/choose-best-static-site-generator) 与 [Best static site generators of 2021](https://www.techradar.com/best/static-site-generators) 两篇文章列出的工具喵~

然后全凭自己主观喜好选择了 [Nuxt.js](https://nuxtjs.org/) 工具链进行测试喵~（你甚至能发现这个项目标题一开始叫 blog.nuxt.test 喵）

咱觉得的优点：

* VUE 模板（个人喜好）
* SSR/SSG 全支持（咱需要SSG，但是SSR存在使得 Live Preview 更简单喵~
* 工具和插件比较完善（ robots/sitemap/SSO 等等工具都有喵~
* 

缺点：

* `98 vulnerabilities (2 low, 91 moderate, 5 high)`

## 开始折腾喵！

按照 [nuxt.js 教程](https://nuxtjs.org/docs/2.x/get-started/installation) 使用 `npm init nuxt-app blog.nuxt.js` 指令初始化项目，然后就是常规的 `npm i` `npm run dev` 就启动起来了喵~

折腾途中绕了不少弯路，比如一开始选了 `bootstrap-vue` 这个组件，然后发现这 Themes 简直就是抢喵……啊，不，抢钱喵！免费的又全部是 Dashboard Style …… 毅然换了 `material-vue` 并启用了 [material-kit](https://demos.creative-tim.com/vue-material-kit/) 作为主题喵~

然后启用 [@nuxt/content](https://content.nuxtjs.org/) 来管理博客文本，并自己写 `_slug.vue` 来定制页面表现形式喵~

## 结局

猫累了，玩不动了喵…… nuxt 是个好的 VUE 扩展框架但是这生态弄的人太累了喵……暂时就这样了喵……（躺平