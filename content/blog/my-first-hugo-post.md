---
title: "My First Hugo Post"
date: 2021-06-05T17:32:15+09:00
tags: ["Hugo", "Blog", "教程", "scoop"]
categories: ["教程"]
toc:
  enable: true
  auto: true
---

……卧槽竟然用 `hugo` 这么简单这么快喵喵喵！而且主题那么多选择毫不费力喵！

## Hugo 从安装到写字到部署

直接复制这些代码就行了喵……保证不超过1分钟就能开始写字喵……

```PowerShell {linenos=table,hl_lines=[6,7,9,10],linenostart=1}
scoop install hugo-extended
hugo new site blog.hugo
cd blog.hugo
git init

# 注意，如果您想用 Cloudflare Pages 进行 CI 部署，则这里推荐用 Subtree 方式替代，否则会出错喵~
git submodule add "https://github.com/upagge/uBlogger.git" themes/uBlogger

#注意，这里可能被 Windows 环境坑掉，您可能需要手动编辑 config.toml 喵~
echo 'theme = "uBlogger"'>>.\config.toml 

hugo new blog/my-first-hugo-post.md
hugo server -D
code .
```

好了现在 vscode 里应该就直接能写字了喵！就像这篇一样喵！超级不折腾喵！点开 [my-first-hugo-post](http://localhost:1313/blog/my-first-hugo-post/) 就能看到 Hotreload 效果的预览喵！

备注：如果没有安装 scoop git 和 vscode 的话，建议先 [安装 scoop](https://scoop.sh/) 然后使用 `scoop` 一键安装提及软件包喵：

```PowerShell
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
iwr -useb get.scoop.sh | iex
scoop install git vscode-portable hugo-extended
```

## 稍微配置一下

* 部署CI（比如用 [Cloudflare Page](https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site) 就行了喵！
* ~~主题配置 （参考自己选择的主题的相关文档进行，我这里选用了 [uBlogger](https://ublogger.netlify.app/) (似乎代码高亮还有点问题喵……~~
* 主题配置 （参考自己选择的主题的相关文档进行，我这里选用了 [Mainroad](https://github.com/Vimux/Mainroad/) 
* 代码高亮主要需要参考 [Syntax Highlighting | HUGO](https://gohugo.io/content-management/syntax-highlighting/) 文档重新配置喵

## 接下来……

之前用 WordPress/Ghost 等工具写的博客，要找个方式导入了喵~ 大概又是一项大工程喵？