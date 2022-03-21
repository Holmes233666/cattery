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

