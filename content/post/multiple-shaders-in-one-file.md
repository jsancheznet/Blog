---
title: "Multiple OpenGL shaders in one file"
date: 2019-12-09
draft: false
---

If you have been using OpenGL for a while, you probably have a lot of
shader files, maybe one for each programmable stage of the
pipeline. At some point during development the directory that contains
the shaders becomes a mess, managing and opening shader files
in your editor becomes tedious. Luckily there's a way to cram all the
programmable shader stages in a single file, reducing the number of
files at least by half and calm the neat freak inside.

These are the two things that make this trick possible.
1. [#ifdef - glsl's preprocessor directive](https://www.khronos.org/opengl/wiki/Core_Language_(GLSL)#Preprocessor_directives)
2. [glShaderSource()](http://docs.gl/gl3/glShaderSource)

The idea is to separate different shader stages inside an #ifdef blocks and setting the corresponding #define directive for the shader stage.

Here's an example of what a shader might look like using this method.
```glsl
#ifdef VERTEX_SHADER

layout(location = 0) in vec3 Vertices;
layout(location = 1) in vec3 Normals;

uniform mat4 Model;
uniform mat4 View;
uniform mat4 Projection;

void main()
{
    gl_Position = Projection * View * Model * vec4(Vertices, 1.0);
}

#endif

#ifdef FRAGMENT_SHADER

uniform vec3 Color;
out vec4 FragColor;

void main()
{
   FragColor = vec4(Color, 1.0);
}

#endif
```

Shader compilation looks something like this:
```C++
char *SourceFile = ReadTextFile("MyShader.glsl");

// Compile Vertex Shader
u32 VertexShaderObject = glCreateShader(GL_VERTEX_SHADER);
char *VertexSource[2] = {"#version 330 core\n#define VERTEX_SHADER\n", SourceFile};
glShaderSource(VertexShaderObject, 2, VertexSource, NULL);
glCompileShader(VertexShaderObject);

// Compile Fragment Shader
u32 FragmentShaderObject = glCreateShader(GL_FRAGMENT_SHADER);
char *FragmentSource[2] = {"#version 330 core\n#define FRAGMENT_SHADER\n", SourceFile};
glShaderSource(FragmentShaderObject, 2, FragmentSource, NULL);
glCompileShader(FragmentShaderObject);

// Compile other shader stages

// Link program
u32 FinalProgram = glCreateProgram();
glAttachShader(FinalProgram, VertexShaderObject);
glAttachShader(FinalProgram, FragmentShaderObject);
glLinkProgram(Result);
```
Notice that before calling glShaderSource we create an array of two
pointers to char, the first one contains glsl's #version preprocessor
directive and the token that defines each shader. In the case of the
vertex shader the first string looks like this "#version 330
core\n#define VERTEX_SHADER\n".  Sadly the #version directive needs to
be the first line in a shader or else compilation fails. Setting the
shader version like this is not as pretty or tidy as we may want but
it's a tradeoff i'm willing to make. When calling glShaderSource we
set the second parameter to "2" indicating 2 strings and set the
string to our newly created array.


I hope you find this useful.
