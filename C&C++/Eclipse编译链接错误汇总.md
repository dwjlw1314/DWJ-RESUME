1.ERROR> amqp_time.c:(.text+0x4d): undefined reference to `clock_gettime'
```c
问题描述：链接的时候查找发现clock_gettime在实时库(real time)里面,由于没有链接这个库导致报错
解决方法：只需在编译的时候加上-lrt即可，eg:执行命令 gcc test.c -lrt 此时没有报任何的错误
附:linux常用的库
libz   压缩库(Z)
librt  实时库(real time)
libm   数学库(math)
libc   标准C库(C lib)
函数"clock_gettime"是基于Linux C语言的时间函数,可以用于计算时间，有秒和纳秒两种精度
函数原型：int clock_gettime(clockid_t clk_id, struct timespec *tp);
其中，cld_id类型四种：
a、CLOCK_REALTIME:系统实时时间,随系统实时时间改变而改变
b、CLOCK_MONOTONIC,从系统启动这一刻起开始计时,不受系统时间被用户改变的影响
c、CLOCK_PROCESS_CPUTIME_ID,本进程到当前代码系统CPU花费的时间
d、CLOCK_THREAD_CPUTIME_ID,本线程到当前代码系统CPU花费的时间
```

2.ERROR>
```

```
