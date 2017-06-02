---
author: Blanboom
layout: post
slug: nixie-clock-lite
title: "四位辉光管时钟"
date: 2016-03-07 12:00:00 +0800
comments: true
categories: 电子与制作
tags:
- 辉光管
- 时钟
- Arduino
---

[上次做的辉光管时钟](https://blanboom.org/2014/qs30-1-nixie-clock)在外观上还有一些不满意的地方。所以最近这几天，我又新做了一个辉光管时钟。不同于上次，为了节省时间，电路和 PCB 都没有自己设计，而是直接采用成品模块。这样制作难度大大下降，同时能够更快地完成制作。

![辉光管时钟](/images/2016/03/nixie-photo-1.jpg)

<!-- more -->

![辉光管时钟](/images/2016/03/nixie-photo-2.jpg)

![辉光管时钟](/images/2016/03/nixie-photo-3.jpg)

# 电路

在电路上，全部使用成品模块进行制作，通过简单的导线连接即可完成。具体介绍如下：

![电路](/images/2016/03/nixie-circuit.png)

**1) Arduino 兼容控制板**

控制板采用 [Bluno Beetle](http://www.dfrobot.com.cn/goods.php?id=1097). Bluno Beetle 是一块兼容 Arduino Uno 的开发板，体积较小，且内置了蓝牙 4.0 功能。使用这块板子不仅可以节省空间，还支持无线下载程序，并能通过蓝牙实现更多功能。


**2) 实时时钟与温度传感器**

实时时钟模块采用 [DFRobot 的 DS1307 模块](http://www.dfrobot.com.cn/goods.php?id=535)，用于在掉电的情况下维持时间。另外，这一模块预留了 DS18B20 温度传感器的焊盘，焊接上温度传感器，即可测量温度。

**3) 滚珠开关**

考虑到外观和制作的难易程度，我没有在其中安装按键或微动开关。时间调整和闹钟设置均通过蓝牙完成。但是当闹钟响起的时候，用手机之类的设备关闭闹钟比较麻烦，这是通过滚珠开关，轻拍辉光管时钟，即可方便地停止闹钟。

**4) 辉光管模块**

直接使用[成品辉光管模块](https://item.taobao.com/item.htm?id=15397910473)，能够避免复杂的电源电路设计工作。该模块供电电压为 5V，可直接使用 USB 供电。

# 外壳

上次制作的辉光管时钟，只有上下两片亚克力板起到保护作用，在外观上似乎有点简陋。所以这一次制作一个完整的亚克力外壳。

![外壳](/images/2016/03/nixie-case.png)

外壳采用 3mm 黑茶色透明亚克力板制作而成，其中，顶部和侧面通过胶水固定，成为一个底部开口的亚克力盒子。所有模块放置于底板，通过螺丝和铜柱与亚克力盒相连，可以自由拆卸。

由于不会使用用 [AutoCAD](http://www.autodesk.com.cn/products/autocad/overview) 和 [CorelDRAW](http://www.coreldraw.com/)，我选择用自己比较熟悉的 PCB 设计软件 [Altium Designer](http://www.altium.com) 进行外壳的初步设计，然后导出为 PDF 格式，在 [Affinity Designer](https://affinity.serif.com) 中进行进一步处理，即可在淘宝上进行亚克力板的定做。（其实只用 Altium Designer 或者只用 Affinity Designer 均可直接完成外壳图纸的绘制，我只是选择了对于自己来说最方便的方法）

另外，由于辉光管的视角有限，当辉光管时钟斜放于桌面时，显示效果最好。所以我打算在时钟底部安装亚克力铰链，从而能够更方便地斜放在桌面上。

# 软件

由于使用 Arduino 兼容的开发板，软件开发会变得更加容易。

另外，这也是我首次使用 [PlatformIO](http://platformio.org), 这是一个兼容 Arduino 和 MBED 的跨平台构建系统，与 [Arduino IDE](https://www.arduino.cc/en/Main/Software) 相比，功能更加强大。如果觉得 Arduino IDE 不好用，可以尝试下这个。

目前，基本的时钟功能（包括阴极中毒保护）已经完成。

GitHub 地址：[https://github.com/blanboom/NixieClockLite](https://github.com/blanboom/NixieClockLite)

# 更多

还有几个月就要毕业，这件作品应该会成为我大学期间最后一个独立完成的电子制作。接下来，就要看书和准备毕业设计了。

上个学期，通过校园招聘，我找到了一份软件开发相关的工作，也算是从 EE 转向 CS 了。毕业之后，接触电子和嵌入式的机会可能会逐渐变少，希望自己对电子 DIY 的兴趣不会改变。
