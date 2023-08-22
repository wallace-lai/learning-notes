# 记一次内存写超错误

作者：wallace-lai </br>
时间：2023-08-22 </br>
更新：2023-08-22 </br>

在运行程序时发现一个基本的`getchar()`调用竟然报了assertion failed错误。非常奇怪，因为`getchar()`是非常基本的接口，几乎不可能出错。

```
main: malloc.c:2379: sysmalloc: Assertion `(old_top == initial_top (av) && old_size == 0) || ((unsigned long) (old_size) >= MINSIZE && prev_inuse (old_top) && ((unsigned long) old_end & (pagesize - 1)) == 0)' failed.
Aborted (core dumped)
```

经过搜索后发现原因可能是有内存错误。函数的调用逻辑如下所示，在`getchar()`调用之前确实有内存申请的操作并且对申请的内存做了写操作。很有可能是内存申请或者申请后的写操作导致后续的`getchar()`调用失败的。
```
- CreateNewGame
    - CreateNewFrame
        - malloc
- GameRun
    - getchar
```

使用valgrind工具进行内存读写的检测，果然发现了非法写入操作，某些写操作超出了内存申请的范围。
```
$ valgrind --leak-check=full ./main
==12468== Memcheck, a memory error detector
==12468== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==12468== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==12468== Command: ./main
==12468==
==12468== Invalid write of size 4
==12468==    at 0x109DAB: CreateNewFrame (frame.c:87)
==12468==    by 0x109740: CreateNewGame (game.c:102)
==12468==    by 0x1093A6: main (main.c:18)
==12468==  Address 0x4a476b0 is 0 bytes after a block of size 304 alloc'd
==12468==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
==12468==    by 0x109C90: CreateNewFrame (frame.c:68)
==12468==    by 0x109740: CreateNewGame (game.c:102)
==12468==    by 0x1093A6: main (main.c:18)
```

回到代码，一开始真的没发现有什么问题，但又确实存在写超的情况。于是我只好手动计算结构体大小，发现申请的大小远远小于了所需的大小。再仔细一看代码，发现问题所在了————额外的mapSize大小内存是用来存放`BlockType`类型数据的，但是mapSize只计算了个数，还应该乘上`BlockType`类型的大小才对！
```c
Frame *CreateNewFrame(int w, int h, int thick, int x, int y)
{
    size_t mapSize = (w + thick * 2) * (h + thick * 2);     // 内存写超了
    size_t totalSize = sizeof(Frame) + mapSize;
    Frame *frame = malloc(totalSize);
    HANDLE_ON_ERROR(frame == NULL);
    memset(frame, 0, totalSize);

    // 对frame进行写操作
    // ...
    frame->map[j + i * frame->mapCol] = // ...
    // ...

    return frame;
}
```

将代码修改成以下的样子，问题解决!
```c
size_t mapSize = (w + thick * 2) * (h + thick * 2) * sizeof(BlockType);
```

## 总结
valgrind是一个内存检测的好工具，合理运用它可以提升代码质量