---
title: 搭建Windows & Visual Studio的OpenGL环境
published: 2025-02-10
description: '根据Youtube上Cherno博主的教程搭建Windows & Visual Studio上OpenGL运行环境'
image: '/post_cover_images/opengl_label.png'
tags: [OpenGL, Cherno]
category: 'OpenGL_Cherno'
draft: false 
lang: ''
---

# 开始之前
## OpenGL
OpenGL事实上只是一个图形编程规范，其作用类似于API，仅仅规范图形编程函数如何执行，以及应该得到怎样的结果，而函数内部的实现细节则并没有提供。OpenGL内部的具体实现依赖于各个GPU厂商，实现细节通常包含在GPU驱动之中，这也意味着不同GPU厂商的OpenGL实现是不相同的。

## GLFW
OpenGL依赖窗口系统以及OpenGL上下文(context)，而它们都与操作系统密切相关。GLFW库提供了跨屏台的窗口系统管理以及OpenGL上下文创建和管理，让我们可以更加专注于OpenGL程序的编写而不是繁杂的系统级别库上。

## GLEW
现代OpenGL提供了更多的特性与扩展，而这些扩展并非所有GPU都全部实现，因此我们不得不调用系统级别的库来检查对应的OpenGL接口是否包含。GLEW库实现跨平台的OpenGL函数自动加载，以及扩展管理，而不需要开发者针对不同系统编写不同扩展判断和调用代码。

# Windows & Visual Studio 2022环境搭建
## 依赖库
除了编译器自带的库以外，还需要上述提到过的三方库，包括GLFW和GLEW。OpenGL（opengl32.lib）库通常已经包含在了Windows SDK中。
- [GLFW](https://github.com/glfw/glfw/releases/download/3.4/glfw-3.4.bin.WIN32.zip)，我们可以直接下载32位的二进制版本，注意这里的32位指的是编译出来的程序而非系统的比特数。
- [GLEW](https://sourceforge.net/projects/glew/files/latest/download)，同样，我们下载GLEW的预编译版本即可。
## 静态库 OR 动态库
GLFW与GLEW均提供了静态和动态库两种预编译库形式，这里我们并没有动态加载库的需求，直接使用静态库即可，因此对于两个三方库我们仅需要其中的**include**文件夹和**lib**文件。以下是整理之后的Visual Studio工程及两个三方库的目录结构。
```
./
├── Dependencies/
│   ├── GLFW/ 
│   │   ├── include/
│   │   └── lib/
│   │       └──glfw3.lib
│   └── GLEW/
│   │   ├── include/
│   │   └── lib/
│   │       └──glew32s.lib
├── ProjectName/
└── ProjectName.sln
```
## 将三方库添加到工程中
1. 现在我们已经下载好了需要的三方库，但是还不能直接在Visual Studio的工程中使用。为了让Visual Studio能够查到到我们的头文件，我们首先需要将两个库的头文件目录添加到工程中。
    ```
    工程属性-> C/C++ -> 常规 -> 附加包含目录
    添加$(SolutionDir)Dependencies\GLFW\include;$(SolutionDir)Dependencies\GLEW\include，与其他项用";"隔开
    ```
2. 除了头文件外，我们还需要将静态库的路径添加到链接器的搜索路径中。
    ```
    工程属性 -> 链接器 -> 常规 -> 附加库目录
    添加$(SolutionDir)Dependencies\GLFW\lib;$(SolutionDir)Dependencies\GLEW\lib;

    添加完链接器搜索路径后还需要添加实际链接期间需要使用的静态库
    工程属性 -> 链接器 -> 常规 -> 附加依赖项
    添加glfw3.lib;glew32s.lib;opengl32.lib，包含两个三方库以及OpenGL库
    ```
3. 由于我们使用的是静态的glew库，我们还需要在预处理器中声明**GLEW_STATIC**宏
    ```
    工程属性-> C/C++ -> 预处理器 -> 预处理器定义
    ``
    添加GLEW_STATIC

# 环境测试
我们可以使用下面代码来对上述OpenGL运行环境进行测试（代码来自glfw，glew文档）</br>
运行后仅会展示一个黑色的窗口(因为我们还什么也没有绘制，仅仅只是创建了一个窗口)。

```cpp
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <iostream>

int main(void)
{
    GLFWwindow* window;

    /* Initialize the library */
    if (!glfwInit())
        return -1;

    /* Create a windowed mode window and its OpenGL context */
    window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);

    if (glewInit() != GLEW_OK)
        std::cout << "Fail to init glew!" << std::endl;

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Render here */
        glClear(GL_COLOR_BUFFER_BIT);

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```

To be continued...

    