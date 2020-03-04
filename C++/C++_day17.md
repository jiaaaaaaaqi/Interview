### $lambda$ 表达式
$lambda$ 表达式定义一个匿名函数，并且可以捕获一定范围内的变量，基本格式如下：
```cpp
[capture](params) mutable ->ReturnType {statement}
```
- $[capture]$：捕获列表，可以捕获上下文的变量来供 $lambda$ 函数使用
  - $[var]$：值传递的方式捕获 $var$
  - $[\&var]$：引用传递的方式捕获 $var$
  - $[=]$：值传递的方式捕获父作用域的所有变量。
  - $[\&]$：引用传递的方式捕获父作用域的所有变量。
  - $[this]$：值传递的方式捕获当前 $this$ 指针。
- $(params)$：参数列表，和普通函数参数列表一致，如果不传参数可以和 $()$ 一起忽略。
- $mutable$ 修饰符号，默认情况下，$lambda$ 表达式是一个 $const$ 函数，可以用 $mutable$ 取消他的常量性。若使用 $mutable$ 修饰，参数列表不可省略。
- **->**$ReturnType$：返回值类型。若是 $void$，可以省略。
- ${statement}$：函数主体，和普通函数一样。

$lambda$ 表达式优点在于代码简洁，容易进行并行计算。缺点在于在非并行计算中，效率未必有 $for$ 循环快，并且不容易调试，对于没学过的程序员来说可读性差。

```cpp
#include<bits/stdc++.h>
using namespace std;

int main() {
    vector<int> g{9, 2, 1, 2, 5, 6, 2};
    int ans = 1;
    sort(g.begin(), g.end(), [](int a, int b) ->bool{return a>b;});
    for_each(g.begin(), g.end(), [&ans](int x){cout << x << " ", ans = ans*x;});
    cout << "\nmul = " << ans << endl;
    return 0;
}
/*
9 6 5 2 2 2 1
mul = 2160
*/
```

### 存在全局变量和局部变量时，访问全局变量
可以通过全局作用符号 $::$ 来完成
```cpp
int a = 10;

int main() {
	// freopen("in", "r", stdin);
	int a = 20;
	cout << a << endl;
	cout << ::a << endl;
	return 0;
}
```

### 全局变量的初始化的顺序
同一文件中的全局变量按照声明顺序，不同文件之间的全局变量初始化顺序不确定。

### 浅拷贝和深拷贝
- 浅拷贝：源对象和拷贝对象相互独立。对其中一个对象修改，不会影响另一个对象。
- 深拷贝：源对象和拷贝对象共用一份实体，仅仅是引用的变量名不同。对其中任意一个修改，都会影响另一个对象。
- 两个对象指向同块内存，当析构函数释放内存时，会引起错误。

从这个例子可以看出，$b$ 通过默认拷贝函数进行初始化，然而进行的是浅拷贝，导致对 $a$ 进行修改的时候，$b$ 的存储值也被修改。

```cpp
struct Node {
	char* s;
	Node(char *str) {
		s = new char[100];
		strcpy(s, str);
	}
	/*	手动实现拷贝构造函数
	Node(const Node &other) {
		s = new char[100];
		strcpy(s, other.s);
	}
	*/
};

int main() {
	// freopen("in", "r", stdin);
	Node a("hello");
	Node b(a);
	cout << b.s << endl;
	strcpy(a.s, "bad copy");
	cout << b.s << endl;
	return 0;
}
```
正确的写法应该自己写一个拷贝函数，而不是用默认的，应该尽量的避免浅拷贝。

