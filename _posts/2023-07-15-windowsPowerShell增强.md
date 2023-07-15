---
title: win的Powershell增强
date: 2023-07-15 08:11:00+0800
categories: [win, powershell]
tags: [win, powershell]     
---

## Powershell

这是windows提供的新的shell，可以代替cmd

但是shell没有自动补全，样式也很简单，相比较linux上的fish，zsh差距比较大。

我们这里让powershell稍微好用一些，起码一些历史命令和自动补全给加上。

## 安装Nerd字体

进入[Nerd Fonts - Iconic font aggregator, glyphs/icons collection, & fonts patcher](https://link.juejin.cn/?target=https%3A%2F%2Fwww.nerdfonts.com%2Ffont-downloads)安装字体，这里我选择了FiraMono。并安装了`FiraMono Nerd Font Propo`系列字体。由于我使用的是`alacritty`，那么在windows下我修改`%APPDATA%\alacritty\alacritty.yml`文件中的字体设置。

```yaml
# Font configuration
font:
  # Normal (roman) font face
  normal:
    # Font family
    #
    # Default:
    #   - (macOS) Menlo
    #   - (Linux/BSD) monospace
    #   - (Windows) Consolas
    family: FiraMono Nerd Font Propo

    # The `style` can be specified to pick a specific face.
    # style: Regular
```

也可以先安装`oh-my-posh`在通过其提供的命令行工具安装字体，会更简单，不过会受限于网络，不好更改网络，下面会讲到。

## 安装最新版本的powershell并进行使用

默认win自带的powershell是`C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`。并不是最新的完整的powershell，而是一个简化版本的。我们安装完整的心得powershell进行使用。

```powershell
winget install --id Microsoft.Powershell --source winget
```

安装成功后，再次修改文件`code %APPDATA%\alacritty\alacritty.yml`。如果没有安装vscode可以用`notepad.exe %APPDATA%\alacritty\alacritty.yml`。

```yaml
# Shell
#
# You can set `shell.program` to the path of your favorite shell, e.g.
# `/bin/fish`. Entries in `shell.args` are passed unmodified as arguments to the
# shell.
#
# Default:
#   - (Linux/BSD/macOS) `$SHELL` or the user's login shell, if `$SHELL` is unset
#   - (Windows) powershell
shell:
 program: C:\Program Files\PowerShell\7\pwsh.exe
#  args:
#    - --login
```

## 安装oh-my-posh，posh-git和Readline

```powershell
Install-Module -Name PowerShellGet -Force
winget install JanDeDobbeleer.OhMyPosh -s winget
PowerShellGet\Install-Module posh-git -Scope CurrentUser -Force
Install-Module PSReadLine

```

然后编辑文件`code $PROFILE`

加入内容

```powershell
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete #Tab键会出现自动补全菜单
Set-PSReadlineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadlineKeyHandler -Key DownArrow -Function HistorySearchForward
# 上下方向键箭头，搜索历史中进行自动补全

oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH/night-owl.omp.json" | Invoke-Expression
Import-Module posh-git # git的自动补全
```

关闭alacritty再次打开应该如图所示

![image-20230715074521166](/assets/image/2023-07-15-windowsPowerShell增强/image-20230715074521166.png)

### 通过oh-my-posh安装字体

```powershell
oh-my-posh font install --user
```



## 乱码

因为采用nerd字体，所以没有nerd字体的话，一些图案无法显示出现乱码。根据上文中的Nerd字体安装进行安装

## oh-my-posh 主题

执行`Get-PoshThemes`会展示所有主题，之后修改`$PROFILE`中的`--config`参数，比如`night-owl.omp.json`替换为`jandedobbeleer.omp.json`。或者在 [网页](https://ohmyposh.dev/docs/themes) 上进行浏览。