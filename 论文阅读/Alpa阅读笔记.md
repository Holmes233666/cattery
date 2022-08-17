# **Alpa: Automating Inter- and Intra-Operator Parallelism for Distributed Deep Learning** 

# **Alpa**阅读笔记

## 一、论文的研究背景、动机和主要贡献

### 背景：大模型和分布式训练

​	  深度学习最近的进展直接导致了模型大小的变化，如：GPD3亿万级别的参数，同时在更大数据集上进行修改以支持兼容性。最新的模型上参数的数量以指数的形式增长。

​	  训练这样大的模型是具有挑战性的。因为模型的参数众多，受限于内存大小，无法在一个单一的加速器上进行训练。同时，这会花费几百年去在一个单独的加速器上训练这样的模型。因此分布式训练是必须的。

​	  迄今为止，分布式训练并不是那么容易，为了训练特定的模型人们必须去建立特定的系统。有一些比较流行的技术，比如数据并行（data parallel），张量并行（tensor parallel）和流水线并行（pipeline parallel）。

> [[细读经典\]三种并行知识](https://zhuanlan.zhihu.com/p/388830967)

​	  三种并行之间有着不同的权衡（trade off），如果要训练一个较大的模型，你必须去找到一个方法去综合运用这三种并行，从而你可以达到一个好的训练吞吐量。以下是专家设计策略的两个例子：

- Megatron-LM策略：在transformer层并行化self-attention模型

  ​	  对于大型的transformer模型，权重的数量非常大，因此不能使用数据并行。所以需要对权重进行分片。在这个模型里面，第一个权重矩阵已经被按列分片，第二个矩阵已经被按行分片，从而减少通信代价。

  <img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/b82852098937a8b6c3cc58b296c5fb8e-image-20220816235610007-36c1a9.png" alt="image-20220816235610007" style="zoom:50%;" />

- GShard Mixture-of-Expert

  ​	  专家层（红色部分）以专家维分片，非专家层以批处理维度分片做数据并行

  <img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/3b074cadae8f47e1af7f6e75312979b1-image-20220816235630054-eee601.png" alt="image-20220816235630054" style="zoom:50%;" />

​	  为了应用这样的并行策略，模型开发者不得不去重写他们的模型定义，特定化分片策略，并且插入必要的通信原语，比如这里的`all-to-all`。这使得开发一个新的模型或者寻找异质模型变得困难。

​	  在我们的论文中，我们的目标是统一所有的这样的并行化策略，并且建立一个编译器去自动产生最优的综合的并行化策略。所以，首先我们总结了现有的并行化策略，把他们分成两种：

- **Type1：Intra-operator Parallelism**

  <img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/bfea1a994bbf09572f2673cc2b167bf2-image-20220816111818037-a85266.png" alt="image-20220816111818037" style="zoom:80%;" />

  ​      我们利用了单个操作中固有的并行性，所以我们称它为`Intra-operator parallelism`

- **Type2：Inter-operator Parallelism**

  <img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/eadda3a3d858209c7c286c11966ff391-image-20220816111802128-e93dd4.png" alt="image-20220816111802128" style="zoom:80%;" />

  ​      我们利用了在整个图中的更高层次的并行，但由于数据依赖，设备2不得不去等待设备1，等待它的输入数据。所以一些设备可能在一些时间内是空闲的。去解决这样的问题，典型的流水线技术被应用，比如我们可以发送多个微小的批到流水线中去实现并行化。

​      有了这两种定义，我们可以把传统的并行技术全部归类到这两种分类中，下图展示了一些比较流行的训练两层MLP的并行化技术：

- `Intra-operator Parallism`例子：

![image-20220816114141419](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/43c8d4416b7ebcaf633ebd318372d1ef-image-20220816114141419-20fbf4.png)

- `Inter-operator Parallism`例子：

![image-20220816125625069](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/56b80f0c31a686e6d6b460778a7dc918-image-20220816125625069-fc5aec.png)

​      特别注意的是，在Device Placement的图中，如果不存在多个并行的设备分支，那么会造成时间的空闲；而在Pipeline Parallelism中通过将输入的数据分割为若干的微小的批次，从而解决了这个问题。

​      在这两种分类之间有着响应的权衡（trade-off）：

- `Intra-operator Parallism`需要大量的通信，比如`all reduce`和`all-to-all`，故高带宽的设备连接是必要的。
- `Inter-operator Parallism`通常需要的是点到点的通信，通信发生在子图的边界内，需要更少的通信。但是为了解决设备的空闲，仔细的调度和分片是需要的。

### Prior Auto-Parallelization Work

​      有效的训练一个大模型通常需要上述的很多技术的综合。如果依赖于人工的设计策略，这需要很大的工程上的努力。所以研究者倾向于去寻找一种基于自动并行。在以前的工作中，有以下几种自动并行化系统：

- Flexflow/Tofu
- Pipedream/Dapple

​      但是他们都有很多的限制，并且不能训练最先进的模型：

- 搜索空间有限

  由于这些模型仅考虑了有限的并行化技术，并没有对所有的并行化技术予以支持。

- 搜索算法限制

  搜索算法的搜索是基于对模型的随机搜索或者强假设的。

### Our Approach

​      本篇的论文解决方案采取了一个不同的并行化视角，将上述的两种并行化技术分类作为一个两层层次化空间。这个层次化的空间自然地与层次化常见的GPU集群结构映射起来，在GPU内部结点中是高带宽的NVLink链接，而在结点之间是较慢的连接，比如以太网。本篇论文建立了一个两层的层次化搜索空间，然后设计响应的算法去获得每个层次的最优解。

![image-20220816154909651](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/60da5dacdd31366edbe4f6874485fcea-image-20220816154909651-fac5e3.png)

​      接下来是方法的详细说明。

## 二、Automating Intra and Inter Operator Parallelism

​      自动并行化（Auto Parallelism）意味着用户不用特意地去修改他们的模型，论文中提到的模型将会将用户的模型从一个单设备的程序转变为一个多设备的程序。Alpa遵循编译器的架构，我们从runtime部分开始说明：

### 两种并行化的runtime支持

​      现在的GPU集群中，GPU被典型的组织成为一个两层的拓扑结构，故在结点内有高速的链接，但在结点间有着相对慢速的连接。论文使用了一个二维设备网络去指代这个拓扑结构。以下图为例，在这个设备网络中，我们有2个Worker，每个设备中有4个加速器，并且我们可以假设，在Worker内部，加速器结点之间通信速度较快，而在Worker之间，通信速度较慢。在组织时，设备之间采用Inter-op Parallesim模式，而在设备内部采用Intra-op Parallelism的形式。

![image-20220816163148345](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/b52f5b119f64384ace0e9b089ebfc24e-image-20220816163148345-057874.png)

### 编译器端

​      对于编译器端，输入是一个计算图和一个特定的设备集群。

​      我们首先运行一个`Inter-op Pass`将计算图分片为多个阶段，对于每个阶段我们对其分配一个设备网，故一个设备网负责一个阶段（stage）的计算。对于每个阶段之间，我们运行一个`Intra-op Pass`去分片所有的操作，即使是在设备网的多个GPU集群之间。

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/e4238f646fdb6e952a4b0d2155c08c8f-image-20220816165347791-7bcf57.png" alt="image-20220816165347791" style="zoom:50%;" />

​      在运行时编排（runtime orchestration）阶段，编译器使用流水线的指令去调度这些阶段，并且所有的指令被静态编译为网络可执行（mesh executable），这个网络可执行被发送到相应的设备网络，然后我们在所有的设备上去执行相应的mesh executable。

故在整个阶段中，最重要的是`Inter-op Pass`和`Intra-op Pass`两个部分：

- `Inter-op Pass`可以通过**动态规划**为`Inter-op Parallism`提供一个局部的最优解；
- `Intra-op Pass`可以通过**整数线性规划**为`Intra-op Parallism`提供一个局部最优解

​      下面对于这两个重要的阶段进行详细的分析：

#### Intra-op Pass

​      `Intra-op Pass`使用的是SPMD风格，这意味着将会对每个操作进行分片，即是是在设备网络中的所有GPU中。SPMD风格简化了许多东西，但是由于其依然可以覆盖很多传统的技术，比如数据并行（Data Prallelism）、操作并行（Operator Parallelism）和零优化器（Zero Optimizer），SPMD是足够强大的。对于`Intra-op Pass`我们的目标是对于每个操作找到一个分片策略。以下图为例，这是一个两层MLP的向前计算图。

![image-20220816171138920](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/1290cdb155db778d4727a55bfbc55caf-image-20220816171138920-dfcf09.png)

​      对于张量的权重，它可以被按行分片、按列分片、全复制或者是部分复制，故每个结点的张量都有一个包含多种可能的分片策略的分片策略集（partition strategy set）。比如矩阵乘法有很多种并行计算的方法去解决该问题。不同的算法对输入层有着不同的要求，如某个算法可能要求权重是按列分片的，另一个算法可能要求权重是按行分片的。同时不同的算法也有着不同的输出布局。如果输入不满足算法的输入要求，那么我们需要在边缘做一个**布局转化**，这会造成相应的通信代价。

​      总之，去执行计算图，总时间代价公式如下：
$$
\text{Total time cost = node cost(compute cost) + edge-cost(communication cost)}
$$
​      我们的目标即去对每个操作（operator）选择一个分片的策略，并且最小化总时间代价（Total time cost）。下面给出一个更加具体的例子，分析不同结点可能的分片策略和边缘代价：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/27cee032eed6cd86a9785d25452ed1dd-image-20220816190519617-b44f10.png" alt="image-20220816190519617" style="zoom:50%;" />

​      输入矩阵和权重矩阵可以是完整副本、部分副本、列分片或者行分片的形式。矩阵乘法需要三层的循环，我们可以将第一层循环或者第二次循环或者这些循环的结合做并行处理，这样会有多个不同的策略：

- Strategy 1：矩阵A为列分片，矩阵B为行分片，最终产生一个结果的完整副本；同时需要进行`all-reduce(c)`去累计部分结果。
- Strategy 2：矩阵A为行分片，矩阵B在两个设备上存有完整的副本，最终产生的结果不需要进行累计，分别存储在两个设备上。
- Strategy 3：矩阵A在两个设备上存有完整副本，矩阵B为列分片，最终产生的结果分别存储在两个设备上。

​      若布局不匹配，那么需要使用集中式通信原语进行布局转换：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/9617e89e9749cd01cad434a61f9c13df-image-20220816191849132-8ffff2.png" alt="image-20220816191849132" style="zoom:50%;" />

​      例如：若需得到张量并未列或行分片的副本时，需要使用`all-gather`原语去进行结果的合并；若进行列分片或者行分片的转换时，需要使用`all-to-all`原语转换；若将一个完整的副本进行分片，并不需要进行通信，在本地进行分片即可。

​      上述的优化问题可以使用整数线性规划求解，以下图所示的依赖关系为例：依赖关系只是在两个结点之间存在，这就产生了一个二次目标规划，将其线性化即可得到一个线性规划。

==整数规划？如何进行？==

![image-20220816194215006](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/4a467a2ea41d93e846acd168f43e06a9-image-20220816194215006-2c016a.png)

​      为建模此问题，需要枚举所有操作（operator）的所有可能的分片策略，并且计算在所有边上的策略对的通信代价，最后以最小化代价为目标，进行整数线性规划。

​      与之前的工作对比：

| Comparison   | Advantages                                                   |
| ------------ | ------------------------------------------------------------ |
| vs. Tofu     | Support general graphs. <br />Support 2-D device mesh        |
| vs. FlexFlow | Optimality guarantee. <br />Support additional partition strategies. |

> Note: 如果考虑通信计算重叠，由于存在复杂的依赖关系，将不再能建模成为一个整数线性规划的问题。实际中，可以先进行整数规划，得到一个较优的解后进行通信计算重叠优化。[通信计算重叠（communication computation overlapping）](https://fleet-x.readthedocs.io/en/latest/paddle_fleet_rst/collective/collective_performance/overlap.html)

#### Inter-op Pass

​      在这部分将会针对流水线并行进行阐述，目前流水线也是针对于大模型训练的较为有效的方式。`Inter-op Pass`的目的是分片计算图为多个子图，以下图为例，分片完整的计算图得到了4个子图。同时，还需将输入集群分片为4个子网，将一个阶段与一个子网匹配。最终使用Pipeline的形式将各个子网之间连接，实现`Inter-op Pass`。

![image-20220816203850209](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/ab10632076efd34d9ac3efaa5158c84a-image-20220816203850209-6b97a5.png)

​      `Inter-op Pass`的目标是最小化流水延迟。流水线延迟由两项组成，首先是预热阶段，后续是稳定阶段，稳定阶段由最长的流水段决定。

![image-20220816204358078](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/a43ef5c8e0722d0f80d2002bdc5e7578-image-20220816204358078-d51904.png)

​      减少流水线延迟是通过动态规划算法实现的，动态规划的输出将是一个计算图的划分和设备网分片策略的结果，即流水线的各个段。下面是动态规划的细节说明：
$$
Minimize\ Pipeline\ latency\quad T=\sum_{i}^St_i+(B-1)·\max_{1\leq j\leq S}\{t_j\}
$$
**Input：**

运行`intra-op Pass`，分析获得每个阶段的用时$t_i=t_{intra}((o_p,...,o_q), submesh(n,m))$。

**Constraint：**

（1）将同一阶段的向前传递和向后传递分配到相同的子网中。

​		基于这个约束，我们假设向后传递的部分是对称的，故只需优化图的向前传递部分。

（2）每个子网可以完全覆盖原始的完整网络。

​		原始的网络是经过分片的，需要仔细选择子网的形状。有些子网的形状可以保证我们		的集群分片策略是始终有效的，并且可以完整覆盖原始的网络。

**Solution：**

​      枚举$\max\{t_j\}$并且将其转换为一个传统的动态规划问题——2维背包问题。

**Complexity：**
$$
O(k^5NM(N+log(M)))
$$
​      由于时间复杂度中涉及到$k^5$这样的高阶项，故此动态规划的算法不能应用于大规模的图。为了解决这个问题，论文进行了一个预处理的过程——运行图集群算法，以集群相似的图为一个大的层次。故可减少图中的操作的数量。

​      以上的说明覆盖了我们去构件Alpa的所有核心的技术。

### Alpa 实现

​      实现时使用JAX作为前端，JAX的一些特点使得能够更简单地构建编译器。例如，可跟踪静态的包括向前传递和向后传递的计算图，优化器更新等，图的提供对于论文的优化是非常必要的。

#### 编程API

论文提供了一个`One-Line Auto-Parallelization`API，JAX程序如下：

```python
# One-line auto-parallelization:
# Put @parallelize decorator on top of the Jax functions
@parallelize
def train_step(state, batch):
    def loss_func(params):
        out = state.forward(params, batch["x"])
        return jax.numpy.mean((out - batch["y"]) ** 2)
grads = grad(loss_func)(state.params)
new_state = state.apply_gradient(grads)
return new_state
# A typical training loop
state = create_train_state()				# 模型的创建
for batch in data_loader:					# 从data loader中加载批
	state = train_step(state, batch)		# 梯度下降计算
```

​      原始的JAX只是在一个加速器上计算，Alpa突破了这一限制，可以在加速器集群上进行计算。

> Note: Alpa的并行化是基于Ray加载处理器和分布式集群，并且使用Ray去检测所有的GPU或者是集群中可用的GPU。

#### 实现细节

- Inter-op Pass

  基于JAX中间层JAX PR实现。由于在此处我们仍然需要一些机器学习的语义，例如向前传递，向后传递，梯度等。

- Intra-op Pass

  基于XLA HLO实现。在该部分，不对forward pass和backword pass进行区分。在一个通用的计算图上运行ILP算法。然后使用XLA将策略lower为可执行程序。

- Pipeline Excecution

  流水线在XLA中是不支持的，故实现时使用自设计的流水指令。

![image-20220816220944381](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/48f3983bfbff767a315fba1afebe9228-image-20220816220944381-9f466e.png)

​      对于运行时的体系架构，实现时使用XLA综合Ray。对于通信部分，使用Nico构建了一个通信库。

## 三、评估与总结

### 评估

Cluster：$8\times AWS\ p3.13$个结点。每个结点总共有8 V100 GPUS。

Models：GPT-3（高达39B），GShard MoE（高达70B），Wide-ResNet（高达13B）

Setting：Weak scaling on model sizes

>  意为可用GPU的数量越多，那么训练的模型越大；但是论文固定了批处理的大小，输入的数据大小是固定的。即，模型并行化的评估与通常的数据并行化的评估是有所不同的。

![image-20220816232809663](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/2022/08/16/a294f2265cca64a7d95352f342cd72b8-image-20220816232809663-050ce6.png)

- GPT-3

  GPT-3被很多的系统进行了相关的优化。Megatron-LM是一个针对GPT-3训练的系统，同时合并了`Intra`和`Inter`的并行。Alpa可以自动找到Megatron的所有的策略并且达到Megatron的性能。同时编译的时间也是可以接受的。

- MoE

  Megatron是针对于GPT-3的，故无法用其训练MoE。对于MoE目前最优且可行的GPU实现是由Deep Speed实现的。Deep Speed针对MoE做了`Intra`和`Inter`的并行，但是他的性能是无法与流水线并行引擎比拟的。

-  W-ResNet

  W-ResNet是一个异质模型，因为在卷积神经网络中，活化的速度变得更慢，同时权重的大小变得更大，对于不同的层次，权重也是不同的，所以针对W-ResNet去人工设计一个策略是比较困难的，目前也没有这样的可行的人工策略。但是Alpa仍可以应用在这种类型的模型上，同时达到一个不错的性能。

### 总结

- Alpa是一个针对自动分布式系统的新的系统设计
- Alpa构建了一个两层的层次化搜索空间并且在每个层次上寻求最优解
- Alpa可以匹配或者超越专门的系统，泛化到新的模型上

