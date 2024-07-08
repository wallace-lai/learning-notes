# 【Linux C】输入输出

作者：wallace-lai <br>
发布：2024-05-29 <br>
更新：2024-05-29 <br>

输入输出对应APUE上的章节为：

- 第3章：文件IO

- 第5章：标准IO库

- 第14章：高级IO

对应的教程：

（1）P123 ~ P141

（2）P220 ~ P235

## P123 ~ P124 标准IO - 介绍

IO是一切实现的基础，IO分为两种：

（1）stdio：标准IO库

（2）sysio：系统调用IO

原则：**优先使用标准IO库，而不是系统调用IO**。标准IO库的一大功能是合并系统调用，提升性能，且具有可移植性。

### 1. 标准IO主要接口

（1）打开与关闭：

- fopen()
- fclose()

（2）字符输入输出

- fgetc()
- fputc()
- fgets()
- fputs()

（3）二进制读写

- fread()
- fwrite()

（4）常用输入输出

- printf()
- scanf()

（5）设置文件位置

- fseek()
- ftell()
- rewind()

（6）刷新缓冲区
- fflush()

## P125 ~ P127 标准IO - 文件打开与关闭

### 1. 接口

（1）打开文件

```c
    #include <stdio.h>

    FILE *fopen(const char *pathname, const char *mode);

    FILE *fdopen(int fd, const char *mode);

    FILE *freopen(const char *pathname, const char *mode, FILE *stream);
```

（2）关闭文件

```c
    #include <stdio.h>

    int fclose(FILE *stream);
```

打开文件中的`mode`可以有以下几种方式：

- r：打开文件（仅）用于读，要求文件必须存在，读取位置设置在文件开头处
- r+：打开文件用于读写，要求文件必须存在，读取位置设置在文件开头处

- w：打开文件（仅）用于写，文件不存在则创建，文件存在在截断为0，写入位置在文件开头处
- w+：打开文件用于读写，文件不存在则创建，文件存在则截断为0，读写开始位置在文件开头处

- a：打开文件用于追加（写），文件不存在则创建，追加位置在文件末尾处
- a+：打开文件用于读和追加，文件不存在则创建，追加和读位置在文件开头处（glibc）

### 2. 打开文件应用实例1

```c
int main()
{
    FILE *fp = fopen("noexist", "w+");
    if (fp == NULL) {
        perror("fopen()");
        exit(1);
    }
    puts("OK");

    fclose(fp);
    return 0;
}
```

### 3. 打开文件应用实例2 - 可打开文件个数上限

```c
int main()
{
    FILE *fp = NULL;
    int count = 0;

    while (1) {
        fp = fopen("fopen.c", "r");
        if (fp == NULL) {
            perror("fopen()");
            break;
        }
        count++;
    }
    printf("count = %d\n", count);

    return 0;
}
```

运行查看程序结果：

```shell
$ ./fopen_max
fopen(): Too many open files
count = 1021

$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15188
max locked memory       (kbytes, -l) 65536
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15188
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

可以看到内核限制进程可打开的文件个数open files为1024个，由于标准输入、输出、错误占用了3个文件个数，所以可打开的文件个数上限为1021个。

## P128 标准IO - 字符输入输出


