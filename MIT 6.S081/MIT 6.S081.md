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

### 第三章 页表

#### 3.1 页表硬件



### 第四章 陷阱指令和系统调用

有三种事件会导致中央处理器搁置普通指令的执行，强制将控制权转移到处理该事件的特殊代码上：

- 系统调用：用户程序执行 `ecall` 指令；
- 异常：用户或内核指令做了一些非法的事情；
- 设备中断。

xv6 陷阱处理分为四个阶段：

- RISC-V CPU 采取的硬件操作；
- 为内核 C 代码执行而准备的汇编程序集“向量”；
- 决定如何处理陷阱的 C 陷阱处理程序；
- 系统调用或设备驱动程序服务例程。

## Lec

### Lec03 OS Organization and System Calls

#### 3.2 操作系统隔离性 (isolation)

进程抽象了 CPU；exec 抽象了内存；files 抽象了磁盘；

#### 3.3 操作系统防御性 (Defensive)

实现强隔离性需要硬件支持，包括两部分：第一部分是 user/kernel mode；第二部分是 page table 或者虚拟内存(Virtual Memory)。

## Labs

### Lab：Xv6 and Unix utilities

#### Boot xv6

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

#### sleep

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

#### pingpong

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

#### primes

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

### Lab: system calls

#### System call tracing

- 如提示；

- 将系统调用相关声明或变量添加到相应文件；

- 在**kernel/sysproc.c**中添加一个`sys_trace()`函数：

  ```cpp
  uint64
  sys_trace(void)
  {
    int n;
    if(argint(0, &n) < 0) {	//从寄存器 a0 中读取值到 n 中
      return -1;
    }
    myproc() -> syscallnum = n;
    return 0;
  }
  ```

  将参数保存到`proc`结构体（请参见**kernel/proc.h**）里的一个新变量中来实现新的系统调用：在 `proc` 结构体中加入新成员变量 `syscallnum`。

- 修改`fork()`（请参阅**kernel/proc.c**）将跟踪掩码从父进程复制到子进程：

  ```cpp
  np -> syscallnum = p -> syscallnum;
  ```

- 修改**kernel/syscall.c**中的`syscall()`函数以打印跟踪输出

  ```cpp
  //记得加上声明和函数指针表
  extern uint64 sys_trace(void);
  static uint64 (*syscalls[])(void) = {
  ...
  [SYS_trace]   sys_trace,
  [SYS_sysinfo] sys_sysinfo
  };
  
  void
  syscall(void)
  {
    int num;
    struct proc *p = myproc();
    char* syscall_name[] = {"fork", "exit", "wait", "pipe",	//建立名称索引数组
                            "read", "kill", "exec", "fstat",
                            "chdir", "dup", "getpid", "sbrk",
                            "sleep", "uptime", "open", "write",
                            "mknod", "unlink", "link", "mkdir",
                            "close", "trace"};
  
    num = p->trapframe->a7;
    if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
      p->trapframe->a0 = syscalls[num]();		// a0 为系统调用返回值
      if((1 << num) & (p -> syscallnum)) {	// 判断需跟踪的系统调用
        printf("%d: syscall %s -> %d\n", p -> pid, syscall_name[num - 1], p -> trapframe -> a0);
      }
    } else {
      printf("%d %s: unknown sys call %d\n",
              p->pid, p->name, num);
      p->trapframe->a0 = -1;
    }
  }
  ```

#### Sysinfo

- 如提示；

- 参考上个实验的恢复编译过程；

- `sysinfo`需要将一个`struct sysinfo`复制回用户空间：

  ```cpp
  // kernel/sysproc.c
  uint64
  sys_sysinfo(void)
  {
    struct sysinfo si;
    uint64 sysinfop;
    if(argaddr(0, &sysinfop) < 0) {	//将寄存器 a0 中的值传至 sysinfop 中，sysinfop 指明了用户空间的 sysinfo 结构体的地址。
      return -1;
    }
    si.freemem = freememsize();
    si.nproc = nproccnt();	// si 位于内核空间
    if(copyout(myproc() -> pagetable, sysinfop, (char*) &si, sizeof(si)) < 0) {
      return -1;
    }
    return 0;
  }
  ```

- 要获取空闲内存量，请在**kernel/kalloc.c**中添加一个函数

  ```cpp
  uint64
  freememsize(void) 
  {
    struct run *begin;
    uint64 size = 0;
    int cnt = 0;
  
    acquire(&kmem.lock);		//获取锁
    begin = kmem.freelist;
    while(begin) {
      begin = begin -> next;
      cnt++;
    }
    release(&kmem.lock);
    size = cnt * PGSIZE;
    return size;
  }
  ```

- 要获取进程数，请在**kernel/proc.c**中添加一个函数

  ```cpp
  uint64
  nproccnt(void)
  {
    uint64 size  = 0;
    struct proc * p;
    for(p = proc; p < &proc[NPROC]; p++) {
      if(p -> state != UNUSED) {
        size++;
      }
    }
    return size;
  }
  ```

  
