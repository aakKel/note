![[Pasted image 20221207171347.png]]

```c
#include "kernel/types.h"  
#include "kernel/stat.h"  
#include "user/user.h"  
  
int  
main(int argc, char *argv[])  
{  
    if(argc != 2)  
        printf("error!");  
    sleep(atoi(argv[1]));  
    exit(0);  
  
}
```
- 在Makefile 中添加 `$U/_sleep\`
- usys.pl  `entry("sleep");`


![[Pasted image 20221207171622.png]]
```c
#include "kernel/types.h"  
#include "kernel/stat.h"  
#include "user/user.h"  
  
#define RD 0  
#define WR 1  
int  
main(int argc, char *argv[])  
{  
    int p1[2];  
    int p2[2];  
    // 打开2个管道
    pipe(p1);  
    pipe(p2);  
    int pid = fork();  
    char buf = 'a';  
    // 创建子进程失败，4个口全部关闭
    if(pid < 0)  
    {  
        fprintf(2,"fork error.\n");  
        close(p1[RD]);  
        close(p2[RD]);  
        close(p1[WR]);  
        close(p2[WR]);  
        exit(1);  
    }  
    //子进程
    //       xWR----P1----RD
    //       WR----P2----RDx
    if(pid == 0)  
    {  
        close(p1[WR]);  
        close(p2[RD]);  
        read(p1[RD],&buf,sizeof(char));  
        fprintf(1, "%d: received ping\n", getpid());  
        write(p2[WR],&buf,sizeof(char));  
        exit(0);  
        close(p1[RD]);  
        close(p2[WR]);  
    }  
    //       WR----P1----RDx
    //       xWR----P2----RD
    else  
    {  
        close(p1[RD]);  
        close(p2[WR]);  
        write(p1[WR],&buf,sizeof(char));  
        read(p2[RD],&buf,sizeof(char));  
        fprintf(1, "%d: received pong\n", getpid());  
        close(p1[WR]);  
        close(p2[RD]);  
        exit(0);  
    }  
}
```

![[Pasted image 20221207172035.png]]

```c
#include "kernel/types.h"  
#include "user/user.h"  
  
#define RD 0  
#define WR 1  
#define MAXNUM 35  //由于 xv6 的文件描述符和进程数量有限，第一个进程可以在 35 处停止。
  
void next(int *);  
  
int main(int argc,char *argv[])  
{  
    int p[2];  
    pipe(p);  
  
  //子进程
    if(fork() == 0)  
    {  
        next(p);  
    }  
    else  
    {  
        close(p[RD]);  
        // 把 2 - 35 写入管道中
        for(int i = 2;i <= MAXNUM; ++i)  
        {  
            write(p[WR],&i,sizeof(int));  
        }  
        close(p[WR]);  
        // 等待子进程关闭
        wait((int *) 0);  
  
    }  
    exit(0);  
}  
  
void next(int *p)  
{  
    int np[2];  
    int n ;  
    close(p[WR]);  
    // 若为第一次调用， 读入的就是2 -> n = 2;
    int read_status = read(p[RD],&n,sizeof(int));  
  // 如果都处理完了 ，退出
    if(read_status == 0)  
        exit(0);  
  
    pipe(np);  
    if(fork() == 0)  
        next(np);  
    else  
    {  
        close(np[RD]);  
        int prime = n;  
        printf("prime %d\n", n);  
        // 若为第一次调用， 把 3 - 35 每个数都与 n = 2 取余 ， 不为 0 的放入管道中.
        // p 为原始管道 ,若为第一次调用 则是 3 - 35 ，然后把 3 ，5， 7 ··  
        //放入np中，成为下一轮的p。
        while(read(p[RD],&n,sizeof(int)) != 0)  
            if(n % prime != 0)  
                write(np[WR],&n,sizeof(int));  
        close(np[WR]);  
  
        wait((int *) 0);  
  
        exit(0);  
  
    }  
  
}
```

![[Pasted image 20221207173236.png]]

```c
#include "kernel/types.h"  
#include "user/user.h"  
#include "kernel/fs.h"  
#include "kernel/stat.h"  
#include "kernel/fcntl.h"  
  
void find(char *,char *);  
int  
main(int argc, char *argv[])  
{  
// find 调用需要3个参数，不为3个参数报错
    if(argc != 3)  
    {  
        fprintf(2,"ERROR\n");  
        exit(1);  
    }  
    // 要寻找的文件路径
    char *target_path = argv[1];  
    // 要寻找的文件
    char *target_file = argv[2];  
    find(target_path,target_file);  
    exit(0);  
}  
  //递归寻找。参考ls代码
void find(char *target_path,char *target_file)  
{  
    char buf[512], *p;  
    int fd;  
	// 文件结构体
	//结构体 stat 和 dirent 分别存储文件的信息和目录的信息
    struct dirent de;  //目录的信息
    struct stat st;  //文件的信息
  
    if((fd = open(target_path, 0)) < 0)  
    {  
        fprintf(2, "cannot open %s\n", target_path);  
        return;  
    }  
  
    if(fstat(fd, &st) < 0)  
    {  
        fprintf(2, "cannot stat %s\n", target_path);  
        close(fd);  
        return;  
    }  
  
    while(read(fd,&de,sizeof(de)) == sizeof(de))  
    {  
        strcpy(buf, target_path);  
        p = buf + strlen(buf);  
        *p++ = '/';  
        if (de.inum == 0) continue;  
  
        memmove(p, de.name, DIRSIZ);  
        p[DIRSIZ] = 0;  
  
        if (stat(buf, &st) < 0)  
            fprintf(2, "ERROR: cannot stat %s\n", buf);  
  
        switch (st.type) {  
            case T_FILE:  
                if (strcmp(target_file, de.name) == 0)  
                    printf("%s\n", buf);  
                break;  
            case T_DIR:  
                if ((strcmp(de.name, ".") != 0) && (strcmp(de.name, "..") != 0))  
                    find(buf, target_file);  
        }  
    }  
    close(fd);  
    return ;  
  
  
}
```
![[Pasted image 20221207183252.png]]
```c
#include "kernel/types.h"  
#include "user/user.h"  
#include "kernel/param.h"  
#define MAX_LEN 100  
  
int main(int argc, char *argv[])  
{  
    char *command = argv[1];  
    char param[MAXARG][MAX_LEN];  
    char *m[MAXARG];  
    char bf ;  
  
    while(1)  
    {  
        int cnt = argc - 1;  
        memset(param,0,MAXARG * MAX_LEN);  
        for(int i = 1;i < argc; ++i)  
        {  
            strcpy(param[i - 1],argv[i]);  
        }  
  
        int cursor = 0;  
        int flag = 0;  
        int read_res ;  
        //from pipe or .. read data to bf and write in param one bite to ..  
        while((read_res = read(0,&bf,1)) > 0 && bf != '\n')  
        {  
            if(bf == ' ' && flag == 1)  
            {  
                flag = 0;  
                cnt ++;  
                cursor = 0;  
            }  
            else if(bf != ' ')  
            {  
                param[cnt][cursor++] = bf;  
                flag = 1;  
            }  
        }  
        if(read_res <= 0) break;  
  
        for(int i = 0;i < MAXARG - 1; ++i)  
            m[i] = param[i];  
  
        m[MAXARG - 1] = 0;  
  
        if(fork() == 0)  
        {  
            exec(command,m);  
            exit(0);  
        }  
        else wait((int*)0);  
  
    }  
    exit(0);  
  
}
```