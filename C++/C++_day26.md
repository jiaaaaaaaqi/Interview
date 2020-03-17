### 通过重载 $new/delete$ 来检测内存泄漏的简易实现
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