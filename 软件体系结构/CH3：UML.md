# CH3：软件体系结构建模和文档化

## 1.UML

### 1.1 Use Case

用例图：**理解系统功能需求**的宝贵工具

用于显示若干的角色与系统提供用例之间的关系，用例是系统的功能

是系统的**外部视图**，没有与系统内部的类的交互

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615143233230.png" alt="image-20220615143233230" style="zoom:50%;" />

### 1.2 Class Diagram

类图：表示类class和类class之间的联系，对**系统静态结构**的描述

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615143618256.png" alt="image-20220615143618256" style="zoom:50%;" />

### 1.3 Object Diagram

对象图：是系统在某一个时间点的对象的快照，表示的是实例instance而不是类，所以也叫Instance diagram

对象图展示对象之间的连接关系，**显示连接在一起的对象的示例**

![image-20220615143839305](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615143839305.png)

### 1.4 Package Diagram

包图表示的是更高层次的单元：

使用包图可以将相关元素纳入一个系统

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615143948091.png" alt="image-20220615143948091" style="zoom:50%;" />

### 1.5 Sequence Diagram

多个对象之间随着时间推移的交互关系

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615144346129.png" alt="image-20220615144346129" style="zoom:50%;" />

序列图便于展示对象之间的协作。

- 如果想要看一个use case中的多个对象的行为，那么使用**序列图**
- 如果想看一个对象在多个use case中的行为，那么使用**状态图**
- 如果想看多个对象在多个线程中的行为，那么使用**活动图**。

### 1.6 Communication Diagram

==协作图（通信图）==：描述对象之间的动态协作

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615144755184.png" alt="image-20220615144755184" style="zoom:50%;" />

协作图（通信图）是一种交互图，强调的是**发送和接收消息的对象之间的组织结构**。一 个协作图显示了一系列的对象及对象之间的联系以及对象间发送和接收的消息。

- 强调时间：序列图
- 强调上下级关系：协作图

$$
交互图\begin{cases}协作图、通信图：强调上下级关系\\序列图：强调时间推移\end{cases}
$$

### 1.7 State Diagram

描述类的**对象所有可能的状态**以及**事件发生时的状态的转移条件**。通常**状态图是对类图的补充**。

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615145303126.png" alt="image-20220615145303126" style="zoom:50%;" />

### 1.8 Activity Diagram

满足用例要求所**要进行的活动**以及**活动间的约束关系**

重点：支持**并行行为parallel**

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615151148930.png" alt="image-20220615151148930" style="zoom:50%;" />

### 1.9 Component Diagram

组件图：描述**代码构件**的**物理结构**和各个**构件之间的依赖关系**

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615151414060.png" alt="image-20220615151414060" style="zoom:50%;" />

### 1.10 Deployment Diagram

软硬件的物理结构，硬件节点，软件视图。

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615151707964.png" alt="image-20220615151707964" style="zoom:50%;" />

### 1.11 Composite Structures

复合结构：分层地将**类分解为内部结构**

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615152418273.png" alt="image-20220615152418273" style="zoom:50%;" />

### 1.12 Interaction Diagram

交互概述图：**活动图和序列图的结合**

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615152755144.png" alt="image-20220615152755144" style="zoom:50%;" />

### 1.13 Timing Diagram

时序图：交互图的另一种形式，一般针对一个或者多个对象描述。

![image-20220615153103421](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615153103421.png)

## 2. “4+1”视图

逻辑视图：支持行为要求。关键抽象，对象或者对象类

- 类图
- 对象图
- 状态图
- 协作图

过程视图：解决并发和分发

- 活动图

开发视图：组织软件

- 包图
- 组件图

物理视图：其他元素映射到处理和通信结点

- 部署图

用例视图（场景）：其他视图映射到重点的用例，帮助用户理解系统功能

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220615160837510.png" alt="image-20220615160837510" style="zoom: 67%;" />
