## Linux系统编程

### 1 GCC

#### 1.1 GCC工作流程

<img src="image\20220829001.png" alt="202208220929" style="zoom: 67%;" />

#### 1.2 GCC常用参数选项

| Gcc编译选项 |                             说明                             |
| ----------- | :----------------------------------------------------------: |
| -E          | 预处理将#include，#define等进行文件插入及将宏定义替换到代码中；会删除与代码运行无关的注释。 |
| -S          | 将对源程序进行预处理、编译，生成`.s`后缀名的汇编文件，里面包含汇编代码。 |
| -c          | 在编译阶段生成的汇编文件会经过汇编器(as)的处理，生成二进制目标文件`.o` |
|             | 链接阶段是将所有相关的目标文件链接起来，形成一个整体，生成一个可执行文件。无选项链接: 这个命令会把二进制目标文件test.o所需的相关文件链接成一个整体，并在当前文件夹自动生成一个名为a.out的可执行文件。 |
| -I          |                指定include 包含文件的搜索目录                |
| -g          |      在编译的时候，生成调试信息，该程序可以被调试器调试      |
| -D          |                 在程序编译的时候，指定一个宏                 |
| -w          |                      不生成任何警告信息                      |

预编译：这个过程主要的处理操作如下：

（1） 将所有的#define删除，并且展开所有的宏定义

（2） 处理所有的条件预编译指令，如#if、#ifdef

（3） 处理#include预编译指令，将被包含的文件插入到该预编译指令的位置。

（4） 过滤所有的注释

（5） 添加行号和文件名标识。

编译：这个过程主要的处理操作如下：

（1） 词法分析：将源代码的字符序列分割成一系列的记号。

（2） 语法分析：对记号进行语法分析，产生语法树。

（3） 语义分析：判断表达式是否有意义。

（4） 代码优化：

（5） 目标代码生成：生成汇编代码。

（6） 目标代码优化：

汇编：这个过程主要是将汇编代码转变成机器可以执行的指令。

链接：将不同的源文件产生的目标文件进行链接，从而形成一个可以执行的程序。

链接分为静态链接和动态链接。

静态链接，是在链接的时候就已经把要调用的函数或者过程链接到了生成的可执行文件中，就算你在去把静态库删除也不会影响可执行程序的执行；生成的静态链接库，Windows下以.lib为后缀，Linux下以.a为后缀。

而动态链接，是在链接的时候没有把调用的函数代码链接进去，而是在执行的过程中，再去找要链接的函数，生成的可执行文件中没有函数代码，只包含函数的重定位信息，所以当你删除动态库时，可执行程序就不能运行。生成的动态链接库，Windows下以.dll为后缀，Linux下以.so为后缀。

```shell
# 预处理
g++ test.cpp -E -o test.i
# 编译
g++ test.cpp -S -o test.s
# 汇编
g++ test.s -c -o test.o
# 链接
g++ test.o -o test.out
g++ test.o -o test
g++ test.o -o test.exe
```

### 2 MakeFile

#### 2.1 简介

`Makefile` 文件描述了 `Linux` 系统下 `C/C++` 工程的编译规则，它用来自动化编译 `C/C++` 项目。`Makefile` 文件定义了一系列规则，指明了源文件的编译顺序、依赖关系、是否需要重新编译等。

#### 2.2 MakeFile文件命名和规则

编写 `Makefile` 的时可以使用的文件的名称 `GNUMakefile` 、`makefile` 、`Makefile` ，`make` 执行时回去寻找 `Makefile` 文件，找文件的顺序也是这样的。

~~~makefile
targets : prerequisites
    command
#或者
targets : prerequisites; command
    command
~~~

相关说明如下：

- targets：规则的目标，是必须要有的，可以是 Object File（一般称它为中间文件），也可以是可执行文件，还可以是一个标签；
- prerequisites：是我们的依赖文件，要生成 targets 需要的文件或者是目标。可以是多个，用空格隔开，也可以是没有；
- command：make 需要执行的命令（任意的 shell 命令）。可以有多条命令，每一条命令占一行；如果 command 太长, 可以用 \ 作为换行符。

**我们的目标和依赖文件之间要使用冒号分隔开，命令的开始一定要使用 `Tab` 键，不能使用空格键。**

#### 2.3 基本原理

~~~
main: main.o   name.o greeting.o
	g++ main.o name.o greeting.o -o main
main.o: main.cpp
	g++ -c main.cpp -o main.o
name.o: name.cpp
	g++ -c name.cpp -o name.o
greeting.o: greeting.cpp
	g++ -c greeting.cpp -o greeting.o
~~~

默认情况下，`make` 执行的是 `Makefile` 中的第一规则（`Makefile` 中出现的第一个依赖关系），此规则的第一目标称之为“最终目标”或者是“终极目标”。

在我们的例子中，第一个规则就是目标 main 所在的规则。规则描述了 main 的依赖关系，并定义了链接 .o 文件生成目标 main 的命令；make 在执行这个规则所定义的命令之前，首先处理目标 main 的所有的依赖文件（例子中的那些 .o 文件）的更新规则（以这些 .o 文件为目标的规则）。
对这些 .o 文件为目标的规则处理有下列三种情况：

- 目标 .o 文件不存在，使用其描述规则创建它；
- 目标 .o 文件存在，目标 .o 文件所依赖的 “.cpp” 源文件 “.h” 文件中的任何一个比目标 .o 文件“更新”（在上一次 make 之后被修改），则根据规则重新编译生成它；
- 目标 .o 文件存在，目标 .o 文件比它的任何一个依赖文件（“.c” 源文件、“.h” 文件）“更新”（它的依赖文件在上一次 make 之后没有被修改），则什么也不做；

#### 2.4 变量

##### 2.4.1 变量的定义

~~~makefile
变量的名称=值列表
name_list = aa bb cc
~~~

调用变量的时候可以用 `$(name_list)` 或者是 `${name_list}` 来替换

##### 2.4.2 变量赋值

Makefile 的变量的四种基本赋值方式：

- 简单赋值 ( := ) 编程语言中常规理解的赋值方式，只对当前语句的变量有效。
- 递归赋值 ( = ) 赋值语句可能影响多个变量，所有目标变量相关的其他变量都受影响。
- 条件赋值 ( ?=) 如果变量未定义，则使用符号中的值定义变量。如果该变量已经赋值，则该赋值语句无效。
- 追加赋值 ( += ) 原变量用空格隔开的方式追加一个新值。

##### 2.4.3 自动变量

| 自动化变量 |                           **说明**                           |
| :--------: | :----------------------------------------------------------: |
|     $@     | 表示规则的目标文件名。如果目标是一个文档文件（Linux 中，一般成 .a 文件为文档文件，也成为静态的库文件），那么它代表这个文档的文件名。在多目标模式规则中，它代表的是触发规则被执行的文件名。 |
|     $%     |    当目标文件是一个静态库文件时，代表静态库的一个成员名。    |
|     $<     | 规则的第一个依赖的文件名。如果是一个目标文件使用隐含的规则来重建，则它代表由隐含规则加入的第一个依赖文件。 |
|     $?     | 所有比目标文件更新的依赖文件列表，空格分隔。如果目标文件时静态库文件，代表的是库文件（.o 文件）。 |
|     $^     | 代表的是所有依赖文件列表，使用空格分隔。如果目标是静态库文件，它所代表的只能是所有的库成员（.o 文件）名。一个文件可重复的出现在目标的依赖中，变量 `$^`只记录它的第一次引用的情况。就是说变量`$^`会去掉重复的依赖文件。 |
|     $+     | 类似`$^`，但是它保留了依赖文件中重复出现的文件。主要用在程序链接时库的交叉引用场合。 |
|     $*     | 在模式规则和静态模式规则中，代表“茎”。“茎”是目标模式中“%”所代表的部分（当文件名中存在目录时，“茎”也包含目录部分）。 |

#### 2.5 语法

~~~makefile
#判断参数是否不相等
ifeq (first, second)
ifneq `first` 'second'
#判断是否有值
ifdef VARIABLE_NAME
ifndef VARIABLE_NAME
#命令包有点像是个函数, 将连续的相同的命令合成一条, 减少Makefile中的代码量
define <command-name>
command
...
endef
~~~

Makefile伪目标并不会创建目标文件，只是想去执行这个目标下面的命令

~~~
clean:
    rm -rf *.o test
~~~

当工作目录下不存在以 clean 命令的文件时，在 shell 中输入 make clean 命令，命令 rm -rf *.o test 总会被执行 ，这也是我们期望的结果。

如果当前目录下存在文件名为 clean 的文件时情况就会不一样了，当我们在 shell 中执行命令 make clean，由于这个规则没有依赖文件，所以目标被认为是最新的而不去执行规则所定义的命令。因此命令 rm 将不会被执行。

为了解决这个问题，删除 clean 文件或者是在 Makefile 中将目标 clean 声明为伪目标。

~~~makefile
.PHONY:clean
~~~

#### 2.6 函数

##### 2.6.1 字符串函数

~~~makefile
#函数调用，参数和函数名之间使用空格分开。
$(<function> <arguments>)
${<function> <arguments>}

#模式字符串替换函数 patsubst
#函数功能是查找text中的单词是否符合模式pattern，如果匹配的话，则用replacement替换。返回值为替换后的新字符串。
$(patsubst <pattern>,<replacement>,<text>)

#字符串替换函数 subst
$(subst <from>,<to>,<text>)

#去空格函数
#函数的功能是去掉字符串的开头和结尾的字符串，并且将其中的多个连续的空格合并成为一个空格。返回值为去掉空格后的字符串
$(strip <string>)

#查找字符串函数 findstring
$(findstring <find>,<in>)

#过滤函数 filter
$(filter <pattern>,<text>)
$(filter-out <pattern>,<text>)

#排序函数 sort
$(sort <list>)

#取单词函数 word
$(word <n>,<text>)
~~~

##### 2.6.2 文件名操作函数

~~~makefile
#从文件名序列names中取出目录部分
$(dir <names>)
$(notdir <names>)

#取后缀名函数
$(suffix <names>)
$(basename <names>)

#添加后缀名函数
$(addsuffix <suffix>,<names>)
$(addperfix <prefix>,<names>)

#链接函数join，把list2中的单词对应的拼接到list1的后面。
$(join <list1>,<list2>)

#获取匹配模式文件名函数
$(wildcard PATTERN1...)
~~~

##### 2.6.3 其它函数

~~~makefile
#把参数<list>中的单词逐一取出放到参数<var>所指定的变量中，然后再执行<text>所包含的表达式
$(foreach <var>,<list>,<text>)

#if 条件选择函数
$(if <condition>,<then-part>)
$(if <condition>,<then-part>,<else-part>)

#expression参数中的变量$(1)、$(2)、$(3)等，会被参数parm1，parm2，parm3依次取代。而expression的返回值就是call函数的返回值
$(call <expression>,<parm1>,<parm2>,<parm3>,...)

#告诉你这个变量是哪里来的
$(origin <variable>)
~~~

**注意： `variable` 是变量的名字，不应该是引用，所以最好不要在 `variable` 中使用 `$` 字符。**

下面是 origin 函数返回值：

- **undefined**：如果 <variable> 从来没有定义过，函数将返回这个值。
- **default**：如果 <variable> 是一个默认的定义，比如说CC这个变量。
- **environment**：如果 <variable> 是一个环境变量并且当 Makefile 被执行的时候，-e参数没有被打开。
- **file**：如果 <variable> 这个变量被定义在 Makefile 中，将会返回这个值。
- **command** line：如果 <variable> 这个变量是被命令执行的，将会被返回。
- **override**：如果 <variable> 是被 override 指示符重新定义的。
- **automatic**：如果 <variable> 是一个命令运行中的自动化变量。

### 3 GDB

`gdb`（`GNU debugger`）是 `UNIX/Linux` 系统中强大的调试工具，它能够调试软件并分析软件的执行过程，帮助我们调查研究程序的正确行为，还能用来分析程序崩溃的原因等。

`gdb` 支持多种语言，可以支持 `C/C++` 、`Go`、`Java`、`Objective-C` 等。

#### 3.1 gdb调试执行

**在编译程序时，使用 `gcc` 或者 `g++` 时一定要加上 `-g` 选项**，如

```shell
gcc -g -o hello hello.c
```

以便调试程序含有调试符号信息，从而能够正常调试程序。否则则会出现如下提示，导致不能调试。

除了不加 `-g` 选项，也可以使用 `Linux` 的 `strip` 命令移除掉某个程序中的调试信息`strip hello`；使用 `strip` 命令之后，程序明显变小了，我们通常会在程序测试没问题以后，将其发布到生产环境或者正式环境中，因此生成不带调试符号信息的程序，以减小程序体积或提高程序执行效率。

**在实际生成调试程序时，一般不仅要加上 -g 选项，也建议关闭编译器的程序优化选项。这样做的目的是为了调试的时候，符号文件显示的调试变量等能与源代码完全对应起来。**

`gdb filename`直接调试目标程序

`gdb attach pid` 附加进程

`gdb filename corename` 调试 `core` 文件



当用 `gdb attach` 上目标进程后，调试器会暂停下来，此时可以在 `gdb` 中输入相关的命令，比如设置断点等，再继续运行程序，此时需要在 `gdb` 中输入命令 `c` 继续运行，程序才能恢复为正常状态。



程序在崩溃的时候有 `core` 文件产生，就可以使用这个 `core` 文件来定位崩溃的原因。使用 `ulimit -c` 命令来查看系统是否开启了 `core` 文件定位崩溃。使用 `ulimit 选项名 设置值` 来修改。可以将 `core` 文件生成改成具体某个值（是临时的）， `core file size` 选项-c表示生成 `core` 文件

~~~shell
ulimit -c unlimited
~~~

#### 3.2 置断点

~~~shell
#在代码的某一行设置断点
(gdb) break 文件名:行号
#为函数设置断点
(gdb) break 函数名
#使用正则表达式设置函数断点
(gdb) rb 正则表达式
# 通过偏移量设置断点
(gdb) b +偏移量  
(gdb) b -偏移量
#设置条件断点
(gdb) b 断点 条件
(gdb) b demo.cpp:8 if i==900
#在指令地址上设置断点
(gdb) b *指令地址
(gdb) p fun_test
(gdb) b *0x400a0b
# 设置临时断点 
(gdb) tb 断点
#启用禁用断点
(gdb) disable   断点编号（可以是范围）
(gdb) enable   断点编号
#启用禁用一次断点
(gdb) enable once 断点编号
#启用断点并删除
(gdb) enable delete 断点编号
#启用断点并命中N次
(gdb) enable count 数量 断点编号
#忽略断点前N次命中
(gdb) ignore 断点编号 次数
#删除断点
(gdb) delete 断点编号
(gdb) clear 函数名
(gdb) clear 行号
#继续运行并跳过当前断点 N 次
(gdb) continue 次数
#继续运行直到当前函数执行完成
(gdb) finish
~~~

#### 3.3 查看当前参数

~~~shell
#查看变量
(gdb) p/print c 
(gdb) print 变量名=值
#断点
(gdb) info b #断点信息
#参数
(gdb) info/i args
#gdb内嵌函数，比如一些c函数sizeof、strcmp
(gdb) p sizeof(int)
#查看结构体/类的值
(gdb) p *new_node
#print格式
(gdb) set print null-stop
(gdb) set print pretty
(gdb) set print array on
#自动显示变量的值，如果 display 命令后面跟多个变量名，则必须要求这些变量的类型相同（比如都是整型变量）。如果长度不相同，则需要分开使用。
(gdb) display 变量名
(gdb) info display
undisplay 编号
~~~

#### 3.4 监控内存

watch命令的使用方式是

```shell
(gdb) watch 变量名或内存地址
#当设置的观察点是一个局部变量时，局部变量无效后，观察点也会失效。在观察点失效时 `GDB` 可能会提示如下信息：
Watchpoint 2 deleted because the program has left the block in which its expression is valid.
```

查看内存使用 `x` 命令查看各个变量的内存信息

~~~
(gdb) x /选项 地址
~~~

#### 3.5 栈回溯

查看栈回溯信息的命令是 `backtrace`，通过 `frame 栈帧号` 的方式来切换栈帧

~~~shell
#执行命令来查看指定数量的栈帧
(gdb) bt 栈帧数量
#切换栈帧
(gdb) frame 2 
(gdb) f 2
(gdb) f 帧地址
(gdb) up/down 
#查看当前帧的所有局部变量的值
(gdb) info locals
~~~

#### 3.6 gdb命令大全

~~~shell
(gdb) list #查看代码
(gdb) list 行号
(gdb) list 函数名
(gdb) s #单步执行
(gdb) next # 执行下一行语句
(gdb) c # continue从当前位置连续执行
(gdb) detach #让程序与 GDB 调试器分离
(gdb) r #程序真正的运行起来
(gdb) q #停止运行
(gdb) set args a b c d #添加参数，如果单个命令行参数之间含有空格，可以使用引号将参数包裹起来。
(gdb) show args #查看命令行参数是否设置成功

~~~

### 4 静态库和动态库

#### 4.1 什么是库

库是写好的现有的，成熟的，可以复用的代码。**现实中每个程序都要依赖很多基础的底层库，不可能每个人的代码都从零开始，因此库的存在意义非同寻常**。

本质上来说库是一种可执行代码的二进制形式，可以被操作系统载入内存执行。库有两种：静态库（.a、.lib）和动态库（.so、.dll）。

![](image\20220831001.png)

#### 4.2 静态库

之所以成为【静态库】，是因为在链接阶段，会将汇编生成的目标文件.o与引用到的库一起链接打包到可执行文件中。因此对应的链接方式称为静态链接。

- 静态库对函数库的链接是放在编译时期完成的。
- 程序在运行时与函数库再无瓜葛，移植方便。
- 浪费空间和资源，因为所有相关的目标文件与牵涉到的函数库被链接合成一个可执行文件。

**创建静态库**，Linux静态库命名规范，必须是"lib[your_library_name].a"

~~~shell
#1.将代码文件编译成目标文件.o
g++ -c demo.cpp
#2.通过ar工具将目标文件打包成.a静态库文件
ar -crv libdemo.a demo.o
#3.添加头文件
#include "demo.h"
#4.在编译的时候，指定静态库的搜索路径（-L选项）、指定静态库名（不需要lib前缀和.a后缀，-l选项）
#-L：表示要连接的库所在目录
#-l：指定链接时需要的动态库
~~~

#### 4.3 动态库

为了解决静态库所带来的问题，我们引入了动态库。动态库在编译时期不会被链接到目标代码中，而是在程序运行的时候才被载入。不同的应用程序如果调用相同的库，那么内存中只需要有一份共享库的实例即可，规避了空间的浪费问题。同时因为是在程序运行时被载入的，也解决了程序的更新问题，不需要重新编译。
Linux动态库命名规范，必须是"lib[your_library_name].so"

~~~shell
#1.将代码文件编译成目标文件.
#-fpic创建与地址无关的编译程序（pic，position independent code），是为了能够在多个应用程序间共享。
g++ -fpic -c demo.cpp
#2.-shared指定生成动态链接库。
g++ -shared -o libdemo.so demo.o
#3.添加头文件
#include "demo.h"
#4.在编译的时候，指定静态库的搜索路径（-L选项）、指定静态库名（不需要lib前缀和.a后缀，-l选项）
#-L：表示要连接的库所在目录
#-l：指定链接时需要的动态库
~~~

如何让系统能够定位共享库文件：

- 如果安装在/lib或者/usr/lib下，那么ld默认能够找到，无需其他操作。
- 如果安装在其他目录，需要将其添加到/etc/ld.so.cache文件中，步骤如下：
  - 编辑/etc/ld.so.conf文件，加入库文件所在目录的路径
  - 运行ldconfig ，该命令会重建/etc/ld.so.cache文件

#### 4.4 区别

**主要在于代码被载入的时刻不同**。静态库是在编译链接期被链接到可执行文件中，运行时不在需要该静态库，因此体积比较大。动态库是在程序运行时才被载入，因此程序运行时候依赖动态库，体积比较小。动态库是为了解决静态库的缺点而产生的。

### 5 文件IO

系统IO：Unix/Linux下的系统文件IO，即文件访问机制不经过操作系统内核的缓存，数据直接在磁盘和应用程序地址空间进行传输。

标准IO：带缓存的IO，又称为标准IO(C标准库中提供了标准IO库，即stdio)，它实现了跨平台的用户缓存解决方案。

文件I/O是操作系统封装了一系列open、close、write、read等API函数构成的一套用来读、写文件的接口供应用程序使用，通过这些接口可以实现对文件的读写操作，但是效率并不是最高的。

文件I/O是采用系统直接调用的方式，因此当使用这些接口对文件进行操作时，就会立刻触发系统调用过程，即向系统内核发出请求之后，系统内核会收到执行相关代码处理的请求，决定是否将操作硬件资源或返回结果给应用程序。

#### 5.1 标准C库IO函数

<img src=".\image\20220902002.png" alt="20220902002" style="zoom: 80%;" />

应用层C语言库函数提供了一些用来做文件读写的函数列表，叫标准IO。标准IO有一系列的C库函数构成（fopen，fclose，fwrite，fread），这些标准IO函数其实是由文件IO封装而来的（fopen内部还是调用了open）；我们通过fwrite写入的内容不是直接进入内核中的buf，而是先进入应用层标准IO库自己维护的buf中，然后标准IO库自己根据操作系统单次write的最佳count来选择好的时机来完成write到内核中的buf中。因此，标准I/O封装了底层系统调用更多的调用函数接口。

#### 5.2 标准C库IO和Linux系统IO的关系

<img src=".\image\20220902001.png" alt="20220902001" style="zoom:50%;" />

- 缓冲区：标准I/O函数接口在对文件进行操作时，首先操作缓存区，等待缓存区满足一定的条件时，然后再去执行系统调用，真正实现对文件的操作。而文件I/O不操作任何缓存区，直接执行系统调用。
- 系统开销：使用标准I/O可以减少系统调用的次数，提高系统效率。例如，将数据写入文件中，每次写入一个字符。采用文件I/O的函数接口，每调用一次函数写入字符就会产生一次系统调用。而执行系统调用时，Linux必须从用户态切换到内核态，处理相应的请求，然后再返回到用户态，如果频繁地执行系统调用会增加系统的开销。
- 执行效率：采用标准I/O的函数接口，每调用一次函数写入字符，并不着急将字符写入文件，而是放到缓存区保存，之后每一次写入字符都放到缓存区保存。直到缓存区满足刷新的条件（如写满）时，再一并将缓存区中的数据写入文件，执行一次系统调用完成此过程，这样便很大程度地减少了系统的调用次数，提高了执行效率。

#### 5.3 虚拟地址空间

以32位机为例，2的32次方为4G

<img src="image\20220902003.png" alt="20220902003" style="zoom:80%;" />

<img src="image\20220902004.png" alt="20220902004" style="zoom:80%;" />

#### 5.4 文件描述符

缩写fd（file descriptor）是0-1023的数字，表示文件。编号0, 1, 2三个文件的含义标准输入，标准输出，错误（stdin，stdout，stderr）

本质是内核中文件和进程相关联的数据结构中的指针数组的下标

文件描述符表默认大小: 1024，每个进程启动之后, 都有一个文件描述符表，**所以每个进程默认能打开的文件个数: 1024**
前三个文件文件描述符是默认被使用了的: - 标准输入 -> 0, 标准输出 -> 1, 标准错误 -> 2

**除去被占用的每个进程默认能打开的文件个数: 1021**

~~~
//open
int open(const char *pathname, int flags);   //不创建文件
int open(const char *pathname, int flags, mode_t mode); // 创建文件，不能创建设备文件
//成功时返回文件描述符；出错时返回EOF
~~~

#### 5.5 Linux系统IO函数

 error是一个全局变量,定义在头文件error.h中。它总是会记录系统最后一次出现错误的值(这个值是一个int类型)。每一个int类型都定义一个字符串描述的字符串。

**void perror(const char*s)**，头文件：stdio.h。根据error对应的字符串加上直接的描述输出到标准错误

~~~cpp
int fd = open("a.txt",O_RDONLY);
perror("file");
// file: No such file or directory

#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
int open(const char *pathname, int flags);
int open(const char* pathname,int flags,mode_t mask);
//pathname:要创建的文件路径
//flags:对文件的操作权限, 必选项-->O_RDONLY  O_RDWR  O_WRONLY这三个互斥 可选项-->O_CREATE
//mode:表示创建出来的新的文件的操作权限(通常用八进制数表示，777)
//返回值：-1表示错误

#include <unistd.h>
size_t read(int fd,void *buf,size_t count);
size_t write(int fd,void *buf,size_t count);
//buf暂存数据的buf(主调函数分配内存，通常为char *)， count buf的大小(通常传sizeof(buf))
//返回值:-1 ==>读文件失败，0 ==>数据读完了(通常我们通过0来判断对方是否已经传递完数据)，>0 ==>读取的字节数

#include <unistd.h>
#include <fcntl.h>
int fcntl(int fd,int cmd,.../*arg*/);
//cmd 表示对文件描述符进行如何操作，arg 可变参数
//cmd参数
//F_DUPFD  -->  复制的文件一个文件描述符，得到一个新的文件描述符(复制的意思是指从文件描述符表中找一个空闲的最小的指向同一个文件,而不是将fd的值复制一份)
//F_GETFL   -->  获取指定文件描述符文件状态flag
//F_SETFL   -->   设置文件描述符状态flag
~~~

#### 5.6 文件属性操作函数

~~~cpp
//修改程序中的 mask掩码
#include <sys/types.h>
#include <sys/stat.h>
mode_t  umask(mode_t mask);

//判断某个文件是否具有某个权限，判断文件是否存在
int access(const char *pathname,int mode);
// mode: R_OK  W_OK  X_OK  F_OK

//减缩或者扩展文件尺寸
int truncate(const char*path,off_t length);

//移动文件读写指针
off_t lseek(int fd,off_t offset,int whence);                
//offset:需要偏移的大小
//whence:从什么地方开始偏移
//返回值:文件读写指针的位置
//SEEK_SET 头
//SEEK_END 尾
//SEEK_CUR 当前位置
~~~

#### 5.7 目录操作函数

~~~cpp
#include <sys/types.h>
#include <sys/stat.h>
int mkdir(const char *pathname,mode_t mode);

#include <unistd.h>
int rmdir(const char *pathname);

#include <stdio.h>
int rename(const char *oldpath, const char *newpath);

#include <unistd.h>
int chdir(const char *path); //修改进程的工作目录

#include <unistd.h>
char *getcwd(char *buf, size_t size);
~~~

#### 5.8 目录遍历函数

~~~cpp
#include <sys/types.h>
#include <dirent.h>
DIR *opendir(const char *name);
//DIR * 类型：理解为目录流信息 错误返回NULL

#include <dirent.h>
struct dirent *readdir(DIR *dirp);
//struct dirent，代表读取到的文件的信息 读取到了末尾或者失败了，返回NULL

#include <sys/types.h>
#include <dirent.h>
int closedir(DIR *dirp);
~~~

#### 5.9 direct结构体和d_type

~~~cpp
struct dirent {
    ino_t          d_ino;       // 此目录进入点的inode
    off_t          d_off;       // 目录文件开头至此目录进入点的位移
    unsigned short d_reclen;    // d_name的实际长度
    unsigned char  d_type;      // d_name所指的文件类型 
    char           d_name[256]; // 文件名
};
~~~

<img src="image\20220913001.png" style="zoom:50%;" />

#### 5.10 dup、dup2函数

~~~cpp
//复制一个文件描述符，返回值返回新的文件描述符
int dup(int oldfd);
//重定义一个文件描述符
int dup2(int oldfd,int newfd);
//oldfd 指向 a.txt    newfd指向 b.txt
//调用完之后，oldfd和newfd都指向 a.txt
~~~

## 进程

### 1 基本概念

程序

进程

单道多程序设计

并行并发

进程控制块（PCB）

### 2 进程状态

### 3 进程相关指令

#### 3.1 查看进程

#### 3.2 杀死进程

### 4 进程相关函数

每个进程都由进程号来标识，其类型为 pid_t（整型），进程号的范围：0～32767。进程号总是唯一的，但可以重用。当一个进程终止后，其进程号就可以再次使用。

任何进程（除 init 进程）都是由另一个进程创建，该进程称为被创建进程的父进程，对应的进程号称为父进程号（PPID）。

进程组是一个或多个进程的集合。他们之间相互关联，进程组可以接收同一终端的各种信号，关联的进程有一个进程组号（PGID）。默认情况下，当前的进程号会当做当前的进程组号。

~~~cpp
 pid_t getpid(void);
 pid_t getppid(void);
 pid_t getpgid(pid_t pid);
~~~

### 5 进程创建

Linux 的 `fork()` 使用是通过`写时拷贝` (copy- on-write) 实现。内核此时并不复制整个进程的地址空间，而是让父子进程共享同一个地址空间。只用在需要写入的时候才会复制地址空间，从而使各个进行拥有各自的地址空间。

~~~cpp
#include <sys/types.h>
#include <unistd.h>

pid_t fork(void);
//用于创建子进程。
//fork()的返回值会返回两次。一次是在父进程中，一次是在子进程中。
//在父进程中返回创建的子进程的ID，在子进程中返回0，在父进程中返回-1，表示创建子进程失败，并且设置errno
~~~

父子进程共同点：某些状态下，子进程刚被创建出来，还没有执行任何的写数据的操作
   - 用户区的数据
   - 文件描述符表

父子进程对变量是不是共享的？

- 刚开始的时候，是一样的，共享的。如果修改了数据，不共享了。
- 读时共享（子进程被创建，两个进程没有做任何的写的操作），写时拷贝。

### 6 exec函数族

exec函数族的作用是根据指定的文件名找到可执行文件，并用它来取代调用进程的内容，也就是在调用进程内部执行一个可执行文件。

~~~cpp
int execl(const char *path, const char *arg, .../* (char *) NULL */);
int execlp(const char *file, const char *arg, ... /* (char *) NULL */);
int execle(const char *path, const char *arg, .../*, (char *) NULL, char *const envp[] */);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[], char *const envp[]);
int execve(const char *filename, char *const argv[], char *const envp[]);

//path:需要指定的执行的文件的路径或者名称，推荐使用绝对路径
//arg:是执行可执行文件所需要的参数列表，第一个参数一般没有什么作用，为了方便，一般写的是执行的程序的名称，从第二个参数开始往后，就是程序执行所需要的的参数列表。参数最后需要以NULL结束（哨兵）
//只有当调用失败，才会有返回值，返回-1，并且设置errno
//argv是需要的参数的一个字符串数组，char * argv[] = {"ps", "aux", NULL};
//char * envp[] = {"/home/nowcoder", "/home/bbb", "/home/aaa"};

l(list) 参数地址列表，以空指针结尾
v(vector) 存有各参数地址的指针数组的地址
p(path) 按 PATH 环境变量指定的目录搜索可执行文件
e(environment) 存有环境变量字符串地址的指针数组的地址
~~~

### 7 进程控制

#### 7.1 进程退出

~~~cpp
#include <stdlib.h>
void exit(int status);

#include <unistd.h>
void _exit(int status);

status参数：是进程退出时的一个状态信息。父进程回收子进程资源的时候可以获取到。
~~~

<img src="image\20220913002.png" alt="20220913002" style="zoom: 50%;" />

#### 7.2 孤儿进程

父进程运行结束，但子进程还在运行（未运行结束），这样的子进程就称为孤儿进程（Orphan Process）。

每当出现一个孤儿进程的时候，内核就把孤儿进程的父进程设置为 init ，而 init 进程会循环地 wait() 它的已经退出的子进程。

这样，当一个孤儿进程结束了其生命周期的时候，init 进程就会处理它的一切善后工作。

因此孤儿进程并不会有什么危害。

#### 7.3 僵尸进程

每个进程结束之后, 都会释放自己地址空间中的用户区数据，内核区的 PCB 没有办法自己释放掉，需要父进程去释放。

进程终止时，父进程尚未回收，子进程残留资源（PCB）存放于内核中，变成僵尸进程。

僵尸进程不能被 kill -9 杀死，这样就会导致一个问题，如果父进程不调用 wait() 或 waitpid() 的话，那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵尸进程，将因为没有可用的进程号而导致系统不能产生新的进程，此即为僵尸进程的危害，应当避免。

#### 7.4 进程回收

在每个进程退出的时候，内核释放该进程所有的资源、包括打开的文件、占用的内
存等。但是仍然为其保留一定的信息，这些信息主要主要指进程控制块PCB的信息（包括进程号、退出状态、运行时间等）。

父进程可以通过调用 wait 或 waitpid 得到它的退出状态，同时彻底清除掉这个进程。

wait() 和 waitpid() 函数的功能一样，区别在于，wait() 函数会阻塞，waitpid() 可以设置不阻塞，waitpid() 还可以指定等待哪个子进程结束。
注意：一次 wait 或 waitpid 调用只能清理一个子进程，清理多个子进程应使用循环。

~~~cpp
#include <sys/types.h>
#include <sys/wait.h>
pid_t wait(int *wstatus);
        功能：等待任意一个子进程结束，如果任意一个子进程结束了，此函数会回收子进程的资源。
        参数：int *wstatus
            进程退出时的状态信息，传入的是一个int类型的地址，传出参数。
        返回值：
            - 成功：返回被回收的子进程的id
            - 失败：-1 (所有的子进程都结束，调用函数失败)

退出信息相关宏函数：
  WIFEXITED(status) 非0，进程正常退出
  WEXITSTATUS(status) 如果上宏为真，获取进程退出的状态（exit的参数）
  WIFSIGNALED(status) 非0，进程异常终止
  WTERMSIG(status) 如果上宏为真，获取使进程终止的信号编号
  WIFSTOPPED(status) 非0，进程处于暂停状态
  WSTOPSIG(status) 如果上宏为真，获取使进程暂停的信号的编号
  WIFCONTINUED(status) 非0，进程暂停后已经继续运行


~~~

调用 wait 函数的进程会被挂起（阻塞），直到它的一个子进程退出或者收到一个不能被忽略的信号时才被唤醒（相当于继续往下执行）。

如果没有子进程了，函数立刻返回，返回-1；如果子进程都已经结束了，也会立即返回，返回-1。

~~~cpp
#include <sys/types.h>
#include <sys/wait.h>
pid_t waitpid(pid_t pid, int *wstatus, int options);
        功能：回收指定进程号的子进程，可以设置是否阻塞。
        参数：
            - pid:
                pid > 0 : 某个子进程的pid
                pid = 0 : 回收当前进程组的所有子进程    
                pid = -1 : 回收所有的子进程，相当于 wait()  （最常用）
                pid < -1 : 某个进程组的组id的绝对值，回收指定进程组中的子进程
            - options：设置阻塞或者非阻塞
                0 : 阻塞
                WNOHANG : 非阻塞
            - 返回值：
                > 0 : 返回子进程的id
                = 0 : options=WNOHANG, 表示还有子进程或者
                = -1 ：错误，或者没有子进程了
~~~

### 8 进程通信

#### 8.1 Linux进程通信方式

<img src="image\20220913003.png" alt="20220913003" style="zoom: 67%;" />

管道其实是一个在内核内存中维护的缓冲器，这个缓冲器的存储能力是有限的，不同的操作系统大小不一定相同。

管道拥有文件的特质：读操作、写操作，匿名管道没有文件实体，有名管道有文件实体，但不存储数据。可以按照操作文件的方式对管道进行操作。

一个管道是一个字节流，使用管道时不存在消息或者消息边界的概念，从管道读取数据的进程可以读取任意大小的数据块，而不管写入进程写入管道的数据块的大小是多少。

通过管道传递的数据是顺序的，从管道中读取出来的字节的顺序和它们被写入管道的顺序是完全一样的。

管道中数据的传递方向是单向的，一端用于写入，一端用于读取，管道是半双工的。

<img src="image\20220913004.png" alt="20220913004" style="zoom: 50%;" />

从管道读数据是一次性操作，数据一旦被读走，它就从管道中被抛弃，释放空间以便写更多的数据，在管道中无法使用 `lseek()` 来随机的访问数据。

匿名管道只能在具有公共祖先的进程（父进程与子进程，或者两个兄弟进程，具有亲缘关系）之间使用。

#### 8.2 匿名管道

~~~cpp
//创建匿名管道
#include <unistd.h>
int pipe(int pipefd[2]);

//查看管道缓冲大小的命令
ulimit –a

//查看管道缓冲大小的函数
#include <unistd.h>
long fpathconf(int fd, int name);

#include <unistd.h>
int pipe(int pipefd[2]);
        功能：创建一个匿名管道，用来进程间通信。
        参数：int pipefd[2] 这个数组是一个传出参数。
            pipefd[0] 对应的是管道的读端
            pipefd[1] 对应的是管道的写端
        返回值：
            成功 0
            失败 -1
~~~

管道默认是阻塞的：如果管道中没有数据，read阻塞，如果管道满了，write阻塞

注意：匿名管道只能用于具有关系的进程之间的通信（父子进程，兄弟进程）



使用管道时，需要注意以下几种特殊的情况（假设都是阻塞I/O操作）

- 所有的指向管道写端的文件描述符都关闭了（管道写端引用计数为0），有进程从管道的读端读数据，那么管道中剩余的数据被读取以后，再次read会返回0，就像读到文件末尾一样。
- 如果有指向管道写端的文件描述符没有关闭（管道的写端引用计数大于0），而持有管道写端的进程也没有往管道中写数据，这个时候有进程从管道中读取数据，那么管道中剩余的数据被读取后，再次read会阻塞，直到管道中有数据可以读了才读取数据并返回。
- 如果所有指向管道读端的文件描述符都关闭了（管道的读端引用计数为0），这个时候有进程向管道中写数据，那么该进程会收到一个信号 SIGPIPE, 通常会导致进程异常终止。
- 如果有指向管道读端的文件描述符没有关闭（管道的读端引用计数大于0），而持有管道读端的进程也没有从管道中读数据，这时有进程向管道中写数据，那么在管道被写满的时候再次write会阻塞，直到管道中有空位置才能再次写入数据并返回。

#### 8.3 命名管道

匿名管道，由于没有名字，只能用于亲缘关系的进程间通信。为了克服这个缺点，提出了有名管道（FIFO），也叫命名管道、FIFO文件。

有名管道（FIFO）不同于匿名管道之处在于它提供了一个路径名与之关联，以 FIFO 的文件形式存在于文件系统中，并且其打开方式与打开一个普通文件是一样的，这样即使与 FIFO 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过 FIFO 相互通信，因此，通过 FIFO 不相关的进程也能交换数据。

一旦打开了 FIFO，就能在它上面使用与操作匿名管道和其他文件的系统调用一样的I/O系统调用了（如read()、write()和close()）。

与管道一样，FIFO 也有一个写入端和读取端，并且从管道中读取数据的顺序与写入的顺序是一样的。FIFO 的名称也由此而来：先入先出。

有名管道（FIFO)和匿名管道（pipe）有一些特点是相同的，其区别在于：

- FIFO 在文件系统中作为一个特殊文件存在，但 FIFO 中的内容却存放在内存中；
- 当使用 FIFO 的进程退出后，FIFO 文件将继续保存在文件系统中以便以后使用；
- FIFO 有名字，不相关的进程可以通过打开有名管道进行通信。

~~~cpp
#include <sys/types.h>
#include <sys/stat.h>
int mkfifo(const char *pathname, mode_t mode);
    参数：
        - pathname: 管道名称的路径
        - mode: 文件的权限 和 open 的 mode 是一样的
                是一个八进制的数
    返回值：成功返回0，失败返回-1，并设置错误号
~~~

#### 8.4 内存映射

`内存映射`（Memory-mapped I/O）是将磁盘文件的数据映射到内存，用户通过修改内存就能修改磁盘文件。

<img src="image\20220913005.png" alt="20220913005" style="zoom: 80%;" />

~~~cpp
#include <sys/mman.h>
void *mmap(void *addr, size_t length, int prot, int flags,int fd, off_t offset);
        - 功能：将一个文件或者设备的数据映射到内存中
        - 参数：
            - void *addr: NULL, 由内核指定
            - length : 要映射的数据的长度，这个值不能为0。建议使用文件的长度。
                    获取文件的长度：stat lseek
            - prot : 对申请的内存映射区的操作权限
                -PROT_EXEC ：可执行的权限
                -PROT_READ ：读权限
                -PROT_WRITE ：写权限
                -PROT_NONE ：没有权限
                要操作映射内存，必须要有读的权限。
                PROT_READ、PROT_READ|PROT_WRITE
            - flags :
                - MAP_SHARED : 映射区的数据会自动和磁盘文件进行同步，进程间通信，必须要设置这个选项
                - MAP_PRIVATE ：不同步，内存映射区的数据改变了，对原来的文件不会修改，会重新创建一个新的文件。（copy on write）
            - fd: 需要映射的那个文件的文件描述符
                - 通过open得到，open的是一个磁盘文件
                - 注意：文件的大小不能为0，open指定的权限不能和prot参数有冲突。
                    prot: PROT_READ                open:只读/读写 
                    prot: PROT_READ | PROT_WRITE   open:读写
            - offset：偏移量，一般不用。必须指定的是4k的整数倍，0表示不偏移。
        - 返回值：
            成功返回创建的内存的首地址
            失败返回MAP_FAILED，(void *) -1

int munmap(void *addr, size_t length);
        - 功能：释放内存映射
        - 参数：
            - addr : 要释放的内存的首地址
            - length : 要释放的内存的大小，要和mmap函数中的length参数的值一样。
~~~

使用内存映射实现进程间通信：

- 有关系的进程（父子进程）
  - 还没有子进程的时候，通过唯一的父进程，先创建内存映射区
  - 有了内存映射区以后，创建子进程
  - 父子进程共享创建的内存映射区

- 没有关系的进程间通信
  - 准备一个大小不是0的磁盘文件
  - 进程1 通过磁盘文件创建内存映射区，得到一个操作这块内存的指针
  - 进程2 通过磁盘文件创建内存映射区，得到一个操作这块内存的指针
  - 使用内存映射区通信

#### 8.5 信号

#### 8.6 共享内存

## 线程

### 1 概念

#### 1.1 简介

#### 1.2 进程线程区别

#### 1.3 线程之间共享和非共享资源

#### 1.4 NPTL

### 2 线程操作函数

### 3 线程同步

#### 3.1 简介

#### 3.2 互斥量

#### 3.3 死锁

#### 3.4 读写锁

#### 3.5 生产者消费者

#### 3.6 条件变量

#### 3.7 信号量

## 网络编程

### 0 IO多路复用epoll