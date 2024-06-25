# 【Linux C】并发操作

作者：wallace-lai <br>
发布：2024-05-29 <br>
更新：2024-06-25 <br>

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

```c
    #include <stdlib.h>

    int system(const char *command);
```

```
The system() library function uses fork(2) to create a child process that executes the shell command
specified in command using execl(3) as follows:

    execl("/bin/sh", "sh", "-c", command, (char *) NULL);

system() returns after the command has been completed.
```

### 8. 进程会计

```c
    #include <unistd.h>

    int acct(const char *filename);
```

### 9. 进程时间

```c
    #include <sys/times.h>

    clock_t times(struct tms *buf);

    struct tms {
        clock_t tms_utime;  /* user time */
        clock_t tms_stime;  /* system time */
        clock_t tms_cutime; /* user time of children */
        clock_t tms_cstime; /* system time of children */
    };
```

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

## P176 ~ P177 进程 - 用户权限和组权限实现原理

普通用户没有查看和修改shadow文件的权限，但是却可以使用passwd命令修改自身账号的密码，这是为何？

```shell
$ cat /etc/shadow
cat: /etc/shadow: Permission denied
$ passwd
Changing password for bob.
Current password:
```

### 1. 用户权限鉴定流程

下面的内容没太理解，先做个记录。

（1）uid和sid均有三个，分别是

- r : real uid/sid

- e : effective uid/sid

- s : save uid/sid

其中s可以没有，但一定会有r和e，鉴权的时候会看r和e。

（2）可以看到passwd命令是有u+s权限的（`rws`）

```shell
$ which passwd
/usr/bin/passwd
$ ls -lah /usr/bin/passwd
-rwsr-xr-x 1 root root 67K Feb  6 12:49 /usr/bin/passwd
```

当一个程序具有u+s权限说明当别的用户来调用该程序时，它的身份会切换成当前程序所属用户的身份。对于passwd命令来说，该用户是root。

如下图所示，当exec在执行passwd命令时发现它有u+s权限。于是passwd的权限不再是继承而来的res值，e和s会变成0，也即passwd实际是以root用户的权限在执行。passwd命令作为子进程执行完后销毁，不复存在。

因此，写passwd命令的程序员应该万分小心，因为passwd是以root的权限在执行的。

![用户权限鉴定](../media/images/Language/linux-c8.png)

### 2. shell命令执行时的身份（账号）是怎么来的

![获取身份信息](../media/images/Language/linux-c9.png)

解释：

（1）初始init进程使用fork和exec产生getty进程，此时屏幕上会提示你输入用户名

（2）getty进程拿到用户名后使用exec产生login进程，此时屏幕上会提示你继续输入用户密码

（3）此时login进程仍然是以root权限在运行，并且已经拿到了用户名和密码。通过使用用户名和密码加上盐值salt的方式得到一个加密哈希串，将该串与shadow文件中的串进行对比，如果相同则说明用户密码正确，成功登录系统。

（4）如果登录成功，则login进程使用fork和exec创建shell进程，这个shell进程就拿到了你的身份（账户）信息

### 3. 相关接口

- getuid
- geteuid
- getgid
- getegid

- setuid
- setgid
- setreuid
- setregid

### 4. mysu的实现

mysu是一个简单的实现类似sudo功能的程序，我们希望能通过mysu来达到获取root权限来执行命令的功能。如下所示，其中0表示root用户的uid值。

```shell
./mysu 0 cat /etc/passwd
```

代码实现如下：

```c
int main(int argc, char **argv)
{
    // ./mysu 0 cat /etc/passwd
    if (argc < 3) {
        fprintf(stderr, "Usage ...\n");
        exit(1);
    }

    pid_t pid = fork();
    if (pid < 0) {
        perror("fork()");
        exit(1);
    }

    if (pid == 0) {
        // child
        setuid(atoi(argv[1]));
        execvp(argv[2], argv + 2);
        perror("execvp()");
        exit(1);
    }

    wait(NULL);
    exit(0);
}
```

解释：

（1）子进程通过setuid将子进程的res均设置为0，即root用户的uid，随后子进程就拥有的root用户权限

（2）子进程获取root用户权限后，使用execvp执行后面传入的命令

（3）父进程等待子进程运行结束

执行mysu命令，发现没有权限！很简单，**要是谁都能通过setuid来获取root权限那还要`su`命令做什么**？

```shell
$ ./mysu 0 cat /etc/shadow
cat: /etc/shadow: Permission denied
```

给mysu设置u+s权限，将root用户权限下放给mysu程序（这个过程需要有root权限）。

（1）查看mysu的owner为xxx

```shell
$ ls -lah mysu
-rwxrwxr-x 1 xxx  xxx  17K Jun 22 10:34 mysu
```

（2）尝试将mysu的owner设置成root，发现需要root权限才能设置

```shell
$ chown root mysu
chown: changing ownership of 'mysu': Operation not permitted
$ sudo chown root mysu
[sudo] password for xxx:
$ ls -lah mysu
-rwxrwxr-x 1 root xxx 17K Jun 22 10:34 mysu
$
```

（3）设置u+s权限

```shell
$ ls -lah mysu
-rwxrwxr-x 1 root xxx 17K Jun 22 10:34 mysu
$ chmod u+s mysu
chmod: changing permissions of 'mysu': Operation not permitted
$ sudo chmod u+s mysu
$ ls -lah mysu
-rwsrwxr-x 1 root xxx 17K Jun 22 10:34 mysu
```

此时，mysu获取了u+s权限，再次执行mysu时将会将root权限下放给mysu程序。此时，mysu中的setuid才能成功执行

```shell
$ ./mysu 0 cat /etc/shadow
...
```

### 5. 解释器文件

下面是一个简单的shell脚本：

```shell
#!/bin/bash

ls
whoami
cat /etc/shadow
ps
```

shell发现文件开头的解释器文件标识符`#!`后会把你要的`bin/bash`加载进来，然后将整个脚本内容直接扔给`bin/bash`。运行结果如下：

```shell
$ ./t.sh
exe.c  few.c  fork1.c  mysh.c  mysu.c  primer1.c  primer2.c  primern.c  t.sh
xxx
cat: /etc/shadow: Permission denied
    PID TTY          TIME CMD
  14285 pts/0    00:00:00 bash
  21206 pts/0    00:00:00 t.sh
  21210 pts/0    00:00:00 ps
```

将shell内容改成以下的内容：

```shell
#!/bin/cat

ls
whoami
cat /etc/shadow
ps
```

运行结果如下，cat直接将脚本内容给打印出来了。

```shell
$ ./t.sh
#!/bin/cat

ls
whoami
cat /etc/shadow
ps
```

## P178 进程 - system、进程会计、进程时间

### 1. system
system()可以看成few的封装，使用system来输出时间戳。

```c
int main()
{
    system("date +%s > /tmp/out");

    exit(0);
}
```

对应的，这是few的实现：

```c
    pid_t pid = fork();
    if (pid < 0) { /* ... */ }

    if (pid == 0) {
        execl("/usr/bin/date", "date", "+%s", NULL);
        perror("execl()");
        exit(1);
    }

    wait(NULL);
```

而system()本质上是这样实现的：

```c
    pid_t pid = fork();
    if (pid < 0) { /* ... */ }

    if (pid == 0) {
        execl("/bin/sh", "sh", "-c", "date +%s", NULL);
        perror("execl()");
        exit(1);
    }

    wait(NULL);
```

即先调用shell，将命令传递给shell执行。

### 2. 进程会计

```c
    #include <unistd.h>

    int acct(const char *filename);
```

```
DESCRIPTION
       The  acct()  system  call  enables or disables process accounting.  If called
       with the name of an existing file as its argument, accounting is  turned  on,
       and  records for each terminating process are appended to filename as it ter‐
       minates.  An argument of NULL causes accounting to be turned off.
```

注意：这个接口只是BSD系统的方言，不在POSIX标准里面

### 3. 进程时间

```c
    #include <sys/times.h>

    clock_t times(struct tms *buf);

    struct tms {
        clock_t tms_utime;  /* user time */
        clock_t tms_stime;  /* system time */
        clock_t tms_cutime; /* user time of children */
        clock_t tms_cstime; /* system time of children */
    };
```

## P179 - 守护进程

**没太理解本节的守护进程的概念**。

守护进程的特点：

（1）脱离控制终端而存在

（2）会话和进程组的leader

### 1. 接口
```c
setsid()
getpgrp()
getpgid()
setpgid()
```
setsid() creates a new session if the calling process is not a process group
leader. The calling process is the leader of the new session (i.e., its session ID
is  made the same as its process ID).  The calling process also becomes the process
group leader of a new process group in the session (i.e., its process group  ID  is
made the same as its process ID)
```



使用`ps axj`查看系统中的守护进程

```shell
$ ps axj
   PPID     PID    PGID     SID TTY        TPGID STAT   UID   TIME COMMAND
      0       1       1       1 ?             -1 Ss       0   0:03 /sbin/init auto automatic-
      0       2       0       0 ?             -1 S        0   0:00 [kthreadd]
      2       3       0       0 ?             -1 I<       0   0:00 [rcu_gp]
      2       4       0       0 ?             -1 I<       0   0:00 [rcu_par_gp]
      2       6       0       0 ?             -1 I<       0   0:00 [kworker/0:0H-kblockd]
      2       8       0       0 ?             -1 I<       0   0:00 [mm_percpu_wq]
      2       9       0       0 ?             -1 S        0   0:00 [ksoftirqd/0]
      2      10       0       0 ?             -1 I        0   0:02 [rcu_sched]
      2      11       0       0 ?             -1 S        0   0:00 [migration/0]
      1     928     928     928 ?             -1 Ssl      0   0:00 /usr/lib/accountsservice/a
      1     935     935     935 ?             -1 Ss       0   0:00 /usr/sbin/cron -f
      1     937     937     937 ?             -1 Ss     103   0:00 /usr/bin/dbus-daemon --sys
      1     946     946     946 ?             -1 Ssl      0   0:00 /usr/sbin/irqbalance --for
      1     949     949     949 ?             -1 Ss       0   0:00 /usr/bin/python3 /usr/bin/
      1     950     950     950 ?             -1 Ssl      0   0:00 /usr/lib/policykit-1/polki
      1     953     953     953 ?             -1 Ssl    115   0:20 /usr/lib/erlang/erts-14.0.
      1     958     958     958 ?             -1 Ssl      0   0:02 /usr/lib/snapd/snapd
      1     969     969     969 ?             -1 Ss       0   0:00 /lib/systemd/systemd-login
      1     973     973     973 ?             -1 Ssl      0   0:00 /usr/lib/udisks2/udisksd
      1     974     974     974 ?             -1 Ss       1   0:00 /usr/sbin/atd -f
      1     979     979     979 ?             -1 Ssl      0   0:00 /usr/sbin/ModemManager
```

守护进程的特征：

（1）PID、PGID、SID是相同的

（2）TTY是问号表示已经脱离了终端

### 2. 守护进程实例

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/process_basic/daemon/mydaemon.c)

下面是一个简单的守护进程的实现。

```c
static int daemonize()
{
    pid_t pid = fork();
    if (pid < 0) {
        perror("fork()");
        return -1;
    }

    if (pid > 0) {
        // parent
        exit(0);
    }

    int fd = open("/dev/null", O_RDWR);
    if (fd < 0) {
        perror("open()");
        return -1;
    }

    dup2(fd, 0);
    dup2(fd, 1);
    dup2(fd, 2);
    if (fd > 2) {
        close(fd);
    }

    setsid();
    chdir("/");
    umask(0);

    return 0;
}
```

解释：

（1）父进程通过调用daemonize()来成为守护进程

（2）正常创建子进程，对于父进程而言正常结束即可

（3）对于子进程，需要在子进程中将前3个流给绑定到`/dev/null`中

（4）通过调用setsid让父进程成为守护进程

（5）子进程正常退出前更改守护进程的工作目录为`/`同时去除umask如果守护进程不创建文件的话

```c
int main()
{
    if (daemonize()) {
        exit(1);
    }

    FILE *fp = fopen(FNAME, "w");
    if (fp == NULL) {
        perror("fopen()");
        exit(1);
    }

    unsigned i = 0;
    while (1) {
        fprintf(fp, "%d\n", i);
        fflush(fp);
        sleep(1);

        i++;
    }

    exit(0);
}
```

解释：

（1）父进程是守护进程的主体，要做的事就是打开`/tmp/out`文件，每隔1s向其中打印计数器值

## P180 进程 - 系统日志

由于守护进程已经和TTY分离了，如果需要得到守护进程的打印内容，需要使用系统日志功能。

（1）系统日志所在路径：`/var/log`

（2）提交系统日志需要用到syslogd服务

### 1. 接口

```c
    #include <syslog.h>

    void openlog(const char *ident, int option, int facility);
    void syslog(int priority, const char *format, ...);
    void closelog(void);

    void vsyslog(int priority, const char *format, va_list ap);
```

```
closelog, openlog, syslog, vsyslog - send messages to the system logger
```

（1）openlog：与syslogd关联

（2）syslog：提交log内容

（3）closelog：与syslogd断联

### 2. 带系统日志功能的守护进程实例

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/process_basic/daemon/mydaemon.c)

### 3. 单实例的守护进程实例

为了实现以单实例的形式（只能启动一个守护进程）启动守护进程，需要用到文件锁。锁文件位于`/var/run/name.pid`之中。

```shell
$ ls -lah /var/run/sshd.pid
-rw-r--r-- 1 root root 5 Jun 22 14:35 /var/run/sshd.pid
$ cat /var/run/sshd.pid
1014
$ ps axj | grep sshd
      1    1014    1014    1014 ?             -1 Ss       0   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
$
```

解释：

（1）守护进程sshd的锁文件是/var/run/sshd.pid

（2）sshd.pid里面保存的是sshd守护进程的pid号1014

## P182 并发 - 异步事件处理的两种方法

异步事件处理的两种方法：

（1）查询法：异步事件发生的频率很高，采用查询法

（2）通知法：异步事件发生的频率很稀疏，采用通知法

### 1. 信号
（1）信号的概念

信号：信号是软件中断，信号的响应依赖于中断。

使用`kill -l`命令可以查看当前的信号

```shell
$ kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```


- 信号编号1 ~ 31：标准信号
- 实时信号32 ~ 64：实时信号

UNIX系统信号列表：

![UNIX系统列表](../media/images/Language/linux-c10.png)



（2）发送信号signal()

```c
    #include <signal.h>

    typedef void (*sighandler_t)(int);

    sighandler_t signal(int signum, sighandler_t handler);
```

注意：**信号会打断阻塞的系统调用**！

（3）信号的不可靠性

标准信号是会丢失的，实时信号不会；

信号的不可靠性：信号处理函数的执行现场是内核处理的，信号的不可靠性指的是**在信号处理函数执行过程中又来了一个相同的信号，第二次的执行现场可能把第一次的执行现场给覆盖了**。


（4）可重入函数

**可重入函数是为了解决信号不可靠问题而产生的概念。所有的Linux系统调用都是可重入的，但只有部分的标准库函数是可重入的**。

```c
    struct tm *localtime(const time_t *timep);
    struct tm *localtime_r(const time_t *timep, struct tm *result);
```

为什么`localtime`是不可重入函数？因为返回的`struct tm *`是指向静态区的，在第一次调用未结束时再次调用`localtime`会把同一块静态区的内容给覆盖掉。`localtime_r`是`localtime`函数的可重入版本，要求你传入一个result作为结果的传出地址。

由此得出结论：**禁止在信号处理函数中调用不可重入函数**！

```c
    void *memcpy(void *dest, const void *src, size_t n);
```

The memcpy() function  copies n bytes from memory area src to memory area dest. The memory
areas must not overlap. Use memmove(3) if the memory areas do overlap.

为什么当dest和src两块内存有重叠时要用memmove？同样是因为有重叠时的memcpy是不可重入的。


（5）信号的响应过程

思考：
- 信号从收到到响应有一个不可避免的延迟。
- 如何忽略一个信号？
- 标准信号为什么要丢失？


（6）信号常用函数

- kill()
- raise()
- alarm()
- pause()
- abort()
- system()
- sleep()


（7）信号集

（8）信号屏蔽字 / pending集合 的处理 

（9）扩展

- sigsuspend()
- sigaction()
- setitimer()

（10）实时信号

### 2. 线程

## P183 ~ P184 并发 - 信号的基本概念

### 1. signal()接口

```c
    #include <signal.h>

    typedef void (*sighandler_t)(int);

    sighandler_t signal(int signum, sighandler_t handler);
```

注意：**不建议使用signal()，尽量使用sigaction()代替该函数**

```c
    #include <signal.h>

    int sigaction(int signum, const struct sigaction *act,
        struct sigaction *oldact);
```

### 2. 简单signal实例

写一个简单的打印星号的程序：

```c
int main()
{
    for (int i = 0; i < 10; i++) {
        write(1, "*", 1);
        sleep(1);
    }

    exit(0);
}
```

使用CTRL + C打断程序的执行：

```shell
$ ./star
*****^C
$
```

（1）使用`CTRL+C`相当于是发送了`SIGINT`信号，即`CTRL+C`是`SIGINT`的快捷键

（2）`SIGINT`信号的默认动作是终止程序

（3）使用`CTRL+\`相当于是发送了`SIGQUIT`信号

使用预定义的`SIG_IGN`忽略`SIGINT`信号：

```c
int main()
{
    signal(SIGINT, SIG_IGN);

    for (int i = 0; i < 10; i++) {
        write(1, "*", 1);
        sleep(1);
    }

    exit(0);
}
```

运行查看结果，可以看到再次输入`CTRL+C`时程序不再退出了

```shell
$ ./star
***^C**^C****^C*$
```

收到信号时打印一个叹号：

```c
void on_sigint_handler(int s)
{
    write(1, "!", 1);
}

int main()
{
    signal(SIGINT, on_sigint_handler);

    for (int i = 0; i < 10; i++) {
        write(1, "*", 1);
        sleep(1);
    }

    exit(0);
}
```

运行查看结果，发现只要程序还未退出那么只要收到`SIGINT`信号，就会打印一个叹号出来。注意`^C`只是`CTRL+C`对应的回显，表示收到了一个`CTRL+C`的输入而已。

```shell
$ ./star
****^C!**^C!*^C!*^C!**$
```

程序运行后按住`CTRL+C`不放，发现程序很快就结束了，远不到规定的10秒钟。由此得到结论**信号会打断阻塞的系统调用**！

```shell
$ ./star
*^C!*^C!*^C!*^C!*^C!*^C!*^C!*^C!*^C!*^C!$
```

### 3. 防止EINTR信号干扰

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/parallel/signal/mycpy.c)

在mycpy程序中，由于使用的open和read以及write系统调用都是阻塞的。而信号可以打断阻塞的系统调用，所以需要对`EINTR`信号进行处理。

```c
    do {
        sfd = open(argv[1], O_RDONLY);
        if (sfd < 0) {
            if (errno != EINTR) {
                perror("open()");
                exit(1);
            }
        }
    } while (sfd < 0);


    do {
        dfd = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0600);
        if (dfd < 0) {
            if (errno != EINTR) {
                close(sfd);
                perror("open()");
                exit(1);
            }
        }
    } while (dfd < 0);
```

解释：

（1）只有errno不是EINTR时才认为open出错了，如果是被EINTR信号打断，则重试即可

```c
    while (1) {
        len = read(sfd, buffer, BUFSIZE);
        if (len < 0) {
            if (errno == EINTR) {
                continue;
            }
            perror("read()");
            break;
        }
        if (len == 0) {
            break;
        }

        pos = 0;
        while (len > 0) {
            ret = write(dfd, buffer + pos, len);
            if (ret < 0) {
                if (errno == EINTR) {
                    continue;
                }
                perror("write()");
                exit(1);
            }

            pos += ret;
            len -= ret;
        }
    }
```

解释：

（1）read和write这两个同步系统调用同样要防止EINTR信号的干扰

## P185 ~ P186 并发 - 信号的响应过程

思考三个问题：

（1）信号从收到到响应有一个不可避免的延迟。

（2）如何忽略一个信号？

（3）标准信号为什么要丢失？

（4）标准信号的响应没有严格的顺序？

从进程角度讲解信号的响应过程，首先内核会为每一个进程都维护一组位图，如下所示：

![信号响应过程](../media/images/Language/linux-c11.png)


（1）mask表示信号屏蔽字，信号屏蔽字用来表示当前信号的状态

（2）pending用来记录当前的进程收到了哪些信号

理论上，mask和pending都是32位的，和标准信号的数量是对应的。一般情况下，mask位图值全部为1，pending位图值全部为0。

（1）当接收到`SIGINT`信号时，其对应的pending位图将会被置为1。此时的信号还无法被及时响应；

（2）当从Kernel态切换成User态时（被中断打断），main函数的所对应的现场（保存了addr值）将会被加入等待队列中，等待被调度。

（3）当main被调度时，Kernel态切换成User态，内核会做`mask & pending`的操作，此时`SIGINT`信号的操作值为1（因为mask和pending均为1），表示收到了`SIGINT`信号

（4）此时将mask和pending均置成0，同时将保存现场的addr值替换成信号处理函数的地址，去执行信号处理函数（比如打印一个叹号）。执行完毕后，再次回到内核态。信号处理函数执行完毕后，将mask恢复成1表示该信号已经被响应完毕了。

（5）再次从Kernel态切换到User态，将addr值恢复成原先的值（返回main被中断处）。如果再没有信号过来，那么就会返回main被中断的地方继续执行（继续打印星号）


为什么信号从收到到响应会有一个不可避免的时延？因为只有内核被时钟中断打断时，才能发现并处理接收到的信号。可以看到信号需要借助中断的机制来实现。

如何忽略一个信号？将信号的mask位置成0即可。无法阻止信号到来，但可以选择不响应它。

标准信号为什么要丢失？因为同一个响应周期内哪怕有多个信号过来，对pending的处理结果都是一样的（置为1），所以只会响应一次，也即后面的相同信号会被丢失。


## P187 并发 - 信号相关接口

### 1. 接口

（1）发送信号kill()

```c
    #include <sys/types.h>
    #include <signal.h>

    int kill(pid_t pid, int sig);
```

The kill() system call can be used to send any signal to any process group or process.

pid的值：

- If pid is positive, then signal sig is sent to the process with the ID specified by
pid.

- If  pid  equals  0,  then  sig is sent to every process in the process group of the
calling process.

- If pid equals -1, then sig is sent to every process for which the  calling  process
has permission to send signals, except for process 1 (init), but see below.

- If  pid  is  less  than  -1, then sig is sent to every process in the process group
whose ID is -pid.

sig的值：

If sig is 0, then no signal is sent, but existence and permission checks are  still
performed;  this  can be used to check for the existence of a process ID or process
group ID that the caller is permitted to signal.

返回值：

On  success  (at least one signal was sent), zero is returned.  On error, -1 is re‐
turned, and errno is set appropriately.

（2）给调用线程发信号raise()

```c
    #include <signal.h>

    int raise(int sig);
```

The raise() function sends a signal to the calling process or thread.  In a single-
threaded program it is equivalent to（个人理解：**single-thread下用getpid，实际上暗示进程就是单线程**）
```c
    kill(getpid(), sig);
```
In a multithreaded program it is equivalent to
```c
    pthread_kill(pthread_self(), sig);
```
If the signal causes a handler to be called, raise() will  return  only  after  the
signal handler has returned.

返回值：
raise() returns 0 on success, and nonzero for failure.

（3）发送`SIGALRM`信号alarm()

```c
    #include <unistd.h>

    unsigned int alarm(unsigned int seconds);
```

alarm()  arranges  for  a  SIGALRM signal to be delivered to the calling process in
seconds seconds.

If seconds is zero, any pending alarm is canceled.

In any event any previously set alarm() is canceled.

返回值：

alarm() returns the number of seconds  remaining  until  any  previously  scheduled
alarm was due to be delivered, or zero if there was no previously scheduled alarm.

（4）等待信号打断pause()

```c
    #include <unistd.h>

    int pause(void);
```

pause() causes the calling process (or thread) to sleep until a signal is delivered
that either terminates the process or causes the invocation  of  a  signal-catching
function.

返回值：
pause()  returns only when a signal was caught and the signal-catching function re‐
turned.  In this case, pause() returns -1, and errno is set to EINTR.

（5）

### 2. alarm应用实例

简单描述alarm的功能就是：在超过给定的秒数之后，向当前进程发送`SIGALRM`信号。而`SIGALRM`信号的默认动作是退出当前进程。

```c
int main()
{
    alarm(5);

    while (1) {
        ;
    }

    exit(0);
}
```

运行程序，发现在5秒钟之后因收到`SIGALRM`信号而终止：

```shell
$ ./alarm
Alarm clock
```

设置多个时间不同的alarm函数：

```c
int main()
{
    alarm(10);
    alarm(5);
    alarm(1);

    while (1) {
        pause();
    }

    exit(0);
}
```

运行程序，发现在1秒钟之后，程序就终止了。说明**alarm无法应用在多任务计时上面，之后最新的alarm会生效**。


使用pause避免CPU盲等：

```c
int main()
{
    alarm(10);
    alarm(5);
    alarm(1);

    while (1) {
        pause();
    }

    exit(0);
}
```

pause会导致程序休眠，因此不会再空耗CPU了。

注意：为了程序可移植性，不要使用`sleep()`调用，因为该调用在不同的平台上的底层实现可能不一样。如果`sleep()`调用是用alarm + pause的方式实现的，那么在sleep和alarm混用的场景下，alarm的计时就不准确了。



### 3. alarm应用实例之定时循环

让数从0开始不断地自增，然后在5秒钟结束后打印其值。这个案例的目的是为了比较`time()`和`alarm()`这两种计时机制的准确度。

首先是使用time()实现版本：

```c
int main()
{
    unsigned long long count = 0;
    time_t end = time(NULL) + 5;

    while (time(NULL) < end) {
        count++;
    }

    printf("5sec by time : %lld\n", count);
    exit(0);
}
```

运行程序，使用time命令查看程序实际运行时间并把结果重定向，结果如下：

```shell
$ time ./5sec > /tmp/out

real    0m4.738s
user    0m4.697s
sys     0m0.037s
$ cat /tmp/out
5sec by time : 2369860683
```

```shell
$ time ./5sec  > /tmp/out

real    0m4.901s
user    0m4.893s
sys     0m0.006s
$ cat /tmp/out
5sec by time : 2487479287
```

```shell
$ time ./5sec  > /tmp/out

real    0m4.901s
user    0m4.893s
sys     0m0.006s
$ cat /tmp/out
5sec by time : 2487479287
```

然后是使用alarm()实现版本：

```c
static int need_loop = 1;

static void on_alarm_handler(int s)
{
    need_loop = 0;
}

int main()
{
    alarm(5);
    signal(SIGALRM, on_alarm_handler);

    unsigned long long count = 0;
    while (need_loop) {
        count++;
    }

    printf("5sec by alarm : %lld\n", count);
    exit(0);
}
```

运行程序，查看结果如下：

```shell
$ time ./5sec_alarm  > /tmp/out

real    0m5.007s
user    0m4.987s
sys     0m0.016s
$ cat /tmp/out
5sec by alarm : 6456083116
```

```shell
$ time ./5sec_alarm  > /tmp/out

real    0m5.006s
user    0m4.974s
sys     0m0.030s
$ cat /tmp/out
5sec by alarm : 6191462278

$ time ./5sec_alarm  > /tmp/out
```

```shell
real    0m5.005s
user    0m4.977s
sys     0m0.026s
$ cat /tmp/out
5sec by alarm : 7329856790
```

对比time和alarm的实现，结论如下：

（1）alarm的定时精度要比time高

（2）alarm所能自增的次数大概是time版本的2到3倍

继续优化alarm版本，使用gcc编译并带上O1优化，发现程序死循环了！

```shell
$ gcc -O1 5sec_alarm.c
$ ./a.out
^C
```

为什么？因为gcc把`need_loop`值给优化掉了，编译器只加载了一次`need_loop`的值。导致其永远为真，程序陷入死循环。给`need_loop`加上`volatile`关键字，问题解决。

```c
static volatile int need_loop = 1;
```

运行加上O1优化后的程序，结果如下：

```shell
$ time ./a.out > /tmp/out

real    0m5.004s
user    0m4.975s
sys     0m0.025s
$ cat /tmp/out
5sec by alarm : 20131710513
```

```shell
$ time ./a.out > /tmp/out

real    0m5.012s
user    0m4.948s
sys     0m0.062s
$ cat /tmp/out
5sec by alarm : 20263928816
```

可以看到，加上O1优化后性能几乎是翻了10倍！

### 4. alarm应用实例之流量控制

[完整源码](https://github.com/wallace-lai/learn-apue/blob/main/src/con/parallel/signal/mycat.c)

我先实现一个没有流量控制的mycat应用，从mycpy基础上修改即可：

```c
    int sfd;
    int dfd = 1;

    do {
        sfd = open(argv[1], O_RDONLY);
        if (sfd < 0) {
            if (errno != EINTR) {
                perror("open()");
                exit(1);
            }
        }
    } while (sfd < 0);
```

解释：

（1）直接设置dfd为1，即标准输出

（2）打开要读取的文件

```c
    while (1) {
        len = read(sfd, buffer, BUFSIZE);
        if (len < 0) {
            if (errno == EINTR) {
                continue;
            }
            perror("read()");
            break;
        }
        if (len == 0) {
            break;
        }

        pos = 0;
        while (len > 0) {
            ret = write(dfd, buffer + pos, len);
            if (ret < 0) {
                if (errno == EINTR) {
                    continue;
                }
                perror("write()");
                exit(1);
            }

            pos += ret;
            len -= ret;
        }
    }
```

解释：

（1）循环从源文件中读取内容，往标准输出中打印

有了alarm之后，程序开始拥有“时间观念”，流控就变得可行了。

