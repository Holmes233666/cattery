# CH1：Introduction

## 1.1 Factors of Programming

- programming language

- IDE

- software achitecture

  架构：

  ![image-20220319200833490](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220319200833490.png)

  架构的类型：
  $$
  \begin{cases}C2\ Style：conponent\  \& \ connector\\Pipe-Fliter\ Style:输入\rightarrow 输出\\Repositories\ Style：以某一数据源为中心的操作\end{cases}
  $$
  成熟的模式：逐步产生商业架构

  - Java EE

  ![image-20220319201358028](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220319201358028.png)

  - CORBA

    ![image-20220319201454935](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220319201454935.png)

  - .NET（与javaEE类似）

    ![image-20220319201524182](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220319201524182.png)



## 1.2 More Important

### 1.2.1 Transaction

finish or null

### 1.2.2 Security

## 1.3 DS and Algorithm

### 1.3.1 Algorithm

Concept：A well-Defined **computational procedure** that takes some value, or set of values, as input and proceduces some value, or set of values, as output.

Broadly：a step-step procedure for solving a problem or accomplishing some end  especially by a computer.

![image-20220319203553868](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220319203553868.png)

issues：

- correctness
- efficiency(amount of work done and space used)
- storage(simplicity, clarity)
- optimality

### 1.3.2 Promblem that can be solved by algorithm

- 基因工程
- 网络
- 电子商务
- 生产

一个经典的问题：you are given n integers(there may be negative ones but not all) a1, a2, ..., an, determine i and j which maximize the sum from $a_i$ to $a_j$.

- Case：6 integers：(-2, 11, -4, 13, -5, -2)
- Ans：i = 2, j = 4, max sum is 20

### 1.3.3 Importance of algorithms

问题：给1000,000个整数排序

| 数据量  | 主频   | 算法时间复杂度 | 计算时间                |
| ------- | ------ | -------------- | ----------------------- |
| 1000000 | 1GHz   | $2n^2$         | $2\times 10^5\ seconds$ |
| 1000000 | 100MHz | $50nlgn$       | $105\ seconds$          |

加之主频的提升不大，算法更为重要

### 1.3.4 About course

- Design and Analysis

  怎么找算法？

  评价算法，定量

  ![image-20220320105027537](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220320105027537.png)

  