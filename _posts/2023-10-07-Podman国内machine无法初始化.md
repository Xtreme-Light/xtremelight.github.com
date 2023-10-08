---
title: podman国内安装无法初始化
date: 2023-10-08 14:18:00+0800
categories: [运维]
tags: [运维,podman]    
---

# Podman国内安装无法初始化

在进行初始化init and start的时候会卡住，实际上是因为无法下载对应的文件。可以手动下载，然后手动执行初始化命令

在这里下载[rootfs.tar.xz](https://github.com/containers/podman-wsl-fedora/releases)

下载可以挂梯子，也可以使用ghproxy，下载完成后手动执行命令进行初始化

`podman machine init --image-path .\rootfs.tar.xz`

参见[podman安装](https://du33169.tech/posts/notes/podmaneasyconnect/)