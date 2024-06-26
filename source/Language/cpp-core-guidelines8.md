# 【CCG】性能

作者：wallace-lai <br/>
发布：2024-02-25 <br/>
更新：2024-03-06 <br/>

真正的问题是，程序员们花了太多的时间在错误的地方和错误的时间里担心效率问题；过早优化是编程中的万恶之源（或者至少是其中大部分罪恶之源）。

———《计算机程序设计艺术》 高德纳

## 一、错误的优化

### 规则1：不要无故优化

### 规则2：不要过早进行优化

### 规则3：不要优化并非性能关键的东西

总之，过早优化是编程中的万恶之源。在进行任何性能假设之前，请使用最关键的规则来进行性能分析，即：**测量程序的性能**。

没有性能数据，你如何确定以下问题的答案：

（1）程序的哪一部分是瓶颈？

（2）对用户来说，多快算好？

（3）程序能有多快？

须知，在错误（性能）假设的基础上进行优化是一个典型的反模式。

## 二、错误的假设

### 规则4：不要假设复杂的代码一定比简单的快

### 规则5：不要假设低级代码一定比高级代码快

### 规则6：不要在没有测量的情况下对性能妄下断言

以上规则都是为了应对错误的（性能）假设，各种对性能想当然的臆测很可能都是错误的。

## 三、启用优化

可以使用Compiler Explorer工具来查看代码生成的汇编指令，这是查比较代码性能好坏的决定性因素。

### 规则7：设计应当允许优化

这条规则尤其适用于移动语义（move），因为你写算法时应该使用移动语义，而不是拷贝语义。使用移动语义会带来一些好处：

（1）算法使用低开销的移动语义，而不是高开销的拷贝语义；

（2）算法要稳定地多，因为它不需要内存分配，所以不会出现`std::bad_alloc`异常；

（3）可将算法运用于只能移动的类型，比如`std::unique_ptr`；

若在一个只能拷贝的类型上应用移动操作，会触发拷贝操作。拷贝语义是移动语义的一种回退。

类型的六大操作：

（1）默认构造函数

（2）析构函数

（3）拷贝构造函数

（4）移动构造函数

（5）拷贝赋值运算符

（6）移动赋值运算符


### 规则10：依赖静态类型系统

有许多方法可以帮助编译器生成更优化的代码。

（1）写本地代码

一般来说，如果使用就地调用的lambda表达式来调整`std::sort`的行为，代码会更快。因为编译器拥有所有可用的信息来生成最优代码。相反，函数可能定义在另一个翻译单元中，这对优化器来说是一个无法跨越的硬边界（优化器无法获取函数的所有可用信息）。

<p style="color:red;">
那么问题来了，如何函数和其调用在同一个翻译单元，此时函数调用和lambda表达式有性能上的差异吗？为什么？
</p>

（2）写简单代码

优化器会搜寻可以被优化的已知模式。如果你的代码是手写的且极为复杂，这会使优化器在寻找已知模式的工作变得更加困难。于是，你得到的是没有被充分优化的代码。

<p style="color:red;">
所以不要再自作聪明地手动进行循环展开优化了！
</p>

（3）给予编译器额外的提示

比如：

- 当函数不能抛出异常，或者你不关心异常时，将它声明为`noexcept`；

- 如果一个虚函数不应该被覆盖，那么可以将其声明为`final`；

### 规则11：将计算从运行期移至编译期

```cpp
constexpr int gcd(int a, int b)
{
    while (b != 0) {
        auto t = b;
        b = a % b;
        a = t;
    }

    return a;
}
```

通过将函数`gcd`声明为`constexpr`，`gcd`就变成了一个可在编译期执行的函数。想要在编译期执行，对于函数`gcd`的要求是：

（1）`gcd`不能使用static或者thread_local变量

（2）`gcd`不能使用异常处理

（3）`gcd`不能使用goto语句

（4）`gcd`的所有入参都必须初始化为字面值类型

注意：将`gcd`声明为`constexpr`函数，并不表示它必须在编译期运行，它只意味着`gcd`可能在编译期运行。

如果要在常量表达式中使用`constexpr`函数，那么要求函数必须在编译期执行。

```cpp
constexpr auto res1 = gcd(121, 11);

auto val = 121;
constexpr auto res2 = gcd(val, 11); // error
```

- `res1`是常量，因为`gcd(121, 11)`可以在编译期求值且使用了`constexpr`修饰；

- `res2`不是常量，主要是因为`val`没有用`constexpr`修饰，不是常量表达式，导致`gcd(val, 11)`无法在编译期求值；

总结：`constexpr`函数可以在运行期间求值，也可以在编译期求值。**如果要使其在编译期求值，则其参数必须是常量表达式**。

### 规则12：以可预测的方式访问内存

这条规则说的是缓冲命中问题。在现代CPU架构中，一个缓存行通常为64字节大小。所以，像`std::vector`这种将数据存储在一个连续的内存块中的数据结构对缓存行很友好。这种友好性也适用于`std::array`和`std::string`。

