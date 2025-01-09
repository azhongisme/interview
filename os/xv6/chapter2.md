# 三个需求
隔离性、多路复用、交互

# 抽象物理资源
周期型放弃CPU，保证所有都能运行

Unix交互通过文件系统open,read,write,close代替直接读写磁盘

Unix在进程之间透明(对用户和应用程序不可见)切换硬件CPU，根据需要保存何恢复寄存器状态

使用exec建立内存镜像而不是直接与物理内存交互，如果内存紧张操作系统会在磁盘中存储一部分进程的数据，exec还为用户提供了文件系统来存储可执行程序映像的便利

进程间交互的许多形式通过文件描述符，抽象细节(数据在管道还是在文件中), 简化交互(如果管道中的一个应用程序发生故障，内核将为管道中的下一个进程生成文件结束的信号)

# 用户模式 监督者模式 系统调用
为了保证隔离，操作系统管理应用程序不能修改(甚至读取)操作系统的数据结构和指令，应用程序无法访问其他进程的内存

CPU提供硬件支持强隔离，RISC-V有三种模式用于CPU执行指令：
- machine mode
  - 所有权限
  - 用于配置计算机
  - 在machine模式下执行几行之后更改为supervisor mode
- supervisor mode
  - 允许执行特权指令（启用和禁用中断，读取和写入保存页表地址的寄存器）
  - 如果应用程序在user mode下尝试执行特权指令，CPU不会执行指令
  - 运行在内核空间
- user mode
  - 只能执行user mode的指令，运行在用户空间

  想要调用内核函数(read系统调用)的应用程序必须转换到内核

  切换到supervisor mode内核检验系统调用的参数(如检查传递给系统调用的地址是否是应用程序内存的一部分)，决定应用程序是否允许执行请求的操作

# 内核组织

## 整体内核
整个操作系统进入内核，在supervisor mode实现所有的系统调用

整个操作系统有所有硬件的权限

## 微内核
作为进程运行的操作系统服务称为服务器

内核提供一个进程间通信机制来发送消息,从一个user mode的进程到宁一个进程

大多数操作系统驻留在用户级服务器

# 进程
隔离通过进程

进程抽象可以防止一个进程破坏或监视宁一个进程的内存,CPU,文件描述符

防止进程破坏内核本身

# starting xv6, the first process and system call
开机首先初始化,运行一个引导加载程序(存储在只读内存中),加载程序将xv6内核上传到内存

然后machine mode下CPU执行_entry
```
_entry:
        # set up a stack for C.
        # stack0 is declared in start.c,
        # with a 4096-byte stack per CPU.
        # sp = stack0 + (hartid * 4096)
        la sp, stack0
        li a0, 1024*4
        csrr a1, mhartid
        addi a1, a1, 1
        mul a0, a0, a1
        add sp, sp, a0
        # jump to start() in start.c
        call start
```
start执行一些仅在机器模式下允许的配置,之后切换到supervisor mode

` mret`切换到supervisor mode

程序计数器切换到main,初始化一些设备和子系统后通过调用`userinit`创建第一个进程


