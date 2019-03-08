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
