# 【C语言最佳实践】为性能编码

作者：wallace-lai <br>
发布：2022-08-04 <br>
更新：2022-08-04 <br>


## 一、什么是性能

性能的两层含义：

（1）空间复杂度（越低越好）

（2）时间复杂度（越低越好）

优化性能：空间复杂度和时间复杂度的平衡

简单案例：编写函数返回一个字节中位值为一的位的个数

（1）版本一

没有废话，就硬算，然而效率不高。

```c
int count_one_bits(unsigned char byte)
{
    int n = 0;

    for (int i = 0; i < 8; i++) {
        if ((byte >> i) & 0x01) {
            n++;
        }
    }

    return n;
}
```

（2）版本二

空间换时间，一个字节共有256个值，打一个256大小的表，然后直接返回即可。

尽管时间复杂度下降为`O(1)`，但是还有两个不足的地方：

- 一个字节的比特位为1的个数最多不超过8，然而使用了int来存储，大量空间被浪费了

- 我们真的需要打一个256这么大的表吗，会不会存在好多浪费？

```c
static int nr_one_bits[] = {
    0, 1, 1, 2, 1, 2, 2, 3,
    // ...
};

static inline int count_one_bits(unsigned char byte)
{
    return nr_one_bits[byte];
}
```

（3）版本三

将一个字节拆分成高四位和低四位分开处理，我们只需要打一张16大小的表即可。

内存空间从1K下降到了16B，运行时间几乎不变。完美！

```c
static unsigned char nr_one_bits_half_byte[] = {
    0, 1, 1, 2, 1, 2, 2, 3,
    1, 2, 2, 3, 2, 3, 3, 4,
};

static inline int count_one_bits(unsigned char byte)
{
    return nr_one_bits_half_byte[byte & 0x0F] +
        nr_one_bits_half_byte[(byte & 0xF0) >> 4];
}
```

## 二、提高性能的两个基本原则

### 两个原则

（1）不做无用功：不要有无意义的代码

（2）杀鸡莫用牛刀：如果使用牛刀的收益覆盖不了牛刀的成本，那么就不该用牛刀

### 不做无用功

常见的无用功：

（1）没必要的初始化

（2）多余的函数调用

```c
void foo(void)
{
    char buf[64] = {0};

    memset(buf, 0, sizeof(buf));
    strcpy(buf, "foo");
}
```

-  既然本意是做strcpy，为啥还要初始化？

-  做了初始化还不够，还要调memset做甚？

### 杀鸡用牛刀

（1）滥用STDIO接口

下面的例子中，往buffer中拷贝字符串完全没有必要使用STDIO这种相对重量级的api，直接使用字符串拷贝就好了。

```c
   // use strcpy and strcat instead
   sprintf(a_buffer, "%s%s", a_string, b_string);
   // use aoti, aotl, atoll, strtol instead
   sscanf(a_string, "%d", &i);
```

（2）滥用高级数据结构，比如，总共就10个数据非要用红黑树、哈希表等等，属实是没必要。

## 三、常见方法和技巧

提升软件性能的常用方法和技巧

### （1）动态缓冲区分配

版本一：不管怎样，我始终钟情于malloc和free

```c
void foo(size_t len)
{
    char *buff = malloc(len);
    // ...
    free(buff);
}
```

版本二：去掉malloc调用

malloc的时间代价很大，能不用就不用。在绝大多数的情况下能用栈内存就不用堆内存

```c
void foo(size_t len)
{
    char stack_buff[PATH_MAX + 1];
    char *buff;

    if (len > sizeof(stack_buff)) {
        buff = malloc(len);
    } else {
        buff = stack_buff;
    }

    // ...

    if (buff && buff != stack_buff) {
        free(buff);
    }
}
```

### （2）字符串匹配

版本一：逐个匹配即可

```c
int get_locale_category_by_keyword(const char *keyword)
{
    if (strcasecmp(keyword, "ctype") == 0) {
        return LC_CTYPE;
    } else if (strcasecmp(keyword, "collate") == 0) {
        return LC_COLLATE;
    } else if (strcasecmp(keyword, "numeric") == 0) {
        return LC_NUMERIC;
    } else if (strcasecmp(keyword, "monetary") == 0) {
        return LC_MESSAGES;
    } else if (strcasecmp(keyword, "time") == 0) {
        return LC_TIME;
    }
    // ...
}
```

上面的代码有问题吗？正确性上没有问题，但性能上有问题

版本二：手工哈希一下，妙哉！

```c
int get_locale_category_by_keyword(const char *keyword)
{
    const char *head = keyword + 1;

    switch (keyword[0]) {
        case 'c':
        case 'C':
            if (strcasecmp(head, "ctype" + 1) == 0) {
                return LC_TYPE;
            } else if (strcasecmp(head, "collate" + 1) == 0) {
                return LC_COLLATE;
            }
            break;
        case 'n':
        case 'N':
            if (strcasecmp(head, "numeric" + 1) == 0) {
                return LC_NUMERIC;
            } else if (strcasecmp(head, "name" + 1) == 0) {
                return LC_NAME;
            }
        // ...
    }
}
```

版本三：牛刀版本

如果字符串的情况比较多，那么干脆就用哈希表吧！牢记不要杀鸡用牛刀的原则。如果用牛刀的收益无法覆盖用牛刀的成本，那么就不要用牛刀。

```c
// 注意定义正确的 SIZEOF_SIZE_T

#if SIZEOF_SIZE_T == 8
    // 2^40 + 2^8 + 0xb3 = 1099511628211
    #define FNV_PRIME   ((size_t)0x100000001b3ULL)
    #define FNV_INIT    ((size_t)0xcbf29ce484222325ULL)
#else
    // 2^24 + 2^8 + 0x93 = 16777
    #define FNV_PRIME   ((size_t)0x01000193)
    #define FNV_INIT    ((size_t)0x811c9dc5)
#endif

static size_t str2Key(const char *str, size_t length)
{
    const unsigned char *s = (const unsigned char *)str;
    size_t hval = FNV_INIT;

    if (str == NULL) {
        return 0;
    }

    if (length == 0) {
        length = strlen(str);
    }

    // FNV-1a hash each octet in the buffer
    while (*s && length) {
        // xor the bottom with the current octet
        hval ^= (size_t)*s++;
        length--;

        // multiply by the FNV magic prime
#ifdef __GUNC__
    #if SIZEOF_SIZE_T == 8
        hval += (hval << 1) + (hval << 4) + (hval << 5) +
            (hval << 7) + (hval << 8) + (hval << 40);
    #else
        hval += (hval << 1) + (hval << 4) + (hval << 7) + (hval << 8);
    #endif
#else
        hval *= FNV_PRIME;
#endif
    }

    // return our new hash value
    return hval;
}

static int categories[] = {
    -1,
    -1,
    LC_CTYPE,
    LC_ADDRESS,
    -1,
    // ...
};

// 运气好，对上面的关键词，当用37对哈希值取模时，刚好没有重复
// 注意：#define SIZEOF_SIZE_T 8时成立
int get_locale_category_by_keyword(const char *keyword)
{
    size_t hval = str2Key(keyword) % (sizeof(categories) / sizeof(categories[0]));
    return categories[hval];
}
```

注意上述版本是有BUG的，如果传入的`keyword`不在`categories`列表中的那12个关键字之内，有可能它的哈希值刚好和列表中的某个关键字冲突了，这时候就会返回一个错误的值。

版本四：屠龙版本——字符串原子化

字符串原子化原理

1. 原子表示一个可以唯一性确定一个字符串常量的整数值

2. 背后的数据结构是一个AVL树或者红黑树，保存着字符串常量和整数之间的映射关系（每当往树中插入一个字符串就返回一个唯一的整数值）

字符串原子化的好处

1. 原先要分配缓冲区存储指针的地方，现在只需要存储一个整数

2. 原先调用strcmp对比字符串的地方，现在可使用`==`直接对比

3. 原先使用复杂判断的地方，现在可以使用switch语句

4. 通用，但是其综合性能未必最优（一般是用在大规模的字符串场景下，比如HTML中的所有标签名。如果字符串数量较小，使用它不划算）

```c
typedef unsigned int purc_atom_t;

PCA_EXPORT purc_atom_t
purc_atom_from_string(const char *string);

PCA_EXPORT purc_atom_t
purc_atom_from_static_string(const char *string);

PCA_EXPORT purc_atom_t
purc_atom_try_string(const char *string);

PCA_EXPORT const char *
purc_atom_to_string(purc_atom_t atom);
```

前两个接口是根据字符串返回一个唯一的整数值，第三个接口是查询该字符串是否已经被原子化了，最后一个接口是根据一个整数值返回其对应的字符串。基于上述接口，屠龙版本可以这样写

```c
static struct category_to_atom {
    const char *name;
    purc_atom_t atom;
    int category
} _atoms[] = {
    { "ctype", 0, LC_CTYPE },
    { "collate", 0, LC_COLLATE },
    // ...
};

#define NR_CATEGORIES (sizeof(_atoms) / sizeof(_atoms[0]))

// 系统初始化时
    for (size_t i = 0; i < NR_CATEGORIES; i++) {
        _atoms[i].atom = purc_atom_from_static_string(_atoms[i].name);
    }

int get_locale_category_by_keyword(const char *keyword)
{
    purc_atom_t atom = purc_atom_try_string(keyword);

    if (atom >= _atoms[0].atom || atom <= _atoms[NR_CATEGORIES - 1].atom) {
        return _atoms[atom - _atoms[0].atom].category;
    }

    return -1;
}
```

屠龙版本有两个需要注意的点：

（1）字符串原子化背后基于AVL树或者红黑树，它的空间代价和时间代价都要比哈希（不冲突）要高，这是使用字符串原子化所需支付的成本

（2）字符串原子化背后的AVL树或者红黑树是全局的，然而无法保证系统不同模块之间不会出现相同的字符串。也即上述的字符串原子化方式需要假定被原子化的各个字符串各不相同。然而这一点很难保证

版本五：倚天版本

在版本四的字符串原子化的基础上加入约束——按照不同的命名空间来管理字符串常量，以避免不同相同关键字具有相同的原子值

```c
#define PURC_ATOM_BUCKET_BITS   4   // 共支持16个bucket（不同的命名空间）
#define PURC_ATOM_BUCKETS_NR    (1 << PURC_ATOM_BUCKET_BITS)

// 往指定的命名空间插入字符串
PCA_EXPORT purc_atom_t
purc_atom_from_string_ex(int bucket, const char *string);

static inline purc_atom_t purc_atom_from_string(const char *string)
{
    return purc_atom_from_string_ex(0, string);
}

PCA_EXPORT purc_atom_t
purc_atom_from_static_string_ex(int bucket, const char *string);

static inline purc_atom_t purc_atom_from_static_string(const char *string)
{
        return purc_atom_from_static_string_ex(0, string);
}

// 定义命名空间，最多16个
enum {
    ATOM_BUCKET_DEFAULT = 0,
    ATOM_BUCKET_LOCAL_CATEGORY,
};

static struct category_to_atom {
    const char *name;
    purc_atom_t atom;
    int category;
} _atoms[] = {
    { "ctype", 0, LC_CTYPE },
    { "collate", 0, LC_COLLATE },
    // ...
};

// 系统初始化时
    for (size_t i = 0; i < NR_CATEGORIES; i++) {
        _atoms[i].atom = purc_atom_from_static_string_ex(
            ATOM_BUCKET_LOCALE_CATEGORY, _atoms[i].name);
    }

int get_locale_category_by_keyword(const char *keyword)
{
    purc_atom_t atom = purc_atom_try_string_ex(
        ATOM_BUCKET_LOCALE_CATEGORY, keyword);

    if (atom >= _atoms[0].atom || atom <= _atoms[NR_CATEGORIES - 1].atom) {
        return _atoms[atom - _atoms[0].atom].category;
    }

    return -1;
}
```

## 四、性能优化总结

1. 现行访问、整数运算永远最快

2. 通常情况下使用空间换时间

3. 通用方案的空间代价高，最佳方案需要因地制宜，不可僵化

## 复杂案例研究

实例研究：如何判断一个自然数是否为素数？

1. 16以内

最佳平衡：使用位图表示是否为素数

3. 1024以内

最佳平衡：打一个1024以内的素数表（有序），然后使用二分查找

5. 任意64位自然数

最佳平衡是？

6. 任意自然数

最佳平衡是？
