# 【CCG】C风格编程

作者：wallace-lai <br/>
发布：2024-02-25 <br/>
更新：2024-03-04 <br/>

## 规则1：优先使用C++而不是C

这是因为：

（1）C++提供比C更好的类型检查（更安全）和更多的写法支持（更灵活）；

（2）C++为高级编程语言提供了更好的支持，并往往生成更高效的代码；

## 规则2：如果必须使用C，请使用C和C++的公共子集，并以C++的方式编译C代码

- **如果有完整的C源代码可用**

这个时候基本没什么大问题，唯一可能需要修改的是C源代码中不是C和C++公共子集的那部分代码。

- **如果没有完整的C源代码**

此时，需要遵循以下几点原则：

（1）使用C++编译`main`函数；

主要是因为C++编译器会生成一些在`main`函数启动之前执行的代码（比如调用全局（静态）对象的构造函数），这个时候你只能选择C++编译器。

（2）使用C++编译器链接程序；

（3）使用同一供应商的C和C++编译器，它们应该具有相同的调用约定；

这个主要是**为了保证ABI的兼容性**，比如函数调用时参数的分配顺序、参数的传递、由调用方还是被调方处理函数栈等等。

## 规则3：如果必须使用C作为接口，则在调用此类接口的代码里使用C++

与C相比，C++支持函数重载。这意味着可以定义名字相同但参数不同的函数。为了支持函数重载，C++编译器会将函数参数的类型和数目也编码到函数名称中，这个过程被称为“名字重编”。并且每个C++编译器都可能有自己的特定编码方式，因为这一过程没有被标准化。

为了在C++中调用C函数，可以使用`extern "C"`链接说明符，用于告知C++编译器不要将这些函数名重编。使用链接说明符后，你既可以在C++中调用C函数，也可以在C函数中调用C++函数。

具体可行的实践方法如下：

（1）在每个函数前加说明符

```cpp
extern "C" void foo(int);
```

（2）一个作用域内的每个函数

```cpp
extern "C" {
    void foo(int);
    double bar(double);
}
```

（3）整个头文件，当使用C++编译器时，宏`__cplusplus`将被定义

```cpp
#ifdef __cplusplus
extern "C" {
#endif

void foo(int);
double bar(double);

#ifdef __cplusplus
}
#endif
```

在工程实践中，方法（3）是最常见的。

