title: Makefile初探
date: 2015-08-14 08:42:29
tags: Linux
---
使用VS、Xcode以及Eclipse等集成开发工具的时候，一般程序员往往是不知道内部的原理，只专注于代码本身。编译、链接、加载等过程的实现机制对一个专业的程序员来说，是一种基本素养。特别是在Linux和Unix环境开发，很难离不开Makefile。下面我们从What、Why、How三个方面介绍。

# What is Makefile
```
在软件开发中，make是一个工具程序（Utility software），经由读取叫做“makefile”的文件，自动化建构软件。它是一种转化文件形式的工具，转换的目标称为“target”；与此同时，它也检查文件的依赖关系，如果需要的话，它会调用一些外部软件来完成任务。它的依赖关系检查系统非常简单，主要根据依赖文件的修改时间进行判断。大多数情况下，它被用来编译源代码，生成结果代码，然后把结果代码连接起来生成可执行文件或者库文件。它使用叫做“makefile”的文件来确定一个target文件的依赖关系，然后把生成这个target的相关命令传给shell去执行。
```
摘自[维基百科](https://zh.wikipedia.org/wiki/Make)
<!-- more -->
# Why to use Makefile

- 使用Makefile最大的好处就是`自动化编译`，只需一条命令对整个工程进行编译，极大地提高开发效率。还可以进行自动化测试等功能的定制。  
- 了解底层，为什么include就可以帮助我们导入头文件，为什么extern关键字就可以引用外部变量，\*.c和\*.h的文件各自之间又是怎么联系在一起。这些Makefile都可以给你答案。
- 为了提升逼格，没有什么比敲一句命令让满屏幕出现高大上的信息更有逼格的事情了。

# How to use Makefile
扯了这么多，那么该如何玩这个看似高大上的玩具？  
对于我来说有逼格且简单的东西就是玩具。没错，Makefile很简单！

## 从Hello World开始
新建一个Makefile文件，填写如下内容：

```
all:
    echo "Hello World"

```

注意echo前面必须是Tab，不能用空格代替。对于JS这类习惯空格代替Tab的程序员可能会遇到Makefile的第一个坑。这个错误往往很难用肉眼发现。  

第一个重要概念-目标（Target），也就是代码中的all。目标写在冒号的左边，echo "Hello World"是目标的命令。  
编辑完成后，在终端中输入make all命令，此时会输出：

```
echo "Hello World"
Hello World
```

第一行显示的是我们在Makefile中的命令，第二行是运行命令得到的结果。

也可以用make all命令达到同样效果，告诉make工具，要生成目标all。  
此时我们修改代码为：

```
all:
    @echo "Hello World"
print:
    @echo "Some Message"	
```

此时执行命令make print就会将`Some Message`打印在终端上。注意@符号，该符号作用就是禁止该命令打印在终端上。  

从这个简单的HelloWorld输出中，我们可以学得这几点知识：

- 一个Makefile可以有多个目标
- 执行make命令，我们要附带目标名；当没有指明目标，make会将Makefile中第一个目标作为默认目标。
- 当make找到目标后，就找到对应的规则，运行规则中的命令来构建目标

## 依赖关系
我们继续修改代码：

```
all:print
    @echo "Hello World"
print:
    @echo "Some Message"
```

主要all目标右边的print，这里要引入依赖关系的概念。  
all目标后面的print告诉make，all目标依赖print，print被认为是all的**先决条件**。一个目标的先决条件可以有多个，make会从左到右的顺序构建规则中的每一个目标。  
所以此时执行make命令，输出如下：

```
Some Message
Hello World
```

从上面我们可以得出一个规则的语法格式：

```
target:prerequisites
    command
```

通过这个例子，我们应该理解的知识点：

- 规则中描述目标，先决条件，生成目标所需要运行的命令
- 如果有依赖关系的目标，需要完成先决条件，才是执行该目标的命令
- 先决条件可以有多个且按照从左往右的顺序执行

## 一个简单的C语言程序
现在在当前目录下编写两个C文件：

```
//foo.c
#include <stdio.h>
void foo() 
{
    printf("This is foo()!\n");
}

//main.c
extern void foo();

int main() 
{
    foo();
    return 0;
}
```

我们有foo.c、main.c和Makefile三个文件，现在要做的就是将这两个C文件编译成一个C程序。  
我们需要有一个大概的流程，首先是将两个.c文件编译成一个目标文件，再将这两个目标文件链接成一个simple的可执行文件。  
修改Makefile内容如下：

```
all:main.o foo.o
    gcc -o simple main.o foo.o
main.o:main.c
    gcc -o main.o -c main.c
foo.o:foo.c
    gcc -o foo.o -c foo.c
```

此时执行make命令，我们会得到两个.o的目标文件和一个simple的可执行文件，以及在终端上显示如下信息：

```
gcc -o main.o -c main.c
gcc -o foo.o -c foo.c
gcc -o simple main.o foo.o
```

此时，目标不再仅仅局限于Makefile中定义的目标，而扩展到文件。现在有必要理解透彻内部机制：

- 判断先决条件中相关文件的时间戳是否大于目标的时间戳，即先决条件中的文件比目标新。
- 如果大于，那么需要运行规则当中的命令重新构建目标，否则将不作处理。

我们再次执行make命令，我们得到如下信息：

```
gcc -o simple main.o foo.o
```

现在我们也就不难理解为什么会出现这条信息。Makefile中第一条规则的目标是all，而all文件在编译过程中并不生成，再次执行make命令找不到all文件就重新编译。

如果我们将Makefile稍作修改：

```
simple:main.o foo.o
    gcc -o simple main.o foo.o
main.o:main.c
    gcc -o main.o -c main.c
foo.o:foo.c
    gcc -o foo.o -c foo.c
```

我们将得到的信息是`make: 'simple' is up to date.`，也就不再重新编译了。  

下面的情况读者可以参照上文所述的机制自己独立思考：

```
xuejijiedeMacBook-Pro:simple xuejijie$ ls -l
total 64
-rw-r--r--  1 xuejijie  staff   153  6 29 19:08 Makefile
-rw-r--r--  1 xuejijie  staff    63  6 29 14:42 foo.c
-rw-r--r--  1 xuejijie  staff   784  6 29 19:05 foo.o
-rw-r--r--  1 xuejijie  staff    55  6 29 13:54 main.c
-rw-r--r--  1 xuejijie  staff   652  6 29 19:05 main.o
-rwxr-xr-x  1 xuejijie  staff  8528  6 29 19:05 simple
xuejijiedeMacBook-Pro:simple xuejijie$ touch foo.o
xuejijiedeMacBook-Pro:simple xuejijie$ ls -l
total 64
-rw-r--r--  1 xuejijie  staff   153  6 29 19:08 Makefile
-rw-r--r--  1 xuejijie  staff    63  6 29 14:42 foo.c
-rw-r--r--  1 xuejijie  staff   784  6 29 19:11 foo.o
-rw-r--r--  1 xuejijie  staff    55  6 29 13:54 main.c
-rw-r--r--  1 xuejijie  staff   652  6 29 19:05 main.o
-rwxr-xr-x  1 xuejijie  staff  8528  6 29 19:05 simple
xuejijiedeMacBook-Pro:simple xuejijie$ make
gcc -o simple main.o foo.o
```

## 假目标
为了可以清空临时文件重新编译，我们修改Makefile：

```
simple:main.o foo.o
    gcc -o simple main.o foo.o
main.o:main.c
    gcc -o main.o -c main.c
foo.o:foo.c
    gcc -o foo.o -c foo.c
clean:
    rm simple main.o foo.o
```

然后执行`make clean simple`，可以得到重新编译的效果。  
此时有个小问题，就是如果我在当前目录下存在一个clean文件，那么上面的命令就通不过了，显示`make: 'clean' is up to date.`，原理可以参考上一节。  
解决这个问题可以采用**.PHONY**关键字消除矛盾，将clean变为假目标。

```
simple:main.o foo.o
    gcc -o simple main.o foo.o
main.o:main.c
    gcc -o main.o -c main.c
foo.o:foo.c
    gcc -o foo.o -c foo.c
clean:
    rm simple main.o foo.o

.PHONY:clean
```
然后再次执行`make clean`，就又可以清除了。

# 变量
Makefile也有变量的概念，修改上期的Makefile文件。

```
CC = gcc
RM = rm
EXE = simple
OBJS = main.o foo.o

$(EXE): $(OBJS)
    $(CC) -o simple main.o foo.o
main.o: main.c
    $(CC) -o main.o -c main.c
foo.o: foo.c
    $(CC) -o foo.o -c foo.c
clean:
    ${RM} ${EXE} ${OBJS}

.PHONY:clean
```
定义一个变量和一般语言区别不大，等号左边是一个变量，右边为变量的值。  
如果要使用变量，则用$(变量名)或${变量名}来调用。

## 自动变量
make会在你的规则中自动生成以下几个变量：

- $@用于表示一个规则中的目标。当我们的一个规则中有多个目标时,$@所指的是其中任何造成命令被运行的目标。
- $^则表示的是规则中的所有先决条件。 
- $<表示的是规则中的第一个先决条件。

我们先输出这个几个变量看看：

```
all: first second third
    @echo "\$$@=$@"
    @echo "$$^=$^"
    @echo "$$<=$<"
first second third:
```

得到的结果为：

```
$@=all
$^=first second third
$<=first
```

这时也就不难理解这几个自动变量了。需要注意的是Makefile中echo输出时需要$$表示为$，同时对于echo来说$@也有特殊意义，所以在前面添加一个转义字符。

## 特殊变量
第一个特殊变量：MAKE变量，它表示make命令名。

我们创建这样一个Makefile然后输出它：

```
all:
    @echo "MAKE=$(MAKE)"
```

在我的Mac上得到的内容如下：（不同平台的make输出情况各异）

```
MAKE=/Applications/Xcode.app/Contents/Developer/usr/bin/make
```

第二个特殊变量：MAKECMDGOALS变量，它表示当前用户输入的目标。

修改Makefile文件：

```
all clean:
    @echo "\$$@=$@"
    @echo "MAKECMDGOALS=$(MAKECMDGOALS)"
```

得到以下输出结果：

```
xuejijiedeMacBook-Pro:makefile xuejijie$ make
$@=all
MAKECMDGOALS=
xuejijiedeMacBook-Pro:makefile xuejijie$ make all
$@=all
MAKECMDGOALS=all
xuejijiedeMacBook-Pro:makefile xuejijie$ make all clean
$@=all
MAKECMDGOALS=all clean
$@=clean
MAKECMDGOALS=all clean
```

从测试结果看来,MAKECMDGOALS指的是用户输入的目标,当我们只运行make命令时，虽然根据Makefile的语法，第一个目标将成为缺省目标,即all目标,但MAKECMDGOALS仍然是空,而不是all，这一点我们需要注意。

第三个特殊变量：CC，指向当前使用的编译器，这里不再赘述。

## 变量的类别
我们定义如下变量：

```
.PHONY: all
foo = $(bar)
bar = $(ugh)
ugh = Huh?
all:
    @echo $(foo)
```

最终结果是会输出`Huh?`。对于这种我们只用一个 = 号来定义的变量，称之为递归扩展变量。  
还有一种称之为简单扩展变量，用:=操作符定义。  
我们修改Makefile文件：

```
.PHONY: all
x = foo
y = $(x) b 
x = later

xx := foo
yy := $(xx) b 
xx := later

all:
    @echo "x = $(y), xx = $(yy)"
```

我们可以很明显看出这两者的区别。简单扩展变量只进行一次扫描和替换。  
还有一种条件赋值符号 ?=，其作用是，如果变量有值，则不进行赋值操作。否则将右边的值赋值给左边。
使用方法不再赘述。

## 变量及其值的来源
除了通过定义变量外，变量的来源主要有以下几个途径：

- 前文提到的自动变量，其值是在每一个规则中根据规则的上下文自动获得变量值的。
- 在运行make时，在make命令行上定义一个或多个变量。例如`make bar=hello`来运行make。
- 从Shell环境中获取，Shell中的export命令来定义。

再介绍一个 += 运算符。

```
.PHONY: all
objects = main.o foo.o bar.o utils.o 
objects += another.o
all:
    @echo $(objects)
```

等价于

```
.PHONY: all
objects = main.o foo.o bar.o utils.o
objects := $(objects) another.o
all:
    @echo $(objects)
```

还要注意的是，每行命令在一个单独的shell中执行，这些shell没有继承关系：

```
all:
    @export foo=bar
    @echo "foo=[$$foo]"
```

执行make all结果是`foo=[]`。因为两个命令是在两个不同的进程执行。  
有两种解决方法：

```
all:
    @export foo=bar; echo "foo=[$$foo]"
```

或者


```
all:
    @export foo=bar; \
    echo "foo=[$$foo]"
```


## 高级变量引用功能

```
.PHONY: all
foo = a.o b.o c.o 
bar := $(foo:.o=.c)
all:
    @echo "bar = $(bar)"
```

输出结果为`bar = a.c b.c c.c`。
上面的例子Makefile展示了变量引用的一种高级功能，即在斌值的同时完成后缀替换操作。

## override指令
用override关键字重载变量，防止变量被覆盖。

```
.PHONY: all 
override foo = x 
all:
    @echo "foo = $(foo)"
```

输出：

```
xuejijiedeMacBook-Pro:makefile xuejijie$ make foo=hello
foo = x 
```

# 模式

使用正则表达式进行模式匹配减少代码：

```
CC = gcc
RM = rm
EXE = simple
OBJS = main.o foo.o

$(EXE): $(OBJS)
    $(CC) -o simple main.o foo.o
%.o: %.c
    $(CC) -o $@ -c $^
clean:
    ${RM} ${EXE} ${OBJS}

.PHONY:clean
```
# 函数
函数是减少重复操作的利器。对于一个源程序文件较多的项目，难以重复写make代码来完成时，这个时候函数就到了用武之地了。  
make函数的语法可以说有点古怪，不过这只是字面上的非主流，和所有其他语言的函数一样，都需要函数名、参数、返回值。  
我们先介绍下make自带的一些常用函数

## addprefix 函数
addprefix 函数是给字符串中的每个子串添加一个前缀。形式如下:  

```
$(addprefix prefix, names...)
```

创建一个Makefile文件如下:  

```
.PHONY:all

without_dir = foo.c bar.c main.o
with_dir = $(addprefix objs/, $(without_dir))

all:
    @echo $(with_dir)
```

执行make输出结果为`objs/foo.c objs/bar.c objs/main.o`

## filter 函数
filter 函数用于从一个字符串中，根据模式得到对应的字符串。形式如下:  

```
$(filter pattern..., text)
```

改写上文Makefile文件: 

```
.PHONY:all

sources = foo.c bar.c baz.s ugh.h
sources := $(filter %.c %.s, $(sources))

all:
    @echo $(sources)
```

得到结果`foo.c bar.c baz.s`

## filter-out 函数
filter-out 函数和filter函数相反，取不匹配模式的字符串。形式如下:

```
$(filter-out pattern..., text)
```

继续修改Makefile文件:

```
.PHONY:all

objects = main1.o foo.o main2.o bar.o
result := $(filter-out main%.o, $(objects))

all:
    @echo $(result)

```

执行make得到的结果为`foo.o bar.o`

## patsubst 函数
patsubst 函数用来进行字符串替换，形式如下:

```
$(patsubst pattern, replacement, text)
```

修改Makefile文件:

```
.PHONY:all
mixed = foo.c bar.c main.o
objects := $(patsubst %.c, %.o, $(mixed))

all:
    @echo $(objects)
```

执行make后得到的结果为`foo.o bar.o main.o`

## strip 函数
strip 函数用于剔除变量中多余的空格，形式如下:

```
$(strip string)
```

修改Makefile文件:

```
.PHONY:all

original = foo.c   bar.c
stripped := $(strip $(original))

all:
    @echo "strpped=$(stripped)"
```

输出结果为`stripped=foo.c bar.c`

## wildcard 函数
wildcard 函数是通配符函数得到需要的文件，形式如下:

```
$(wildcard pattern)
```

继续修改Makefile文件如下:

```
.PHONY:all

SRCS = $(wildcard *.c)

all:
    @echo $(SRCS)
```

得到的输出结果为`bar.c foo.c main.c`

# 总结
通过这三篇博文的Makefile教程，读者已经基本掌握Makefile的概念，在更复杂的项目中Makefile还需要更多的知识。  
请读者期待后续博文。谢谢！