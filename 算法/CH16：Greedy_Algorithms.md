# CH16：Greedy Algorithms

MindMap：

![CH16：Greedy Algorithm](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/CH16%EF%BC%9AGreedy%20Algorithm.png)

DC---优化--->DP----->Greedy

DP：有时overkill

Greedy：认准一种

## 16.1 Activity-Selection Problem

活动集合：$A = \{a_1,a_2,...,a_n\}$

不同的活动持续时间不同，开始时间和结束时间：$(s_i,f_i)\qquad 1\leq i\leq n$

目标：最大不冲突活动集合   **"non-overlapping" activities**

- Selection

$$
\begin{cases}按结束时间排序√\\按开始时间排序？\end{cases}
$$

### 16.1.1 例子分析

若穷举：$2^n$种选择

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609093250908.png" alt="image-20220609093250908" style="zoom:50%;" />

#### 16.1.1.1 分治

##### （1）二分

$a_1,a_2,……,a_k$|$a_{k+1},……,a_n$

​              3                            4

​                            3+4 ？  不一定，可能冲突

从合并的角度看，两个子问题不相互独立，可能冲突，无法合并。

##### （2）n推n-1

与二分问题相同，也可能冲突

##### （3）cross

需要子问题提供所有的最优解，次优解……规模失控

##### （4）建模子问题：活动集

$$
s_{ij} = \{a_k\in S: f_i\leq s_k<f_k\leq s_j\}
$$

活动i结束后开始，活动j开始前结束的活动集合。

将活动作为隔离：

使用一个集合$s_{0,n+1}$，其中$s_0$的结束时间为0时刻，$s_{n+1}$的开始时间为无穷

![image-20220609094800078](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609094800078.png)

最终的答案是$S_{0,n+1}$

使用某一个活动作为隔离，

中间的终止条件：$s_{i,j}$为空，如：a_i的结束比a_j的开始要晚

$A_{i,j} = A_{i,k}\cup {a_k}\cup A_{k,j}$

那么，问题的最优解的值可以递归定义为：
$$
c[i,j]=\begin{cases}0&&if\ S_{i,j}=\empty\\
\max\limits_{i<k<j}{c[i,k]+c[k,j]+1}&& if S_{i,j}\neq \empty\end{cases}
$$

#### 16.1.1.2 Greedy

##### （1）k的挑选

是否真的要挑所有的k？

- 冲突的活动不需要作为k进行挑选
- 问题特殊性：知道k=?时最优？

最优解中一定包含结束最早的。——不用穷举所有的k，直接找结束时间最早的。

![image-20220609100249053](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609100249053.png)

##### （2）求解实现

- 递归实现

![image-20220609100759740](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609100759740.png)

没有要和j比的，aj的开始时间是无穷大，肯定满足。

- 迭代实现

按照结束时间排序，进行扫描即可。

![image-20220609101214110](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609101214110.png)

时间复杂度为$O(n)$

```cpp
void iterate_AS(vector<Activity> acts, vector<Activity>& res) {
	sort(acts.begin(), acts.end(), Activity::cmp);
	res.push_back(acts[0]);
	int last_act = 0;
	for (int i = 1; i < acts.size(); i++) {
		if (acts[i].start >= acts[last_act].end) {
			res.push_back(acts[i]);
			last_act = i;
		}
	}
}
```

### 16.1.2 贪心策略分析

贪心策略何时满足：

- Greedy Choice property **一个问题的全局最优解能够通过该局部最优解得到**
- Optimal subtructure **一个问题一定使用子问题的最优解**

## 16.2 Knapsack Problem

$$
\begin{cases}0-1背包：要么拿，要么不拿，穷举2^n种可能性\\
分数背包：拿多少，穷举可能性为无穷大\end{cases}
$$

最优子结构的判定：是否一定使用子问题的最优解？

子问题：是否剩下的容量一定用其最大价值？是  

——满足最优子结构，推给子问题。

### 16.2.1 Fractional Knapsack

无法推给子问题。

最优解一定装满性价比最高的。

![image-20220609102149983](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609102149983.png)

### 16.2.2 0-1 Knapsack Problem

#### （1）分治思路分析

寻找规模压缩的方法
$$
规模\begin{cases}背包的容量\\物品的数量\end{cases}\qquad 任何一个减少都是规模压缩
$$
思路：

① 二分容量和物品：经常不对，子问题的情况太多

② n推n-1

那么推给子问题有两种情况：
$$
\begin{cases}容量全给子问题\\容量自己留一部分，剩余部分给子问题\end{cases}
$$
**子问题递归求解**

最优解的值：能得到的最大价值：

$c[i,j]：前i个物品，背包容量为j$：
$$
c[i,j] = max(c[i-1, j], c[i-1,j-w[i]] + v[i])
$$
对物品的顺序无要求，实际逻辑就是要**列举每个物品装入或者不装入**

![image-20220609103345542](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609103345542.png)

**实际上递归比两个for要快，：只会调部分，不是所有的子问题都要计算。**

初始的条件：背包容量为空和可选物品为空时，代价均为0

![image-20220609103808065](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609103808065.png)

```cpp
// 0-1背包，递归求解前item_num个物品在背包容量为capacity时的最大价值
// 使用备忘录
double knapstack_dp(vector<Item>& items, int item_num, int capacity, vector<vector<double>>& result, vector<bool>& choose) {
	// 查看备忘录
	if(result[item_num][capacity] != -99999) {
		return result[item_num][capacity];
	}
	// 当前物品的重量已经超过背包容量，直接返回给子问题
	if(items[item_num-1].weight > capacity) {
		result[item_num][capacity] = knapstack_dp(items, item_num-1, capacity, result, choose); // 打备忘录
		choose[item_num - 1] = false;
		return knapstack_dp(items, item_num-1, capacity, result, choose);	// 返回
	}
	// 否则两个子问题中挑出更优的
	double with_now = knapstack_dp(items, item_num - 1, capacity - items[item_num-1].weight, result, choose) + items[item_num-1].value;		// 装入当前的物品
	double without_now = knapstack_dp(items, item_num - 1, capacity, result, choose);	// 不装当前的物品
	// 二挑一
	if(with_now >= without_now) {
		result[item_num][capacity] = with_now;
		choose[item_num - 1] = true;
	}else{
		result[item_num][capacity] = without_now;
		choose[item_num - 1] = false;
	}
	return result[item_num][capacity];
}

// 初始化备忘录
void init_memoization(vector<vector<double>>& result, int item_num, int capacity) {
	for (int i = 0; i <= item_num; i++) {   // 背包容量为0时，最大价值为0
        result[i][0] = 0;
	}
    for(int j = 1; j <= capacity; j++) {    // 物品数量为0时，最大价值为0
        result[0][j] = 0;
    }
}

double knapstack_recursive(vector<Item>& items, int capacity, vector<vector<double>>& result, vector<bool>& choose) {
    init_memoization(result, items.size(), capacity);   // 初始化表格
    for (int i = 1; i <= items.size(); i++) {
        for ( int j = 1; j <= capacity; j++) {
            if (j >= items[i-1].weight && result[i-1][j - items[i-1].weight] + items[i-1].value > result[i-1][j]) {
                choose[i-1] = true;
                result[i][j] = result[i-1][j - items[i-1].weight] + items[i-1].value;
            }else{
                choose[i-1] = false;
                result[i][j] = result[i-1][j];
            }
        }
    }
    return result[items.size()][capacity];
}

```

## 16.3 Huffman codes

### 16.3.1 应用背景

定长编码和变长编码效率不同：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609110603135.png" alt="image-20220609110603135" style="zoom:50%;" />

**![image-20220609110641796](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609110641796.png)**

### 16.3.2 Prefix-free Code

无前缀码，解码结果是唯一的。

哈夫曼编码的求解过程：

![image-20220609111025205](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609111025205.png)

![image-20220609111142674](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220609111142674.png)

