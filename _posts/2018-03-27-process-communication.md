---
layout:     post
title:      "Linux 进程间通信"
subtitle:   "信号、管道、I/O 重定向"
category :  basictheory
date:       2018-03-27
author:     "Max"
header-img: "img/post-2018.jpg"
catalog:    true
tags:
    - process
    - linux
---

## 1. 信号

信号是各种进程间通信机制中唯一的异步通信机制。

信号有可靠和不可靠之分，学名分别为实时信号和非实时信号。不可靠信号如果进程不及时处理就会出现的情况；可靠信号通过引入信号队列来保证信号不丢，没有处理过的信号会保存在队列中。可靠信号的请求号全部介于 SIGRTMIN 和 SIGRTMAX 之间。

### 1.1 信号的发生

信号的发生主要由两个来源：一是硬件来源，如按下键盘特殊按键组合或某些硬件故障；二是软件来源，如常见的系统调用 kill()、raise()、alarm()、setitimer()、sigqueue() 等。

#### `kill()`

~~~c
#include <sys/types.h>
#include <signal.h>
int kill(pid_t pid, int sig);
~~~

可以向任意进程发送任意进程号。调用成功返回 0； 否则，返回 -1。

pid 参数就是目标进程的进程号，有以下四种取值：

1. n (n>0) ---->指定进程
2. 0 ---->当前进程所在进程组的全部进程
3. -1  ---->进程号比 1 大的所有进程（不会对 init 进程有任何影响）
4. -n (n>1) ---->进程组号为 n 的所有进程

sig 参数就是要发送的信号量。当为 0 时，实际不发送任何信号，但会进行错误检查。因此可用于检查目标进程是否存在，以及当前进程是否具有向目标发送信号的权限。

#### `raise()`

~~~c
#include <signal.h>
int raise(int sig);
~~~

向本进程发送信号量，完全等价于 `kill(getpid(), sig)`。

#### `alarm()`

```c
#include <unistd.h>
unsigned int alarm(unsigned int seconds);
```

在指定时间（秒）后向本进程发送 SIGALRM 信号，因此又称为闹钟信号。

返回值是上一次还没发出的 SIGALRM 信号与本次调用的剩余时间的间隔（秒），否则返回 0。若上次的信号还没发出，本次调用后将不会再发出。

#### `setitimer()`

```c
#include <sys/time.h> 
int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);
```

which 参数决定了计时器的类型：

1. ITEMER_REAL：在指定的绝对时间（秒）后向本进程发送 SIGALRM 信号
2. ITEMER_VIRTUAL：在指定的程序运行时间（秒）后向本进程发送 SIGALRM 信号
3. ITEMER_PROF：在指定的系统处理这个进程的所有时间（秒）后向本进程发送 SIGALRM 信号

new_value 设置时间间隔。第三个参数 old_value 没什么实际用途。

```c
struct timeval {
    long tv_sec;
    long tv_userc;
};

struct itimerval {
    struct timeval it_interval;
    struct timeval it_value;
};
```

这个结构体的怪异之处在于需要设定一个第一次发出 SIGALRM 信号的时间，再设定一个间隔时间。按照大多数人的经验，只需要设定一个间隔时间就满足需求了，所以大多数情况都会将这两个值设置为相同的值。

需要注意的是，alarm() 和 setitimer() 共用一个计时器，同时调用可能会相互干扰。

#### `sigqueue()`

```c
#include <sys/types.h> 
#include <signal.h>
int sigqueue(pid_t pid, int sig, const union sigval value);
```
sigqueue() 是比较新的发送信号系统调用，支持信号带有参数（第三个参数），可与函数 sigaction() 配合使用。

```c
union sigval {
    int sival_int;
    void *sival_ptr;
};
```

sigqueue() 比 kill() 传递了更多的附加信息，但 sigqueue() 只能向一个进程发送信号，而不能发送信号给一个进程组。

### 1.2 信号的处理

对于收到的任何信号，进程的默认处理方式大都是进程退出。使用 signal() 或 sigaction() 能够改变这种情况。

#### `signal()`

```c
#include <signal.h>
void (*signal(int sig, void (*func)(int)))(int);

typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
```

第一个参数指定信号的值；第二个参数指定信号的处理方式，可以忽略该信号（参数设为SIG_IGN），可以采用系统默认方式处理信号（参数设为SIG_DFL）；也可以自己实现处理方式（参数指定一个信号处理函数）。

如果 signal() 调用成功，返回实际信号处理函数的返回值；失败则返回 SIG_ERR，并设置 errno 表明错误原因。

#### `sigaction()`

```c
#include <signal.h>
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
```

sigaction() 系统调用可改变除 SIGKILL 和 SIGSTOP 外所有信号的处理行为，信号值由第一个参数 signum 指定。
若第二个参数 act 指针非空，新的处理行为在此指定；若第三个参数 oldact 指针非空，旧的处理行为会存储至此。

```c
struct sigaction {
    void     (*sa_handler)(int);
    void     (*sa_sigaction)(int, siginfo_t *, void *);
    sigset_t   sa_mask;
    int        sa_flags;
    void     (*sa_restorer)(void);
};
```

在某些架构的实现中，sa_handler 和 sa_sigaction 是一个 union，因此不能同时指定。若 sa_flags 带 SA_SIGINFO 标记，则 sa_sigaction 生效；否则 。sa_handler 生效。sa_handler 可用于指定自定义的信号处理函数，用法与 signal() 相同；sa_sigaction 除接受信号量作为第一个参数外，还可接受一个
siginfo_t 的指针和一个 ucontext_t 的指针（被转换为 void *）。

sa_mask 指定在当前 sa_handler 和 sa_sigaction 信号处理程序执行过程中，哪些信号应当被阻塞。默认情况下当前信号本身被阻塞，防止信号的嵌套发送，除非指定 SA_NODEFER 或者 SA_NOMASK 标志位。

sa_flags 指定一系列比特位标志影响信号处理行为。具体定义看参考 manpage。

sa_restorer 目前已淘汰，POSIX 不支持。

```c
siginfo_t {
    int      si_signo;    /* Signal number */
    int      si_errno;    /* An errno value */
    int      si_code;     /* Signal code */
    int      si_trapno;   /* Trap number that caused
                             hardware-generated signal
                             (unused on most architectures) */
    pid_t    si_pid;      /* Sending process ID */
    uid_t    si_uid;      /* Real user ID of sending process */
    int      si_status;   /* Exit value or signal */
    clock_t  si_utime;    /* User time consumed */
    clock_t  si_stime;    /* System time consumed */
    sigval_t si_value;    /* Signal value */
    int      si_int;      /* POSIX.1b signal */
    void    *si_ptr;      /* POSIX.1b signal */
    int      si_overrun;  /* Timer overrun count; POSIX.1b timers */
    int      si_timerid;  /* Timer ID; POSIX.1b timers */
    void    *si_addr;     /* Memory location which caused fault */
    int      si_band;     /* Band event */
    int      si_fd;       /* File descriptor */
}
```

需注意，除 si_signo、si_errno、si_code 对所有信号有效外，结构体的其它变量可能被定义为一个 union，只在特定的信号处理中有效。

sigaction() 调用成功返回 0； 否则，返回 -1，并设置 errno。

## 2. 管道

管道本质上是内存中一个固定大小的缓冲区（32位下 4K，64位下 1M），除了大量应用在 shell 命令拼接上外，也可用于具有亲缘关系的进程间通信。

如有如下特征：

* 管道是半双工的，数据只能向一个方向流动；需要双方通信时，需要建立起两个管道。
* 只能用于父子进程或者兄弟进程之间（具有亲缘关系的进程）通信。
* 单独构成一种独立的文件系统。
    
    管道对于管道两端的进程而言，就是一个文件，但它不是普通的文件，它不属于某种文件系统，而是自立门户，单独构成一种文件系统，并且只存在与内存中。

* 一个进程向管道中写的内容被管道另一端的进程读出，写入的内容每次都添加在管道缓冲区的末尾，并且每次都是从缓冲区的头部读出数据。

### 2.1 管道的创建

```c
#include <unistd.h>
int pipe(int pipefd[2]);
```

pipe() 调用成功后会返回两个文件描述符，pipefd[0]（可称为读端）保存用于读数据的文件描述符，pipefd[1]（可称为写端）则保存用于写数据的文件描述符。
既然管道的本质就是文件，那么一般的文件 I/O 函数都可以使用，如close()、read()、write() 等。

### 2.2 管道的使用

pipe() 函数创建的管道的两端处于一个进程中，在实际应用中没有太大意义，因此，一个进程在由 pipe() 创建管道后，一般再 fork 一个子进程，然后通过管道
实现父子进程间的通信。

简单示例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <signal.h>
#include <sys/wait.h>

void child(int fd)
{
    int i = 0;
    for (;;)
    {
        ++i;
        write(fd, &i, sizeof(i));
        sleep(1);
    }
}

int main(int argc, char* argv[])
{
    pid_t pid;
    int pfd[2], i;
    long data;

    pipe(pfd);

    pid = fork();
    if (0 == pid)
    {
        close(pfd[0]);
        child(pfd[1]);
        exit(0);
    }

    close(pfd[1]);
    for (i=0; i < 10; ++i)
    {
        read(pfd[0], &data, sizeof(data));
        printf("data: %ld\n", data);
    }
    kill (pid, 9);
    waitpid(pid, NULL, 0);
    return 0;
}
```

## 3. I/O 重定向

在命令行中使用管道其实还隐含了 I/O 重定向的应用。这样的机制也可以通过程序来完成。

```c
#include <unistd.h>
int dup(int oldfd);
int dup2(int oldfd, int newfd);
int dup3(int oldfd, int newfd, int flags);
```

这三个函数若调用成功，都可以返回一个已有文件描述符 oldfd 的拷贝。作为拷贝，两个文件描述符指向同一个打开的文件，并共享文件偏移量和文件状态标志。
通过一个文件描述符操作文件后，另一个描述符指向的文件会有相同的改动。但是两个描述符不共享文件描述符标志（close-on-exec 标志）。函数调用失败返回
-1 并设置 errno。

dup() 会直接返回一个值最小的可用文件描述符。

dup2() 会先关闭文件描述符 newfd 然后将其做成 oldfd 的拷贝。若 oldfd 本身是无效的文件描述符，则调用失败且 newfd 不会被关闭；若 oldfd 有效但是
newfd 与其相同，则 dup2() 不做任何改变并返回 newfd。

dup3() 与 dup2() 功能类似。但是当 oldfd 与 newfd 有效且相同的时候，dup3() 会以 EINVAL 原因失败。并且，dup3() 可以通过在 flags 参数中指定
 O_CLOEXEC 强制使两个文件描述符共享文件描述符标志。指定该标志可避免重复使用 fcntl() F_SETFD 操作在多线程/进程环境下设置 FD_CLOEXEC 标志以
规避数据竞争。

简单示例：
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <signal.h>
#include <sys/wait.h>

#define BUF_SIZE 1024

void child(int fd, char *envp[])
{
    char *newargv[] = {"ls", "-l", NULL};
    dup2(fd, STDOUT_FILENO);
    execve("/bin/ls", newargv, envp);
}

int main(int argc, char* argv[], char *envp[])
{
    pid_t pid;
    int pfd[2], i;
    char *buf[BUF_SIZE+1];

    pipe(pfd);
    pid = fork();
    if (0 == pid)
    {
        close(pfd[0]);
        child(pfd[1], envp);
        exit(0);
    }

    close(pfd[1]);
    for (;;)
    {
        memset(buf, 0, BUF_SIZE+1);
        if (0 == read(pfd[0], buf, BUF_SIZE))
        {
            break;
        }
        printf("%s", buf);
    }
    waitpid(pid, NULL, 0);
    return 0;
}
```
