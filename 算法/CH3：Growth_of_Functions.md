# CH3：Growth of Functions

Topics：

- Growth of Functions
- O/$$\Theta$$/$$\Omega$$ notations

Asymptotic Growth：

We want to express rate of growth of standard functions:

- the **leading term** with respect to n 【第一项】
- ignoring constants in front of it 【忽略常数项】

## 3.1 Asymptotic Notation

### 3.1.1 O-notation

O(g(n)) = {f(n) ：There exist **positive**  constants c  and $$n_0$$ such that  **0≤f(n) ≤cg(n)** **for all** $n≥n_0$}

- O(.) 用来渐近表示计算时间的上届 —— 【最坏情况】
- O(.) is used to bound **worest-case running time**

![image-20220321210439661](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321210439661.png)

将f(n)用简洁的O(.)来表示

![image-20220321210645750](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321210645750.png)

任何一个二次多项式都是$O(n^2)$

Note：

- When we say "the running time is O(n)"：the worest case running time is O(n)
- We often write f(n) = O(g(n)), instead of f(n) ∈ O(g(n))
- Use O(n) in equations：$$2n^2+3n+1 = 2n^2+O(n)$$



### 3.1.2 $\Omega-notation$

$$\Omega(g(n)) = \{\text{There exist positive constants c and n0 such that 0<=cg(n)<=f(n) for all n>=n0}\}$$ 【大于g（n）的整数倍】

$$\Omega-notation$$用于给出函数的下界

![image-20220321212341811](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321212341811.png)

![image-20220321213027592](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321213027592.png)

任何一个二次多项式都是$$\Omega(n^2)$$的。



### 3.1.3 $$\Theta-notation$$

$$\Theta(g(n)) = \{\text{There exist positive constants c and n0 such that } 0\leq c_1g(n)\leq f(n)\leq c_2g(n)\  \text{for all n>=n0}\}$$

$$\Theta(g(n))$$提供了函数的一个**tight bound 紧致界**

**$f(n) = \Theta(g(n))$当且仅当$$f(n) = O(g(n))$$并且$$f(n) =\Omega(g(n))$$**

![image-20220321214159832](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321214159832.png)

![image-20220321215646192](https://gitee.com/sun-yunqi/img/raw/master/pictureStore/image-20220321215646192.png)

只要最高次项，那就是$\Theta(.)$

---

Note：

- We often think of f(n) = O(g(n)) as corresponding to f(n)≤ g(n)
- f(n) = $$\Theta(g(n))$$ corresponds to f(n) = g(n)
- f(n) = $$\Omega(g(n))$$ corresponds to f(n) ≥ g(n)

---

Example：

$$4n^3 + 3n^2 + 2n + 1 = 4n^3 + 3n^2 + Θ(n)= 4n^3 + Θ(n^2) = Θ(n^3)$$



