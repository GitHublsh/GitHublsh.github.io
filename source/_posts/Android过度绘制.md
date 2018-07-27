---
title: Android过度绘制
date: 2018-07-27 11:17:23
tags: [Android]
---
#### 1.什么是过度绘制？

过度绘制（Overdraw）描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次重叠的 UI 结构里面，如果不可见的 UI 也在做绘制的操作，会导致某些像素区域被绘制了多次，同时也会浪费大量的 CPU 以及 GPU 资源，绘制示意图如下图所示

![overdraw](http://ot29getcp.bkt.clouddn.com//blogoverdraw.png)