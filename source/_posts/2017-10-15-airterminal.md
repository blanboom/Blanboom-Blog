---
author: Blanboom
layout: post
slug: airterminal
title: "AirTerminal: 用于 Raspberry Pi 等嵌入式设备的蓝牙终端"
date: 2017-11-30 22:04:00 +0800
comments: true
categories: 软件与工具
tags:
- AirTerminal
- 蓝牙
- BLE
- 终端
- iOS
---



AirTerminal 是我近期完成的一个 iOS App 作品，能够通过蓝牙 4.0 BLE 连接蓝牙串口透传模块，访问 Raspberry Pi 等设备的串口终端。该 App 可在没有 Wi-Fi 或 SSH 连接的情况下访问嵌入式设备，方便对设备进行操作或调试。



经历了 Apple 谜一样的审核流程（等待审核状态持续了 45 天，然后只用了不到 45 分钟就审核通过😂），AirTerminal 已在 App Store 上架，下载链接：[https://itunes.apple.com/cn/app/id1296588408](https://itunes.apple.com/cn/app/id1296588408)



![AirTerminal with screenfetch](/images/2017/10/airterm.png)

<!-- more -->



在我大学毕业之前，只有一台旧笔记本电脑，在图书馆或实验室等比较安静的地方，有时候 CPU 比较忙，风扇突然狂转，发出比较大的噪音，影响到周围的同学。



那时候刚好有了一个平板，打算用 Raspberry Pi Zero 加上锂电池，做一个能够随身携带的小电脑，无线连接到平板后，在平板上使用 Raspberry Pi 的 Linux 环境，以取代自己的笔记本。



由于学校网络的限制，平板和 Raspberry Pi 无法同时连接到 Wi-Fi, 也就没有办法使用 SSH 等工具。这时候就有了做一个 App，使平板通过蓝牙与 Raspberry Pi 连接的想法。



毕业之后，已经不再有这种需求，这时候开发 AirTerminal，主要是为了将其用做嵌入式开发工具，以便于使 iPad/iPhone 连接无法访问网络的嵌入式设备，或在网络故障时，通过 AirTerminal 进行应急操作。



# 使用场景

- 使用 iPad 连接一个迷你 Linux 设备，代替电脑进行使用
- 对嵌入式设备进行开发和调试
- 在设备发生故障，网络连接、SSH 不可用时，使用 AirTerminal 和蓝牙串口透传模块，对设备进行应急处理



# 功能与特色

- 全功能终端，支持 bash, vim, top, tmux 等程序
- 支持外接键盘与蓝牙键盘
- 内置 ESC, TAB, CTL, 方向键等常用按键
- 内置 "Fit" 按键，可通过该按键自动输入 stty 命令，调整终端大小
- 支持主模式 (central) 和从模式 (peripheral)



# 其他

1. 由于 Raspberry Pi 3 的板载蓝牙目前不支持串口透传，目前 AirTerminal 不能直接支持 Raspberry Pi 3 上的板载蓝牙。
2. DFRobot 的 [USB BLE Link](http://www.dfrobot.com.cn/goods-1065.html) ([Bluno Link](https://www.dfrobot.com/product-1220.html)) 会将 USB 串口输入的数据，通过 USB 串口原路返回输出。所以 AirTerminal 和 USB BLE Link 配合使用会出现异常。
3. 提高串口波特率，可以带来更好的使用体验。