# 【CppCoreGuidelines】源文件

作者：wallace-lai <br/>
发布：2024-02-25 <br/>
更新：2024-03-05 <br/>

## 一、接口与实现

### 规则1：如果项目没有其他约定，那么源文件使用cpp后缀，头文件使用h后缀

除非你的项目已经有了其它的后缀命名策略，否则一律按照规则1来处理。

### 规则2：头文件中不可包含对象定义或者非内联函数定义

如果你的头文件里含有**对象**或者**非内联函数**的定义，则可能导致链接步骤报错。实际上，在C++中，有所谓的单一定义规则ODR。ODR在函数定义方面有以下几点说法。

（1）一个函数在任何翻译单元中不能有一个以上的定义；

（2）一个函数在程序中不能有一个以上的定义；

（3）有外部链接属性的内联函数可以在一个以上的翻译单元中被定义，这些定义必须满足一个要求：它们全相同；

注意：将**类**定义在头文件中，并不会引起链接器的不满，比如下面这个案例。

```cpp
// header.h

class Base {
    Base() {}
    ~Base() {}

    void FormatBase(void) {
        cout << "FormatBase" << endl;
    }
};
```

```cpp
// t1.cpp
#include "header.h"
void fun1(void)
{
    Base b;
    b.FormatBase();

    return;
}
```

```cpp
// t2.cpp
#include "header.h"
void func2(void)
{
    Base b;
    b.FormatBase();
    return;
}

int main()
{
    Base b;
    b.FormatBase();

    return 0;
}
```

同理，内联函数和类一样，也是可以被定义在头文件中的。

### 规则5：.cpp源文件必须包含其接口定义的头文件.h

如果不这么做，则会在链接过程中出问题，比如下面的案例：

```cpp
// impl.h

int func(int);
```

```cpp
// impl.cpp
std::string func(int x)
{
    return to_string(x + 32);
}
```

此时源文件`impl.cpp`并没有包含对应的头文件`impl.h`，并且函数`func`的声明和定义中的返回值类型不一致。

```cpp
// main.cpp

int main()
{
    int result = func(32);
    cout << result << endl;

    return 0;
}
```

在主函数中调用`func`，则会得到一个`undefined reference`的错误。

```
main.cpp:(.text+0x12): undefined reference to `func(int)'
```

如果遵循了规则5，那么类似这种浅显的错误就可以在编译过程中暴露出来，不用等到链接过程。

### 规则8：为所有的.h头文件使用宏防止出现多重包含

给所有的头文件都添加宏，用于防止出现多重包含的情况。

```cpp
// footer.h
#ifndef FOOTER_H_
#define FOOTER_H_

// ...

#endif /* FOOTER_H_ */
```

有两点需要注意的：

（1）你应该保证不同的头文件的防护宏的名字应该是唯一的；

（2）使用`#pragma once`预处理指令来解决多重包含是不可移植的，因为`#pragma`指令并未标准化；

### 规则9：避免源文件之间循环依赖

比如下面这个案例就是典型的头文件之间循环依赖，无法通过编译。

```cpp
// a.h
#pragma once

#include "b.h"
class A {
    B b;
};
```

```cpp
// b.h
#pragma once

#include "a.h"
class B {
    A a;
};
```

```cpp
// main.cpp

#include "a.h"
#include "b.h"

int main()
{
    A a;
    B b;

    return 0;
}
```

编译报错为：

```
b.h:6:5: error: ‘A’ does not name a type
```

解决办法是**前置所依赖类的声明并使用指针**，如下所示：

```cpp
// a.h
#pragma once

class B;

class A {
    B *b;
};
```

### 规则10：避免对隐含#include进来的名字的依赖

比如下面这个案例在GCC 5.4和微软编译器19.00.23506版本中进行编译，前者没问题；后者无法通过编译。

```cpp
#include <iostream>

int main()
{
    std::string s = "hello world";
    std::cout << s << std::endl;

    return 0;
}
```

原因在于GCC 5.4编译器的`iostream`头文件中包含了`string`；而微软编译器中的`iostream`没有。我们需要手动包含`string`以摆脱代码对隐含名字的依赖。

### 规则11：头文件应该是自包含的

应该说是在头文件能够做到自包含的情况下，尽量达成头文件的自包含。

## 二、命名空间

### 规则16：（仅）对代码迁移、基础程序库（比如std）或者在局部作用域中使用using namespace指令

下面的案例将无法通过编译，原因在于局部变量`sqrt`和标准库中的`sqrt`函数重名了。

```cpp
#include <cmath>
using namespace std;

int g(int x)
{
    int sqrt = 7;
    // ...
    return sqrt(x);
}
```

不仅如此，using namespace指令隐藏了名称的来源，且破坏了代码的可读性。只允许在以下三种场景下使用using namespace指令：

（1）代码迁移（这是什么？）；

（2）基础库std；

（3）局部作用域中；

### 规则17：不要在头文件的全局作用域中使用using namespace

头文件中全局作用域的using namespace会将名字注入到包括该头文件的每个源文件中。这种注入有一些不好的后果：

（1）当你使用这个头文件时，你无法再撤销其中的using指令；

（2）名字冲突的可能性急剧增加；

（3）对被包含的namespace的改变可能会破坏你的编译，比如因为其中引入了一个新的名字（导致冲突）；

### 规则20：使用namespace表示逻辑结构

### 规则21：不要在头文件中使用匿名命名空间

### 规则22：为所有的内部（不导出的实体）使用匿名命名空间

匿名namespace使用的是内部链接。而内部链接意味着匿名namespace内的名称只能在当前翻译单元内引用，而不能导出。不能导出这一点同样适用于在匿名namespace中声明的名称。

而当你在头文件中使用匿名namespace时（规则21），每个翻译单元都定义了这个无名namespace的唯一实例，这会导致：

（1）产生的可执行文件大小会膨胀

（2）匿名namespace中的任何声明都将是不同翻译单元中的不同实体，这可能不是程序员期望的行为

匿名namespace的用法类似C语言里使用的static关键字。

