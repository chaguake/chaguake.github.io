---
layout: post
author: chaguake
title: Quaternion Visualization Tool for IMU Sensor Based on OpenGL Fixed Rendering Pipeline
categories: 
- OpenGL
tags: 
- OpenGL
---

> 本文是对以往工作内容所使用到的一些技术框架的总结。

## 一、需求

公司内部研制的动捕设备使用的是IMU传感器，为了方便嵌入式同事调试IMU传感器，从而开发了这么一个简单的四元数可视化工具。

为了快速完成开发，并且减少使用同事环境配置工作，我选择使用OpenGL的固定渲染管线，工具库使用freeglut。

## 二、四元数

四元数的公式如下：

$$
q = w + xi + yj + zk
$$

其中，$i$分量表示YZ平面中正Z轴向正Y轴旋转（欧拉角中的Pitch俯仰分量），$j$分量表示XZ平面中正X轴向正Z轴旋转（欧拉角中的Yaw偏航分量），$k$分量表示XY平面中正Y轴向正X轴旋转（欧拉角中的Roll滚动分量）。

四元数可以用来表示物体在三维空间的旋转运动，表示绕着某个旋转轴$v$旋转一定角度$θ$。

$$
w = \cos(\frac{\theta}{2}) \\
x = v_x\sin(\frac{\theta}{2}) \\
y = v_y\sin(\frac{\theta}{2}) \\
z = v_z\sin(\frac{\theta}{2}) \\
$$

三、OpenGL和freeglut

OpenGL是用于渲染2D、3D矢量图形的跨语言、跨平台的应用程序编程接口。freeglut是一个开源的OpenGL工具库，用它来创建窗口、初始化 OpenGL 上下文和处理输入事件所需的所有系统特定的杂务。

使用OpenGL固定渲染管线的原因是为了方便，当然，用可编程渲染管线也不会浪费开发时间。

四、实现

```
glutInit(&argc, argv);
```

初始化freeglut库。

```
glutInitDisplayMode(GLUT_RGB | GLUT_DOUBLE | GLUT_DEPTH | GLUT_MULTISAMPLE);
```

初始化程序显示模式，GLUT_RGB设置RGB模式，当然使用GLUT_SRGB会全局使用Gamma校正，GLUT_DOUBLE表示使用双缓冲，GLUT_DEPTH表示使用深度测试，GLUT_MULTISAMPLE表示使用多重采样。

```
glutInitWindowSize(500, 500); 
glutCreateWindow("OpenGL Quaternion Visualization");
```

设置窗口大小，然后创建窗口。

```
glViewport(0, 0, width, height); 
glClearColor(0.3, 0.3, 0.3, 0.0);
```

设置视口大小，以及默认清理填充颜色。

```
glMatrixMode(GL_PROJECTION); 
glMatrixMode(GL_MODELVIEW);
```

由于固定渲染管线很多参数都是内置的，glMatrixMode函数则用于指定当前使用哪个矩阵。其他的跟可编程渲染管线一致，不多做介绍。

```
glutReshapeFunc(resize); 
glutDisplayFunc(display); 
glutIdleFunc([]() {glutPostRedisplay(); });
```

glutReshapeFunc用于注册窗口形状发生变化时的回调函数，glutDisplayFunc用于注册窗口绘制回调函数，glutIdleFunc函数用于注册窗口空闲时的回调函数，一般让它执行一个glutPostRedisplay函数以标记窗口需要重新绘制。

```
/* /* LightName */ 
#define GL_LIGHT0                         0x4000 
#define GL_LIGHT1                         0x4001 
#define GL_LIGHT2                         0x4002 
#define GL_LIGHT3                         0x4003 
#define GL_LIGHT4                         0x4004 
#define GL_LIGHT5                         0x4005 
#define GL_LIGHT6                         0x4006 
#define GL_LIGHT7                         0x4007 

/* LightParameter */ 
#define GL_AMBIENT                        0x1200 
#define GL_DIFFUSE                        0x1201 
#define GL_SPECULAR                       0x1202 
#define GL_POSITION                       0x1203 
#define GL_SPOT_DIRECTION                 0x1204 
#define GL_SPOT_EXPONENT                  0x1205 
#define GL_SPOT_CUTOFF                    0x1206 
#define GL_CONSTANT_ATTENUATION           0x1207 
#define GL_LINEAR_ATTENUATION             0x1208 
#define GL_QUADRATIC_ATTENUATION          0x1209 
*/ 

glEnable(GL_LIGHT0)； 
glEnable(GL_LIGHT1)； 
... 

glLightfv(GL_LIGHT0, GL_AMBIENT, light_ambient); 
glLightfv(GL_LIGHT0, GL_DIFFUSE, light_diffuse); 
glLightfv(GL_LIGHT0, GL_SPECULAR, light_specular); 
glLightfv(GL_LIGHT0, GL_POSITION, light_position1); 

glLightfv(GL_LIGHT1, GL_AMBIENT, light_ambient); 
glLightfv(GL_LIGHT1, GL_DIFFUSE, light_diffuse); 
glLightfv(GL_LIGHT1, GL_SPECULAR, light_specular); 
glLightfv(GL_LIGHT1, GL_POSITION, light_position2); 
...

```

OpenGL固定渲染管线只允许至多添加8个光源，可以为每个光源设置参数数值。

```
glMaterialfv(GL_FRONT, GL_AMBIENT, mat_ambient); 
glMaterialfv(GL_FRONT, GL_DIFFUSE, mat_diffuse); 
glMaterialfv(GL_FRONT, GL_SPECULAR, mat_specular); 
glMaterialfv(GL_FRONT, GL_SHININESS, high_shininess);
```

OpenGL固定渲染管线支持双面渲染功能，所以需要指定当前设置的材质参数是用于哪一面的。

```
glutMainLoop();
```

进入GLUT事件处理循环。

```
void draw_model(float axis_x, float axis_y, float axis_z, float angle) 
{   
	glPushMatrix();   
	glColor3f(0.0, 1.0, 1.0);   
	glRotated(angle, axis_x, axis_y, axis_z);   
	glutSolidCube(cube_size);   
	glPopMatrix(); 
}
```

OpenGL固定渲染管线提供一个对矩阵压栈出栈的操作，glPushMatrix函数和glPopMatrix函数配对。