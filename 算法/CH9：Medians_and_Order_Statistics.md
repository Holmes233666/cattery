# CH9：Medians and Order Statistics

## 9.1 Order Statistics

### 9.1.1 背景介绍

**OrderStatistics 排位统计**：集合中第i小的元素

中位数：快排中找中位数，代价大

寻找集合中第i小的元素：

- 最大：i = n
- 最小：i = 1
- 中位数<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608143214854.png" alt="image-20220608143214854" style="zoom:33%;" />

我们可以多快解决这个问题？

- 最大或者最小：O(n)
- 排序：O(nlgn)

下面尝试用O(n)解决

### 9.1.2 Randomized algorithm for finding the i th element 随机算法

#### （1）算法思路

算法思路：

- 利用带有随机的partition算法确定pivot的位置，按照pivot位置和要找的元素的排名对对应的部分进行递归

  ![image-20220608143235496](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608143235496.png)

- 终止条件：区间长度为1或者是pivot位置就是rank位置

- **递归找左还是右不要搞错**

算法实现：

```cpp
int randomizedSelect(vector<int>& vec, int start, int end, int rank) {
	if (start == end) return vec[start];
	int pivot_index = randomizedPartition(vec, start, end);
	int pivot_rank = pivot_index - start + 1;
	if (pivot_rank == rank) {
		return vec[pivot_index];
	}
	if (rank < pivot_rank) {
		return randomizedSelect(vec, start, pivot_index - 1, rank);				// 左边找 
	}else{
		return randomizedSelect(vec, pivot_index + 1, end, rank - pivot_rank);	// 右边找 
	}
}
```

#### （2）算法分析

最坏情况下时间复杂度：$T(n) = T(n-1) +\Theta(n) = \Theta(n^2)$

![image-20220608143314381](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608143314381.png)

与快排区别，**快排是两个子问题**。

### 9.1.3 对于快排的优化

对于快排的时间复杂度，若找中值是线性开销，那么：
$$
T(n) = 2T(n/2) + O(n) + O(n)
=\Theta(nlgn)
$$
若找中值是最坏情况：
$$
T(n) = 2T(n/2) + O(n) +O(n^2)
$$

---

线性的找中值：

![image-20220608143326408](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220608143326408.png)

每5个一组，快速通过直接插入排序找到中值，每组中值的中值作为pivot。
$$
T(n) = T(n/5) + \Theta(n) + T(7n/10) = \Theta(n)
$$
