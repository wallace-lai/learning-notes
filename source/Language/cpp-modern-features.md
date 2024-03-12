# 【C/C++】从内存角度透视现代C++的（部分）关键特性

作者：wallace-lai </br>
发布：2024-03-13 </br>
更新：2024-03-14 </br>

## 理解对象的移动

考虑下面的一个常见对象Widget：

```cpp
struct Point {
    int x;
    int y;
};

struct WidgetBase {
    double value;
};

struct Widget : WidgetBase {
    int no;
    std::string text;
    Point *data;
};
```

**（1）它的深拷贝是如何实现的？**

**（2）它的移动是如何实现的？**

未完待续

## 集合变更会有深拷贝代价