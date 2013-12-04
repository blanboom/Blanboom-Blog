---
author: Blanboom
comments: true
date: 2013-06-07 15:45:15+00:00
layout: post
slug: mcu-double-click-detect
title: '[51学习笔记]按键双击、连按的检测'
wordpress_id: 24
categories:
- 学习笔记
tags:
- '51'
- 单片机
- 按键
---

昨晚上课时，老师说：“用单片机检测按键的双击，看似简单，但实现起来，需要一点小技巧。”

这句话引起了我的兴趣，我打算自己尝试一下。经过折腾，算是解决了这个问题。

<!-- more -->

Update(2013.8.3): 最近看了 [从单片机初学者迈向单片机工程师](https://www.google.com/search?q=%E4%BB%8E%E5%8D%95%E7%89%87%E6%9C%BA%E5%88%9D%E5%AD%A6%E8%80%85%E8%BF%88%E5%90%91%E5%8D%95%E7%89%87%E6%9C%BA%E5%B7%A5%E7%A8%8B%E5%B8%88) 这个帖子，里面有更合理的方法，值得借鉴。


# 最初的思路


当判断按键按下后，延时适当时间，再判断一次按键是否按下，如果按下，说明按键双击，如果没按下，说明是单击。

不过，这样做的话，对两次按键的时间间隔有要求，不能过长或过短。实际测试时确实如此，基本上不能正常、稳定地使用。
程序如下：

    
    //51单片机的键盘双击检测，简易版
    //程序功能：单击按键灭灯，双击亮灯
    //By Blanboom
    //http://blanboom.org
    //2013.6.6
    
    #include <reg52.h>
    #define uint unsigned int
    #define uchar unsigned char
    sbit LED=P0^4;
    sbit KEY=P3^4;
    
    void delay(uint time);       //毫秒级延时子程序
    
    int main()
    {	
    	LED=0;
    	while(1)
    	{
    		if(KEY==0)				//先判断是否按键
    		{
    			delay(10);
    			if(KEY==0)
    			{
    				while(!KEY);	    
    				delay(100);
    				if(KEY==0)		//如果按键，则判断是否有第二次按键
    				{
    					LED=0;		//如果第二次按键（双击），则灯亮
    				}
    				else
    				{
    					LED=1;		 //如果第二次没按键，则灯不亮
    					while(!KEY); //按键松下后才能继续执行。
    					delay(50);	 //消除抖动
    				}		
    			}	
    		}
    	}			
    }
    
    void delay(uint time)			  
    {
    	uint i,j;
    	for(i=110;i>0;i--)
    		for(j=time;j>0;j--);
    }





# 最终方案


后来，上网找了一些资料，通过判断 **每次按键按下之后的一段时间内，判断按键是否再次按下** 的方法，把问题解决了：

    
    //51单片机的键盘双击检测，定时器版
    //程序功能：单击按键灭灯，双击亮灯
    //By Blanboom
    //http://blanboom.org
    //晶振频率：11.0592M
    //2013.6.7
    
    #include <reg52.h>
    #define uint unsigned int
    #define uchar unsigned char
    sbit LED = P0^4;		 //LED所连接的IO口
    sbit KEY = P3^4;		 //按键所连接的IO口
    #define KEY_AVALIABLE 0	 //值为0时，按键低电平触发；值为1时，按键高电平触发
    #define LED_AVALIABLE 0	 //值为0时，低电平LED亮；值为1时，高电平LED亮
    #define MAX_WAIT_TIME 4  //按键后等待另一次按键的时间，单位：50ms
    
    uint wait_time = 0;		 //按键后的等待时间
    uchar key_state = 0;	 //按键次数
    
    void timer1_initial();	 //定时器初始化
    void delay(uint time);	 //毫秒级延时子程序
    uchar key_read();		 //按键读取程序
    
    void main()
    {
    	//LED，按键，定时器的初始化
        LED = LED_AVALIABLE;
    	KEY = !KEY_AVALIABLE;
        timer1_initial();  
    
        while(1)
        {
    		uchar key_state_temp;
    		key_state_temp = key_read();
    		if(key_state_temp == 1)
    		{
    			LED = !LED_AVALIABLE;		//按键1次，灯灭
    		}
    		else if(key_state_temp >=2)
    		{
    			LED = LED_AVALIABLE;		//按键大于1次，灯亮
    		}
    	}
    }
    
    void timer1() interrupt 3		//定时器1：50ms
    {
        TH1 = (65536-45872)/256;
        TL1 = (65536-45872)%256;
    	wait_time++;
    }
    
    void timer1_initial()			 //定时器初始化
    {
        TH1 = (65536-45872)/256;
        TL1 = (65536-45872)%256;             
        IE = 0x88;                             
        TMOD = 0x10;                     
        TR1 = 1;                         
    }
    
    void delay(uint time)
    {
    	uint i,j;
    	for(i=110;i>0;i--)
    		for(j=time;j>0;j--);
    }
    
    uchar key_read()				         //读取按键,并返回按键次数
    {	
    	uchar key_state_temp = 0;
    	if(KEY == KEY_AVALIABLE)
    	{
    		delay(10);
    		if(KEY == KEY_AVALIABLE);
    		{
    			while(KEY == KEY_AVALIABLE);
    			wait_time = 0;			     //按键后重新计时
    			key_state++;
    		}
    	}
    
    	//超过一定时间没按键，函数将返回按键次数
    	if(wait_time >= MAX_WAIT_TIME)	 
    	{
    		wait_time = 0;
    		key_state_temp = key_state;
    		key_state = 0;
    	}
    
    	return key_state_temp;
    }


实际测试中，这种方法已经能够正常使用了。如果有更好的办法欢迎告诉我。
