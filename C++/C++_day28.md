### 为什么函数参数入栈顺序从右到左
为了支持 **不定长参数函数**
```cpp
int add(int num, ...) {
	va_list valist;
	int sum = 0;
    int i;
    va_start(valist, num);
    for (i = 0; i < num; i++) {
        sum += va_arg(valist, int);
    }
    va_end(valist);
    return sum;
}
```
如果是从左到右入栈，$num$ 变量将在栈底，而不定长参数需要这个 $num$ 来确定元素的个数，在栈底自然是取不出来的。所以通过从右向左入栈，可以获得不定长参数的长度。

### $C98$ 和 $C11$ 中的枚举
$C98$ 中的枚举是不限定作用域的
```cpp
enum color{red, green, blue, yellow};
enum color2{red, green};    // ERROR，因为 red 和 green 已经在 color 中定义过了
auto x = red;   //OK，因为 red 没有限定作用域
auto y = color::red;   //OK
```
$C11$ 中引入了强类型枚举，是限定作用域的
```cpp
enum struct color{red, green, blue, yellow};
enum struct color2{red, green}; // OK，red 和 green 在不同作用域内
auto x = red;   // ERROR，red 没有指定作用域
auto y = color::red;
```
强类型转换的优点在于
- 限定作用域的枚举类型将名字空间污染降低
- 限定作用域的枚举类型是强类型的，无法通过隐式转换到其他类型，而不限定的枚举类型可以自动转换为整形

### 宏定义和枚举的区别
1. 枚举是一种实体，占内存。宏定义是一种表达式，不占内存。
2. 枚举在编译阶段进行处理，宏定义在与编译阶段就完成了文本替换。

