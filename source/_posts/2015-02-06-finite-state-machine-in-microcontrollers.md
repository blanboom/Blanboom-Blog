---
author: Blanboom
layout: post
slug: finite-state-machine-in-microcontrollers
title: 有限状态机在单片机编程中的应用
date: 2015-02-06 22:02:00 +0800
comments: true
categories: 学习笔记
tags:
- 有限状态机
- 状态机
- 单片机
- 多任务
---

在单片机编程中，如果在不使用操作系统的情况下同时执行多个任务，可能会遇到下面这些情况：

* 一个任务的**执行时间过长**，导致其他任务无法及时执行
* 在一些任务中大量使用 delay() 等函数进行软件延时，这些**延时函数占用过多时间**，影响其他任务的执行
* 一些复杂任务的程序**逻辑不清晰**，不便于以后对程序进行维护，或添加新功能

本文介绍的有限状态机，可以做到将一个耗时较长的复杂任务**分解为多个简单任务**，同时使代码**逻辑更加清晰**，从而解决上述问题。

#### 目录：


* <a href="#toc_0">1. 什么是有限状态机</a>
* <a href="#toc_1">2. 有限状态机的作用</a>
	* <a href="#toc_2">2.1 分解耗时过长的任务</a>
	* <a href="#toc_3">2.2 避免软件延时对 CPU 资源造成浪费</a>
	* <a href="#toc_4">2.3 使程序逻辑更加清晰</a>
* <a href="#toc_5">3. 有限状态机的实现</a>
	* <a href="#toc_6">3.1 通过 switch - case 语句实现</a>
	* <a href="#toc_7">3.2 通过 Arduino 库实现</a>
	* <a href="#toc_8">3.3 其他方式</a>
* <a href="#toc_9">4. 示例一：按键去抖动程序的优化</a>
	* <a href="#toc_10">4.1 传统的按键去抖动程序</a>
	* <a href="#toc_11">4.2 优化后的按键去抖动程序</a>
* <a href="#toc_12">5. 示例二：通过有限状态机实现的闹钟程序</a>
* <a href="#toc_13">6. 后记</a>

<!-- more -->

<h1 id="toc_0">1. 什么是有限状态机</h1>

根据维基百科上的定义，有限状态机（finite-state machine, FSM，简称状态机）是表示有限个状态以及在这些状态之间的转移和动作等行为的数学模型。<sup id="fnref1"><a href="#fn1" rel="footnote">1</a></sup>

为了理解这句话，假设自己还有三天就要考试，这时候就要进入紧张的备考状态，将空闲时间用在**复习**上。但是，为了保证足够的精力，**小睡**一会儿也是十分有必要的。那么，什么时候复习，什么时候睡觉呢？可以这样描述：

>在复习的时候：

>>**如果** 感到瞌睡，**则** 睡觉

>>**如果** 没有感觉到瞌睡，**则** 继续复习

>在小睡的时候：

>>**如果** 感觉不再瞌睡，**则** 开始复习

>>**如果** 感觉依旧瞌睡，**则** 继续睡觉

也可通过一幅简单的示意图（也叫「状态转移图」）表示出来：

![图 1  复习与小睡](/images/2015/02/fsm_sleep.jpg)

这个例子其实就是一个简单的有限状态机，其中，复习和小睡是两个**状态**，感觉瞌睡和感觉清醒这两个**条件**可以使状态发生**转换**。<sup id="fnref2"><a href="#fn2" rel="footnote">2</a></sup>

另外，Programming Basics<sup id="fnref3"><a href="#fn3" rel="footnote">3</a></sup> 网站上也提供了状态机相关的教程，用形象化的图片解释了什么是有限状态机，可[通过此链接访问](http://www.programmingbasics.org/zh/beginner/fsm.html)。

在嵌入式程序设计中，如果一个系统需要处理**一系列连续发生的任务**，或在**不同的模式**下对输入进行不同的处理，常常使用有限状态机实现。例如测量、监测、控制等控制逻辑型应用。<sup id="fnref4"><a href="#fn4" rel="footnote">4</a></sup>

<h1 id="toc_1">2. 有限状态机的作用</h1>

<h2 id="toc_2">2.1 分解耗时过长的任务</h2>

大家应该都知道，CPU 没有并行执行任务的能力。**计算机「同时」运行多个程序，其实是多个程序依次交替执行**，给人以程序同时运行的错觉。各个程序在什么时候开始执行，执行多长时间后切换到下一个程序，由操作系统决定。

单片机执行多任务也是类似的过程，但由于其资源有限，为了节省对 CPU 和存储空间的占用，在很多情况下没有使用操作系统。这时，**单片机中运行的各个任务必须在一定时间内主动执行完毕，才能保证下一个任务能够及时执行**。

对于一些需要长时间执行的任务，例如按键去除抖动、读取和播放 MP3 文件等，采用有限状态机的方式，**将任务划分为多个小的步骤（状态），每次只执行其中的一步**。这样，其他任务就有机会「插入」到这个任务之中，确保了各个任务都能按时执行。

![图 2  状态机用于分解耗时过长的任务](/images/2015/02/fsm_adv_1.jpg)


<h2 id="toc_3">2.2 避免软件延时对 CPU 资源造成浪费</h2>

对于一些简单的程序，可通过 delay(), delay_ms() 之类的函数进行软件延时。这些延时函数，一般是通过将某个变量循环递加或递加，到达一定值后跳出循环，从而**通过消耗 CPU 时间实现了延时**。

这种方式虽然简单，但在延时函数执行的过程中，其他程序无法运行，消耗了大量 CPU 资源。而通过状态机，**有助于减少软件延时的使用，提高 CPU 利用率**。

![图 3  用状态机避免软件延时](/images/2015/02/fsm_adv_2.jpg)

请参考下文中的 **示例一：按键去抖动程序的优化**，这一例子展示了如何通过软件延时分解耗时较长的任务，同时减少软件延时的使用。


<h2 id="toc_4">2.3 使程序逻辑更加清晰</h2>

通过状态机，将一个复杂任务划分为多个状态，可以使程序清晰易懂，便于维护。以后想要添加、删除程序中的功能，都会变得非常容易。

![图 4  用状态机优化程序逻辑](/images/2015/02/fsm_adv_3.jpg)

下文中的 **示例二：通过状态机实现的闹钟** 展示了如何通过状态机优化程序逻辑。

<h1 id="toc_5">3. 有限状态机的实现</h1>

<h2 id="toc_6">3.1 通过 switch - case 语句实现</h2>

如果使用 C 语言，switch - case 语句，即可简单地实现有限状态机。

``` c
/* 定义各个状态所对应的数值 */
#define STATUS_A 0
#define STATUS_B 1
#define STATUS_C 2

/* 该变量的值即为当前状态机所处的状态 */
uint8_t currentStatus = STATUS_A;

/* 通过状态机实现的某个任务，
 * 需要放入 while(1) 等地方循环执行
 * /
void fsm_app(void)
{
	switch(currentStatus) /* 根据现在的状态执行相应的程序 */
	{
	case STATUS_A:  /* 状态 A */
		doThingsForStatusA(); /* 执行状态 A 中需要执行的任务 */
		/* 若满足状态转换的条件，则转换到另一个状态 */
		if(condition_1){ currentStatus = STATUE_B; }
		break;
	case STATUS_B:  /* 状态 B */
		doThingsForStatusB(); /* 执行状态 B 中需要执行的任务 */
		/* 若满足状态转换的条件，则转换到另一个状态 */
		if(condition_2){ currentStatus = STATUE_C; }
		if(condition_3){ currentStatus = STATUE_A; }
		break;
	case STATUS_C:  /* 状态 C */
		doThingsForStatusB(); /* 执行状态 B 中需要执行的任务 */
		/* 若满足状态转换的条件，则转换到另一个状态 */
		if(condition_4){ currentStatus = STATUE_A; }
		break;
	default:
		currentStatus = STATUE_A;
	}
}
```

通过这段程序，即可实现一个具有三个状态的状态机。状态转移图如下图所示：

![图 5  通过 switch - case 语句实现的状态机](/images/2015/02/fsm_example.jpg)

<h2 id="toc_7">3.2 通过 Arduino 库实现</h2>

对于 Arduino 用户，还可以使用 FSM Library 实现。这一库将有限状态机进行了封装，可以以更简洁的方式实现状态机。

下载地址及使用说明：[http://playground.arduino.cc/Code/FiniteStateMachine](http://playground.arduino.cc/Code/FiniteStateMachine)

<h2 id="toc_8">3.3 其他方式</h2>

对于一些更复杂的任务，使用 switch - case 语句，代码可能会太简洁。这时候，使用其他方式实现状态机，可能会更好。具体请查阅相关资料。

<h1 id="toc_9">4. 示例一：按键去抖动程序的优化</h1>

<h2 id="toc_10">4.1 传统的按键去抖动程序</h2>

初学单片机时，我们接触的按键去抖动程序一般是这样的<sup id="fnref5"><a href="#fn5" rel="footnote">5</a></sup>：

``` c
void keyscan()
{
	if(key1 == 0)         // 如果按键 1 按下
	{
		delayms(10);      // 延时 10ms，消除因干扰产生的抖动
			if(key1 == 0) // 再次检测按键 1，如果依旧按下
			{
				doSomething(); //此时说明按键 1 已按下，执行按键 1 需要执行的任务
				while(!key1);  // 等待按键释放
			}
	}
}
```

对应的流程图如下：

![图 6  传统的按键去抖动程序](/images/2015/02/psm_keyscan_1.jpg)

从流程图中可知，delayms() 延时函数和最后的等待按键释放的程序，会占用过多时间。

<h2 id="toc_11">4.2 优化后的按键去抖动程序</h2>

如果使用有限状态机的思路，可以按照下图方式实现：

![图 7  用状态机实现的按键去抖动程序](/images/2015/02/fsm_keyscan_2.jpg)

该状态机有三个状态，分别是**按键未按下**，**等待**，**按键按下**。当按键按下时，则会进入等待状态，若在等待状态中按键一直保持按下，说明按键已经稳定地按下，进入按键按下的状态，等待按键释放。程序代码如下：
``` c
/* 按键去抖动状态机中的三个状态 */
#define KEY_STATE_RELEASE    // 按键未按下
#define KEY_STATE_WAITING    // 等待（消抖）
#define KEY_STATE_PRESSED    // 按键按下（等待释放）

/* 等待状态持续时间
 * 需要根据单片机速度和按键消抖程序被调用的速度来进行调整
 */
#define DURIATION_TIME 40

/* 按键检测函数的返回值，按下为 1，未按下为 0 */
#define PRESSED 1
#define NOT_PRESSED 0

/* 按键扫描程序所处的状态
 * 初始状态为：按键按下（KEY_STATE_RELEASE）
 */
uint8_t keyState = KEY_STATE_RELEASE;

/* 按键检测函数，通过有限状态机实现
 * 函数在从等待状态转换到按键按下状态时返回 PRESSED，代表按键已被触发
 * 其他情况返回 NOT_PRESSED
 */
uint8_t keyDetect(void)
{
	static uint8_t duriation;  // 用于在等待状态中计数
	switch(keyState)
	{
	case KEY_STATE_RELEASE:
		if(readKey() == 1)     // 如果按键按下
		{
			keyState = KEY_STATE_WAITING;  // 转换至下一个状态
		}
		return NOT_PRESSED;    // 返回：按键未按下
		break;
	case KEY_STATE_WAITING:
		if(readKey() == 1)     // 如果按键按下
		{
			duriation++;
			if(duriation >= DURIATION_TIME)    // 如果经过多次检测，按键仍然按下
			{	// 说明没有抖动了，可以确定按键已按下
				duriation = 0;
				keyState = KEY_STATE_PRESSED;  // 转换至下一个状态
				return PRESSED;
			}
		}
		else  // 如果此时按键松开
		{	// 可能存在抖动或干扰
			duriation = 0;  // 清零的目的是便于下次重新计数
			keyState = KEY_STATE_RELEASE;  // 重新返回按键松开的状态
			return NOT_PRESSED;
		}
		break;
	case KEY_STATE_PRESSED:
		if(readKey() == 0)       // 如果按键松开
		{
			keyState = KEY_STATE_RELEASE;  // 回到按键松开的状态
		}
		return NOT_PRESSED;
		break;
	default:
		keyState = KEY_STATE_RELEASE;
		return NOT_PRESSED;
	}
}
```

该程序也可经过扩展，实现判断按键双击、长按等功能。只需增加相应的状态和转移条件即可。

<h1 id="toc_12">5. 示例二：通过有限状态机实现的闹钟程序</h1>

最近正在制作一个闹钟。这个闹钟支持播放 MP3 格式的闹钟声<sup id="fnref6"><a href="#fn6" rel="footnote">6</a></sup>，支持贪睡模式，同时还有一些功能打算以后再添加上。

为了使程序逻辑更加清晰，也为了更方便地添加新功能，我打算采用有限状态机实现。相关程序如下：

``` c
#include "App_Alarm.h"
#include "USART1.h"
#include <stdio.h>
#include "diag/Trace.h"

/* 相关常量定义 */
#define ALARM_MUSIC_END 0      // 闹钟音乐播放完毕
#define FORMAT_OK		0	   // 格式正确
#define FORMAT_ERROR	(-1)   // 格式错误

/* 输入信息定义
 * 作为函数的返回值供函数 getInput() 使用
 * getInput() 将获取并返回键盘或触摸屏等设备中输入的控制命令或闹钟时间值
 */
#define INPUT_ERROR    (-1)    // 输入格式错误
#define INPUT_CANCEL   (-2)    // 输入了「取消」命令
#define INPUT_SNOOZE   (-3)    // 输入了「小睡」命令
#define INPUT_ALARM_ON (-4)    // 输入了「打开闹钟」命令
#define NO_INPUT	   (-10)   // 没有输入

/* 输出信息定义
 * 作为为函数的参数供函数 displayMessege() 使用
 * displayMessege() 用于在显示屏上显示相关的提示信息
 */
#define MESSEGE_SET_ALARM_TIME 	(0)  // 提示：设置闹钟时间
#define MESSEGE_CLEAR			(1)  // 提示：已取消
#define MESSEGE_ALARM_IS_ON		(2)  // 提示：闹钟已打开
#define MESSEGE_WAITING			(3)  // 提示：等待闹钟响起
#define MESSEGE_SET_SNOOZE_TIME	(4)  // 提示：设置小睡时间
#define MESSEGE_GET_UP			(5)  // 提示：该起床了

/* 闹钟的状态 */
enum alarmStates
{
	ALARM_OFF,				// 闹钟关闭
	SET_ALARM_TIME,			// 设置闹钟时间
	WATING_FOR_ALARM,		// 等待闹钟响起
	PLAY_ALARM_MUSIC,		// 播放闹钟音乐
	SET_SNOOZE_TIME			// 设置贪睡时间
} alarmState = ALARM_OFF;	// 默认状态：闹钟关闭

/* 相关函数的定义 */
int16_t getInput(void);
void displayMessege(uint8_t);
void setAlarm(int16_t);
int16_t alarmTimeDiff(void);
int8_t playAlarmMusic(void);
void setSnooze(int16_t);
uint8_t checkAlarmFormat(int16_t);
uint8_t checkSnoozeFormat(int16_t);

/*
 * 闹钟主程序，需要放入 while(1) 中循环调用
 */
void alarmApp(void)
{
	int16_t input;		// 输入值暂存在这个变量中
	switch (alarmState) // 获取闹钟状态，下面程序将根据闹钟的状态执行相应的任务
	{
	/* 状态：闹钟关闭
	 * 在此状态中，将会不断检查是否打开闹钟，如果打开了闹钟，则会进入下一个状态：设置闹钟时间
	 */
	case ALARM_OFF:
		if (getInput() == INPUT_ALARM_ON)  // 检查是否打开了闹钟
		{   // 如果打开了闹钟
			displayMessege(MESSEGE_SET_ALARM_TIME);	// 在屏幕或串口上提示：请设置闹钟时间
			alarmState = SET_ALARM_TIME;			// 进入下一个状态：设置闹钟时间
		}
		break;
	/* 状态：设置闹钟时间
	 * 在此状态中，将会检查输入值，
	 * 如果
	 * 		输入“取消”命令，则取消闹钟设置，返回到闹钟关闭的状态
	 * 		输入闹钟时间格式错误，则状态不变，等待下一次重新输入
	 * 		输入了正确的闹钟时间，则设置闹钟，显示闹钟设置成功，并进入下一状态：等待闹钟响起
	 */
	case SET_ALARM_TIME:
		input = getInput();  		// 获取输入值
		if(input == INPUT_CANCEL)	// 如果输入了“取消”
		{
			displayMessege(MESSEGE_CLEAR);  // 显示“已取消”
			alarmState = ALARM_OFF;			// 进入状态：关闭闹钟
		}
		else if(checkAlarmFormat(input) == FORMAT_OK)	// 如果输入格式正确
		{
			displayMessege(MESSEGE_ALARM_IS_ON); // 显示“成功设置闹钟，闹钟已启动”
			setAlarm(input); // 根据输入值设置闹钟
			alarmState = WATING_FOR_ALARM; // 进入下一状态：等待闹钟响起
		}
		break;
	/* 状态：等待闹钟响起
	 * 在此状态中，将会检查是否到达闹钟时间，如果到达，则进入下一状态：播放闹钟音乐
	 * 同时，在此状态中也会检查输入，如果输入了“取消”的命令，则进入闹钟关闭的状态
	 */
	case WATING_FOR_ALARM:
		displayMessege(MESSEGE_WAITING); // 显示等待闹钟响起的信息，例如离闹钟响起还有多长时间
		if (alarmTimeDiff() <= 0) // 检查离闹钟响起还有多少时间，如果时间小于等于零（到达闹钟时间）
		{
			alarmState = PLAY_ALARM_MUSIC;  // 进入下一个状态：播放闹钟音乐
		}
		if(getInput() == INPUT_CANCEL) // 如果输入了“取消”命令
		{
			displayMessege(MESSEGE_CLEAR);
			alarmState = ALARM_OFF;      // 进入闹钟关闭的状态
		}
		break;
	/* 状态：播放闹钟音乐
	 * 在此状态中，将播放闹钟音乐，若播放完毕，进入闹钟关闭的状态
	 * 同时，在此状态中也会检查输入，
	 * 		如果输入了“小睡”的命令，则进入状态：设置小睡时间
	 * 		如果输入了“取消”的命令，则进入状态：闹钟关闭
	 */
	case PLAY_ALARM_MUSIC:
		displayMessege(MESSEGE_GET_UP);  // 显示消息：“该起床了”
		if(playAlarmMusic() == ALARM_MUSIC_END) // 播放闹钟音乐
		{ // 若音乐播放完毕
			displayMessege(MESSEGE_CLEAR);
			alarmState = ALARM_OFF; // 进入状态：闹钟关闭
		}
		input = getInput();
		if(input == INPUT_SNOOZE) // 若输入了“小睡”的命令
		{
			displayMessege(MESSEGE_SET_SNOOZE_TIME); // 显示消息：“请设置小睡时间”
			alarmState = SET_SNOOZE_TIME; // 进入状态：设置小睡时间
		}
		if(input == INPUT_CANCEL) // 若输入了“取消”命令
		{
			displayMessege(MESSEGE_CLEAR);
			alarmState = ALARM_OFF;   // 进入状态：闹钟关闭
		}
		break;
	/* 状态：设置小睡时间
	 * 在此状态中，将从输入获取小睡时间，并将闹钟时间加上小睡时间，进入状态：等待闹钟响起
	 */
	case SET_SNOOZE_TIME:
		input = getInput(); // 获取输入
		if(input == INPUT_CANCEL)
		{   // 若输入“取消”，则进入“闹钟关闭”的状态
			displayMessege(MESSEGE_CLEAR);
			alarmState = ALARM_OFF;
		}
		else if(checkSnoozeFormat(input) == FORMAT_OK)
		{   // 若输入格式正确
			setSnooze(input);  // 设置新的闹钟时间
			alarmState = WATING_FOR_ALARM;  // 进入状态：等待闹钟响起
		}
		break;
	default:
		displayMessege(MESSEGE_CLEAR);
		alarmState = ALARM_OFF;
	}
}
```
状态转移图如图所示：

![图 8  用状态机实现的闹钟](/images/2015/02/fsm_alarm.jpg)


<h1 id="toc_13">6. 后记</h1>
在单片机编程时，如果遇到代码复杂、任务占用时间过长等问题，可以尝试通过有限状态机解决。

之前写过一个[针对 Arduino 的合作式任务调度器](https://blanboom.org/2013/arduino-task-scheduler-library)。配合有限状态机，更有利于多任务处理。

另外，instructables 上的一篇文章通过三个实例演示了有限状态机在 Arduino 上的应用，如果感兴趣，可以通过这个链接阅读：[http://www.instructables.com/id/Arduino-Finite-State-Machine/](http://www.instructables.com/id/Arduino-Finite-State-Machine/)

</br></br>

#### 备注：

<div class="footnotes"><ol><li id="fn1">来源：<a href="http://zh.wikipedia.org/zh-cn/%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E6%9C%BA">http://zh.wikipedia.org/zh-cn/有限状态机</a>&nbsp;<a href="#fnref1">&#8617;</li><li id="fn2">为了便于理解，此处描述的状态机及状态转移图省略了一些内容，例如没有标明开始状态&nbsp;<a href="#fnref2">&#8617;</li><li id="fn3">Programming Basics 针对初学者的互动编程学习网站，网址为：<a href="http://programmingbasics.org">http://programmingbasics.org</a>&nbsp;<a href="#fnref3">&#8617;</li><li id="fn4">根据 <a href="http://www.ti.com/mcu/docs/litabsmultiplefilelist.tsp?sectionId=96&amp;tabId=1502&amp;literatureNumber=slaa402a&amp;docCategoryId=1&amp;familyId=342">Finite State Machines for MSP430 (Rev. A)</a> 翻译&nbsp;<a href="#fnref4">&#8617;</li><li id="fn5">改编自<a href="http://book.douban.com/subject/3413850/">《新概念51单片机C语言教程》</a>中相关内容&nbsp;<a href="#fnref5">&#8617;</li><li id="fn6">其实 MP3 播放程序也可以通过有限状态机实现，因为为了实现 MP3 播放持续时间较长（一首歌的时间），而且需要完成多个步骤（打开文件、读取文件、将数据发送到 MP3 解码芯片、告诉 MP3 解码芯片音乐播放完毕等）&nbsp;<a href="#fnref6">&#8617;</li></ol></div>
