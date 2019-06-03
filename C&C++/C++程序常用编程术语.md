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
