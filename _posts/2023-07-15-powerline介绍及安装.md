---
title: powerline介绍及其安装
date: 2023-07-15 09:00:00+0800
categories: [font]
tags: [font]    
---

[powerline](https://github.com/powerline/powerline)是VIM的状态线插件，也为其他的应用程序提供了支持，比如zsh，bash，Fish，tmux等。效果如下图

![效果](https://camo.githubusercontent.com/853e176b09fed9071a6e9c61040ecb96d900d087dd780dd6fb3704e51dd32ca6/68747470733a2f2f7261772e6769746875622e636f6d2f706f7765726c696e652f706f7765726c696e652f646576656c6f702f646f63732f736f757263652f5f7374617469632f696d672f706c2d6d6f64652d6e6f726d616c2e706e67)

![效果2](https://camo.githubusercontent.com/d85a1aa7e05b3cf42a73c66b117e451287c2dd2fa722a4ff17fdda7d2067173a/68747470733a2f2f7261772e6769746875622e636f6d2f706f7765726c696e652f706f7765726c696e652f646576656c6f702f646f63732f736f757263652f5f7374617469632f696d672f706c2d6d6f64652d696e736572742e706e67)

![效果三](https://camo.githubusercontent.com/153b3d06360b29045a16f7da2b60e439b39789210cec5fd6bec2fa0dcbc8c7fa/68747470733a2f2f7261772e6769746875622e636f6d2f706f7765726c696e652f706f7765726c696e652f646576656c6f702f646f63732f736f757263652f5f7374617469632f696d672f706c2d6d6f64652d76697375616c2e706e67)

通过 [powerline fonts](https://github.com/powerline/fonts)可以很方便的为linux，mac，win安装对应的字体

比如 通过`apt-get`

```shell
sudo apt-get install fonts-powerline
```

或者安装部分指定字体

```shell
wget https://github.com/powerline/powerline/raw/develop/font/PowerlineSymbols.otf
wget https://github.com/powerline/powerline/raw/develop/font/10-powerline-symbols.conf
mv PowerlineSymbols.otf ~/.local/share/fonts/
fc-cache -vf ~/.local/share/fonts/
mv 10-powerline-symbols.conf ~/.config/fontconfig/conf.d/

```

