---
title: c中编写/使用汇编
date: 2024-06-13 23:26:32
tags:
- C
- Gas
---

## 函数调用传参约定

在 Linux x86 中，使用 gcc 编译器进行程序编译时，函数调用时的参数传递规则如下：

- 函数参数通过栈传递，按照从右往左的顺序入栈
- 函数返回值保存在寄存器 eax 中

在 Linux x86_64 中，函数调用时的参数传递规则如下：
- 前 6 个参数按从左往右的顺序分别通过寄存器 rdi、rsi、rdx、rcx、r8、r9，剩下的参数按从右往左的顺序通过栈传递
- 函数返回值保存在寄存器 rax 中

函数调用时的参数传递规则实际上与函数调用约定有关，与编译器无关，常见的函数调用约定包括 c调用约定、std调用约定、x86 fastcall约定以及 C++调用约定等。gcc 编译器采用的 c 调用约定。

## 代码

```c
#include <stdio.h>
 
int add(int a, int b) {
    int r;

    asm(
        "addl %%edi, %%esi\n\r"
        "movl %%esi, %%eax\n\r"
        :"=r"(r)
    );

    return r;
}
 
int main() {
    printf("%d\n", add(2, 3));
    return 0;
}

```

## c扩展Asm

- 格式
```c
asm ( assembler template 
    : output operands                  /* optional */
    : input operands                   /* optional */
    : list of clobbered registers      /* optional */
    );
```
汇编器模板由若干条汇编指令组成,每个操作数（括号里C语言的变量）都有一个限制符（“”中的内容）加以描述.
冒号用来分割输入的操作和输出的操作. 如果每组内有多个操作数,用逗号分割它们.  
操作数最多为10个, 或者依照具体机器而异（以较大者为准）。

如果没有输出操作, 但是又有输入, 你必须使用连续两个冒号, 两个连续冒号中无内容, 表示没有输出结果的数据操作 .
```c
asm ("cld\n\t"
    "rep\n\t"
    "stosl"
    : /* no output registers */
    : "c" (count), "a" (fill_value), "D" (dest)
    : "%ecx", "%edi" 
    );
```

上面这段代码做了什么? 这段内嵌汇编把 fill_value, count装入寄存器edi。它还告诉 gcc，寄存器eax和edi的内容不再有效。让我们再看一个例子:
```c
int a=10, b;
asm ("movl %1, %%eax; 
    movl %%eax, %0;"
    :"=r"(b)        /* output */
    :"r"(a)         /* input */
    :"%eax"         /* clobbered register */
    );    
```

这里我们所做的是使用汇编指令使“b”的值等于“a”的值。一些有趣的点如下：
- “b” 是输出操作数，由 %0 引用，“a” 是输入操作数，由 %1 引用。
- “r” 是操作数的约束。稍后我们将详细了解约束。目前，“r” 告诉 GCC 使用任何寄存器来存储操作数。输出操作数约束应该有一个约束修饰符“=”。这个修饰符表示它是输出操作数并且是只写的。
- 寄存器名称前面有两个 % 前缀。这有助于 GCC 区分操作数和寄存器。操作数有一个 % 作为前缀。
- 第三个冒号后的破坏寄存器 %eax 告诉 GCC，%eax 的值将在“asm”内部被修改，因此 GCC 不会使用该寄存器来存储任何其他值。
当“asm”执行完成后，“b”将反映更新的值，因为它被指定为输出操作数。换句话说，在“asm”内部对“b”所做的更改应该反映在“asm”外部。


## 参考
- [C 与汇编语言混合使用](https://cq674350529.github.io/2020/01/13/C%E4%B8%8E%E6%B1%87%E7%BC%96%E8%AF%AD%E8%A8%80%E6%B7%B7%E5%90%88%E4%BD%BF%E7%94%A8/)
- [关于调用约定(cdecl、fastcall、stcall、thiscall) 的一点知识](https://www.laruence.com/2008/04/01/116.html)
- [x86 调用约定](https://en.wikipedia.org/wiki/X86_calling_conventions)
- [C/C++ 中的调用约定](https://www.geeksforgeeks.org/calling-conventions-in-c-cpp/)
- [内联汇编：将汇编与 C/C++ 混合](https://www.cs.uaf.edu/courses/cs301/2014-fall/notes/inline-assembly/)
- [5.34 带有 C 表达式操作数的汇编指令](https://gcc.gnu.org/onlinedocs/gcc-4.0.2/gcc/Extended-Asm.html#Extended-Asm)
- [GCC 内联汇编指南](https://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)