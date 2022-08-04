# CH7：QuickSort

## 7.1 Description of QuickSort

### 7.1.1 Randomized algorithm

随机的应用：

> QuickSort
>
> rand()：产生0~1之间随机数
>
> 为什么使用随机？——没有更好的解决方法

### 7.1.2 快排的概述

分治的结果：分解时做得少，那么合并时就要做的多。反之，分解时做得多，那么合并时做得少。

Divide：利用pivot将数组分为两半，一半比pivot大，一半比pivot小

Conquer：递归求解子数组

Combine：Trival

#### 7.1.2.1 Divide and Conquer

##### （1）QuickSort

主算法：`quickSort(A, p, r)`

主要做分治，分解问题时做较多的石墙：**每次确定pivot的位置**

pivot位置确定后，分别递归求解左边和右边的数组

算法为原地排序，不需要合并

```cpp
void quickSort(vector<int>& vec, int start, int end) {
	if (start < end) {
		// 分
		int pivot_index = partition(vec, start, end);
		// 治 
		quickSort(vec, start, pivot_index-1);
		quickSort(vec, pivot_index+1, end); 
		// 原地排序，不需要合并 
	}
}
```

##### （2）Partition

注意`i`的初始化

注意各个区间的表示含义：

![image-20220607225855006](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220607225855006.png)

程序如下：

```cpp
int partition(vector<int>& vec, int start, int end) {
	// pivot的选择
	int pivot = vec[end];	// 选择最后一个元素充当pivot
	int i = start - 1;
	for (int j = start; j <= end; j++) {
		if (vec[j] <= pivot) {
			i++;
			int temp = vec[i];			// 一个比pivot大的数 
			vec[i] = vec[j];			// 和找到的小于等于pivot的数交换 
			vec[j] = temp;
		}
	} 
	return i;
}
```

有多少个比pivot小的，那么就加几次1

#### 7.1.2.2 快排的分析

假设每个元素是不一样的，现实中，如果有重复的元素，那么有更好的算法存在。

##### （1）快排的最坏情况

我们假设$T(n)$是最坏情况下的运行时间，那么：

<font color = red>**最坏情况发生在pivot是最大或者最小的元素时**</font>，此时，问题分解成两个问题，其中一个问题的规模为0，另一个问题的规模为n-1。

一次只会减少规模1.
$$
T(n) = T(n-1) + T(0) +\Theta(n)
$$
递归树如下：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220607230840358.png" alt="image-20220607230840358" style="zoom:50%;" />

##### （2）快排的最好情况

快排的最好情况发生在每次选取的pivot为中位数时，
$$
T(n) = 2T(n/2) +\Theta(n)
$$
此时，递归树如下：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220607231150669.png" alt="image-20220607231150669" style="zoom:50%;" />

时间复杂度与归并相同。

##### （3）1:9情况

假设pivot每次把数组分为1:9的两个数组，那么递归式如下：
$$
T(n) = T(9n/10)+T(n/10) +\Theta(n)
$$
此时的递归树如下：

<img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220607231529911.png" alt="image-20220607231529911" style="zoom:50%;" />

每次都除以$9/10$，那么树高为$log_{\frac{10}{9}}^{n}$

总代价小于$cnlog_{\frac{10}{9}}^{n}$，所以时间复杂度$T(n) = O(nlgn)$

##### （4）引入随机的快排

随机选择一个元素作为Pivot。

```cpp
// 引入随机
int RandomizedPartition(vector<int>& vec, int start, int end) {
	int p = rand() % (end - start + 1) + start;		// 随机选择一个作为pivot
	int temp = vec[p];
	vec[p] = vec[end];
	vec[end] = temp; 
	return partition(vec, start, end);
}
```

每次都选到最大或者最小的数的概率为：

$\frac{2}{100}\times\frac{2}{99}.......\times \frac{2}{2}$

excepted time 为$O(nlgn)$