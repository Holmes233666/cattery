# **PATHWAYS: ASYNCHRONOUS DISTRIBUTED DATAFLOW FOR ML** 

# 针对机器学习的异步分布式数据流阅读笔记

## 摘要

​      本篇论文介绍了用于加速器的新的大规模的编排层orchestration layer的设计——Pathways。论文新型异步分布式数据流关键在两点：

- 异步：通信延迟和调度延迟的处理
- 分布式：突破原本JAX只能在2048个TPU上运行的限制——针对模型规模变大，硬件异构的处理

## 一、论文研究背景、动机和主要贡献

### 1.1 研究背景

​      近10年里的趋势是机器学习（Machine Learning, ML）的算法和系统与硬件一起演进（co-evolution）这种co-evolution有一定的隐患：系统过于针对当前的任务，而不能很好地适应未来的需求。

​      PATHWAYS：针对的是分布式机器学习，具有的相关特性会被未来的任务所需要。

​      目前的分布式ML系统的现状：当今大多数最先进的ML工作负载都使用MPI的**“单程序多数据”（SPMD）**模型。

>  所谓的SPMD可以类比2014年OSDI的另一篇论文参数服务器中的例子——多机在完成梯度的计算后交换信息，做一次数据的同步。

### 1.2 研究动机

​      未来ML需要一些特定的功能，哪些特性是未来ML所具有的？

- **模型较大**：对于大语言模型模型在一个加速器上是不够的，纯粹的数据并行是不够的

  开始考虑流水线（pipeline）而不是纯的数据并行（文中以MoE为对象介绍，他是一种MPI）

- **存在计算稀疏性**：使用细粒度的控制流和夸加速器的异构计算

- **硬件异构**：就硬件上的变化来说，机器学习集群变得越来越异构

​      针对硬件的异构性，论文中提出了一个“岛”(island)的概念，<u>**“岛”指的是一组连接在一起的加速器**</u>。单机的加速器之间连接较为紧密，而多机之间的加速器连接并不紧密。如果需要维持多机之间紧密的联系需要耗费较大的带宽，这通常是比较浪费的：用户需要试图保持所有加速器持续繁忙。

​      这种异构性的存在，进一步推动研究人员转向**“多程序多数据”(MPMD)**计算，通过**将整体计算的子部分映射到更容易获得的小型加速器岛的集合，从而实现更大的灵活性**。为了提高利用率，一些ML硬件资源管理人员以细粒度的方式在工作负载之间复用硬件，从而支持工作负载弹性，并提高容错能力。

​      最后，研究人员开始标准化一套基础模型，这些模型是在大数据上大规模训练的，可以适应多种下游任务。对此类模型的训练和推断提供了通过跨许多任务多路复用资源来提高集群利用率的机会，并有效地在它们之间共享状态。

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/10/478db6744763d75e1cb68d00f4ca3075-image-20220810195437069-cf4f2a.png" alt="image-20220810195437069" style="zoom:50%;" />

> 结合Jeff Dean去年的博客[Pathways: A next-generation AI architecture (blog.google)](https://blog.google/technology/ai/introducing-pathways-next-generation-ai-architecture/)（上图），能够较好地理解这个**基础模型**的概念。

### 1.3 研究贡献

​      Pathways系统与最先进的ML系统状态的功能和性能匹配，支持未来ML工作负载所需要的功能。Pathways使用的体系结构为client-server体系结构。这个体系结构使得Pathways运行时能够代表许多客户在系统管理的计算岛上执行程序。

创新性：

- Pathways旨在透明有效执行跨越TPU的"POD"程序的系统
- 采用数据流执行（DataFlow Execution）模型来扩展到数千个加速器

​      PATHWAYS使非SPMD计算变得容易，并启用了CENLALALIZED资源管理和虚拟化，改善了加速器的利用率。

## 二、设计动机

### 2.1 现有ML系统的设计

​      现有的ML系统的设计难以支持大型的，稀疏的，或者不规则的模型。

#### 2.1.1 Multi-controller System

​      用于训练最先进的**SPMD模型**的分布式ML系统通常采用**多控制器（multi-controller）体系结构**，其中相同的客户机可执行文件直接在系统中的所有主机上运行，在程序执行期间独占这些主机上的资源
$$
训练最先进的SPMD的分布式ML系统例子\begin{cases}MPI\\Pytorth\\
JAX\\
TensorFlow\\
......\end{cases}
$$
​      这种架构的优点是计算的低延迟：因为用户代码的相同副本运行在每个加速器的主机上，并且分派补丁只涉及到通过相对快速的PCIe链路进行通信。所有其他跨主机的通信只通过使用专用互连的集合发生，如NVLink，而不使用主机内存。

​      缺点：

- 这种架构与**使用流水线**或者具有**计算稀疏性**的线代ML工作负载不匹配。
- multi-controller系统中，任何超过标准集合的通信需要用户实现他们自己的协调原语
- multi-controller通常假定独占硬件资源，使得构建高效的集群范围ML设计变得复杂

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/10/c295e2c58b0a26a2747f9202245cd843-image-20220810225219876-1e898a.png" alt="image-20220810225219876" style="zoom: 50%;" />

#### 2.1.2 Single-controller System

​      单控制器（single-controller）系统，如TensorFlow v1提供了一个包括优化图内控制流的非常通用的分布式数据流模型。

​      TensorFlow (TF) Python客户端构建一个**计算图**，并将其移交给**协调运行时**，协调运行时将图划分为每个worker的子图，并将子图的执行委托给worker上的本地运行时。worker之间的协调是通过数据和控制边缘在数据中心网络(DCN)上传递消息来执行的。

虽然单控制器设计提供了灵活的编程模型和虚拟化资源，但它提出了**实现的挑战**：

##### 2.1.2.1 TF1 SPMD

- 通信更慢：单控制系统系统中的客户端更远，分派延迟包括DCN上的通信，通常比PCIe慢一个数量级
- controller为了更好的分派子图，可能会预先编译优化图，这样的优化会使**debug变得困难**
- 在涉及许多跨主机传输的程序中，例如使用大量的阶段，这些**调度延迟累积**，导致低效的加速器利用率。

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/11/5fc7d4ce72ccb27c40ac820ae095f5c1-image-20220811111621366-f9ac99.png" alt="image-20220811111621366" style="zoom:50%;" />

##### 2.1.2.2 TF1 non-SPMD

- 为了支持带有SPMD的子计算的MPMD程序并发执行，运行时需要有某种机制来支持加速器计算的**分组调度**，否则可能会出现死锁。
- 计算结点变多，边变多，中央控制器指令收发成为一种瓶颈——带宽变低，通信量增多



<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/11/5164b5e21dab065833633c93189f3fc1-image-20220811111354323-8ceb0f.png" alt="image-20220811111354323" style="zoom:50%;" />

### 2.2 PATHWAYS目标

​      而针对Pathways来说，Pathways并不是打算去解决TF1在分组调度、debug困难这样的**易用性**的问题，而是认为TF1这样的设计是合理的，即在系统上，把这样有一个中央controller，能够把任务分配到各个结点上。同时这个概念称为**dataflow**。——论文题目dataflow的由来

>  dataflow：
>
> 计算表示成一个计算的图，一个结点是一个计算，箭头表示依赖的关系。
>
> 系统的目标：给一个计算图，有向无环图，如何把他映射到硬件上，更好地调度执行。

​      **TF1受限的问题：TF1考虑的是比较小的图，是在几百个同构的加速器上执行。**

​      TF1 执行大图的问题：

​      TF1中图的结点和边变多，结点进行的运算较少，中央控制器进行指令收发成为了一个瓶颈——带宽变低，通信量增多了 not good

​      现在模型更大，异构，如何修改设计？

- 仍采用单控制器（single controller）：利用计算稀疏性和异构性，促进资源共享和虚拟化
- **异步调度**来匹配**多控制器系统的性能**
- 支持集中资源管理和调度，使分片数据流系统高效协调

## 三、 PATHWAYS 编程模型

>  Google深度学习框架前置知识：
>
>  第一代深度学习框架disBlief
>
>  第二代深度学习框架TensorFlow v1：调试不方便，性能优化方便；运行得到的是一张图，而不是中间的结果；运行完后得到的完整的执行图
>
>  - 好处：后端可以进行性能优化；计算图与Python无关，灵活性提高
>
>  - 缺点：调试变得困难
>
>  TensorFlow v2：与TensorFlow v1不同，几乎是另一套框架
>
>  谷歌之后开始研究TPU，为TPU开发了**编译器XLA**；XLA之后也支持了GPU和TPU，是一个统一的后端
>
>  JAX：XLA的前端，设计理念是提供一个与numpy类似的前端——可以按行执行，实际上采用的是一种**延后执行**的方法；
>
>  - 从用户看：和Python按行执行似乎没有什么区别
>  - 从后端：拿到很多图，只是图大小变小很多
>
>  JAX提供自动求导等功能

### 3.1 JAX和TensorFlow对PATHWAYS支持

#### 3.1.1 JAX评估的优点

​      目前进度：实现了使用TensorFlow和JAX编写的源程序对PATHWAYS的支持

​      JAX优点：可以显式使用Python的装饰器（decorator），指示那些原本应该被编译成SPMD的python代码。

```python
@pw.program # Program tracing (optional)
    def f(v):
        x = a(v)
        y = b(x)
        z = a(c(x))
        return (y, z)
```

​      这些XLA计算的特点是**已知输入**和**输出类型和形状**，可以提前估计计算的资源需求。称为**“编译函数”**（compiled function），并且每个这样的函数都映射到PATHWAYS程序中的单个（分片）计算节点。

#### 3.1.2 PATHWAYS+JAX实现TPU数量扩展

- multi-controller下的JAX

  使用XLA集合传输所有数据，JAX无法扩展到单个TPU pod外——无法适应当前ML发展的需求

- 使用PATHWAYS作为JAX替代后端
  - JAX代码不需要修改
  - PATHWAYS可以通过ICI和DCN进行通信，可以使**JAX程序扩展到多个TPU pods，**可访问数千个TPU核心

#### 3.1.3 编程模型

​      下面是Python用户代码应用于在多个TPU上的PATHWAYS示例：

```python
def get_devices(n):
    """Allocates `n`virtual TPU devices on an island."""
    device_set = pw.make_virtual_device_set()
    return device_set.add_slice(tpu_devices=n).tpus

    a = jax.pmap(lambda x: x * 2., devices=get_devices(2))		
    // get_devices(2)返回两个虚拟TPU，运行时再根据PATHWAYS做具体的映射与调度
    b = jax.pmap(lambda x: x + 1., devices=get_devices(2))
    c = jax.pmap(lambda x: x / 2., devices=get_devices(2))

    @pw.program # Program tracing (optional)
    def f(v):
        x = a(v)
        y = b(x)
        z = a(c(x))
        return (y, z)
    print(f(numpy.array([1., 2.])))
    # output: (array([3., 5.]), array([2., 4.]))
```

​      简要分析下程序：`a()`,`b()`,`c()`三个函数首先声明在什么硬件上运行，这里使用的是`JAX`的`pmap()`函数，`pmap()` 实现并行的映射，将函数映射到两个TPU设备上。`get_devices()`是PATHWAYS的函数，PATHWAYS用户可以通过`get_devices()`请求两个虚拟的TPU，待运行时再做具体的映射物理TPU和优化。系统将自动处理所有相关计算之间的数据移动和重分片。

##### 3.1.3.1 无`Program tracing`下的PATHWAYS（默认情况）

​      假如程序中line 11`@pw.program # Program tracing (optional)`不存在，那么，即如下程序：

```python
def get_devices(n):
    """Allocates `n`virtual TPU devices on an island."""
    device_set = pw.make_virtual_device_set()
    return device_set.add_slice(tpu_devices=n).tpus

    a = jax.pmap(lambda x: x * 2., devices=get_devices(2))		
    b = jax.pmap(lambda x: x + 1., devices=get_devices(2))
    c = jax.pmap(lambda x: x / 2., devices=get_devices(2))

    def f(v):
        x = a(v)
        y = b(x)
        z = a(c(x))
        return (y, z)
    print(f(numpy.array([1., 2.])))
    # output: (array([3., 5.]), array([2., 4.]))
```

​      在上述程序中，`f()`函数会触发4次编译，4次调度：每个编译后的函数`a()`，`b()`或`c()`都会转换为一个独立的，仅包含一个分片计算的PATHWAYS函数。在调度时，编译好的函数发送到具体的物理`TPU`上运行，并返回结果。鉴于`a()`，`b()`，`c()`三个函数较为简单，若是采用编译为单个分片的PATHWAYS函数再发送的策略，如下图，会在RPC上耗费较大的代价。

![image-20220815120556682](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/15/6b134fe544e383a997c1ba996f3f5cf1-image-20220815120556682-915753.png)

因此，PATHWAYS使用了`Program tracing`。

##### 3.1.3.2 含`Program tracing`下的PATHWAYS

​      在上述代码中增加`@pw.program`。

>  `@`表示python中的装饰器

​      增加后，PATHWAYS会把`f()`编译称为一个单独的函数，只需要编译一次，调用时将整个完整的函数发送，以一个更大的任务形式去执行。

### 3.2 PATHWAYS支持的编程模型成果

- 硬件的抽象与映射

  `devices=get_devices()`

- 额外的接口函数，打包更大任务去执行

  `@pw.program`

​      PATHWAYS其实不是一个框架，而是一个后端，通过这样编程接口的方式嵌入到前端上。这里的前端是JAX，但也可以用到TensorFlow上。

## 四、PATHWAYS 系统架构

​      PATHWAYS广泛建立在以前的ML系统的基础上，基础包括：

- XLA表示和执行TPU运算
- TensorFlow图来表示和执行分布式CPU计算
- Python编程框架，JAX等

​      PATHWAYS能更关注一些新颖的协调方面的任务，使用更少的代码优化ML模型。

### 4.1 资源管理器：实现虚拟设备映射

​      一组加速器组成了PATHWAYS的后端，这些加速器被分组为紧密耦合的孤岛（tightly-coupled island），孤岛之间通过DCN相互连接。**岛内的TPU连接是高带宽的**，而**岛与岛之间**的连接需要使用共享的数据中心网络，**速度较慢**。另外一个概念为PODS，一个PODS最多有2048个TPU核，在模型较小的情况下不需要跨POD计算，而在模型较大时，单个的POD不能支持较大模型。

​      PATHWAYS即是为了解决这样的问题提出的，其目标则是针对在一个POD无法支持较大模型的情况下，使用多个POD，类比单机来说，即是一种**多机多卡**的训练形式。

​      PATHWAYS资源管理器（Resource Manager）负责集中管理岛屿上的所有设备，可以根据用户的需求进行加速器的分配。将虚拟的TPU映射到物理的TPU上。

- 目前：一对一的映射，按需分配
- 未来：支持动态的结点加与减

![image-20220815143900379](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/15/c1a1feb2c5606a94f3b07dbf268e836b-image-20220815143900379-2e2684.png)

### 4.2 客户端 Client：任务执行的过程

​      论文这部分描述了用户任务执行的细节过程，即PATHWAYS是如何将用户的任务编译并运行的：

step1：用户运行traced program

step2：调用PATHWAYS客户端程序库，未之前没有运行过的计算分配物理设备，并在资源管理器中注册，触发服务器后台编译计算

step3：JAX将用户程序（任务）编译成为MILR

> MILR：可以认为是XLA的一个上层的语言，用于支持动态的数据流与动态形状的输入和输出，是一个高层的IR

step4：IR通过一系列标准的编译器传递逐渐“降低”，最终输出一个**包含物理设备位置的低级表示**。

​      在整个过程中，需要考虑一些分布式通讯的问题，会映射到一些网络通讯架构。          PATHWAYS使用RDMA实现对远端主机内存访问，由于使用了RDMA，所以主机上的一块内存可能虽然不被本机使用，但是远端机器可能会使用。以前的single-controller和现在的PATHWAYS针对于这方面的处理有所不同：

- older single-controller：协调数千个数据缓冲区，比如说每个小块维护一个counter，维护数千个counter去计数引用。
- PATHWAYS：使用shared buffer抽象一个可以分布在多个设备上的逻辑缓冲区，通过shared buffer分摊引用计数的成本，帮助客户扩展。

### 4.3 Coordination implementation：数据的收发

​      这部分说明的是如何在数据中心的网络上高效地收发数据。

​      数据中心的网络是有一定的结构的，如何选择发送的路径，发送的方式（同时发？）等是需要动态调整的。PATHWAYS使用PLAQUE进行优化，将低层次的IR表示为plaque的表示，可以满足具体的需要。

![image-20220815152330337](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/15/7b61140b00e3c9fad5cacf5b83a74d83-image-20220815152330337-38f0fa.png)



​      PLAQUE优化支持稀疏通信，这是PATHWAYS希望启用的重要功能之一。同时也可以使用Ray做以优化。

### 4.4 Gang-Scheduled dynamic dispatch：死锁的预防

​      TPU只支持单线程，多个TPU之间可能发生死锁。解决方法：

- 引入多线程
- Gang-Schedule

​      所谓的Gang-Scheduled指的是在每个岛上都有一个集中的调度器，调度器负责将任务进行合理的排序，避免死锁的发生。目前是按照FIFO的顺序工作。

### 4.5 Parallel asynchronous dispatch 并行异步分发

​      当在加速器上运行时，系统可以用异步API将计算与协调重叠。

#### 4.5.1 顺序分发 Sequencial dispatch

​      如下图，其中的正方形对应三个节点A、B和C，它们运行在主机A、B和C所连接的加速器上。所有节点计算都是常规的**编译函数**。

![image-20220815164651806](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/15/d4c9d7ae55dc57000a4fc06b4a245b38-image-20220815164651806-f01aed.png)

> XLA在编译一段代码时必须知道这段代码的输入和输出形状，知道形状后可以在加速器上创建内存。

​      上图描述了一个这样的顺序处理过程：

- 主机A排队等待节点A，接收到A输出的`future`，并将`future`发送给主机B。（棕色箭头）
- 主机B分配B的输入，将输入的缓冲区地址发送给主机A，完成启动节点B功能的大部分准备工作。（红棕色箭头）
- 当节点A完成时，它的输出通过加速器互连直接**发送到节点B的输入缓冲区**，然后主机B启动节点B。

​      一个节点完成和下一个节点启动之间的延迟可以比数据传输时间多一点。从图中我们可以看出来，这样的顺序分配的方式存在大量的stall，效率较低。

![image-20220815171155097](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/15/237f961b46371902fab3bc600c4fe5fb-image-20220815171155097-a526ab.png)

​      即这样的处理回到了之前提出的存在的问题：当程序的计算图很大，结点之间的依赖很复杂，同时每个结点的计算量不是很大时，那么很大一部分的时间都消耗在stall上了。

​      并且如果编译的函数都是规则的，那么实际上甚至可以在前一个节点的计算进入队列之前计算后继节点的输入形状。

#### 4.5.2 并行分发 Parallel dispatch

​      对于函数是规则的情况下是可以进行优化的：客户端编译之后知道要发送的形状，收到的形状。在通知结点A时会同时通知结点B和结点C它们将会分别从结点A和结点B收到的输入的形状。于是等待时间得到了缩减。

![image-20220815172717225](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/15/06b14f2aad174d503f8c60468cc3369f-image-20220815172717225-f11da6.png)

​      由于工作只能在函数是规则（regular）的情况下进行并行调度，所以PATHWAYS将并行调度视为一种优化，并在一个节点的资源需求直到前一个计算完成后才知道的情况下(例如，由于依赖数据的控制流)，退回到传统模型。

### 4.6 数据管理 Data management

​      每个主机管理一个分片的对象存储，类似于Ray的对象存储，但扩展到跟踪每个分片上的加速器HBM中持有的缓冲区。

## 五、评估

### 5.1 实验

​      使用的任务较小，主要还是探讨不同的决策带来的结果。

### 5.2 一些讨论

- 是否能够使用在GPU上？

  GPU设计理念与TPU不同，GPU的编译上不会优化过多。文章提到的优化大部分已经在GPU上做到了。

- 资源的管理

  何如去做到动态的资源的管理，而整个系统主要关心的是大模型的运行，动态资源管理作为未来工作。

- 稀疏模型

  作为未来的工作

### 5.3 评价

​      帮助JAX如何突破在一个TPU Pods上进行计算的局限：不仅可以在2048个核上训练。
