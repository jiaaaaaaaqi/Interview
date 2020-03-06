### 隐式类型转换
1. 表达式中，低精度类型向高精度类型发生转换。
2. 条件语句中，非布尔类型向布尔类型发生转换。
3. 初始化语句中，初始值向变量类型发生转换。
4. 赋值语句中，右侧运算对象向左侧运算对象发生转换。
5. 可以用 **单个形参** 来调用的构造函数定义了从 形参类型 到 该类类型 的一个隐式转换。注意 **单个形参** 并不是只有一个形参，可以有多个形参，但其他形参要有默认实参。
```cpp
#include<iostream>
using namespace std;

class Node {
	public :
		string s1;
		int a;
		Node(string s, int val=0) : s1(s), a(val) {
		}
		bool ok(Node other) const {
			return s1 == other.s1;
		}
};

int main() {
	Node x("xxxxx");
	cout << x.ok(string("xxxxx")) << endl;  // 隐式
	cout << x.ok(Node("xxxxx")) << endl;    // 显式
	return 0;
}
```

### extern "C"
在 $C$ 和 $C$++ 混编的程序中，由于 $C$ 不存在重载机制，无法区分两个同名函数，于是引出 **extern "C"** 语句块，告诉 $C$++ 编辑器这段代码按 $C$ 标准编译，尽可能的保存 $C$ 和 $C$++ 的兼容性。

### 函数调用过程
每一个函数调用都分配一个函数栈，先将返回地址入栈，在将当前函数的栈指针入栈，然后在栈内执行函数。

### $C$++ 函数返回值和返回引用的区别
1. 函数返回值时，生成一个临时变量，返回该临时变量，然后在调用处把该临时变量赋值给左侧变量。
2. 函数返回引用时，返回和接收应是 **int&** 类型，不能返回局部变量的引用。不生成临时变量，直接返回该引用。
```cpp
#include<iostream>
using namespace std;

int x, y;

int get1() {
	cout << "get1 中 x 的地址" << &x << endl;
	return x;
}

int& get2() {
	cout << "get2 中 y 的地址" << &y << endl;
	return y;
}

int main() {
	int x = get1();
	cout << "main 中 x 的地址" << &x << endl;
	int& y = get2();
	cout << "main 中 y 的地址" << &y << endl;	
	return 0;
}
/*
get1 中 x 的地址0x4c600c
main 中 x 的地址0x6efef8
get2 中 y 的地址0x4c6010
main 中 y 的地址0x4c6010
*/
```

### $C$++中拷贝赋值函数的形参能否进行值传递？
不能，会造成无限循环。
```cpp
#include<iostream>
using namespace std;

class Node {
	public:
		int x;
		Node(int a) : x(a) {
		}
		Node(Node& a){	// 方法1，正确
			x = a.x;
		}
		Node(Node a) {	// 方法2，错误，无法编译通过
			x = a.x;
		}
};

int main() {
	Node x(10);
	Node y(x);
	return 0;
}
```
如果使用方法1，在传入 $x$ 的时候，构造函数处 $a$ 是引用，可以正常赋值。   
如果使用方法2，在传入 $x$ 的时候，构造函数处 $a$ 是形参，$a$ 需要再次调用的 $a$ 本身的构造函数，往下也是同样的道理，将造成无限构造的处境。
