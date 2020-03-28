# C++ membery primitives
|分配|释放|类别|可否重载|
|---|---|---|---|
|malloc|free|C 函数|不可|
|new|delete|C++ 表达式| 不可|
|::operator new|::operator delete|C++ 函数|可|
|allocator< T >::allocate|allocator< T >::deallocate|C++ 标准库|可自由设计以搭配任何容器|

- 第四种方法在不同的编译版本上有着不同的使用方法
```cpp
#ifdef _MSC_VER
    int *p4 = allocator<int>().allocate(3, (int*)0);
    allocator<int>().deallocate(p4, 3);
#endif

#ifdef __BORLANDC__
    int *p4 = allocator<int>().allocate(5);
    allocator<int>().deallocate(p4, 5);
#endif

#ifdef __GNUC__
    // 在早期的 GUNC 中使用这种调用方法
    void *p4 = alloc::allocate(512);
    alloc::deallocate(p4, 512);
    // 在后期的 GUNC 中进行了进一步的优化和区分
    int *p4 = allocator<int>().allocate(7);
    allocator<int>().deallocate((int*)p4, 7);
    // 而早期的 alloc 仍然十分重要，于是换成了这种写法
    void *p5 = __gnuc_cxx::__pool_alloc<int>().allocate(9);
    __gnuc_cxx::__pool_alloc<int>().deallocate((int*)p5, 9);
#endif
```

## new/delete expression
new 会先分配内存，然后调用构造函数，所以 new 的具体实现如下，而平时重载的 new，都是重载了 ::operator new
```cpp
Complex *pc = new Complex(1, 2);
            ↓↓
try {
    void *mem = operator new(sizeof(Complex));// operator new 的实现调用了 malloc
    pc = static_cast<Complex*>(mem);
    pc->Complex::Complex(1, 2);
    // 在 GCC 中用户无法直接使用构造函数
} catch(std::bad_alloc) {

}

delete pc;
    ↓↓
pc->Complex::~Complex();
operator delete(pc);    // operaotr delete 的实现调用了 free
```
