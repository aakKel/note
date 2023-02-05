1.Backtrace
![[Pasted image 20230205203147.png]]
![[Pasted image 20230205203200.png]]
![[Pasted image 20230205203209.png]]
解析：
利用已经给定的代码
```c
static inline uint64
r_fp()
{
  uint64 x;
  asm volatile("mv %0, s0" : "=r" (x) );
  return x;
}
```
调用`r_fp`函数
即：
```c
//**_kernel/printf.c_**
void  
backtrace(void) {  
    printf("backtrace:\n");  
    // 读取当前帧指针  
    uint64 fp = r_fp();  
    // 递归打印
    while (PGROUNDUP(fp) - PGROUNDDOWN(fp) == PGSIZE) {  
        // 返回地址保存在-8偏移的位置  
        uint64 ret_addr = *(uint64*)(fp - 8);  
        printf("%p\n", ret_addr);  
        // 前一个帧指针保存在-16偏移的位置  
        fp = *(uint64*)(fp - 16);  
    }  
}
```


2 Alarm
题目：
![[Pasted image 20230205231231.png]]

what should we do :
添加2个系统调用函数，分别是sigalarm和sigreturn
其中，sigalarm接收2个参数，第一个参数为 消耗的CPU滴答数，第二个为要调用的函数指针

滴答数取决于硬件的中断频率


解答：

```c
//MakeFile
$U/_alarmtest\
//Usys.pl
entry("sigalarm");  
entry("sigreturn");
//user.h
int sigalarm(int ticks, void (*handler)());  
int sigreturn(void);
//syscall.h
#define SYS_sigalarm 22  
#define SYS_sigreturn 23
//syscall.c
extern uint64 sys_sigalarm(void);  
extern uint64 sys_sigreturn(void);

[SYS_sigalarm] sys_sigalarm,  
[SYS_sigreturn] sys_sigreturn,

//proc.h
struct proc {
···
	int ticks;  
	int ticks_cnt;  
	uint64 handler;   //函数参数
	int is_handler;  // 是否已经有函数调用系统调用sigalarm，防止多个调用破环现场
	uint64 epc_tmp; //proc中的epc 寄存器的副本，这个寄存器存放的是下一条指令的地址对应pc
}


//sysproc.c
// 添加sys_sigalarm 实际代码
/*
	首先给定2个入参
	


*/
uint64  
sys_sigalarm(void)  
{  
    int ticks;  
    uint64 handler;  
    // 判断入参是否有错误  
    argint(0,&ticks);  
    argaddr(1,&handler);  
    struct proc *p = myproc();  
    // 参数保存至进程结构体中  
    // 初始化 进程结构体 参数
    p -> ticks = ticks;  
    p -> handler = handler;  
    p -> ticks_cnt = 0;  
  
    return 0;  
}

uint64  
sys_sigreturn(void)  
{  
    struct proc *p = myproc();  
    // 调用结束，可以再次进行sigalarm
    p->is_handler = 0;  
    p->trapframe->epc = p->epc_tmp;  
    // restore() 将trapfram 中的寄存器恢复 -> 恢复现场
    restore();  
    return 0;  
}

//trap.c
if(which_dev == 2) {  
    if (p->ticks > 0) {  
        p->ticks_cnt ++;  
        if (p->is_handler == 0 && p->ticks_cnt > p -> ticks) {  
            p->is_handler = 1;  
            p->ticks_cnt = 0;  
            p->epc_tmp = p->trapframe->epc;  
            store();  
            p->trapframe->epc = p->handler;  
        }  
    }  
    yield();  
  
}
```