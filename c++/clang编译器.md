> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/30e111d4bec4)

clang 编译过程
==========

> `clang`是一个 `C`、`C++`和 `Objective-C`的编译器, 包含了`预处理`、`语法解析`、`代码生成`、`优化`、`汇编`和`链接`阶段, 尽管`clang`是高度集成的, 但是理解编译的各个阶段, 仍然很有必要.  
> **过程**:
> 
> > 预处理 -> 语法解析 -> 代码生成 & 优化 -> 汇编 -> 链接  
> > .c -> AST -> .s -> .o -> .out

编译过程
----

### 驱动

> clang 可执行文件实际上是一个小的**驱动程序**, 控制其他工具 (如编译器、汇编器和链接器) 的总体执行. 通常你不需要直接和驱动程序交互, 就可以使用 clang 来运行其他工具.

### 预处理

> 这个阶段会对输入的源文件进行标记化处理、宏扩展、`#include`扩展和其他预处理器指令的处理. 对`C`输出`.i`, 对`C++`输出 `.ii`, 对 `OC` 输出 `.mi`, 对`Objective-C++` 输出 `.mii`, 分别对应如下:
> 
> 输入: `.c`、 `.cpp`、 `.m`、 `.mm`  
> 输出: `.i`、 `.ii`、 `.mi`、 `.mii`

### 语法分析和语义分析

> 这个阶段会解析输入文件, 将预处理标记转换为解析树. 一旦以解析树的形式出现, 它也会用语义分析来计算表达式的类型, 确认代码格式是否正确.  
> 该阶段负责生成大多数的`编译警告`和`错误`, 输出为 `AST`(抽象语法树, Abstract Syntax Tree)
> 
> 输出: `AST`

### 代码生成和优化

> 这个阶段会将`AST`转换为底层的中间代码 (称为 `LLVM IR`), 再最终转换为机器码. 该阶段负责优化生成的代码, 并处理特定目标的代码生成. 输出通常称为 `.s` 文件或者 `汇编`文件.
> 
> 输出: `.s`、 `汇编`文件

### 汇编

> 这个阶段将编译器的输出转换为`目标文件`, 输出为 `.o` 文件 或者 `object` 文件
> 
> 输出: `.o`、 `object文件`

### 链接

> 这个阶段会将多个目标文件合并为一个`可执行文件`或动态库. 输出为 `a.out` 或者 `.so` 文件
> 
> 输出: `.out`、 `.so`

示例
--

> 下面用一个简单的`hello world`来演示一下:

**第 1 步**: 创建源码文件`hello.c`如下:

```c++
//
//  hello.c
//

#include <stdio.h>

#define ADD(x,y) (x+y)
#define ADD10(x, y) ADD(x+10, y)


int main() {
#if A
    printf("hello world A"); // A
#else
    printf("hello world B"); // B
#endif
    printf("result:%d", ADD10(1, 2));
}
```

**第 2 步**: 对其进行预编译, 得到`.i`输出文件, 使用命令:

```shell
$ clang -E hello.c -o hello.i
```

> `-E` 选项为进行预编译 (更多的编译选项可以在查看[这里](https://links.jianshu.com/go?to=http%3A%2F%2Fclang.llvm.org%2Fdocs%2FCommandGuide%2Fclang.html%23options))  
> `hello.i` 文件内容如下:

```c++
# 1 "hello.c"
# 1 "<built-in>" 1
# 1 "<built-in>" 3
# 362 "<built-in>" 3
# 1 "<command line>" 1
# 1 "<built-in>" 2
# 1 "hello.c" 2
.
.
.
# 10 "hello.c" 2



int main() {



    printf("hello world B");

    printf("result:%d", (1 +10 +2));
}
```

从中可以看到预处理做的一些工作

*   **`include 扩展`:** `# 10 "hello.c 2"`上边的均为`stdio.h`中的内容
*   **`标记化处理`:** `.i` 文件中行首行末的数字
*   **`去除注释`:** `hello.c` 的注释都没有了
*   **`条件编译`:** 未定义宏`A`, 故删除`#if A`分支下的代码块, 并保留该空行
*   **`宏删除`:** `ADD10` 和 `ADD` 宏被删除; 条件编译的`宏指令`也被删除
*   **`宏替换`:** `ADD10`和`ADD`宏直接替换为 (1+10+2), 并删除`宏定义`, 保留该空行

**第 3 步**: 编译, 得到`.s`文件, 使用命令

```
$ clang -S hello.i -o hello.s
```

> `-S` 选项为进行汇编之前的所有阶段 (`代码生成`、`优化`、`指定目标的代码生成`), 产生一个`汇编文件` (更多的编译选项可以在查看[这里](https://links.jianshu.com/go?to=http%3A%2F%2Fclang.llvm.org%2Fdocs%2FCommandGuide%2Fclang.html%23options))  
> `hello.s`文件内容如下:

```
.section    __TEXT,__text,regular,pure_instructions
    .build_version macos, 10, 15    sdk_version 10, 15
    .globl  _main                   ## -- Begin function main
    .p2align    4, 0x90
_main:                                  ## @main
    .cfi_startproc
## %bb.0:
    pushq   %rbp
    .cfi_def_cfa_offset 16
    .cfi_offset %rbp, -16
    movq    %rsp, %rbp
    .cfi_def_cfa_register %rbp
    subq    $16, %rsp
    leaq    L_.str(%rip), %rdi
    movb    $0, %al
    callq   _printf
    leaq    L_.str.1(%rip), %rdi
    movl    $13, %esi
    movl    %eax, -4(%rbp)          ## 4-byte Spill
    movb    $0, %al
    callq   _printf
    xorl    %esi, %esi
    movl    %eax, -8(%rbp)          ## 4-byte Spill
    movl    %esi, %eax
    addq    $16, %rsp
    popq    %rbp
    retq
    .cfi_endproc
                                        ## -- End function
    .section    __TEXT,__cstring,cstring_literals
L_.str:                                 ## @.str
    .asciz  "hello world B"

L_.str.1:                               ## @.str.1
    .asciz  "result:%d"


.subsections_via_symbols
```

**第 4 步**: 汇编, 得到`.o`文件, 使用命令

```
$ clang -c hello.s -o hello.o
```

> `-c` 选项为执行`汇编`及之前的所有阶段, 生成`机器码0101`, 产生一个`目标文件` (更多的编译选项可以在查看[这里](https://links.jianshu.com/go?to=http%3A%2F%2Fclang.llvm.org%2Fdocs%2FCommandGuide%2Fclang.html%23options))  
> `hello.o`文件内容如下:

```
cffa edfe 0700 0001 0300 0000 0100 0000
0400 0000 0802 0000 0020 0000 0000 0000
1900 0000 8801 0000 0000 0000 0000 0000
0000 0000 0000 0000 0000 0000 0000 0000
b800 0000 0000 0000 2802 0000 0000 0000
.
.
.
68ff ffff ffff ffff 3900 0000 0000 0000
0041 0e10 8602 430d 0600 0000 0000 0000
2800 0000 0100 002d 1900 0000 0200 0015
1200 0000 0100 002d 0b00 0000 0200 0015
0000 0000 0100 0006 0100 0000 0f01 0000
0000 0000 0000 0000 0700 0000 0100 0000
0000 0000 0000 0000 005f 6d61 696e 005f
7072 696e 7466 0000
```