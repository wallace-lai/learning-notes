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

### 1. 管道通信

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


### 2. XSI（SysV）通信

（1）XSI提供的通信机制

- Message Queues

- Shared Memory Segments

- Semaphore Arrays

（2）ipcs命令

```
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
```

可以看到，三种方式中都有key字段。key的作用是让IPC通信的各个实例能够在没有亲缘关系的进程中共享。

（3）使用方法

- 使用ftok获得key值

- 使用三套接口来操作

上述通信机制提供了三套接口，分别是xxxget、xxxop、xxxctl。xxx表示通信机制，分别是msg（消息队列）、shm（共享内存）、sem（信号量）。使用`man xxxget`命令可以得到对应的man手册。


### 3. 网络套接字

## P237 进程间通信 - 管道详解

### 1. 匿名管道实例

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

### 2. 命名管道实例

（1）使用mkfifo创建一个命名管道，发现管道的文件类型是p（管道文件）

```
$ cd /tmp/
$ mkfifo nfifo
$ ls -lah nfifo
prw-rw-r-- 1 lzh lzh 0 Jun  6 14:39 nfifo
```

（2）往管道中写入数据，发现写入操作会被阻塞

```
$ date
Thu 06 Jun 2024 02:39:08 PM UTC
$ date > nfifo
```

（3）在另一个命令行中读取管道数据，可以将数据读出，此时写入操作才会结束

```
$ cat nfifo
Thu 06 Jun 2024 02:39:23 PM UTC
```

### 3. 总结

（1）匿名管道无法应用在没有亲缘关系的进程之间通信；

（2）命名管道可以应用在没有亲缘关系的进程之间通信，因为命名管道可以对应磁盘上的管道文件

## P238 ~ P240 进程间通信 - 消息队列详解

### 1. 消息队列

**消息队列的特点**

（1）相关接口

- ftok()
- msgget()
- msgop()
- msgctl()

（2）双工，可读可写

**消息队列的实现案例：基础版**

我们首先定义好发送端和接收端之间的消息格式以及用于生成key的数据，如下所示：

```c
// proto.h

#define KEYPATH "/etc/services"
#define KEYPROJ 'G'

#define NAME_SIZE 16

typedef struct _msg {
    long mtype;
    char name[NAME_SIZE];
    int math;
    int chinese;
} msg;
```

对于接收端，需要先获取key，随后创建消息队列得到msgid，最后将接受到的消息解码后打印：

```c
key_t key = ftok(KEYPATH, KEYPROJ);
if (key < 0) { /* ... */ }

int msgid = msgget(key, IPC_CREAT | 0600);
if (msgid < 0) { /* ... */ }

msg rbuf;
while (1) {
    ssize_t len = msgrcv(msgid, &rbuf, sizeof(msg) - sizeof(long), 0, 0);
    if (len < 0) { /* ... */ }

    printf("NAME = %s\nMATH = %d\nChinese = %d\n", rbuf.name, rbuf.math, rbuf.chinese);
}

msgctl(msgid, IPC_RMID, NULL);
exit(0);
```

对于发送端，则是发送消息：

```c
key_t key = ftok(KEYPATH, KEYPROJ);
if (key < 0) { /* ... */ }

int msgid = msgget(key, 0);
if (msgid < 0) { /* ... */ }

srand(time(NULL));

msg sbuf;
sbuf.mtype = 1;
strncpy(sbuf.name, "Alice", NAME_SIZE);
sbuf.math = rand() % 100;
sbuf.chinese = rand() % 100;

int ret = msgsnd(msgid, &sbuf, sizeof(sbuf) - sizeof(long), 0);
if (ret < 0) { /* ... */ }

puts("OK");
exit(0);
```

注意：

（1）我们将创建消息队列的任务交给接收端来完成，因为接收端不启动，则没有通信的必要；

（2）一般推荐接收端进程先启动，随后才是发送端进程启动。如果发送端进程先启动并发送多个消息，那么消息将会被缓存

启动发送端并连续发送三个消息：

```shell
$ ./snder
OK
$ ./snder
OK
$ ./snder
OK
$
```

则接收端启动时会一下子收到三个消息：

```shell
$ ./rcver
NAME = Alice
MATH = 83
Chinese = 90
NAME = Alice
MATH = 92
Chinese = 13
NAME = Alice
MATH = 50
Chinese = 44
```

每个消息队列的缓存区大小可以通过ulimit查看，可以看到默认设置下的消息队列缓存区大小为800KB（819200B）。这一数值可以修改。如果发送端发送的消息超过了这个限制，那么会产生消息丢失。

```shell
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

**消息队列的实现案例：ftp实例**
【进度在此】

## P241 进程间通信 - 信号量详解
