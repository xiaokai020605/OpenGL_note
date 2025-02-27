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
### 顶点着色器
- 使用shader语言编写顶点着色器->编译这个着色器
- 非常基础的GLSL顶点着色器源代码：
```cpp
#version 330 core //版本声明和使用核心模式
layout (location = 0) in vec3 aPos; //声明输入顶点属性，位置数据，变量名aPos
void main() 
{ 
	gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0); //将位置数据赋值给预定义的gl_position变量是vec4类型的
}
```
- 使用`in`关键字声明所有的输入顶点属性
- GLSL有一个向量数据类型，包含1到4个`float`分量，可以从后缀数字看出例：`vec3`包含3个
	- 一个向量最多==4个分量==，每个分量值都代表空间中的一个坐标
	- 通过`vec.x、vec.y、vec.z、vec.w`其中vec.w不是表达空间中的位置，而是透明度
- `layout (location = 0)`设定了输入变量的位置值(Location)
- 在main函数最后，将gl_position设置的值成为该顶点着色器的输出
### 编译着色器
- 将顶点着色器的源代码硬编码在c字符串中
```cpp
const char* vertexShaderSource = "#version 330 core\n"
"layout (location = 0) in vec3 aPos;\n"
"void main()\n"
"{\n"
"   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
"}\0";
```
- 需在运行时动态编译源代码
- 创建一个着色器对象，还是使用ID引用，储存这个顶点着色器为`unsigned int`,然后用`glCreateShader`创建着色器
```cpp
unsigned int vertexShader;//声明顶点着色器对象
vertexShader = glCreateShader(GL_VERTEX_SHADER);//创建顶点着色器对象
```
- 把着色器源码附加到着色器对象上
```cpp
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);//将着色器源码附加到着色器上，第一个参数是着色器对象，第二个参数是着色器源码的数量，第三个参数是着色器源码的地址，第四个参数是着色器源码的长度
glCompileShader(vertexShader);//编译着色器
```
#### 检测编译是否错误和错误信息获取
```cpp
//检测编译是否错误
int  success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);//检查是否编译成功

//如果编译失败，打印错误信息
if (!success)
{
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```
### 片段着色器
- 片段着色器(Fragment Shader)是第二个也是最后一个我们打算创建的用于渲染三角形的着色器。片段着色器所做的是计算像素最后的颜色输出
- 输出变量使用`out`关键字声明
- 片段着色器源码：
```cpp
片段着色器创建和编译
const char* fragmentShaderSource = "#version 330 core\n"
    "out vec4 FragColor;\n"
    "void main()\n"
    "{\n"
    "   FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);\n"
    "}\0";
unsigned int fragmentShader;//声明片段着色器对象
fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);//创建片段着色器对象
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);//将着色器源码附加到着色器上，第一个参数是着色器对象，第二个参数是着色器源码的数量，第三个参数是着色器源码的地址，第四个参数是着色器源码的长度
glCompileShader(fragmentShader);//编译着色器
```
### 着色器程序
- 着色器程序对象(Shader Program Object)是多个着色器合并之后并==最终链接完成的版本==。如果要使用刚才编译的着色器我们必须把它们**链接(Link)** 为一个着色器程序对象，然后在渲染对象的时候激活这个着色器程序
- 当链接着色器至一个程序的时候，它会把每个着色器的输出链接到下个着色器的输入。当输出和输入不匹配的时候，会得到一个连接错误。
- 将顶点数据发送给gpu，并知识gpu如何在顶点和片段着色器中处理
- 创建着色器程序对象;
```cpp
 //着色器程序对象
 unsigned int shaderProgram;
 shaderProgram = glCreateProgram();//创建程序返回新创建程序对象的ID引用

 //链接对象
 glAttachShader(shaderProgram, vertexShader);//添加顶点着色器
 glAttachShader(shaderProgram, fragmentShader);//添加片段着色器
 glLinkProgram(shaderProgram);//链接对象
```
- 激活着色器程序对象：`glUseProgram(shaderProgram);`
- 链接之后需删除顶点着色器和片段着色器：
```cpp
//删除着色器
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```
### 链接顶点属性
- 在顶点着色器中必须手动指定输入数据的哪一个部分对应顶点着色器的哪一个顶点属性，==在渲染前指定如何解释顶点数据==
- 顶点缓冲数据解析：
	- ![[顶点数据缓冲解析图.png]]
	- 位置数据被储存为32位（4字节）浮点值。
	- 每个位置包含3个这样的值。
	- 在这3个值之间没有空隙（或其他值）。这几个值在数组中==紧密排列(Tightly Packed)。==
	- 数据中第一个值在缓冲开始的位置。
- 使用`glVertexAttribPointer`函数解析顶点数据(应用到逐个顶点属性上)
	```cpp
	glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
	glEnableVertexAttribArray(0);//启用顶点属性，以顶点属性位置值作为参数,默认禁用
	```
	- **第一个参数**：指定要配置的顶点属性，就是顶点着色器源码中的`layout(location=0)`里的location值
	- **第二个参数**：指定顶点属性的大小，顶点属性是一个`vec3`，它由3个值组成，所以大小是3
	- **第三个参数**：指定数据的类型，这里是GL_FLOAT(GLSL中`vec*`都是由浮点数值组成的)。
	- **第四个参数**：定义是否希望数据被标准化，有两个参数`GL_TRUE`和`GL_FALSE`，如果是`GL_TRUE`,数据会被映射到0(对于有符号型signed的数据是-1)到1之间，
	- **第五个参数**：是步长，设置在连续的顶点属性组之间的间隔，因为下一组是在3个`float`之后，也可以设置为0来让OpenGL决定具体步长是多少==(只有当数值是紧密排列时才可用)==
	- **第六个参数**：类型时`void*`，强制类型转换，表示位置数据在缓冲中起始位置的偏移量，因为位置数据在数组的开头所以设置为0
### 顶点数组对象
- 顶点数组对象(Vertex Array Object, VAO)可以像顶点缓冲对象那样被绑定，任何随后的顶点属性调用都会储存在这个VAO中。
- **优点：**
	- 配置多个顶点属性指针时，只需要将那些调用函数执行一次即可，之后再绘制物体只需要绑定对应的VAO即可
- 一个顶点数组对象储存以下内容
	- glEnableVertexAttribArray和glDisableVertexAttribArray的调用。
	- 通过glVertexAttribPointer设置的顶点属性配置。
	- 通过glVertexAttribPointer调用与顶点属性关联的顶点缓冲对象。
- 创建VAO源码：
	```cpp
	 //顶点数组对象
	unsigned int VAO;
	glGenVertexArrays(1, &VAO);
	```
- 使用VAO需要使用`glBindVertexArray`绑定VAO，从绑定之后起，我们应该绑定和配置对应的VBO和属性指针，之后解绑VAO供之后使用。当我们打算绘制一个物体的时候，我们只要在绘制物体前简单地把VAO绑定到希望使用的设定上就行了
- **当打算绘制多个物体时，首先要生成/配置所有的VAO==(和必须的VBO属性指针)==如何储存它们，当打算绘制物体的时候就拿出相应的VAO，绑定它，绘制完之后再解绑**
### 绘制函数
- `glDrawArrays`函数：它使用当前激活的着色器，之前定义的顶点属性配置，和VBO的顶点数据（通过VAO间接绑定）来绘制图元。
	- 例：`glDrawArrays(GL_TRIANGLES, 0, 3);`
	- **第一个参数**：传递要绘制的图元类型,这里传递`GL_TRIANGLES`三角形
	- **第二个参数**：指定顶点数组的起始索引
	- **第三个参数**：指定打算绘制多少个顶点
### 元素缓冲对象
- 元素缓冲对象(Element Buffer Object，EBO)，也叫索引缓冲对象(Index Buffer Object，IBO)
- 由于OpenGL只要处理三角形，当绘制矩形的时候需要六个顶点数据，但是其中会有两个重复的，**元素缓冲对象**是用来存储不同的顶点并且设定绘制这些顶点的顺序--**索引绘制**
- EBO是一个缓冲区，就像一个顶点缓冲区对象一样，它存储 OpenGL 用来决定要绘制哪些顶点的索引。
	- 先要定义顶点不重复的，和绘制矩形所需的索引
	- 但是也是先绘制第一个三角形的顺序然后是第二个三角形的
```cpp
float vertices[] = {
0.5f, 0.5f, 0.0f,   // 右上角
0.5f, -0.5f, 0.0f,  // 右下角
-0.5f, -0.5f, 0.0f, // 左下角
-0.5f, 0.5f, 0.0f   // 左上角
};

unsigned int indices[] = {
	// 注意索引从0开始! 
	// 此例的索引(0,1,2,3)就是顶点数组vertices的下标，
	// 这样可以由下标代表顶点组合成矩形

	0, 1, 3, // 第一个三角形
	1, 2, 3  // 第二个三角形
};
unsigned int EBO;
glGenBuffers(1, & EBO);

glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);//绑定缓冲
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
```
- 需要使用**glDrawElements**来替换**glDrawArrays**函数，表示我们要从索引缓冲区渲染三角形。使用glDrawElements时，我们会使用当前绑定的索引缓冲对象中的索引进行绘制
	- `glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);`
	- **第一个参数**：指定绘制的模式，和glDrawArrays的一样
	- **第二个参数**：是打算绘制顶点的个数
	- **第三个参数**：是索引的类型，是GL_UNSIGNED_INT。
	- **第四个参数**：指定EBO中的偏移量或者传递一个索引数组，但是这是当你不在使用索引缓冲对象的时候），但是我们会在这里填写0。
### 线框模式
- 使用`glPolygonMode(GL_FRONT_AND_BACK, GL_LINE)`函数
- **第一个参数**：表示我们打算将其应用到所有的三角形的正面和背面
- **第二个参数**：告诉使用线来绘制，之后的绘制调用会一直以线框模式绘制三角形直到用`glPolygonMode(GL_FRONT_AND_BACK, GL_FILL)`设置会默认模式
### 练习
- 添加更多顶点到数据中，使用glDrawArrays，尝试绘制两个彼此相连的三角形
	- ==只需将所需的顶点数据放置在顶点位置数组当中即可==
- 创建相同的两个三角形，但对它们的数据使用不同的VAO和VBO
	- 渲染器设置相同，但是要把顶点数据分成两个数组
	- VAO和VBO可以设置成为**数组存储**也可**单体存储**，但是有多少个VBO和VAO就要设置多少次**原始数据**并且也要绑定和设置指针等所有内容，包括绘画函数要**n次**
- 创建两个着色器程序，第二个程序使用一个不同的片段着色器，输出黄色；再次绘制这两个三角形，让其中一个输出为黄色
	- 将需要改变颜色的片段着色器重新编译新的源代码
	- 创建第二个着色器程序绑定修改颜色的片段着色器
### 总结
1. 顶点数据数组
2. 顶点着色器和片段着色器编程加编译
3. 声明VAO和VBO储存顶点缓冲数据
4. 顶点数据指针
5. 绑定缓冲顶点类型
6. 渲染循环绘画
## 着色器(Shader)
- 运行在管线各个阶段的程序，每个阶段都有自己默认的Shader
- 有些阶段的着色器可以由自己编写OpenGL着色器用OpenGL着色器语言（GLSL）
- ![[渲染管线的每个阶段的抽象展示.png]]
### 顶点数据
- 以数组的形式传递3个3d坐标作为图形渲染管线的输入--用来表示一个三角形
- 是一系列数据的集合
- 一个顶点的数据用顶点属性来表示，可以包含任何我们想用的数据和一些颜色值组成
#### 图元(Primitive)
- 告诉OpenGL我们的坐标和颜色值构成的是什么（点、三角形、长长的线）
- 绘制的指令的调用将图元告诉OpenGL
- 常见的指令有：`GL_POINTS、GL_TRIANGLES、GL_LINE_STRIP`
#### 片段
- OpenGL中的一个片段是OpenGL渲染一个像素所需的所有数据。
### GLSL语言
- 着色器开头总是要声明版本
- 然后是输入输出变量、uniform和main函数
	- 当是顶点着色器的时候，每个输入变量也叫**顶点属性**，确保至少有16个4分量的顶点属性可用
- 每个着色器的入口都是main函数，处理所有的输入变量，并将结果输出到输出变量中
```GLSL
#version version_number

in type in_variable_name;

in type in_variable_name;

out type out_variable_name;

uniform type uniform_name;

void main()

{

  // 处理输入并进行一些图形操作

  ...

  // 输出处理过的结果到输出变量

  out_variable_name = weird_stuff_we_processed;

}
```
#### 数据类型
- 包含c等默认基础数据类型：`int、float、double、uint、bool`
- GLSL自己有两种容器类型：`Vector、Matrix`分别是向量和矩阵
##### 向量
- 是一个可以包含有2、3、4个分量的容器，分量的类型可以是任意基础类型的其中之一

|类型|含义|
|---|---|
|`vecn`|包含`n`个float分量的默认向量|
|`bvecn`|包含`n`个bool分量的向量|
|`ivecn`|包含`n`个int分量的向量|
|`uvecn`|包含`n`个unsigned int分量的向量|
|`dvecn`|包含`n`个double分量的向量|
- 以上n代表分量的数量
- 一个向量的分量可以通过`vec.x`获取，x指该向量的第一个分量，允许对颜色使用`rgba`或是对纹理坐标使用`stpq`访问相同的分量
- 向量可以进行**重组**
	- 可以使用xyzw字母任意组合来创建和所需向量长的同类型新向量
	- 也可以把向量作为一个参数传给不同的向量构造函数
```GLSL
vec2 someVec; 
vec4 differentVec = someVec.xyxx; 
vec3 anotherVec = differentVec.zyw; 
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;

vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0); 
vec4 otherResult = vec4(result.xyz, 1.0);
```
### 输入输出
- 使用`in,out`关键字来实现输入输出目的
- 上一个着色器的**输出变量**与下一个着色器的**输入变量**匹配就会传递下去，==在顶点和片段着色器中不同==
- ==顶点着色器==直接从顶点数据中接收输入
	- 定义顶点数据如何管理----使用`location`**元数据**指定输入变量，才能在cpu上配置顶点属性
	- 例如`layout(location=0)`
- ==片段着色器==需要一个vec4颜色输出变量，因为片段着色器需要生成一个最终输出的颜色，假如没有定义，会把物体渲染为黑色或者白色
- ==要学会看编译结果如果有错误的话==
### Uniform
- 是一种从我们的应用程序在cpu上传递数据道GPU上的着色器方式，==是一个变量==
- uniform是**全局**的，意味着uniform变量必须在每个着色器程序对象中都是独一无二的，而且可以被任意着色器在任意阶段访问
- 无论把值设置成什么，uniform会一直保存数据，直到被重置或更新
- 要在GLSL中声明uniform，只需在着色器中使用`uniform`关键字，并且带上类型和名称
- 定义uniform的值
	- 首先要找到着色器uniform属性的索引/位置值
	- 然后就可以更新它的值了
```cpp
float timeValue = glfwGetTime();//获取运行的秒数
float greenValue = (sin(timeValue) / 2.0f) + 0.5f;//使用sin函数让颜色在0.0-1.0之间改变，然后将值储存到greenValue
int vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");//使用glGetUniformLocation查询uniform ourColor的位置值，参数为着色器程序和uniform的名字，如果返回-1就代表没有找到
glUseProgram(shaderProgram);
glUniform4f(vertexColorLocation, 0.0f, greenValue, 0.0f, 1.0f);//设置uniform的值
```
- **注意：** 查询uniform地址不要求你之前使用过着色器程序，但是更新一个uniform之前你**必须**先使用程序（调用glUseProgram)，因为它是在当前激活的着色器程序中设置uniform的。
- glUniform有多种类型看，主要看后缀

|后缀|含义|
|---|---|
|`f`|函数需要一个float作为它的值|
|`i`|函数需要一个int作为它的值|
|`ui`|函数需要一个unsigned int作为它的值|
|`3f`|函数需要3个float作为它的值|
|`fv`|函数需要一个float向量/数组作为它的值|
### 其他
- 可以在顶点数据中加入颜色数据
```cpp
 float vertices[] = {
     // positions         // colors
      0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,  // bottom right
     -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,  // bottom left
      0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f   // top 

 };
```
- 加入之后需要调整顶点着色器
```cpp
#version 330 core 
layout (location = 0) in vec3 aPos; // 位置变量的属性位置值为 0 
layout (location = 1) in vec3 aColor; // 颜色变量的属性位置值为 1 
out vec3 ourColor; // 向片段着色器输出一个颜色 
void main() { 
gl_Position = vec4(aPos, 1.0);
ourColor = aColor; // 将ourColor设置为我们从顶点数据那里得到的输入颜色 
}
```
- 片段着色器
```cpp
#version 330 core 
out vec4 FragColor; 
in vec3 ourColor; 
void main() { 
FragColor = vec4(ourColor, 1.0); 
}
```
- 修改重新配置顶点属性指针
```cpp
// 位置属性 
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)0); glEnableVertexAttribArray(0); // 颜色属性 
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6 * sizeof(float), (void*)(3* sizeof(float))); 
glEnableVertexAttribArray(1);
```
- **片段插值**：当渲染一个三角形时，光栅化(Rasterization)阶段通常会造成比原指定顶点更多的片段。光栅会根据每个片段在三角形形状上所处相对位置决定这些片段的位置。
### 着色器类
- 从文件流读取文件
- Shader头文件
![[Shader.h]]
### 练习
- 修改顶点着色器让三角形上下颠倒
	- 修改顶点着色器当中的y值为负即可就上下颠倒了
- 使用uniform定义一个水平偏移量，在顶点着色器中使用这个偏移量把三角形移动到屏幕右侧：
	- 定义uniform变量，将原先的位置数据加上该变量的数据即可，可以自定义也可以使用预先定义好的方法
- 使用`out`关键字把顶点位置输出到片段着色器，并将片段的颜色设置为与顶点位置相等（来看看连顶点位置值都在三角形中被插值的结果）。做完这些后，尝试回答下面的问题：为什么在三角形的左下角是黑的?
	- 要将位置值替换原先的颜色值，并且要在main函数内替换，不要只在out关键字之后更改
	- 因为左下角的数值是负数，直接会变为0所以是黑色，直到三角形中间位置会被开始正插值
## 纹理
### 纹理坐标
- 将纹理映射在物体上的每个顶点所关联的坐标
- 如何再图形的其他位置进行片段插值
- **2d纹理**：纹理坐标范围在0-1之间，起始点为左下角(0,0)终止点为右上角(1,1)，
	- 也就是原本图片的大小的左下角和右上角范围
- **采样**：使用纹理坐标获取纹理颜色
![[纹理.png]]
- 纹理坐标数据代码：
```cpp
float texCoords[] = 
{ 0.0f, 0.0f, // 左下角 
1.0f, 0.0f, // 右下角 
0.5f, 1.0f // 上中
};
```
### 纹理环绕方式
- 主要用来处理超过范围之外的(0-1)

| 环绕方式               | 描述                                           |
| ------------------ | -------------------------------------------- |
| GL_REPEAT          | 对纹理的默认行为。重复纹理图像。                             |
| GL_MIRRORED_REPEAT | 和GL_REPEAT一样，但每次重复图片是镜像放置的。                  |
| GL_CLAMP_TO_EDGE   | 纹理坐标会被约束在0到1之间，超出的部分会重复纹理坐标的边缘，产生一种边缘被拉伸的效果。 |
| GL_CLAMP_TO_BORDER | 超出的坐标为用户指定的边缘颜色。                             |
![[纹理环绕方式.png]]
- 所有选项都可以使用`glTexParameter*`函数进行单独的坐标设置（`s`、`t`（如果是使用3D纹理那么还有一个`r`）和`x`、`y`、`z`是等价的）：
	-  **第一个参数**：指定纹理目标，使用2d纹理所以是`GL_TEXTURE_2D`
	- **第二个参数**：需要我们指定设置的选项与应用的纹理轴。我们打算配置的是`WRAP`选项，并且指定`S`和`T`轴
	- **第三个参数**：传递一个环绕方式，当前设定的是`GL_MIRRORED_REPEAT`镜像
	- **注意**：如果我们选择GL_CLAMP_TO_BORDER选项，我们还需要指定一个边缘的颜色。这需要使用glTexParameter函数的`fv`后缀形式，用GL_TEXTURE_BORDER_COLOR作为它的选项，并且传递一个float数组作为边缘的颜色值
```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_MIRRORED_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_MIRRORED_REPEAT);

//另一个选项
float borderColor[] = { 1.0f, 1.0f, 0.0f, 1.0f }; 
glTexParameterfv(GL_TEXTURE_2D, GL_TEXTURE_BORDER_COLOR, borderColor);
```
### 纹理过滤
- **概念**：是决定如何从纹理图像中采样颜色值的关键技术。当纹理被映射到三维模型时，由于视角变换或缩放，纹理像素（texel）与屏幕像素（pixel）的对应关系可能不一致，此时需要通过纹理过滤算法来平滑或精确地**计算颜色值。**
- 目前两种纹理过滤`GL_NEAREST和GL_LINEAR`
- `GL_NEAREST`邻近过滤，是OpenGL默认的纹理过滤方式
	- 会选择中心点最接近纹理坐标的那个像素，
	- **优点**：计算快、适合像素风或需要锐利边缘的效果
	- **缺点**：放大时可能出现锯齿
	![[邻近过滤.png]]
- `GL_LINEAR`线性过滤，会基于纹理坐标附近的纹理像素，计算出一个插值，近似出纹理像素之间的颜色，中心距离纹理坐标越近，那么纹理像素的颜色对最终的样本颜色**贡献越大**
	- 对周围4个texel进行加权平均，生成平滑过渡的颜色
	- **优点**：放大/缩小时更平滑，适合大多数场景。可以清晰看到组成纹理的像素
	- **缺点**：略微模糊，可能丢失细节。
	![[线性过滤.png]]
- **Mipmap过滤**
    - Mipmap是预先生成的多分辨率纹理链，用于优化缩小（缩小观察距离）时的表现。
    - **过滤模式**：
        - `GL_NEAREST_MIPMAP_NEAREST`：选择最匹配的Mipmap层级，并取最近邻texel。
        - `GL_LINEAR_MIPMAP_NEAREST`：选择最匹配的Mipmap层级，进行线性插值。
        - `GL_NEAREST_MIPMAP_LINEAR`：在两个最接近的Mipmap层级上取最近邻，再混合结果。
        - `GL_LINEAR_MIPMAP_LINEAR`（三线性过滤）：在两个Mipmap层级上分别线性插值，再混合结果，最平滑但计算量最大。
- 进行**放大**和**缩小**操作可以设置纹理过滤的选项,比如在纹理缩小的适合使用**邻近过滤**，放大的时候使用**线性过滤**
```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST); glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```
### 多级渐远纹理
- 用来解决一个很大房间里有很多种物体上都有纹理，但是有些很远有些很近但是分辨率相同，并且获取正确的颜色值比较困难，会产生**不真实感**
- 多级渐远纹理：一系列的纹理图像，后一个图像时前一个的二分之一，距观察者的距离超过一定的阈值，OpenGL会使用不同的多级渐远纹理，即最适合物体的距离的那个，**性能较好**
- 创建多级渐远纹理使用`glGenerateMipmap`函数
- 由于不同级别的多级渐远纹理会产生**不真实**的生硬边界，可以使用纹理过滤来解决

|过滤方式|描述|
|---|---|
|GL_NEAREST_MIPMAP_NEAREST|使用最邻近的多级渐远纹理来匹配像素大小，并使用邻近插值进行纹理采样|
|GL_LINEAR_MIPMAP_NEAREST|使用最邻近的多级渐远纹理级别，并使用线性插值进行采样|
|GL_NEAREST_MIPMAP_LINEAR|在两个最匹配像素大小的多级渐远纹理之间进行线性插值，使用邻近插值进行采样|
|GL_LINEAR_MIPMAP_LINEAR|在两个邻近的多级渐远纹理之间使用线性插值，并使用线性插值进行采样|
```cpp
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR); glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```
- **常见错误**：把放大过滤的选项设置为多级渐远过滤选项之一会没有任何效果，==多级渐远纹理主要使用在纹理被缩小的情况下==如果为放大设置会产生一个`GL_INVALID_ENUM`错误代码
### 加载和创建纹理
- 加载库使用`stb_image.h`[这里下载](https://github.com/nothings/stb/blob/master/stb_image.h)
- 使用`stbi_load`函数加载图片
```cpp
int width, height, nrChannels;
unsigned char *data = stbi_load("container.jpg", &width, &height, &nrChannels, 0);
```
- 这个函数首先接受一个图像文件的位置作为输入。接下来它需要三个`int`作为它的第二、第三和第四个参数，`stb_image.h`将会用图像的**宽度**、**高度**和**颜色通道的个数**填充这三个变量
### 生成纹理
- 纹理也需要使用ID引用
```cpp
unsigned int texture;
glGenTextures(1, &texture);
```
- `glGenTextures`函数需要输入生成纹理的数量，然后把它们存储在第二个参数`unsigned int`数组中
- 绑定纹理：`glBindTexture(GL_TEXTURE_2D, texture);`
- 生成纹理使用`glTexImage2D`2D选**后缀为2d**
	-  **第一个参数**：指定纹理目标，选择为2d纹理会生成与当前绑定的纹理对象在同一个目标上的纹理
	- **第二个参数**:指定纹理多级渐远纹理的级别，0为基本级别
	- **第三个参数**：告诉OpenGL希望把纹理存储为何种格式，因为选择的图像只有==RGB==值所以是RGB
	- **第四个和第五个参数**：设置最终的宽度和高度，使用对应的变量即可
	- **第六个参数**：设为0即可
	- **第七个第八个参数**：定义了原图的格式和数据类型，使用RGB值加载这个图像，并存储为==char数组==
	- **第九个参数**：真正的图像数据
	- 如果要使用多级渐远纹理，可以选择手动设置(第二个参数)或者在生成纹理之后调用`glGenerateMipmap`会自动生成所有需要的多级渐远纹理
```cpp
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data); 
glGenerateMipmap(GL_TEXTURE_2D);
```
- 生成了对应和相应的纹理之后需要释放图像的内存：`stbi_image_free(data);`
### 应用纹理
- 使用新的顶点数据
```cpp
//纹理数据
float vertices[] = {
    //     ---- 位置 ----       ---- 颜色 ----     - 纹理坐标 -
         0.5f,  0.5f, 0.0f,   1.0f, 0.0f, 0.0f,   1.0f, 1.0f,   // 右上
         0.5f, -0.5f, 0.0f,   0.0f, 1.0f, 0.0f,   1.0f, 0.0f,   // 右下
        -0.5f, -0.5f, 0.0f,   0.0f, 0.0f, 1.0f,   0.0f, 0.0f,   // 左下
        -0.5f,  0.5f, 0.0f,   1.0f, 1.0f, 0.0f,   0.0f, 1.0f    // 左上
};
```
- 由于添加了额外的属性，所以需要重新设置顶点属性指针
	- 步长为`8 * sizeof(float)`
	- ![[包含纹理坐标的内存图.png]]
```cpp
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void*)(6 * sizeof(float))); 
glEnableVertexAttribArray(2);
```
- 再调整顶点着色器使其能够接收顶点坐标为一个顶点属性，并把坐标传给片段着色器
- 在顶点着色器当中添加一个vec2的位置点，再进行输出该变量
- 在片段着色器当中将顶点着色器的输出变量作为输入变量
- 将纹理对象传给片段着色器要有一个供纹理对象使用的内建数据类型--==采样器==
	- 采样器以纹理类型作为后缀，例如：`sampler1D`、`sampler3D`
	- 声明一个uniform sampler2D，将纹理赋值给该uniform
	- 使用`texture`函数来采样纹理颜色
		- 第一个参数是纹理采样器
		- 第二参数是对应的纹理坐标
	- 假如想让纹理颜色和顶点颜色进行混合，只需将纹理颜色和顶点颜色在片段着色器当中相乘来混合：`FragColor = texture(ourTexture, TexCoord) * vec4(ourColor, 1.0);`
### 纹理单元
- 一个纹理的位置值称为**纹理单元**，一个纹理的默认纹理单元式0
- 使用`glUniformli`可以给纹理采样器分配一个位置值，==这样可以在一个片段着色器中设置多个纹理==
- 需要激活对应的纹理单元，使用`glActiveTexture`，==在绑定之前激活==
- 在OpenGL当中至少保证有16个纹理单元可以使用，按顺序定义的也可也通过`GL_TEXTURE0+8`来得到`GL_TEXTURE8`在循环当中可以使用
```cpp
#version 330 core ... 
uniform sampler2D texture1; 
uniform sampler2D texture2; 
void main() { 
FragColor = mix(texture(texture1, TexCoord), texture(texture2, TexCoord), 0.2); 
}
```
- mix函数需要接受两个值作为参数，并对它们根据第三个参数进行线性插值。如果第三个值是`0.0`，它会返回第一个输入；如果是`1.0`，会返回第二个输入值。`0.2`会返回`80%`的第一个输入颜色和`20%`的第二个输入颜色，即返回两个纹理的混合色。
### 练习
- 修改片段着色器，**仅**让笑脸图案朝另一个方向看
	- 可以将texture里的第二个参数纹理坐标改为vec2向量，将x的分量改为**负**的
- 尝试用不同的纹理环绕方式，设定一个从`0.0f`到`2.0f`范围内的（而不是原来的`0.0f`到`1.0f`）纹理坐标。试试看能不能在箱子的角落放置4个笑脸：[参考解答](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/4.4.textures_exercise2/textures_exercise2.cpp)，[结果](https://learnopengl-cn.github.io/img/01/06/textures_exercise2.png)。记得一定要试试其它的环绕方式
	- 将纹理坐标改为0-2，将设置的环绕方式改为限制在0-1之间，但是第二张纹理不进行改变
- - 尝试在矩形上只显示纹理图像的中间一部分，修改纹理坐标，达到能看见单个的像素的效果。尝试使用GL_NEAREST的纹理过滤方式让像素显示得更清晰：[参考解答](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/4.5.textures_exercise3/textures_exercise3.cpp)
	- 将坐标改为只有一个像素大小，将过滤方式改为邻近过滤
- - 使用一个uniform变量作为mix函数的第三个参数来改变两个纹理可见度，使用上和下键来改变箱子或笑脸的可见度：[参考解答](https://learnopengl.com/code_viewer_gh.php?code=src/1.getting_started/4.6.textures_exercise4/textures_exercise4.cpp)。
	- 在片段着色器代码中添加一个uniform变量放置在mix函数的混合参数里，
	- 再在按键函数当中实现按键实现
## 变换/数学
### 向量
- **最基本的定义**：有一个方向和大小(==强度或长度)==通常只使用2到4维
	- 例如：向左走10步，向北走3步，然后向右走5步”；“左”就是方向，“10步”就是向量的长度
- 下图可以看到向量v¯和w¯是相等的，尽管起始点不同
	![[向量在图的表示.png]]
- 使用公式表达：
	![[向量公式表示.png]]
### 向量与标量运算
- **标量**：是一个数字/仅有一个分量的向量，当把一个向量加减乘除一个标量的时候，将向量的每个分量和标量进行**运算**
### 向量取反
- 对一个向量取反会将其方向逆转，**例如**：一个指向东北的向量取反后指向的是西南方向，==在每个分量前加一个负号就可以**实现取反**或者乘以-1== 
- $-\overline{v}=-\left(\begin{array}{c}v_x\\v_y\\v_z\end{array}\right)=\left(\begin{array}{c}-v_x\\-v_y\\-v_z\end{array}\right)$ 
### 向量加减
- 向量加法定义是分量相加，**将一个向量中的每一个分量加上另一个向量的对应分量**
- 向量`v = (4, 2)`和`k = (1, 2)`可以直观地表示为：
	- 前后顺序的改变，三角形面积不变，对称性，在图里相加为先一个向量之后的终点加上另一个向量，但是总值也不会改变
![[向量相加.png]]
- **向量的减法**：等于加上第二个向量的相反向量
	- 两个向量的相减会得到两个向量指向位置的差
	- 如果得到一个分量是**负数**，代表和**当前方向相反**，但是最后还是得到一个三角形的样子
![[向量相减.png]]
### 长度
- 使用**勾股定理**来获取向量的长度，获取斜边的长度
![[计算向量的长度.png]]
- 单位向量：是一个特殊的向量，长度为1
	- 可以使用任意向量的每个分量除以向量的长度得到它的单位向量n^
	- $\hat{n} = \frac{\overline{v}}{\|\overline{v}\|}$ 
- **向量的标准化**：单位向量头上有一个^样子的记号。通常单位向量会变得很有用，特别是在我们只关心方向不关心长度的时候（如果改变向量的长度，它的方向并不会改变）
### 向量相乘
#### 点乘
- 两个向量的点乘等于它们的数乘结果乘以两个向量之间夹角的余弦值
- 公式：$\bar{v} \cdot \bar{k} = \|\bar{v}\| \cdot \|\bar{k}\| \cdot \cos\theta$ 
	- 如果${\overline{v}}$ 和${\overline{k}}$ 都是单位向量，长度就将是1，**上式**=${\cos\theta}$ 
- 使用点乘可以测试两个向量是否**正交**或者**平行**，==正交意味着互为直角==
- 要计算两个单位向量间的夹角，我们可以使用反余弦函数cos−1cos−1 ，可得结果是143.1度点乘在**计算光照的时候有用**
- 点乘计算方法
	![[点乘表示.png]]
#### 叉乘
- 叉乘只在3D空间中有定义，它需要两个不平行向量作为输入，生成一个正交于两个输入向量的第三个向量。如果输入的两个向量也是正交的，那么叉乘之后将会产生3个互相正交的向量。
- 叉乘在3d空间中的样子：
	![[叉乘在空间的表示.png]]
- 叉乘的运算公式：$\left(\begin{array}{c} A_x \\ A_y \\ A_z \end{array}\right) \times \left(\begin{array}{c} B_x \\ B_y \\ B_z \end{array}\right) = \left(\begin{array}{c} A_y \cdot B_z - A_z \cdot B_y \\ A_z \cdot B_x - A_x \cdot B_z \\ A_x \cdot B_y - A_y \cdot B_x \end{array}\right)$
### 矩阵







