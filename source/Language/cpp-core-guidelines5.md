# 【CCG】枚举

作者：wallace-lai <br/>
发布：2024-02-25 <br/>
更新：2024-02-25 <br/>

枚举通常用于定义整数值集以及定义此类值集的类型，C++中有两种枚举，分别是普通的enum和enum class。

## 规则1：优选枚举而不是宏定义

宏定义和枚举相比，缺少作用域和类型方面的限制。宏定义在预处理阶段就会被处理掉，不会出现在诸如调试器这样的工具当中。

```cpp
// webcolors.h (third party header)
#define RED   0xFF0000
#define GREEN 0x00FF00
#define BLUE  0x0000FF

// productinfo.h
// The following define product subtypes based on color
#define RED    0
#define PURPLE 1
#define BLUE   2

int webby = BLUE;   // webby == 2; probably not what was desired
```

以上代码显示了使用宏定义的一大缺陷：**在不同头文件中定义相同名称的宏可能导致你引用出错**。相反，如果你使用enum class，这样的问题就不会出现。

```cpp
enum class WebColor {
    red = 0xFF0000,
    green = 0x00FF00,
    blue = 0x0000FF
};

enum class ProductInfo {
    red = 0,
    purple = 1,
    blue = 2
};

int webby = blue;   // error : be specific
WebColor webby = WebColor::blue;
```

## 注：C++中的enum和enum class

在C++中，`enum`和`enum class`都是用于定义枚举类型的关键字，但它们之间存在一些重要的区别。

1. **作用域**：`enum`定义的枚举类型中的枚举值具有**全局作用域**，而`enum class`定义的枚举类型中的枚举值具有**局部作用域**。也就是说，如果你使用`enum`定义了一个枚举类型，那么你可以在任何地方直接使用枚举值，而如果你使用`enum class`定义了一个枚举类型，那么你必须使用枚举类型名作为前缀来引用枚举值。

例如：
```cpp
enum Color { RED, GREEN, BLUE };
enum class ColorClass { RED, GREEN, BLUE };

// 使用enum
Color myColor = RED;

// 使用enum class
ColorClass myColorClass = ColorClass::RED;
```

2. **类型转换**：`enum`定义的枚举类型与整数类型之间可以进行隐式转换，而`enum class`定义的枚举类型与整数类型之间则不能进行隐式转换，只能进行显式转换。这可以防止因类型不匹配而导致的错误。

例如：


```cpp
enum Color { RED, GREEN, BLUE };
enum class ColorClass { RED, GREEN, BLUE };

Color color = RED;
int intColor = color;  // 隐式转换，这是可以的

ColorClass colorClass = ColorClass::RED;
int intColorClass = colorClass;  // 错误：不能进行隐式转换
int intColorClassCorrect = static_cast<int>(colorClass);  // 正确：进行显式转换
```

**注：限制隐式类型转换而允许你显示转换，目的是让你脑子里对转换过程有根弦，避免在你感知不到的地方发生错误**！

3. **枚举值的默认类型**：`enum`定义的枚举值的默认类型是`int`，而`enum class`定义的枚举值的默认类型是枚举类型本身。

4. **枚举值的范围**：`enum`定义的枚举值的范围与`int`相同，而`enum class`定义的枚举值的范围则与底层类型相同，可以通过指定底层类型来改变枚举值的范围。

例如：
```cpp
enum Color : char { RED, GREEN, BLUE };  // 枚举值的范围与char相同
```

总的来说，`enum`和`enum class`各有其优点和适用场景。`enum`在C++的早期版本中使用较多，但由于其作用域和类型转换的特性可能导致一些问题，**因此在现代的C++代码中，更推荐使用`enum class`来定义枚举类型**。

## 规则2：使用枚举来表示相关的命名常量
使用枚举来表示相应的命名常量后，在使用switch语句的过程中，编译器可以自动检查出关于case分支的一些异常情况。

```cpp
enum class ProductInfo {
    red = 0,
    purple = 1,
    blue = 2
};

void print(ProductInfo info)
{
    switch (info) {
        // 使用enum class使得编译器可以检查一些不寻常的case分支
        switch (info) {
            case ProductInfo::red :
                cout << "red" << endl;
                break;
            case ProductInfo::purple :
                cout << "purple" << endl;
                break;
        }
    }

    return;
}
```

比如对于上面的代码，编译器将给出如下的警告。这样就可以在编译过程中发现一些潜在的错误。

```
warning: enumeration value ‘blue’ not handled in switch [-Wswitch]
```

## 规则3：优选enum class而不是enum

具体原因在enum和enum class之间的不同这小节中已经说明了。概括来说是enum class在类型转换和作用域方面比enum有更加严格的限制，更加安全。

## 规则4：为了更加安全和方便，请在枚举之上定义操作函数

如下所示，请将操作定义在枚举之上（**前提是它适合定义成枚举**）：

```cpp
enum class Day {
    Mon,
    Tue,
    Wed,
    Thu,
    Fri,
    Sat,
    Sun
};

Day &operator++(Day &d)
{
    return d = (d == Day::Sun) ? Day::Mon :
        static_cast<Day>(static_cast<int>(d) + 1);
}
```

注意上面的代码，尽管多次使用static_cast让代码变得不够美观，但是如果不使用则会得到一个错误。

```cpp
Day &operator++(Day &d)
{
    return d = (d == Day::Sun) ? Day::Mon :
        Day{++d};   // error
}
```

## 规则5：禁止在枚举中使用全大写定义

```cpp
 // webcolors.h (third party header)
#define RED   0xFF0000
#define GREEN 0x00FF00
#define BLUE  0x0000FF

// productinfo.h
// The following define product subtypes based on color

enum class Product_info { RED, PURPLE, BLUE };   // syntax error
```

这条规则的目的主要是**为了避免和宏定义产生命名冲突**，因为宏定义是使用全大写的形式，所以禁止枚举也使用全大写的形式。

## 规则6：避免使用匿名枚举

如果你不对枚举进行命名，那么枚举值就无法和名字关联起来。尽量避免下面这种匿名枚举类型的出现。

```cpp
// bad example
enum {
    red = 0xFF0000,
    scale = 4,
    is_signed = 1
};
```

如果无法避免，那么请考虑使用`constexpr`进行替代：

```cpp
constexpr int red = 0xFF0000;
constexpr short scale = 4;
constexpr bool is_signed = true;
```

## 规则7：只在需要时指定枚举值类型

C++中enum枚举值的默认类型是int，int是最方便用于读写的类型，这和C语言也是兼容的。只在有特殊需要的时候指定枚举值类型，除此之外一律使用默认的int类型。

```cpp
// 为了节省内存空间
enum class Direction : char {
    n, s, e, w,
    ne, nw, se, sw
};

// 多余的类型指定，默认就是int32_t
enum class WebColor : int32_t {
    red = 0xFF0000,
    green = 0x00FF00,
    blue = 0x0000FF,
};
```

注意：在以下这种前置声明的场景下需要指定enum的类型

```cpp
enum Flags : char;

void f(Flags);

// ...

enum Flags : char {
    // ...
}
```

## 规则8：只在需要时指定枚举的具体值

```cpp
enum class Col1 {
    red,
    yellow,
    blue
};

enum class Col2 {
    red = 1,
    yellow = 2,
    blue = 2
};
```

和C语言一样，不指定的情况下，C++从0开始连续不重复地分配枚举值。因此，只在有特殊需要的时候才需要指定枚举的具体值。