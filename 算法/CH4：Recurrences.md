# <center>CH4：Recurrences</center>

Review：

- MergeSort
- 分析MergeSort的时间复杂度
  - 递归树 Picture of Recursion Tree
  - 迭代法 Telescoping
  - 数学归纳法 Mathematical Induction
- 渐近分析
  - $O-Notation$
  - $\Omega-Notation$
  - $\Theta-Notation$

---

算法包含对自身的调用时，其运行时间可以用递归式表示：

$MergeSort：T(n) = aT(n/b) +D(n) +C(n) = 2T(n/2) + O(n)$

3种求解递归函数的方法：

- 代换法-------------Substitution
- 递归树法----------Recursion Tree
- 主方法-------------Master Method

## 4.1 Substitution Method 代换法

### （1）步骤

步骤：

- 猜测一个解的形式 Guess the form of solution
- 使用数学归纳法证明 Verify by induction
- 求解常数项 Solve for constants

### （2）例子

#### Example1：$T(n) = 4T(n/2) + 100n$

猜测：$O(n^3)$<font color = "red">（如果要证明$\Theta$的话，那么要分别证明O和$\Omega$）</font>

归纳基础：$T(1) = \Theta(1)$

**<font color="#0000dd">归纳假设：$T(k) \leq ck^3 $在$k < n$时成立</font>**

使用数学归纳法证明：$T(n) \leq cn^3$

证明：
$$
\begin{aligned}
\mathrm{T}(\mathrm{n}) &=4 \mathrm{~T}(\mathrm{n} / 2)+100 n \\
& \leq 4 \mathrm{c}(\mathrm{n} / 2)^{3}+100 n \\
&=(\mathrm{c} / 2) n^{3}+100 n \\
&=\mathrm{cn}^{3}-\left((\mathrm{c} / 2) n^{3}-100 n\right)\\
&\leq\mathrm{cn}^{3}
\end{aligned}
$$
![image-20220605163554811](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605163554811.png)

最后确定c和n的值即可：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605163808829.png" alt="image-20220605163808829" style="zoom:50%;" />

#### Example2：$T(n) = 2T(\lfloor n/2\rfloor) + n$

要严格遵守证明的结果的模式，下面的证明是错误的：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605164632112.png" alt="image-20220605164632112" style="zoom:50%;" />

#### Example3：尝试变量替换

- 考虑：$T(n) = 2T(\sqrt{n}) + lgn$

- Rename：$m = lgn\Rightarrow T(2^m) = 2T(2^{\frac{m}{2}})+m$

- 令$S(m) = T(2^m)$，则有：$S(m) = 2S(m/2) + m$

  设当$m_0\leq m$时有：$S(m_0)\leq cm_0lgm_0$
  $$
  S(m) = 2S(m/2) + m\\
  \leq 2c(m/2)lg(m/2) + m\\
  =cmlg(m/2)+m\\=cmlgm-cm + m\\
  =cmlgm-(c-1)m\\=O(mlgm)
  $$

- 转换回来

  $T(2^m) = S(m) = O(mlgm)$

  $T(m) = O(lgmlgm)$

  $T(n) = O(lgnlglgn)$

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605171949471.png" alt="image-20220605171949471" style="zoom:50%;" />

### （3）猜测方法

使用代换法的一个重要技巧是要猜测

猜测的方式：

- 递归树的方法帮助猜测

- 与先前见过的相似

- 先整较松的上届然后缩小区间

  降低上届，提高下界，缩小不确定区间

## 4.2 Recursion-tree Method

递归树是一个好的猜测的直接方法  

递归树方法可能不可靠，就像任何使用省略号的方法一样

### 4.2.1 例子说明

$T(n) = 3T(n/4) + \Theta(n^2)$

#### （1）递归树猜测上界

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605173026528.png" alt="image-20220605173026528" style="zoom:50%;" />

![image-20220605175243692](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605175243692.png)

- 最后一层叶子数：$3^{log_4^n} = n^{log_4^3}$

- 每层开销：$((\frac{3}{16})^icn^2)$

- 总开销：$cn^2 + (\frac{3}{16})cn^2+(\frac{3}{16})^2cn^2+......+(\frac{3}{16})^{log_4^n-1}cn^2 + O(n^{log_4^3})=\frac{1\times (1-(\frac{3}{16})^{log_4^n+1})}{1-\frac{3}{16}}+O(n^{log_4^3})$

  使用等比数列的上界作为上限：
  $$
  cn^2\sum_{i=0}^{log_4^n-1}(\frac{3}{16})^i = \frac{cn^2}{1-\frac{3}{16}}=\frac{13}{16}cn^2
  $$
  所以：
  $$
  T(n) =cn^2 + (\frac{3}{16})cn^2+(\frac{3}{16})^2cn^2+......+(\frac{3}{16})^{log_4^n-1}cn^2 + O(n^{log_4^3})=\frac{1\times (1-(\frac{3}{16})^{log_4^n+1})}{1-\frac{3}{16}}+O(n^{log_4^3})\\
  \leq \frac{13}{16}cn^2 +O(n^{log_4^3}) = O(n^2)
  $$
  

#### （2）迭代法证明

递归基础：$T(1) = O(1)$

归纳假设：$T(k) \leq ck^2，当k<n时成立$

证明：$T(n) \leq cn^2$
$$
T(n) = 3T(n/4) + d(n^2)\\
\leq 3c(n/4)^2 + d(n^2)\\
=\frac{3}{16}cn^2 + d(n^2)\\
\leq cn^2
$$
当$d\geq \frac{16}{13}c$时成立。

## 4.3 Master Method

“cook book” Method for solving recurrences of the form:
$$
T(n) = aT(n/b) + f(n)\\
其中a \geq 1 , b >1
$$
![image-20220606103409477](../../../../Users/%E5%AD%99%E8%95%B4%E7%90%A6/AppData/Roaming/Typora/typora-user-images/image-20220606103409477.png)

---

叶子的个数：$a^h = a^{log_b^n} = n^{log_b^a}$

比较树根$f(n)$和$n^{log_b^a}$

三种情况：
$$
由根到叶子\begin{cases}减少\\不变\\增大\end{cases}
$$

---

### 4.3.1 Compare f(n) with $n^{log_b^a}$

#### （1）$f(n) = O(n^{log_{b}^{a}-\epsilon})$  叶子开销严格大于树根

$f(n)$多项式增长慢于$n^{log_b^a}(by\ an\ n^{\epsilon}\ factor)$

$Solution:T(n) = \Theta(n^{log_b^a})$

**<font color = "red">注意是$\Theta$</font>**

#### （2）$f(n) = O(n^{log_{b}^{a}}(lgn)^k)$ 树根开销为叶子开销的$lgn$的$k$次方倍

其中$k\geq 0$，整数

$Solution:T(n) = \Theta(n^{log_b^a}lgn^{k+1})$最终结果多乘以一个树高的k次方

#### （3）$f(n) = O(n^{log_{b}^{a}+\epsilon})$ 叶子开销小于树根开销

需要额外满足$af(n/b)\leq cf(n)$

$T(n)=\Theta(f(n))$树根量级

### 4.3.2 Conclusion

![image-20220606103532574](CH4%EF%BC%9ARecurrences.assets/image-20220606103532574.png)

