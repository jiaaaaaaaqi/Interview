### $vector$ 释放内存
$vector$ 在插入大量数据后，即使把数据全部删除，但并没有改变容器的容量，仍然会占用内存。

通过 $swap$ 操作来释放 $vector$ 的内存，原理是 $vector()$ 使用 $vector$ 的默认构造函数建立了临时对象，使用 $swap$ 操作让原本对象和临时对象交换内存，$swap$ 完成后，临时对象就是容量非常大的 $vector$， 接着临时对象消失，释放内存。
```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
	vector<int> g;
	for(int i=1; i<=10; i++)
		g.push_back(i);
	cout << "capacity = " << g.capacity() << endl;
	vector<int>().swap(g);
	cout << "capacity = " << g.capacity() << endl;
	return 0;
}
```

### $STL$ 利用迭代器删除元素
1. 对于关联容器 $map、set$，删除当前的迭代器仅仅会使当前迭代器失效。因为内部使用红黑树实现，所以只要在 $erase$ 之前递增当前迭代器即可。
2. 对于序列容器 $vector、deque$，删除当前迭代器会使后面所有元素迭代器都失效。因为内部是连续分配的内存，删除一个元素会使后面的所有元素向前移动一个位置。但 $erase$ 可以返回下一个有效的迭代器。
3. 对于 $list$，它使用了不连续分配的内存，并且 $erase$ 也会返回下一个有效的迭代器，所以两种方法都可行。
```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
	{
		set<int> st = {1, 2, 3, 4, 5, 6};
		for(set<int>::iterator iter=st.begin(); iter!=st.end(); ) {
			if(*iter == 3) {
				st.erase(iter++);   // 传给 erase 的是 iter 的一个副本
			} else {
				iter++;
			}
		}
	}
	{
		vector<int> g = {1, 2, 3, 4, 5, 6};
		for(vector<int>::iterator iter=g.begin(); iter!=g.end(); ) {
			if(*iter == 3) {
				iter = g.erase(iter);
			} else {
				iter++;
			}
		}
	}
	return 0;
}
```

### $STL\ allocator$
在 $C$++ 的内存配置和释放操作中
- $new$ 运算分为两个阶段：1. 调用 **::operaotr new** 分配一个对象的内存。 2. 调用该对象的构造函数。
- $delete$ 运算也分为两个阶段：1. 调用对象的析构函数。 2. 调用 **::operaotr delete** 释放内存。

而 $allocator$ 将这两个阶段分开，用四个函数实现
- $alloc::allocate()$ 负责内存配置
- $::construct()$ 负责对象构造
- $::destory()$ 负责对象析构
- $alloc::deallocate()$ 负责内存释放

为了提高内存管理效率，减少申请小内存产生的内存碎片，$SGI\ STL$ 设计了双层级配置器
- 当分配内存超过 $128B$ 时，使用第一层配置器，第一层配置器直接使用 $malloc()、free()$ 进行内存的分配和释放。
- 当分配内存小于 $128B$ 时，使用第二层配置器，第二层配置器采用内存池技术、通过空闲链表来管理内存。 

### $C$++11 特性
1. $auto$ 关键字，编译器可以通过初始值自动推导出类型，但不能用于函数传参和数组类型的推导。
2. $nullptr$ 关键字，在 $C$++ 中 $NULL$ 被宏定义为 $0$，在遇到重载时可能有问题，而 $nullptr$ 一直是一个空指针，可以转换成其他任意类型的指针类型。
3. 智能指针，新增了 $share\_ptr、weak\_ptr$ 等智能指针，用于解决内存管理。
4. 初始化列表，可以使用初始化列表对类初始化。
5. 右值引用，可以实现移动语句和完美转发，消除两个对象交互时不必要的拷贝，提高效率。
6. 新增 $STL$ 容器 $array$ 和 $tuple$。
   1. $array$ 是封装固定大小数组的容器，支持传统数组的随机访问，还支持迭代器访问、获取容量等，并且不会退化成指针。
   2. $tuple$ 是多元组容器。

### $C$++11 可变参数模板
$C$++ 11 的可变参数模板，可以表示任意数目，任意类型的参数，语法为在 $class$ 或 $typename$ 后加上省略号。
```cpp
#include <bits/stdc++.h>
using namespace std;

void print() {
	cout << endl;
}

template<class T, class... Args>
void print(T num, Args... rest) {
	cout << num << " ";
	print(rest...);
}

int main() {
	print(1, 2, 3, 4, 5);
    return 0;
}
```
