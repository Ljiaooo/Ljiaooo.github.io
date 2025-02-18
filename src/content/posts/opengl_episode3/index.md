---
title: 3、OpenGL shader代码文件加载与解析
published: 2025-02-18
description: '将shader代码放进单独的文件进行加载和解析'
image: '/post_cover_images/opengl_label.png'
tags: [OpenGL, Windows, C++, Cherno]
category: 'OpenGL'
draft: false 
lang: ''
---

## 开始之前
在[上一期](https://ljiaooo.github.io/posts/opengl_episode2/)中，我们编写了一个最简单的shader代码，并且利用这个shader实现了一个红色三角形的绘制。我们之前直接在CPP中以字符串的形式编写了GLSL着色器代码，这通常并不利于着色器代码的管理，因此这一期我们将着色器代码拿出来放置在单独的文件中，并在主程序加载着色器代码。

## 转移shader代码
在工程中新建**res/shader/basic.shader**文件，将上期提到的shader代码拷贝过去并移除冒号及换行符，分别在顶点着色器和片段着色器代码上添加"#shader vertex"和"#shader fragment"用于在解析文件时区别两个着色器代码。</br>
顶点着色器和片段着色器的代码也可以分开放置在两个shader文件，这里我们遵循Cherno的视频，将两个shader的源码放置在同一个文件进行解析。
```glsl
#shader vertex
#version 330 core

layout(location = 0) in vec4 position;

void main()
{
	gl_Position = position;
};

#shader fragment
#version 330 core

layout(location = 0) out vec4 color;
void main()
{
	color = vec4(1.0, 0.0, 0.0, 1.0);
};
```

## shader文件解析器
- **首先定义一个结构体用于同时返回两个shader的源代码**
	```cpp
	struct shaderProgramSource
	{
		std::string vertexSource;
		std::string fragmentSource;
	};
	```
- **编写shader解析代码**</br>
	这里直接使用cpp提供的文件流的库来打开文件流，定义**std::string ss[2]**用于将文件流中的字符串输出到字符串流，并根据**ShaderType**枚举类保存当前解析到的shader类型。
	```cpp
	static shaderProgramSource ParseShader(std::string filepath)
	{
		enum class ShaderType
		{
			NONE = -1, VERTEX = 0, FRAGMENT = 1
		};
		std::ifstream stream(filepath);
		std::stringstream ss[2];
		std::string line;
		ShaderType type = ShaderType::NONE;
		while (getline(stream, line))
		{
			if (line.find("#shader") != std::string::npos)
			{
				if (line.find("vertex") != std::string::npos)
					type = ShaderType::VERTEX;
				else if (line.find("fragment") != std::string::npos)
					type = ShaderType::FRAGMENT;
			}
			else if (ShaderType::NONE != type)
			{
				ss[(int)type] << line << "\n";
			}
		}
		return { ss[0].str(), ss[1].str()};
	}
	```
## 修改源代码
将以字符串形式保存的shader代码替换为文件解析的方式
```cpp
...
shaderProgramSource source = ParseShader("res/shader/basic.shader");
unsigned int shader = CreateShader(source.vertexSource, source.fragmentSource);
glUseProgram(shader);
...
```
至此，已经将主程序中字符串形式的shader代码通过文件解析的方式进行替换，我们仍可以得到一个红色三角形。

::github{repo="ljiaooo/LearnOpenGL"}

[Commit Link](https://github.com/Ljiaooo/LearnOpenGL/tree/5571268601f5a198db99d282086a8e9358c74296)