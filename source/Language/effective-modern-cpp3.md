# 【EMC】第四章：智能指针

作者：wallace-lai </br>
发布：2024-02-28 </br>
更新：2024-02-28 </br>

C++中的裸指针功能强大，但稍有不慎极易伤及自身。裸指针有以下这些常见的可怕之处：

（1）它的声明无法指示它到底是个对象还是个数组

```cpp
int *p1 = new int;
int *p2 = new int[16];  // 对象与数组无法区分
```

（2）它的声明没有告诉你是否应该销毁它，即指针是否拥有所指之物（即指针所指对象的归属权是否属于该指针）

```cpp
void f(int *p)
{
    // use p
    delete p    // 我应该销毁该指针所指对象吗？不知道
}
```

（3）如果你决定你应该销毁指针所指向对象，没人告诉你该用delete还是其它的析构机制


（4）接着（3），如果是用delete销毁，是delete单个对象还是delete数组，无法确定

```cpp
void f(int *p)
{
    // use p
    // 使用 delete 还是其它的析构机制？不知道
    // 即使确定使用delete，那么是delete单个对象还是delete数组？不知道
}
```

（5）即使你能够正确销毁指针所指对象，但你很难确定你**在所有执行路径上（包括异常路径）都恰好执行了一次销毁操作**。少一次路径可能会导致泄露，而多销毁一次会导致未定义行为

（6）一般来讲没有办法告诉你指针是否变成了悬空指针

```cpp
void f(int *p)
{
    // use p
    // ...
}

int *p = new int;

// use p

delete p;

f(p);   // p已经成为了悬空指针，但是f不知道
```

综上所述：裸指针很危险，极容易造成内存泄露和各种未定义行为，我们需要智能指针

## 条款18：对于独占资源使用unique_ptr

未完待续

## 条款22：当使用Pimpl手法时，请在实现文件中定义特殊成员函数

### Pimpl手法

以下是一个常见的`Widget`类的实现。

```cpp
// Widget.h
#include <string>
#include <vector>
#include "gadget.h"

class Widget {
public:
    Widget();
    // ..

private:
    std::string name;
    std::vector<double> data;
    Gadget g1, g2, g3;  // Gadget是用户自定义类型
};
```

因为`Widget`依赖了`std::string`和`std::vector`以及`Gadget`，所以在编译`Widget`的时候必须将这些头文件包含进来。

（1）首先，这增加了`Wdiget`的编译耗时；

（2）其次，任何一个头文件的内容变动，都将导致类`Widget`的重新编译

为了解决编译耗时的问题，我们可以使用Pimpl手法，如下所示：

```cpp
// Widget.h
class Widget {
public:
    Widget();
    // ..

private:
    struct Impl;
    Impl *pImpl;
};
```

类`Widget`此时不再依赖`std::string`和`std::vector`以及`Gadget`，所以`Widget`的使用者可以不需要因为头文件内容的改动而重新编译程序。类`Widget`的真正实现放在了Widget.cpp中，如下所示。

```cpp
// Widget.cpp
#include "Widget.h"
#include "gadget.h"
#include <vector>
#include <string>

struct Widget::Impl {
    std::string name;
    std::vector<double> data;
    Gadget g1, g2, g3;
};

Widget::Widget() : pImpl(new Impl) {}

Widget::~Widget() { delete pImpl; }
```

然而上述代码是有问题的，可以使用智能指针优化一下。

```cpp
// Widget.h
class Widget {
public:
    Widget();
    // ..

private:
    struct Impl;
    std::unique_ptr<Impl> pImpl;    // 使用智能指针而不是原始指针
};
```

```cpp
// Widget.cpp
#include "Widget.h"
#include "gadget.h"
#include <vector>
#include <string>

struct Widget::Impl {
    std::string name;
    std::vector<double> data;
    Gadget g1, g2, g3;
};

Widget::Widget() : pImpl(std::make_unique<Impl>()) {}
```

此时，析构函数不见了，因为`std::unique_ptr`在自身析构的同时会自动销毁它所指向的对象。
