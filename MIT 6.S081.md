# MIT 6.S081

## Pre

课程网址：https://pdos.csail.mit.edu/6.828/2021/schedule.html

课程教材：https://pdos.csail.mit.edu/6.828/2021/xv6/book-riscv-rev2.pdf

课程视频翻译：https://mit-public-courses-cn-translatio.gitbook.io/mit6-s081/

课程教材及实验翻译：http://xv6.dgs.zone/labs/requirements/lab1.html

参考代码：

Xiao Fan：https://fanxiao.tech/posts/2021-03-02-mit-6s081-notes/#11-processes-and-memory

ZachVec：https://github.com/ZachVec/MIT-6.S081

githubtoken: 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

## book-riscv-rev2

### 第一章 操作系统接口

### 第二章 操作系统架构

#### 2.2 用户态、核心态以及系统调用

RISC-V 有三种 CPU 可以执行指令的模式：机器模式（Machine Mode）、用户模式（User Mode）和管理模式（Supervisor Mode）。

#### 2.3 内核组织

宏内核（monolithic kernel）：整个操作系统都驻留在内核中，所有系统调用的实现都以管理模式运行；

微内核（microkernel）：为了降低内核出错的风险，操作系统设计者可以最大限度地减少在管理模式下运行的操作系统代码量，并在用户模式下执行大部分操作系统。

## Lab：Xv6 and Unix utilities

### Boot xv6

```shell
git clone git://g.csail.mit.edu/xv6-labs-2021
cd xv6-labs-2021
//切换到 util 分支
git checkout util
git commit -am 'my solution for util lab exercise 1'
make qemu //编译

退出 qemu
Ctrl-a x
```

在 github 建立远程仓库：http://xv6.dgs.zone/labs/use_git/git1.html

**练习评分需要退出 qemu 界面** 例如

```shell
./grade-lab-util sleep
```

### sleep

```cpp
#include "kernel/types.h"
#include "user/user.h"

int main(int argc, char* argv[]) {
    if(argc <= 1) {
        fprintf(2, "usage: sleep [time]");
        exit(1);
    }
    sleep(atoi(argv[1]));
    exit(0);
}
```

### pingpong

```cpp
#include "kernel/types.h"
#include "user/user.h"

#define READEND  0
#define WRITEEND 1

int main(int argc, char *argv[])
{
    if(argc != 1) {
        fprintf(2, "usage: pingpong\n");
        exit(1);
    }
    int p1[2];	//需要父子进程互传需要两个管道
    int p2[2];
    int pid;
    char buf[1];
    pipe(p1);
    pipe(p2);
    if((pid = fork()) < 0) {
        fprintf(2, "fork err!\n");
        exit(1);
    }else if(pid == 0) {
        close(p1[WRITEEND]);
        close(p2[READEND]);
        read(p1[READEND], buf, 1);
        printf("%d: received ping\n", getpid());
        write(p2[WRITEEND], "a", 1);
        close(p1[READEND]);
        close(p2[WRITEEND]);
        exit(0);
    }else {
        close(p1[READEND]);
        close(p2[WRITEEND]);
        write(p1[WRITEEND], "a", 1);
        read(p2[READEND], buf, 1);
        printf("%d: received pong\n", getpid());
        close(p1[WRITEEND]);
        close(p2[READEND]);
        exit(0);
    }
}
```

### primes

```cpp
#include "kernel/types.h"
#include "user/user.h"

#define READEND     0
#define WRITEEND    1

void child(int* pl);

int main(int argc, char *argv[])
{
    int p[2];
    pipe(p);
    if(fork() == 0) {
        child(p);
    } else {
        close(p[READEND]);
        for(int i = 2; i <= 35; ++i) {
            write(p[WRITEEND], &i, sizeof(int));
        }
        close(p[WRITEEND]);
        wait((int *) 0);	//父进程需要等待子进程退出
    }
    exit(0);
}

void child(int* pl) {
    int pr[2];
    int n;

    close(pl[WRITEEND]);
    if(read(pl[READEND], &n, sizeof(int)) == 0) {	//边界条件：不再收到数字
        exit(0);
    }
    pipe(pr);
    if(fork() == 0) {
        child(pr);
    }else {
        int prime = n;
        printf("prime %d\n", prime);
        close(pr[READEND]);
        while(read(pl[READEND], &n, sizeof(int)) != 0) {
            if(n % prime != 0) {
                write(pr[WRITEEND], &n, sizeof(int));
            }
        }
        close(pr[WRITEEND]);
        wait((int *) 0);
    }
    exit(0);
}
```

