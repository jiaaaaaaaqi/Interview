## 引用 和 $const$ 引用
引用是另一个 **变量** 的别名，必须初始化且一旦定义就不可改变。

$const$ 引用和引用不同的点在于 $const$ 引用是指向 **常量** 的引用。
```cpp
const int x = 10;
const int &y = x;
const int &z = 20;
```
如此一来，由于 $x$ 是 $const$ 的，不可被改变，将 $y$ 也声明成 $const$ 的，防止了 $x$ 的值通过 $y$ 被改变。$const$ 引用也可以直接指向一个常量。

## 左值和右值
**左值：** 指向某内存空间的表达式，并且可以通过 **&** 获得其地址。

**右值：** 非左值即右值。

## 右值引用
实现了转移语义和精确传递。可以消除两个对象交互时不必要的对象拷贝，提高效率，可以更简明的定义泛型函数。

转移语义可以将资源从一个对象转移到另一个对象上，减少不必要的临时对象的创建、拷贝以及销毁。

精确传递就是将一组参数 **原封不动** 的传递给另一个函数，**原封不动** 不仅仅是 **参数值** 不变，还有 **左值/右值** 和 **const/not-const**。
```cpp
#include<bits/stdc++.h>
using namespace std;

void func(int& x) {
	cout << "左值引用" << endl;
}
void func(int&& x) {
	cout << "右值引用" << endl;
}
void func(const int& x) {
	cout << "const 左值引用" << endl;
}
void func(const int&& x) {
	cout << "const 右值引用" << endl;
}

template<typename T> void fun(T&& x) {
	func(forward<T>(x));
}

int main() {
	fun(10);
	
	int x = 0;
	fun(x);
	fun(move(x));
	
	const int y = 0;
	fun(y);
	fun(move(y));
    return 0;
}
/*
右值引用
左值引用
右值引用
const 左值引用
const 右值引用
*/
```

## 左值引用和右值引用的区别
1. 左值引用表达为 **int&**，右值引用表达为 **int&&**。
2. 左值持久，右值短暂。除了 $const$ 引用，左值引用绑定到左值，右值引用只能绑定到将要销毁的对象。
