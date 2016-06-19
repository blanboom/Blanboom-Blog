---
author: Blanboom
layout: post
slug: nes-emulator
title: "使用 C 和 Allegro 实现的 NES 模拟器"
date: 2016-04-20 20:39:00 +0800
comments: true
categories: 软件与工具
tags:
- NES
- Famicom
- C
- Allegro
---

NES (Nintendo Entertainment System, [Wikipedia](https://en.wikipedia.org/wiki/Nintendo_Entertainment_System)) 是历史上一款著名的游戏机，由任天堂于 1983 年推出，又被称做 FC (Family Computer) 或红白机。在国内，或许大家对「小霸王学习机」这个名字更加熟悉，应该有不少人就是在这台学习机上第一次接触了电子游戏。其实，「小霸王学习机」就是 NES 的山寨版，兼容 NES 游戏，在硬件上与 NES 基本相同。

即使对游戏不感兴趣的人，听到「超级马里奥」（超级玛丽）等，也应该不会陌生。而这些游戏角色，正是由于 NES 的普及，才变得更加知名。

在 NES 推出的时候，计算机多媒体技术并不是十分发达。而且不论是 NES，还是当时流行的 Apple II，其内存只有几 KB 到几百 KB. 为了能够流畅地显示游戏画面、播放游戏声音，NES 采用了不少巧妙的技术。

<!-- more -->

# 我的 NES 模拟器

由于 NES 已经是上个世纪的硬件，目前已经存在许多 NES 模拟器，可以运行在电脑、手机、游戏机甚至 STM32 之类的单片机上。而我只做这一 NES 模拟器，只是是为了让自己能够更加清晰地理解计算机底层的工作原理（<s>顺便做为毕设的一部分</s>）。如果需要在自己的程序中使用 NES 模拟器，建议参考更加成熟的项目。

目前，该模拟器具有下列功能：

1.  运行基本的 NES 程序
2.  使用 Allegro 显示游戏画面
3.  反汇编
4.  显示 NES ROM 的相关信息
5.  在模拟器运行的过程中，显示 CPU 和 PPU 寄存器上的数值

一些较为复杂的游戏，在其卡带中，除了 ROM 和 RAM 之外，还内置有内存控制器等芯片，或者使用了未在 NES CPU (6502) 文档中公开的特殊指令。这些游戏在本模拟器中无法运行。

模拟器的源代码、使用方法、编译方法已经放在 GitHub: https://github.com/blanboom/bEMU

# 资料整理

这个模拟器中的代码并非全部由本人编写，而是参考了 [LiteNES](https://github.com/NJUOS/LiteNES), [nesemu2](https://github.com/holodnak/nesemu2), [mwillsey’s NES](https://github.com/mwillsey/NES) 等项目。在此对作者表示感谢。

另外，在完成这一模拟器中，发现了许多有用的资料和链接。现在整理在此处，希望能够对更多对 NES 感兴趣的人带来帮助。

-   [Nesdev Wiki](http://wiki.nesdev.com)  
    提供 NES 开发的相关信息，内容较为全面
-   [6502.org](http://www.6502.org)  
    提供 6502 CPU 的文档、教程，以及与之相关的个人项目汇总
-   [ewind 的 NES 模拟器笔记](http://ewind.us/tags/NES/) (JavaScript)
-   [NESASM 教程](https://yq.aliyun.com/articles/5784)  
    使用汇编语言开发 NES 程序的教程
-   [Writing an NES emulator in Go](http://nwidger.github.io/blog/post/writing-an-nes-emulator-in-go-part-1/)
-   [Easy 6502 by skilldrick](http://skilldrick.github.io/easy6502/)  
    互动式 6502 汇编教程
-   [NES EMULATION by Tom Gowing and Brian Pescatore](https://courses.cit.cornell.edu/ee476/FinalProjects/s2009/bhp7_teg25/bhp7_teg25/)  
    使用 AVR 单片机的 NES 模拟器
-   [6502CPU以及NES游戏机系统](http://49.212.183.201/6502/6502_report.htm)
-   [NES 光枪的工作原理](https://www.zhihu.com/question/32899950)

# 调试与优化

## NES 测试程序

完成 NES 模拟器后，需要对 CPU 的各个指令、PPU 的各项功能进行测试，确定模拟器是否运行正常。这时候，通过测试程序，能够更方便地完成这些功能。

在调试模拟器的过程中，我使用了 NEStress 和 nestest 两个测试程序。其中，NEStress 可以对 CPU、PPU、APU、IO 等进行测试，而 nestest 仅仅用于对 CPU 的测试，但比 NEStress 覆盖更多的指令。

这些测试程序的下载地址可从这里找到：http://wiki.nesdev.com/w/index.php/Emulator_tests

## 通过 SIGINFO 显示调试信息

在本模拟器中，还增加了显示 CPU、PPU 状态信息的功能。模拟器运行过程中，可通过 Ctrl+T，发送 SIGINFO，随时查看相关信息。

相关程序如下：

``` c
static void sig_info() {
    /* 显示时间 */
    static time_t timer;
    static struct tm * timeinfo;
    time(&timer);
    timeinfo = localtime(&timer);
    printf("%s\n", asctime(timeinfo));
 
    cpu_debugger();
    ppu_debugger();
    printf("--------------------------------------------\n\n");
 }
 ```

## CPU 模拟器的优化技巧

这篇文章介绍了作者在对 6502 模拟器的一些优化技巧，可供参考：http://www.slack.net/~ant/nes-emu/6502.html


# 其他想法

关于这个模拟器的更多打算。如果以后有时间，可能会考虑将其实现。

## 多线程

将 CPU 与 PPU 通过两个线程实现，两者并行执行，有可能能提高效率。不过需要面临两者之间的同步问题。

## 通过 FPGA 制作「片上 NES」

通过 FPGA，以硬件的方式实现 CPU 和 PPU，加深对 CPU 硬件结构的理解。

## 扩展至 32 位

尝试将 CPU 扩展至 32 位，扩充指令集，并兼容 6502 原有的指令集。

## 移植到 3DS

将模拟器移植到 3DS 上。


