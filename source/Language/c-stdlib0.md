# 【C标准库】stdlib

作者：wallace-lai <br/>
发布：2018-02-11 <br/>
更新：2024-04-02 <br/>

## qsort

### 函数原型

```c
void qsort(
    void *base,
    size_t nmemb,
    size_t size,
    int (*compar)(const void *, const void *)
    );
```

函数功能：对数组`base`进行排序，数组有`nmemb`个元素，每个元素大小为`size`；

（1）`base`：指向数组的起始地址，通常该位置传入的是一个数组名；

（2）`nmemb`：表示该数组的元素个数；

（3）`size`：表示该数组中每个元素的大小（字节数）；

（4）`compar`：指向比较函数的函数指针，决定了排序的依据；

函数返回值：无

注意：如果两个元素的值是相同的，那么它们的前后顺序是不确定的。也就是说`qsort`是一个不稳定的排序算法。

### compar参数
`compar`参数指向一个比较两个元素的函数。比较函数的原型如下所示。注意两个形参必须是`const void *`型，在`compar`函数内部会将`const void *`型转换成实际类型。

```c
int compar(const void *p1, const void *p2);
```

（1）如果`compar`返回值小于0，那么p1所指向元素会被排在p2所指向元素的前面；

（2）如果`compar`返回值等于0，那么p1所指向元素与p2所指向元素的顺序不确定；

（3）如果`compar`返回值大于0，那么p1所指向元素会被排在p2所指向元素的后面；

因此，如果想让`qsort`进行从小到大（升序）排序，那么一个通用的`compar`函数可以写成这样：

```c
int compareMyType (const void * a, const void * b)
{
    if ( *(MyType*)a <  *(MyType*)b ) return -1;
    if ( *(MyType*)a == *(MyType*)b ) return 0;
    if ( *(MyType*)a >  *(MyType*)b ) return 1;
}
```

### 使用案例
```c
/* qsort example */
#include <stdio.h>      /* printf */
#include <stdlib.h>     /* qsort */

int values[] = { 40, 10, 100, 90, 20, 25 };

int compare (const void * a, const void * b)
{
    return ( *(int*)a - *(int*)b );
}

int main ()
{
    int n;
    qsort (values, 6, sizeof(int), compare);
    for (n=0; n<6; n++)
        printf ("%d ",values[n]);

    return 0;
}
```


