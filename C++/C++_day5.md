### 动态内存分配
在执行程序过程中动态分配或回收存储空间的分配内存的方法。   
**C语言** 一般使用 $malloc、free$，**C++** 一般使用 $new、delete$。
1. $malloc$ 原型为 **void \*malloc(unsigned int size)**，开辟一块长度为 $size$ 连续内存空间，返回 $void$ 类型指针。失败返回 $NULL$。
2. $free$ 原型为 **void free (void\* ptr)**，释放动态分配的内存，$malloc$ 和 $free$ 要配套使用。
3. $new$ 返回分配内存单元的起始地址，需要存放在一个**指针变量**中。失败抛出异常 **bad_alloc**。
4. $delete$ 也是用来释放动态分配的内存，分为两种使用 **delete <指针变量>** 和 **delete[] <数组名>**。$new$ 和 $delete$ 要配套使用。
```cpp
int *p = (int*)malloc(sizeof(int)*100);
free(p);
int *a = new int;
delete a;
int *q = new int[100];
delete[] q;
```

### malloc/free 和 new/delete
| **malloc/free**            | **new/delete**           |
| -------------------------- | ------------------------ |
| 是标准函数库               | 是 **C**++ 运算符        |
| 从堆分配内存               | 从自由存储区分配内存     |
| 需要显式指出分配内存大小   | 编辑器自行计算           |
| 不会调用构造/析构函数      | 会调用构造/析构函数      |
| 返回无类型指针 (**void***) | 返回有类型指针           |
| 不可调用 **new/delete**    | 可以基于 **malloc/free** |
| 不可被重载                 | 可以被重载               |

### new/delete 和 new[]/delete[]
1. **new/delete** 是对分配/回收单个对象指针的处理。
2. **new[]/delete[]** 是对分配/回收对象数组指针的处理。
3. **new[]** 开辟的数组空间会多 4 字节来存放数组大小

### C++ 类的访问权限
1. $public:$ 公有的，可以被任意对象访问。
2. $protected:$ 受保护的，只允许本类和子类的成员函数访问。
3. $private:$ 私有的，只允许本类的成员函数访问。

### struct 和 class 区别
1. $struct$ 成员、继承方式默认都是 $public$ 的，$class$ 默认的方式是 $private$ 的。
2. $struct$ 不可用于声明模板类，而 $class$ 可以。
```cpp
template<class T>   // 存在
template<struct T>  // 不存在
```

### C++类内定义引用数据成员
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