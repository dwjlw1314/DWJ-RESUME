printf格式输出
```
code: printf("%*.*s\n",6,4,"1234567890");
desc: 是以6位宽度，输出“1234567890”的前4个有效字符
eg: printf("exchange %.*s\n",exchange.len,exchange.value);
```

检查linux系统中程序内存是否泄漏
```
valgrind -v --leak-check=full --tool=memcheck ./dataAccess
```

指向指针的引用
```
int *p = nullptr;
//ref是一个对指针p的引用；从右向左理解ref，离变量名最近的符号对变量有最直接的影响，因此下面表达式中
ref是一个引用，符号*说明ref引用的是一个指针，最后指出ref引用的是一个int指针
int *&ref = p;
```

在预处理过程中产生编译时错误信息一般用以下方式
```
#if !defined(__cplusplus)
#error C++ compiler required.
#endif
```

extern多维数组对象声明
```
多维数组声明需要提供除了最左边一维之外其他维度的长度-给编译器足够的信息产生相应的代码
int m[10][4][3];
extern int m[][4][3];
```

编译时检查msg_t是否适合zmq_msg_t，在编译期提示错误
```
typedef char check_size[sizeof(msg_t) == sizeof(zmq_msg_t) ? 1 : -1];
check_size gjsy_array;   //声明只有一个char类型元素的数组
gjsy_array[0] = 'y';
```

c++代码中，凡是涉及到new操作，都采用new(std::nothrow)，然后进行Test-for-NULL检查
```
//当new一个对象失败时，默认设置该对象为NULL
char *p = new (std::nothrow) char[1];
//进行Test-for-NULL检查
if (NULL == p)
	...
```

c++中new的三种用法
```
1.new表达式完成了两件事情：申请内存和初始化对象
string* ps = new string("abc");

2.new操作符类似于C语言中的malloc，只是负责申请内存
void* buffer = operator new(sizeof(string));

3.官方说法placement new，它用于在给定的闲置内存中初始化对象
//在buffer所指向的已存在内存中初始化string类型的对象
buffer = new(buffer) string("abc");
```

memmove与memcpy的区别及实现
```
1.与字符串函数strcpy区别：
memcpy与memmove都是对内存进行拷贝可以拷贝任何内容，而strcpy仅是对字符串进行操作
memcpy与memmove拷贝多少是通过其第三个参数进行控制而strcpy是当拷贝至'\0'停止

2.函数说明：         
memcpy函数的功能是从源src所指的内存地址的起始位置开始拷贝N个字节到目标dst所指的内存地址的起始位置中
memmove函数的功能同memcpy基本一致，但是当src区域和dst内存区域重叠时，memcpy可能会出现错误，而memmove能正确进行拷贝

3.拷贝情况：
拷贝的具体过程根据dst内存区域和src内存区域可分为三种情况：
1).当src内存区域和dst内存区域完全不重叠
2).当src内存区域和dest内存区域重叠时且dst所在区域在src所在区域前
3).当src内存区域和dst内存区域重叠时且src所在区域在dst所在区域前
上述三种情况，memcpy可以成功对前两种进行拷贝，对第三种情况进行拷贝时，由于拷贝dst前两个字节时覆盖了src原来的内容，所以接下来的拷贝会出现错误。而memmove对第三种情况进行拷贝时会从src的最后向前拷贝N个字节，避免了覆盖原来内容的过程

4.代码实现:
memcpy：
void* _memcpy(void* dest, const void* src, size_t count)
{
	assert(src != nullptr&&dest != nullptr);
	char* tmp_dest = (char*)dest;
	const char* tmp_src = (const char*)src;
	//将指针dest和指针src由void强转为char,使得每次均是对内存中的一个字节进行拷贝
	while (count--) *tmp_dest++ = *tmp_src++;
	return dest;
}

memmove:
void* _memmove(void* dest, const void* src, size_t count)
{
	assert(src != nullptr&&dest != nullptr);
	char* tmp_dest = (char*)dest;
	const char* tmp_src = (const char*)src;

	if (tmp_src < tmp_dest) //当src地址小于dest地址时，从头进行拷贝
    while (count--) *tmp_dest++ = *tmp_src++;
	else if (tmp_src > tmp_dest) //当src地址大于dest地址时，从后进行拷贝
	{
		tmp_src += count - 1;
		tmp_dest += count - 1;
		while (count--)	*tmp_dest-- = *tmp_src;
	}
	//else(tmp_src==tmp_dest) 此时不进行任何操作
	return dest;
}
```

UNIX系统实现逐字符的输入方式(其他方式：curses函数库只提供基于字符的屏幕控制函数；中断驱动I/O)
```
1.stty终端实现方式，是一种间接阻塞模式
int main(void) {
	char c;
	/* 终端字符驱动修改为一次一字符模式 */
	system("stty raw -echo");
	c = getchar();
	/* 终端字符驱动模式还原到一次一行模式 */
	system("stty cooked echo");
}

2.ioctl系统调用方式，非阻塞模式

int kb()
{
	int i;
	ioctl(0, FIONREAD, &i);
	return i;
}
int main(void) {

	char c = ' ';
	system("stty raw -echo");
	while(1)
	{
		if (kb())
		{
			c = getchar();
		}
	}
	perror("run ");
	system("stty cooked echo");
}
```

使用指针从函数返回一个数组方式
```
1.声明和定义一个function函数，返回一个指向包含20个int元素的数组指针
int (*function())[20]
{
	int (*p)[20];
	p = malloc(20 * sizeof(int));
	return p;
}
2.调用方法
int (*p)[20];
p = function();
(*p)[3] = 5;
3.或者使用结构体存储数组的方式进行返回
```

cmake和make编译选项
```
CMAKE_INSTALL_PREFIX=/opt     #指定安装路径
CMAKE_BUILD_TYPE=Release      #编译release版本
-DBUILD_SHARED_LIBS=1         #编译动态库
make DESTDIR=/opt/ install    #指定安装路径
```
