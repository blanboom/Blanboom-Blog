---
author: Blanboom
layout: post
slug: hackerremote
title: "HackerRemote: 蓝牙遥控器 App, 支持数据的双向收发 (iOS)"
date: 2016-06-06 15:12:00 +0800
comments: true
categories: 软件与工具
tags:
- iOS
- 蓝牙
- BLE
- HackerRemote
---

这是我前段时间学习 Swift 时的练手作品，也是我的第一个 iOS App. 

HackerRemote 是一个游戏手柄形状的蓝牙 (BLE) 遥控器，可搭配 DFRobot Bluno 或蓝牙转串口模块，用于各种需要手机遥控的电子制作。

除了能将数据发送至蓝牙设备，该 App 还可以从设备中接收数据（例如电池电量、飞行器飞行高度等），并显示在界面上，方便查看设备信息，对设备进行调试。

![HackerRemote 截图](http://blanboom.org/images/2016/06/hackerremote.jpg)

App 的界面还比较简陋，在功能上，还有一些想法尚未实现。我会在空闲时间，根据情况对其进行进一步完善。

下载链接：

<a href="https://itunes.apple.com/cn/app/id1120243546"><img src="http://blanboom.org/images/2016/06/appstore.png" align="left"></a></br>

[https://itunes.apple.com/cn/app/id1120243546](https://itunes.apple.com/cn/app/id1120243546)



<!-- more -->

以下为 App 的使用方法与功能介绍：

# 连接蓝牙设备

1. 通过屏幕右上角的 Scan 按钮，搜索并选择需要连接的蓝牙设备。
2. 输入蓝牙转串口模块的串口部分所对应的 Service UUID, 以及 RX, TX 引脚所对应的 UUID.
3. 通过屏幕下方的 Connect 按钮，连接蓝牙设备，如果连接成功，即可进入遥控器界面。

Service UUID, TX UUID, RX UUID 的具体值请参考蓝牙转串口模块的官方文档。例如 DFRobot Bluno, 其这三项值分别为 DFB0, DFB1, DFB1.

如果不需要接收数据，也可将 RX UUID 留空。

# 向蓝牙设备发送数据

遥控器界面共有 9 个按键，每个按键对应一个字母。当按键按下后，发送按键对应的大写字母，当按键释放后，发送按键对应的小写字母。

例如当 A 键按下后，发送字母 ”A". 当 A 键释放后，发送字母 “a".

# 接收数据

当输入 RX UUID 并连接蓝牙设备后，可在软件界面中显示蓝牙设备发送的字符串。

目前只支持显示单行字符串，同时，仅支持通过查询的方式获取数据（每隔 0.5 秒更新一次数据）。在下一个版本中，将会对此进行优化。

