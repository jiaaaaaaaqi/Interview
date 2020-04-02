## 通过位运算实现加减乘除取模
- 加法操作

对于每一位而言，在不考虑进位的情况下，可以得到
$$
    0+0 = 0\\
    0+1 = 1\\
    1+0 = 1\\
    1+1 = 0
$$
显然，上面的情况符合 **异或** 操作且只有第四种情况发生了进位，进位情况符合 **与** 操作。在所有发生进位处，应该在更高的一位处加一，这个值可以通过 **左移** 操作实现。那么就可以得到
$$
    x+y = x \oplus y + (x \& y)<<1
$$
可以发现，后面的式子变成了一个新的加法式，那么只要递归计算即可。当 $(x \& y)<<1 == 0$ 时，就消除了加法式。
```cpp
ll add(ll x, ll y) {
	ll newx = x^y;
	ll newy = (x&y)<<1;
	if(newy == 0)	return newx;
	return add(newx, newy);
}
```

- 减法操作

减法操作可以看成 
$$
\begin{aligned}
    -y &= \sim y+1\\
    x+y & = x+(-y)\\
        & = x + \sim y+1
\end{aligned}
$$
同样可以通过加法式得到
```cpp
ll sub(ll x, ll y) {
	return add(x, add(~y, 1));
}
```

- 乘法操作

假设 $y=1010$，则可以关注于二进制上的 $1$ 位，那么可以将 $x*y$ 做出拆解
$$
\begin{aligned}
    x*y &= x*1010\\
        &= x*1000 + x*0010    
\end{aligned}
$$
而这个当乘数只有一个 $1$ 时，可以通过二进制的左移操作实现。
```cpp
ll mul(ll x, ll y) {
	ll ans = 0;
	int flag = x^y;
	x = x<0 ? add(~x, 1) : x;
	y = y<0 ? add(~y, 1) : y;
	for(int i=0; (1ll<<i)<=y; i++) {
		if(y&(1ll<<i)) {
			ans = add(ans, x<<i);
		}
	}
	return flag<0 ? add(~ans, 1) : ans;
}
```

- 除法操作

和乘法操作思想一样，枚举答案每一位是否为 $1$，通过左移来得到乘积并减去。先从大的开始找，如果有一位是 $1$，那么就在答案将这一位设成 $1$。
```cpp
ll divide(ll x, ll y) {
	ll ans = 0;
	int flag = x^y;
	x = x<0 ? add(~x, 1) : x;
	y = y<0 ? add(~y, 1) : y;
	for(int i=31; i>=0; i--) {
		if(x >= (1ll*y<<i)) {
			ans |= (1ll<<i);
			x = sub(x, 1ll*y<<i);
		}
	}
	return flag<0 ? add(~ans, 1) : ans;
}
```

- 取模操作

已经得到了除法的结果，那么取模操作也很容易实现了
```cpp
ll mod(ll x, ll y) {
	x = x<0 ? add(~x, 1) : x;
	y = y<0 ? add(~y, 1) : y;
	return sub(x, mul(y, divide(x, y)));
}
```

## 为什么子类对象可以赋值给父类对象而反过来却不行
- 子类继承于父类，它含有父类的部分，又做了扩充。如果子类对象赋值给父类变量，则使用该变量只能访问子类的父类部分。
- 如果反过来，这个子类变量如果去访问它的扩充成员变量，就会访问不到，造成内存越界。

## 为什么 $free$ 时不需要传指针大小
$free$ 要做的事是归还 $malloc$ 申请的内存空间，而在 $malloc$ 的时候已经记录了申请空间的大小，所以不需要传大小，直接传指针就可以。
