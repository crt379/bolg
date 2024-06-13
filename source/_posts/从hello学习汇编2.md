---
title: 从hello world学习gas汇编1
date: 2024-06-10 23:49:04
tags: 
- Assembly
- Gas
---

## 示例
- c代码
```c
#include <stdio.h>

int main() {
   // printf() displays the string inside quotation
   printf("Hello, World!");
   return 0;
}
```

- AT&T
```asm
	.text
	.file	"hello.c"
	.globl	main                            # -- Begin function main
	.p2align	4, 0x90
	.type	main,@function
main:                                   # @main
	.cfi_startproc
# %bb.0:
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset %rbp, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register %rbp
	subq	$16, %rsp
	movl	$0, -4(%rbp)
	leaq	.L.str(%rip), %rdi
	movb	$0, %al
	callq	printf@PLT
	xorl	%eax, %eax
	addq	$16, %rsp
	popq	%rbp
	.cfi_def_cfa %rsp, 8
	retq
.Lfunc_end0:
	.size	main, .Lfunc_end0-main
	.cfi_endproc
                                        # -- End function
	.type	.L.str,@object                  # @.str
	.section	.rodata.str1.1,"aMS",@progbits,1
.L.str:
	.asciz	"Hello, World!"
	.size	.L.str, 14

	.ident	"Ubuntu clang version 18.1.6 (++20240518023229+1118c2e05e67-1~exp1~20240518143321.130)"
	.section	".note.GNU-stack","",@progbits
	.addrsig
	.addrsig_sym printf

```

- 2
```asm
hello:
    .string "Hello world!\n"
    len = . - hello
.text
.globl _start 
_start:
    movl $4, %eax
    movl $1, %ebx
    movl $hello, %ecx
    movl $len, %edx
    int $0x80

    movl $1, %eax
    movl $0, %edx
    int $0x80
```
[“Hello world” written using AT&T assembly](https://gist.github.com/gandaro/1966795)

- 3
```
hello:
    .string "Hello world!\n"
    len = . - hello
.text
.global _start
_start:
    nop
    .byte 0x90
    mov $1, %rax
    mov $1, %rdi
    mov $hello, %rsi
    mov $len, %rdx
    syscall
    mov $60, %rax
    mov $0, %rdi
    syscall
```

### 运行方式
as -o main.o main.S
ld -o main.out main.o
./main.out

## 组成
- 指示（Directives）以点号开始，用来指示对编译器，连接器，调试器有用的结构信息。指示本身不是汇编指令。例如，.file 只是记录原始源文件名。.data表示数据段(section)的开始地址, 而 .text 表示实际程序代码的起始。.string 表示数据段中的字符串常量。 .globl main指明标签main是一个可以在其它模块的代码中被访问的全局符号 。至于其它的指示可以忽略。
- 标签（Labels） 以冒号结尾，用来把标签名和标签出现的位置关联起来。例如，标签.LC0:表示紧接着的字符串的名称是 .LC0. 标签main:表示指令 pushq %rbp是main函数的第一个指令。按照惯例， 以点号开始的标签都是编译器生成的临时局部标签，其它标签则是用户可见的函数和全局变量名称。
- 指令（Instructions） 实际的汇编代码 (pushq %rbp), 一般都会缩进，以便和指示及标签区分开来。

## .p2align
https://sourceware.org/binutils/docs/as/P2align.html#P2align

```asm
.p2align	4, 0x90
.p2align 4,,15
```

第一个参数表示的是所需的 2 的幂字节对齐，.p2align 4填充在 16 字节边界上对齐，.p2align 532 字节边界等。
第二个参数表示用作填充的值。对于 x86，最好保留它并让汇编器选择，因为有一系列指令是有效的无操作。在某些对齐指令中，您会看到0x90，即NOP指令。
最后一个参数是填充的最大字节数 - 如果对齐要求的字节数大于此值，则跳过该指令。

## .cfi_* 汇编指示符
https://sourceware.org/binutils/docs-2.31/as/CFI-directives.html

CFI 即 Call Frame Information，是 DWARF 2.0 定义的函数栈信息，DWARF 即 Debugging With Attributed Record Formats ，是一种调试信息格式。

汇编文件里头经常看到 .cfi_ 开头的汇编指示符，例如 .cfi_startproc 、.cfi_undefined 、.cfi_endproc 等，CFI 即 Call Frame Information，是 DWARF 2.0 定义的函数栈信息，DWARF 即 Debugging With Attributed Record Formats ，是一种调试信息格式。 .cfi_ 开头的汇编指示符用来告诉汇编器生成相应的 DWARF 调试信息，主要是和函数有关。.cfi_startproc 定义函数开始，.cfi_endproc 定义函数结束。

## .cfi_def_cfa_offset
https://sourceware.org/binutils/docs/as/CFI-directives.html#CFI-directives

修改了计算 CFA 的规则。寄存器保持不变，但偏移量是新的。请注意，绝对偏移量将被添加到定义的寄存器以计算 CFA 地址。

## .cfi_offset 
寄存器的先前值保存在距 CFA 偏移量offset处。

## .cfi_def_cfa_register
修改了计算 CFA 的规则。从现在起将使用寄存器代替旧寄存器。偏移量保持不变。

## pushq %rbp
GAS 汇编指令通常以字母“b”、“s”、“w”、“l”、“q”或“t”作为后缀，以确定正在操作的操作数的大小。
- b = 字节（8 位）
- s = 短整型（16 位整数）或单整型（32 位浮点数）
- w = 字（16 位）
- l = long（32 位整数或 64 位浮点数）
- q = 四字（64 位）
- t = 10 字节（80 位浮点）
如果未指定后缀，并且指令没有内存操作数，则 GAS 根据目标寄存器操作数（最终操作数）的大小推断操作数的大小。

引用寄存器时，寄存器需要以“%”为前缀。常数需要以“$”为前缀。

pushq指令的功能是把字节数据压入到栈上。

pushq %rbp 保存了调用者的帧指针或保存了前一个堆栈帧的地址。

## x86（32 位和 64 位）寻址模式
x86（32 位和 64 位）有多种寻址模式可供选择。它们都具有以下形式：
```
[base_reg + index_reg*scale + displacement]      ; 或这个的一个子集
[RIP + displacement]     ; 或 RIP 相关：仅 64 位。不允许索引注册

segment:displacement(base register, index register, scale factor)
segment:[base register + displacement + index register * scale factor]
```

### 如何使用
如果您有一个int*，并且您有一个元素索引而不是字节偏移量，则通常会使用比例因子按数组元素大小缩放索引。（出于代码大小原因，最好使用字节偏移量或指针，以避免索引寻址模式，并且在某些情况下尤其是在英特尔 CPU 上的性能可能会损害微融合）。但您也可以做其他事情。如果您在 esi 中 有一个指针char array*:
- mov al, esi：无效，无法汇编。没有方括号，根本无法加载。这是一个错误，因为寄存器的大小不一样。
- mov al, [esi]加载指向的字节，即array[0]或*array。
- mov al, [esi + ecx]负载array[ecx]。
- mov al, [esi + 10]负载array[10]。
- mov al, [esi + ecx*8 + 200]负载array[ecx*8 + 200]
- mov al, [global_array + 10]从 加载global_array[10]。在 64 位模式下，这可以且应该是 RIP 相对地址。DEFAULT REL建议使用 NASM，默认生成 RIP 相对地址，而不必始终使用[rel global_array + 10]。我认为 MASM 默认会这样做。没有办法直接将索引寄存器与 RIP 相对地址一起使用。正常方法是lea rax, [global_array] mov al, [rax + rcx*8 + 10]或类似方法。
- mov al, [global_array + ecx + edx*2 + 10]从加载global_array[ecx  + edx*2 + 10] 显然，您可以使用单个寄存器索引静态/全局数组。 甚至可以使用两个单独的寄存器来索引 2D 数组。 （使用额外的指令预缩放一个，以获得除 2、4 或 8 之外的比例因子）。 请注意，数学global_array + 10是在链接时完成的。 目标文件（汇编器输出，链接器输入）通知链接器将 +10 添加到最终绝对地址，以将正确的 4 字节位移放入可执行文件（链接器输出）。
mov al, 0ABh根本不是加载，而是存储在指令内部的立即常数。（请注意，您需要在前缀 a 上加前缀，0以便汇编程序知道它是一个常量，而不是符号。有些汇编程序也会接受0xAB，而有些则不会接受0ABh：[请参阅更多](https://stackoverflow.com/questions/11733731/how-to-represent-hex-value-such-as-ffffffbb-in-x86-assembly-programming/37152498#37152498)）。


```
+------------------------+----------------------------+-----------------------------+
| Mode                   | Intel                      | AT&T                        |
+------------------------+----------------------------+-----------------------------+
| Absolute               | MOV EAX, [0100]            | movl           0x0100, %eax |
| Register               | MOV EAX, [ESI]             | movl           (%esi), %eax |
| Reg + Off              | MOV EAX, [EBP-8]           | movl         -8(%ebp), %eax |
| Reg*Scale + Off        | MOV EAX, [EBX*4 + 0100]    | movl   0x100(,%ebx,4), %eax |
| Base + Reg*Scale + Off | MOV EAX, [EDX + EBX*4 + 8] | movl 0x8(%edx,%ebx,4), %eax |
+------------------------+----------------------------+-----------------------------+



GAS memory operand      NASM memory operand
------------------      -------------------
100                     [100]
%es:100                 [es:100]
(%eax)                  [eax]
(%eax,%ebx)             [eax+ebx]
(%ecx,%ebx,2)           [ecx+ebx*2]
(,%ebx,2)               [ebx*2]
-10(%eax)               [eax-10]
%ds:-10(%ebp)           [ds:ebp-10]
Example instructions,
mov %ax,    100
mov %eax,   -100(%eax)


GAS syntax        NASM syntax
==========        ===========

jmp   *100        jmp  near [100]
call  *100        call near [100]
jmp   *%eax       jmp  near eax
jmp   *%ecx       call near ecx
jmp   *(%eax)     jmp  near [eax]
call  *(%ebx)     call near [ebx]
ljmp  *100        jmp  far  [100]
lcall *100        call far  [100]
ljmp  *(%eax)     jmp  far  [eax]
lcall *(%ebx)     call far  [ebx]
ret               retn
lret              retf
lret $0x100       retf 0x100
```


## movl	$0, -4(%rbp)
参考 x86（32 位和 64 位）寻址模式

%eax是寄存器 EAX；(%eax)是地址包含在寄存器 EAX 中的内存位置；8(%eax)是地址为 EAX 的值加 8 的内存位置。

## .equ
.equ就像#define在 C 中一样：
```
#define bob 10
.equ bob, 10
```
类型
```
.equ x, 123


mov $x, %eax
/* eax == 123 */
```

## .word
.word就像unsigned int在 C 中一样：
```
unsigned int ted;
ted: 
.word 0
```
或者用一个值初始化：
```
unsigned int alice = 42;
alice:
.word 42
```

## int
int表示中断，数字0x80表示中断号。中断将程序流转移到处理该中断的程序，0x80在本例中为中断。在 Linux 中，0x80中断处理程序是内核，用于其他程序对内核进行系统调用。

[ARM 汇编中 .equ 和 .word 有什么区别？](https://stackoverflow.com/questions/21624155/difference-between-equ-and-word-in-arm-assembly)

## 参考
- [.cfi_* 汇编指示符](https://blog.csdn.net/zoomdy/article/details/80700750)
- [x86 汇编/GNU 汇编语法](https://en.wikibooks.org/wiki/X86_Assembly/GNU_assembly_syntax)
- [x86 汇编/与 Linux 的接口](https://en.wikibooks.org/wiki/X86_Assembly/Interfacing_with_Linux)
- [CSE374：编程概念和工具-第 20 讲 — x86 汇编中的寻址和算术](https://courses.cs.washington.edu/courses/cse374/16wi/lectures/20-x86_addressing_arithmetic.html)
- [所有可用的 x86 寻址模式](https://stackoverflow.com/questions/34058101/referencing-the-contents-of-a-memory-location-x86-addressing-modes/34058400#34058400)
- [不同寻址模式的 AT&T(GNU) 语法与 NASM 语法表](https://stackoverflow.com/questions/6819957/what-does-the-bracket-in-movl-eax-eax-mean/6820015#6820015)
- [gnu as (gas) 文档](https://web.archive.org/web/20080313132324/http://sourceware.org/binutils/docs-2.16/as/index.html)
- [AT&T 汇编语法](https://web.archive.org/web/20080215230650/http://sig9.com/articles/att-syntax)
- [GNU 汇编程序](https://en.wikipedia.org/wiki/GNU_Assembler)
- [x86 汇编语言](https://en.wikipedia.org/wiki/X86_assembly_language#Syntax)
- [汇编文件中的CFI指令](https://garlicspace.com/2019/07/10/%E6%B1%87%E7%BC%96%E6%96%87%E4%BB%B6%E4%B8%AD%E7%9A%84cfi%E6%8C%87%E4%BB%A4/)
- [汇编代码中的“int 0x80”是什么意思？](https://stackoverflow.com/questions/1817577/what-does-int-0x80-mean-in-assembly-code)
- [Linux 上 32 位代码中的“int 0x80”或“syscall”哪个更好？](https://stackoverflow.com/questions/12806584/what-is-better-int-0x80-or-syscall-in-32-bit-code-on-linux)