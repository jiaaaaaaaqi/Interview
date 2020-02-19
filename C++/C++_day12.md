### $shared\_ptr$ 指针的实现
```cpp
template<typename T> class Shared_ptr {
private:
	T *ptr;
	int *use_count;	// 用指针保证指向同一块地址
public:
	Shared_ptr() {
		ptr = nullptr;
		use_count = nullptr;
		cout << "Created" << endl;
	}
	Shared_ptr(T *p){
		ptr = p;
		use_count = new int(1);
		cout << "Created" << endl;
	}
	Shared_ptr(const Shared_ptr<T> &other) {
		ptr = other.ptr;
		++(*other.use_count);
		use_count = other.use_count;
		cout << "Created" << endl;
	}
	Shared_ptr<T> operator=(const Shared_ptr &other) {
		if(this == &other)	return *this;
		(*other.use_count)++;
		if(ptr && --(*use_count) == 0) {
			delete ptr;
			delete use_count;
		}
		ptr = other.ptr;
		use_count = other.use_count;
		return *this;
	}
	T& operator*() {	// 返回指针的引用
		return *ptr;
	}
	T* operator->() {
		return ptr;
	}
	int get_use_count() {
		if(use_count == nullptr)	return 0;
		return *use_count;
	}
	~Shared_ptr() {
		if(ptr && --(*use_count) == 0) {
			delete ptr;
			delete use_count;
			cout << "Destroy" << endl;
		}
	}
};
```

### 红黑树
[面试常问：什么是红黑树?](https://blog.csdn.net/qq_36610462/article/details/83277524)

