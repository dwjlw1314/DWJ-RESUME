<font color=#FF0000 size=5> <p align="center">__attribute__((visibility("default")))</p></font>

code: #define AMQP_PUBLIC_FUNCTION __attribute__((visibility("default")))
```
visibility用于设置so文件中函数的可见性，将变量或函数设置为default，则该变量或函数在其他库可见
如果将变量或函数设置为hidden(默认模式)，则该变量或函数仅在本so中可见，在其他库中则不可见

g++在编译时，可用参数-fvisibility指定所有符号的可见性(不加此参数时默认外部可见，参考man g++中-fvisibility部分);
若需要对特定函数的可见性进行设置，需在代码中使用__attribute__设置visibility属性

编写大型程序时，可用-fvisibility=hidden设置符号默认隐藏，针对特定变量和函数，在代码中使用__attribute__ ((visibility("default")))令该符号外部可见，这种方法可用有效避免动态库之间的符号冲突
```
