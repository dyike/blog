---
title: GDB调试器的学习
date: 2016-08-13 23:37:58
tags: GDB,C,Golang
---

> 关于GDB是什么，我就不多介绍了，这个不了解可以去谷歌一下。反正就是一个很强大的程序调试的工具。
> 为什么要学习这个？之前学习C语言不是在Linux下编程，后来发现之前折腾的跟现实还是有点出入的，所以现在有的时候还是挺懵逼的，每次都为调试代码这件事而苦恼。
> 还有就是学习Golang的都知道，go语言是支持GDB调试的。Go语言作为一门静态语言，当然可以通过Println之类的打印，但每次修改都需要重新编译。 
> 这么看来，既然有这么一个强大的工具，有什么理由不掌握它呢？这里我从网上收集的，整理了一下，希望对你的学习也有帮助。

GDB能做的事情如下几个方面：
* 启动程序，根据自己的想法来运行程序
* 让程序在指定的断点出停住，断点可以是条件表达式
* 在断点处，查看上下文
* 动态的改变程序的执行环境

先说说安装吧，在ubuntu下安装也不是一件很难的事，这个自行谷歌。最好尝试编译安装，国内下载可能非常非常的慢。我这里给一个我之前下载好的，放在[百度云盘](https://pan.baidu.com/s/1qYtOIio)中, <strong>密码: 3drw</strong>

下面直接切入学习吧！

我们来编写一个简单的C程序：example.c

```c
#include <stdio.h>

int add(int a, int b)
{
    return a+b;
}

int main()
{
    int sum[5] = {0, 0, 0, 0, 0};
    int i;
    int array1[5] = {1, 2, 3, 4, 5};
    int array2[5] = {2, 4, 6, 8, 10};
    for (i = 0; i < 5; i++)
    {
        sum[i] = add(array1[i], array2[i]);
    }
}
```
编译：`gcc -g example.c -o example`,得到一个二进制文件example,执行`gdb example`命令进入调试状态。
> 注意这里的编译命令，想想为什么要带上-g参数。如果没有-g参数，就看不到程序的函数名、变量名，所替代的全是运行时的内存地址。

你将看到的是：

```
root@ityike:/vagrant/cproject/learngdb  # gdb example
GNU gdb (GDB) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-pc-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from example...done.
(gdb)
```

启动GDB的方法有：
* `gdb <program>`      | program也就是你的执行文件，一般在当前目录下。
* `gdb <program> core`  | 用gdb同时调试一个运行程序和core文件，core是程序非法执行后core dump后产生的文件。
* `gdb <program> <PID>`  | 可以指定这个服务程序运行时的进程ID，gdb会自动attach上去，并调试他。（一般用于调试已经在运行的程序）

## 常用命令

### 1、list【缩写：l】

 list命令后面可以带的参数：
 * `<linenum>` 行号
 * `<+offset>` 当前行号的正偏移量
 * `<-offset>` 当前行号的负偏移量
 * `<filename:linenum>` 哪个文件的哪一行
 * `<function>` 函数名
 * `<filename:function>` 哪个文件的哪个函数
 * `<*address>` 程序运行时语句在内存中的地址

* `list <linenum>`,显示第linenum行附近的源程序。
* `list <function>`,显示函数名为function函数上下的源程序。
* `list` ,显示当前行后面的源程序。
* `list -`,显示当前行前面的源程序。
一般默认显示行数是10，也可以设置自己的行数。
* `show listsize`,查看当前设置的list行数。
* `set listsize <count>`,设置自己的一次显示的行数。
其他常用的list命令：
* `list <first>,<last>`  显示从first到last行之间的代码。
* `list ,<last>`  显示从当前行到last行之间的代码。
* `list +`  往后显示源代码。

看到这里，不难发现，list就是一个显示内容的命令，是不是没啥呢？其实GDB还提供了代码搜索的命令。
* `forward-search <regexp>` | `search <regexp>` 向前搜索
* `reverse-search <regexp>` 全部搜索
> `<regexp>`是正则表达式，你是不是跟我一样还要去学习正则表达式，没有啥怕的！


### 2、run【缩写:r】
 程序的运行，你有可能需要设置下面四方面的事:
 * 程序运行参数
 > `set args` 指定运行时的参数。（set args 1,2,3,4,5)
 > `show args` 查看当前设置好的运行参数。
 * 运行环境
 > `show paths` 查看程序的运行路径。
 > `path <dir>` 设定程序的运行路径。
 > `set enviroment varname[=value]` 设置环境变量。（set env USER=ityike)
 > `show enviroment [varname]` 查看环境变量。
 * 工作目录
 > `cd <dir> `相当于shell的cd命令。
 > `pwd` 显示当前的所在目录。
 * 程序的输入输出
 > `info terminal` 显示你程序用到的终端的模式。
 > 使用重定向控制程序输出。如：`run > outfile`
 > tty命令可以指写输入输出的终端设备。如：`tty /dev/ttyb`

### 3、break【缩写:b】

* `break <function>` 进入指定函数时停住。在C++中使用class::function或者function(type,type)格式来指定函数名。
* `break <linenum>` 在指定行停住。
* `break +offset / break -offset`在当前行号的前面或后面的offset行停住。
* `break filename:linenum`在filename文件的linenum行停住。
* `break filename:function`在filename文件的function函数的入口处停住。
* `break *address `在程序运行的内存地址处停住。
* `break`不带参数表示在下一条指令处停住。
* 在上面的带有参数的命令后面，还可以带有condition表示条件。`break ... if <condition>`，条件成立的时候停住。
* 查看断点，可使用info命令，如`info breakpoints [n]`、`info break [n]`（n表示断点号）。

### 4、单步命令

next命令可以用于单步执行，但是next不会进入函数的内部，那怎么办呢？还有一个命令step【缩写:s】会进入函数的内部。

* `step <count>` step后面不加count表示一条条地执行，加表示执行后面的count条指令，然后再停住。
* `next <count> `next后面不加count表示一条条地执行，加表示执行后面的count条指令，然后再停住。
* `set step-mode` 
 
> set step-mode on用于打开step-mode模式，这样，在进行单步跟踪时，程序不会因为没有debug信息而不停住，这个参数的设置可便于查看机器码。
> set step-mod off用于关闭step-mode模式。

* `finish` 运行程序，直到当前函数完成返回，并打印函数返回时的堆栈地址和返回值及参数值等信息。
* `until`【缩写:u】一直在循环体内执行单步，退不出来是一件令人烦恼的事情，until命令可以运行程序直到退出循环体。


### 5、continue【缩写:c】
当程序被停住后，可以使用continue恢复程序的运行知道程序结束，或达到下一个端点。
* `continue [ignore-count]`  ignore-count表示忽略其后多少次断点。


### 6、print 【缩写:p】

* `print <expr>`  <expr>是表达式，是被调试的程序中的表达式。
* `print /<f> <expr>`   <f>是输出的格式，如果要把表达式按16进制的格式输出，那么就是/x。

 > “@”是一个和数组有关的操作符，“::”指定一个在文件或是函数中的变量，“{<type>} <addr>”表示一个指向内存地址<addr>的类型为type的一个对象。
  
 > x 按十六进制格式显示变量。
 > d 按十进制格式显示变量。
 > u 按十六进制格式显示无符号整型。
 > o 按八进制格式显示变量。
 > t 按二进制格式显示变量。
 > a 按十六进制格式显示变量。
 > c 按字符格式显示变量。
 > f 按浮点数格式显示变量。

下面我们演示一下sum[]的数组变化过程：

```
(gdb) b 14
Breakpoint 1 at 0x400584: file example.c, line 14.
(gdb) r
Starting program: /vagrant/cproject/learngdb/example

Breakpoint 1, main () at example.c:14
14          sum[i] = add(array1[i], array2[i]);
(gdb) p sum
$1 = {0, 0, 0, 0, 0}
(gdb) n
12      for (i = 0; i < 5; i++)
(gdb) n

Breakpoint 1, main () at example.c:14
14          sum[i] = add(array1[i], array2[i]);
(gdb) p sum
$2 = {3, 0, 0, 0, 0}
(gdb) n
12      for (i = 0; i < 5; i++)
(gdb) p sum
$3 = {3, 6, 0, 0, 0}
(gdb) n

Breakpoint 1, main () at example.c:14
14          sum[i] = add(array1[i], array2[i]);
(gdb) p sum
$4 = {3, 6, 0, 0, 0}
(gdb) n
12      for (i = 0; i < 5; i++)
(gdb) p sum
$5 = {3, 6, 9, 0, 0}
```

如果要修改变量，如x的值，可使用命令：`print x=4`


还有关于数组，有时需要查看一段连续内存空间的值，比如数组的一段，或者是动态分配的数据大小。可以使用“@”操作符。“@”左边是一个内存的地址的值，“@”右边是你查看内存的长度。比如：

```
(gdb) p sum
$7 = {3, 6, 9, 12, 0}
(gdb) p *sum@5
$8 = {3, 6, 9, 12, 0}
(gdb) p *sum@2
$9 = {3, 6}
```


### 7、watch命令
watch一般来观察某个表达式（变量也是一种表达式）的值是否有变化了，如果有变化，马上停住程序。
* `watch <expr>`：为表达式（变量）expr设置一个观察点。
* `rwatch <expr>`：当表达式（变量）expr被读时，停住程序。
* `awatch <expr>`：当表达式（变量）的值被读或被写时，停住程序。
* `info watchpoints`：列出当前所设置了的所有观察点。

看看i在连续运行next时一旦发现i变化，i值就会显示出来。

```
(gdb) watch i
Hardware watchpoint 2: i
(gdb) next

Hardware watchpoint 2: i

Old value = 2
New value = 3
0x00000000004005ae in main () at example.c:12
12      for (i = 0; i < 5; i++)
(gdb) n

Breakpoint 1, main () at example.c:14
14          sum[i] = add(array1[i], array2[i]);
(gdb) n
12      for (i = 0; i < 5; i++)
(gdb) n

Hardware watchpoint 2: i

Old value = 3
New value = 4
0x00000000004005ae in main () at example.c:12
12      for (i = 0; i < 5; i++)
(gdb) info watchpoints
Num     Type           Disp Enb Address            What
2       hw watchpoint  keep y                      i
    breakpoint already hit 2 times
```


<strong>下面这一部分结合上面的break命令</strong>

#### 设置捕捉点（CatchPoint）
你可设置捕捉点来捕捉程序运行时的一些事件。如：载入共享库（动态链接库）或是C++的异常。

* catch <event>

 > 当event发生时，停住程序。event可以是下面的内容：
 > 1、throw 一个C++抛出的异常。（throw为关键字）
 > 2、catch 一个C++捕捉到的异常。（catch为关键字）
 > 3、exec 调用系统调用exec时。（exec为关键字，目前此功能只在HP-UX下有用）
 > 4、fork 调用系统调用fork时。（fork为关键字，目前此功能只在HP-UX下有用）
 > 5、vfork 调用系统调用vfork时。（vfork为关键字，目前此功能只在HP-UX下有用）
 > 6、load 或 load <libname> 载入共享库（动态链接库）时。（load为关键字，目前此功能只在HP-UX下有用）
 > unload 或 unload <libname> 卸载共享库（动态链接库）时。（unload为关键字，目前此功能只在HP-UX下有用）

* tcatch <event>
只设置一次捕捉点，当程序停住以后，该点被自动删除。

#### 维护停止点
GDB中的停止点也就是断点、观察点、捕捉点。如果你觉得停止点没有用了，可以使delete、clear、disable、enable来进行维护。

* `clear`    清除所有的已定义的停止点。
* `clear <function> /clear <filename：function>` 清除所有设置在函数上的停止点。
* `clear <linenum> / clear <filename：linenum>`  清除所有设置在指定行上的停止点。
* `delete [breakpoints] [range...] `   删除指定的断点，breakpoints为断点号。如果不指定断点号，则表示删除所有的断点。range 表示断点号的范围（如：3-7）。简写:d

比删除更好的一种方法是disable停止点，disable了的停止点，GDB不会删除，当你还需要时，enable即可，就好像回收站一样。
* `disable [breakpoints] [range...]` disable所指定的停止点，breakpoints为停止点号。如果什么都不指定，表示disable所有的停止点。简写:dis。
* `enable [breakpoints] [range...]` enable所指定的停止点，breakpoints为停止点号。
* `enable [breakpoints] once range...` enable所指定的停止点一次，当程序停止后，该停止点马上被GDB自动disable。
* `enable [breakpoints] delete range...` enable所指定的停止点一次，当程序停止后，该停止点马上被GDB自动删除。

#### 停止条件的维护
前面我们说到的设置断点时，可以设置一个条件，当条件成立时，程序会自动停止。这个的条件的相关维护命令，为断点设置一个条件，使用if关键字，后面加上断点条件。条件设置好，我们可以使用condition命令来修改断点的条件。（只有break和watch命令支持if，catch目前暂不支持if）
* `condition <bnum> <expression>` 修改断点号为bnum的停止条件为expression。
* `condition <bnum>` 清除断点号为bnum的停止条件。

还有一个特殊的命令ignore
* `ignore <bnum> <count>`  表示忽略断点号为bnum的停止条件count次。

#### 为停止点设定运行的命令
为断点号bnum指写一个命令列表。当程序被该断点停住时，gdb会依次运行命令列表中的命令。格式如下：

```
commands [bnum]
... command-list ...
end
```

例如：

```
break add if sum>0
commands
printf "sum is %d\n",sum
continue
end
```
断点设置在函数add中，断点条件为sum》0，一旦sum的值大于0，就会自动打印sum的值，并继续运行程序。
如果要清除断点上的命令序列，那么只要简单的执行一下commands命令，并直接再打个end就行了。

### 8、Signals
信号是一种软中断，是一种处理异步事件的方法。一般来说，操作系统都支持许多信号。UNIX定义了许多信号，比如SIGINT表示中断字符信号，也就是Ctrl+C的信号，SIGBUS表示硬件故障的信号；SIGCHLD表示子进程状态改变信号；SIGKILL表示终止程序运行的信号，等等。
GDB的handle命令可以完成这个调试，格式如下：
`handle <signal> <keywords...>`
信号<signal>可以以SIG开头或不以SIG开头，可以用定义一个要处理信号的范围（如：SIGIO- SIGKILL，表示处理从SIGIO信号到SIGKILL的信号，其中包括SIGIO，SIGIOT，SIGKILL三个信号），也可以使用关键字 all来标明要处理所有的信号。一旦被调试的程序接收到信号，运行程序马上会被GDB停住，以供调试。其<keywords>可以是以下几种关键字的一个或多个。

* `nostop` 当被调试的程序收到信号时，GDB不会停住程序的运行，但会打出消息告诉你收到这种信号。
* `stop` 当被调试的程序收到信号时，GDB会停住你的程序。
* `print` 当被调试的程序收到信号时，GDB会显示出一条信息。
* `noprint` 当被调试的程序收到信号时，GDB不会告诉你收到信号的信息。
* `pass | noignore` 当被调试的程序收到信号时，GDB不处理信号。这表示，GDB会把这个信号交给被调试程序处理。
* `nopass | ignore`   当被调试的程序收到信号时，GDB不会让被调试程序来处理这个信号。
* `info signals | info handle`  查看有哪些信号在被GDB检测中。

### 9、线程
很多时间会涉及到多线程，那么断点是不是要打在所有的线程或者某个特定的线程上，GDB可以出色的完成这一项，这也是它强大的原因。

```
break <linespec> thread <threadno>
break <linespec> thread <threadno> if ...
```
linespec指定了断点设置在的源程序的行号。threadno指定了线程的ID

> 这个ID是GDB分配的，你可以通过`info threads`命令来查看正在运行程序中的线程信息.

如果你不指定thread <threadno>则表示你的断点设在所有线程上面。还可以为某线程指定断点的条件。当你的程序被GDB停住时，所有的运行线程都会被停住。这方便你查看运行程序的总体情况。而在你恢复程序运行时，所有的线程也会被恢复运行。哪怕是主进程在被单步调试时。

#### 查看栈信息
如果程序被停住了，此刻需要做的第一件事就是查看程序是在哪儿停住的。当你的程序调用了一个函数，函数的地址，参数，函数内部的局部变量都会被压入栈中。这时你又可以发现GDB的强大之处了。

* `命令backtrace 【缩写:bt】`可以打印当前函数调用栈的所有信息。
* `bt <n> ` n是一个正整数，表示只打印栈顶上n层的栈信息。
* `bt <-n>` -n表一个负整数，表示只打印栈底下n层的栈信息。
如果你要查看某一层的信息，你需要切换当前栈，一般来说，程序停止时，最顶层的栈就是当前栈，如果你需要查看栈下面层的详细信息，你需要做的就是切换当前栈。
* `frame <n> | f <n>` n是一个从0开始的整数，是栈中的层编号。比如：frame 0，表示栈顶，frame 1，表示栈的第二层。
* `up <n>`  表示向栈的上面移动n层，可以不打n，表示向上移动一层。
* `down <n>`  表示向栈的下面移动n层，可以不打n，表示向下移动一层。
上面这三条命令都会打印栈层的信息，如果不想打印其信息，可以使用这三条命令：

```
select-frame <n> 对应于 frame 命令。
up-silently <n> 对应于 up 命令。
down-silently <n> 对应于 down 命令。
```

### 10、info命令
info命令可以在调试时用来查看寄存器、断点、观察点和信号等信息。
*  `info registers` 查看除了浮点寄存器以外的寄存器
*  `info all-registers` 查看所有寄存器，包括浮点寄存器
*  `info registers <regname ...>`  查看所指定的寄存器
*  `info break`  列出当前所设置的所有观察点 
*  `info frame | info f` 打印出更为详细的当前栈层的信息，只不过，大多数都是运行时的内存地址。比如：函数地址，调用函数的地址，被调用函数的地址，目前的函数是由什么样的程序语言写成的、函数参数地址及值、局部变量的地址等等。
*  `info args`    打印出当前函数的参数名及其值。
*  `info locals`  打印出当前函数中所有局部变量及其值。
*  `info catch`   打印出当前的函数中的异常处理信息。 
*  `info source`  查看当前文件的程序语言。

### 11、jump命令
一般情况程序都是顺序执行的，现在想要乱序执行怎么办呢？GDB也是可以修改程序的执行顺序，让程序随意跳转。
* `jump <linespec>` 来指定下一条语句的运行点。<linespec>可以是文件的行号，可以是file:line格式，也可以是+num这种偏移量格式，表示下一条运行语句从哪里开始。
* `jump <address>` 这里的<address>是代码行的内存地址。

> jump命令不会改变当前的程序栈中的内容，所以，如果使用jump从一个函数跳转到另一个函数，当跳转到的函数运行完返回，进行出栈操作时必然会发生错误，这可能导致意想不到的结果，所以最好只用jump在同一个函数中进行跳转。

### 12、强制函数返回 return

`return`
`return <expression>` 使用return命令取消当前函数的执行，并立即返回，如果指定了<expression>，那么该表达式的值会被认作函数的返回值。

### 13、强制调用某函数 call
 
* `call <expr>` 表达式中可以一是函数，以此达到强制调用函数的目的，它会显示函数的返回值（如果函数返回值不是void）

### 14、set命令
* `set print address on/off`   打开/关闭地址输出，当程序显示函数信息时，GDB会显出函数的参数地址。
* `set print array on/off`    打开/关闭 数组显示。
* `set print elements <number-of-elements>`   设置数组的长度，，如果长度太长，超过了就不再往下显示了。
* `set print null-stop <on/off>`  打开了这个选项，那么当显示字符串时，遇到结束符则停止显示。这个选项默认为off。
* `set print pretty on`   GDB显示结构体时会比较漂亮。
* `set print sevenbit-strings <on/off> `   设置字符显示，是否按“\nnn”的格式显示，如果打开，则字符串或字符数据按\nnn显示，如“\065”。
* `set print union <on/off>`   设置显示结构体时，是否显式其内的联合体数据。
* `set scheduler-locking off|on|step`

> off 不锁定任何线程，也就是所有线程都执行，这是默认值。
> on 只有当前被调试程序会执行。
> step 在单步的时候，除了next过一个函数的情况以外，只有当前线程会执行。

* 还有其他的。比如修改寄存器，修改内存

### 15、自动显示 display
expr是一个表达式，fmt表示显示的格式，addr表示内存地址，当你用display设定好了一个或多个表达式后，只要你的程序被停下来，GDB会自动显示你所设置的这些表达式的值。
* `display <expr>`
* `display/<fmt> <expr>`
* `display/<fmt> <addr>`
格式i和s同样被display支持，一个非常有用的命令是：`display/i $pc`
$pc是GDB的环境变量，表示着指令的地址，/i则表示输出格式为机器指令码，也就是汇编。于是当程序停下后，就会出现源代码和机器指令码相对应的情形，这是一个很有意思的功能。

删除自动显示：
* `disable display <dnums...>`
* `enable display <dnums...>`

———end----有错误欢迎指定!-----------










