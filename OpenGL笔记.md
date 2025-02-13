# 入门
## 窗口创建
### 初始化
1. 先使用`glfwInit();`函数初始化==GLFW==
2. 再使用`glfwWindowHint(枚举值,GL版本)`函数配置GLFW
	1. 第一个参数代表选项的名称，从GLFW_开头的各个枚举值中选择
	2. 第二个参数使用一个整型，代表gl的版本
3. 函数选项和值[GLFW： 窗口指南](https://www.glfw.org/docs/latest/window.html#window_hints)
- 例子
	```cpp
	int main() { 
	// 初始化GLFW，配置OpenGL 3.3核心模式
		glfwInit(); 
		glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);  // 主版本号
		glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3); // 次版本号
		glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE); // 核心模式（无兼容特性）
		//glfwWindowHint(GLFW_OPENGL_FORWARD_COMPAT, GL_TRUE); return 0; 
	}
	```
### 创建窗口对象
- 例子：
```cpp
GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL); //生成一个glfw窗口对象
if (window == NULL) 
{ 
std::cout << "Failed to create GLFW window" << std::endl; glfwTerminate(); 
return -1; 
} 
glfwMakeContextCurrent(window);// 将窗口绑定到当前线程
```
1. `glfwCreateWindow(宽度，高度，窗口名，null，null)`返回一个`GLFWwindow`对象
2. `GLFWwindow`对象在其他的GLFW操作会使用到
3. 通知GLFW将窗口设置为当前线程的主上下文
### GLAD
- GLAD用来管理OpenGL函数指针，在调用OpenGL函数之前需要初始化GLAD
```cpp
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) 
{ 
	std::cout << "Failed to initialize GLAD" << std::endl; 
	return -1; 
}
```
- 给GLAD传入了用来加载系统相关的OpenGL函数指针地址的函数。GLFW给我们的是`glfwGetProcAddress`，==它根据我们编译的系统定义了正确的函数。==
### 视口/渲染前需要做的
- 告诉OpenGL渲染窗口的尺寸大小，之后OpenGL才能知道怎样根据窗口大小显示数据和坐标
- 调用`glViewport(0，0，800，600)`设置视口尺寸
- 前两个参数控制窗口左下角的位置，后两个参数控制渲染窗口的宽度和高度
- 也可以将视口的大小设置的比GLFW窗口小，这样渲染的效果会在更小的窗口内显示，但是可以将一些其他元素显示在OpenGL视口之外
#### 窗口自适应
- 封装一个回调函数(CallbackFunction)，==会在每次窗口大小被调整的时候调用==
- 回调函数原型----帧缓冲大小函数
```cpp
void framebuffer_size_callback(GLFWwindow* window, int width, int height) 
{ 
	glViewport(0, 0, width, height); 
}
```
- 使用`glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);`==窗口调整大小调用==
- 还可以将函数注册到其他回调函数中
	- 例子：**以创建一个回调函数来处理手柄输入变化，处理错误消息等。我们会在创建窗口之后，渲染循环初始化之前注册这些回调函数。**
### 渲染循环
- 不断绘制图像并且能够接收用户输入，使用while循环，让GLFW在退出之前一致保持运行，
```cpp
while(!glfwWindowShouldClose(window)) 
{ 
	glfwSwapBuffers(window); 
	glfwPollEvents(); 
}
```
- ==glfwWindowShouldClose==函数在我们每次循环的开始前检查一次GLFW是否被要求退出，如果是的话，该函数返回`true`，渲染循环将停止运行，之后我们就可以关闭应用程序。
- ==**glfwPollEvents**==函数检查有没有触发什么事件（比如键盘输入、鼠标移动等）、更新窗口状态，并调用对应的回调函数（可以通过回调方法手动设置）。
- ==glfwSwapBuffers==函数会交换颜色缓冲（它是一个储存着GLFW窗口每一个像素颜色值的大缓冲），它在这一迭代中被用来绘制，并且将会作为输出显示在屏幕上。
#### 双缓冲
- 避免渲染中途画面闪烁：后缓冲完成全部绘制后，再交换到前缓冲显示。
### 渲染结束
- 渲染循环结束后需要正确释放/删除之前的分配的所有资源
- 调用`glfwTerminate();`
### 输入
- 输入控制使用GLFW的输入函数`glfwGetKey`函数，需要一个窗口和一个按键作为输入，
	- 这个函数会返回该按键是否正在按下
	- 创建一个processInput函数保持整洁
```cpp
void processInput(GLFWwindow *window) 
{ 
	if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS) 
	glfwSetWindowShouldClose(window, true); 
}
```
- 检查用户是否按下了返回键(Esc)（如果没有按下，glfwGetKey将会返回GLFW_RELEASE。如果用户的确按下了返回键，我们将通过使用glfwSetwindowShouldClose把`WindowShouldClose`属性设置为 `true`来关闭GLFW。下一次while循环的条件检测将会失败，程序将关闭。
### 清屏
- 每次渲染迭代之后需要清屏否则能看见上一次迭代的渲染结果
- 使用`glClear`函数来清屏，接收一个缓冲位(Buffer Bit)来指定要清空的缓冲
	- 常见的缓冲位：`GL_COLOR_BUFFER_BIT，GL_DEPTH_BUFFER_BIT和GL_STENCIL_BUFFER_BIT`
- 可以调用`glClearColor`来设置清空屏幕所用的颜色
```cpp
glClearColor(0.2f, 0.3f, 0.3f, 1.0f); 
glClear(GL_COLOR_BUFFER_BIT);
```
- glClearColor函数是一个**状态设置**函数，而glClear函数则是一个**状态使用**的函数，它使用了当前的状态来获取应该清除为的颜色。
### 总结
1. **初始化 GLFW**
2. **配置窗口提示**
3. **创建窗口对象**
4. **设置当前上下文**
5. **初始化 GLAD**
6. **设置视口**
7. **注册窗口大小回调**
8. **进入渲染循环**
[[创建窗口总结步骤和简化笔记]]
## 画三角形
- 基础知识：
	- 顶点数组对象：Vertex Array Object，VAO
	- 顶点缓冲对象：Vertex Buffer Object，VBO
	- 元素缓冲对象：Element Buffer Object，EBO 或 索引缓冲对象 Index Buffer Object，IBO
### 图形渲染管线
- 概念主旨：一堆原始图形数据途径一个输送管道，期间经过各种变化处理最终出现在屏幕的过程
- 主要作用：
	- 将3d坐标转换处理为2d坐标
	- 把2d坐标转变为实际的有颜色的**像素**
		- 2D坐标和像素也是不同的，2D坐标精确表示一个点在2D空间中的位置，而2D像素是这个点的近似值，2D像素受到你的屏幕/窗口分辨率的限制。
- 管线可以被分为几个阶段，每个阶段将前一个阶段的**输出**作为**输入**并且每个阶段可以**并行执行**
- 每个阶段都有自己所需要处理的程序---**着色器(shader)**
### 顶点输入
- **标准化设备坐标**：所有3d坐标x,y,z都在-1到1的范围内，==我们所需做的就是把所有坐标控制在这个范围==超过这个范围的不会显示
- 渲染三角形需要指定三个顶点，定义一个`float`数组
```cpp
float vertices[] = { -0.5f, -0.5f, 0.0f, 0.5f, -0.5f, 0.0f, 0.0f, 0.5f, 0.0f };
```
- OpenGL是在3D空间中工作的，而我们渲染的是一个2D三角形，我们将它顶点的z坐标设置为0.0。这样子的话三角形每一点的*深度*都是一样的，从而使它看上去像是2D的。
-  ![[标准化设备坐标.png]]
- **视口变换**：标准化设备坐标变换为屏幕空间坐标。使用`glViewport`函数
- 作为输入发送给第一个处理阶段：顶点着色器
	- 它会在GPU上创建内存用于储存顶点数据
	- 还要配置OpenGL如何解释这些内存，并且指定其如何发送给显卡。
#### 顶点缓冲对象(Vertex Buffer Objects， VBO)
- 是一个OpenGL对象：==管理顶点内存==
- 可以一次性将大批数据发送到显卡上
- 可以使用`glGenBuffers`函数生成一个带有**缓冲ID**的VBO对象
```cpp
unsigned int VBO; 
glGenBuffers(1, &VBO);//第一个参数是id第二个参数是声明的需要绑定的变量
```
- 允许同时绑定多个缓冲，只需要是**不同的缓冲类型**，使用`glBindBuffer`
	- 例子：`glBindBuffer(GL_ARRAY_BUFFER, VBO);`
	- `GL_ARRAY_BUFFERGL_ARRAY_BUFFER`是另一种缓冲对象类型
- 调用`glBufferData`函数把之前定义的顶点数据复制到缓冲的内存中---GL_ARRAY_BUFFERVBO
	- 例子：`glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);`
	- `glBufferData`是专门用来把用户定义的数据**复制到**当前绑定缓冲的函数
	- **第一个参数**：目标缓冲的类型：顶点缓冲对象当前绑定到目标上
	- **第二个参数**：指定传输数据的大小(字节为单位)
	- **第三个参数**：希望发送的实际数据--==定义好的顶点数据数组==
	- **第四个参数**：希望显卡如何管理给定的数据，有三种形式
		- GL_STATIC_DRAW：数据不会或几乎不会改变。
		- GL_DYNAMIC_DRAW：数据会被改变很多。
		- GL_STREAM_DRAW：数据每次绘制时都会改变。
## 着色器(Shader)
- 运行在管线各个阶段的程序，每个阶段都有自己默认的Shader
- 有些阶段的着色器可以由自己编写OpenGL着色器用OpenGL着色器语言（GLSL）
- ![[渲染管线的每个阶段的抽象展示.png]]
#### 顶点数据
- 以数组的形式传递3个3d坐标作为图形渲染管线的输入--用来表示一个三角形
- 是一系列数据的集合
- 一个顶点的数据用顶点属性来表示，可以包含任何我们想用的数据和一些颜色值组成
##### 图元(Primitive)
- 告诉OpenGL我们的坐标和颜色值构成的是什么（点、三角形、长长的线）
- 绘制的指令的调用将图元告诉OpenGL
- 常见的指令有：`GL_POINTS、GL_TRIANGLES、GL_LINE_STRIP`
##### 片段
- OpenGL中的一个片段是OpenGL渲染一个像素所需的所有数据。



