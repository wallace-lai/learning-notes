# 【Linux C】并发操作

作者：wallace-lai <br>
发布：2024-05-29 <br>
更新：2024-06-19 <br>

并发操作对应APUE上的章节为：

- 第10章：信号

- 第11章：线程

- 第12章：线程控制

对应的教程：P167 ~ P235

## P167 进程 - 进程的概念

### 1. 进程标识符pid

- 类型：`pid_t`

- 命令：`ps`

- 进程号是顺次向下使用的

- 返回当前进程的pid号：`getpid()`

- 返回当前进程的ppid号：`getppid()`


### 2. 进程的产生

（1）接口
- fork()

- vfork()

（2）fork()原理

![fork原理](../media/images/Language/linux-c5.png)

父进程通过fork调用，拷贝了一个子进程出来，父子进程除了以下几点（不是全部）不一样之外，其他都一样：

- fork返回值不一样：父进程返回一个非负整数（子进程的pid），子进程返回0

- 父子进程的pid不同，ppid也不相同

- 未决信号和文件锁不继承

- 新创建子进程的资源利用量清0

- init进程（1号进程）是所有进程的祖先进程

- 不要假定父子进程谁会先被执行，这取决于linux内核调度器的调度策略

- fflush的重要性（在fork子进程之前刷新父进程打开的流）

（3）vfork原理

为什么会有vfork的出现？

因为早期的fork是通过内存拷贝实现的，即父进程内存块中的内容会被完整地拷贝到子进程中。这样带来的一个问题是，fork调用的开销巨大。想象这样一个场景：父进程从数据库中读取了大量的数据，此时需要开辟子进程做一些简单的任务，比如打印额外的信息，此时子进程的开辟就很不划算，因为要拷贝大量与打印任务无关的父进程中的数据。

为了解决这个问题，vfork应用而生。vfork通过父子进程共享同一块内存的方式，避免了拷贝只读的数据。后来的fork抛弃了内存拷贝的实现方式，使用COW技术实现，已经完全替代了vfork的功能。因此，现今的vfork基本处于废弃状态，使用fork就行了。

### 3. 进程的消亡及资源释放

```c
    #include <sys/types.h>
    #include <sys/wait.h>

    pid_t wait(int *wstatus);

    pid_t waitpid(pid_t pid, int *wstatus, int options);
```

### 4. exec函数族

```c
    #include <unistd.h>

    extern char **environ;

    int execl(const char *pathname, const char *arg, ... /* (char  *) NULL */);
    int execlp(const char *file, const char *arg, ... /* (char  *) NULL */);
    int execle(const char *pathname, const char *arg, ...
        /*, (char *) NULL, char *const envp[] */);
    int execv(const char *pathname, char *const argv[]);
    int execvp(const char *file, char *const argv[]);
    int execvpe(const char *file, char *const argv[], char *const envp[]);
```

### 5. 用户权限和组权限

### 6. 观摩课：解释器文件

### 7. 系统调用system()

### 8. 进程会计

### 9. 进程时间

### 10. 守护进程

### 11. 系统日志

## P168 ~ P169 进程 - fork实例

### 1. 简单fork实例

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/process_basic/fork1.c)

写一个简单的fork实例代码，如下所示：

```c
    pid_t pid;

    printf("pid [%d] : begin ...\n", getpid());

    pid = fork();
    if (pid < 0) { /* ... */ }

    if (pid == 0) {
        // child
        printf("pid [%d] : I'm child ...\n", getpid());
    } else {
        // parent
        printf("pid [%d] : I'm parent ...\n",getpid());
    }

    printf("pid [%d] : end ...\n", getpid());
    exit(0);
```

解释：

（1）fork完成后，子进程通过复制父进程而产生，随后父子进程的执行顺序由调度器决定

（2）打印begin在fork之前，所以只有父进程会打印这句话

（3）fork完成之后的打印顺序是不确定的，同样是因为这取决于具体的调度策略

上述代码的运行结果之一为：

```shell
pid [2657] : begin ...
pid [2657] : I'm parent ...
pid [2657] : end ...
pid [2658] : I'm child ...
pid [2658] : end ...
```

**进程父子关系**

在程序`exit(0)`之前，暂停进程，使用`ps axf`命令查看父子进程的关系。如下所示：

```shell
   2248 pts/0    Ss     0:00  |       \_ -bash
   3070 pts/0    S+     0:00  |           \_ ./fork
   3071 pts/0    S+     0:00  |               \_ ./fork
```

可以看到，bash进程创建了父进程fork，然后父进程通过fork调用创建了子进程fork。

**begin为什么会打印两次**

和上面一样的程序，将程序输出重定向到文件中。发现begin打印了两次，且两次begin都是父进程打印的，为什么会这样？

```shell
$ ./fork > /tmp/out
$ cat /tmp/out
pid [3588] : begin ...
pid [3588] : I'm parent ...
pid [3588] : end ...
pid [3588] : begin ...
pid [3589] : I'm child ...
pid [3589] : end ...
```

如果在fork之前刷新所有打开的流，再重复之前的步骤，发现begin只打印了一次，为什么是这样？

```c
    printf("pid [%d] : begin ...\n", getpid());
    fflush(NULL);

    pid = fork();
    if (pid < 0) { /* ... */ }
```

```shell
$ ./fork > /tmp/out
$ cat /tmp/out
pid [3963] : begin ...
pid [3963] : I'm parent ...
pid [3963] : end ...
pid [3964] : I'm child ...
pid [3964] : end ...
```

原因在于，**printf中的内容还没往文件/tmp/out中写入，还在缓冲区的时候，开始fork子进程。此时，父子进程的缓冲区中各有一句begin的打印。即使printf的输出内容带了换行符也不行，因为文件是全缓冲，换行符不会触发缓冲区的刷新**。

所以，在每次fork子进程之前都需要调用fflush将父进程打开的流给刷新一遍。

### 2. 求取质数的多进程并发版本

[完整源码](https://github.com/wallace-lai/learn-apue/tree/main/src/con/process_basic)

先写一个单进程版本，如下所示：

```c
    for (int i = BEG; i <= END; i++) {
        mark = 1;
        for (int p = 2; p < i / 2; p++) {
            if (i % p == 0) {
                mark = 0;
                break;
            }
        }

        if (mark) {
            printf("%d is a primer.\n", i);
        }
    }
```

改造成多进程版本，思路是每枚举到一个i值，将判断i是否为质数的功能扔给子进程去完成。

```c
    for (int i = BEG; i <= END; i++) {
        pid_t pid = fork();
        if (pid < 0) {
            perror("fork()");
            exit(1);
        }

        if (pid == 0) {
            mark = 1;
            for (int p = 2; p < i / 2; p++) {
                if (i % p == 0) {
                    mark = 0;
                    break;
                }
            }

            if (mark) {
                printf("%d is a primer.\n", i);
            }

            exit(0);
        }
    }
```

解释：

（1）父进程只负责创建子进程

（2）判断i是否为质数的工作交给子进程去完成

（3）注意子进程判断完毕无比调用exit(0)退出，否则系统有死机风险


比较这两个程序的运行所需时间，结果如下：

```shell
$ time ./primer1 > /dev/null

real    0m0.972s
user    0m0.944s
sys     0m0.027s

$ time ./primer2 > /dev/null

real    0m0.096s
user    0m0.000s
sys     0m0.018s
```

## P170 进程 - init进程和vfork

让子进程在退出前睡眠一段时间，让父进程正常结束，使用`ps axf`查看进程详情如下：

```shell
   2317 pts/0    S      0:00 ./primer2
   2318 pts/0    S      0:00 ./primer2
   2319 pts/0    S      0:00 ./primer2
   2320 pts/0    S      0:00 ./primer2
   2321 pts/0    S      0:00 ./primer2
   2322 pts/0    S      0:00 ./primer2
   2323 pts/0    S      0:00 ./primer2
   2324 pts/0    S      0:00 ./primer2
   2325 pts/0    S      0:00 ./primer2
   2326 pts/0    S      0:00 ./primer2
```

解释：

（1）子进程的状态为S，表示处于睡眠状态

（2）子进程顶格显示（没有显示其父进程），说明此时子进程的父进程成了init进程

注：父进程提前退出后，子进程将被init进程收养

让父子进程在退出前都睡眠一段时间，查看进程详情如下：

```shell
   1775 pts/0    Ss     0:00  |       \_ -bash
   3057 pts/0    S+     0:00  |           \_ ./primer2
   3058 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3059 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3060 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3061 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3062 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3063 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3064 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3065 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3066 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
   3067 pts/0    Z+     0:00  |               \_ [primer2] <defunct>
```

解释：

（1）子进程状态为Z，僵尸态。父进程状态为S，睡眠态

（2）由于子进程已经结束了，但是父进程没有回收它们的资源，导致子进程“无人收尸”

（3）当父进程被唤醒后，僵尸态的子进程将被init进程收养，其资源由init进程释放

## P171 进程 - wait和waitpid

### 1. 接口
```c
    #include <sys/types.h>
    #include <sys/wait.h>

    pid_t wait(int *wstatus);

    pid_t waitpid(pid_t pid, int *wstatus, int options);
```

解释：

（1）wait指的是wait for process to change state

（2）子进程的状态通过`wstatus`指针带回

**子进程返回状态检测**

（1）`WIFEXITED(wstatus)`：如果子进程正常结束，则该宏为真

注：进程正常终止的三种方式

- 调用`exit()`

- 调用`_exit()`

- 从`main()`中返回

（2）`WEXITSTATUS(wstatus)`：如果子进程正常结束，则该宏可以打印子进程结束的返回码

（3）`WIFSIGNALED(wstatus)`：如果子进程被信号终止，则该宏为真

（4）`WTERMSIG(wstatus)`：如果子进程被信号终止，则该宏可以打印造成子进程终止的信号编号


**更高级的等待子进程返回**

waitpid相比wait更加复杂一点，具体：

（1）`pid`值可以是：

- `< -1`： meaning wait for any child process whose process group ID is equal to the absolute value of pid（比如pid值为-5，表示等待组号为5的任意一个子进程）

- `-1`：meaning wait for any child process（等待任意一个子进程）

- `0`： meaning wait for any child process whose process group ID is equal to that of the calling process at the time of the call to waitpid()（等待与我同组的任意一个子进程）

- `> 0`： meaning wait for the child whose process ID is equal to the value of pid（等待指定的子进程）

（2）`options`可以是：

- `WNOHANG`：return immediately if no child has exited（由阻塞变成了非阻塞）

所以，wait相当于如下的waitpid：

```c
waitpid(-1, &wstatus, 0);
```

### 2. wait使用案例

我们将求解质数的多进程并发版本写完整，如下所示：

```c
    for (int i = BEG; i <= END; i++) {
        pid_t pid = fork();
        if (pid < 0) {
            perror("fork()");
            exit(1);
        }

        if (pid == 0) {
            // ...
            exit(0);
        }
    }

    for (int i = BEG; i <= END; i++) {
        wait(NULL);
    }
```

解释：

（1）子进程判断i是否为质数，记得最后一定要使用exit退出

（2）父进程在创建完子进程后，使用wait等待所有的子进程结束，且不关心子进程退出状态

## P172 进程 - 进程的交叉分配法实现

假设将`[1, 201]`范围内的数是否为质数的任务分配给3个进程，有以下的进程任务分配的方法：

（1）分块法：将范围平均划分成前中后三个部分，分别分配给3个进程

缺点：进程负载不均衡，显然进程1的负载最重（最小范围内的质数密度最高）


（2）交叉分配法：类似发牌的方式，将范围内的数一个个地按序分配给3个进程

缺点：仍然存在负载不均衡的问题！显然进程1拿到的数一定是3的倍数，它不可能是质数，因此进程1的负载基本为空。

注意：由于求解质数的案例比较特殊，输入是一个范围内的连续整数，所以导致交叉分配法的性能拉胯。实际上，在一般的任务中，交叉分配法的性能是比较好的。

（3）池类算法：将任务放入容器中，由进程从容器中抢任务，谁抢到了谁执行。一般而言，池类算法具有很好的随机性。

![池类算法](../media/images/Language/linux-c6.png)

### 1. 将求解质数的程序改造成交叉分配法

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/process_basic/primern.c)

```c
    for (int n = 0; n < N; n++) {
        pid_t pid = fork();
        if (pid < 0) { /* ... */ }

        if (pid == 0) {
            // child
            for (int i = BEG + n; i <= END; i += N) {
                if (is_primer(i)) {
                    printf("[%d] %d is primer number\n", n, i);
                }
            }

            exit(0);
        }
    }

    for (int n = 0; n < N; n++) {
        wait(NULL);
    }
```

解释：

（1）父进程循环创建N个子进程，每个子进程只选择能整除进程编号n的数去判断

（2）父进程等待N个子进程结束

代码执行结果：

```shell
$ ./primern
[2] 30000023 is primer number
[1] 30000001 is primer number
[1] 30000037 is primer number
[2] 30000041 is primer number
[1] 30000049 is primer number
[2] 30000059 is primer number
[1] 30000079 is primer number
[2] 30000071 is primer number
[1] 30000109 is primer number
[2] 30000083 is primer number
[1] 30000133 is primer number
[2] 30000137 is primer number
[1] 30000163 is primer number
[2] 30000149 is primer number
[1] 30000169 is primer number
[2] 30000167 is primer number
[1] 30000193 is primer number
[1] 30000199 is primer number
```

可以看到0号进程确实没有处理一个质数

## P173 进程 - exec函数族

`few`三个字符搭建了整个Linux的进程框架，它们是：

（1）`fork`：创建子进程

（2）`exec`：执行文件

（3）`wait`：等待子进程退出

### 1. 接口

The exec() family of functions replaces the current process image with a new process image.

```c
#include <unistd.h>

extern char **environ;

int execl(const char *pathname, const char *arg, ... /* (char  *) NULL */);
```

（1）`pathname`：可执行程序所在的路径

（2）`arg`：给程序的传参，多个参数最后以NULL结尾

```c
int execlp(const char *file, const char *arg, ... /* (char  *) NULL */);
```

（1）`file`：可执行程序的名称，从`environ`中指定的路径下去查找该程序

（2）`arg`：给程序的传参

```c
int execle(const char *pathname, const char *arg, ... 
    /*, (char *) NULL, char *const envp[] */);
```

（1）`envp`：给程序传递的环境变量

```c
int execv(const char *pathname, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[],
    char *const envp[]);
```

（1）`argv`：以argv的形式传参

### 2. exec使用案例之打印时间戳1

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/process_basic/exe.c)

```c
int main()
{
    puts("my date begin ...");

    execl("/usr/bin/date", "date", "+%s", NULL);
    perror("execl()");
    exit(1);

    puts("my date end ...");
    exit(0);
}
```

解释：

（1）使用`execl`直接将当前进程的进程映像替换成data的

（2）如果替换不成功，那么程序会往下走`perror`报错退出；如果替换成功，那么当前整个镜像都会被干掉

（3）可以预想到`my date end`是不会打印出来的，因为main的进程镜像被干掉了

运行结果：

```shell
$ ./exe
my date begin ...
1718810449
```

如果将输出给重定向的话，查看结果：

```shell
$ ./exe > /tmp/out
$ cat /tmp/out
1718810906
```
`my date begin`的打印为什么消失了？因为文件（往文件`/tmp/out`中写入）是全缓冲，所以换行符不会触发缓冲区的刷新。`execl`执行后，在缓冲区的`my date begin`还没来得及打印就随着整个进程镜像的替换而被干掉了，自然也就打印不了了。

由此得出一个结论：**和fork类似，在执行exec之前也需要调用fflush刷新进程当前打开的流**！

```c
int main()
{
    puts("my date begin ...");

    fflush(NULL);
    execl("/usr/bin/date", "date", "+%s", NULL);
    perror("execl()");
    exit(1);

    puts("my date end ...");
    exit(0);
}
```

再看程序结果：

```shell
$ ./exe > /tmp/out
$ cat /tmp/out
my date begin ...
1718811669
```


如何形容`execl`？**你还是你（pid号以及父子进程的关系不变），但你已经不是你了（进程映像已经被替换了）**！

### 3. exec使用案例之打印时间戳2

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/process_basic/few.c)

使用`few`实现打印时间戳，即创建子进程并替换子进程的映像，让子进程去执行时间戳的打印任务，父进程回收子进程。

```c
    puts("my date begin ...");

    fflush(NULL);
    pid_t pid = fork();
    if (pid < 0) { /* ... */ }

    if (pid == 0) {
        // child
        execl("/usr/bin/date", "date", "+%s", NULL);
        perror("execl()");
        exit(1);
    }

    wait(NULL);

    puts("my date end ...");
    exit(0);
```

## P174 ~ P175 进程 - shell命令实现原理

为什么fork出来的子进程和父进程内容打印在了同一个终端中？

![父子进程共享终端](../media/images/Language/linux-c7.png)

因为fork出来的子进程的打开文件列表和父进程是一样的，前3个文件都指向了同一个设备。

### 1. mysh的简易shell实现

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/process_basic/mysh.c)

这次案例我们需要实现自己的一个建议shell命令，目前只支持内部命令。

```c
    while (1) {
        prompt();

        char *buffer = NULL;
        size_t buffer_size = 0;

        ret = getline(&buffer, &buffer_size, stdin);
        if (ret < 0) { /* ... */ }

        cmd_t cmd;
        parse(buffer, &cmd);

        if (0) {
            // ...
        } else {
            fflush(NULL);
            pid_t pid = fork();
            if (pid < 0) { /* ... */ }

            if (pid == 0) {
                execvp(cmd.result.gl_pathv[0], cmd.result.gl_pathv);
                perror("execvp()");
                exit(1);
            } else {
                wait(NULL);
            }
        }
    }
```

解释：

（1）父进程是一个死循环，第一步先打印提示符

（2）第二步使用getline获取用户输入

（3）第三步使用parse解析用户的输入

（4）第四部fork子进程去执行命令

（5）最后父进程等待子进程结束

```c
typedef struct cmd_s {
    glob_t result;
} cmd_t;

static void prompt(void)
{
    printf("mysh - 0.1 >>> ");
}

static void parse(char *line, cmd_t *cmd)
{
    // ls -l -a -h /home/xyz/
    int i = 0;
    char *tok = NULL;

    while (1) {
        tok = strsep(&line, DELIMS);
        if (tok == NULL) {
            break;
        }
        if (tok[0] == '\0') {
            continue;
        }

        if (glob(tok, GLOB_NOCHECK | GLOB_APPEND * i, NULL, &cmd->result) != 0) {
            break;
        }
        i = 1;
    }
}
```

解释：

（1）使用strsep分割完整字符串

（2）如果为NULL说明分割完毕，退出循环

（3）如果为`\0`说明当前分割的子串为空，跳过

（4）使用glob将所有分割的子串组装成类似`argv[]`的结构

## P176 进程 - 用户权限和组权限实现原理

普通用户没有查看和修改shadow文件的权限，但是却可以使用passwd命令修改自身账号的密码，这是为何？

```shell
$ cat /etc/shadow
cat: /etc/shadow: Permission denied
$ passwd
Changing password for bob.
Current password:
```

