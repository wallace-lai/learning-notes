# 【CppCoreGuidelines】Resource Management

作者：wallace-lai <br/>
发布：2024-02-27 <br/>
更新：2024-02-27 <br/>

资源管理主题中的资源指的是需要在代码中（显式或者隐式地）获取并释放的东西，包括但不限于**内存**、**文件描述符**、**套接字**、**锁**等资源。资源管理的目的是：

（1）避免产生资源泄露；

（2）及时释放不再需要的资源；

## 一、资源管理原则
对于资源管理，有几个总的使用原则如下所示。稍后将针对这几个总的原则给出一些细则。

### （1）使用resource handles和RAII来自动地管理资源

```cpp
void send(X* x, string_view destination)
{
    auto port = open_port(destination);
    my_mutex.lock();
    // ...
    send(port, x);
    // ...
    my_mutex.unlock();
    close_port(port);
    delete x;
}
```

上面的send是很常见的写法，但是它没有使用RAII来自动管理资源（文件描述符port、锁my_mutex）。不要这样，请使用resource handle和RAII来管理这些资源，代码如下所示。

```cpp
void send(unique_ptr<X> x, string_view destination)  // x owns the X
{
    Port port{destination};            // port owns the PortHandle
    lock_guard<mutex> guard{my_mutex}; // guard owns the lock
    // ...
    send(port, x);
    // ...
} // automatically unlocks my_mutex and deletes the pointer in x

class Port {
    PortHandle port;
public:
    Port(string_view destination) : port{open_port(destination)} { }
    ~Port() { close_port(port); }
    operator PortHandle() { return port; }

    // port handles can't usually be cloned, so disable copying and assignment if necessary
    Port(const Port&) = delete;
    Port& operator=(const Port&) = delete;
};
```

和原始版本相比，有以下几点变化：

- 使用智能指针`unique_ptr<X>`，避免了使用没有ownner的裸指针；

- 使用resource handler来处理文件描述符`PortHandle`，利用resource handler的自动构造和析构过程自动管理文件描述符port的打开与关闭；

- 使用`lock_guard`来处理锁，与智能指针类似；

这样做的好处是**程序员不再需要关心资源何时释放**的问题了，利用RAII完全可以实现资源的自动获取和释放，避免资源泄露问题的产生。

### （2）尽量不要使用原始指针进行传参

对于能用vecotr等容器替代的场景，尽量不使用原始指针。使用vector等容器，编译器可以更好地进行边界检查，避免读写越界问题的产生。

```cpp
void f1(int *p, int n)
{
    // ...
    p[2] = 7;   // bad
    // ...
}

void f2(vector<int> &p)
{
    // ...
    p[2] = 7;   // recommend
    // ...
}
```

### （3）原始的指针或者引用对所引用的对象不具备所有权，尽量不要使用

```cpp
int *p1 = new int(7);           // bad : raw owning pointer
auto p2 = make_unique<int>(7);  // good : the int is owned by a unique pointer
```

### （4）优先使用栈内存，尽量不使用动态内存分配

这条规则应该很好理解，因为动态内存分配需要支付额外的性能代价，能不用就不用

## 二、资源申请与释放规则

### 规则1：避免在C++中使用malloc和free申请和释放内存

`malloc()`和`free()`并不支持内存申请之后的额外的构造与析构操作，与C++中的`new`和`delete`不能够很好地兼容。因此，尽量避免在C++代码中使用它们。

```cpp
class Record {
    int id;
    string name;
};

// p1可能是空指针nullptr
// *p1没有被初始化，所以其中的name不是一个真正的string，只是包含了一定大小的二进制比特位
Record *p1 = static_cast<Record*>(malloc(sizeof(Record)));

// p2默认就会被初始化，除非new过程中抛出了异常
auto p2 = new Record;

// p3可能为空指针nullptr（new抛出异常），否则它必定会被初始化
auto p3 = new(nothrow) Record;

// error : 不能delete由malloc所申请的内存
delete p1;

// error : 不能free由new创建的对象
free(p2);
```

### 规则2：避免显示地调用new和delete

当你写下一个“裸露”的`new`时，意味着你在别的什么地方需要有一个同样“裸露”的`delete`。在大型程序当中，过多“裸露”的`new`和`delete`调用，往往的是BUG的来源。不要这样，使用智能指针，比如`unique_ptr`来避免显示地调用`new`和`delete`。

### 规则3：所有申请得来的资源都必须立刻交给它的管理对象

如果不这样做，一个不经意抛出的异常或者return语句都可能导致资源泄露。

```cpp
void func(const string &name)
{
    FILE *f = fopen(name, "r");         // open the file
    vector<char> buf(1024);
    auto _ = finally([f] {fclose(f);}); // remember to close the file
    // ...
}
```

上述代码中的`buf`在申请过程中可能抛出异常，于是导致文件描述符f被泄露。为了解决这个问题，可以使用file handle，即ifstream，来管理打开的文件描述符。

```cpp
void func(const string &name)
{
    ifstream f{name};
    vector<char> buf(1024);
}
```

ifstream的使用更加安全、高效，这就是RAII的威力！

### 规则4：在一条语句中最多只允许一次显式的资源申请

比如有以下的函数，如果不注意，可能会写成以下的调用方式。

```cpp
void fun(shared_ptr<Widget> sp1, shared_ptr<Widget> sp2);

// BAD : potential leak
fun(shared_ptr<Widget>(new Widget(a, b)), shared_ptr<Widget>(new Widget(c, d)));
```

上述代码中，假如在new第二个Widget的过程中抛出异常，那么第一个Widget中的内存将会泄露！

常见的解决办法是**一条语句中只进行一次显式的资源申请**，如下所示。这个方法怎么说呢，可以但显得很蠢。

```cpp
shared_ptr<Widget> sp1(new Widget(a, b));
fun(sp1, new Widget(c, d));
```

**最好的办法是使用智能指针**，根本不进行显式的资源申请操作，如下所示。

```cpp
fun(make_shared<Widget>(a, b), make_shared<Widget>(c, d));
```

哪怕第二次内存申请失败，第一次成功申请的内存依然可以由智能指针自动地释放，不会产生内存泄露。

### 规则5：避免对指针取下标，使用span代替

如下所示，建议使用`gsl::span`来代替传递数组指针的方式。

```cpp
void f(int[]);          // not recommended

void f(int*);           // not recommended for multiple objects
                        // (a pointer should point to a single object, do not subscript)

void f(gsl::span<int>); // good, recommended
```

我认为，**最好的办法其实是用容器代替传递数组指针的方式**，直接避免指针的传递。

### 规则6：把申请与释放当做不可分割的一对接口来重载

```cpp
class X {
    // ...
    void *operator new(size_t s);
    void operator delete(void *);
    // ...
};
```

即使你不希望对象可以被析构，那你也是将delete操作置上delete标记，而不是只写new而不写delete。

```cpp
void operator delete(void*) = delete;
```

## 三、智能指针

### 规则1：使用unique_ptr或者shared_ptr表示（对象的）归属

如下所示，使用智能指针，不要使用裸指针。

```cpp
void f()
{
void f()
{
    X *p1 {new X};  // bad : p1 will leak
    auto p2 = make_unique<X>(); // good : unique ownership
    auto p3 = make_shared<X>(); // good : shared ownership
}
}
```

### 规则2：优选unique_ptr，除非你有共享对象归属的需求才使用shared_ptr

从概念上来讲，unique_ptr要比shared_ptr更加简单，性能更高，行为更可预测。所以，优选unique_ptr。

```cpp
void f1()
{
    // bad example
    shared_ptr<Base> base = make_shared<Derived>();
    // 在代码局部使用base，base的引用计数不会超过1，因此应该使用unique_ptr
}
```

### 规则3：使用make_shared()创建shared_ptr

```cpp
shared_ptr<X> p1 { new X{2}};   // bad
auto p = make_shared<X>(2);     // good
```

推荐使用make_shared来创建shared_ptr，具体原因没太理解

### 规则4：使用make_unique()创建unique_ptr

利用同规则3

### 规则5：使用weak_ptr打破shared_ptr循环

具体内容在智能指针章节中叙述。

### 规则6：仅在需要表示生命周期语义时使用智能指针作为参数来传递

> Passing a smart pointer transfers or shares ownership and should only be used when ownership semantics are intended. A function that does not manipulate lifetime should take raw pointers or references instead.

只在函数有操作对象生命周期时才使用智能指针作为参数传递，否则只应该传递原始裸指针。

> Passing by smart pointer restricts the use of a function to callers that use smart pointers. A function that needs a widget should be able to accept any widget object, not just ones whose lifetimes are managed by a particular kind of smart pointer.

给函数传递智能指针会限制函数的使用，如果一个函数需要接收一个widget对象，那么理应所有的widget都是可接受的，而不是仅接受一个生命周期由智能指针所管理的对象。

> Passing a shared smart pointer (e.g., std::shared_ptr) implies a run-time cost.

传递share_ptr会增加代码运行时的开销。

### 规则7：使用第三方智能指针时，请遵循std中的智能指针模式

这个一般用在代码中引用了第三方智能指针的场景。

### 规则8：使用`unique_ptr<Widget>`作为入参来表示归属权的转移

```cpp
void sink(unique_ptr<Wdiget>);  // takes ownership of the Widget
void uses(widget*);             // just uses the Widget

// bad example, usually not what you want
void thinko(const unique_ptr<Widget> &);
```

上面的`sink`函数的入参表示Widget对象的归属权将转移到`sink`函数中；而`uses`的入参则表示`uses`函数只是使用了Wdiget对象，但它并不会造成Wdiget对象的归属权的转移。

使用`const unique_ptr<Wdiget> &`作为入参无法达成归属权的转移，因为`unique_ptr`具有移动语义，归属权会被转移到`sink`函数的局部入参中。

### 规则9：使用`unique_ptr<Wdiget>`作为入参来表示归属权的保留

```cpp
void reseat(unique_ptr<Wdiget>&);

// bad example, usually not what you want
void thinko(const unique_ptr<Widget> &);
```

因为使用引用的方式，所以`reseat`中不会创建智能指针的副本，所有对智能指针的修改都将反应的原始的指针上。

禁止使用`const unique_ptr<Widget> &`的方式代替`unique_ptr<Widget> &`。

### 规则10：Take a `shared_ptr<Widget>` parameter to express shared ownership

### 规则11：Take a `shared_ptr<Widget>&` parameter to express that a function might reseat the shared pointer

### 规则12：Take a `const shared_ptr<widget>&` parameter to express that it might retain a reference count to the object

### 规则13：Do not pass a pointer or reference obtained from an aliased smart pointer

越来越复杂了