---
layout: post
title: 发现是否有人用USB偷插你的电脑
categories: [blog ]
tags: [USB, ]
description: 插入U盘记录(tip)
---
### windows注册表验证USB设备的插入
下面我们将演示下如何找到设备的信息：

1. 同时按下WIN+R键，也就是打开“运行”。

2. 输入regedit，按下回车。

3. 转到HKEY_LOCAL_MACHINE\SYSTEM\ControlSet00x\Enum\USBSTOR（这里ControlSet00x里的下可以是任何数字）
咱们点击下注册表中任意一台设备就可以发现，它们都有由设备制造商指定的独立id。所以，如果我们想分辨是否有新的USB设备连接到该电脑上，肯定是非常容易的。