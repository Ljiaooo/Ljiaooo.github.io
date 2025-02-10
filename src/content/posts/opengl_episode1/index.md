---
title: 在windows创建一个简单的OpenGL示例
published: 2025-02-10
description: '根据Youtube上Cherno博主的教程，从搭建Windwos上的OpenGL环境到创建第一个可以运行的OpenGL示例程序。'
image: '/:post_cover_images/opengl_label.png'
tags: [OpenGL, Cherno]
category: 'OpenGL_Cherno'
draft: true 
lang: ''
---

# 开始之前
## OpenGL
OpenGL事实上只是一个图形编程规范，其作用类似于API，仅仅规范图形编程函数如何执行，以及应该得到怎样的结果，而函数内部的实现细节则并没有规定。OpenGL内部的具体实现依赖于各个GPU厂商，实现细节通常包含在GPU驱动之中，这也意味着不同GPU厂商的OpenGL实现是不相同的。

## GLFW
OpenGL依赖窗口系统以及OpenGL上下文(context)，而它们都与操作系统密切相关。GLFW库提供了跨屏台的窗口系统管理以及OpenGL上下文创建和管理，让我们可以更加专注于OpenGL程序的编写而不是繁杂的系统级别库上。

## GLEW
现代OpenGL提供了更多的特性与扩展，而这些扩展并非所有GPU都全部实现，因此我们不得不调用系统级别的库来检查对应的OpenGL接口是否包含。GLEW库实现跨平台的OpenGL函数自动加载，以及扩展管理，而不需要开发者针对不同系统编写不同扩展判断和调用代码。
