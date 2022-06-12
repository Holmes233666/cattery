# <center>CH2：Achitecture Style</center>

## 2.1 架构风格的定义

不存在统一的定义：

架构风格 Achitecture Style  = 

- 组件 component
- 连接件Connector vocabulary
- 约束Semantic Constraint 

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

  ![image-20220610194442613](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610194442613.png)

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

