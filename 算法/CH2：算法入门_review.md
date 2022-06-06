# 	<center>CH2 算法入门</center>

## MindMap

![image-20220605145111689](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605145111689.png)

排序问题：

输入：序列$<a_1,a_2,...,a_n>$

输出：非降序排列：$<a_1',a_2',...,a_n'>$，满足$a_1'\leq a_2'\leq...\leq a_n'$

## 2.1 Insert Sort

程序：

```cpp
void insertSort(vector<int>& vec) {
	for (int i = 2; i < vec.size(); i++) {
		int key = vec[i];
		int j = i - 1;
		while (j > 0 && vec[j] > key) {
			vec[j+1] = vec[j];			// 后移操作 
			j--;
		}
		vec[j+1] = key;
	}
}
```

注意后移操作：`A[j+1] = A[j]`

![image-20220604100637491](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220604100637491.png)

若位置0处数字为负无穷，那么`while`循环中将不用写条件`j>0`。

## 2.2 Analysis of algorithms

### 2.2.1 运行时间

Running Time：

- 数据的初始特征
- 数据规模
- we seek <u>***upper bounds***</u> on the running time

Kinds of Analysis：

- Worst - case（<font color=#0099ff >usually 最坏情况最优先考虑</font>）
- Average - case（sometimes）
- Best - case（bogus）

### 2.2.2 loop-invariant 

Proving **correctness** using loop-invariant：最外层必须是循环，证明某一性质每一次循环都不变

需要证明的三个性质

- 初始化：他在循环的第一轮迭代开始前，应该是正确的
- 保持：如果在某一次的迭代开始前是正确的，那么在下一次的迭代开始前他也应该保持正确
- 终止：循环结束时，不变式给了一个有用的性质，表明算法是正确的

> 使用循环不变式证明插入排序
>
> 证明：在每次迭代开始前子数组A[1...j-1]中元素为原数组中的A[1...j-1]，但是是有序的
>
> - 初始化 j = 2
> - 保持：A[j - 1], A[j - 2],… are moved one at a time till  the proper position for A[j] is found.
> - 结束：当j=n+1时，整个序列有序

### 2.2.3 机器独立时间 machine independent time

- relative time：在相同的机器上——关注算法
- absolute time：在不同的机器上

忽略机器相关的约束，直接考虑 $n\rightarrow \infty$时的$T(n)$ —— **渐近分析 Asymptotic Analysis**

### 2.2.4 $\Theta-Notation$

#### （1）定义

$\Theta$表示法的定义：

- 数学定义：$\Theta(g(n)) = \{存在正整数c_1和c_2，并且n_0>0，满足当所有的n\geq n_0时，0\leq c_1g(n)\leq f(n)\leq c_2g(n)\}$

工程上：关心增长率，低阶和常数项丢弃

Example：$3n^3 + 9n^2-5n+8344 = \Theta(n^3)$

#### （2）Asyptotic Performance

当n变得足够大的时候，一个$\Theta(n^2)$的算法一定比一个$\theta(n^3)$的算法要好

### 2.2.5 Insertion sort analysis

最差情况：输入是逆序的——每个数都要前移
$$
T(n) = \sum_{j=2..n}\Theta(j)=\Theta(n^2)
$$
平均情况：
$$
T(n) = \sum_{j=2..n}\Theta(j/2)=\Theta(n^2)
$$

## 2.3 算法分析的入门

### 2.3.1 算法分析的概念

算法分析指的是对一个算法所需的资源进行分析，内存，通信带宽或者计算机硬件是主要关心的，但通常希望测量计算时间。

在我们分析算法之前，我们必须有一个将要使用的实现技术的模型，包括该技术的资源及其成本的模型

算法实现的技术模型：单处理器，随机存储模型[RAM]

### 2.3.2 Random Access Model

不考虑并发

不考虑中断，原子操作

假设每条指令花费常数时间

内存足够大

……

在RAM模型下，需要考虑的是**基本操作的次数 counting fundamental operations**

### 2.3.3 Analysis of insert operation

$cost$：每次基本操作花费的代价

$t_j$：假设第$j$次迭代需要检查$while$循环条件的次数为$t_j$

![image-20220604222716214](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220604222716214.png)

$$
T(n) = c_1n+c_2(n-1) + c_4(n-1) +c_5\sum_{j=2}^n t_j+c_6\sum_{j=2}^n (t_j-1)+c_7\sum_{j=2}^n (t_j-1)+c_8(n-1)
$$

- best-case：数组是有序的，$t_j=1$检查一次while，不需要后移

$$
T(n) = c_1n+c_2(n-1) + c_4(n-1) +c_5(n-1)+c_8(n-1) = (c_1+c_2+c_4+c_5+c_8)n -(c_2+c_4+c_5+c_8)
$$

即，最好情况下插入排序开销为线性开销。

- worest-case：数组是逆序的，那么$t_j = j$（$i=0$也会检查）

$$
\begin{aligned}
T(n)=& c_{1} n+c_{2}(n-1)+c_{4}(n-1)+c_{5}(n(n+1) / 2-1)+c_{6}(n(n-1) / 2) \\
& c_{7}(n(n-1) / 2)+c_{8}(n-1) \\
=&\left(c_{5} / 2+c_{6} / 2+c_{7} / 2\right) n^{2} \\
&+\left(c_{1}+c_{2}+c_{4}+c_{5} / 2-c_{6} / 2-c_{7} / 2+c_{8}\right) n-\left(c_{2}+c_{4}+c_{5}+c_{8}\right)
\end{aligned}
$$

运行时间是平方级开销。

---

在一些算法中，最坏情况经常发生，比如查询数据库

有时候平均情况和最差情况一样糟糕

---

Order of Growth

- 忽略低阶项
- 忽略常数项

## 2.4 Designing Algorithms

Insert Sort：增量法

### 2.4.1 Divide-and-Conquer

To solve P:

**Divide** P into smaller problems P1 , P2 , …, Pk . 

**Conquer** by solving the (smaller) subproblems  recursively. 

**Combine** the solutions to P1 , P2 , …, Pk into the solution  for P.

> note：解决问题前想一想：该问题难以解决是否是因为规模太大了，若是规模足够小，那么是不是可以解决

![image-20220604225717400](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220604225717400.png)

### 2.4.2 Merge Sort

使用分治策略，我们首先分析归并排序。

- Divide：把n个元素分成两个子序列，每个子序列有$n/2$个元素
- Conquer：递归求解两个子问题 Solve the two subsequences recursively
- Combine：合并两个子序列，得到有序的结果

#### （1）`Merge-Sort`函数

假设我们有一个`Merge(A,p,q,r)`函数，他把两个有序的数组`A[p……q]`和`A[q+1……r]`合并

---

Merge-Sort

- 输入：需要排序的n个数序列A
- 输出：有序序列

```cpp
#include<iostream>
#include<algorithm>
#include<vector>

using namespace std;

void Merge_Sort(vector<int>& vec, int start, int end);				// 分治算法主体
void Merge(vector<int>& A, int start, int mid, int end);			// 合并的过程，重点在合并
void printVec(vector<int>& vec);

int main() {
	vector<int> nums = {8,5,9,10,3,6,2,7};
	Merge_Sort(nums, 0, nums.size()-1);
	printVec(nums);
	return 0;
}

void Merge_Sort(vector<int>& vec, int start, int end) {
	if (start < end) {
		int mid = (start + end) / 2; 
		Merge_Sort(vec, start, mid);
		Merge_Sort(vec, mid+1, end);
		Merge(vec, start, mid, end);
	}
}

void Merge(vector<int>& A, int start, int mid, int end) {
	int n1 = mid - start + 1;		// 数组1个数
	int n2 = end - (mid + 1) + 1;	// 数组2元素个数 
	
	vector<int> vec1(n1+1), vec2(n2+1);		// 临时数组初始化 
	
	// 数组末尾设为无穷大简化了扫尾工作 
	vec1[n1] = 99999;
	vec2[n2] = 99999;
	
	// 数组赋值
	for (int i = start; i <= mid; i++) {
		vec1[i-start] = A[i];
	} 
	for (int i = mid + 1; i <= end; i++) {
		vec2[i-mid-1] = A[i];
	}
	
	//合并
	int j = 0, k = 0;
	for (int i = start; i <= end; i++) {
		if (vec1[j] <= vec2[k]) {
			A[i] = vec1[j];
			j++;
		}else {
			A[i] = vec2[k];	
			k++;
		}
	}	 
}

void printVec(vector<int>& vec) {
	for (int i = 0; i < vec.size(); i++) {
		cout << vec[i] << " ";
	}
}
```

注意分治时采取自上而下的递归处理时，每个分支执行的顺序，如下：

![image-20220605102039787](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605102039787.png)

### 2.4.3 Analysis divide-and-conquer Algorithm

#### （1）符号定义

$T(n)$：$\text{running time on a problem of size n}$， 问题的运行时间，n为问题规模

$D(n)$：分解的时间开销

$C(n)$：合并的开销   $子问题\rightarrow 原问题$

统一模式：
$$
T(n)=\begin{cases}\Theta(1)&& if\ n\leq c\\
D(n) + aT(n/b) + C(n)&& otherwise\end{cases}
$$

- 当问题规模足够小时，不需要分治。
- 否则递归求解

#### （2）MergeSort Analysis

$$
T(n)=\begin{cases}\Theta(1)&& if\ n\leq c\\
2T(n/2) + C(n)=2T(n/2) + \Theta(n)&& otherwise\end{cases}
$$

在归并排序中，分解时分解为2个子问题，每个问题的规模是原来的$n/2$。

Claim：$T(n) = nlog2^n$，所有情况下均为$nlgn$【平均、最好、最坏】

证明：

![image-20220605111614002](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605111614002.png)

递归到何时终止？$n/x == 1$时终止，**<font color='red'>次数：除法$log_2^n$，树高$log_2^n+1$</font>**

![image-20220605112407254](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605112407254.png)
$$
T(n) = cn·lg^n + c'n
$$

---

- 代入法证明 Proof by Telescoping

$$
T(n) =2T(n/2) + n\\
=2(2T(n/4)+n/2) +n=4T(n/4)+2n\\
=4(2T(n/8) + n/4) + 2n = 8T(n/8) + 3n\\
=……\\=nT(n/2^k) + kn\\=……\\
=nT(n/2^{lgn}) + nlgn  
$$

- 数学归纳法证明

$$
T(n) = nlgn
$$

![image-20220605133420731](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605133420731.png)

