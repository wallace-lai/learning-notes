# 【C语言最佳实践】接口设计模式

作者：wallace-lai <br/>
发布：2022-07-31 <br/>
更新：2023-11-26 <br/>

## 一、好接口的标准是什么？

- （1）恰当的抽象，比如
  - 1. POSIX的文件描述符
  - 2. POSIX的DIRENT结构
  - 3. STDC的FILE结构
- （2）调用者友好 
- （3）符合惯例，学习成本低 
- （4）没有过度设计
- （5）接口设计稳定，有助于提升软件质量与可维护性

## 二、两个接口设计原则

1. 完备：完整、不缺东西

2. 自洽：无逻辑漏洞，自圆其说

### 完整性的保证：对称设计

1. 有new，必然有delete 

2. 有init，必然有deinit、terminate或cleanup 

### 接口设计的一般性方法和技巧

1. 尽量避免返回void

2. 明确定义返回值的含义

3. 正确使用const修饰符

4. 使用最合适的参数类型

举一个接口设计案例如下：
```c
void *memcpy(void *dst, const void *src, size_t n);

// 错误的设计
void my_memcpy(void *dst, void *src, int n);
```

可以看到，上面的`my_memcpy`存在的几个设计问题：

1. 函数无返回值

2. 入参`src`没有使用`const`进行修饰，存在被篡改的可能

3. 入参`n`不应该使用`int`，而应该使用`size_t`，否则有传入负数的可能性

4. 依据惯例，函数参数应该以先入参后出参的顺序排列

## 三、常用接口设计模式

### 3.1 模式一：抽象数据类型

抽象数据类型定义：

（1）隐藏实现细节，为增强、优化和可扩展性打下基础

（2）围绕抽象数据结构设计接口

一些抽象数据类型案例：

（1）STDC中的STDIO接口

```c
FILE *fopen(const char *path, const char *mode);
FILE *fdopen(int fildes, const char *mode);
```

（2）MiniGUI RWops接口 

（3）GLib IO Channels接口


### 3.2 模式二：抽象算法

抽象算法定义：

（1）围绕抽象算法设计接口

（2）算法不依赖于具体的数据类型

（3）算法不依赖于具体的数据存储方式

抽象算法的使用案例：

（1）C标准库中的qsort函数
```c
void qsort(
    void *base,     // 待排序数组首元素指针
    size_t nitems,  // 数组中元素的个数
    size_t size,    // 数组中每个元素的大小
    // 自定义的排序函数
    int (*compar)(const void *, const void *)
);
```

```c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>

int CompareInt(const void *lhs, const void *rhs)
{
    int a = *((int *)lhs);
    int b = *((int *)rhs);
    return a - b;
}

int CompareString(const void *lhs, const void *rhs)
{
    char *a = (char *)lhs;
    char *b = (char *)rhs;
    return strcmp(a, b);
}

int main()
{
    size_t len1;
    size_t len2;

    int nums[] = {
        15, 96, 82, 64, 32,
        87, 17, 20, 37, 70,
    };

    char *strs[] = {
        "apple", "cat", "dog", "banana",
        "elephant", "fish", "gorilla", "house",
    };

    len1 = sizeof(nums) / sizeof(nums[0]);
    qsort(&nums[0], len1, sizeof(nums[0]), CompareInt);

    len2 = sizeof(strs) / sizeof(strs[0]);
    qsort(&strs[0], len2, sizeof(strs[0]), CompareString);

    for (size_t i = 0; i < len1; i++) {
        printf("%d ", nums[i]);
    }
    printf("\n");
    for (size_t i = 0; i < len2; i++) {
        printf("%s\n", strs[i]);
    }
    printf("\n");

    return 0;
}
```

```
15 17 20 32 37 64 70 82 87 96

apple
cat
dog
banana
elephant
fish
gorilla
house
```

注意上面的`qsort`函数依赖底层数据是连续的，即以数组的方式存放在一起。但如下所示，同样可以定义不依赖连续存储的`qsort`版本。

```c
void qsort_ex(
    void *array,
    void *(get_member)(void *, int idx),
    void (*exchange_members)(void *, int idx_one, int idx_oth),
    int (*compar)(const void *, const void *)
);
```

### 3.3 模式三：隐式上下文

隐式上下文定义：

（1）上下文通常用于保存当前的设置、状态等信息，通常被设计为句柄、隐藏细节的结构指针等

（2）上下文在图形绘制接口中常见，比如在MiniGUI或者Cairo中绘制一个矩形

举个例子，使用OpenGLAPI绘制矩形，发现它并没有用到显示的上下文结构。这是因为OpenGLAPI不用上下文吗？不是，它只是利用了隐式上下文而已。

```c
glColor3f(1.0, 0.0, 0.0);

glBegin(GL_LINE_LOOP);
glVertex3f(0.0, 0.0, 0.0);
glVertex3f(0.0, 1.0, 0.0);
glVertex3f(1.0, 1.0, 0.0);
glVertex3f(1.0, 0.0, 0.0);
glEnd();
glFlush();
```

OpenGL中的隐式上下文 

1. 使用线程本地存储（TLS）保存上下文信息。应用程序无需关注默认上下文的创建以及销毁

2. 在同一个线程内，使用eglCreateContext创建多个上下文（同一个线程内在多个不同的表明绘制图形的时候需要多个上下文），使用eglMakeCurrent函数切换上下文

隐式上下文的好处 

1. 减少函数中的参数传递，尤其是上下文和线程绑定时

2. 解决接口的历史遗留问题

隐式上下文实例

（1）STDC中的错误码errno

文档中的errno是一个全局变量extern int errno;，然而其实际被定义为：

```c
extern int *__error(void);
#define errno (*__error())
```

也即errno被定义为了一个TLS变量，也即每一个线程都有它自己的errno副本

### 3.4 模式四：事件/消息驱动

事件/消息驱动定义：

1. 事件/消息驱动最早出现在GUI编程中，如Win32和各种GUI库，用于处理人机交互事件、窗口时间等

2. 在Glib，WTF，MiniGUI中，被进一步抽象，可用来监听文件描述符（包括套接字和管道）、定时器以及用户定制的事件

3. 还可以用作线程间通讯机制使用

事件/消息驱动接口设计的基本概念：

1. 事件/消息生产者

2. 事件/消息消费者

3. 事件/消息处理器（一般是回调函数），以MiniGUI为例

    1. 消息驱动接口围绕窗口设计，每个窗口都有一个自己的消息处理回调函数
    2. 在MiniGUI多线程模式下，每个线程可以创建一个自己的消息队列
    3. 一个消息由一个整数标识符和两个参数组成

4. 消息生产者，可以通过PostMessage、SendNotifyMessage和SendMessage三个接口产生消息

    1. 邮寄消息，使用循环队列存储，可能会溢出（丢失）。一般用于不太重要（允许丢失）的消息
    2. 通知消息，使用链表存储，不会丢失
    3. 发送消息，同步等待消息的处理并获得处理返回值
    5. 消息的消费者，通过窗口消息处理回调函数接受消息并进行处理

以GLIB事件驱动接口为例，需要创建事件循环对象GMainLoop

```c
static GMainLoop *loop;
static gint counter = 10;

static gboolean my_timer_callback(gpointer arg)
{
    if (--counter == 0) {
        g_main_loop_quit(loop);
        return FALSE;
    }
    
    return TRUE;
}

    // 创建main loop对象
    loop = g_main_loop_new(NULL, FALSE);
    
    // 添加定时器到当前的main loop中
    g_timeout_add(1000, my_timer_callback, NULL);
    
    // 启动main loop
    g_main_loop_run(loop);
    
    // 销毁main loop
    g_main_loop_unref(loop);
```

GLIB监听文件描述符，为了实现简单的监听一个文件描述符创建了很多个对象（GMainContext、GIOChannel、GSource、GMainLoop），或许是为了实现跨平台而作出的妥协。

```c
static GMainLoop *loop;
static gboolean my_callback(GIOChannel *channel)
{
    // 处理GIOChannel上的可读数据
    ...
}

// 新建一个GMainContext上下文对象给main loop使用
GMainContext *context = g_main_context_new();

// 在给定的文件描述符上创建一个GIOChannel对象
GIOChannel *channel = g_io_channel_unix_new(fd);

// 在上面创建的GIOChannel对象基础上创建一个可监听的数据源
GSource *source = g_io_create_watch(channel, G_IO_IN);
g_io_channel_unref(channel);

// 设置数据源上可读时，要调用的回调函数
g_source_set_callback(source, (GSourceFunc)my_callback, ...);

// 将GSource附加到GMainContext对象上
g_source_attach(source, context);
g_source_unref(source);

// 使用GMainContext对象创建GMainLoop对象
loop = g_main_loop_new(context, FALSE);

// 进入事件循环
g_main_loop_run(loop);

g_main_loop_unref(loop);
```

事件/消息处理器的粒度：

1. 粗粒度：一个事件处理器处理所有的事件

    1. 优点：简洁
    2. 缺点：需要自行析构参数（即代码中会有类似switch分支语句），消息回调函数的代码冗长

2. 细粒度：一个事件处理器处理指定的事件

    1. 优点：直接
    2. 缺点：需要更多的内存保存事件和事件处理器之间的映射关系（需要有一张表记录映射关系）

### 3.5 模式五：通用数据结构

通用数据结构含义：在算法数据结构中保留用户数据字段。以树形结构为例，可以在节点结构中包含用户数据user_data。

```c
typedef struct purc_tree_node {
    // user data
    void *user_data;
    
    // number of children
    size_t nr_children;
    
    // the parent
    struct purc_tree_node *parent;
    
    // the first and last children
    struct purc_tree_node *first, *last;
    
    // the next and previous siblings
    struct purc_tree_node *next, *prev;
} purc_tree_node;

// 设置节点上的用户数据
void purc_tree_node_set_user_data(purc_tree_node_t node,
    void *user_data);

// 获取节点上的用户数据
void *purc_tree_node_get_user_data(purc_tree_node_t node);
```

然而，还有对应的另一种方式是在用户数据结构中包含节点数据结构（回想一下Linux内核中的链表是怎么实现的）。

```c
struct list_head {
    struct list_head *next;
    struct list_head *prev;
};

// 用户数据里包含节点
struct Node {
    int key;
    struct list_head list;
};
```

```c
// @brief 通过结构体成员的地址反查结构体的首地址
// @param ptr : 结构体成员地址
// @param type : 结构体类型
// @param member : 结构体中的成员名字
#define container_of(ptr, type, member) ({ \
        const typeof( ((type *)0)->member ) *__mptr = (ptr); \
        (type *)( (char *)__mptr - offsetof(type,member) );})

struct Node node;
struct list_head *pList = &node.list;
Node *ptr = container_of(pList, struct Node, list);
```

这样做的好处：

1. 一次内存分配即可（上面的方法中节点需要分配一次，user_data还需要分配一次）
2. 简洁、性能高、通用性更好
3. 广泛用于使用节点的各类数据结构，如链表、树等

AVL树实现举例，它也采用在用户数据结构里包含AVL树节点的方式，只有这样才能将AVL数据结构做得足够通用。

https://gitlab.fmsoft.cn/hybridos/hibox/-/blob/master/src/hibox/include/avl.h
https://gitlab.fmsoft.cn/hybridos/hibox/-/blob/master/src/hibox/datastructure/avl.c

### 3.6 模式六：抽象聚类

标准IO接口设计中的抽象聚类

1. 除了我们熟悉的普通文件，我们还可以将内存块看做输入输出流

```c
FILE *open(const char *pathname, const char *mode);
FILE *fdopen(int fd, const char *mode);
FILE *freopen(const char *pathname, const char *mode,
    FILE *stream);

FILE *fmemopen(void *buf, size_t size, const char *mode);
```

这种方法的好处有：

（1）读写接口可同时作用于文件和内存块

（2）提高代码复用率（可维护性好）

1. 使用格式化字符串将各种数据类型的格式化输入和输出统一成了两个基本接口
```c
int printf(const char *format, ...);
int fprintf(FILE *stream, const char *format, ...);
int dprintf(int fd, const char *format, ...);
int sprintf(char *str, const char *format, ...);
int snprintf(char *str, size_t size, const char *format, ...);

// 上述所有接口最后都调用vfprintf实现
int vfprintf(FILE *stream, const char *format, va_list ap);
```

抽象聚类在上述例子中带来的好处：

（1）简化接口的设计，降低学习成本

（2）灵活性：一个接口处理所有数据类型；一个接口处理多个数据

（3）可扩展性：增加新的格式化记号，不需要增加接口

2. MiniGUI中装载不同图片格式的接口
```c
int LoadBitmapFormFile(HDC hdc, PBITMAP pBitmap,
    const char *spFileName);

int LoadBitmapFromMem(HDC hdc, PBITMAP pBitmap,
    const void *mem, size_t size, const char *ext);

// 上述两个接口都是LoadBitmapEx的包装
int LoadBitmapEx(HDC hdc, PBITMAP pBitmap,
    MG_RWops *area, const char *ext);

const char *CheckBitmapType(MG_RWops *rwstream);
```

抽象聚类在上述例子中带来的好处：

（1）通过后缀名来确定装载的图片格式；后缀名失效时使用CheckBitmapType来确定实际图片格式

（2）可从文件或内存中装载；底层使用一个类似STDIO FILE的抽象读写流对象

（3）支持新的图片格式时，无需新增新的接口

### 3.7 模式七：面向对象
早期C++编译器是将C++代码翻译为C代码然后再编译成二进制代码的，对于面向对象，它的实现方式如下

1. C++的class被翻译成两个C的数据结构：
    1. 一个数据结构定义该对象的属性（普通结构成员）

    2. 另一个定义这个类的内部数据、类方法（函数指针）以及可重载的虚函数；这个数据结构对所有这个类的对象公用

2. 对于namespace、class等，在生成的C函数名称之前增加前缀

使用C语言实现面向对象接口的特点：

1. 两个数据结构用于操作集及对象数据的解耦

2. 充斥着大量的宏和指针运算

3. 派生比较容易实现，虚函数重载和多态的实现和使用比较别扭

目前使用C实现面向对象非常麻烦且应用场景不多，与其用C别扭地实现面向对象还不如直接用C++。使用C实现面向对象可以参考mgncs

https://gitlab.fmsoft.cn/VincentWei/mgncs

### 3.8 模式八：接口的扩展和兼容性

对于保留至今的上古旧接口经常出现设计考虑不周的情况

1. 在标准C库中，大量早期接口的实现必须使用全局变量，从而导致这些接口不是线程安全的（上古时候没有多线程的东西）

2. 某些接口的参数或者返回值之参数类型设计不当

扩展方法一：新旧接口共存

```c
char *strtok(char *str, const char *delim);
char *strtok_r(char *str, const char *delim, char **saveptr);
```

`strtok`是早期C标准库里提供的字符串分割函数，然而它不是线程安全的，因为它内部使用了静态缓冲区。`strtok_r`是后期的线程安全版本，新老版本同时共存于标准库当中。

扩展方法二：旧接口只是新接口的包装

```c
// 最新接口
HWND CreateMainWindowEx2(...)
{
    ...
}

// 次新接口是最新接口的封装
HWND CreateMainWindowEx1()
{
    CreateMainWindowEx2(...);
}

// 初始接口是次新接口的封装
HWND CreateMainWindow()
{
    CreateMainWindowEx1();
}
```

扩展方法三：强制使用新接口，旧接口标记位废弃或者移除

```c
gpointer g_memdup(gconstpointer mem, guint byte_size);
// since 2.68
gpointer g_memdup2(gconstpointer mem, gsize byte_size);

// 使用时
#if GLIB_CHECK_VERSION(2, 68, 0)
    cache->cache[pos] = g_memdup2(entry, sizeof(*entry));
#else
    cache->cache[pos] = g_memdup(entry, sizeof(*entry));
#endif
```

扩展接口需要考虑的因素：

1. 新增接口而非修改原有接口的语义

2. 二进制兼容还是源代码兼容？

3. 宏或者内联函数实现接口的向后兼容性，无法保证二进制兼容

注：

这两个术语"二进制兼容"和"源代码兼容"是分别描述了两种不同级别的兼容性。

二进制兼容（Binary Compatibility）：

- 二进制兼容指的是不同软件项目（例如，操作系统、应用程序、库等）在二进制层面上能够相互兼容，即它们可以在同一系统上运行，而不需要进行任何修改或重新编译。
- 这种兼容性通常在大型系统中非常重要，例如在操作系统中，新版本的操作系统需要能够运行旧版本的二进制可执行文件，以确保系统的平滑升级。
- 要实现二进制兼容，可能需要付出很大的努力，例如需要确保系统能够正确地处理各种不同的数据类型、调用约定、系统调用等。

源代码兼容（Source Code Compatibility）：

- 源代码兼容是指两个或多个版本的源代码（通常是同一项目的不同版本）能够在相同的编译环境下编译，并且生成相同的可执行文件或库。
- 源代码兼容是确保软件平滑升级和版本控制的关键因素。如果源代码不兼容，那么每次更新都需要对所有相关的源代码进行重新编译和测试，这会消耗大量时间和资源。
- 要实现源代码兼容，可能需要保持向后兼容的API（应用程序接口），确保编译器和链接器的行为一致，以及处理各种可能的源代码变化。

总的来说，二进制兼容和源代码兼容都是为了使软件系统更加稳定、易于升级和维护。在开发过程中，程序员需要考虑到这两种兼容性，以确保软件能够在不同版本和环境下正常运行。