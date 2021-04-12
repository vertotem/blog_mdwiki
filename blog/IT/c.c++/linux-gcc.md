## gcc
使用gcc，程序员可以对编译过程有更多的控制。
编译阶段： 预处理、编译、汇编、链接。
程序员可以再编译的任何阶段结束后停止整个编译过程以检查编译器在该阶段输出的信息。
### gcc常用选项
```
-E 预编译
-c 只编译，不链接        gcc –o hello hello.o  链接目标文件
-o filename 输出文件名，如果没指定filename，默认为a.out
-lstdc++ 代表gcc要用标准的c++的库完成链接
-g  包含调试信息
-D 后面直接跟宏命(**中间不加空格**)，相当于定义这个宏，默认这个宏的内容是1   例如-DLINUX   ==   #define  LINUX  1
-l  链接库文件,  库文件一般放在/lib和/usr/lib和/usr/local/lib，库名字为libxxx.so
-L  指定其他库文件所在的目录名,即去哪里找so文件， 需要配合-l一起使用
-I  参数是用来指定头文件目录，/usr/include目录一般是不用指定的，gcc知道去那里找，但是如果头文件
    不在/usr/include里我们就要用-I参数指定了，比如头文件放在/myinclude目录里，那编译命令行就要
    加上-I/myinclude参数了
-O  优化编译后的代码
-w  关闭所有告警信息
-Wall   开启所有告警信息
-Werror 所有告警当error来处理
ar cr libstaticprocess.a process.o
```
gcc默认只链接c的标准库，并不链接c++标准库，需要添加-lstd++
$gcc –lstdc++ -o hello hello.cpp
对于C语言和C++语言，gcc编译完的**目标代码**是不同的，最大的区别是C++会修改编译后的变量以及函数各种标号的名称
g++是gcc的c++编译器，如果是编译c++的文件，建议使用g++
### toolschain
nm(可能是name mapping的缩写) （查看可执行程序内部的标号，函数名等信息）
ldd （print shared library dependencies）
ar t libxxx.a  查看静态库中包含的目标文件
ar -r  libtinyxml.a  tinystr.o  tinyxml.o   打包成静态库


### 编译共享库so
so文件的源文件中不需要有main函数，即使有也不会被执行。
编译的时候gcc需要加-fPIC选项，这可以使gcc产生与位置无关的代码。
连接的时候gcc使用-shared选项，指示生成一个共享库文件。
共享库文件名要以lib开头，扩展名为.so。
makefile的例子：
```
.SUFFIXES:.c .o


CC=gcc
SRCS=test.c


EXEC=libtest.so


OBJS=$(SRCS:.c=.o)


start:$(OBJS)
$(CC) -shared -o $(EXEC) $(OBJS) 


.c.o:
$(CC) -g -fPIC -o $@ -c $<


clean:
rm -f $(OBJS)
```
so文件使用方法:
1. 为了让linux能找到so文件的位置，需要在.bash_profile中添加`export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:.`
或者将so文件放入linux的系统目录/usr/lib/
2. 在c文件中使用so文件，首先需要 `#inluce`相关h文件。
3. gcc连接时添加 –L参数指明so文件存放路径,-l参数指明so文件名
4. 为了使我们编写的so文件同时可以被C或者C++调用，我们需要修改一下函数申明部分 增加带有__cplusplus的预编译指令，extern "C"{}


## make
一种自动化编译的代码的工具
### Makefile结构说明
Makefile里主要包含了五个东西：变量定义、显式规则、隐晦规则、文件指示和注释。
1、变量的定义。在Makefile中我们要定义一系列的变量，变量一般都是字符串，这个有点像C语言中的宏，当Makefile被执行时，其中的变量都会被扩展到相应的引用位置上。
2、显式规则。显式规则说明了，如何生成一个或多的的目标文件。这是由Makefile的书写者明显指出，要生成的文件，文件的依赖文件，生成的命令。
3、隐晦规则。由于我们的make有自动推导的功能，所以隐晦的规则可以让我们比较粗糙地简略地书写Makefile，这是由make所支持的。
4、文件指示。其包括了三个部分，一个是在一个Makefile中引用另一个Makefile，就像C语言中的include一样。
5、注释。Makefile中只有行注释，和UNIX的Shell脚本一样，其注释是用“#”字符
### make是怎么完成工作的呢？
makefile是一个文本形式的脚本文件，其中包含一些规则告诉make编译哪些文件，怎么样编译以及在什么条件下编译。
makefile规则遵循以下通用形式
```
target:dependency [dependency[…]]
    command
    command
    […]
```
每个command第一个字符必须是tab键，而不是空格键，不然make会报错并停止。
### 最简单的makefile
```
start:
    gcc -o hello hello.c
```
### 稍微复杂的makefile
```
start:hello.o
        gcc -o hello hello.o
        @echo '---------------ok---------------'
hello.o:
        gcc -o hello.o -c hello.c
clean:
        rm -f hello.o
```
1. target start后面的hello.o代表其下的command依赖与hello.o这个target。所以make先执行了hello.o这个target下的command。
2. 输入make clean,make会直接执行clean其下的command。
3. 增加了@echo，显示编译成功语句，为了不将语句本身输出，所以前面加@符号。
### 使用变量
为了简化编辑和维护makefile，可以在makefile中使用变量。
varname=some_text
把变量用括号括起来，前面加`$`就可以引用该变量的值,  `$(varname)`
按照惯例makefile的变量都是大写（只是习惯而已，不是必须的）。
```
CC=gcc
SRCS=hello.c
OBJS=hello.o
# OBJS=$(SRCS:.c=.o)  更简便的写法，意思是将SRCS变量中的.c替换为.o
EXEC=hello
start:hello.o
        $(CC) -o $(EXEC) $(OBJS)
        @echo '---------------ok---------------'
hello.o:
        $(CC) -o $(OBJS) -c $(SRCS)
clean:
        rm -f hello.o
```
### 隐晦规则
```
%.o : %.c
```
`%`通配符，当想生成x.o文件时发现与`%.o : %.c`匹配，则将%替换为x，也就是x.o : x.c，则会执行规则的命令。
表示make定义了一条规则，任何x.o文件都从 x.c编译而来
make定义了一些有用的**预定义变量**
`$@`  表示目标文件
`$<`  表示第一个依赖
完整例子：
```
CC=$(CROSS)clang $(TARGET_FLAGS)  -lstdc++
AR=$(CROSS)ar -cr
SRCS=Circle.cpp Edge.cpp Math.cpp NavPath.cpp Path.cpp Point.cpp Polygon.cpp Triangle.cpp
OBJS=$(SRCS:.cpp=.o)
EXEC=libNavPath.a
start: $(OBJS)
    $(AR)  $(EXEC) $(OBJS)
    @echo '----------------ok------------$(CC)'
%.o: %.cpp:
    $(CC) -Wall -g -o $@ -c $<
clean:
    rm -f $(OBJS)
    rm -f libNavPath.a
```
#### 规则中加匹配
```
$（C_OBJS）：$（OBJ_PATH）/%.o:%.c


$(C_OBJS) 中有很多文件，假设各种各样的都有 （实际中肯定是你自己定义的，不会乱七八糟）
看看这些文件里面，找出匹配符合 $（OBJ_PATH）/%.o的，其实就是过滤出满足$（OBJ_PATH）/%.o的。


一般来说
$（C_OBJS）：$（OBJ_PATH）/%.o:%.c
和规则
$（P_OouDIR）/%.o:%.c
是等价的
但是一般性而言，并不等价，比如$(C_OBJS)还定义了其它文件，比如 ./lib/xxx.o
静态模式规则的好处，是能精确定义，哪个文件，依赖哪个文件。规定了预选范围。
%.o:%.c 则是泛泛而谈，只要匹配，就是这样。
```


### 其他
#### foreach使用
```
$(foreach <var>,<list>,<text>)
```
这个函数的意思是，把参数`<list>`中的单词逐一取出放到参数`<var>`所指定的变量中，然后再执行`< text>`所包含的表达式。每一次`<text>`会返回一个字符串，循环过程中，`<text>`的所返回的每个字符串会以空格分隔，最后当整个循环结束时，`<text>`所返回的每个字符串所组成的整个字符串（以空格分隔）将会是foreach函数的返回值。


所以，`<var>`最好是一个变量名，`<list>`可以是一个表达式，而`<text>`中一般会使用`<var>`这个参数来依次枚举`<list>`中的单词。举个例子：
```
    names := a b c d


    files := $(foreach n,$(names),$(n).o)
```
上面的例子中，`$(name)`中的单词会被挨个取出，并存到变量“n”中，`$(n).o`每次根据`$(n)`计算出一个值，这些值以空格分隔，最后作为foreach函数的返回，所以，$(files)的值是`a.o b.o c.o d.o`。


#### wildcard
在Makefile规则中，通配符会被自动展开。但在变量的定义和函数引用时，通配符将失效。这种情况下如果需要通配符有效，就需要使用函数“wildcard”，它的用法是：`$(wildcard PATTERN...)` 。在Makefile中，它被展开为已经存在的、使用空格分开的、匹配此模式的所有文件列表。
一般我们可以使用`$(wildcard *.c)`来获取工作目录下的所有的.c文件列表。复杂一些用法；可以使用`$(patsubst %.c,%.o,$(wildcard *.c))`，首先使用“wildcard”函数获取工作目录下的.c文件列表；之后将列表中所有文件名的后缀.c替换为.o。这样我们就可以得到在当前目录可生成的.o文件列表。因此在一个目录下可以使用如下内容的Makefile来将工作目录下的所有的.c文件进行编译并最后连接成为一个可执行文件：
```
objects := $(patsubst %.c,%.o,$(wildcard *.c))
foo : $(objects)
    cc -o foo $(objects)
```
### 常见的make出错信息
* No rule to make target ‘target’.Stop
makefile中没有包含创建指定target所需要的规则，而且也没有默认规则可用。
* ‘target’ is up to date
指定的target相关文件没有变化。
* command：Command not found
make找不到命令，通常是因为命令被拼写错误或者不在$PATH路径下。
* makefile:2: *** missing separator. Stop. make
makefile的缩进是用的tab，而不是空格


### 一个比较通用的例子
见附件


## 常用编译方式
在一些开源项目中，会使用 autogen.sh+configure+make 方式进行代码编译。
1. 运行 autogen.sh，生成 configure 脚本；
2. 运行 configure 脚本，检查系统配置；
3. 运行 make 命令，执行代码的编译操作；
4. 运行 make install 命令，安装编译生成的文件。


## gdb
GDB 不仅是一个调试工具，它也是一个学习源码的好工具。用它开跟踪代码的分支和数据的流向，这样走 上几个来回之后，再结合源码，就能够对程序的整体情况“了然于胸”。


gdb 程序名 [corefile]                 --  corefile是可选的，但能增强gdb的调试能力。
Linux默认是不生成corefile的，所以需要在.bashrc文件中添加(开发期间设置打开就行)
ulimit -c unlimited
(修改完.bashrc文件后记得. .bashrc让修改生效)


### 常用命令
```
gdb  -p pid         attach指定pid的程序
run  r 运行程序， 它的后面可以加上程序的参数，程序运行完重新执行的时候也用这个
where 显示出错函数调用栈
bt  打印出错的函数调用栈
list [m,n] m,n是要显示包含错误首次出现位置的起始行和结尾行。不带参数的list将显示附近的10行代码
print 变量，表达式。
    print 'file'::变量，表达式,''是必须的，以便让gdb知道指的是一个文件名。
    print funcname::变量，表达式
whatis 命令可以告诉你变量的类型
ptype 告诉你结构的定义。
break b命令设置断点
    break linenum
    break funcname
    break filename:linenum
    break filename:funcname
info break  查看断点
continue 命令从断点以后继续执行，知道运行到下一个断点。
delete 删除一个断点，若没有参数删除所有，若加断点序号删除指定的
set variable varname = value  改变一个变量的值
step 相当于step into
next 相当于step over
return [value]  停止执行当前函数，将value返回给调用者，相当于step return
enter  gdb记住了最后一个被执行的命令，只要简单的按enter键就可以重复执行最后的命令
退出  quit或者Ctrl+d


pt：查看变量的真实类型，不受 typedef 的影响。
up/down：在函数调用栈里上下移动。
fin：直接运行到函数结束。
i b：查看所有的断点信息。
wh：启动“可视化调试”。这个是我最喜欢的命令，可以把屏幕分成上下两个窗口，上 面显示源码，下面是 GDB 命令输出，不必再用“l”频繁地列出源码了，能够大大提高 调试的效率。


frame x    到第几个函数帧
set print elements 0   使打印的字符串长度不受限制
```
【程序运行参数】
set args 可指定运行时参数。（如：set args 10 20 30 40 50） 
show args 命令可以查看设置好的运行参数。 


### TUI模式
`-tui` 选项进入TUI模式  或者  `layout src`
`Ctrl+X+A`组合键可以在 TUI模式和非TUI模式切换


### 远程调试
```
gdbserver 192.168.1.21:6666 a.out
(gdb) target remote 192.168.1.23:6666
```