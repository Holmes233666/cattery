# CH15：Dynamic Programming 



动态规划：分治应用在优化问题上

## 15.1 Optimization Problems

### 15.1.1 动态规划与DC

动态规划是一种算法设计策略，类似分治。

动态规划是自下而上的计算，而不是自上而下的计算。

**bottom up  rather than top down**

### 15.1.2 动态规划求解问题的步骤

动态规划的方法有4个步骤：

- 刻画问题的最优解的结构，如何**利用子问题求解父问题【最优子结构找出来】**

  问题的最优解一定使用子问题的最优解

- <u>递归定义</u>**最优解的值**

  区分最优解和最优解的值，动态规划求解问题时通常计算的是最优解的值，而不是最优解

  **递推式代表的是最优解的值**，表示的是怎么用子问题的最优解

- 按照从下至上的模式求解

  递归+备忘录   //    自下而上，计算次序需注意

- 重构最优解

  最优解的值知道了，还需要计算最优解
  
  怎么挑？过程需要保存。

## 15.2 装配线问题 Assembly Line Scheduling

### 15.2.1 背景

工厂中有两条平行的装配线，每条线n个站：$S_{i,1},S_{i,2},...,S_{i,n}$，$i=1,2$

每个站$S_{1,i}$和$S_{2,j}$做的事情相同，但可能有不同的装配时间：$a_{i,j}$

此外若是从装配线$i$到装配线$j$，需要花费传送时间：$t_{i,j}$   **流水线可以混用，切换流水线需要代价**

开始时有准备时间$e_i$和最后的离开时间$x_i$

![image-20220608183942841](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608183942841.png)

如何选择调度，才能获得时间上最短的最优解？

### 15.2.2 算法分析

#### （1）分析

对于当前站$S_{1,j}$，选择只有两种：

-  if $j = 1$，那么只有一种
- 如果$j \geq 2$，那么：
  - 直接从$S_{1,j-1}$站到达
  - 由$S_{2,j-1}$站传递到$S_{1,j-1}$

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608184358391.png" alt="image-20220608184358391" style="zoom:50%;" />

![image-20220608184504978](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608184504978.png)

依赖于**最优子结构Optimal substructure**：一个问题的最优解一定包含子问题的最优解

#### （2）递推式

分两个线路写递推式：

- 站点1：

$$
f_{1,j} = \begin{cases}a_{1,1} + e_1&& if \ j=1\\
min(f_{1,j-1}+a_{1,j}, f_{2,j-1}+t_{2,j-1}+a_{1,j})&& if \ j\geq2\end{cases}
$$

- 站点2：
  $$
  f_{2,j} = \begin{cases}a_{2,1} + e_2&& if \ j=1\\
  min(f_{2,j-1}+a_{2,j}, f_{1,j-1}+t_{1,j-1}+a_{2,j})&& if \ j\geq2\end{cases}
  $$
  

Total Time： $min\{f_1[n]+x_1, f_2[n]+x2\}$

### 15.2.3 算法实现

#### （1）直接实现

直接按照上述思路，实现程序如下：

```cpp
int fun1(int j) {
	if (j == 1) {
		return t1[0] + a1[1];
	}
	return min(fun2(j-1) + t2[j-1] + a1[j], fun1[j-1] + a1[j]);
}

int fun2(int j) {
	if (j == 1) {
		return t2[0] + a2[1];
	}
	return min(fun1(j-1) + t1[j-1] + a2[j], fun2[j-1] + a2[j]);
}

int main() {
	int time = min(fun1(n) + t1[j], f2(n) + t2[j]);
	return 0;
}
```

算法时间复杂度为$T(n) = 2T(n-1) + O(1)$

含有大量的重复计算的过程，时间复杂度为$O(n^2)$，可以使用备忘录的思路。

#### （2）自下而上

从1~n计算，非递归

| j     | 1    | 2    | 3    | 4    | 5    | 6    |
| ----- | ---- | ---- | ---- | ---- | ---- | ---- |
| f1[j] | 9    | 18   | 20   | 24   | 32   | 35   |
| f2[j] | 12   | 16   | 22   | 25   | 30   | 37   |

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608191009275.png" alt="image-20220608191009275" style="zoom:50%;" />

$f^* = min(35+3, 37+2)$

算法实现：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608191359169.png" alt="image-20220608191359169" style="zoom:50%;" />

使用数组$L$记录本次选择。

打印站点的选择情况：

- 栈打印
- 递归打印

## 15.3 Matrix Chain Product

### 15.3.1 问题背景

n个矩阵，每个矩阵维度可能不同，保证能够连乘，求解链乘的最小代价。

代价的定义：使用标量乘法的次数表示代价**scalar multiplications**

- $A_1:p\times q\quad A_2:q\times r$

  那么$A_1\times A_2$的代价为：$p\times q\times r$

非方阵，乘法的计算顺序导致最后的计算量不同。

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608192203052.png" alt="image-20220608192203052" style="zoom:50%;" />

- Order1

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608192745266.png" alt="image-20220608192745266" style="zoom:50%;" />

### 15.3.2 求解

#### 15.3.2.1 暴力解法

穷举加括号的方式，多少种加括号方式？
![image-20220608192918343](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608192918343.png)

#### 15.3.2.2 分治

##### （1）刻画最优解结构

考虑常见的两种分治方法：

二分：

不一定分成中间两半：最优解不一定用到子问题的最优解

n推n-1：

99个知道---->100个：不一定用第99个的最优解

上述两种情况：**二分和n推n-1提供的都是子问题的一种可能**，需要列举所有的子问题，<font color = "red">**划分很重要，所有的划分都要列举**</font>。



![image-20220608193833623](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608193833623.png)

需要列举所有的可能的子问题：n-1个子问题，原问题的最优解一定用的是n-1个子问题中的一个。

**子问题如何求解：递归求解。**

##### （2）递归定义最优解的值

最优解：怎么加括号

最优解的值：最小的计算量

首先描述该问题需要左括号位置和右括号位置两个维度，所以该递归式的公共样式必 然是一个二维表达式，不妨用𝑚[𝑖,𝑗]表示： 

- 其中𝑖是右括号的位置
- 𝑗是左括号的位置
- 𝑚[𝑖,𝑗]为规模为𝑖 ∼ 𝑗的矩阵链乘问题的最小代价值 

其次，结合上图的加括号的方式，规模为𝑛时是从𝑛 − 1个子问题中挑选。我们可以推得，对于任意一个规模为𝑖 ∼ 𝑗的问题，其子问题的个数为𝑗 − 𝑖个。 对应的子问题可以描述为𝑖 ∼ 𝑘个矩阵作为子问题计算，𝑘 + 1 ∼ 𝑗个矩阵作为另一个子 问题计算，𝑘的取值范围应该满足𝑖 ≤ 𝑘 < 𝑗 

最后，需要注意父问题的代价不仅仅是两个子问题的最小代价相加，还包括两个子问 题合并的代价，以代价数组的形式描述合并的代价即为：𝑃[𝑖 − 1]𝑃[𝑘]𝑃[𝑗] 

递归式：
$$
m[i,j] = \min_{i\leq k < j}\{m[i,k] + m[k+1,j] + p[i-1]p[k]p[j]\}
$$

##### （3）自下而上计算

###### ①递归算法

终止条件： `i==j`：开销为0

因为重复计算存在，所以，需要打备忘录

备忘录结果如下：

```cpp
// 递归写法的初始化备忘录
void init_memo(vector<vector<int>>& res, vector<int>& p) {
    for(int i = 0; i < p.size(); i++) {
        for(int j = 0; j < p.size(); j++) {
            res[i][j] = 999999;
            res[i][i] = 0;
        }
    }
}

int matrix_chain_product(vector<int>& cost, int start, int end, vector<vector<int>>& choose, vector<vector<int>>& res) {
	if (res[start][end] != 999999) return res[start][end];
	for (int k = start; k < end; k++) {
		int cost1 = matrix_chain_product(cost, start, k, choose, res);
		int cost2 = matrix_chain_product(cost, k+1, end, choose, res);
		res[start][k] = cost1;
		res[k+1][end] = cost2;
		int new_cost = cost1 + cost2 + cost[start-1] * cost[k] * cost[end];
		if (res[start][end] > new_cost) {
			res[start][end] = new_cost;
			choose[start][end] = k;
		}
	}
	return res[start][end];
}
```

###### ② 自下而上

![image-20220608205108715](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608205108715.png)

注意规模：

- 初始化，矩阵规模为1时，计算量为0；规模为其他时，初始将计算量置为无穷
- 起始矩阵确定，规模确定，那么末尾矩阵确定
- 在起始和末尾之间遍历k，选择最小的

![image-20220608204526497](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608204526497.png)

程序实现：

```cpp
// 自下而上的求解
int matrix_chain_product_down_top(vector<int>& p, int start, int end, vector<vector<int>>& choose, vector<vector<int>>& res) {
	for (int l = 2; l < p.size(); l++) {	// l表示问题的规模，最小的规模为1个矩阵，已经初始化，这里从规模为2开始 
		for (int i = 1; i < p.size() - l + 1; i++) {		// 注意填矩阵顺序 
			int j = i + l - 1;	// j定了
			for (int k = i; k < j; k++) {
				int cost = res[i][k] + res[k+1][j] + p[i-1]*p[k]*p[j];
                if(res[i][j] >= cost) {
                    res[i][j] = cost;
                    choose[i][j] = k;
                }
			} 
		} 
	}
	return res[start][end];
}
```

**时间复杂度是$O(n^3)$**

Example：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608205513191.png" alt="image-20220608205513191" style="zoom:50%;" />

##### （4）重构最优解

`p[i,j] = "("  p[i, s[i,j]]   p[s[i,j]+1, j] ")"`

```cpp
// 打印加括号的方法
void printBracket(vector<vector<int>> s, int i, int j) {
    if(i == j){
        cout << "A" << i;
        return;
    }
    cout << "(";
    printBracket(s, i, s[i][j]);
    printBracket(s, s[i][j]+1, j);
    cout << ")";
}
```

## 15.3 动态规划原理

最优子结构 optimal substructure 满足：可用DC---->DP

Q：

- 是否一定用子问题的最优解？怎么用？
- 维数
- 最优解的值：几挑一

### 15.3.1 Optimal substructure

不一定能应用在所有问题上，有些问题只能穷举：TSP

不满足最优子结构的例子：**最长/短简单路径Longest simple path**

简单路径：不能绕环

## 15.4  Longest Common Subsequence 

子序列：不一定连续，但是顺序一定

![image-20220608221251811](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608221251811.png)

### 15.4.1 算法分析

暴力求解：首先找到较短串的所有子序列$O(2^m)$，然后在在较长串中比较$O(n)$，总时间复杂度为$O(n2^m)$

分治：

串变短方式：

- 二分
- n推n-1
- 串末尾比较

$$
\begin{cases}末尾相同：最优解一定会用该字母，最终解为子问题最优解 + 1\\
末尾不同：不可能两个字母都用，只有一个可用，挑一个最优的\end{cases}
$$

递归式：
$$
c[i,j] = \begin{cases}c[i-1,j-1]+1&& x[i]==y[j]\\
max(c[i-1,j],c[i,j-1])&& x[i]\neq y[j]\end{cases}
$$
边界条件：

$j==0或i==0$，某一串为空，那么最长公共子序列长度为0

### 15.4.2 算法实现

#### （1）直接实现

时间复杂度较高，重复子问题

```cpp
// 递归-无备忘录
int LCS_recursive(string& x, string& y, int i, int j) {
	if (j == -1 || i == -1) {
		return 0;
	}
	if (x[i] == y[j]) return LCS_recursive(x, y, i-1, j-1) + 1;
	return max(LCS_recursive(x, y, i-1, j), LCS_recursive(x, y, i, j-1));
} 

```



![image-20220608222356997](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608222356997.png)

#### （2）递归+备忘录

```cpp
/** 递归写法，带备忘录 **/
int recursive_LCS_Memoization(string& x, string& y, int i, int j, vector<vector<int>>& res, vector<vector<string>>& s) {
    if(res[i+1][j+1] != -999999) {   // 自己不为空，那么直接返回即可。
        return res[i+1][j+1];
    }
    if(x[i] == y[j]) {
        int num = recursive_LCS_Memoization(x, y, i - 1, j - 1, res, s);
        res[i+1][j+1] = num + 1;
    }else{
        int num1 = recursive_LCS_Memoization(x, y, i-1, j, res, s);
        int num2 = recursive_LCS_Memoization(x, y, i, j-1, res, s);
        res[i+1][j+1] = max(num1, num2);
    }
    return res[i+1][j+1];
}

/* 备忘录的初始化 */ 
void init_memoization(vector<vector<int>>& res, string& x, string& y) {
    // 第0行和第0列初始化为0
    for(int i = 0; i < x.length() + 1; i++) {
        res[i][0] = 0;
    }
    for(int j = 0; j < y.length() + 1; j++) {
        res[0][j] = 0;
    }
    // 其他值初始化为-999999
    for(int i = 1; i < x.length() + 1; i++) {
        for(int j = 1; j < y.length() + 1; j++) {
            res[i][j] = -999999;
        }
    }
}
```

#### （3）自下而上

```cpp
/** 递归写法，带备忘录 **/
int recursive_LCS_Memoization(string& x, string& y, int i, int j, vector<vector<int>>& res, vector<vector<string>>& s) {
    if(res[i+1][j+1] != -999999) {   // 自己不为空，那么直接返回即可。
        return res[i+1][j+1];
    }
    if(x[i] == y[j]) {
        int num = recursive_LCS_Memoization(x, y, i - 1, j - 1, res, s);
        res[i+1][j+1] = num + 1;
    }else{
        int num1 = recursive_LCS_Memoization(x, y, i-1, j, res, s);
        int num2 = recursive_LCS_Memoization(x, y, i, j-1, res, s);
        res[i+1][j+1] = max(num1, num2);
    }
    return res[i+1][j+1];
}

/* 备忘录的初始化 */ 
void init_memoization(vector<vector<int>>& res, string& x, string& y) {
    // 第0行和第0列初始化为0
    for(int i = 0; i < x.length() + 1; i++) {
        res[i][0] = 0;
    }
    for(int j = 0; j < y.length() + 1; j++) {
        res[0][j] = 0;
    }
    // 其他值初始化为-999999
    for(int i = 1; i < x.length() + 1; i++) {
        for(int j = 1; j < y.length() + 1; j++) {
            res[i][j] = -999999;
        }
    }
}
```

## 15.5 最大子数组和

漏掉不能递归，是不愿看到的。

改变建模方式：以$a_k$结尾的最大和子数组

$b[j] = max(b[j-1]+a[j], a[j])$

子问题是不是正数？正数就加

![image-20220608230647820](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608230647820.png)
