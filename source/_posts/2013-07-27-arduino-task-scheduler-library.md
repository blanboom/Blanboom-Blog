---
author: Blanboom
comments: true
date: 2013-07-27 09:45:47+00:00
layout: post
slug: arduino-task-scheduler-library
title: '[Arduino库]任务调度器，更好地处理多任务'
wordpress_id: 33
categories:
- 制作
tags:
- Arduino
- Arduino库
- 多任务
- 调度器
---

一般情况下，处理 Arduino 的多个任务，是把所有任务放在 void loop() 里，然后用 delay() 控制时间。不过，任务一多，这种方法就不太方便了。

最近刚刚看了一本书：《[时间触发嵌入式系统设计模式](http://book.douban.com/subject/1168618/)》，里面介绍的调度器，可以以特定的周期执行特定的任务，值得在 Arduinio 项目中借鉴。我也刚刚把这个调度器移植到 Arduino 中：[https://github.com/blanboom/Arduino-Task-Scheduler](https://github.com/blanboom/Arduino-Task-Scheduler)


# 基本使用方法


这是一个使用调度器的例子，各个函数的功能都已在注释中标出：
<!-- more -->

    
    // Arduino 任务调度器 演示程序
    // Created by Blanboom
    // 2013.7.27
    // http://blanboom.org
    
    #include "TaskScheduler.h"  //包含此头文件，才能使用调度器
    
    // 用于储存 LED 状态
    boolean g_led1State=1;
    boolean g_led2State=0;
    
    void setup()
    {
    	// 第12、13脚接有 LED
    	pinMode(13,OUTPUT);
    	pinMode(12,OUTPUT);
    
    	Sch.init(); //初始化调度器
    
    	//向调度器中添加任务
    	//第一个参数为要添加任务的函数名
    	//第二个参数为任务第一次执行的时间，
    	//    合理设置有利于防止任务重叠，有利以提高任务执行的精度
    	//第三个参数是任务执行的周期
    	//第二、三个参数的单位均为毫秒，也可配置定时器修改其单位
    	//第四个参数代表任务是合作式还是抢占式
    	//    一般取1就可以，更多用法请参考下文
    	Sch.addTask(led1Update,0,1000,1);  //从第 0 毫秒开始闪烁 LED，每隔 1s, LED 状态改变一次
    	Sch.addTask(led2Update,20,500,1);  //从第 20 毫秒开始闪烁 LED，每隔 0.5s, LED 状态改变一次
    
    	Sch.start();//启动调度器
    }
    
    void loop()
    {
    	Sch.dispatchTasks();  // 执行被调度的任务，用调度器时放上这一句即可
    }
    
    // 把要调度的任务函数放下面
    
    // 闪烁第 13 脚的 LED
    void led1Update()
    {
    	if(g_led1State==0)
    	{
    		g_led1State=1;
    		digitalWrite(13,HIGH);
    	}
    	else
    	{
    		g_led1State=0;
    		digitalWrite(13,LOW);
    	}
    }
    
    // 闪烁第 12 脚的 LED
    void led2Update()
    {
    	if(g_led2State==0)
    	{
    		g_led2State=1;
    		digitalWrite(12,HIGH);
    	}
    	else
    	{
    		g_led2State=0;
    		digitalWrite(12,LOW);
    	}
    }


程序执行后，两个 LED 分别会以程序中指定的周期和时间闪烁。


# 更多功能


**1. 添加抢占式任务**

抢占式任务，简单说，就是优先级比正常任务（合作式任务）高的任务。在这个调度器中，抢占式任务可以打断正常任务，优先执行。

对于一些对时间精度要求较高的任务，可以将任务模式改为抢占式。

修改方法：

在添加任务的函数 Sch.addTask(任务名称,开始时间,执行周期,1) 函数中，将最后一个参数由 1 改为 0，即：

Sch.addTask(任务名称,开始时间,执行周期,1)

这样，该任务就成了抢占式任务。

**2. 添加单次执行的任务**

可以添加只执行一次的任务，在一段时间后执行。

只需把 Sch.addTask(任务名称,开始时间,执行周期,1) 中的执行周期改为 0 即可。

**3. 删除任务**

使用函数 Sch.addTask(任务名称,开始时间,执行周期,1) 时，会返回这个任务的 ID，将这个 ID 赋给一个变量。需要删除任务时，用删除任务函数 Sch.DeleteTask(任务ID) ，就能把任务删除。

**4. 调整被调度的任务数量**

打开 TaskScheduler.h，找到 #define MAX_TASKS (10) ，将 10 修改为需要被调度的任务的数量。

**5. 自动进入空闲模式**

这个调度器能在没有任务的情况下自动进入空闲模式，以节省电量。不需要对程序进行其他修改。

**6. 错误报告**

打开 TaskScheduler.h，找到

//#define REPORT_ERRORS // Remove "//" to enable error report，

将前面的 // 去掉，打开错误报告功能。

然后，这条语句的下面，定义了相关错误代码，可根据情况修改。

最后，打开 TaskScheduler.cpp，找到函数 void Schedule::_reportStatus(void)，在里面添加合适的错误报告代码即可。

欢迎大家对这个调度器进行测试，找出 bug 和需要优化的地方。
