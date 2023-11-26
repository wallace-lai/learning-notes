# 【C语言最佳实践】子驱动程序模式

作者：wallace-lai <br>
发布：2022-08-02 <br>
更新：2023-11-26 <br>

## 一、设计和编码水平弱的根本原因

根本原因：**抽象能力不足**

（1）对事物的正确认知建立在归纳总结之上

（2）抽象是归纳总结的一种升华

（3）如何提高自己的抽象能力：多看多写


## 二、子驱动程序模式

子驱动程序模式在大量的稍有规模的C项目中大量应用，比如：

（1）Unix中的一切皆文件

（2）Unix/Linux内核的虚拟文件系统以及设备驱动程序

（3）MiniGUI中支持多种类型的图片格式以及逻辑字体

子驱动程序模式的一般实现套路：

（1）一套聚类接口

（2）一些公共数据组成的抽象对象（数据结构）

（3）一组函数指针组成的操作集（数据结构）

（4）针对不同子类的操作集实现

以STDIO接口的简单实现举例

（1）`file_obj`是内部的抽象对象，`file_ops`则是内部的抽象的操作集，`FILE`则是对外抽象数据结构，用户根本无需知晓其中的具体字段。

```c
struct _file_obj;
typedef struct _file_obj file_obj;

struct _file_ops {
    file_obj *open(void *pathname_buf, size_t size, const char *mode);
    ssize_t read(file_obj *file, void *buf, size_t count);
    ssize_t write(file_obj *file, const void *buf, size_t count);
    off_t lseek(file_obj *file, off_t offset, int whence);
    void close(file_obj *file);
};

struct _FILE;
typedef struct _FILE FILE;
```

（2）如果只考虑基本功能，`FILE`结构里只需要有一个抽象文件对象的指针和对应的操作集即可。

```c
struct _FILE {
    struct _file_ops *ops;
    struct _file_obj *obj;
};
```

（3）对于`fopen`来说，它只需要将传入的文件名对应的文件打开即可。打开时返回一个文件描述符即可，因此对于该类接口的实现可以是如下的方式

```c
struct _file_obj {
    int fd;
};

static file_obj *file_open(void *pathname, size_t size, const char *mode)
{
    (void)size;
    // ...
}

static struct _file_ops file_file_ops = {
    .open = file_open;
    // ...
};
```

对外提供的`fopen`接口可以这样包装

```c
FILE *fopen(const char *pathname, const char *mode)
{
    FILE *file = NULL;

    file_obj *obj = file_open(pathname, 0, mode);
    if (obj) {
        file = calloc(1, sizeof(FILE));
        file->obj = obj;
        file->ops = &file_file_ops;
    }

    return file;
}
```

（4）同理，对于`fmemopen`来说也是类似的处理

```c
#define MEM_FILE_FLAG_READABLE      0x01
#define MEME_FILE_FLAG_WRITEABLE    0x02

struct _file_obj {
    void            *buf;
    size_t          size;
    unsigned int    flags;
    off_t           rw_pos;
};

static file_obj *mem_open(void *buf, size_t size, const char *mode)
{
    // ...
}

static file_obj *mem_open(void *buf, size_t size, const char *mode)
{
    // ...
}

static struct _file_ops mem_file_ops = {
    .open = mem_open;
    // ...
};

FILE *fmemopen(void *buf, size_t size, const char *mode)
{
    FILE *file = NULL;

    file_obj *obj = mem_open(buf, size, mode);
    if (obj) {
        file = calloc(1, sizoef(FILE));
        file->obj = obj;
        file->ops = &mem_file_ops;
    }

    return file;
}
```

更进一步考虑，STDIO是带有缓冲区功能的，那么请思考以下问题：

（1）缓冲区信息应该在FILE中维护还是在file_obj中维护？

<span style="color:red;">
个人理解：缓冲区的信息应该在FILE中维护，它属于使用策略的一部分。
</span>

（2）当前读写位置在什么地方维护？

<span style="color:red;">
个人理解：当前读写位置也应该在内部的file_obj里维护，它属于最基本的信息，属于机制的一部分。
</span>

（3）子驱动程序设计的关键点

1. 抽象对象的数据结构如何确定？

2. 操作集如何取舍？

对于第三个问题，有一个一般性的指导原则，我们首先需要正确区分机制和策略

1. 机制：需要提供什么功能（放在子驱动程序里做）

2. 策略：如何使用这些功能（放在子驱动程序的上层抽象层里做）

以STDIO为例：

1. 对于STDIO而言，需要提供什么样的功能？需要提供一组**最小**的**完备**的文件操作集合`_file_ops`，如下所示。

```c
struct _file_obj;
typedef struct _file_obj file_obj;

struct _file_ops {
    file_obj *open(void *pathname_buf, size_t size, const char *mode);
    ssize_t read(file_obj *file, void *buf, size_t count);
    ssize_t write(file_obj *file, const void *buf, size_t count);
    off_t lseek(file_obj *file, off_t offset, int whence);
    void close(file_obj *file);
};
```

任何不同类型的文件对象（对于内存映射文件也如此）都需要这五个操作接口。所以，这个文件操作集合就是机制。而**机制应该放到子驱动程序里做**。

2. 而在基于这组最小完备的文件操作集之上的功能，比如，带有缓冲区支持的格式化输入输出属于使用策略，对不同类型的文件对象是一样的，**应该放到抽象层去做**。
