# CH6：HeapSort

## MindMap

![image-20220804203355365](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220804203355365.png)

Topics：

- Heaps
- HeapSort
- Priority queue

## Recap and Overview

强调稳定性时多关注的是多关键字排序。

- 插入排序

  时间复杂度是$O(n^2)$，原地排序

- 归并排序

  时间复杂度是$O(nlgn)$，需要辅助存储空间

- 堆排序

  时间复杂度是$\Theta(nlgn)$，原地排序

- 快排

  平均情况的时间复杂度是$O(nlgn)$，原地排序

- 线性时间的排序

## 6.1  Heap

### 6.1.1 堆存储和堆数据结构

#### （1）堆内存的使用

堆的使用：

- 内存的分类

$$
\begin{cases}栈内存：局部变量，方法和函数\\
堆内存：malloc\ new\\
静态方法区：全局变量\end{cases}
$$

#### （2）堆数据结构的使用

堆数据结构：

- 元素按照下标索引——堆数据结构的实现是一个一维数组

- 每个元素支持：`PARENT`，`LEFT`和`RIGHT`操作

- 父子关系的映射逻辑为：

  若**数组下标从1开始**，那么父子映射关系函数为：

  - $PARENT(i) = \lfloor i/2 \rfloor$

  - $LEFT(i) = 2i$

  - $RIGHT(i) = 2i+1$

    <img src="https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220607092832741.png" alt="image-20220607092832741" style="zoom:50%;" />

  若**数组下标从0开始**，那么父子映射关系函数为：

  - $PARENT(i) = \lceil i/2 \rceil -1$

  - $LEFT(i) = 2i+1$

  - $RIGHT(i) = 2i+2$

    ```cpp
    int getParent(int child) {
    	return ceil(child / 2.0) - 1;
    }
    
    int getLeft(int parent) {
    	return parent * 2 + 1;
    }
    
    int getRight(int parent) {
    	return parent * 2 + 2;
    }
    ```

- 堆中父子的大小关系
  $$
  \begin{cases}大顶堆：A[PARENT(i)]\geq A[i]\\
  小顶堆：A[PARENT(i)]\leq A[i]\end{cases}
  $$

- 堆中的第一个元素是$A[0]$或者$A[1]$，对应两种不同的情况

  结点的高度是从该结点到叶子结点的最长路径。

  **堆的高度是树根的高度**。

  **叶子结点的高度为0，树根的高度为树的高度**。

  有n个结点的树的高度为$\lfloor lgn\rfloor$

#### （3）时间复杂度概述

`MAX-HEAPIFY`：保证堆是一个大顶堆，时间复杂度是$O(lgn)$

`Build-Max-Heap`：建堆，时间复杂度是$O(n)$

堆排序：堆排序时，我们首先要建成一个大顶堆，然后反复执行，取下结点后调用`MAX-HEAPIFY`的过程，时间复杂度是$O(nlgn)$

### 6.1.2 堆的建立

建堆两种方式：递归建堆和非递归建堆

#### （1）递归建堆

递归建堆三个步骤：分、治、合并

- 分：分解成子树
- 治：递归处理左子树和右子树
- 合并：`MaxHeapify`

```cpp
void buildMaxHeap(int parent, vector<int>& vec) {
	if (parent < vec.size()) {// 考虑是否还有孩子结点
		// 分 
		int left = getLeft(parent);
		int right = getRight(parent);
		// 治 
		buildMaxHeap(left, vec);
		buildMaxHeap(right, vec);
		// 合并 
		maxHeapify(vec, parent);
	}	
}
```

#### （2）非递归建堆

非递归建堆过程采用循环的建堆方式，从最后一个非叶子结点开始，逐步建堆

最后一个非叶子结点的下标：$\lfloor vec.size()/2\rfloor$

```cpp
void buildMaxHeap_iteration(vector<int>& vec) {
	int last_non_leaf = vec.size() / 2;
	for (int i = last_non_leaf; i >= 0; i--) {
		maxHeapify(vec, i);
	}
} 
```

![image-20220607145156611](https://cdn.jsdelivr.net/gh/Holmes233666/blogImage@main/img/image-20220607145156611.png)

#### （3）合并过程：`MAXHEAPIFY`

`MaxHeapify()`时间复杂度为$O(h)$，其中$h$是树的高度，$h = \lfloor lgn \rfloor $

- 递归写法

写成递归处理的形式如下：

```cpp
void maxHeapify(vector<int>& vec, int parent) {
	int largest = parent;
	int left = getLeft(parent);
	int right = getRight(parent);
	// 找到父子之间最大值的位置 
	if (left < vec.size() && vec[left] > vec[largest]) {		// 注意短路判决 
		largest = left;
	}
	if (right < vec.size() && vec[right] > vec[largest]) {    	// 注意短路判决 
		largest = right;
	}
	// 更换位置
	if (largest != parent) {
		int temp = vec[largest];
		vec[largest] = vec[parent];
		vec[parent] = temp;
		// 接着maxheapify处理子树
		maxHeapify(vec, largest); 
	} 
}
```

- 循环写法

上述的写法是在结束的时候调用递归处理，这种写法很容易改成循环的形式。如下：

```cpp
void maxHeapify(vector<int>& vec, int parent) {
    int largest = parent;
    bool ifChange = true;
    while(ifChange) {
        int left = getLeft(parent);
		int right = getRight(parent);
        // 找到父子之间最大值的位置 
        if (left < vec.size() && vec[left] > vec[largest]) {		// 注意短路判决 
            largest = left;
        }
        if (right < vec.size() && vec[right] > vec[largest]) {    	// 注意短路判决 
            largest = right;
        }
        // 更换位置
        if (largest != parent) {		// 需要交换
            int temp = vec[largest];
            vec[largest] = vec[parent];
            vec[parent] = temp;
            ifChange = true;
        }else{
            ifChange = false;
        }
    }
}
```

#### （4）运行时间分析

若采取直接的方式分析运行时间，那么时间将会是：
$$
T(n) = O(n/2)\times O(lgn) = O(nlgn)
$$
<font color = "red">**key observation：**</font>

$$
T(n) = 2T(n/2) + O(lgn)
$$
$f(n) = lgn = O(n^{log_2^2-\epsilon})=O(n^{1-\epsilon})$

所以堆排序的时间复杂度为$\Theta(n)$

### 6.1.3 堆排序

堆排序策略：每次取出堆顶的元素，使用堆中最后一个元素与堆顶元素交换，然后调用`MaxHeapify`

```cpp
void HeapSort(vector<int>& vec) {
	int size = vec.size();	// 从0开始的 
	for (int i = size - 1; i >= 1; i--) {	// 从最后一个结点开始 
		int temp = vec[i];
		vec[i] = vec[0];
		vec[0] = temp;
		size = size - 1;
		maxHeapify(vec, 0, size);
	}
	
}
```

- 时间复杂度：$O(nlgn)$
- 原地排序 inplace
- 不稳定排序

## 6.2 PriorityQueue

### 6.2.1 应用

最大值优先队列的应用：

- 在共享计算机上调度工作
- 实现优先队列

---

最小值优先队列应用：

- 迪杰特斯拉算法
- Prim 最小生成树算法
- 哈夫曼编码

### 6.2.2 优先队列

操作：

- 插入一个元素`Insert(S, x)`

- 获得最大值`MAXIMUM(S)`

- 取出最大值`EXTRACT-MAX(S)`

- `INCREASE-KEYS(S, x, k)`

  将位置x的元素正大为key

#### （1）获得最大元素

```cpp
MAXMUM(A)
    return A[0];
```

#### （2）去除最大元素

- 先检查溢出
- 最后一个元素与第一个元素交换
- maxHeapify调整

```cpp
int PriorityQueue::extract_max() {
	if (vec.size() < 1) {
		cout << "Error：HEAP UNDERFLOW" << endl;
		return 0;
	} 
	int max = vec[0];	// 需要返回的值 
	vec[0] = vec[vec.size() - 1];	// 与最后一个元素交换
	vec.pop_back();		// 弹出最后一个元素 
	maxHeapify(vec.size(), 0);
	return max;
}
```

#### （3）增大元素

- 首先判断是否是增大
- 一直和父亲换，直到父亲的值更大

```cpp
void PriorityQueue::increase_key(int index, int key) {
	if (key < vec[index]) {
		cout << "Error：" <<key << "is smaller than the original number " << vec[index] << endl;
	}
	vec[index] = key;
	while (index > 0 && vec[parent(index)] < vec[index]) {
		int temp = vec[index];
		vec[index] = vec[parent(index)];
		vec[parent(index)] = temp;
		index = parent(index); 
	}
}
```

#### （4）插入元素

- 堆大小加一
- 将末尾元素设置为无穷小
- 调用增大元素的函数进行处理

```cpp
void PriorityQueue::insert(int key) {
	// 放到最后
	vec.push_back(key); 
	// 向上比较调整
	increase_key(vec.size()-1, key); 
}
```





