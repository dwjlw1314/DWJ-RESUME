GDB调试的三种方式：
* 目标板直接使用GDB进行调试
* 目标板使用gdbserver，主机使用xxx-linux-gdb作为客户端
* 目标板使用ulimit -c unlimited，生成core文件；然后主机使用xxx-linux-gdb ./test ./core

Brendan Gregg关于gdb介绍 http://www.brendangregg.com/blog/2016-08-09/gdb-example-ncurses.html

```c++
main.c:
#include <stdio.h>
#include <stdlib.h>

extern int sum(int value);

struct inout {
    int value;
    int result;
};

int main(int argc, char * argv[])
{
    struct inout * io = (struct inout * ) malloc(sizeof(struct inout));
    if (NULL == io) {
        printf("Malloc failed.\n");
        return -1;
    }

    if (argc != 2) {
        printf("Wrong para!\n");
        return -1;
    }

    io -> value = *argv[1] - '0';
    io -> result = sum(io -> value);
    printf("Your enter: %d, result:%d\n", io -> value, io -> result);
    return 0;
}
sum.c:
int sum(int value) {
    int result = 0;
    int i = 0;
    for (i = 0; i < value; i++)
        result += (i + 1);
    return result;
}
```
然后gcc main.c sum.c -o main -g, 得到main可执行文件, -g 参数是调试开关

<font color=#FF0000 size=3>设置断点</font>

设置断点可以通过b或者break设置断点，断点的设置可以通过函数名、行号、文件名+函数名、文件名+行号以及偏移量、地址等进行设置

格式为：
```
break 函数名
break 行号
break 文件名:函数名
break 文件名:行号
break +偏移量
break -偏移量
break *地址
```
通过info proc查看进程信息
```
(gdb) info proc
process 25193
cmdline = 'top'
cwd = '/root'
exe = '/usr/bin/top'
```
通过info sharedlibrary查看动态库加载情况
```
(gdb) info sharedlibrary
From                To                  Syms Read   Shared Object Library
0x00007ffff7ddbb10  0x00007ffff7df6710  Yes (*)     /lib64/ld-linux-x86-64.so.2
0x00007ffff7a37480  0x00007ffff7b7dbcf  Yes (*)     /lib64/libc.so.6
(*): Shared library is missing debugging information.
```
查看断点，通过info break查看断点列表
```
(gdb) break main.c:6
Breakpoint 1 at 0x4005cc: file main.c, line 6.
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000004005cc in main at main.c:6
```
删除断点命令:
```
delete <断点id> 删除指定断点
delete 删除所有断点
clear 函数名
clear 行号
clear 文件名:行号
clear 文件名:函数名
```
断点可以条件断住：
```
break 断点 if 条件: 比如break sum if value==9，当输入的value为9的时候才会断住
condition 断点编号：给指定断点删除触发条件
condition 断点编号 条件：给指定断点添加触发条件

(gdb) break sum if value==9
Breakpoint 1 at 0x400663: file sum.c, line 2.
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400663 in sum at sum.c:2
        stop only if value==9
(gdb) condition 1
Breakpoint 1 now unconditional.
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000400663 in sum at sum.c:2
```
断点还可以通过disable/enable临时停用启用
```
disable 断点编号
disable display 显示编号
disable mem 内存区域

enable 断点编号
enable once 断点编号：该断点只启用一次，程序运行到该断点并暂停后，该断点即被禁用
enable delete 断点编号
enable display 显示编号
enable mem 内存区域
```

```c++
#include <stdio.h>

int total = 0;

int square(int i)
{
    int result=0;
    result = i*i;
    return result;
}

int main(int argc, char **argv)
{
    int i;
    for(i=0; i<10; i++)
    {
        total += square(i);
    }
    return 0;
}
```
对如上程序square函数参数i为5的时候断点，并在此时打印栈、局部变量以及total的值；编写gdb.init如下：
```
set logging on gdb.log

b square if i == 5
commands
  bt full
  i locals
  p total
  print "Hit break when i == 5"
end
```
在gdb shell中执行 source gdb.init，然后r执行命令，结果如下：
```
(gdb) bt full
#0  square (i=5) at main.c:7
        result = 16
#1  0x000000000040056c in main (argc=1, argv=0x7fffffffe568) at main.c:20
        i = 5
(gdb) i local
i = 5
(gdb) p total
$3 = 30
(gdb)  print "Hit break when i == 5"
$4 = "Hit break when i == 5"
```
在unix/linux系统下使用gdb进行调试时，如果出现：No symbol table is loaded. Use the "file" command.
```
原因是没有在Makefile中添加-g调试参数，或者添加位置出错，解决的办法是在Makefile文件的第一行加上：
CFLAGS = -g
```
run可以在gdb下运行命令；如果命令需要参数则跟在run之后.如果需要断点在main()处，直接执行start就可以
```
(gdb) start
Temporary breakpoint 1 at 0x400559: file main.c, line 18.
Starting program: /root/main

Temporary breakpoint 1, main (argc=1, argv=0x7fffffffe568) at main.c:18
18          for(i=0; i<10; i++)
```
如果遇到断点而暂停执行，或者coredump可以通过bt显示栈帧
```
bt full：不仅显示backtrace，还显示局部变量
bt N：显示开头N个栈帧
```
print可以显示变量内容
```
如果查看指定变量，可以通过 p total
如果需要一行监控多个变量，可以通过p {var1, var2, var3}
如果要跟踪自动显示，可以使用display {var1, var2, var3}
```
info reg可以显示寄存器内容
```
(gdb) i r
rax            0x5      5
rbx            0x0      0
rcx            0x4005b0 4195760
rdx            0xe      14
rsi            0x7fffffffe568   140737488348520
rdi            0x5      5
rbp            0x7fffffffe450   0x7fffffffe450
rsp            0x7fffffffe450   0x7fffffffe450
r8             0x7ffff7dd5e80   140737351868032
r9             0x0      0
r10            0x7fffffffe260   140737488347744
r11            0x7ffff7a39b10   140737348082448
r12            0x400440 4195392
r13            0x7fffffffe560   140737488348512
r14            0x0      0
r15            0x0      0
rip            0x40053b 0x40053b <square+14>
eflags         0x297    [ CF PF AF SF IF ]
cs             0x33     51
ss             0x2b     43
ds             0x0      0
es             0x0      0
fs             0x0      0
gs             0x0      0
```
在寄存器名之前加$可以显示寄存器内容
```
p $寄存器：显示寄存器内容
p/x $寄存器：十六进制显示寄存器内容
```
用x命令可以显示内容内容
```
x $pc：显示程序指针内容
x/i $pc：显示程序指针汇编
x/10i $pc：显示程序指针之后10条指令
x/128wx 0xfc207000：从0xfc20700开始以16进制打印128个word
```
可以通过disassemble指令来反汇编
```
disassemble 程序计数器 ：反汇编pc所在函数的整个函数
disassemble addr-0x40,addr+0x40：反汇编addr前后0x40大小
```
单步执行有两个命令next和step，两者的区别是next遇到函数不会进入函数内部，step会执行到函数内部;如果需要逐条汇编指令执行，可以分别使用nexti和stepi
调试时，使用continue命令继续执行程序。程序遇到断电后再次暂停执行；如果没有断点，就会一直执行到结束
```
continue：继续执行
continue 次数：继执行一定次数
```
continue、step、stepi、next、nexti都可以指定重复执行的次数

ignore 断点编号 次数：可以忽略指定次数断点

要想找到变量在何处被改变，可以使用watch命令设置监视点watchpoint
```
watch <表达式>：表达式发生变化时暂停运行
awatch <表达式>：表达式被访问、改变时暂停执行
rwatch <表达式>：表达式被访问时暂停执行
其他变种还包括watch expr [thread thread-id] [mask maskvalue]，其中mask需要架构支持
```
GDB不能监控一个常量，比如watch 0x600850报错；但是可以watch *(int *)0x600850

通过 set variable <变量>=<表达式> 来修改变量的值
>set $r0=xxx：设置r0寄存器的值为xxx

通过generate-core-file生成core.xxxx转储文件
>(gdb) generate-core-file  //提示 Saved corefile core.24743

另一命令gcore可以从命令行直接生成内核转储文件,无需停止正在执行的程序已获得转储文件
>[root@localhost ~]# gcore 'pidof program_name'

如果程序已经运行，或者是调试陷入死循环而无法返回控制台进程，可以使用attach命令
>[root@localhost ~]# gdb attach 25193

Linux环境下初始化文件为.gdbinit，如果存在.gdbinit文件，gdb在启动的之前就将其作为命令文件运行

初始化文件和命令文件执行顺序为：HOME/.gdbinit > 运行命令行选项 > ./.gdbinit > -x指定命令文件

默认情况下core文件存在调试程序当前路径下，为了区分可以进行设置
```
区分core主要通过/proc/sys/kernel/core_uses_pid和/proc/sys/kernel/core_pattern进行设置
/proc/sys/kernel/core_uses_pid：可以控制产生的core文件的文件名中是否添加pid作为扩展，如果添加则文件内容为1，否则为0
proc/sys/kernel/core_pattern：可以设置格式化的core文件保存位置或文件名，比如文件内容是/tmp/core-%e-%p

以下是部分参数列表:
%p - insert pid into filename 添加pid
%u - insert current uid into filename 添加当前uid
%g - insert current gid into filename 添加当前gid
%s - insert signal that caused the coredump into the filename 添加导致产生core的信号
%t - insert UNIX time that the coredump occurred into filename 添加core文件生成时的unix时间
%h - insert hostname where the coredump happened into filename 添加主机名
%e - insert coredumping executable name into filename 添加命令名
```
当然，你可以用下列方式来完成：
>[root@localhost ~]# sysctl -w kernel.core_pattern=/tmp/core-%e-%p

在已运行的Linux上，如果发生死机异常等问题，这时候定位问题需要使用jtag连接上,连接方法是：
```
gdb-----------------------------------------------进入gdb shell
target remote localhost:1025----------------------在gdb shell中通过ip:port连接上target
file vmlinux--------------------------------------加载符号表
```
