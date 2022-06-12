# CH24：Shorest Path

## 24.1 定义

最短路径问题：

最优解：最短路径，怎么走

最优解的值：最短路径的长度

最短路径的权重定义为：
$$
最短路径的权重定义为：\begin{cases}min(\{w(p)：\text{if there is a path p from u to v}\}
\\
\infty\quad otherwise\end{cases}
$$
一般使用有向图，无向图视作双向。

最短路径一定满足最优子结构（不可用simple约束）

---

三角不等式：
$$
\sigma (u,v)\leq \sigma(u,x)+\sigma(x,v)
$$
![image-20220609142951319](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609142951319.png)

如果图中包含负权环，那么最短路径可能不存在。

![image-20220609142636207](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609142636207.png)

## 24.2 Single-Source shortest path

单源：源点的位置固定，source固定

每对$\sigma(u,v)$：每一对结点之间

### 24.2.1 算法分析

满足最优子结构：一定使用子问题的最优解

- 二分

k是中间的一个结点，找u->k + k->v，穷举所有的k，挑一个最小的

无法计算：可能存在子问题互相包含。

- n推n-1

一样的问题。

**计算次序的确定是最短路径的关键。**

计算次序是否存在?
$$
\begin{cases}无顺序：人为的增加一个顺序\\
有顺序：尝试，分别做子问题的尝试\end{cases}
$$

### 24.2.2 dijkstra算法

#### （1）思路分析

顺序是否真的不存在？考虑下面的图：

![image-20220609145418264](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609145418264.png)

特点：无负权，可能存在计算次序

无负权值的图有特点：越往后走路径只能越长，所以对于从A到其余的点，C离A更近，只可能C是别的问题的子问题，不可能别的问题是C的子问题。于是：

- 尝试让C当别的问题的子问题
- 尝试让E当别的问题的子问题
- 尝试让B充当别的问题的子问题
- ……

与前面的DP逻辑的不同：

- DP：父问题找子问题
- 现在：子问题找父问题

即dijkstra算法

#### （2）程序实现

![image-20220609153520962](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609153520962.png)

#### （3）时间复杂度分析

最坏的情况下，每条边都会用来做松弛，时间复杂度为：
$$
T(n) = \Theta(VT_{EXTRACT\_MIN} +ET_{DECREASE\_KEY})
$$

| Q              | $T_{EXTRACT\_MIN}$ | $T_{DECREASE\_KEY}$ | Total     |
| -------------- | ------------------ | ------------------- | --------- |
| array          | O(V)               | O(1)                | O(v2)     |
| Binary Heap    | O(lgV)             | O(lgV)              | O(ElgV)   |
| Fibonacci Heap | O(lgV)             | O(1)                | O(E+VlgV) |

### 24.2.3 其他边权的情况

#### 24.2.3.1 无权图

使用广度优先搜索，堆数据结构换为队列即可。

![image-20220609160619997](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609160619997.png)

时间复杂度$Time=O(V+E)$

![image-20220609161024581](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609161024581.png)

### 24.2.4 Bellman-Ford

如果真的含有负权值边，那么之前的计算次序将不可用，使用Bellman-Ford

其思想在于：若子问题嵌套，那么多使用几次。只要有一条边就拿来尝试更新，但不是只使用一次。

本质：穷举所有的前驱

循环次数：|V|-1

程序实现：

![image-20220609161615633](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609161615633.png)

算法实现时，可以为遍历边的次序进行规定。按照次序利用每条边尝试进行松弛。

```cpp
/***  Bellman-Ford算法  ***/
void Graph::bellman_ford(char s) {
    int source = s - 'A';   // 源点的下标
    vector<int> distance(vexnum, 999999);   // 距离数组的初始化
    distance[source] = 0;
    vector<char> pre(vexnum, '@');
    int iternum = vexnum - 1;
    for (int i = 0; i < iternum; i++) {  // 松弛次数为顶点数-1
        for (int j = 0; j < arcList.size(); j++) {   // 邻接表头
            for (int k = 0; k < arcList[j].size(); k++) {    // 遍历每条边
                int target = arcList[j][k].adjvec;
                if (distance[target] > distance[j] + arcList[j][k].weight) {
                    distance[target] = distance[j] + arcList[j][k].weight;
                    pre[target] = j + 'A';
                }
            }
        }
    }
    // 再松弛一次，若有负环则报错
    for (int i = 0; i < arcList.size(); i++) {
        for (int j = 0; j < arcList[i].size(); j++) {   // 遍历每条边
            int target = arcList[i][j].adjvec;
            if(distance[target] > distance[i] + arcList[i][j].weight) {
                cout << "Error: There exists a negative weight circle in the graph!";
                return;
            }
        }
    }
    // 否则没有负权环，打印最短路径长度
    printDist(distance, s, pre);
}
```

思考，为什么进行n-1轮即可。

一个无负环的图，最短路径的边的最多条数为n-1条，否则会出现环。

不绕环的最短路径，边数最多是n-1

![image-20220609162440441](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609162440441.png)

## 24.3 All-pairs shorest path

所有结点对之间的最短路径：

- 无负权：跑n遍dijkstra   $O(V^2lgV + EVlgV)$
- 含负权：跑n遍Bellman-Ford 

### 24.3.1 用边数做约束的DP

#### （1）算法思想

Dynamic Programming 规模变小：边数变少

限制边的数量——n推n-1

定义：
$$
d_{ij}^m:从i→j最多过m条边的最短路径
$$
那么有：
$$
d_{ij}^0 =\begin{cases}0&& if \ i=j\\
\infty&& if\ i\neq j\end{cases}
$$
松弛为：

求 k ---> v的最短路径，过100条边的最短路径，求99条，穷举k
$$
d_{ij}^{(m)}=\min_{k}\{d_{i,k}^{(m-1)} + a_{k,j}\}
$$
最终刻画问题为最多过x条边的最短路径。

![image-20220609165216909](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609165216909.png)

- 自上而下：推到m=0时就有答案了。
- 自下而上：从m=0开始推，m从小到大计算

$$
d_{ij}^0：i到j最多0条边\rightarrow d_{ij}^1：i到j最多1条边\rightarrow……d_{ij}^{n-1}：i到j最多n-1条边
$$

**无负环，最短路径最多n-1条边**。

假设使用迭代的方式：

```cpp
for m = 1~n-1	// 每个m计算一个二维数组
    for i = 1~n
        for j = 1~n
            for k = 1~n	// 所有前驱
```

需要使用三维数组，逻辑上是一个二维数组

实际的操作没有标m：

![image-20220609170508574](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609170508574.png)

计算的过程可能被加速：

无负环：
$$
\sigma(i,j) = d_{ij}^{(n-1)} = d_{ij}^{(n)}=d_{ij}^{(n+1)}……
$$
故m在工程中可以省略。

负环如何检测？   再计算一轮未必能检测出来

**解决：i=j的场景，检查对角线是否有负值，并将m算到n**

将该算法应用到单源点，即为Bellman-Ford算法。

#### （2）改进：Matrix multiplication

每次使用上一次计算的结果和邻接矩阵计算一个新的矩阵。上述思想的伪代码表示如下：

```cpp
for m = 1 ~ n-1
    for i = 1 ~ n
        for j = 1 ~ n
            for k = 1 ~ n
                if d_ij^m >= d_ik^m + a_{k,j}
					d_ij^m = d_ik^m + a_{k,j}
```

需要穷举所有的k，所以使用的是结果矩阵的第i行去加上邻接矩阵的第j列。——类似于矩阵相乘的逻辑

![image-20220609203755221](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609203755221.png)

$$
C_{ij} = \min\{a_{ik}+b_{kj}\}
$$

### 24.3.2 用顶点数做约束的DP：Folyed Warshell

#### （1）算法思路

定义：
$$
c_{ij}^k表示从i到j最多经过前面k个顶点的最短路径
$$
尝试n推n-1：

过前n个结点 尝试 推给过前n-1个结点，类比0-1背包，那么只有两种情况：
$$
c_{ij}^{(k)}\begin{cases}过第k个顶点：c_{ik}^{(k-1) }+ c_{kj}^{(k-1)}\\
不过第k个顶点：c_{ij}^{(k-1)}\end{cases}
$$


过0个结点时，$a_{ij}$的权值就是答案
$$
c_{ij}^{(k)} = \min_k\{c_{ij}^{(k-1)}, c_{ik}^{(k-1)}+c_{kj}^{(k-1)}\}
$$
<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610105556208.png" alt="image-20220610105556208" style="zoom:50%;" />

最终过不过第k个结点，比较进行确定。

伪代码：

```cpp
for k = 1 ~ n
    for i = 1 ~ n
        for j = 1 ~ n
            do if cij > cik + ckj			// Relaxation
                then cij = cik + ckj	
```

只用一个数组，和上述的用边约束相同，可能会加速计算，但结果不会出错。

Example：

- 一个结点都不过：原距离矩阵

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610111624976.png" alt="image-20220610111624976" style="zoom:50%;" />

- 过第1个结点

  <img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610112008609.png" alt="image-20220610112008609" style="zoom:50%;" />

- 过第2个结点

  <img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610112050354.png" alt="image-20220610112050354" style="zoom:50%;" />

- 过第3个结点

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610112035534.png" alt="image-20220610112035534" style="zoom:50%;" />

#### （2）负环检测

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610112325464.png" alt="image-20220610112325464" style="zoom:50%;" />

## 24.4 Johnson 算法

要是是正权值，那么可以使用dijkstra算法，如何**重赋权**？

考虑下面的图，若顶点权值如下，使用$w(u,v)+h(u)-h(v)$得到一个新的权值，为正值：

![image-20220610114431299](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610114431299.png)

- 运行一遍dijkstra，得到的最短路径值做reweighting

- 选择谁当source

  - 数学性质上任何点都可以成立，但是要用于修改原来的权值
  - **source到每个点的距离不能是无穷大**
  - 实际工程上，使用一个不存在的源点source，到任何点的距离为0

- 使用哪个算法？

  **Bellman-Ford**

算法步骤：

- Bellman-Ford增加点的情况下跑最短路径值，作为顶点值
- 重赋权
- 跑dijkstra





