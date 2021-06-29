---
title: "PoserShell Tips"
date: 2021-06-30T05:01:40+09:00
tags: ["Tool", "Code", "PowerShell"]
categories: ["Tool"]
draft: false
isCJKLanguage: true
---

## 安装软件包管理器

个人推荐使用 [Scoop](https://scoop.sh/) 包管理器，下面很多提及指令都可以通过 Scoop 直接安装喵~

```PowerShell
# Fix policy
Set-ExecutionPolicy RemoteSigned -scope CurrentUser

# Install
iwr -useb get.scoop.sh | iex
```

## 增加常用 Linux Shell 命令

```PowerShell
scoop install aria2 sudo which git nvm
```

## 美化 Shell

推荐安装 [`Oh my posh`](https://ohmyposh.dev/) 来进行美化（虽然它还可以用来做更多的事情喵）

```PowerShell
sudo pwsh
Install-Module oh-my-posh -Scope AllUsers
Import-Module oh-my-posh
```

顺便我使用的主题配置文件[在这里](https://gist.github.com/omtg-mo/02346ef3b01b170e729dda671efeccc9)喵~ (使用 OneDrive 同步喵~
截图字体采用的是 `"NotoMono NF"` 喵~

![ScreenShot of PowerShell](https://i.imgur.com/NV9b2GQ.png)

国内图床：

![ScreenShot of PowerShell](https://i.bmp.ovh/imgs/2021/06/ab6fce84043f099b.png)

## 同步远程桌面密码

如果你在 Windows 10 里使用了微软帐户或者域账户登录，然后远程桌面到这台计算机始终提示密码错误，一个可能的原因是本地系统里账户密码没和域密码同步喵~

使用下列命令来同步密码：

```PowerShell
sudo runas /u:MicrosoftAccount\your@domain.com cmd.exe
```

## 加载 Visual Studio 编译环境

1. 在 PowerShell 环境里添加一个导出 cmd env 的函数[Source](https://github.com/majkinetor/posh/tree/master/MM_Admin)：

    PS: 可以直接将函数放到自己的 `$PROFILE` 里面，每次开启 PowerShell 就会自动加载进来喵~

```PowerShell
function Invoke-Environment {
    param
    (
        # Any cmd shell command, normally a configuration batch file.
        [Parameter(Mandatory=$true)]
        [string] $Command
    )

    $Command = "`"" + $Command + "`""
    cmd /c "$Command > nul 2>&1 && set" | . { process {
        if ($_ -match '^([^=]+)=(.*)') {
            [System.Environment]::SetEnvironmentVariable($matches[1], $matches[2])
        }
    }}

}
```

2. 使用该函数来设置 Visual Studio 环境，比如 x64 环境：

```PowerShell
Invoke-Environment "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvars64.bat"
```