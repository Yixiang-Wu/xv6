- 配置环境
    - 跟着下面的教程走
    - https://www.bilibili.com/video/BV11K4y127Qk?spm_id_from=333.1007.top_right_bar_window_history.content.click

- 添加用户编写的应用程序
    1. 将app.c放入xv6/user中
    2. 在xv6/Makefile中添加	$U/_app\到合适的位置
    3. 到xv6目录下，make qemu

- xv6操作
    - 退出(ctrl+A) + X
    - ctrl+D是结束符号

- 系统调用
1. fork
```c
int fork()
//创建一个子进程,和父进程在用户空间的内存(指令,数据和堆栈)一摸一样,除了PID
//代码段是共享的,数据段和堆栈段不共享的
//在父进程中返回子进程的PID,在子进程中返回0
//注意,这里的父进程和子进程共用一段代码,因此通过PID来进行判断是否是子进程
/*
阅读sh源码可知:
在父进程中,会执行wait(0),然后由主函数返回,exit(0)
在子进程中,会执行runcmd, 子进程在runcmd中返回,exit
*/
```
2. exit
```c
void exit(int status)
//使用status终止当前进程，status报告给父进程的wait,不返回.正常运行的话,status应该输入0,否则输入1
```
3. wait
```c
int wait(int * xstatus)
//如果执行wait的是父进程,那么等待一个子进程退出，将退出状态放入*xstatus，返回子进程的PID
//如果不关心子进程状态,那么可以使用wait(0)
//如果执行wait的是子进程,那么立即返回-1
```
4. exec
```c
int exec(char *file, char *argv[])
//执行某个可执行文件,不会创建子进程,而是使用那个文件所描述的指令,数据和堆栈来替换本进程中的指令数据和堆栈
//那个文件必须遵循某种格式以告知exec,哪里是指令,哪里是堆栈,指令从何开始等内容
//xv6使用ELF格式标准,在chapter3中介绍
//exec只有出错时才会有返回(进入那个文件,就有exit了,肯定不返回了)
```
5. read
```c
int read(int fd, char  buf[], int n)
//fd是文件描述符,从fd读到buf,最多n个字符.注意fd==0是标准输入.返回具体读了多少个字符.
//若fd==0,读到换行就结束
//只是读，不做添加（不在最后添加一个结束符）
```
6. write
```c
int write(int fd, char  buf[], int n)
//从buf写到fd,n个字符,返回n,注意fd==1是标准输出,fd==2是标准错误
```
7. close
```c
int close(int fd)
//释放文件描述符fd,成功返回0.注意open,pipe,dup等系统调用所使用的文件描述符为当前进程中编号最小的未使用描述符.
//每个进程有自己的一套文件描述符，通常情况下，shell会保证每个新进程，初始情况下都有3个打开的文件描述符（0，1，2）。
//fork之后，父进程和子进程的文件描述符是一样的。
//之所以将fork和exec分开，就是为了方便子进程的I/O重定向，参照shell中执行cat < input.txt的代码
//复制了文件描述符后，对于一个进程文件描述符的修改，不会影响到另外一个进程
//但二者，若文件描述符没修改，那么文件偏移量是同步的
```

8. dup
```c
int dup(int fd)
//返回一个新的文件描述符，指向与fd相同的文件
//二者的文件偏移量是共享的
```

9. pipe
```c
int pipe(int p[])
//创建一个管道，用于通信
//p[0]存放文件描述符，被进程用于read
//p[1]存放文件描述符，被进程用于write
//返回0，没啥用

//若read找不到可用data（read需要n个字符，而管道里只有k<n个)，那么read会一直等待，直到所有p[1]都被close。这也就是read端，需要提前close(p[1])的原因

//shell中可以如下语句：
//echo hello world | wc
//实质上创建了两个并行的子进程，前者使用了p[1]， 后者使用了p[0]

/*
管道比起临时文件的优点：
1. 管道使得两个进程可以并行地使用管道，临时文件必须要串行，先输入，再输出
2. 管道会自动清理自己，而临时文件就需要手动清理
3. 管道可以允许任意长地数据流，而临时文件要求要有足够大地空间
4. 管道是块读写，速度比较快
*/
```

10. 文件系统相关的系统调用

```c
open(char * file, int flags)  // flags在 kernel/fcntl.h
chdir(char *)
mkdir(char *)
int fstat(int fd, struct stat *) //inode ,stat的定义在 kerenl/stat.h
int stat(char * file, struct stat *)
int link(char * file1, char * file2) //为file1创建一个新名字
int unlink(char * file)
//只有删除所有的link，最后才能删除inode
int mknod //这个不太了解

//下面这种做法，会创建一个临时inode，进程退出或者关闭fd时删除inode
fd = open("haha.txt", O_CREATE | O_RDWR);
unlink("haha.txt");

//一个例外是cd命令，必须要在shell运行，其他的都创建子进程运行
```





