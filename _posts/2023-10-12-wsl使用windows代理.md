---
title: wsl使用windows代理
date: 2023-10-12 00:22:38 +0800
categories: [windows, wsl]
tags: [代理, wsl]     
---

阅读官方文档[使用 WSL 访问网络应用程序](https://learn.microsoft.com/zh-cn/windows/wsl/networking)

里面提到，windows访问wsl上运行的程序，直接访问`localhost`即可。

而wsl访问windows，则
1. 通过在 Linux 分发版中运行以下命令来获取主机的 IP 地址：`cat /etc/resolv.conf`
2. 复制以下词语后面的 IP 地址：`nameserver`。
3. 使用复制的 IP 地址连接到任何 Windows 服务器。

![img](https://learn.microsoft.com/zh-cn/windows/wsl/media/wsl2-network-l2w.png)


## 为wsl上的git设置github代理
举例windows上运行了一个代理软件，允许LAN访问，端口为7890。

在wsl执行命令
先 `cat /etc/resolv.conf`假设获取到的ip为`172.20.160.1`，则再执行以下命令
`git config --global http.https://github.com.proxy http://172.20.160.1:7890`
即可将访问github的仓库地址的流量指向windows上的代理软件进行加速。
