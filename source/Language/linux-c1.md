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

- stat
- fstat
- lstat

（2）文件访问权限

- `st_mode`是一个16位的位图，用于表示文件类型、访问权限、特殊权限位

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

### 4. 文件类型

（1）测试文件类型

```
    S_ISREG(m)  is it a regular file?
    S_ISDIR(m)  directory?
    S_ISCHR(m)  character device?
    S_ISBLK(m)  block device?
    S_ISFIFO(m) FIFO (named pipe)?
    S_ISLNK(m)  symbolic link?  (Not in POSIX.1-1996.)
    S_ISSOCK(m) socket?  (Not in POSIX.1-1996.)
```

（2）文件类型应用案例

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/fs/ftype.c)

```c
    if (stat(fname, &res) < 0) {
        perror("stat");
        exit(1);
    }

    int ret = 0;
    if (S_ISREG(res.st_mode)) {
        ret = '-';
    } else if (S_ISDIR(res.st_mode)) {
        ret = 'd';
    } else if (S_ISCHR(res.st_mode)) {
        ret = 'c';
    } else if (S_ISBLK(res.st_mode)) {
        ret = 'b';
    } else if (S_ISFIFO(res.st_mode)) {
        ret = 'p';
    } else if (S_ISSOCK(res.st_mode)) {
        ret = 's';
    } else if (S_ISLNK(res.st_mode)) {
        ret = 'l';
    } else {
        ret = '?';
    }

    return ret;
```

（3）`st_mode`位图定义

```
The following mask values are defined for the file mode component of the st_mode field:

    S_ISUID     04000   set-user-ID bit (see execve(2))
    S_ISGID     02000   set-group-ID bit (see below)
    S_ISVTX     01000   sticky bit (see below)

    S_IRWXU     00700   owner has read, write, and execute permission
    S_IRUSR     00400   owner has read permission
    S_IWUSR     00200   owner has write permission
    S_IXUSR     00100   owner has execute permission

    S_IRWXG     00070   group has read, write, and execute permission
    S_IRGRP     00040   group has read permission
    S_IWGRP     00020   group has write permission
    S_IXGRP     00010   group has execute permission

    S_IRWXO     00007   others (not in group) have read,  write,  and
                        execute permission
    S_IROTH     00004   others have read permission
    S_IWOTH     00002   others have write permission
    S_IXOTH     00001   others have execute permission
```

## P145 ~ P146 文件系统 - 文件权限的更改

### 1. umask

创建文件时如果没有指定访问权限，则其默认值为`0600 & ~umask`。umask的作用是防止产生权限过大的文件

```c
    #include <sys/types.h>
    #include <sys/stat.h>

    mode_t umask(mode_t mask);
```

### 2. 更改文件权限

```c
    #include <sys/stat.h>

    int chmod(const char *pathname, mode_t mode);
    int fchmod(int fd, mode_t mode);
```

### 3. 粘住位

t位（粘住位）的原始设计是针对二进制可执行文件，设置t位后可保留其使用痕迹，下次再次装载运行该文件时可以更快速。

现在最常用的是给目录设置t位，比如`/tmp`目录就设置了t位

```shell
$ ls -l /
dr-xr-xr-x  13 root root          0 Jul 23 22:55 sys
drwxrwxrwt  15 root root      12288 Jul 23 23:26 tmp
drwxr-xr-x  14 root root       4096 Aug 31  2022 usr
drwxr-xr-x  15 root root       4096 Jun 14 16:11 var
```

### 4. FAT文件系统

略

## P147 文件系统 - UFS文件系统

略

## P148 文件系统 - 链接文件

### 1. 命令
（1）创建硬链接：`ln /tmp/bigfile /tmp/bigfile_link`

（2）创建符号链接：`ls -s /tmp/bigfile /tmp/bigfile_s`

注意：硬链接与目录项是同义词，但建立硬链接有限制，即不能跨分区建立也不能给目录建立。符号链接可以跨越分区，也可以给目录建立符号链接。

### 2. 接口
```c
    #include <unistd.h>

    int link(const char *oldpath, const char *newpath);
```

（1）使用link创建硬链接

```c
    #include <unistd.h>

    int unlink(const char *pathname);
```

（2）使用unlink删除文件，不能删除非空目录

```c
    #include <stdio.h>

    int remove(const char *pathname);
```

（3）使用remove删除文件或者目录，不能删除非空目录

```c
    #include <stdio.h>

    int rename(const char *old, const char *new);
    int renameat(int oldfd, const char *old, int newfd,
        const char *new);
```

### 3. 更改时间

```c
    #include <sys/types.h>
    #include <utime.h>

    int utime(const char *filename, const struct utimbuf *times);
```

utime用于改变文件的时间，可以更改atime和mtime

### 4. 创建和删除目录

```c
    #include <sys/stat.h>
    #include <sys/types.h>

    int mkdir(const char *pathname, mode_t mode);

    #include <unistd.h>
    int rmdir(const char *pathname);
```

### 5. 更改当前工作目录

```c
    #include <unistd.h>

    int chdir(const char *path);
    int fchdir(int fd);
```

```c
    #include <unistd.h>

    char *getcwd(char *buf, size_t size);
```

获取当前的工作路径

## P149 ~ P152 文件系统 - 目录解析

目录遍历有2中方式：

（1）使用glob

- glob()
- globfree()

（2）使用一些列的接口

- opendir()
- closedir()
- readdir()
- rewinddir()
- seekdir()
- telldir()

### 1. glob

```c
    #include <glob.h>

    typedef struct {
        size_t   gl_pathc;    /* Count of paths matched so far  */
        char   **gl_pathv;    /* List of matched pathnames.  */
        size_t   gl_offs;     /* Slots to reserve in gl_pathv.  */
    } glob_t;

    int glob(const char *pattern, int flags,
            int (*errfunc) (const char *epath, int eerrno),
            glob_t *pglob);
    void globfree(glob_t *pglob);
```

### 2. glob应用案例

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/fs/glob.c)

使用glob去匹配模式

```c
    #define PAT "/etc/*"

    glob_t res = { 0 };

    err = glob(PAT, 0, NULL, &res);
    if (err) {
        printf("ERROR CODE : %d\n", err);
        exit(1);
    }

    for (int i = 0; i < res.gl_pathc; i++) {
        puts(res.gl_pathv[i]);
    }
```

解释：

（1）模式为`"/etc/*"`表示要找到`/etc`目录下的所有文件

### 3. 解析目录系列接口

（1）打开目录，和打开文件极其类似

```c
    #include <sys/types.h>
    #include <dirent.h>

    DIR *opendir(const char *name);
    DIR *fdopendir(int fd);
```

（2）关闭目录

```c
    #include <sys/types.h>
    #include <dirent.h>

    int closedir(DIR *dirp);
```

（3）读取目录内容

```c
    #include <dirent.h>

    struct dirent {
        ino_t          d_ino;       /* Inode number */
        off_t          d_off;       /* Not an offset; see below */
        unsigned short d_reclen;    /* Length of this record */
        unsigned char  d_type;      /* Type of file; not supported
                                        by all filesystem types */
        char           d_name[256]; /* Null-terminated filename */
    };

    struct dirent *readdir(DIR *dirp);
```

（4）设置readdir()的读取位置，与操作文件位置接口极其类似

```c
    #include <dirent.h>

    void seekdir(DIR *dirp, long loc);
    void rewinddir(DIR *dirp);
    long telldir(DIR *dirp);
```

### 4. readdir应用案例1

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/fs/readdir.c)

使用readdir解析目录下文件（不递归）

```c
    #define PAT "/etc"

    dp = opendir(PAT);
    if (dp == NULL) {
        perror("opendir()");
        exit(1);
    }

    while ((curr = readdir(dp)) != NULL) {
        puts(curr->d_name);
    }
```

解释：

（1）先使用`opendir`打开目录，得到`DIR`指针

（2）使用`readdir`读取`/etc`目录下的所有文件名

