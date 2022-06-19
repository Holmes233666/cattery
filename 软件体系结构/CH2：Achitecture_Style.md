# <center>CH2：Achitecture Style</center>

MindMap：

![Architecture Styles](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/Architecture%20Styles.png)

## 2.1 架构风格的定义

不存在统一的定义：

架构风格 Achitecture Style  = 

- 一组组件类型 component type / 连接件类型 connector type（交互机制）
- 组件的拓扑分布
- 一组对拓扑和行为的约束
- 一些对风格的代价和益处的非正式描述

如何描述的？重点要看

---

分类法：

![image-20220610194509625](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610194509625.png)

纯粹的体系结构在现实中很难遇到，实际上的系统通常：

- 经常偏离学术定义
- 典型的，融合很多的体系风格的特色

作为一个架构师，必须理解“纯”的风格，理解优点和缺点，理解背离这种风格会带来什么结果

- 没有完备的列表
- 风格彼此是重叠的
- 一个系统通常表现出来多种风格

## 2.2 Data Flow

### 2.2.1 数据流风格概述

#### （1）风格

数据流系统：

- 数据控制计算 the availability of **data controls the computation**

- 系统结构由数据在处理之间的有序移动决定

  the structure of the design is dominated by orderly motion of **data  from process to process**

  <img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610194442613.png" alt="image-20220610194442613" style="zoom:50%;" />

- 数据流系统的结构是显而易见的

**纯数据流**系统中，处理之间**除了数据交换，没有其他的别的交互**，变化的是：

- 如何施加控制——数据的控制方式

  - pull    拉

    消费者角度

  - push 推

    从数据源角度

- 并行的程度

  一条数据流，多条数据流

- topology

  顺序，循环

#### （2）Component & Connector & System

- Component

组件接口是输入端口和输出端口

Interfaces are **input ports** and **output ports**

- Connector：Data Stream

通常是异步的，有缓冲。同步的容易阻塞

- System

任意拓扑结构，函数式编程

![image-20220610195210165](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610195210165.png)



#### （3）数据流和控制流

- 控制流

主要问题是**控制点control**怎么在程序或者系统之间移动

数据可能跟着控制走，但是并不起到推动系统运转的作用

关注的核心是计算顺序

eg：冯诺依曼结构

- 数据流

主要问题是数据怎么在运算单元之间流动

数据到了，计算单元便工作

我们关心数据是否可用，转换，延迟

### 2.2.2 Three Examples of Data Flow

#### 2.2.2.1 Batch Sequential 批处理

##### （1）图

![image-20220610195836168](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610195836168.png)

每一个处理步骤是一个独立的程序，每一步在上一步结束后才能开始，数据必须是完整的，以整体的方式传送。

##### （2）应用

典型应用：

- 编译器
- case工具

#### 2.2.2.2 Pipe-and-Filter 管道过滤器

##### （1）图

![image-20220610200731177](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610200731177.png)

##### （2）管道与过滤器

- fliter

功能：将数据流作为输入，经过处理后输出

进行流到流的转换：
$$
\begin{cases}丰富数据\\精炼数据\\转换数据\end{cases}
$$
数据不再是自包含的

fliter是无状态的计算。没有上下文，状态保存等。

- pipe

将数据流从一个过滤器传递到另一个过滤器

数据传送引起动作

##### （3）优点

- 隐蔽性，高内聚，低耦合
- 可支持并发，多个过滤器并发执行
- 系统容易维护和扩展

##### （4）缺点

- 不适合交互性很强的应用——计算过程事先确定
- 数据传输没有通用标准，每个过滤器需要额外解析和合成数据

##### （5）数据流与管道过滤器的区别

![image-20220610201720795](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610201720795.png)

#### 2.2.2.3 Proccess Control 控制

- 开环控制 Open loop Control

- 闭环控制Close Loop Process Control

  闭环控制有两种形式：反馈控制和前馈控制

![image-20220610203019341](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610203019341.png)

当软件系统的运行受到外部干扰时，需要为软件体系结构考虑一种过程控制范例

### 2.2.3 选择数据流方式的方针

任务由数据主导

事先知道数据的流向

数据流动带来性能损坏

## 2.3 Call/Return

### 2.3.1 风格类型

经典的例子：

- 主程序和子程序 main program and subroutines

  经典编程范式：函数分解

- 面向对象的抽象数据类型 

  信息的隐藏

- 层次化结构

  每一层只和他的近邻通信

- 其他

  客户机-服务器

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610204045295.png" alt="image-20220610204045295" style="zoom:50%;" />

### 2.3.2 历史过程

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610204503976.png" alt="image-20220610204503976" style="zoom:50%;" />
$$
主程序（子程序）\rightarrow 函数模块\rightarrow ADT\rightarrow Object\rightarrow OO架构\rightarrow Component
$$

### 2.3.3 Call/Return的具体风格类型

#### 2.3.3.1 Main Program and Subroutine

##### （1）示意图

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610205124396.png" alt="image-20220610205124396" style="zoom:50%;" />

Problem：适用于通过**过程定义层次结构**适当定义计算的应用程序

Context：命名空间局部性

Solution：

- 系统模型：**调用和定义层次结构，子系统通常模块化定义**

- 组件：过程和显式可见数据

  所有数据对外完全可见

- 连接器：过程调用和显式数据共享

- 控制结构：单线程

##### （2）Pipe Versus Procedures

![image-20220612155153229](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220612155153229.png)



#### 2.3.3.2 Data Abstaction or Object-Oriented

系统模型：局部化状态保持

组件：对象

连接件：过程调用

控制结构：去中心化，通常是单线程

##### （1）模块分解

隐藏细节：

- 数据+策略

隐藏可能改变的细节，接口中公开不太可能改变的假设

##### （2）封装和信息隐藏

对象有状态和操作，但是也有完整性。

对象被外界知道的是他的接口

对象从模板产生

##### （3）示意图

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610210621525.png" alt="image-20220610210621525" style="zoom:50%;" />

对象架构的元素：
$$
\begin{cases}封装：限制信息的访问\\交互：过程调用或其他协议\\多态：运行时选择具体的操作\\继承：对共享的功能保持一致的接口\\复用和维护：封装和聚合提高生产力\end{cases}
$$

##### （4）优缺点

- 管理大量的对象

  对象的海洋需要额外结构容纳

- 管理很多交互

  单一的接口能力有限且笨拙（友元）

- 分散的行为责任

  系统的功能难以理解

- 捕获关联的设计

  类和类型通常是不够的，设计模式是发展

#### 2.3.3.3 Layered system

##### （1）OSI模型的例子

##### （2）概述

- 适用问题

包含可以按层次结构安排的**不同服务类别**

- 系统模型

不透明的层次结构

- 构件

各层次内部的构件，过程的集合

- 连接件

取决于组件的结构;在受限的可见性下，过程调用通常也可能是客户机/服务器

- 限制
- 控制结构

单线程

##### （3）层次风格的特点

- 每一层为上一层提供服务，使用下一层的服务，只能见到与自己相邻的层
- 大问题逐渐分解成为若干个渐进的小问题，逐步解决，隐藏了很多复杂度
- 修改一层，最多影响两层，而通常影响上层，接口稳固，则对谁都不影响
- 上层必须知道下层的身份，不能调整层次之间的顺序
- 层层相调，影响性能

#### 2.3.3.4 Client/Server

分类：
$$
\begin{cases}两层CS结构\\三层CS结构\\BS结构\end{cases}
$$

##### （1）两层CS

- 客户机应用程序、数据库服务器、网络
- 特点：瘦服务器，胖客户机
- 缺点：
  - 客户端对硬件软件配置要求高，客户机臃肿
  - 升级维护较为困难
  - 数据不安全，客户端可以直接访问服务器数据
  - 信息内容和形式单一
  - 客户端程序编写困难

##### （2）三层CS

- 客户机、数据库服务器、应用服务器、网络
- 特点：瘦客户机
- 应用功能划分：功能层、表示层、数据层

##### （3）BS架构

三层CS的特例

客户端使用http浏览器即可，使用http协议，省去了很多麻烦

只能拉，不能推

客户端之间的通信只能通过服务器进行中转

对客户机和其他网络资源的利用受限

客户端资源浪费，服务器压力较大

BS的速度相对于CS慢

## 2.4 Data Center / Data Sharing

### 2.4.1 Shared Information System 以数据为中心的体系结构风格概述

#### 2.4.1.1 组件和风格特征

这种风格描述了很多系统，共同特点是**共享数据**

收集、操作、保存大量的数据

定义：以数据为中心的风格架构涉及到一种**共享数据源**方法来**传递信息**。

example：

- 剪切板
- 注册表
- 数据库

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614194759799.png" alt="image-20220614194759799" style="zoom:50%;" />

- 优点

系统耦合程度低

方便添加/删除/生产/消费/操纵数据

单个生产者的错误无影响

- 问题

同步问题——数据发生改变时，控制反转，数据仓库压力大

配置和管理

ACID特性

---

#### 2.4.1.2 应用场合

早期的数据共享出现在批处理系统

迫切需要数据时即时存取

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614200755982.png" alt="image-20220614200755982" style="zoom:50%;" />

如今：

应用在各个场景，比如web就是一个巨大的分布式数据库

GitHub

### 2.4.2 Repository Architecture 

#### 2.4.2.1 概述

仓库是共享和维护信息中心的场所

组件：

- **中心数据结构**，表示当前的数据状态
- 一组对中心数据结构进行操作的**独立组件**

连接件：

- **计算单元**与**中心数据结构**之间通过之间数据访问或者过程调用的**交互**

控制结构：



![image-20220614201149419](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614201149419.png)

典型应用场合：

数据库



如，编译器的符号表：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614200941102.png" alt="image-20220614200941102" style="zoom:50%;" />

### 2.4.3 BlackBorad Repository

#### 2.4.3.1 概述

问题：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614203900625.png" alt="image-20220614203900625" style="zoom:50%;" />

问题特征：

- **多种方法**可以解决问题，**找不到确定的解决思路**

- 求解的每个步骤都**有可能产生多个可能的解**，寻求最佳或者可接受的解。

- 需要多个领域的专门知识协作解决

eg：

- 自然语言处理
- 语音处理
- 模式识别
- 图像处理

如何求解此类问题？——黑板体系结构

- 一个大问题被分解成若干的子问题
- 每个子问题的解决需要不同的问题表达方式和求解模型，分别设计求解程序

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614205012421.png" alt="image-20220614205012421" style="zoom:50%;" />

#### 2.4.3.2 黑板模型

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614205535232.png" alt="image-20220614205535232" style="zoom:50%;" />

知识元：问题分解成几个部分，每个部分独立计算——**知识元对黑板进行修改，逐步找到问题的解**

黑板数据结构：全局数据库包含解域的全部状态

控制：完全由黑板的状态驱动，**黑板的状态的改变决定使用的特定知识**

**让知识元响应偶然事件**

##### （1）知识源

目标：

提供解决问题的知识，分别存放且相互独立

动作：

**只修改黑板**，知识源之间的通讯交互只通过黑板进行

负责：

知道什么时候能发挥作用：“条件-动作”形式

##### （2）中心数据结构

- 目标：

保存知识源所需要的数据，保存来自解空间的数据

- 组织：

解决问题中的状态数据，以层次的形式组织起来

各个知识源只通过黑板进行交互

##### （3）控制 Control

时刻监控黑板的状态，对黑板上的当前信息进行判断和评价

满足知识源执行条件时，知识源被控制器触发，并进行计算，结果写在黑板上

这种更新又导致其他的知识源参与计算，直到找到问题的解为止

目标：

让知识源响应偶然事件，了解各个知识源的能力，决策解决问题的步骤

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614210956573.png" alt="image-20220614210956573" style="zoom:50%;" />

#### 2.4.3.3 应用

信号处理、模式处理、模式识别

人工智能

![image-20220614211053584](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220614211053584.png)





## 2.5 Virtual Machine

- Interpreters：模拟不是硬件固有的功能
- Rule-Based System：解释器的特例

### 2.5.1 Interpreter

#### （1）应用问题

应用问题：应用的最佳的执行环境或者语言不能够得到直接的支持。
$$
引入中间层\begin{cases}应用不能直接支持\\环境不适合\\跨平台\end{cases}
$$

- 无法直接使用最合适的语言或者机器来执行解决方案
- 核心问题是定义解决方案的符号的应用程序，如脚本
- 有时以链的形式使用，在一系列阶段的过程中从需要的语言/机器翻译

上下文：

**桥接需要的机器和语言与执行环境已经支持的机器（虚拟的）和语言**

Solution：

- 系统模型：一个虚拟机

- 组件：一个状态机和三个存储
  $$
  \begin{cases}状态机：执行的引擎\\
  三个存储：\begin{cases}执行引擎的当前状态\\被解释的程序\\被解释的程序目前所处的状态\end{cases}\end{cases}
  $$

- 连接件：数据访问和过程调用
- 控制结构：通过引擎状态转移，输入确定选择翻译什么

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615093544857.png" alt="image-20220615093544857" style="zoom:50%;" />

#### （2）优缺点

- 优点

功能性 functionality ：可以模拟非本机功能

测试性 testing ：可以模拟灾难模式（安全攸关的应用）

灵活性 flexibility ：非常通用的工具

- 缺点

效率：比硬件慢，慢两个数量级；比编译器慢

测试：需要测试额外的中间层

#### （3）解释器的应用

- 解释型语言

VB, JS, HTML, Java字节码

- 通信协议
- 用户输入

### 2.5.2 规则系统

#### （1）问题

根据当前的状态，基于事实，判断我需要的输出

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615100200622.png" alt="image-20220615100200622" style="zoom:50%;" />

知识库：要被执行的代码

规则引擎：翻译引擎

规则/数据选择：解释器的状态

工作存储：当前代码的状态

#### （2）理解

对于一个架构，可以从不同的架构去理解

eg：hearsay

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615103944597.png" alt="image-20220615103944597" style="zoom:50%;" />

## 2.6 Independent Component

### 2.6.1 进程间通信

#### 2.6.1.1 背景概述

分布式环境，出现多进程多线程，节点之间相互独立。

已经广泛应用的：

- 操作系统
- 分布式应用

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615105611923.png" alt="image-20220615105611923" style="zoom:50%;" />

#### 2.6.1.2 进程间通信

##### （1）应用问题

- 包含一系列**不同的，在很大程度上独立的计算的应用程序**，这些计算的执行应该独立进行。
- 计算涉及**数据的协调**或者**对离散的时间点**的控制。因此，系统的正确性需要注意**消息的路由和同步**。

eg：网络游戏的例子

##### （2）背景

通信策略的选择通常由可用操作系统提供的通信支持决定。

##### （3）解决方案solution

- 系统模型：独立交流的进程
- 组件：向明确接收方发送和接收消息的进程
- 连接件：离散的消息==（没有共享数据 important）==
- 控制结构：每个线程都有自己的线程控制

##### （4）重要变化

确保消息一定收到。

##### （5）设计需要考虑的问题

图的拓扑结构、失败模型、性能

## 2.7 事件系统

![image-20220615114022791](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615114022791.png)

### 2.7.1 隐式调用和显式调用

显式调用：明确知道消息的接收者

隐式调用：中间层调用

#### 2.7.1.1 隐式调用 implicit invocation

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615115959832.png" alt="image-20220615115959832" style="zoom:50%;" />

##### （1）问题

适用于：

- **松耦合的组件集合应用程序**，每个组件执行某些操作，并可能在流程中启用其他操作，通常是**响应式系统**。
- 对于**必须动态进行，可重新配置的应用程序，该模式是有用的**。

上下文：

- 需要**事件处理程序**，接收对事件的兴趣，并且在**事件引发时通知组件**。注册行为

##### （2）特点

- 构件**不直接调用一个过程**，而是触发广播或者多个事件
- **不能假定构件的处理顺序**，<u>甚至不知道哪些构件会被调用</u>
- 各构件之间彼**此无连接关系，相互独立存在**

![image-20220615132637996](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615132637996.png)

##### （3）解决方案 solution

系统模型：独立交互式进程

组件：在不知道接收方的情况下发送接收消息的进程

连接件：自动调用已对事件感兴趣的进程

控制：分散，单个组件不知道消息的接收方

##### （4）应用

- Debugger-编辑器和变量监视器登记Debugger断点事件
- 在编程环境中集成各种工具
- 数据库管理系统中确保数据的一致性
- 用户管理界面系统中管理数据





