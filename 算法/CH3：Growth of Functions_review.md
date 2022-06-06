# CH3：Growth of Functions

## MindMap

![image-20220605154233378](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605154233378.png)

## Asymptotic Growth

分析算法时我们对算法输入规模为n时的最坏情况下运行时间函数感兴趣

- 不考虑常数项
- 不考虑低阶项

## 3.1 Asymptotic Notation

### 3.1.1 O-Notation

#### （1）定义

$$
O(g(n)) = \{f(n):\text{There exists positive c and }n_0\text{ such that } 0\leq f(n)\leq cg(n)\text{ for all }n\geq n_0\}
$$

- 正整数c
- 正整数n0
- <font color = "red">g(n)是最高次项</font>
- `exists`：存在即可

<font color = \#006666>**$O(g(n))$是一个满足上述条件的运行时间函数的集合**</font>

---

$O(.)$：用于渐进上界函数 $\text{is used to asymptotic upper bound a function}$

$O(.)$：用来表示最坏情况的运行时间$\text{is used to bound worst-case running time}$

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605145004035.png" alt="image-20220605145004035" style="zoom:50%;" />

---

#### （2）Example

![image-20220605145849484](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605145849484.png)

#### （3）使用情况

当我们说道“**运行时间是O(n)**”时，我们指的是<font color = "blue">**最坏情况下**</font>的运行时间是O(n)

通常写为：$f(n) = O(g(n))$，而不是$f(n)\in O(g(n))$——虽然是个集合

我们也在等式中使用$O(n)$，例子：
$$
2n^2+3n +1 = 2n^2 +O(n)
$$
$O(1)$表示常数时间

### 3.1.2 $\Omega-Notation$

#### （1）定义

$$
\Omega(g(n)) = \{f(n):\text{There exists positive c and }n_0\text{ such that } 0\leq cg(n)\leq f(n) \text{ for all }n\geq n_0\}
$$

我们使用$\Omega(.)$来表示运行时间的下界

![image-20220605150431979](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605150431979.png)

#### （2）Example

![image-20220605150735127](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605150735127.png)

when we say “the running time is $Ω(n^2 )$” we mean that  the best-case running time is $Ω(n^2)$ ——the worst case might be  worse.

![image-20220605150910298](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605150910298.png)

### 3.1.3 $\Theta-Notation$

#### （1）定义

$$
\Theta(g(n)) = \{f(n):\text{There exists positive }c_1，c_2\text{ and }n_0\text{ such that } 0\leq c_1g(n)\leq f(n)\leq c_2g(n) \text{ for all }n\geq n_0\}
$$

我们使用$\Theta-Notation$来表示**紧致界**

充要条件：$f(n) = Θ(g(n))\ if\ and\ only\  if\ f(n) =O(g(n))\ \ and\ \ f(n) = Ω(g(n))$

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605151409968.png" alt="image-20220605151409968" style="zoom:50%;" />

#### （2）Example

![image-20220605151503639](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605151503639.png)

### 3.1.4 总结

#### （1）表示法

![image-20220605151600769](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605151600769.png)

#### （2）在等式中的使用

使用$O、\Theta、\Omega$在函数中替换低阶项简化函数表达

![image-20220605151734530](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220605151734530.png)