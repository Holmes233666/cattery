# <center>CH2：算法入门

The problem of sorting

- input：sequence <$a_1,a_2,...,a_n$> of n natural numbers
- output：permutation <$a_1', a_2',...,a_n'$> such that $a_1'\leq a_2'\leq ....\leq a_n'$

## 2.1 Insert Sort

算法：

```cpp
for(i = 2; i < length[A]; i++){
    key = A[i];	// 把A[i]插入序列A[1...i-1]	排A[i]时，前j个已经排好
    i = j - 1 
    while(j >= 1 and A[j] > key){
        A[j+1] = A[j];	// 后移
        j--;
    }
    A[j+1] = key;
}
```

![image-20220320202930700](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220320202930700.png)

## 2.2 Kinds of Analysis

- **worstest - case（usually）：最坏情况优先考虑**
- Average - case（sometimes）
- Best - case（bogus）

### 2.2.1 循环不变式 loop-invariants

Proving correctness Using Loop-invariants【<font color=#FF0000 >外层为循环</font>】

需要证明的三个性质：

- **初始化**：它在循环的第一轮迭代开始时，应该是正确的
- **保持**：如果在某一次的迭代开始前是正确的，那么在下一次迭代开始前，他也应该保持正确
- **终止**：不变式给了一个有用的性质，表明算法是正确的

### 2.2.2 运行时间

#### （1）Running Time

运行时间的取决因素：

- 输入序列：已经排列好的序列【数据特征】
- 数据规模：更短的序列更容易排好

通常，我们考虑运行时间的上界，因为这是一个保证

#### （2）Machine-independent time

插入排序的最差时间取决于计算机的速度：

- 相对速度（on the same machine）——关注算法
- 绝对速度（on different machine）

通常我们忽视机器相关时间的限制，关注“**the growth of T(n) as n → ∞**”

称为渐近分析**“Asympotic Analysis”**

#### （3）$\theta-$notation

关心增长率，低阶和常数丢弃 Drop low-order terms; ignore leading constants

Example：$3n^3+90n^2-5n+6046 = \theta(n^3)$

当n变得足够大时，一个$\theta(n^2)$的算法总会打败一个$\theta(n^3)$的算法

### 2.2.3 Insertion Sort Analysis

- Worest case：Input reverse sorted【逆序的序列】

$$
T(n) = \sum\limits_{j = 2,...,n}\theta(j) = \theta(n^2)
$$

- Average case：All permutations equally likely
  $$
  T(n) = \sum\limits_{j = 2,...,n}\theta(j/2)=\theta(n^2)
  $$

## 2.3 Analysis Algorithm

Analysis An Algorithm：

算法分析指的是对一个算法所需的资源进行分析，预测算法所需的资源，包括内存、通信带宽或计算机硬件等。  但是通常我们更想测量计算时间。

### 2.3.1 RAM

RAM：单处理机，随机存储模型  random-access-machine（RAM） Model

- with no concurrent operations 不考虑并发
- executed as an atom operation 不考虑中断，原子操作
- 每条指令需要固定时间
- 内存容量足够大

在RAM模型下，**对基础的操作进行计数即可 count fundamental operation**。

### 2.3.2 Analysis of Insert Operation

![image-20220321154855659](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321154855659.png)

插入排序的时间计算如下：
$$
T(n) = c_1n+c_2(n-1) + c_4(n-1)+c_5\sum\limits_{j=2}^nt_j+c_6\sum\limits_{j=2}^n(t_j-1) + c_7\sum\limits_{j=2}^n(t_j-1)+c_8(n-1)\\
$$

- best case：数组是有序的

  <font color=#0099ff >**$t_j = 1$，检查一次，不需要后移**</font>

$$
T(n) = c_1n+c_2(n-1) + c_4(n-1)+c_5(n-1)+c_8(n-1)\\=(c_1+c_2+c_4+c_5+c_8)n-(c_2+c_4+c_5+c_8)
$$

运行时间是n的线性函数

- worest case：数组是倒序的

  <font color=#0099ff >**$t_j = j$，检查一次，不需要后移**</font>

$$
T(n) = c_1n+c_2(n-1)+c_4(n-1)+c_5(n(n+1)/2-1)+c_6(n(n-1)/2)+c_7(n(n-1)/2)+c_8(n-1)\\=(c_5/2+c_6/2+c_7/2)n^2+(c_1+c_2+c_4+c_5/2-c_6/2-c_7/2+c_8)n-(c_2+c_4+c_5+c_8)
$$

**运行时间是n的二次函数。**

---

note：

- 对于某些算法来说，最坏情况经常发生
- 平均情况有时候和一般情况一样糟糕

### 2.3.3 Order of Growth

我们只考虑运行时间的增长顺序:  

- 我们可以忽略低阶项，因为对于非常大的n，它们相对不重要。  
- 我们也可以忽略前项的常系数，因为对于非常大的n，它们对计算效率的增长速度并不重要。  

我们刚刚说过，最好的情况是n的线性而最坏/平均的情况是n的二次型。 

## 2.4 Designing Algorithm

### 2.4.1 Divide and Conquer

To solve P：

- **Divide** P into smaller problems $$P_1, P_2, ... , P_k$$
- **Conquer** by solving the (smaller) subproblems recursively
- **Combine** the solutions to $$P_1,P_2,...,P_k$$ into the solution  for P

![image-20220321164743811](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321164743811.png)

#### （1）Merge Sort

Using Divide and Conquer, we can obtain a merge sort algorithm

- Divide：divide n elements into two subsequences of n/2 elements each
- Conquer：Solve the two subsequences recursively
- Combine：Merge the two sorted subsequences to produce the sorted answer

---

Merge-Sort(A，p，r)

**INPUT**: a sequence of n numbers stored in array A

**OUTPUT**: an ordered sequence of n numbers

```cpp
MergeSort(A,p,r)
    if p < r
        q = floor((p+r)/2);
        MergeSort(A, p, q);
        MergeSort(A, q+1, r);
        Merge(A, p, q, r);

Merge(A, p, q, r)
    n1 ← q - p + 1;
	n2 ← r - q;           // r - (q + 1) + 1 = r - q
	create arrays L[1...n1+1] and R[1...n2+1]	// L和R均多创建一个位置
    for i ← 1 to n1
        do L[i] ← A[p+i-1]	// i = 1起始，左半
    for j ← 1 to n2
        do R[j] ← A[q+j]	// j = 1起始，右半
    L[n1 + 1] ← ∞
    R[n2 + 1] ← ∞	// 无穷大的设置，不用再额外增加一个循环进行扫尾
    i ← 1
    j ← 1
    for k ← p to r
        do if L[i] ≤ R[j]
            then A[k] ← L[i]
            	 i ← i + 1
            else A[k] ← R[j]
                 j ← j + 1
```

----

**Actions of Merge Sort：**

![image-20220321172825181](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321172825181.png)



### 2.4.2 Analysis divide and conquer Algorithm

- $T(n)$

  running time on a problem of size n 【n  ：问题规模】

  假设把原问题分成a个子问题，每个子问题是原问题规模的$1/b$

  - 对于MergeSort来说，**a = 2**，每个问题的规模是原问题的**1/b**

- $D(n)$

  分解的开销

- $C(n)$

  合并的开销   子问题 --合并-->原问题

##### MergeSort Recurrence Equation

$$
T(n) = \begin{cases}\Theta(1)\quad if\ n≤c\\
aT(n/b)+D(n) + C(n)\quad otherwise\end{cases}
$$

对于MergeSort来说，递归式如下：
$$
T(n) = \begin{cases}\Theta(1)\quad if\ n≤c\\
2T(n/2)+\Theta(n)\quad otherwise\end{cases}
$$
Claim：
$$
T(n) = nlog_2^n
$$
所有情况下均为$$nlog_2^n$$

- 最好
- 最坏
- 平均

![image-20220321192232644](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321192232644.png)

$log_2^n+1$是针对结点的，若是针对边则是$log_2^n$。每层的和是一定的，都是cn

所以总的花费时间为：
$$
\sum_{i=1}^{log_2^n} cn = cn·log_2^n
$$

##### Proof by Telescoping

$$
T(n) = 2T(n/2)+n = 2(2T(n/4) + n/2)+n = 4T(n/4)+2n\\
=......=nT(n/n)+log_2^n n=nT(1)+nlog_2^n
$$

