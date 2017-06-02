---
author: Blanboom
layout: post
slug: omnifocus-quick-entry-applescript-v2
title: OmniFocus 快速收集脚本 v2
date: 2014-11-04 21:48:00 +0800
comments: true
categories: 软件与工具
tags:
- OmniFocus
- AppleScript
---

之前写过一个 [OmniFocus 快速收集脚本](./omnifocus-quick-entry-applescript.html)，可以通过此脚本打开 OmniFocus 的 Quick Entry 窗口，快速将自己的灵感或者想做的事记录下来。但是，OmniFocus 的启动速度不是很快，第一次运行脚本后需要等上几秒钟，窗口才能出现，使用体验不是很好。

刚刚对这个脚本进行了一点改进，执行脚本时，会弹出一个简单的对话框，在对话框中输入要保存的内容即可，无需等待 OmniFocus 启动。

![脚本运行截图](/images/2014/11/OmniFocus-Script.png)

<!-- more -->

AppleScript 代码如下，可以在 Automator 中保存为服务，用快捷键随时启动。或打包成 app, 通过 Spotlight 启动：

    set tmp to display dialog "What do you want to do?" default answer ""
    set taskString to text returned of tmp
    
    set isRunning to false
    tell application "System Events"
        if exists process "OmniFocus" then
            set isRunning to true
        end if
    end tell
    
    if isRunning is true then
        tell application "OmniFocus"
            tell front document
                make new inbox task with properties {name:(taskString)}
            end tell
        end tell
    else
        tell application "OmniFocus" to activate
        
        tell application "OmniFocus"
            tell front document
                make new inbox task with properties {name:(taskString)}
            end tell
        end tell
    end if


#### 参考资料：

1. [Create new task in OmniFocus inbox](http://www.alfredforum.com/topic/1041-create-new-task-in-omnifocus-inbox/)
