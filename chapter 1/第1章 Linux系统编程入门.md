# 第1章 Linux系统编程入门

## 1.1 Linux开发环境搭建

### 1.安装LInux系统(虚拟机安装)

### 2.安装XSHELL XFTP

### 3.安装Visual Studio Code

## 1.2 GCC(1)

### 源代码 .h .c .cpp

### 预处理后源代码(注释删掉 头文件展开，宏替换等)，以.i结尾

### 汇编代码 以.s结尾

### 启动代码 目标代码 库代码，以.o结尾

### 可执行文件

## 1.3 GCC(2)

### -o选项用来指定输出文件的文件名

### -I directory 指定include 包含文件的搜索目录

### -g 在编译的时候，生成调试信息，该程序可以被调试器调试

### -D 在程序编译的时候，指定一个宏 调试方便

### -w 不生成任何警告信息

### -Wall 生成所有警告信息

### -On  n取值范围为0-3 编译器优化的4个级别

- -O0 表示没有优化
- -O1为缺省值
- -O3 优化级别最高

### -l 在程序编译的时候，指定使用的库

### -L 指定编译的时候，搜索的库的路径

### -fprc/fpic 生成与位置无关的代码

### -shared 生成共享目标文件，通常用在建立共享库时

### -std 指定c语言，如:-std=c99, gcc默认的方言是GUN C

## 1.4 静态库的制作

### 什么是库？

- 库文件是计算机上的一类文件，可以简单的把库文件看成一种代码仓库，它提供给使用者一些可以直接拿来用的变量、函数或类。

### 库的好处？

- 1.代码保密
- 2.方便部署和分发

### 命名规则

- Linux：libxxx.a

	- lib：前缀
	- xxx:库的名字，自己起
	- .a：后缀

### 制作过程

- gcc获得.o文件
- 将.o文件打包，使用ar工具（archive）

	- ar rcs libxxx.a xxx.o xxx.o
	- r - 将文件插入备份文件中
	- c - 建立备存文件
	- s - 索引

## 1.5 静态库的使用

### gcc main.c -o app -I ./include/ -l calc -L ./lib/

- -I directory 指定include 包含文件的搜索目录
- -l 在程序编译的时候，指定使用的库
- -L 指定编译的时候，搜索的库的路径

## 1.6 动态库的制作和使用

### 命名规则

- Linux:libxxx.so

	- lib:前缀（固定）
	- xxx:库的名字，自己起
	- .so : 后缀(固定)
	- 在linux下是一个可执行文件

### 动态库的制作

- gcc得到.o文件，得到和位置无关的代码

	- gcc -c -fpic/-fPIC a.c b.c

- gcc得到动态库

	- gcc -shared a.o b.o -o libcalc.so

### 动态库的使用

- gcc main.c -o main -I ./include/ -L lib/ -l calc

## 1.7 动态库加载失败的原因

### 静态库：gcc 进行链接时，会把静态库中代码打包到可执行程序中

### 动态库：gcc进行链接时，动态库的代码不会被打包到可执行程序中

### 程序启动之后，动态库会被动态加载到内存中，通过ldd命令检查动态库依赖关系

### 如何定位共享库文件？

- 当系统加载可执行代码时候，能够知道其所依赖的库的名字，但是还需要知道绝对路径。此时就需要系统的动态载入器来获取该绝对路径。对于elf格式的可执行程序，是由ld-linux.so完成的，它先后搜索elf文件的
DT_RPATH段-->
环境变量LD_LIBRARY_PATH --> 
/etc/ld.so.cache文件列表 -->
/lib/, /usr/lib目录
找到库文件后将其载入内存
- ldd main 是查找库文件位置

## 1.8 解决动态库加载失败问题

### export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/wcf/test/lesson06/library/lib

### 临时性配置：
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/wcf/test/lesson06/library/lib

### 用户级别长久配置：
home目录下：vim .bashrc
source .bashrc //刷新

### 系统级别长久配置：
sudo vim /etc/profile

### sudo vim /etc/ld.so.conf 中添加路径
sudo ldconfig //刷新配置文件 

### 不建议动态库路径放置 /lib/, /usr/lib目录，跟系统库路径在一块容易误操作！

## 1.9 静态库和动态库的对比

### 静态库、动态库区别来自链接阶段如何处理，链接成可执行程序。分别称为静态链接方式和动态链接方式

### 静态库的优缺点

- 优点

	- 1.静态库被打包到应用程序中加载速度快
	- 2.发布程序无需提供静态库，移植方便

- 缺点

	- 1.消耗系统资源，浪费内存
	- 2.更新、部署、发布麻烦

### 动态库的优缺点

- 优点

	- 1.可以实现进程间资源共享（共享库）
	- 2.更新、部署、发布简单
	- 3.可以控制何时加载动态库

- 缺点

	- 1.加载速度比静态慢
	- 2.发布程序时需要提供依赖的动态库

## 1.10 Makefile

### 1.什么是Makefile?

- Makefile 文件定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作
- Makefile带来的好处就是“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率

### 2.makefile文件命名和规则

- 目标 ...：依赖 ...
         命令(Shell 命令)
         ...

	- 目标:最终要生成的文件
	- 依赖:生成目标所需要的文件或是目标
	- 命令:通过执行命令对依赖操作生成目标

- 工作原理

	- 命令在执行之前，需要先检查规则中的依赖是否存在?存在执行，不存在，检查下面的其他规则
	- makefile其他的规则都是为第一条服务的，如果没使用到下面的规则，则下面的规则不会执行！
	- 检测更新：比较依赖和目标的时间新旧程度，main,c比main.o时间更晚，说明更新了，重新执行命令

		- 好处：只执行更改的，不用去编译未更改的，节省时间

### 3.变量

- 自定义变量

	- 变量名=变量值
var=hello

- 预定义变量

	- AR:归档维护程序的名称，默认值为ar
	- CC:c编译器的名称，默认值为cc
	- CXX:c++编译器的名称，默认值为g++
	- $@:目标的完整名称
	- $<:第一个依赖文件的名称
	- $>:所有的依赖文件

- 获取变量的值

	- $(变量名)

### 4.模式匹配

- %.o:%.c
    gcc -c $< -o $@

	- -%:通配符，匹配一个字符串
	- -两个%匹配的是同一个字符串

### 5.函数

- $(wildcard PATTERN ...)

	- 功能：获取指定目录下指定类型的文件列表
	- 参数：PATTERN指的是某个或多个目录下的对应的某种类型的文件，如果有多个目录，一般使用空格间隔
	- 返回：得到的若干个文件的文件列表，文件名之间使用空格间隔
	- 示例：
$(wildcard *.c ./sub/*.c)
返回值格式：a.c b.c d.c e.c

- $(patsubst <pattern>,<replacement>,<text>)

	- 功能：查找<text>中的单词，符合pattern 则替换为replacement
	- 示例：|
$(patsubst %.c,%.o,x.c bar.c)
返回值格式：x.o bar.o

### 6.注意事项

-   .PHONY:clean
 clean:
     rm $(objs) -f 

	- 伪目标不生成文件，每次都会执行，不会受到已存在文件影响

## 1.11 GDB

### 1.什么是GDB?

- GDB是GNU软件系统社区提供的调试工具，同GCC配套组成了一套完整的开发环境
- 功能

	- 1.启动程序，可以按照自定义的要求随心所欲的运行程序
	- 2.可让被调试的程序在所指定的调试的断点处停住
	- 3.当程序被停住时，可以检查此时程序中所发生的事
	- 4.可以改变程序，将一个BUG产生的影响修正从而测试其他BUG

### 2.准备工作

- 关掉编译器的优化选项（’-O‘）
- 打开调试选项('-g')

	- -g选项的作用是在可执行文件中加入源代码的信息

- 打开所有warning '-Wall'

### 3.GDB 命令-启动、退出、查看代码

- 启动和退出：
gdb 可执行程序
quit
- 给程序设置参数/获取设置参数
set args 10 20
show args
- GDB 使用帮助
help
- 查看当前文件代码
list/l (从默认位置显示)
list/l 行号 （从指定的行显示）
list/l 函数名 （从指定的函数显示）
- 查看非当前文件代码
 list/1 文件名:行号
 list/1 文件名:函数名
- 设置显示的行数
show list/listsize
set list/listsize 行数

### 4.GDB命令-断点操作

- 设置断点
b/break 行号
b/break 函数名
b/break 文件名：行号
b/break 文件名：函数

	- 断点位置的语句还没有执行！！！

- 查看断点
i/info b/break
- 删除断点
 d/del/delete 断点编号
- 设置断点无效
 dis/disable 断点编号
- 设置断点生效
 ena/enable 断点编号
- 设置条件断点(一般用在循环位置)
b/break 10 if i==5

### 5.GDB命令-调试命令

- 运行GDB程序
 start(程序停在第一行)
 run(遇到断点才停)
- 继续运行，到下一个断点停
c/continue
- 向下执行一行代码(不会进入函数体，整个函数当条语句执行了！)
n/next
- 变量操作
p/print 变量名(打印变量值)
ptype 变量名 (打印变量类型)
- 向下单步调试(进入函数体)
s/step
finish(跳出函数体，函数里面不能有断点)
- 自动变量操作
 display num(自动打印指定变量的值)
 i/info display
 undisplay 编号
- 其它操作
 set var 变量名=变量值
 until (跳出循环，循环里面不能有断点)

## 1.12 标准C库IO函数和Linux系统IO函数对比

### 1.标准C库IO函数

- 文件IO

	- 文件角度：
输入：从内存写到文件
输出：从文件读取到内存
	- 以内存为主体，站在内存角度：
输入：从文件把数据读入到内存
输出：从内存把数据输出到文件

- 两种跨平台方式

	- 1.java虚拟机，任何平台写的代码一致
	- 2.调用不同平台api

- 调用fopen

	- 返回FILE * fp

		- 文件描述符(整数值)

			- 定位文件

		- 文件读写指针位置
		- I/O缓冲区(内存地址)

			- 作用：提高执行效率，不够及时，不用频繁操作硬件
			- 1.刷新缓冲区
2.缓冲区已满
3.正常关闭文件

### 2.标准C库IO和Linux系统IO的关系

- 用户程序

	- C标准I/O库

		- 内核
write
read

			- 磁盘

### 3.虚拟地址空间

- 32位机器 虚拟地址空间为4G

	- 内核区

		- 内存管理
		- 进程管理
		- 设备驱动管理
		- VFS虚拟文件系统

	- 用户区

		- 环境变量
		- 命令行参数
		- 栈空间(局部变量) 高地址->低地址
		- 共享库(动态库、静态库)
		- 堆空间（new、malloc）低地址->高地址
		- .bss(未初始化全局变量)
		- .data(已初始化全局变量)
		- .text(代码段，二进制机器指令)
		- 受保护的地址（0-4k）

### 4.文件描述符

- 内核区负责管理

	- PCB进程控制块

		- 文件描述符表

			- 默认打开 指向当前终端 是设备文件

				- 0--> 标准输入
				- 1--> 标准输出
				- 2--> 标准错误

			- 默认是打开状态
			- 每打开一个新文件，则占用一个文件描述符，而且是空闲的最小的一个文件描述符
			- 一个文件可以被打开多次，且打开文件描述符不一致

### 5.Linux系统IO函数

- int open(const char *pathname,int flags);

	- 参数：
 - pathname: 要打开的文件路径
 - flags: 对文件的操作权限设置和其他的设置
 O_RDONLY,O_WRONLY,O_RDWR 这三个设置是互斥的
	- 调用失败返回-1，调用成功返回文件描述符
	- errno:属于Linux系统函数库，库里面一个全局变量，记录的是最近的错误号。
	-  void perror(const char *s);作用：打印errno对应的错误描述

		- s参数：用户描述，比如hello，最终输出的内容是 hello:xxx(实际的错误描述)

- int open(const char *pathname, int flags, mode_t mode)

	- 参数：
            - pathname:要创建的文件的路径
            - flags:对文件的操作权限和其他设置
-必选项：O_RDONLY, O_WRONLY,  O_RDWR 互斥
-可选项：O_CREAT 文件不存在，创建新文件

		- flags参数是一个int类型的数据，占4个字节，32位。
flags 32个位，每一位就是一个标志位。

	-             - mode:八进制的数，表示用户对创建处的新的文件的操作权限，比如0777
	-             当前用户 用户所在组 其他组: -rwxrwxrwx
            umask 0002 
            0777    111 111 111
            最终的权限：mode & ~umask  --> 0775 
            umask作用就是抹去某些权限    

- ssize_t read(int fd,void *buf,size_t count)

	- 参数：
- fd:文件描述符，open得到的，通过这个文件描述符操作某个文件
- buf:需要读取数据存放的地方，数组的地址(空的字符串放进去读，传出参数)
- count:指定的数组的大小
	- 返回值：
 -成功：
           >0:返回实际的读取到的字节数
           =0:文件已经读取完了
 -失败：-1，并且设置errno

- ssize_t write(int fd,const void *buf,size_t count)

	- 参数：
            - fd:文件描述符，open得到的，通过这个文件描述符操作某个文件
            - buf:要往磁盘写入的数据，数组
            - count:要写的数据的实际的大小
	- 返回值：
        成功：实际写入的字节数
        失败：返回-1，设置errno

- off_t lseek(int fd,off_t offset,int whence)

	-     标准C库的函数
    #include <stdio.h>
    int fseek(FILE *stream, long offset, int whence);
	-     Linux系统函数
    off_t lseek(int fd, off_t offset, int whence);

		- 参数：
   - fd:文件描述符，通过open得到的，通过这个fd操作某个文件
   - offset:偏移量
   - whence:
    SEEK_SET              
           设置文件指针的偏移量
    SEEK_CUR
   设置偏移量：当前位置+第二个参数offset的值
    SEEK_END
   设置偏移量：文件大小+第二个参数offset的值
		- 返回值：返回文件指针的位置
		- 作用

			- 1.移动文件指针到文件头
lseek(fd,0,SEEK_SET)
			- 2.获取当前文件指针的位置
lseek(fd,0,SEEK_CUR)
			-  3.获取文件长度
 lseek(fd,0,SEEK_END);
			- 4.扩展文件的长度，当前文件10b,110b,增加了100个字节
lseek(fd,100,SEEK_END)
需要写一次数据

- int stat(const char *pathname, struct stat *statbuf);

	- 作用：获取一个文件相关的信息
	- 参数：
- pathname:操作的文件的路径
- statbuf：结构体变量，传出参数，用于保存获取到的文件的信息

		- stat结构体

			- 文件的设备编号
			- 节点
			- 文件的类型和存取的权限

				- st_mode变量，一共16位

					- 0-8 是权限位
					- 9-11 是特殊权限位
					- 12-15 是文件类型

			- 连到该文件的硬连接数目
			- 用户ID
			- 组ID
			- 设备文件的设备编号
			- 文件字节数(文件大小)
			- 块大小
			- 块数
			- 最后一次访问时间
			- 最后一次修改时间
			- 最后一次改变时间

	- 返回值：
成功：返回0
失败：返回-1，设置errno

- int lstat(const char *pathname, struct stat *statbuf);

	- 与上面不同之处是 获取软链接文件的信息

### 6.文件属性操作函数

- int access(const char *pathname, int mode);

	- 作用：判断某个文件是否有某个权限，或者判断文件是否存在
	-  参数：
 - pathname:判断的文件路径
 - mode:
     R_OK:判断是否有读权限
     W_OK:判断是否有写权限
     X_OK:判断是否有执行权限
     F_OK:判断文件是否存在
	- 返回值：成功返回0，失败返回-1

- int chmod(const char *pathname, mode_t mode);

	- 作用：修改文件的参数
	- 参数：
        - pathname:需要修改的文件的路径
        - mode:需要修改的权限值，八进制的数
	- 返回值:成功返回0，失败返回-1

- int chown(const char *pathname, uid_t owner, gid_t group);

	- 参数：
    pathname:文件的路径
	- 作用：修改文件的所有组和拥有者

- int truncate(const char *path, off_t length);

	- 作用：缩减或者扩展文件的尺寸至指定的大小
	- 参数：
        -path:需要修改的文件的路径
        -length:需要最终文件变成的大小
	- 返回值：
    成功返回0,失败返回-1

### 7.目录操作函数

- int mkdir(const char *pathname,mode_t mode);

	-  作用:创建一个目录
	- 参数：
      pathname:创建的目录的路径
      mode:权限，八进制的数
	- 返回值：
     成功返回0，失败返回-1

- int rmdir(const char *pathname);

	- 作用：刪除空目录
	- 参数：
      pathname:创建的目录的路径
	- 返回值：
     成功返回0，失败返回-1

- int rename(const char *oldpath, const char *newpath);

	- 作用：重命名文件及文件夹

-  int chdir(const char *path);

	- 作用：修改进程的工作目录
比如在/home/wcf启动了一个可执行程序a.out,进程的工作目录是/home/wcf
	- 参数：
path:需要修改的工作目录

- char *getcwd(char *buf, size_t size);

	- 作用：获取当前的工作目录
	- 参数：
 -buf：存储的路径，指向的是一个数组(传出参数)
 -size：数组的大小
	- 返回值：
    返回的指向的一块内存，这个数据就是第一个参数

### 8.目录遍历函数

- DIR *opendir(const char * name);

	- 作用：打开目录流
	- 参数：
 -name:需要打开的目录的名称
返回值:
DIR *类型,理解为目录流
错误返回NULl

- struce dirent *readdir(DIR *dirp);

	- 读取目录中的数据
	- -参数：
    dirp是opendir返回的结果
-返回值：
    struct dirent,代表读取到的文件的信息
    读取到了末尾或者失败返回null

- int closedir(DIR *dirp);

	- 作用：关闭目录流

### 9.dup、dup2函数

- int dup(int oldfd);

	- 作用：复制一个新的文件描述符，新的和旧的文件描述符指向同一个文件
新的是从空闲的文件描述符表中找一个最小的

- int dup2(int oldfd, int newfd);

	- 作用：重定向文件描述符
	- oldfd 指向a.txt newfd指向b,txt
调用函数成功后:newfd和b.txt做close,newfd指向了a.txt
	- oldfd 必须是一个有效的文件描述符
oldfd和newfd值相同，相当于什么都没有做

### 10.fcntl函数

- int fcntl(int fd, int cmd, ...);

	- fd:表示需要操作文件描述符
	- cmd:表示对文件描述符进行如何操作

		- - F_DUPFD：复制文件描述符，复制的是第一个参数fd,得到一个新的文件描述符(返回值返回)
int ret =fcntl(fd,F_DUPFD);
		- - F_GETFL：获取指定的文件描述符文件状态flag
 获取的flag和我们通过open函数传递的flag是一个东西。
		- - F_SETFL：设置文件描述符文件状态flag

			- 必选项:O_RDONLY,O_WRONLY,O_RDWR,不可以被修改
			- 可选项:O_APPEND、O_NONBLOCK

				- O_APPEND 表示追加数据
				- O_NONBLOCK 设置成非阻塞
				- 阻塞和非阻塞描述的是函数调用的行为

