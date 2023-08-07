# GDB basic

如何使用 GDB 调试 C 程序

```c
> cat args.c
#include <stdio.h>

#define NUM 10

int main(int argc, char *argv[])
{
    printf("参数个数：%d\n",argc);
    for (int i = 0; i < argc; i++) {
        printf("%d\n",NUM);
        printf("参数 %d: %s\n",i,argv[i]);
    }

    return 0;
}

```

## 添加调试信息

```sh
> gcc -g demo.c # gcc 编译时通过 -g 参数添加调试信息
```

## 开始调试

```sh
> gdb ./a.out # 通过 gcc -g 选项编译后，可通过 gdb 直接调试
GNU gdb (GDB) 13.2
Copyright (C) 2023 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-pc-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./a.out...
(gdb)
```

### 为 args 设置参数

```sh
(gdb) set args 1 2 3 4 5 6 # 为 args 设置参数
(gdb) show args # 显示设置的 args 参数
Argument list to give program being debugged when it is started is "1 2 3 4 5 6".
```

## 在 gdb 中启动程序

`run` 和 `start` 命令都可以启动程序

`run` 命令在`无断点` 的情况下会一直执行，直至程序结束

`start` 命令默认会执行一行，然后暂停，若`需要继续执行程序至结束，可输入 c (continue)` 命令

## 查看代码

```sh
(gdb) l # list 命令可以查看 main() 函数中的前 10 行代码

(gdb) l 行号 # 列出该行号对应的上下文代码，默认只显示 10 行

(gdb) l 函数名 # 显示这个函数的上下文内容，默认显示 10 行
```

### 切换文件后查看

> 在查看源码时，很多情况下会切换文件，只需要在 list 命令后面指定要查看的文件名即可。切换命令执行完毕后，这个文件就成了当前文件

```sh
# 切换到指定文件，并显示行号对应的上下文代码，默认显示 10 行
(gdb) l 文件名:行号

# 切换到指定文件，并显示这个函数的上下文内容，默认显示 10 行
(gdb) l 文件名:函数名
```

### 设置显示的行数

> 默认 list 命令只能显示 10 行代码，如果需要显示更多，可以通过 `set listsize` 命令来设置。如果想查看当前显示的行数，可通过 `show listsize` 查看，这里的 `listsize` 可缩写为 `list`

```c
# 以下两个命令中的 listsize 都可缩写为 list
(gdb) set list 行数

# 查看当前 listsize 一次显示的行数
(gdb) show listsize
```

## 断点

> 断点的设置有两种。1：常规断点 -> 程序只要运行到这个位置就会阻塞 2: 条件断点 -> 只有指定的条件被满足了，程序才会阻塞

- 设置普通断点到当前文件
```sh
# 在当前文件的某一行设置断点
(gdb) b 行号
(gdb) b 函数名 # 停止在函数的第一行
```

- 设置普通断点到某个非当前文件上

```sh
# 在非当前文件的某一行设置断点
(gdb) b 文件名:行号
(gdb) b 文件名:函数名 # 停止在函数的第一行
```

- 设置条件断点

```sh
# 必须满足某个条件，程序才会停在断点上
# 一般循环中条件断点使用较多 
(gdb) b 行号 if 变量名 == 某个值
```

### 查看断点

> 断点设置好后，可通过 `info break` 命令查看断点，其中 `info` 可以缩写为 `i`

```sh
(gdb) b 7
Breakpoint 1 at 0x1161: file demo.c, line 7.
(gdb) b 9
Breakpoint 2 at 0x1185: file demo.c, line 9.
(gdb) b 11
Breakpoint 3 at 0x1195: file demo.c, line 11.
(gdb) b 14
Breakpoint 4 at 0x11ae: file demo.c, line 14.
(gdb) b 8 if ch=='a'
Breakpoint 5 at 0x1163: file demo.c, line 8.
(gdb) info break
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x0000000000001161 in demo at demo.c:7
2       breakpoint     keep y   0x0000000000001185 in demo at demo.c:9
3       breakpoint     keep y   0x0000000000001195 in demo at demo.c:11
4       breakpoint     keep y   0x00000000000011ae in demo at demo.c:14
5       breakpoint     keep y   0x0000000000001163 in demo at demo.c:8
        stop only if ch=='a'
```

在显示的断点信息中，有些属性需要说明

- `NUM` 断点的编号，删除断点或设置断点状态时都需要使用
- `End` 当前断点的状态，y 表示断点可用，n 表示断点不可用
- `What` 描述断点被设置在了哪个文件的哪一行或函数中

### 删除断点

> 删除断点的命令是 `delete 断点编号` ，`delete` 可缩写为 `del` 或 `d`。删除断点的方式有两种：`删除一个或多个指定断点` 。`删除一个连续的断点区间`

```sh
# 删除第二个断点
(gdb) d 2
# 删除第三个断点
(gdb) d 3

# 删除连续范围内的断点
# 下行为删除第 1 到 4 个断点
(gdb) d 1-4
```

### 设置断点状态

> 如果某个断点只是不想用了，但并不想删除断点，则可以将其状态设置为不可用，使用 `disable 断点编号` 即可。当需要的时候再将其设置为可用状态 `enable 断点编号`

```sh
# 设置第三个断点不可用
(gdb) dis 3

# 设置连续范围内的断点不可用
# 下行为设置第 1 到 4 个断点不可用
(gdb) dis 1-4
```

## 打印变量值

`print` 和 `ptype` 分别为打印变量的值、打印变量的类型

### 自动打印信息

`display` 和 `print` 一样，也用于调试时查看某个变量或表达式的值，他们的区别在于：`display` 命令查看变量或表达式时，每当程序暂停执行(如单步执行)时，GDB 调试器都会自动打印出来，而 `print` 则不会

```sh
# 在变量的有效取值内，自动打印变量的值(设置一次后，会自动打印)
(gdb) display 变量名
```

### 查看自动打印的变量

> 对于通过 `display 变量名` 查看的变量或表达式，可通过 `info display` 查看都跟踪了哪些变量或表达式

### 取消自动打印

> 删除自动打印的变量或表达式，可用 `undisplay 变量名`

```sh
# 下行命令中的 编号，是通过 info display 得到的
(gdb) undisplay 编号
```

## 单步调试

> `step` 和 `next` 都可用于单步调试。区别在于 `next` 不会进入函数体，而 `step` 可以进入函数体

如果通过 `step` 进入函数体，想要跳出该函数体，可以执行 `finish` 命令，`如果要跳出函数体，必须保证函数体内不能有有效断点，否则无法跳出`

### 跳出循环

* `until` 命令可以直接跳出循环体。如果想要跳出循环体，必须满足以下条件
    - 要跳出的循环体不能有有效断点
    - 必须在循环的开始/结束行执行该命令

```sh
(gdb) until
```

## 设置变量的值

> 调试时，我们需要某个变量等于一个特殊值，此时就可以通过 `set var 变量名=值` 来设置

```sh
# 可以在循环中使用
(gdb) set var 变量名=值
```
