# ARM-MDK-Debug-collect
# ARM-MDK 调试汇总

> **version :**    *v1.0*      *「2022.7.25」*    *第一版赶在回家前写出来的，可能存在问题，或链接视频失效什么的*
>
> **author：**  *Y.Z.T.*
>
> 
>
> **摘要：**  *汇总在使用keil的过程中的一些调试方法*
>
> **简介：** 
>
> *:one:前几天看哈工程直播的时候，发现他们是用的**Ozone**进行开发，花了点时间去简单了解了一下；*
>
> *:two:发现了**Ozone**的调试功能是比较强的，具备了一些**keil**所没有的功能（如：DebugSnapshot（快照功能））,所以谁比较闲可以试一下用（**VSCode 编辑代码 + Keil 编译 + Ozone 调试**）来开发。*
>
> *:three:但更多的，我发现**Ozone** 有的功能**keil**也基本都有，虽然很多因为硬件限制而用不了；所以我打算汇总一下keil的调试技巧 ,毕竟keil本身的调试功能是非常强的，我们平时用的很多都是比较浅的*
>
> *:four: 本篇文章的内容来自笔者的调试经验及网络，**受限于硬件**笔者并未全部尝试过，仅**意在抛砖引玉**。借用蔡老板一句话：* <font face="华文行楷" color=black size=4> “不要老是说什么学这个没用，我用keil就行了，不要搞那种奇奇怪怪的开发方式，有些东西就是得尝试一下”</font>



<font face="华文彩云" color=red size=6> 特别注意：本文档可能并不完善，欢迎各位进行补充。</font>





:point_down:相关代码、资料、本文档源文件已放入**GitHub**。需要修改的可以，自行下载源文件。

[项目地址](https://github.com/ye2020/ARM-MDK-Debug-collect)



## :star:<font face="华文行楷" color=black size=7> 目录</font>



[TOC]

## :exclamation: 前言



**接下来的调试介绍大概分成以下几个部分：**

- ​    基础调试
- ​    具体调试
- :star:ITM程序跟踪
- ​     其他调试工具



### 测试环境

>**开发板：**             *ACE实验室F4工程板   //   正点原子精英板   // ACE实验室H7通用板*
>
>> **主控芯片：**	*STM32F407VET6  // STM32F103ZET6   // STM32H750VBT6*
>
>> **CPU：**			*Cortex-M4  // Cortex-M3   //  Cortex-M7*
>
>> **最高主频：**     *168MHZ   // 72MHZ  // 480MHZ*
>
>
>
>**测试程序：**     			  *[debug_testF407]*  / *[debug_testH750_v1.3.5]*
>
>>**FreeRTOS版本：** *v10.3.1*
>
>>**FreeRTOS说明：**  *FreeRTOS为STM32CubeMX配置的未修改版本。*
>
>
>
>**开发平台：**	    *VSCODE、MDK-ARM*
>
>>**程序编辑：**    *VSCODE*
>
>>**程序调试：**     *MDK-ARM     [ v 5.35.0.2]*
>
>
>
>**调试设备：**			*J-Link仿真器 ：    [ACE - Sentry-哨兵]*
>
>​								*CAN分析仪 ：    CANalyest - III*
>
>​								*DJI电池、C610电调、3508电机*
>
>





----



## :one: 基础调试

<font face="华文彩云" color=red size=5> 本节的介绍都是比较基础的，写给22级或是一些刚接触MDK的；</font>

<font face="华文彩云" color=red size=5>你们会了的自己跳到后面部分去</font>

:point_down: :point_down: 

<a href="#two">传送门</a>



---



### 1.1 keil5 进入调试

#### 	1.1.1 选择调试器



**点击魔术棒 ![image-20220725233132528](ARM- MDK 调试汇总.assets/image-20220725233132528.png)-> Debug** 

![image-20220725233149354](ARM- MDK 调试汇总.assets/image-20220725233149354.png)

​										==这里我是用的**J-Link**所以选**J-Link**== 

---



#### 1.1.2  配置调试器



**点击setting进入调速器的设置界面**

<img src="ARM- MDK 调试汇总.assets/f3de5e18d9942f7cde6463efe81cd00.jpg" alt="f3de5e18d9942f7cde6463efe81cd00" style="zoom:80%;" />



<font face="宋体" color=red size=4>大致介绍一下SW 和 JTAG的区别：</font> 

- SWD需要的线很少，仅需要**双向数据线**（SWDIO）、**时钟线**（SWCLK）、以及**VCC**和**GND** 四根线；
- 而且SWD在高速模式下更可靠，而且再增加一根线**SWO**后，可以使用<a href="#ITM">ITM</a> 在线调试，功能更加强大。
- **JTAG**接口一般可转接成**SWD**，通常能用**JTAG**的都能使用**SWD**， 除非是在调试带宽不够时，不能一般用**SWD**就够了



----

#### 1.1.3 配置flash

**点击flash Download**

<img src="ARM- MDK 调试汇总.assets/image-20220725235613035.png" alt="image-20220725235613035" style="zoom:80%;" />



#### 1.1.4 进入debug

![image-20220725235735387](ARM- MDK 调试汇总.assets/image-20220725235735387.png)

----



### 1.2 调试窗口介绍



**初始化后的默认调试窗口布局如下：**

![image-20220726000354137](ARM- MDK 调试汇总.assets/image-20220726000354137.png)

:one:调试操作栏

```txt
[复位|运行|停止|单步运行|单行运行|跳出函数体|运行到光标处|跳转PC指针]
调试时代码的运行操作在此设置
```

:two:主界面窗口设置

**[命令行|反汇编窗口|符号窗口|内核寄存器|回调栈及局部变量|变量显示|内存窗口|虚拟串口|系统分析窗口|TRACE窗口|外设寄存器]**

1. **命令行：**==调试操作命令==及返回等消息在这里显示，也可以直接在这里操作执行一些语法命令，如断点打开关闭显示等。
2. **反汇编窗口：**C/C++编写的代码转换为==汇编语言==，此窗口与代码窗口同步滚动，但并不是代码执行时的顺序**(可能是编译的顺序🤔)**
3. **符号窗口：**编译后所有的==函数变量及类型==均在此，可通过这里查看有没有编译某些变量
4. **内核寄存器：**显示内核先关的==寄存器数值==
5. **回调部变量：**当调试时停留在断点时，这里会显示函数的调用关系，以及压进的变量值，这个一般和断点配合反推异常调用以及查看局部变量无需使用变量查看串口
6. **变量显示**：用于查看单片机中某个==变量的值==，一般多用于查看全局变量以及外设寄存器数值、表达式显示变量，也==可直接操作变量值。== (**相当于直接改变内存**🤔)
7. **内存窗口：**显示内存地址以及地址处内存的数据，一般可==查看变量==以及==寄存器数据==、==函数地址==等
8. **虚拟串口窗口：**通过调试查看的串口，一些需要重定向才能使用。(也可用`ITM_SendChar(ch)`函数，详见<a href="#ITM">ITM</a> )
9. **系统分析窗口：**这里有一些代码分析的高级功能，用的最多的是==软件逻辑分析仪==的功能。(可用作**简易示波器**，和**显示变量波形**）
10. **trace窗口：** 与==代码跟踪==有关，==相对高级的功能==下面有介绍<a href="#ITM">ITM</a> ，具体可参考keil5的[帮助文档](https://developer.arm.com/documentation/101407/0537/Debugging/Debug-Windows-and-Dialogs/Trace-Exceptions)。
11. **外设寄存器窗口：**==比较常用==，用来查看你打开的串口/spi/can/Tim/rcc等等寄存器的内容,当然也可==直接修改操作寄存器==。

:three:在线调试启动/关闭

:four:断点设置 设置、取消、关闭、删除所有断点等操作。
:five:内核寄存器窗口
:six:C代码对应的汇编程序窗口 **注意代码左侧灰色部分代码已编译部分，可设置断点(此部分可以查看代码是否编译或者是否被编辑器优化)**

:seven:   C/C++代码窗口
:eight:Command窗口 调试命令行，可通过设置命令执行所有调试操作，如断点设置、变量寄存器访问、数据转换、基本计算等功能
:nine:回调栈及局部变量窗口

:keycap_ten: 用于查看单片机中某个==变量的值==，一般多用于查看全局变量以及外设寄存器数值、表达式显示变量，也==可直接操作变量值。== (**相当于直接改变内存**)



(:point_down:更多的可以参考一下这篇文章)

[keil 基础调试](https://mp.weixin.qq.com/s?__biz=MzU2MDgyNzgyMw==&mid=2247483787&idx=2&sn=f52924e67df948b00c42bf4993c191d6&chksm=fc035d4bcb74d45d844202d624b244b6931f568266a359eea51c15d5b8578059d34e89f9c605&scene=178&cur_album_id=1341389839287156737#rd)

------------------------------------------------

### 1.3  基本调试操作

#### 1.3.1  全速运行、打断点、查看变量

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/2.mp4" type="video/mp4"> </video>



#### 1.3.2  复位、停止程序

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/3.mp4" type="video/mp4"> </video>





#### 1.3.3  单行/单步运行



<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/4.mp4" type="video/mp4"> </video>





---



## :two: 具体调试

<span id="two" name="two"></span>

### 2.1  寄存器直接操作

#### 2.1.1 中断控制器

嵌套向量中断控制器 (NVIC) 对话框（适用于 Cortex-M3、Cortex-M4 和 Cortex-M7 内核）显示所有异常的状态。对于每个异常，对话框都会显示编号、来源、名称、状态和优先级。



##### 2.1.1.1 官方文档

![image-20220726180924428](ARM- MDK 调试汇总.assets/image-20220726180924428.png)



官方文档指出我们可以==选定指定中断==，并控制==改变异常的状态==

（:point_down:感兴趣的可以看一下官方文档）

[官方文档](https://developer.arm.com/documentation/101407/0537/Debugging/Debug-Windows-and-Dialogs/Core-Peripherals/Armv7-M-cores/Armv7-M--Nested-Vector-Interrupt-Controller)



##### 2.1.1.3  中断控制器介绍

<img src="ARM- MDK 调试汇总.assets/image-20220727160612637.png" alt="image-20220727160612637" style="zoom:80%;" />



你的中断设置的**优先级是多少**，**是否开启了**，**是否挂起了**，**是否处于活动状态**，在这里一目了然。

假如你想要让程序**尽快进入中断**程序运行，只要**勾选挂起选项**，这样程序运行后马上就能到**中断处理函数中**执行了。





当然这里只是简单介绍，具体内容可以自己去看文档

（:point_down:感兴趣的可以看一下官方文档）

[官方文档](https://developer.arm.com/documentation/101407/0537/Debugging/Debug-Windows-and-Dialogs/Core-Peripherals/Armv7-M-cores/Armv7-M--Nested-Vector-Interrupt-Controller)





----







##### 2.1.1.2 开启方法

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/8.mp4" type="video/mp4"> </video>







----

#### 2.1.2  外设寄存器查看

##### 2.1.2.1 开启方法

在`systemview window` 窗口可以查看各个外设的寄存器值

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/10.mp4" type="video/mp4"> </video>



##### 2.1.2.2 外设寄存器查看



图中示例了串口1的寄存器查看，同样的==中断设置==/==时钟分频==/==定时器==均可通过此查看。甚至可以通过数据的变动与否判断外设是否正常打开。

<img src="ARM- MDK 调试汇总.assets/image-20220726211842092.png" alt="image-20220726211842092" style="zoom:67%;" />





##### 2.1.2.3  直接修改寄存器值

可通过在外设寄存器窗口，直接修改寄存器的值，以控制寄存器

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/9.mp4" type="video/mp4"> </video>



视频中，可以看到，通过控制**TIM8的CCR寄存器**，来改变**PWM的占空比**。



##### 2.1.2.4   输出IO控制

<span id="外设窗口" name="外设窗口"></span>

可以通过控制GPIO的ODR寄存器

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/11.mp4" type="video/mp4"> </video>





##### 2.1.2.5  输入IO检测

当某个单片机IO口设置为输入时，可直接通过输入寄存器查看电平状态即使代码处于断点停止状态也可以查看(断点停止有时需要手动点击**toolbox**中的更新窗口按钮)，在项目中可用来**检测外部开关信号**等省去了万用表测电压。

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/12.mp4" type="video/mp4"> </video>



##### 2.1.2.6 直接控制串口输出

通过直接修改串口的数据寄存器来进行操作

<img src="ARM- MDK 调试汇总.assets/image-20220726214720503.png" alt="image-20220726214720503" style="zoom: 67%;" />





<font face="华文彩云" color=red size=5>类似直接通过寄存器来进行调试还有很多示例，这里仅演示其中几种，有其他比较有用的也欢迎补充。</font>



----





### 2.2  测量某段代码运行时间

方法比较多， 如利用系统内核计算（dwt）、利用其它工具如<a href="#Systemview">Systemview</a> 、利用断点等



#### 2.2.1  利用内核计数（dwt）

##### 原理

<img src="ARM- MDK 调试汇总.assets/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xhaWZlbmd5dWFuMQ==,size_16,color_FFFFFF,t_70.png" alt="img" style="zoom:80%;" />

如：

```c
//测试代码运行时间（加在被测代码块之后）
float DTW_Time_Difference_ms(void)
{
  static uint32_t old_counter;
  uint32_t counter,couter_current;
  couter_current = DWT_get_time();
  if(couter_current > old_counter)
    counter = couter_current - old_counter;
  else
    counter = couter_current + 0XFFFFFFFF - old_counter;
  old_counter = couter_current;
  return (counter / (SystemCoreClock/1000));
}

/*********************************************************/
{
DTW_Time_Difference_ms();
... 被测代码
    
time = DTW_Time_Difference_ms();
}    
```





##### 运行结果



![image-20220726142150301](ARM- MDK 调试汇总.assets/image-20220726142150301.png)



![image-20220726153314600](ARM- MDK 调试汇总.assets/image-20220726153314600.png)



<font face="华文彩云" color=red size=5>可以看到基本能实现US级的 测量</font>

---

#### 2.2.2  利用断点时间测量

该方法也是基于基于内核时钟，进行计数测量的，经过笔者的测试，在大部分情况下，**精度都是非常高的**



##### 2.2.2.1 前置步骤 

要用到断点，也即是keil本身来获取准确时间，需要先进行Trace功能的配置

（:point_down:具体配置见下文）

<a href="#Trace 配置">Trace 配置</a> 



##### 2.2.2.2  测量前置

![image-20220726161044333](ARM- MDK 调试汇总.assets/image-20220726161044333.png)

1. 打开仿真界面
2. 打开==内核寄存器窗口==
3. 将寄存器窗口保持固定在如图所示界面时钟显示（计时的时钟就是此处的Sec）
4. 鼠标右键弹出菜单
5. 复位t1/t2定时。



<font face="华文彩云" color=red size=5>为什么要打开寄存器窗口：</font>

有的时候，右下角时间窗口![image-20220726161524525](ARM- MDK 调试汇总.assets/image-20220726161524525.png)可能没有显示时间或时间不更新； 一个简单的解决方法是，切换到**寄存器窗口**，这样时间窗口就能正常显示了。



##### 2.2.2.3  测试原理

在**寄存器窗口**显示的时间是从单片机运行的**第一条代码** 开始的时间，这个时间是累计的。



而**右下角时间窗口** 

![image-20220726162829068](ARM- MDK 调试汇总.assets/image-20220726162829068.png)

可以重置（Reset）时间，比如现在用t1显示的时间（t0 和 **寄存器窗口**显示的时间一样，不信你可以看看），只要先重置一下这个t1（最上面那个），然后运行代码后暂停，t1显示的就是这段代码的运行时间了。



##### 2.2.2.4  实际操作

###### 2.2.2.4.1 关闭固定窗口刷新

(代码运行同时刷新变量窗口数据，会影响测量时间的准确性)

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/7.mp4" type="video/mp4"> </video>



###### 2.2.2.4.2  进行测量

==设置断点==后，开启代码运行，下一次测量时应==复位测量计时器T1/T2==.

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/6.mp4" type="video/mp4"> </video>

###### 2.2.2.4.3  测量结果

<img src="ARM- MDK 调试汇总.assets/image-20220726171051157.png" alt="image-20220726171051157" style="zoom: 67%;" />



可以看到**测量结果**和之前用**DWT**进行测量基本是一致的

----

### 2.3  高级断点调试

#### 2.3.1 简介

Keil5软件帮助文档中指明了断点有三种类型：==存取断点==，==执行断点==，==条件断点==。

- **存取断点：**某一个变量度或者写操作时执行断点操作。
- **执行断点**：执行到代码某个位置时产生断点操作。一般直接在代码左侧打的断点就是此类断点，用的最多。
- **条件断点**：当满足某个表达式时，如某个变量==0x01时执行断点。



![image-20220505233316651](ARM- MDK 调试汇总.assets/8e617df86b1714b388d26aad723b1e4b.png)

​															==图为官方文档介绍==



#### 2.3.2 开启方法

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/13.mp4" type="video/mp4"> </video>



#### 2.3.3  窗口介绍

![image-20220726223923570](ARM- MDK 调试汇总.assets/image-20220726223923570.png)





- 仅操作1处表达式，表达式为某个函数地址时为**==执行断点==**，Count表示**执行次数**。

- 1和3处同时操作时表示为==存取断点==，3中`SIZE`表示存取变量的字节大小，`Bytes`与`Objects`的区别在于单一变量与结构体的不同。当存取某个变量或者结构体中某个成员时使用`Bytes`，存取某个结构体时使用`Objects`。
- 仅操作1和2为==条件断点==，如变量==、>=某个值时执行printf指令打印变量值。注意此处的printf仅输出到调试界面的Command窗口。command指令的执行并不会使程序中断停止，相当于此处为软件断点。





**图中官方示例的各断点的定义的意义：**

![image-20220726222939321](ARM- MDK 调试汇总.assets/image-20220726222939321.png)



<img src="ARM- MDK 调试汇总.assets/image-20220726223551422.png" alt="image-20220726223551422" style="zoom:80%;" />



<font face="华文彩云" color=red size=5>注意：</font>

1. 除直接在代码左侧设置执行断点外，其余断点需要通过断点管理窗口来实现。
2. 断点管理窗口中的断点需要停止代码执行才能正确设置。



#### 2.3.4 断点操作

##### 2.3.4.1 存取断点

###### 2.3.4.1.1  **存取断点定义**

可以在Watch中右键添加断点。

![image-20220726230713728](ARM- MDK 调试汇总.assets/image-20220726230713728.png)





如果想知道某个变量在什么地方被访问，假定有一个变量a，可以按以下进行配置：

![image-20220726232613670](ARM- MDK 调试汇总.assets/image-20220726232613670.png)



因为 Count 值设置为 1，所以==每一次读取a的值==，程序都会停止。



###### 2.3.4.1.2  **运行结果** 



在对**变量a** 进行读取的时候，程序自动停止运行

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/14.mp4" type="video/mp4"> </video>



如果是写操作（Write）访问时，会发现从复位程序开始运行后，程序会停止在某个地方。

因为==全局变量==会在进入 main 函数之前被初始化。







还可以更改`Count` 的值，如把count 改成2， 则当a被**写入两次**时，程序会停止。



<img src="ARM- MDK 调试汇总.assets/31cefc441d95f1eb33c3087c9f946347.png" alt="结构体写断点" style="zoom: 67%;" />



----



##### 2.3.4.2 结构体断点

实际结构体断点也是属于==存取断点==的一种，因为要写的比较多，单独列出来



<font face="华文彩云" color=red size=5>注意：这部分我测试的时候感觉有点问题，没完全解决</font>





###### 2.3.4.2.1  设置结构断点



:one: <font face="华文彩云" color=red size=5>出现的问题</font>



试了一下，直接在`watch`窗口 将**整个结构变量**设为断点会报错(包括该变量里的其他结构体成员，如**pitch.c**)

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/15.mp4" type="video/mp4"> </video>

![image-20220727161416471](ARM- MDK 调试汇总.assets/image-20220727161416471.png)



​																		==试着解决了一下 ，没解决==



---

:two: <font face="华文彩云" color=red size=5>简单的解决方法</font>

可以采用更加通用的方法：

![image-20220727162119072](ARM- MDK 调试汇总.assets/image-20220727162119072.png)



可以看到结构体变量各个成员的**地址**是不同的，比如我们`gimbal_control.auto.c` 的地址是`0x20004A10` （由此我们知道也可以通过这

个来看出一个结构体变量的地址是多少），所以设置后的结果如下：

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/16.mp4" type="video/mp4"> </video>





<font face="华文彩云" color=red size=5>注意：经测试好像是不支持指针操作，不太确定</font>

比如有这样三个结构体：`gimbal_pitch_control_t`、`gimbal_control_t`、

```c
gimbal_control_t gimbal_control;

typedef struct
{
    uint16_t position;
    int16_t speed;
    int16_t given_current;
    uint8_t temperate;
    int16_t last_position;


    int16_t angle;
    int16_t speed_filt;
    int16_t first_Flag;
    int32_t yaw_angle;
    int32_t pitch_angle;
    int32_t actual_Position;  //真实位置
} motor_measure_t;


typedef struct  //申明pitch轴电机变量
{
    motor_measure_t *pitch_motor_measure;

    float accel_up;
    float accel_down;
    
    int8_t init_flag;    //初始化成功标志
    float Auto_record_location;                      // 实际位置

    first_order_filter_type_t LowFilt_Pitch_Data;    //P轴低通滤波器
    sliding_mean_filter_type_t Slidmean_Pitch_Data;  //P轴滑动滤波器

    first_order_filter_type_t LowFilt_auto_pitch;    //自瞄P轴低通滤波器
    sliding_mean_filter_type_t Slidmean_auto_pitch;  //自瞄P轴滑动滤波器

    int16_t filt_output; //P轴滤波值

    int16_t output;

} gimbal_pitch_control_t;

typedef struct
{
    const RC_ctrl_t *gimbal_RC;     //底盘使用的遥控器指针
    Vision_Auto_Data_t *auto_c;     //申明自瞄变量

    VisionStatus_E VisionStatus;    // 敌人出现状态
    const Fire_task_t *Fire_task_control;
   
    gimbal_behaviour_e gimbal_behaviour;

    gimbal_pitch_control_t pitch_c;   //申明pitch轴电机变量
    gimbal_yaw_control_t yaw_c;       //申明yaw轴电机变量
    motor_measure_t motor_chassis[4];

    uint8_t Gimbal_all_flag;  //全部初始化完成标志

} gimbal_control_t;

```

如果要调用pitch轴的电机数据：

```c
gimbal_control.pitch_c.pitch_motor_measure->position
```



会导致报错：

![image-20220727163656271](ARM- MDK 调试汇总.assets/image-20220727163656271.png)



----



##### 2.3.4.3  条件断点

有些时候，我们并不关注地址访问情况，而对变量的数据内容感兴趣，比如变量`test_param.test1 == 10`的时候停下来。就可以这样设置：

![image-20220727164350007](ARM- MDK 调试汇总.assets/image-20220727164350007.png)



**运行结果：**

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/17.mp4" type="video/mp4"> </video>



----



##### 2.3.4.4  添加命令

可以在命令行增加命令，例如，程序运行100次后，将flag置1，以达到==控制程序运行的目的==

![image-20220727170551681](ARM- MDK 调试汇总.assets/image-20220727170551681.png)



<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/18.mp4" type="video/mp4"> </video>



通过Command命令还能做到很多事情，比如打印信息什么的

例如：写入变量a 50次后打印信息：

![image-20220727172357919](ARM- MDK 调试汇总.assets/image-20220727172357919.png)



<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/19.mp4" type="video/mp4"> </video>

（:point_down:详见下文）

<a href="#Command ">Command 窗口操作</a>



### 2.4  watch变量查

#### 2.4.1 常规操作

右键添加到变量观察窗口，或者直接选中后拖拉变量到窗口。

观察的对象比较广泛：

==全局/局部变量==、==寄存器==、==函数地址==、==数组结构体==等，对于局部变量只能在变量的==有效局部内==才能显示具体数值。



<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/20.mp4" type="video/mp4"> </video>



#### 2.4.2 表达式操作

表达式可进行简单的数学运算，甚至可以当做一个简单的进制转换、计算器来使用。如下所示，动态显示random的值减去70000，0xf777转换为十进制。

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/21.mp4" type="video/mp4"> </video>





#### 2.4.3 直接修改变量的值



可以直接在变量窗口**修改变量的值**，相当于直接在修改内存。 在**调pid**的时候比较有用， 直接修改，不用退出编译。

（当然这只是**临时**的，不重新编译的话，下次进入debug还会是原来的值）



<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/22.mp4" type="video/mp4"> </video>

---



---

### 2.5  （Call Stack + Locals）回调栈局部变量窗口

#### 2.5.1 常规用法

打断点后，通过回调栈窗口可以看到当前函数的调用情况以及内部变量的值。

![image-20220727175629366](ARM- MDK 调试汇总.assets/image-20220727175629366.png)



当程序封装太多层时可参照如下方式进行一层一层跳转分析。



![image-20220727175927384](ARM- MDK 调试汇总.assets/image-20220727175927384.png)



----

#### 2.5.2  查看进入Hardfault函数的位置

当存在==Hardfault等错误==以及程序死循环时，可以通过==断点==或==停止按钮==停止程序运行，在==回调栈窗口==查看调用

hardfault的位置，位置不是很准确，但是能反应大概位置，然后通过局部变量的值进行判断==异常位置==。下边演示

了数组越界引起了异常，大概定位到了前后的位置，其它的错误定位可以再研究一下。



<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/23.mp4" type="video/mp4"> </video>





---



### 2.6 Command 窗口操作

<span id="Command" name="Command"></span>

<font face="华文彩云" color=red size=5>keil的命令调试功能很强大，用的好能解决很多问题</font>



比如，通过执行命令函数或表达式直接实现查看与修改变量、对象、寄存器及内存。

（:point_down:详见官方文档）

[command 命令](https://developer.arm.com/documentation/101407/0537/Debug-Commands)



这里贴几张官方文档的图：

#### 2.6.1 断点命令

![image-20220727210748102](ARM- MDK 调试汇总.assets/image-20220727210748102.png)

![image-20220727210816082](ARM- MDK 调试汇总.assets/image-20220727210816082.png)



#### 2.6.2 一般命令

![image-20220727210852655](ARM- MDK 调试汇总.assets/image-20220727210852655.png)

![image-20220727210840035](ARM- MDK 调试汇总.assets/image-20220727210840035.png)



#### 2.6.3  内存命令和程序命令

![image-20220727210948094](ARM- MDK 调试汇总.assets/image-20220727210948094.png)

![image-20220727210923858](ARM- MDK 调试汇总.assets/image-20220727210923858.png)



#### 2.6.4 使用示例



![image-20220727220755045](ARM- MDK 调试汇总.assets/image-20220727220755045.png)



<font face="华文彩云" color=red size=5>注意：当退出调试模式之后，KEIL 将自动保存 Command 数据到文件中（也就是说在此之前你是看不到这些调试数据的）</font>



![image-20220727220857248](ARM- MDK 调试汇总.assets/image-20220727220857248.png)





#### 2.6.5 自定义TOOLBOX按键



有些时候我们并不满足监控数据，还想**定义一些自己的按键**，比如当我按下按钮时，系统电源关闭，再按下按钮时系统电源开启等。

这个功能其实使用前面所说的<a href="#外设窗口">外设窗口</a> 也是完成能完成要求的，麻烦的是，如果使用外设窗口，要控制 IO 口，那你每次都得找到对应的 IO 口才行，很是麻烦，但是使用按键就会简单许多。



#### 2.6.5.1 定义按键

###### 2.6.5.1.1 可以通过建立ini文件



（:point_down:ini文件详见）

<a href="#ini">ini 文件</a>



```c
Kill BUTTON * //删除全部按键
DEFINE BUTTON "Power OFF", "GPIOB->ODR |= (0x01 << 7)"  //按键置1
DEFINE BUTTON "Power ON", "GPIOB->ODR &= ~(0x01 << 7)"  //按键置1

```



**保存后导入**

![image-20220727225207329](ARM- MDK 调试汇总.assets/image-20220727225207329.png)



**之后进入 Debug 模式即可，在这里你可以看到你定义的按键：**

![image-20220727225446135](ARM- MDK 调试汇总.assets/image-20220727225446135.png)



###### 2.6.5.1.2 通过命令窗口

之前说过**.ini 文件**和手工在 **Command 窗口**输入命令没啥区别，只是使用文件的话可以将常用命令保存下来。



所以直接在命令窗口写入命令也行

![image-20220727230259459](ARM- MDK 调试汇总.assets/image-20220727230259459.png)



###### 2.6.5.1.3 按键删除

```c
Kill Button 3   //移除第3个按钮  （参数为* 的话会删除所有按键）
```



###### 2.6.5.1.4 运行结果

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/26.mp4" type="video/mp4"> </video>



可以看到通过自定义按键 **控制io口输出**，控制灯的亮灭。



---



<font face="华文彩云" color=red size=5>注意：命令调试也存在缺点</font>

- KEIL 命令调试不支持指针，这个已经多次强调了，要实现指针的功能，只能间接使用。
- 对程序运行造成一定的影响（事实上这个不关 KEIL 的事，是调试系统本身的问题）



调试器可以说是第三方监视器，虽然几乎没有侵入性（事实上对 CPU 还是有影响的），但是它还是会窃取 CPU 时钟的，而且在执行断点的时候，虽然由 ini 文件定义的函数由 KEIL 执行了，实际上上执行这些函数也是需要时间的，那这个时间怎么来，就是通过暂停 CPU 后去执行这些代码，这个你可以通过 DWT 计数器看出来，因为只有 CPU 执行了 DWT 才会计数，但是你会发现在执行这些代码时，DWT 是没有进行计数的（在 KEIL 函数的前后获取 DWT 计数，可以发现计数值不变）



也就是说 CPU 和 KEI 是在交替使用系统时钟的。平常来看，由于 KEIL 执行速度很快，看不出来问题，但到中断的时候却会出现问题。





----



## :three:  ITM程序跟踪

<span id="ITM" name="ITM"></span>

### 3.1 ITM简介

**ITM：**Instrumentation Trace Macrocell，指令跟踪宏单元



ITM是一应用驱动的跟踪源，它支持printf类的调试手段来跟踪操作系统(OS)和应用事件，并发布判定的系统信息。ITM以包的形式发布跟踪信息，它由以下部分组成：
● 软件跟踪：软件可以通过直接写ITM激发寄存器来发布包信息。
● 硬件跟踪：ITM会发布由DWT产生的信息包。
● 时间戳：时间戳被发布到相应的包上。ITM包含一个21位的计数器以产生时间戳。Cortex-M3的时钟或串行线观测器(Serial Wire Viewer)的位时钟率给计数器提供时钟。
由ITM发送的信息包输出到TPIU(Trace Port Interface Unit)，TPIU再添加一些额外的包(参考TPIU)，然后输出完整的包序列给调试器。



使用ITM能做到，在keil上显示各个中断的运行时间，进入时间；打印变量波形；打印虚拟串口数据等；



==简单添加几张效果图==：

:one:中断时间戳：

![image-20220727235419383](ARM- MDK 调试汇总.assets/image-20220727235419383.png)



:two: 打印变量波形：

![image-20220727235646860](ARM- MDK 调试汇总.assets/image-20220727235646860.png)



:three: 打印虚拟串口：

![image-20220727235835065](ARM- MDK 调试汇总.assets/image-20220727235835065.png)



-----



### 3.2  Trace配置

<span id="Trace 配置" name="Trace 配置"></span>

#### 3.2.1 硬件准备

##### 3.2.1.1 仿真器准备



一个**J-Link仿真器**或是**ST-LINK**、有**J-Trace**更好，直接一步到位。



不管是什么仿真器，只要能引出VCC、GND、TMS/SWDIO、TCK/SWCLK、JTDO这5根线的话就可以尝试一下。

| 序号 | 连接网络 | 备注           |
| ---- | -------- | -------------- |
| 1    | VCC      | 电源正，3.3V   |
| 2    | GND      | 电源负         |
| 3    | SWO      | SW跟踪功能     |
| 4    | SWDIO    | SW数据输入输出 |
| 5    | SWCLK    | SW时钟信号     |



这里我用的是J-Link：

<img src="ARM- MDK 调试汇总.assets/image-20220728000426452.png" alt="image-20220728000426452" style="zoom:67%;" />



==通过平常的SWD四线接口，再加上一根SWO的飞线==



##### 3.2.1.2 开发板准备



准备一个有引出**==PB3==**引脚的板子

![image-20220728001142766](ARM- MDK 调试汇总.assets/image-20220728001142766.png)

​											==图为数据手册==



<font face="华文彩云" color=red size=5>我看了周围我能用的板子汇总如下：</font>

:one: 

<img src="ARM- MDK 调试汇总.assets/image-20220728001822760.png" alt="image-20220728001822760" style="zoom:80%;" />



>**开发板：** ACE实验室H7通用开发板
>
>>**有无引出PB3：**  否
>
>>**能否飞线：**        否
>
>
>
>**说明：** 因为采用50pin 0.5mm的贴片排针，故无法飞线。



:two:

![image-20220728002422648](ARM- MDK 调试汇总.assets/image-20220728002422648.png)



>**开发板：** ACE实验室F4系列开发板
>
>>**有无引出PB3：**  否
>
>>**能否飞线：**        能
>
>
>
>**说明：** 可从最小系统板飞线，也可从拓展板底部飞线



![image-20220728003548132](ARM- MDK 调试汇总.assets/image-20220728003548132.png)



<font face="华文彩云" color=red size=5>最终我是选择了用F4的板子，飞线进行测试，之后可以让硬件在设计的时候就引出来PB3</font>



:three:

![image-20220728002623027](ARM- MDK 调试汇总.assets/image-20220728002623027.png)



>**开发板：** 正点原子精英板
>
>>**有无引出PB3：**  有
>
>>**能否飞线：**        能
>
>
>
>**说明：** 调起来怪怪的，能用但不想用，也没必要，而且还要外接电源供电





:four: 

![image-20220728002843443](ARM- MDK 调试汇总.assets/image-20220728002843443.png)

>**开发板：** RM官方开发板C型
>
>>**有无引出PB3：**  无
>
>>**能否飞线：**        否
>
>
>
>**说明：** PB3被用于SPI1_CLK ,也无法复用:sob: ， 所以说能不用官方板就别用， 太受限了！！
>
>



---



#### 3.2.2 软件配置

##### 3.2.2.1 CUBEMX配置

选择异步跟踪模式，此时PB3将会自动定义为SWO端口。

![在这里插入图片描述](ARM- MDK 调试汇总.assets/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NjYyMzM1MA==,size_16,color_FFFFFF,t_70.png)



##### 3.2.2.2 KEIL软件设置



**在debug 的Trace窗口进行配置：**



确保用的是==SW模式== :

![image-20220728134449387](ARM- MDK 调试汇总.assets/image-20220728134449387.png)



---



### 3.3  keil调试的ini文件用法

<span id="ini" name="ini"></span>

<font face="华文彩云" color=red size=5>通过ini文件，可以动态的分析调试，也可以还原bug发生的情景。</font>

（感觉很有用，但我自己只是简单测试，就不乱说了，后面附上文章自己看）



#### 3.3.1 ini 文件使用

ini文件可解释为一个配置文件，相当于一个**.C文件**，这个文件的执行本质与仿真时的**命令行**执行一致(如果觉得不麻烦可以在命令行中敲所有的命令而不用加载ini文件).



**==ini文件的加载主要在两个地方：==**

:one: 点击调试时会加载图片所示的ini文件:

![在这里插入图片描述](ARM- MDK 调试汇总.assets/b1dc527f3d36d44e51c54717b74c7579.png)



:two: 调试时使用专用管理器调用

<img src="ARM- MDK 调试汇总.assets/1028363a017fec65d7b424b11b17bb8b.png" alt="在这里插入图片描述" style="zoom: 50%;" />

<img src="ARM- MDK 调试汇总.assets/1adfd2d3dc41c756f343fee1e7817f9f.png" alt="在这里插入图片描述" style="zoom:67%;" />

调用ini主要用来生成一些配置，如打开**itm端口**、**生成log**、**设置断点**等功能，下边介绍一下小功能。



----

#### 3.3.2 **新建按钮**

使用任何编辑器新建后缀为.ini的文件，内容如下：

```c
DEFINE BUTTON "显示random值", "printf (\"random=%04XH\\n\",random)"
DEFINE BUTTON "置位", "GPIOF->ODR |= (1<<9)"
DEFINE BUTTON "复位", "GPIOF->ODR &= ~(1<<9)"
```

这样即可实现按钮的多个定义。





#### 3.3.3 功能函数

命令行不能直接调用函数，与之前讲过的一样，这里的功能函数仅用来调试使用，可以定义一些打印输出，修改变量、外设寄存器值等操作。

使用任何编辑器新建后缀为.ini的文件，内容如下：

```c
FUNC void clearValue(void)
{
 random =0;
}
FUNC void LEDON(void)
{
 GPIOF->ODR |= (1<<9);
}
DEFINE BUTTON "CLEAR","clearValue();"
DEFINE BUTTON "LED","LEDON();"

```

以上内容为定义了两个函数清除变量**clearValue**和打开LED灯**LEDON**，然后将两个函数做成**toolbox**按钮。



#### **3.3.4 不复位在线调试**

###### 3.3.4.1 使用SWD4线连接目标板(Jtag接口会产生复位)

###### 3.3.4.2 外部工程目录下创建一个noresetDBG.ini配置文件，内容为：

```c
LOAD %L INCREMENTAL
```



没有这个也可以不复位调试，但打不了断点

<img src="ARM- MDK 调试汇总.assets/9914c825e8b5c0a04053b9aa2592a185.png" alt="https://gitee.com/jay_0/open_picture/raw/master/img/image-202205090257236451.png" style="zoom:67%;" />



###### 3.3.4.3 取消勾选`Load Application at Startup`，加载调试初始化配置

<img src="ARM- MDK 调试汇总.assets/461b8ddeba444b49c892a849a8dfaf13.png" alt="image-20220509025437380" style="zoom:67%;" />

(2也可省略，直接在仿真后的command窗口输入效果是一样的)

<img src="ARM- MDK 调试汇总.assets/42fd011438daadd64647d781c0517fec.png" alt="img" style="zoom:67%;" />



###### 3.3.4.4 取消调试器连接后的复位(这个建议不连接板子，只插烧写器进行设置，否则会造成复位一次)

<img src="ARM- MDK 调试汇总.assets/fe746c4fb333d15a6c6c95ef1e76e444.png" alt="image-20220509030107531" style="zoom: 50%;" />



###### 3.3.4.5 调试时目标更新也取消勾选(这一步在F407上调试发现不操作也能实现，可有可无的样子)



![image-20220509030107531](ARM- MDK 调试汇总.assets/fe746c4fb333d15a6c6c95ef1e76e444-165891717886615.png)



###### 3.3.4.6 效果测试:

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/24.mp4" type="video/mp4"> </video>

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/25.mp4" type="video/mp4"> </video>

（:point_down:详见下文）

[KEIL 调试的ini文件有什么用](https://mp.weixin.qq.com/s?__biz=MzU2MDgyNzgyMw%3D%3D&chksm=fc035d33cb74d425b8daf8e162aa4f24b2993698426823b7c77f25e38d7c7fa6730f202f995f&idx=1&mid=2247483891&scene=21&sn=569d5e4f49cfff2b59da5b68b707fe51#wechat_redirect)

[BUG 终结者，现场抓获！|颠覆认知](https://mp.weixin.qq.com/s?__biz=MzU2MDgyNzgyMw==&mid=2247484451&idx=1&sn=73a0e08b6fbe15982fecd70c56f1ef66&chksm=fc0358e3cb74d1f56616fc0d8d5148ee3382280bb95de908b4adff1eddcdb3cf446c307c32b1&scene=178&cur_album_id=1341389839287156737#rd)



---

### 3.4  打印变量波形

keil本身是有逻辑分析仪的，你们应该都知道，但一般我们调车的时候**打印波形**都是用的<font face="宋体" color=red size=5>J-Scope</font>，因为正常我

们不配置<a href="#ini">ITM</a>的话，波形是不会刷新的，就一直卡在那里。



#### 3.4.1 使用方法

直接将变量添加至`analyzer`窗口就行

<font face="华文彩云" color=red size=5>注意：最多只能有四个变量</font>

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/27.mp4" type="video/mp4"> </video>



#### 3.4.2 与J-Scope的区别

- ==J-Scope==： 采样频率高，波形更加准确，可以将不同波形放在一起对比； 例如，将pitch轴的**实际曲线**和设**定**

  **曲线**对比，可以直观的看出当前的**闭环控制效果**

  

- ==逻辑分析仪：==  方便快捷，直接将变量拖入即可显示，前提是已经配置好了；缩放范围大，可更直观的看出整

  体的趋势； 可查看io端口的电平变化。



#### 3.4.3 使用示例：

通过逻辑分析仪 **打印电机速度曲线**：



<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/29.mp4" type="video/mp4"> </video>





----



### 3.5 虚拟串口



直接用虚拟串口**打印信息**，不再需要占用串口来打印信息：

利用这个函数，**直接调用**就行

```c
/**
  \brief   ITM Send Character
  \details Transmits a character via the ITM channel 0, and
           \li Just returns when no debugger is connected that has booked the output.
           \li Is blocking when a debugger is connected, but the previous character sent has not been transmitted.
  \param [in]     ch  Character to transmit.
  \returns            Character to transmit.
 */
__STATIC_INLINE uint32_t ITM_SendChar (uint32_t ch)
{
  if (((ITM->TCR & ITM_TCR_ITMENA_Msk) != 0UL) &&      /* ITM enabled */
      ((ITM->TER & 1UL               ) != 0UL)   )     /* ITM Port #0 enabled */
  {
    while (ITM->PORT[0U].u32 == 0UL)
    {
      __NOP();
    }
    ITM->PORT[0U].u8 = (uint8_t)ch;
  }
  return (ch);
}

```



这个函数相当于串口字节发送函数，只不过串口字节发送函数是通过串口传输的，这里就通过JTDO这根线传输。然后你可能需要使用printf函数打印，那么重定向即可

![img](ARM- MDK 调试汇总.assets/20190118201030717.png)





**查看输出：**

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/28.mp4" type="video/mp4"> </video>



---



### 3.6 统计进入中断的数据



#### 3.5.1 Trace Exceptions简介

**Trace Exceptions**窗口显示有关跟踪的异常和中断的统计数据，如**在此中断中花费的最短和最长时间**、 **上次输入**

**此异常的时间**等。



==这是官方文档的介绍：==

![image-20220728142909163](ARM- MDK 调试汇总.assets/image-20220728142909163.png)







#### 3.5.2 Event Counters简介

事件计数，用于统计==总指令==、==异常==、==休眠==、==存储==、==折叠==指令。

![在这里插入图片描述](ARM- MDK 调试汇总.assets/6fa96e210a9ff893232b51508acb1c62.png)



:point_down:  ==官方文档介绍：==

![image-20220728143344329](ARM- MDK 调试汇总.assets/image-20220728143344329.png)



#### 3.5.3 使用示例



通过**can分析仪**，发送can数据，并**查看统计情况**：

<video id="video" controls="" preload="none"> <source id="mp4" src="./keil5 调试汇总附录/30.mp4" type="video/mp4"> </video>



## :four:  其它调试工具

### 4.1 Systemview 

<span id="Systemview" name="Systemview"></span>

#### 4.1.1 Systemview简介

[SystemView](https://www.segger.com/products/development-tools/systemview/)是SEGGER开发的针对嵌入式系统的trace工具，支持多种RTOS，也支持自定义OS的移植（需实现trace API，参见User

 Manual）。其核心基于**SEGGER RTT**，一个**Host-Targe**t间的通信框架，可通过多种方式连接，除J-LINK之外还可以使用串口及TCP-IP协

议，对非商业用途免费且无功能限制。



SystemView 是一个用于虚拟分析嵌入式系统的工具包。SystemView 可以完整的深入观察一个应用程序的运行时行为，这远远超出一个

调试器所能提供的。这在开发和处理具有多个线程和事件的复杂系统时尤其有效。

[官方介绍](http://www.segger.com/products/development-tools/systemview/)





#### 4.1.2 使用效果



![image-20220720231458954](ARM- MDK 调试汇总.assets/image-20220720231458954.png)



[SystemView 介绍与移植](./keil5 调试汇总附录/SystemView 介绍与移植.pdf)





---



### 4.2  J-Link的RTT打印

可以通过占用一部分内存空间来实现非在线调试时swd端口的数据输出，一般用在串口紧张时代替串口实现printf调试。



我没用过，所以具体见网上教程

（:point_down:详见下文）

[JLink的RTT使用](https://blog.csdn.net/qq1291917670/article/details/119414735?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-3-119414735-blog-120769285.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-3-119414735-blog-120769285.pc_relevant_default&utm_relevant_index=6)



----





### 4.3 J-Scope

#### 4.3.1  J-Scope 简介

J-Scope是SEGGER公司推出的，可以在目标MCU运行时，实时分析数据并图形化显示的软件。它不需要SWO或目

标上的任何额外引脚等功能，但使用可用的标准调试端口。J-Scope可以以类似示波器的方式显示多个变量的值。



<font face="宋体" color=red size=5>可以用于平时调车的时候，打印变量波形，如对比电机设定曲线和实际曲线来观察闭环控制效果。</font>





#### 4.3.2 使用效果



==如图是六轴陀螺仪三轴的工作曲线：==

![image-20220716174600639](ARM- MDK 调试汇总.assets/image-20220716174600639.png)





（:point_down:**官网**及**使用教程**）

[J-Scope 官网](https://www.segger.com/products/debug-probes/j-link/tools/j-scope/)

[使用教程](https://blog.csdn.net/qq_23852045/article/details/108837881)





----



### 4.4 MATLAB



作为一个工科生，==MATLAB==有多重要就不用多说了吧，基本什么都能干。 比如，我们调车的时候，云台的串级pid大部分都是用的经验调

参。如果能熟练掌握==MATLAB== ,就可以通过仿真来整定参数。

![image-20220728150321273](ARM- MDK 调试汇总.assets/image-20220728150321273.png)





（:point_down:具体的实现方法可以参考这两篇文章）

[系统辨识基础](https://bbs.robomaster.com/thread-4941-1-1.html)

[云台传递函数](https://bbs.robomaster.com/thread-5059-1-1.html)



----





## :exclamation:参考文章

[官方文档](https://developer.arm.com/documentation/101407/0537/Debugging/Code-and-Data-Trace--Cortex-M-/Tracepoint-Expressions)

[ keil 调试经验总结](https://blog.csdn.net/weixin_42876465/article/details/107171974?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165881686416780357239382%2522%252C%2522scm%2522%253A%252220140713.130102334.pc%255Fblog.%2522%257D&request_id=165881686416780357239382&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~first_rank_ecpm_v1~rank_v31_ecpm-1-107171974-null-null.185^v2^control&utm_term=%E8%B0%83%E8%AF%95&spm=1018.2226.3001.4450)

[STM32调试利器之ITM](https://blog.csdn.net/weixin_42876465/article/details/86545913)

[KEIL 调试的 ini 文件有什么用](https://mp.weixin.qq.com/s?__biz=MzU2MDgyNzgyMw%3D%3D&chksm=fc035d33cb74d425b8daf8e162aa4f24b2993698426823b7c77f25e38d7c7fa6730f202f995f&idx=1&mid=2247483891&scene=21&sn=569d5e4f49cfff2b59da5b68b707fe51#wechat_redirect)

[STM32开发，通过SWO功能输出Printf函数到Utility](https://blog.csdn.net/weixin_46623350/article/details/105305426)
