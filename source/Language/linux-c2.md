# 【Linux C】并发操作

作者：wallace-lai <br>
发布：2024-05-29 <br>
更新：2024-06-16 <br>

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

### 4. exec函数族

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

