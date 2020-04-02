## 动态内存分配
在执行程序过程中动态分配或回收存储空间的分配内存的方法。   
$C$ 一般使用 $malloc、free$，$C$++ 一般使用 $new、delete$。
1. $malloc$ 原型为 **void \*malloc(unsigned int size)**，开辟一块长度为 $size$ 连续内存空间，返回 $(void*)$ 类型指针。失败返回 $NULL$。
2. $free$ 原型为 **void free (void\* ptr)**，释放动态分配的内存，$malloc$ 和 $free$ 要配套使用。
3. $new$ 返回分配内存单元的起始地址，需要存放在一个**指针变量**中。失败抛出异常 $bad\_alloc$。
4. $delete$ 也是用来释放动态分配的内存，分为两种使用 $delete <指针变量>$ 和 $delete[] <数组名>$。$new$ 和 $delete$ 要配套使用。
```cpp
int *p = (int*)malloc(sizeof(int)*100);
free(p);
int *a = new int;
delete a;
int *q = new int[100];
delete[] q;
```

## A* a = new A; a->i = 10; 在内存分配上发生了什么?
1. A* a，$a$ 是一个局部变量，类型为指针，所以操作系统会开辟 $4$ 字节的空间分配给指针 $a$。
2. new A，通过 $new$ 在堆区申请类 $A$ 大小的空间。
3. a = new A，将指针 $a$ 的内存区域中填入在栈中申请到的类 $A$ 的地址的地址。
4. a->i = 10，先找到 $a$ 的地址，然后通过 $a$ 存储的地址和 $i$ 在类 $A$ 中的偏移量得到 a->i 的地址，然后在该地址内完成赋值操作。

## $malloc$ 原理
$malloc$ 通过两种方法分配内存
1. 当分配内存小于 $128K$ 时，调用 $brk$ 完成，从堆顶往高地址分配对应的内存。
2. 当分配内存大于 $128K$ 时，调用 $mmap$ 完成，在堆和栈中间找到一块空闲的虚拟内存分配。

$malloc$ 的分配都是在虚拟地址上的分配，具体分配的物理内存由操作系统决定。

## $malloc/free$ 和 $new/delete$
| $malloc/free$            | $new/delete$           |
| ------------------------ | ---------------------- |
| 是标准函数库             | 是 $C$++ 运算符        |
| 从堆分配内存             | 从自由存储区分配内存   |
| 需要显式指出分配内存大小 | 编译器自行计算         |
| 不会调用构造/析构函数    | 会调用构造/析构函数    |
| 返回无类型指针 ($void*$) | 返回有类型指针         |
| 不可调用 $new/delete$    | 可以基于 $malloc/free$ |
| 不可被重载               | 可以被重载             |

## $new/delete$ 和 $new[]/delete[]$
1. $new/delete$ 是对分配/回收单个对象指针的处理。
2. $new[]/delete[]$ 是对分配/回收对象数组指针的处理。
3. $new[]$ 开辟的数组空间会多 $4$ 字节来存放数组大小

## $C$++ 类的访问权限
1. $public:$ 公有的，可以被任意对象访问。
2. $protected:$ 受保护的，只允许本类和子类的成员函数访问。
3. $private:$ 私有的，只允许本类的成员函数访问。

## $struct$ 和 $class$ 区别
1. $struct$ 成员、继承方式默认都是 $public$ 的，$class$ 默认的方式是 $private$ 的。
2. $struct$ 不可用于声明模板类，而 $class$ 可以。
```cpp
template<class T>   // 存在
template<struct T>  // 不存在
```
## $struct$ 和 $union$ 区别
1. $union$ 是共用体，任何时刻都只存放一个被选中的成员，而结构体存放所有成员变量。
2. 对共用体不同的成员赋值，其他对象也被改变，原本的值就不存在了，而结构体不同成员并不影响。
3. $union$ 内存分配按照结构内部最大成员，结构体遵循字节对齐原则。

## $C$++类内定义引用数据成员
1. 不能使用默认的构造函数，必须提供构造函数。
2. 必须在初始化列表中初始化，不能再构造函数内初始化。
3. 构造函数的形参也必须是引用类型。

**构造函数分为初始化和计算两个阶段，第一阶段对应初始化列表，第二阶段对应函数主体，引用必须在第一阶段完成。**
```cpp
#include <iostream>
using namespace std;

class Node {
public:
	int& a;
	int b, c, d;
	Node(int &x, int y, int z, int k) : a(x), b(y), c(z), d(k) {
	}
};

int main() {
	int t = 1;
	Node x(t, 2, 3, 4), y(t, 2, 3, 4);
	x.a++;
	cout << y.a << endl;
	return 0;
}
```
