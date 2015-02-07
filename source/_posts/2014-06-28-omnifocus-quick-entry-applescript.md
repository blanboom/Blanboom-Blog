---
author: Blanboom
layout: post
slug: omnifocus-quick-entry-applescript
title: OmniFocus 快速收集脚本
date: 2014-06-28 23:21:00 +0800
comments: true
categories: 软件与工具
tags:
- OmniFocus
- AppleScript
---

刚刚换用微软的一款无线键盘，键盘上有 5 个快速启动按键，按下按键即可运行指定的程序。配合这一脚本使用，就能做到**一键打开 OmniFocus 的快速收集窗口**，实现随时记录与收集。

<!-- more -->

**Update: [OmniFocus 快速收集脚本 v2](http://blanboom.org/omnifocus-quick-entry-applescript-v2.html)**

AppleScript 代码如下：

    set isRunning to false
     
    tell application "System Events"
    	if exists process "OmniFocus" then
    		set isRunning to true
    	end if
    end tell
     
    if isRunning is true then
    	tell application "System Events"
    		keystroke " " using {control down, option down}
    		-- 按下 Qiuck Entry 快捷键
    	end tell
    else
    	tell application "OmniFocus" to activate
    	
    	tell application "OmniFocus"
    		set miniaturized of window 1 to true
    	end tell
    	
    	tell application "System Events"
    		keystroke " " using {control down, option down}
    		-- 按下 Qiuck Entry 快捷键
    	end tell
    end if

顺便推荐下 [cd to...](https://github.com/jbtule/cdto)，这个 App 可以**在 Finder 的当前目录中打开终端**。和键盘上的快速启动按键一起使用，更加方便。

对于 Alfred 2 用户，也可以直接使用 [OmniFocus Inbox Task Workflow](http://www.alfredforum.com/topic/1041-create-new-task-in-omnifocus-inbox/)，功能与本脚本类似。



 <br />
 <br />
 <br />

参考资料：

1. [用AppleScript在Mac系统下实现按键精灵的功能以及在游戏中的运用](http://blog.xcodev.com/archives/auto-key-press-using-appscript/)
