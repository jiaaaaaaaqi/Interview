## 通过重载 $new/delete$ 来检测内存泄漏的简易实现
讲每次 $new$ 产生的内存记录，并在 $delete$ 的时候删去记录，那么最后剩下的就是发生内存泄漏的代码。
```cpp
#include <bits/stdc++.h>
using namespace std;

class TraceNew {
public:
	class TraceInfo {
	private:
		const char* file;
		size_t line;
	public:
		TraceInfo();
		TraceInfo(const char *File, size_t Line);
		~TraceInfo();
		const char* File() const;
		size_t Line();
	};
    TraceNew();
    ~TraceNew();
    void Add(void*, const char*, size_t);
    void Remove(void*);
	void Dump();
private:
    map<void*, TraceInfo> mp;
} trace;

TraceNew::TraceInfo::TraceInfo() {
}

TraceNew::TraceInfo::TraceInfo(const char *File, size_t Line) : file(File), line(Line) {
}

TraceNew::TraceInfo::~TraceInfo() {
	delete file;
}

const char* TraceNew::TraceInfo::File() const {
	return file;
}

size_t TraceNew::TraceInfo::Line() {
	return line;
}

TraceNew::TraceNew() {
    mp.clear();
}

TraceNew::~TraceNew() {
	Dump();
	mp.clear();
}

void TraceNew::Add(void *p, const char *file, size_t line) {
	mp[p] = TraceInfo(file, line);
}

void TraceNew::Remove(void *p) {
	auto it = mp.find(p);
	if(it != mp.end())	mp.erase(it);
}

void TraceNew::Dump() {
	for(auto it : mp) {
		cout << "0x" << it.first << " " << "memory leak on file: " << it.second.File() << " line: " << it.second.Line() << endl;
	}
}

void* operator new(size_t size, const char *file, size_t line) {
	void* p = malloc(size);
	trace.Add(p, file, line);
	return p;
}

void* operator new[](size_t size, const char *file, size_t line) {
	return operator new(size, file, line);
}

void operator delete(void *p) {
	trace.Remove(p);
	free(p);
}

void operator delete[](void *p) {
	operator delete(p);	
}

#define new new(__FILE__,__LINE__)

int main() {
	int *p = new int;
	int *q = new int[10];
 	return 0;
}

/*
0x0xa71850 memory leak on file: a.cpp line: 90
0x0xa719b8 memory leak on file: a.cpp line: 91
*/
```

## 垃圾回收机制
之前使用过，但现在不再使用或者没有任何指针再指向的内存空间就称为 "垃圾"。而将这些 "垃圾" 收集起来以便再次利用的机制，就被称为“垃圾回收”。

垃圾回收机制可以分为两大类：
1. 基于引用计数的垃圾回收器
   - 系统记录对象被引用的次数。当对象被引用的次数变为 $0$ 时，该对象即可被视作 "垃圾" 而回收。但难以处理循环引用的情况。
2. 基于跟踪处理的垃圾回收器
   - **标记-清除**：对所有存活对象进行一次全局遍历来确定哪些对象可以回收。从根出发遍历一遍找到所有可达对象(活对象)，其它不可达的对象就是垃圾对象，可被回收。
   - **标记-缩并**：直接清除对象会造成大量的内存碎片，所以调整所有活的对象缩并到一起，所有垃圾缩并到一起，然后一次清除。
   - **标记-拷贝**：堆空间分为两个部分 $From$ 和 $To$。刚开始系统只从 $From$ 的堆空间里面分配内存，当 $From$ 分配满的时候系统就开始垃圾回收：从$From$ 堆空间找出所有的活对象，拷贝到 $To$ 的堆空间里。这样一来，$From$ 的堆空间里面就全剩下垃圾了。而对象被拷贝到 $To$ 里之后，在 $To$ 里是紧凑排列的。接下来是需要将 $From$ 和 $To$ 交换一下角色，接着从新的 $From$ 里面开始分配。

