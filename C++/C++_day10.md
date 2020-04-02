## 四种 $cast$ 转换
四种 $cast$ 转换为：$const\_cast、static\_cast、dynamic\_cast、reinterpret\_cast$。
1. $const\_cast：$ 用于将 $const$ 变量转换成非 $const$。
2. $static\_cast：$ 用于各种隐式转换，比如非 $const$ 转 $const$、$void*$ 转指针等，能用于多态中向上转化，向下转化能成功但结果未知。
3. $dynamic\_cast：$ 动态类型转换，只能用于含有虚函数的类，用于类层次间的向上和向下转化，只能转指针或引用。向下转换时，如果非法，对于指针返回 $NULL$，对于引用抛出异常。
4. $reinterpret\_cast：$ 几乎什么都可以转，但可能会出问题。

$C$ 语言的强制转化看起来功能强大，但转化不够准确，不能进行错误检查，不安全。

## 四种智能指针
四种智能指针：$auto\_ptr、unique\_ptr、shared\_ptr、weak\_ptr$，第一种已经被 $C$++11 弃用。
- $unique\_ptr$(替换 $auto\_ptr$) 实现独占式拥有概念。保证同一时间只有一个 $unique\_ptr$ 指针指向某个内存，对于避免内存泄漏很有用。
- $shared\_ptr$ 实现共享式拥有概念。多个智能指针可以指向相同对象，该对象和其相关资源会在 **"最后一个引用被销毁"** 时释放。
  - 两个对象互相使用一个 $shared\_ptr$ 成员变量指向对方，会造成循环引用，使得两个对象都不会自动释放，造成内存泄漏。如下例子，可以看出析构函数没有被调用。为了解决这个问题，引入了 $weak\_ptr$。

```cpp
#include<bits/stdc++.h>
using namespace std;

class A;
class B;
class A {
public:
	A() {
		cout << "A Created" << endl;
	}
	~A() {
		cout << "A Destroyed" << endl;
	}
	shared_ptr<B> ptr;
};
class B {
public:
	B() {
		cout << "B Created" << endl;
	}
	~B() {
		cout << "B Destroyed" << endl;
	}
	shared_ptr<A> ptr;
};

int main() {
	shared_ptr<A> pt1(new A());
	shared_ptr<B> pt2(new B());
	pt1->ptr = pt2;
    pt2->ptr = pt1;
    cout << "use of pt1: " << pt1.use_count() << endl;
    cout << "use of pt2: " << pt2.use_count() << endl;
    return 0;
}
/*
A Created
B Created
use of pt1: 2
use of pt2: 2
*/
```
- $weak\_ptr$ 是一种不控制对象生命周期的智能指针，它与一个 $shared\_ptr$ 绑定，却不参与引用计数。一旦最后一个 $shared\_ptr$ 销毁，对象就会释放。
  - $weak\_ptr$ 作用是在需要的时候变出一个 $shared\_ptr$，其他时候不干扰 $shared\_ptr$ 的引用计数。
  - $weak\_ptr$ 没有 * 和 -> 符号，需要用 $lock()$ 获得 $shared\_ptr$，进而使用对象。
  - 利用 $weak\_ptr$ 可以消除上面的循环引用，例子如下。

```cpp
#include<bits/stdc++.h>
using namespace std;

class A;
class B;
class A {
public:
	A() {
		cout << "A Created" << endl;
	}
	~A() {
		cout << "A Destroyed" << endl;
	}
	weak_ptr<B> ptr;
};
class B {
public:
	B() {
		cout << "B Created" << endl;
	}
	~B() {
		cout << "B Destroyed" << endl;
	}
	weak_ptr<A> ptr;
};

int main() {
	shared_ptr<A> pt1(new A());
	shared_ptr<B> pt2(new B());
	pt1->ptr = pt2;
    pt2->ptr = pt1;
    cout << "use of pt1: " << pt1.use_count() << endl;
    cout << "use of pt2: " << pt2.use_count() << endl;
    return 0;
}
/*
A Created
B Created
use of pt1: 1
use of pt2: 1
B Destroyed
A Destroye
*/
```

因为存在这种情况：申请的空间在函数结束后忘记释放，造成内存泄漏。使用智能指针可以很大程度的避免这个问题，因为智能指针是一个类，超出类的作用范围后，类会调用析构函数释放资源，所以智能指针的作用原理就是在函数结束后自动释放内存空间。