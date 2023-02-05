![[Pasted image 20221207201144.png]]
```c
//sysproc.c
uint64  
sys_trace(void)  
{  
    int n;  
    if(argint(0,&n) < 0)  
        return -1;  
    myproc() -> syscallnum = n;  
    return 0;  
}
//syscall.h
#define SYS_trace  22
//proc.h
···
struct proc{
int syscallnum;
}
//proc.c
int fork(void){
//跟踪掩码从父进程复制到子进程。
	np -> syscallnum = p -> syscallnum;
}
//kernel/syscall.c
void  
syscall(void)  
{  
  int num;  
  struct proc *p = myproc();  
  char* syscall_name[22] = {"fork", "exit", "wait", "pipe", "read", "kill", "exec", "fstat", "chdir", "dup", "getpid", "sbrk", "sleep", "uptime", "open", "write", "mknod", "unlink", "link", "mkdir", "close", "trace"};  
//使用a7寄存器进行传递调用  
// 传入的是系统调用的编号 如syscall.h 中的编号
  num = p->trapframe->a7;  
  /*
	  #define NELEM(x) (sizeof(x)/sizeof((x)[0]))
	  以下的syscalls 变量是 一个 保存函数指针的数组 : 
	  static uint64 (*syscalls[])(void)
	  当前的trace 就以[SYS_trace]   sys_trace, 保存其中
   */
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {  
    p->trapframe->a0 = syscalls[num]();  
    if((1 << num) & (p ->syscallnum))  
        printf("%d: syscall %s -> %d\n", p->pid, syscall_name[num-1], p->trapframe->a0);  
  } else {  
    printf("%d %s: unknown sys call %d\n",  
            p->pid, p->name, num);  
    p->trapframe->a0 = -1;  
  }  
}
```