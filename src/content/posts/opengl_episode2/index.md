---
title: 2、实现一个简单的OpenGL Demo
published: 2025-02-11
description: '在上一期的基础上添加简单的OpenGL绘制代码，实现三角形绘制'
image: '/post_cover_images/opengl_label.png'
tags: [OpenGL, Windows, C++, Cherno]
category: 'OpenGL'
draft: false 
lang: ''
---

## 开始之前
>Cherno的OpenGL视频教程: [YouTube](https://www.youtube.com/playlist?list=PLlrATfBNZ98foTJPJ_Ev03o2oq3-GGOS2) / [bilibili(中译)](https://www.bilibili.com/video/BV1Ni4y1o7Au/?spm_id_from=333.1387.homepage.video_card.click)</br>
>OpenGL在线接口文档: [docs.gl](https://docs.gl/)</br>
>OpenGL在线教程网址: [中文](https://learnopengl-cn.github.io/) / [英文](https://learnopengl.com/)

在[上一期](https://ljiaooo.github.io/posts/opengl_episode1/)中我们在Windows和Visual Studio搭建了OpenGL的开发运行环境并进行了环境测试，这一期我们将在之前的测试代码基础上添加简单的绘制代码，并了解OpenGL绘制大概流程。

## OpenGL绘制Pipeline
<img src="https://learnopengl-cn.github.io/img/01/04/pipeline.png" align=center></img></br>

如上图所示，想要利用OpenGL在屏幕上绘制图形，
1. 首先我们需要给GPU提供需要绘制的数据(Vertex Data)，这些数据可以包含绘制的顶点坐标，颜色，法向量等等，虽然名为"Vertex(顶点)"，可以包含的数据远远不止顶点坐标。下面我们在上一期的代码中添加Vertex Data定义：
    ```cpp
    ...
    if (glewInit() != GLEW_OK)
        std::cout << "Fail to init glew!" << std::endl;

    /* 定义绘制数据，这里只定义了三角形的三个顶点坐标 */
    float positions[6] = {
        -0.5f, -0.5f,
         0.0f,  0.5f,
         0.5f, -0.5f
    }

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    ...

    ```

2. 定义完数据后，我们需要将数据传递到GPU，这通常需要：
    - 调用**glGenBuffers**来生成GPU缓冲区对象，但是并不会分配显存
    - 调用**glBindBuffer**将缓冲区对象绑定到特定的缓冲区目标，之后作用于缓冲区目标的操作都会作用域该缓冲区对象（OpenGL状态机机制）
    - 调用**glBufferData**将内存中的数据传递到GPU显存</br>
    :::tip
    [docs.gl](https://docs.gl/)是一个非常好的OpenGL接口说明网站，包括了接口说明，参数说明以及用法案例等，使用OpenGL接口前查阅一下或许会避免很多意想不到的问题。
    :::
    ```cpp
    ...
    if (glewInit() != GLEW_OK)
        std::cout << "Fail to init glew!" << std::endl;

    /* 定义绘制数据，这里只定义了三角形的三个顶点坐标 */
    float positions[6] = {
        -0.5f, -0.5f,
         0.0f,  0.5f,
         0.5f, -0.5f
    }

    /* 将数据传递到GPU */
    unsigned int buffer;                    // 用于存储申请的缓冲区对象的标识
    glGenBuffers(1, &buffer);               // 申请1个缓冲区对象，并将标识存储于buffer变量
    glBindBuffer(GL_ARRAY_BUFFER, buffer);  // 将缓冲区对象buffer绑定到GL_ARRAY_BUFFER缓冲区目标

    /* 将缓冲区数据拷贝到GPU显存，注意这里并没有调用buffer变量，因为经过上一步绑定后，所有作用于GL_ARRAY_BUFFER的操作将作用于buffer变量。 */
    glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), positions, GL_STATIC_DRAW); 

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    ...
    ```

3. 我们在内存定义了绘制数据，并且将这些数据传递到了GPU。然而GPU只是获取到了一堆数据，我们还需要告诉GPU如何组织这些数据，例如上述**positions**数组，对于GPU而言只是拿到了6 * sizeof(float)个字节的数据而并不知道这是三角形三个顶点的二维坐标。在OpenGL中，通过如下方式告诉GPU数据的组织方式：
    ```cpp
    ...
    glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), positions, GL_STATIC_DRAW); 

    /* 在上述代码基础上继续添加 */
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 2, GL_FLOAT, false, 2 * sizeof(float), 0);  // 告诉GPU数据组织方式，详细用法参照docs.gl
    ```

4. 到目前为止，我们向GPU传输了数据同时通知了GPU数据的组织方式，下一步就需要告诉GPU如何进行绘制。作为开发者，我们通知GPU绘制方式的方法就是编写着色器（shader），着色器的本质就是运行在GPU的程序，类似于CPU程序，着色器也需要经过编译，链接等过程。最常用的两个着色器分别是顶点着色器（vertex shader）和片段着色器（fragment shader），如[上图](#opengl绘制pipeline)所示，vertex shader负责坐标变换，传递数据等；而fragment shader用于计算每一个像素的颜色，这包括考虑纹理，材质，光照等等因素。</br>
顶点着色器为每一个顶点（例子中为三角形的3个顶点）执行一次，而片段着色器则需要为每一个像素（例子中为三角形内的每一个像素）执行一次，因此片段着色器的执行次数可以非常多，我们应该避免在片段着色器中执行过多复杂操作。
    - **在OpenGL中使用着色器**</br>
    如上面所说，着色器的本质是一个运行在GPU的程序。程序需要有文本形式的源代码，在OpenGL中，使用高级shader语言GLSL作为编写着色器的高级编程语言，其语法与C相似。编写好源代码后，OpenGL分别提供了**glCompileShader**和**glLinkProgram**两个接口，来进行着色器的编译以及链接到程序，可以将多个着色器链接到同一个程序。</br>
    我们可以为顶点着色器和片段着色器编写统一的接口：
    ```cpp
    /*
    * @brief 根据顶点着色器和片段着色器源代码创建一个着色器程序，这个着色器程序会在顶点处理器和片段处理器上执行。
    * @param vertexShader 顶点着色器源代码
    * @param fragmentShader 片段着色器源代码
	*
    * @return 返回值为编译链接好的程序的句柄
    */
    static unsigned int CreateShader(const std::string& vertexShader, 
                                     const std::string& fragmentShader)
    {
	    unsigned int program = glCreateProgram();
	    unsigned int vs = CompileShader(GL_VERTEX_SHADER, vertexShader);
	    unsigned int fs = CompileShader(GL_FRAGMENT_SHADER, fragmentShader);

	    glAttachShader(program, vs);
	    glAttachShader(program, fs);
	    glLinkProgram(program);
	    glValidateProgram(program);

	    glDeleteShader(vs);
	    glDeleteShader(fs);

	    return program;
    }

    /*
    * @brief 编译着色器
    * @param type 着色器类型
    * @param source 着色器源代码
    * 
    * @return 返回编译好的着色器的句柄
    */
    static unsigned int CompileShader(unsigned int type, const std::string& source)
    {
	    unsigned int id = glCreateShader(type);
	    const char* src = source.c_str();
	    glShaderSource(id, 1, &src, nullptr);
	    glCompileShader(id);

	    int result;
	    glGetShaderiv(id, GL_COMPILE_STATUS, &result);
	    if (GL_FALSE == result)
	    {
	    	int length;
	    	glGetShaderiv(id, GL_INFO_LOG_LENGTH, &length);
	    	char* message = (char*)alloca(sizeof(char) * length);
	    	glGetShaderInfoLog(id, length, &length, message);
	    	std::cout << "Fail to compile " << (type == GL_VERTEX_SHADER ? "vertex" : "fragment") << " shader" << std::endl;
	    	std::cout << message << std::endl;
	    	glDeleteShader(id);
	    	return 0;
	    }

	    return id;
    }
    ```

    - **顶点着色器**</br>
    着色器的源代码由高级着色器语言GLSL编写而成，代码可以存储在文件中也可以直接以字符串的形式在代码中提供。下面展示的是直接在代码中以字符串的形式提供顶点着色器源代码。</br>
    代码中的layout(location = 0)与前面代码中的glEnableVertexAttribArray(0);中的0相对应，表示传入第0个顶点属性。

    ```cpp
    std::string vertexShader =
		"#version 330 core\n"
		"\n"
		"layout(location = 0) in vec4 position;\n"
		"\n"
		"void main()\n"
		"{\n"
		"	gl_Position = position;\n"
		"}\n";
    ```
    
    - **片段着色器**</br>
    片段着色器输出每个像素的颜色，即下述代码中的color属性。out vec4 color;用于声明输出四维向量变量color。

    ```cpp
    std::string fragmentShader =
		"#version 330 core\n"
		"\n"
		"out vec4 color;\n"
		"void main()\n"
		"{\n"
		"	color = vec4(1.0, 0.0, 0.0, 1.0);\n"
		"}\n";
    ```

## 完整代码
依据上述OpenGL绘制流程及每个流程的示例代码，得到如下完整绘制一个红色三角形的源代码。
```cpp
#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <iostream>

static unsigned int CompileShader(unsigned int type, const std::string& source)
{
	unsigned int id = glCreateShader(type);
	const char* src = source.c_str();
	glShaderSource(id, 1, &src, nullptr);
	glCompileShader(id);

	int result;
	glGetShaderiv(id, GL_COMPILE_STATUS, &result);
	if (GL_FALSE == result)
	{
		int length;
		glGetShaderiv(id, GL_INFO_LOG_LENGTH, &length);
		char* message = (char*)alloca(sizeof(char) * length);
		glGetShaderInfoLog(id, length, &length, message);
		std::cout << "Fail to compile " << (type == GL_VERTEX_SHADER ? "vertex" : "fragment") << " shader" << std::endl;
		std::cout << message << std::endl;
		glDeleteShader(id);
		return 0;
	}

	return id;
}

static unsigned int CreateShader(const std::string& vertexShader, const std::string& fragmentShader)
{
	unsigned int program = glCreateProgram();
	unsigned int vs = CompileShader(GL_VERTEX_SHADER, vertexShader);
	unsigned int fs = CompileShader(GL_FRAGMENT_SHADER, fragmentShader);

	glAttachShader(program, vs);
	glAttachShader(program, fs);
	glLinkProgram(program);
	glValidateProgram(program);

	glDeleteShader(vs);
	glDeleteShader(fs);

	return program;
}

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

	float positions[6] = {
		-0.5f, -0.5f,
		 0.0f,  0.5f,
		 0.5f, -0.5f
	};

	unsigned int buffer;
	glGenBuffers(1, &buffer);
	glBindBuffer(GL_ARRAY_BUFFER, buffer);
	glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), positions, GL_STATIC_DRAW);

	glEnableVertexAttribArray(0);
	glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, 2 * sizeof(float), 0);

	std::string vertexShader =
		"#version 330 core\n"
		"\n"
		"layout(location = 0) in vec4 position;\n"
		"\n"
		"void main()\n"
		"{\n"
		"	gl_Position = position;\n"
		"}\n";
	std::string fragmentShader =
		"#version 330 core\n"
		"\n"
		"layout(location = 0) out vec4 color;\n"
		"void main()\n"
		"{\n"
		"	color = vec4(1.0, 0.0, 0.0, 1.0);\n"
		"}\n";

	unsigned int shader = CreateShader(vertexShader, fragmentShader);
	glUseProgram(shader);

	/* Loop until the user closes the window */
	while (!glfwWindowShouldClose(window))
	{
		/* Render here */
		glClear(GL_COLOR_BUFFER_BIT);

		glDrawArrays(GL_TRIANGLES, 0, 3);

		/* Swap front and back buffers */
		glfwSwapBuffers(window);

		/* Poll for and process events */
		glfwPollEvents();
	}

	glDeleteProgram(shader);
	glfwTerminate();
	return 0;
}
```

---

- 如果成功运行上述代码，我们将得到一个红色三角形。
<div align=center> 

![](/src/content/posts/opengl_episode2/result.png)
</div>



















