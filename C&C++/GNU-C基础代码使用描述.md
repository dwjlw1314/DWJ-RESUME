<font color=#FF0000 size=5> <p align="center">__attribute__((visibility("default")))</p></font>

code:
```
#if defined(__GNUC__) && __GNUC__ >= 4
#define AMQP_PUBLIC_FUNCTION __attribute__((visibility("default")))
```
desc:
```
visibility用于设置so文件中函数的可见性，将变量或函数设置为default，则该变量或函数在其他库可见
如果将变量或函数设置为hidden(默认模式)，则该变量或函数仅在本so中可见，在其他库中则不可见

g++在编译时，可用参数-fvisibility指定所有符号的可见性(不加此参数时默认外部可见，参考man g++中-fvisibility部分);
若需要对特定函数的可见性进行设置，需在代码中使用__attribute__设置visibility属性

编写大型程序时，可用-fvisibility=hidden设置符号默认隐藏，针对特定变量和函数，在代码中使用__attribute__ ((visibility("default")))令该符号外部可见，这种方法可用有效避免动态库之间的符号冲突
```

<font color=#FF0000 size=5> <p align="center">__attribute__((format()))</p></font>

code:
```
__attribute__((format(printf, a, b)))
__attribute__((format(scanf, a, b)))
```
desc:
```
属性可以给被声明的函数加上类似printf或者scanf的特征，它可以使编译器检查函数声明和函数实际调用参数之间的格式化字符串是否匹配。format属性告诉编译器，按照printf, scanf等标准C函数参数格式规则对该函数的参数进行检查,这对自己封装调试信息的接口时非常的有用

format的语法格式为：format (archetype, string-index, first-to-check)
其中，“archetype”指定是哪种风格；“string-index”指定传入函数的第几个参数是格式化字符串；“first-to-check”指定从函数的第几个参数开始按上述规则进行检查

#include <stdarg.h>

#if 1
#define CHECK_FMT(a, b)	__attribute__((format(printf, a, b)))
#else
#define CHECK_FMT(a, b)
#endif

void TRACE(const char *fmt, ...) CHECK_FMT(1, 2);

void TRACE(const char *fmt, ...)
{
  va_list ap;
  va_start(ap, fmt);
  (void)printf(fmt, ap);
  va_end(ap);
}

int main(void)
{
	TRACE("iValue = %d\n", 6);
	TRACE("iValue = %d\n", "test");
	return 0;
}
注意：需要打开警告信息即(-Wall)，编译结果如下所示：
main.cpp: In function ‘int main()’:
main.cpp:26:31: warning: format ‘%d’ expects argument of type ‘int’, but argument 2 has type ‘const char*’ [-Wformat=] TRACE("iValue = %d\n", "test")
如果不使用__attribute__ format则不会有警告
```

<font color=#FF0000 size=5> <p align="center">__attribute__ ((const))</p></font>

code:
```
extern int square(int n) __attribute__((const));
```
desc:
```
该属性只能用于带有数值类型参数的函数上。当重复调用带有数值参数的函数时，由于返回值是相同的，所以此时编译器可以进行优化处理，除第一次需要运算外，其它只需要返回第一次的结果就可以了，进而可以提高效率。该属性主要适用于没有静态状态和副作用的一些函数，并且返回值仅仅依赖输入的参数。下面例子将重复调用一个带有相同参数值的函数，具体如下：

extern int square(int n) __attribute__((const));
...
for (i = 0; i < 100; i++)
{
  total = square(i) + total;
}
通过添加__attribute__((const))声明，编译器只调用了函数一次，以后只是直接得到了相同的一个返回值。类似getchar或time的函数是不适合使用该属性的
```

<font color=#FF0000 size=5> <p align="center">__attribute__ ((noreturn))</p></font>

code:
```
extern void exit(int) __attribute__((noreturn));
extern void abort(void) __attribute__((noreturn));
```
desc:
```
该属性通知编译器函数从不返回值，当遇到类似函数需要返回值而却不可能运行到返回值处就已经退出来的情况，该属性可以避免出现错误信息。C库函数中的abort和exit的声明格式就采用了这种格式

//name: noreturn.c 测试__attribute__((noreturn))
extern void myexit();

int test(int n)
{
  if ( n > 0 )
  {
    myexit();
    /* 程序不可能到达这里 */
  }
  else
    return 0;
}
编译显示的输出信息为：
noreturn.c: In function `test':
noreturn.c:10: warning: control reaches end of non-void function

加上((noreturn))之后，编译就不会再出现警告信息
```

<font color=#FF0000 size=5> <p align="center">__attribute__ ((__nothrow__))</p></font>

code:
```
# if !defined __cplusplus && __GNUC_PREREQ (3, 3)
#   define __THROW	__attribute__ ((__nothrow__))
#   define __NTH(fct)	__attribute__ ((__nothrow__)) fct
# else
#   if defined __cplusplus && __GNUC_PREREQ (2,8)
#     define __THROW	throw ()
#     define __NTH(fct)	fct throw ()
#   else
#     define __THROW
#     define __NTH(fct)	fct
#   endif
# endif
```
desc:
```
__attribute__关键字主要是用来在函数或数据声明中设置其属性。给函数赋给属性的主要目的在于让编译器进行优化
在C语言下为表示该函数不会抛出异常，但是在C++中就会定义为throw(),和C++异常有关,在#include<cdefs.h>中定义
```

<font color=#FF0000 size=5> <p align="center">__attribute__ ((constructor))和((destructor))</p></font>

code:
```
__attribute__ ((constructor)) static void data_init(void);
__attribute__ ((destructor)) static void data_uninit(void);
```
desc:
```
constructor指定的函数在共享库loading的时候调用，destructor指定的函数在共享库unloading的时候调用

1.动态库源码文件data.c如下
__attribute__ ((constructor)) static void data_init(void);
__attribute__ ((destructor)) static void data_uninit(void);

void data_init(void)
{
    printf("call data init./n");
}

void data_uninit(void)
{
    printf("call data uninit./n");
}

void data_test(const char* msg)
{
    printf("call %s./n",msg);
}

2.编译为共享库libdata.so
gcc -fPIC -c data.c
gcc -shared -o libdata.so data.o

3.调用文件calllib.c
int main(int argc, char **argv)
{
    data_test("first");

    /* below only for testing dynamic linking loader */
    void *handle;
    void (*test)();

    handle = dlopen ("/path_to_lib/libdata.so", RTLD_LAZY);
    test = dlsym(handle, "data_test");
    (*test)("second");
    dlclose(handle);
    return 0;
}

4.编译calllib运行, data_init()和data_uninit()已被自动调用，结果如下：
call data init.
call first.
call second.
call data uninit.
```

<font color=#FF0000 size=5> <p align="center">__attribute__((deprecated))和__declspec(deprecated)</p></font>

code:
```
#if AV_GCC_VERSION_AT_LEAST(3,1)
#    define attribute_deprecated __attribute__((deprecated))
#elif defined(_MSC_VER)
#    define attribute_deprecated __declspec(deprecated)
#else
#    define attribute_deprecated
#endif
```

desc:
```
gcc和VC编译器分别使用__attribute__((deprecated))和__declspec(deprecated)来管理过时的代码
```

<font color=#FF0000 size=5> <p align="center">cdecl和stdcall的区别</p></font>

code:
```
#define PASCAL __stdcall
#define AMQP_CALL __cdecl
```
desc:
```
函数的调用方式有两种：一种是PASCAL调用方式，另一种是C调用方式
使用PASCAL调用方式，被调函数在返回到调用者之前将参数从栈中删除
使用C调用方式，参数的删除是调用者完成的

VC中主要有两种函数调用方式：一种是__stdcall;另一种是__cdecl;还有第三种naked(自己编汇编控制堆栈)

Windows系统规定由系统调用的函数都遵守 PASCAL调用方式。但是VC中函数的缺省调用方式是__cdecl，也就是C调用方式。
在Windows编程中将遇到很多声明修饰符，如CALLBACK,WINAPI,PASCAL这些在IntelCPU的计算机上都是__stdcall
几乎我们写的每一个WINDOWS API函数都是__stdcall类型的

__cdecl和__stdcall两者之间的区别：

windows的函数调用时需要用到栈(STACK，一种先入后出的存储结构)。当函数调用完成后，栈需要执行清除操作，
如果函数使用了_cdecl，那么栈的清除工作是由调用者，用COM的术语来讲就是客户来完成的。
这样就带来了一个棘手的问题，不同的编译器产生栈的方式不尽相同，那么调用者就不能正常的完成清除工作
如果使用__stdcall，上面的问题就解决了，函数自己解决清除工作。所以，在跨开发平台的调用中，都使用__stdcall
当遇到这样的函数如fprintf()它的参数是可变的，不定长的，被调用者事先无法知道参数的长度，事后的清除工作也无法正常的进行，
因此，这种情况只能使用__cdecl。如果程序中没有涉及可变参数，最好使用__stdcall关键字
```
