## 有 $static、virtual$ 之类的一个类的内存分布
- $static$ 修饰成员变量
  - 静态成员变量在 **全局存储区** 分配内存，本类的所有对象共享，在还没生成类对象之前也可以使用。
- $static$ 修饰成员函数
  - 静态成员函数在 **代码区** 分配内存。静态成员函数和非静态成员函数的区别在于非静态成员函数存在 $this$ 指针，而静态成员函数不存在，所以静态成员函数没有类对象也可以调用。
- $virtual$ 
  - 虚函数表存储在常量区，也就是只读数据段
  - 虚函数指针存储在对象内，如果对象是局部变量，则存储在栈区内。

## 使用宏定义求结构体成员偏移量
```cpp
#include<bits/stdc++.h>
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE*)0)->MEMBER)
/*
(TYPE*)0 将零转型成 TYPE 类型指针
((TYPE*)0->MEMBER) 访问结构体中的成员
&((TYPE*)0->MEMBER) 取出数据成员地址，也就是相对于零的偏移量
(size_t) & ((TYPE*)0)->MEMBER) 将结果转换成 size_t 类型。
*/

struct Node {
	char a;
	short b;
	double c;
	int d;
};

int main() {
	printf("%d\n", offsetof(Node, a));
	printf("%d\n", offsetof(Node, b));
	printf("%d\n", offsetof(Node, c));
	printf("%d\n", offsetof(Node, d));
	return 0;
}
```
$size\_t$ 在可以理解成 $unsigned\ \ int$，在不同平台下被 $typedef$ 成不同类型。

## $C$++ 中哪些函数不可以是虚函数
1. 普通函数(非成员函数)：普通函数不属于类成员，不能被继承。普通函数只能被重载，不能被重写，因此声明成虚函数没有意义。
2. 构造函数：构造函数本来就是为了初始化对象而存在的，没有定义为虚函数的必要。而且对象还没构造出来，不存在虚函数指针，也无法成为虚函数。
3. 内联成员函数：内联函数是为了在代码中直接展开，减少调用函数的代价，是在编译的时候展开。而虚函数需要动态绑定，这不可能统一。只有当编译器知道调用的是哪个类的函数时，才可以展开。
4. 静态成员函数：对于所有对象都共享一个函数，没有动态绑定的必要。
5. 友元函数：$C$++ 不支持友元函数的继承，自然不能是虚函数。

## 手撕堆排序
[堆排序的思路以及代码的实现
](https://blog.csdn.net/qq_36573828/article/details/80261541)

令 $k = log_2n$

初始化堆复杂度 $O(N)$ 。分析：假设每一次都到叶子的父节点
$$
\begin{aligned}
	S &= 2^0k + 2^1(k-1)+2^2(k-2)+...+2^{k-2}1\\
	2S &= 2^1k + 2^2(k-1)+2^3(k-2)+...+2^{k-1}1 \\
	S &= 2^{k-1} + 2^{k-2} + ... + 2^2 + 2^1 - 2^0k \\
	  &= 2^k - k + 2 \\
	  &= n - logn + 2
\end{aligned}
$$

排序重建堆复杂度 $O(NlogN)$。分析：假设每一次都到叶子节点。
$$
\begin{aligned}
	S &= logn + log(n-1) + log(n-2) + ... + log2 \\
	  &= logn! < logn^n \\
	  &= nlogn
\end{aligned}
$$
```cpp
void adjust(vector<int>& nums, int i, int n) {
	while(2*i+1 < n) {
		int j = 2*i+1;
		if(j+1<n && nums[j]<nums[j+1])	j++;
		if(nums[i] > nums[j])	break;
		swap(nums[i], nums[j]);
		i = j;
	}
}

vector<int> sortArray(vector<int>& nums) {
	if(nums.size() <= 1)	return nums;
	int n = nums.size();
	for(int i=n/2-1; i>=0; i--)	adjust(nums, i, n);	// 初始化堆
	for(int i=n-1; i>0; i--) {	// 排序重建堆
		swap(nums[i], nums[0]);
		adjust(nums, 0, i);
	}
	return nums;
}
```

## $main$ 函数前后还会执行什么
全局对象的构造在 $main$ 函数之前完成，全局对象的析构在 $main$ 函数之后完成。

## $const$ 和 $define$
1. $define$ 在预编译阶段起作用，$const$ 在编译、运行的时候起作用。
2. $define$ 只是简单的替换功能，没有类型检查功能，$const$ 有类型检查功能，可以避免一些错误。
3. $define$ 在预编译阶段就替换掉了，无法调试，$const$ 可以通过集成化工具调试。

## $inline$ 和 $define$
1. $inline$ 在编译时展开，$define$ 在预编译时展开。
2. $inline$ 可以进行类型安全检查，$define$ 只是简单的替换。
3. $inline$ 是函数，$define$ 不是函数。
4. $define$ 最好用括号括起来，不然会产生二义性，$inline$ 不会。
5. $inline$ 是一个建议，可以不展开，$define$ 一定要展开。

## $inline$ 函数的要求
1. 含有递归调用的函数不能设置为 $inline$
2. 循环语句和 $switch$ 语句，无法设置为 $inline$
3. $inline$ 函数内的代码应很短小。最好不超过５行