系统调用过程
user/user.h: 用户态程序调用跳板函数 trace() 
user/usys.S: 跳板函数 trace() 使用 CPU 提供的 ecall 指令，调用到内核态 kernel/syscall.c 到达内核态统一系统调用处理函数 syscall()，所有系统调用都会跳到这里来处理。 
kernel/syscall.c syscall() 根据跳板传进来的系统调用编号，查询 syscalls[] 表，找到对应的内核函数并调用。 
kernel/sysproc.c 到达 sys_trace() 函数，执行具体内核操作

用户态使用跳板函数中，cpu提供的ecall指令，调用到内核态，内核态的syscall() 根据跳板函数的系统调用编号，找到对应的函数指针，最后根据函数指针执行相应操作。



https://cloud.tencent.com/developer/article/2142322
https://www.cnblogs.com/SVicen/p/16756223.html