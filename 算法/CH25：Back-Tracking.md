# CH25：Back-Tracking

MindMap：

![CH26：Back-Tracking Algorithm](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/CH26%EF%BC%9ABack-Tracking%20Algorithm.png)

## 15.1 Back-Tracking Paradigm

使用场景：无法使用最优子结构

问题特征：

- 算法使用策略

- 可以用来解优化问题也可以用来求解可行解
  $$
  回溯\begin{cases}寻找可行解\\
  寻找最优解\end{cases}
  $$
  将问题建模为n元组：$(x_1,x_2,...,x_n)$

约束条件：
$$
\begin{cases}显式约束：explict变量取值范围\\
隐式约束：inplicit分量与分量之间的关系\end{cases}
$$

## 15.2 N-Queens

特征：寻找可行解

### 15.2.1 问题分析

若采用分治策略：

- 无法合并，子问题之间并不是相互独立的
- n推n-1，仍无法解决子问题不相互独立的问题

求解？

——暴力穷举【搜索是一种类似于穷举的方式】

每个棋盘8×8个位置不用都尝试，n×n棋盘一定每一行放一个，每一列放一个。

做法：

- 给行编号，第$i$个棋子放在第$i$行
- 给列编号，每个棋子做**列的选择**

候选解即一个8元组：$(x_1,x_2,...,x_8)$，$x_i$是棋子的列号

### 15.2.2 问题求解

#### （1）约束条件

显式约束：每个棋子的列号在1~8之间，即$x_i\in\{1,2,3,4,5,6,7,8\},1\leq i\leq 8$

隐式约束：棋子不能同行，同列，同斜线，即
$$
x_i\neq x_j且|x_i-x_j|\neq |i-j|
$$

#### （2）求解空间树

所有的候选的解：从根到叶子的所有路径

每一个节点：问题解决的目前状态

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610164712337.png" alt="image-20220610164712337" style="zoom: 33%;" />

节点的分类：

- Alive-Node：孩子节点还未生成完的结点——可能不止一个活结点
- E-Node：孩子结点正在生成——只有一个E-结点
- Dead-Node：孩子结点已经挑选完，或者人为去除

#### （3）搜索的DFS和分支限界

- DFS

问题的深度优先搜索的过程：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610165002258.png" alt="image-20220610165002258" style="zoom:50%;" />

DFS：只有当深搜无法向下继续时，才会切换E-节点

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610165445024.png" alt="image-20220610165445024" style="zoom:33%;" />

- 分支限界 

  BFS是分支限界的一种，但分支限界不都是BFS

使用**限界函数bound function**去认为的标记活结点为死结点，是回溯和暴力搜索的区别。

### 15.2.3 回溯的一般写法

#### （1）一般写法

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610170428141.png" alt="image-20220610170428141" style="zoom:50%;" />

#### （2）N皇后问题求解

- 主程序

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610170727713.png" alt="image-20220610170727713" style="zoom:50%;" />

- 限界函数



## 15.3 0-1背包问题

背包类比皇后，背包问题中使用的限界函数为分数背包的思想。

```cpp
// 回溯法求解背包问题-[迭代]
vector<int> knapsack_dfs(int capacity, vector<Item>& items, vector<int>& res) {
    sort(items.begin()+1, items.end(), Item::cmp);
    int k = 1;
    int item_num = items.size() - 1;    // 去除两个额外的位置
    double max_v_w = items[1].v_w;
    res[0] = 0;    // 初始化为0
    res[item_num + 1] = 0;
    vector<int> bestChoice = res;   // 初始化什么物品都不装, 下标第一个位置放此时的物品总价值
    while (k > 0) { // 回退到第0个物品，不存在第0个物品，终止
        res[k] += 1;                // 值递增
        while (res[k] <= 1 && !check(k, res, items, capacity, max_v_w, bestChoice[items.size()])) {
            res[k] += 1;    // 递增
        }
        if (res[k] <= 1) {  // 得到了一个尝试的结果
            if (k == item_num) {  // 是一个【更优解】
                if (value_cal(res, items) > bestChoice[items.size()]) {
                    bestChoice = res;   // 重置最好的值。
                }
            }else{  // 可以继续尝试
                k++;
                res[k] = -1;
            }
        } else {    // 该结点没有再被尝试的可能，回退
            k--;    // 做下一步的选择
        }
    }
    return bestChoice;
}
```

## 15.4 General Method

### 15.4.1 Branch & Bound Paradigm

#### （1）D-Search

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610173524053.png" alt="image-20220610173524053" style="zoom:50%;" />

- 广搜使用队列
- D-Search使用栈

#### （2）皇后问题使用分支限界

在展开时使用限界函数：

使用队列作为数据结构

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610174021719.png" alt="image-20220610174021719" style="zoom:50%;" />

分支限界需要把树保留下来。否则变量之间的关系会丢失

#### （3）分支限界的可优化性

- 分支限界与回溯的比较

- 若只找一个解，那么活结点的选择上，无论是回溯还是分支限界，选择活结点的方式过于死板

  栈或者是队列都无助于找到answer

  使用别的方式：活结点评价函数

利用评价函数选择更好的活结点，结点的代价：cost

- 以X为树根的树中需要生成多少个结点才能找到answer
- X离他最近的answer需要切换几次

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610174704724.png" alt="image-20220610174704724" style="zoom: 33%;" />

此时活结点表应该使用堆，这种解法有些事后诸葛亮

#### （4）LC-Search

**对代价进行估计：LC-Search**
$$
考虑因素：\begin{cases}估计代价\\根到该结点的深度\end{cases}
$$
![image-20220610175103569](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610175103569.png)

- g(x)：估计从结点到一个解的代价——引导我们做深度优先搜索

  一般孩子结点代价<父节点代价

- h(x)：该结点到根的深度

## 15.5 15-Puzzle

问题空间：16!种可能性

### 15.5.1 使用DFS或BFS

#### （1）DFS

![image-20220610175720972](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610175720972.png)

#### （2）BFS

![image-20220610175801394](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610175801394.png)

15迷问题使用回溯是不合适的

### 15.5.2 LCS

代价定为：有几块没摆好

但要加修正：**深度+“估计代价”**

未用先验知识：

**LC-Search的核心：设计c(x)**

![image-20220610175939702](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220610175939702.png)