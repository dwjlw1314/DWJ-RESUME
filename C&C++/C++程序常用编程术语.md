下图显示了编译器和链接器分别在程序段中写入的内容
![image](https://github.com/dwjlw1314/DWJ-RESUME/raw/master/PictureSource/1.1.1.jpg)

BSS段是(Block Started by Symbol)缩写，该段只保存未初始化的变量，运行时所需要的BSS段大小记录在目标程序中，
且BSS段不占据目标程序的任何空间。目标程序和可执行文件有不同的格式，SVr4中采用ELF，其他系统使用COFF格式，
局部变量声明或定义只有在程序运行期间占用目标程序空间

![image](https://github.com/dwjlw1314/DWJ-RESUME/raw/master/PictureSource/1.1.2.jpg)

堆栈段是用于保存局部变量、临时数据、传递到函数中的参数等。注意虚拟地址空间的最低部分未被映射，任何对它的引用都是非法的，
典型情况下它是从地址零开始的几K字节，用于捕捉使用空指针和小整形值的指针引用内存的情况

下图是ASCII码对照表
![image](https://github.com/dwjlw1314/DWJ-RESUME/raw/master/PictureSource/1.1.3.jpg)

下图是常见类型提升
![image](https://github.com/dwjlw1314/DWJ-RESUME/raw/master/PictureSource/1.1.4.jpg)

在所有表达式中，每个char都被转换为int，float被转换成double。函数参数也是表达式，所以参数传递给函数时也发生转换。
ANSI C表示如果编译器能够保证运算结果一致，可以忽略类型提升---这通常出现在表达式中存在常量操作数的情况

C++重载overload,覆盖override,重写overwrite的区别
```
1.override（方法覆盖）
子类继承了父类的同名无参函数。当子类从父类继承了一个无参函数，而又定义了一个同样的无参函数，则子类定义的方法覆盖父类的方法，称为覆盖

特征：函数名字相同，参数不同

2.overwrite（方法重写）
当前类的同名方法。通过方法的重写，一个类可以有多个具有相同名字的方法，由传递给它们不同的个数和类型的参数来决定使用哪种方法。因此，重写的名称是当前类中的同名函数，不是父类中的函数名

特征：位于派生类当中，函数名字相同，参数相同，基类中必须有vitural关键字，即必须是虚函数

3.overload（方法重载）
子类继承了父类的同名有参函数。当子类继承了父类的一个同名方法，且方法参数不同，称为重载。通过方法的重载，子类可以重新实现父类的某些方法，使其具有自己的特征

特征：位于派生类中，函数名字相同，参数不同或者相同，基类中没有virtual关键字，其实和覆盖一样
```
