# 【Linux C】文件系统

作者：wallace-lai <br>
发布：2024-05-29 <br>
更新：2024-05-29 <br>

文件系统对应APUE上的章节为：

- 第4章：文件和目录

- 第6章：系统数据文件和信息

- 第7章：进程环境

对应的教程：P142 ~ P166

## P142 文件系统 - 介绍

### 1. 目录和文件

（1）获取文件属性

（2）文件访问权限

（3）umask

（4）文件权限的更改、管理

（5）粘住位

（6）文件系统：FAT和UFS

（7）硬链接、符号链接

（8）utime

（9）目录的创建和销毁

（10）更改当前工作目录

### 2. 系统数据文件和信息

### 3. 进程环境

## P143 ~ P145 文件系统 - stat

### 1. 接口

```c
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <unistd.h>

    int stat(const char *pathname, struct stat *statbuf);
    int fstat(int fd, struct stat *statbuf);
    int lstat(const char *pathname, struct stat *statbuf);

    struct stat {
        dev_t     st_dev;         /* ID of device containing file */
        ino_t     st_ino;         /* Inode number */
        mode_t    st_mode;        /* File type and mode */
        nlink_t   st_nlink;       /* Number of hard links */
        uid_t     st_uid;         /* User ID of owner */
        gid_t     st_gid;         /* Group ID of owner */
        dev_t     st_rdev;        /* Device ID (if special file) */
        off_t     st_size;        /* Total size, in bytes */
        blksize_t st_blksize;     /* Block size for filesystem I/O */
        blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */

        /* Since Linux 2.6, the kernel supports nanosecond
            precision for the following timestamp fields.
            For the details before Linux 2.6, see NOTES. */

        struct timespec st_atim;  /* Time of last access */
        struct timespec st_mtim;  /* Time of last modification */
        struct timespec st_ctim;  /* Time of last status change */

    #define st_atime st_atim.tv_sec      /* Backward compatibility */
    #define st_mtime st_mtim.tv_sec
    #define st_ctime st_ctim.tv_sec
    };
```

### 2. stat应用案例

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/fs/flen.c)

使用stat输出文件大小

```c
static off_t flen(const char *pathname)
{
    struct stat res;
    if (stat(pathname, &res) < 0) {
        perror("stat()");
        exit(1);
    }

    return res.st_size;
}
```

### 3. 空洞文件

```c
    off_t     st_size;        /* Total size, in bytes */
    blksize_t st_blksize;     /* Block size for filesystem I/O */
    blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */
```

（1）`st_size`为文件逻辑上的大小

（2）`st_blksize`为文件系统的块大小

（3）`st_blocks`为所申请的512B大小的块个数

```shell
$ stat flen.c
  File: flen.c
  Size: 560             Blocks: 8          IO Block: 4096   regular file
```

解释：文件`flen.c`大小为560字节，文件系统块大小为4K，文件`flen.c`占用了8块512B的块（扇区）

创建一个包含大量空洞的5G大小文件：

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/fs/big.c)

```c
    int fd = open(argv[1], O_WRONLY | O_CREAT | O_TRUNC, 0600);
    if (fd < 0) {
        perror("open()");
        exit(1);
    }

    off_t len = 5LL * 1024 * 1024 * 1024 - 1;
    ret = lseek(fd, len, SEEK_SET);
    if (ret < 0) {
        perror("lseek()");
        exit(1);
    }

    write(fd, "\0", 1);
    close(fd);
```

解释：

（1）使用lseek将文件当前位置设置到5G - 1的位置

（2）然后写入一个尾零`\0`

```shell
$ stat /tmp/bigfile
  File: /tmp/bigfile
  Size: 5368709120      Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 655390      Links: 1

$ stat /tmp/bigfile.bak
  File: /tmp/bigfile.bak
  Size: 5368709120      Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 655391      Links: 1
```

可以看到这个5G大小的文件实际只占用了4K大小的磁盘空间

