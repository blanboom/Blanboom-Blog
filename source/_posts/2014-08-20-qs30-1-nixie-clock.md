---
author: Blanboom
layout: post
slug: qs30-1-nixie-clock
title: QS30-1 辉光管时钟
date: 2014-08-20 23:46:00 +0800
comments: true
categories: 制作
tags:
- 辉光管
- 时钟
- 蓝牙
---

用暑假的空闲时间，断断续续做了一个多月，辉光管时钟基本完成。

辉光管使用了 QS30-1，通过四个氖灯显示时间的冒号。每个辉光管下面各安装一个全彩 LED，可控制其显示颜色。

该时钟使用 MC34063，配合 MOS 管和电感等构成 DC-DC 升压电路，将 12V 电压升至 170V，供辉光管使用。通过 HV57708 驱动辉光管。LPD6803 用于控制全彩 LED。主控芯片采用 STC15F2K60S2，时钟芯片采用 SD2405ALPI，蓝牙模块采用 RF-BM-S02.

程序源代码和 PCB 图都已上传至 GitHub：[https://github.com/blanboom/NixieClock](https://github.com/blanboom/NixieClock)

![辉光管时钟照片](http://blanboom.org/images/2014/08/NixieClock.jpg)

<!-- more -->

![辉光管时钟照片 2](http://blanboom.org/images/2014/08/NixieClock_2.jpg)

![辉光管时钟照片 3](http://blanboom.org/images/2014/08/NixieClock_4.jpg)

