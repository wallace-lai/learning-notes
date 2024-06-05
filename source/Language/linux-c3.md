# 【Linux C】进程间通信

作者：wallace-lai <br>
发布：2024-05-29 <br>
更新：2024-06-05 <br>

进程间通信对应APUE上的章节为：

- 第8章：进程控制

- 第13章：守护进程

- 第15章：进程间通信

- 第16章：网络IPC：套接字

对应的教程：P236 ~ P266

## P236 进程间通信详解

### 管道通信

（1）管道特征

- 内核提供
- 单工：一端为读端，一端为写端
- 自同步机制：管道读空了，读操作会被阻塞；管道写满了，写操作会被阻塞

（2）匿名管道pipe

```c
#include <unistd.h>

int pipe(int pipefd[2]);
```

（3）命名管道mkfifo

```c
#include <sys/types.h>
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);
```


### XSI（SysV）通信

### 网络套接字

## P237 进程间通信 - 管道实例

### 匿名管道实例

```c
int pd[2];
if (pipe(pd) < 0) {
    perror("pipe()");
    exit(1);
}

pid = fork();
if (pid < 0) {
    perror("fork()");
    exit(1);
}

if (pid == 0) {
    // child
    close(pd[1]);
    int len = read(pd[0], buffer, BUFSIZE);
    write(1, buffer, len);
    close(pd[0]);
    exit(0);
} else {
    // parent
    const char *msg = "hello message from parent.\n";
    close(pd[0]);
    write(pd[1], msg, strlen(msg));
    close(pd[1]);
    wait(NULL);
    exit(0);
}
```

注意：

（1）对于子进程而言，因为不需要往管道写入，所以需要关闭写端`pd[1]`以避免麻烦。同时读取完毕后在退出程序前需要关闭读端`pd[0]`；

（2）对于父进程而言，则是关闭读端`pd[0]`，在写完后再关闭写端`pd[1]`；

（3）记得父进程得使用`wait(NULL)`来等待子进程退出；

如果要基于上述匿名管道完成流媒体播放功能，伪代码可以这么写：

```c
if (pid == 0) {
    // child
    close(pd[1]);
    dup2(pd[0], 0);
    close(pd[0]);

    int fd = open("/dev/null", O_RDWR);
    dup2(fd, 1);
    dup2(fd, 2);
    execl("/usr/bin/mpg123", "mpg123", "-", NULL);

    perror("execl()");
    exit(1);
} else {
    // parent
    close(pd[0]);

    // 网络流媒体：父进程从网上收数据，往管道中写

    close(pd[1]);
    wait(NULL);
    exit(0);
}
```

注意：

（1）子进程中的mpg123通过`-`选项读取标准输入作为输入，我们并不需要mpg123本身的信息输出，所以需要将标准输出2和标准错误输出3给重定向到`/dev/null`中去；

### 命名管道实例

进度：P237 22:00