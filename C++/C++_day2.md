### 函数指针
在编译过程中，每一个函数都有一个入口地址，而函数指针就是指向该入口地址的指针。
```cpp
#include<iostream>
using namespace std;

void fun1(int x) {
	cout << x << endl;
}

void fun2(int x) {
	cout << x+x <<endl;
}

int main() {
	void (*pf)(int);
	pf = fun1;
	pf(222);
	pf = fun2;
	pf(222);
}
```

### 多态性和虚函数(virtual)
1. 静态多态主要表现为重载，在编译时就确定了。
2. 动态多态的基础是虚函数机制，在运行期间动态绑定，决定了基类调用哪个函数。
```cpp
#include<iostream>
using namespace std;

class Shape {
	public:
        void show() {   //  未定义为虚函数
			cout << "Shape::show()" << endl;
        }
		void virtual show() {   //  定义为虚函数
			cout << "Shape::show()" << endl;
		}
};

class Line : public Shape {
	public:
		void show() {
			cout << "Line::show()" << endl;
		}
};

class Point : public Shape {
	public:
		void show() {
			cout << "Point::show()" << endl;
		}
};

int main() {
	Shape *pt;
	pt = new Line();
	pt->show();
	pt = new Point();
	pt->show();
	return 0;
}
/*
未定义为虚函数时输出结果
Shape::show()
Shape::show()

定义为虚函数时输出结果
Line::show()
Point::show()
*/
```

### 纯虚函数
有些情况下，基类生成的对象是不合理的，比如动物可以派生出狮子、孔雀等，这些派生类显然存在着较大的差异。那么可以让基类定义一个函数，并不给出具体的操作内容，让派生类在继承的时候在给出具体的操作，这样的函数被称为纯虚函数。含有纯虚函数的类成为抽象类，抽象类不能声明对象，只能用于其他类的继承。   
纯虚函数的定义方法为：
```cpp
void ReturnType Function() = 0;
```

子类可以不重写虚函数，但一定要重写纯虚函数。

### 静态函数和虚函数
静态函数在编译时就确定了调用它的时机，而虚函数在运行时动态绑定，虚函数由于用到了虚函数表和虚函数虚函数指针，会增加内存使用。

### 构造函数和析构函数
1. 构造函数在每次创建对象的时候调用，函数名称和类名相同，无返回类型，构造函数可以为类初始化某些成员。
2. 析构函数在每次删除对象的时候调用，函数名称和类名相同，但在前面加了一个 $\sim$ 符号，同样无返回类型。若对象在调用过程中用 $new$ 动态分配了内存，可以在析构函数中写 $delete$ 语句统一释放内存。
3. 如果用户没有写析构函数，编译系统会自动生成默认析构函数。
4. 假设存在继承：孙类继承父类，父类继承爷类
   1. 孙类构造过程：爷类 -> 父类 -> 孙类
   2. 孙类析构过程：孙类 -> 父类 -> 爷类

### 析构函数和虚函数
1. 可能作为继承父类的析构函数需要设置成虚函数，这样可以保证当一个基类指针指向其子类对象并释放基类指针的时候，可以及时释放掉子类的空间。
2. 虚函数需要额外的虚函数表和虚函数指针，占用额外的内存，所以不会作为继承父类的析构函数不用设置成虚函数，否则会浪费内存。
3. $C$++默认的析构函数不是虚函数，只要当其作为父类的时候，才会设置为虚函数。

### 重载和重写(覆盖)
1. 重载是指在同一访问作用域中，声明几个参数列表不同的同名函数，根据参数列表决定调用哪个函数，和函数返回值无关。
2. 重写是在派生类中重新定义的函数，其函数名、参数列表都需要和基类中被重写函数一致，并且基类中被重写函数必须是虚函数。

### 在 $main$ 函数前后执行函数
可以通过 **attribute** 关键字，声明 **constructor** 和 **destructor** 来实现。
```cpp
#include<iostream>
using namespace std;

__attribute((constructor)) void before_main() {
	cout << __FUNCTION__ << endl;
}

__attribute((destructor)) void after_main() {
	cout << __FUNCTION__ << endl;
}

int main() {
	cout << __FUNCTION__ << endl;
    return 0;
}
```
