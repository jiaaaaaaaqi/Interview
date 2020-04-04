## 如何控制一个类只能在堆或栈上创建对象
在 $C$++ 中创建对象的方法有两种，一种是静态建立，一个是动态建立。
- 静态建立由编译器为对象分配内存，通过调用构造函数实现。这种方法创建的对象会在栈上。
- 静态建立由用户为对象分配内存，通过 $new$ 来实现，间接调用构造函数。这种方法创建的对象会在堆上。

**只能从堆上分配对象：**

当建立的对象在栈上时，由编译器分配内存，因此会涉及到构造函数和析构函数。那么如果无法调用析构函数呢？也就是说析构函数是 $private$ 的，编译器会先检查析构函数的访问性，由于无法访问，也就防止了静态建立。

但这种方法存在一种缺点，就是把析构函数设成 $private$ 后，如果这个类要作为基类的话，析构函数应该设成虚函数，而设成 $private$ 后子类就无法重写析构函数，所以应该把析构函数设成 $protected$。然后额外设置一个接口来 $delete$。
```cpp
class Node {
public:
	Node(){};
	void Destroy() {
		delete this;
	}
protected:
	~Node(){};
};
```
此时解决了静态建立的过程，但使用时，通过 $new$ 创建对象，通过 $Destroy$ 函数释放对象，为了统一，可以把构造函数和析构函数都设成 $protected$，重写函数完成构造和析构过程。
```cpp
class Node {
public:
	static Node* Create() {
		return new Node();
	}
	void Destroy() {
		delete this;
	}
protected:
	Node(){};
	~Node(){};
};
```

**只能从栈上分配对象：**

同样的道理，只需要禁止通过动态建立对象就可以实现在栈上分配对象，所以可以重载 $new$ 和 $delete$ 并设为 $private$，使用户只能静态建立对象。
```cpp
class Node {
public:
	Node(){};
	~Node(){};
private:
	void* operator new(size_t t){}
	void operator delete(void* p){}
};
```

## $memcpy$ 和 $memmove$ 的实现
$memcpy$ 可以直接通过指针自增赋值，但要求源地址和目的地址无重合。
```cpp
void mymemmove1(void* s, const void* t, size_t n) {
	char *ps = static_cast<char*>(s);
	const char *pt = static_cast<const char*>(t);
	while(n--) {
		*ps++ = *pt++;
	}
}
```

如果源地址和目的地址存在重合，会因为地址的重合导致数据被覆盖，所以要通过 $memmove$ 来实现，需要从末尾往前自减赋值。

为了加快速度还可以使用 $4$ 字节赋值的方式

```cpp
// 直接按字节进行 copy
void mymemmove1(void* s, const void* t, size_t n) {
	char *ps = static_cast<char*>(s);
	const char *pt = static_cast<const char*>(t);
	if(ps<=pt && pt<=ps+n-1) {
		ps = ps+n-1;
		pt = pt+n-1;
		while(n--) {
			*ps-- = *pt--;
		}
	} else {
		while(n--) {
			*ps++ = *pt++;
		}
	}
}

// 加快速度，每次按 4 字节进行 copy
void mymemmove2(void *s, const void *t, size_t n) {
	int *ts = static_cast<int*>(s);
	const int *tt = static_cast<const int*>(t);
	char *ps = static_cast<char*>(s);
	const char *pt = static_cast<const char*>(t);
	int x = n/4, y = n%4;
	if(ps<=pt && pt<=ps+n-1) {
		ps = ps+n-1;
		pt = pt+n-1;
		while(y--) {
			*ps-- = *pt--;
		}
		ps++, pt++;
		ts = reinterpret_cast<int*>(ps);
		tt = reinterpret_cast<const int*>(pt);
		ts--, tt--;
		while(x--) {
			*ts-- = *tt--;
		}
	} else {
		while(y--) {
			*ps++ = *pt++;
		}
		ts = reinterpret_cast<int*>(ps);
		tt = reinterpret_cast<const int*>(pt);
		while(x--) {
			*ts++ = *tt++;
		}
	}
}
```
