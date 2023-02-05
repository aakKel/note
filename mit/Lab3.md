课件参考资料 trans
https://blog.csdn.net/InnerPeaceHQ/article/details/125330276

1，![[Pasted image 20221224161216.png]]
```c
//proc.h
struct proc {
	···
	//usyscall 声明  
	struct usyscall *usyscall;
}
//proc.c
//共享内存页的初始化，以及对共享内存块的页表初始化。
pagetable_t  
proc_pagetable(struct proc *p)  
{
	// 只读 PTE_R, 同时也要标记该页已经用过 PTE_U  
    if (mappages(pagetable, USYSCALL, PGSIZE,  
                 (uint64)(p->usyscall),PTE_R | PTE_U) < 0) {  
        uvmunmap(pagetable, TRAMPOLINE, 1, 0);  
        uvmunmap(pagetable, TRAPFRAME, 1, 0);  
        uvmfree(pagetable, 0);  
        return 0;  
    }
}
//增加释放共享内存块
static void  
freeproc(struct proc *p)  
{
	if (p->usyscall)  
    kfree((void*)p->usyscall);  
	p->usyscall = 0;
}
//增加申请共享内存页。
static struct proc*  
allocproc(void)  
{
	//给usyscall 分配页表  
  if((p->usyscall = (struct usyscall *)kalloc()) == 0){  
      freeproc(p);  
      release(&p->lock);  
      return 0;  
  }  
  p->usyscall->pid = p->pid;
}
	
```
2.Print a page table
![[Pasted image 20221224164437.png]]

```c
//首先应当在defs.h 中定义打印的vmprint() 函数
void            vmprint(pagetable_t );
// 其次，在exec.c 中增加一行
// 输出第一个进程的内存地址空间  
if(p->pid==1) vmprint(p->pagetable);
//vm.c
// 递归打印 从顶级页表打印至0级页表  
// 实现的时候用的是从 0 - 2  
void vmprintlevel(pagetable_t pt, int level) {  
	//这边512 的原因：一个页表占4096，一个PTE占64byte ,则一个页表有512个页表项
    for (int i = 0; i < 512; i++) {  
        pte_t pte = pt[i];  
        //如果页表项存在  
        if ((pte & PTE_V)) {  
            printf("%s","..");  
            for (int j = 0;j < level; ++j) {  
                printf("%s"," ..");  
            }  
            printf("%d: pte %p pa %p\n", i, pte, PTE2PA(pte));  
            uint64 child = PTE2PA(pte);  
            // 这个条件判断 应该使用 判断页表项指针指向的页表是存在的  
            // if ((pte & (PTE_R | PTE_W | PTE_X)) == 0)  
            // 用level < 2 也一样，总共只有三级页表，一开始输入的是0级->1级->2级
            // 与freewalk 一致  
            if (level < 2) {  
                vmprintlevel((pagetable_t)child, level + 1);  
            }  
        }  
    }  
}  
//打印  
void vmprint(pagetable_t pt) {  
    printf("page table %p\n", pt);  
    vmprintlevel(pt, 0);  
}
```
3.![[Pasted image 20221224170111.png]]
```c
//defs.h
int             pgaccess(pagetable_t ,uint64 , int , uint64 );  
//为了能在sysproc.c 中使用walk  
pte_t*          walk(pagetable_t, uint64, int);
//ricv.h
// 查表，第六位为PTE_A  
#define PTE_A (1L << 6) // 1 -> accessed
//kernel/sysproc.c
// 辅助函数  
int pgaccess(pagetable_t pagetable,uint64 start_va, int page_num, uint64 result_va)  
{  
    // 使用的是u64，一种每页使用一位的数据结构，其中第一页对应于最低有效位  
    // 所以最多查询64页  
    if (page_num > 64)  
    {  
        panic("pgaccess: too much pages");  
        return -1;  
    }  
    // 存储是否被访问  
    unsigned int bitmask = 0;  
    int cur_bitmask = 1;  
    int count = 0;  
    uint64 va = start_va;  
    pte_t *pte;  
    for (; count < page_num; count++, va += PGSIZE)  
    {  
        //查询不到页表项，报错  
        if ((pte = walk(pagetable, va, 0)) == 0)  
            panic("pgaccess: pte should exist");  
        if ((*pte & PTE_A))  
        {  
            // 迭代查询，若当前页被访问，使用一个cnt来记录当前第几页  
            bitmask |= (cur_bitmask<<count);  
            //清除PTE_A  
            *pte &= ~PTE_A;  
        }  
    }  
    copyout(pagetable,result_va,(char*)&bitmask,sizeof(bitmask));  
    return 0;  
}
```