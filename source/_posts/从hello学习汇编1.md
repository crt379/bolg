---
title: 从hello world学习MASM汇编1
date: 2024-06-10 22:00:55
tags: 
- Assembly
---

## 代码示例

- 1
```asw
DATA   SEGMENT
PRINT  DB "Hello World!", 0AH, 0DH, '$'
DATA   ENDS
STACK  SEGMENT STACK 
       DW  20  DUP(0)
STACK  ENDS
ASSUME CS:CODE, DS:DATA, SS:STACK
CODE   SEGMENT
START:
        MOV AX, DATA                         
        MOV DS, AX
        MOV DX, OFFSET  PRINT
        MOV AH, 09
        INT 21H
        MOV AH, 4CH
        INT 21H
CODE   ENDS
END    START
```

- 2
```asw
assume ds:datasg,ss:stacksg
stacksg segment stack
    db 256 dup(?)
stacksg ends
assume cs:datasg
datasg segment
    msg: 
    db 'Hello World!!$'
datasg ends
 
 
assume cs:codesg
codesg segment
main proc
    mov ax,datasg
    mov ds,ax
 
;forward-referenced,must far ptr call
    call far ptr greed
    
    mov ax,4c00h
    int 21h
    
main endp
codesg  ends
 
assume cs:greedsg
greedsg segment
greed proc far
    mov ah,9
    mov dx,OFFSET msg
    int 21h
    ret
greed endp
greedsg ends
 
end main
```

## 数据定义(伪指令助记符DB DD DQ DT)
- DB 定义的变量为字节型 Define Byte
- DW 定义的变量为字类型（双字节）Define Word
- DD 定义的变量为双字型（4字节）Define Double Word
- DQ 定义的变量为4字型（8字节）Define Quadra Word
- DT 定义的变量为10字节型 Define Ten Byte

## segment伪指令
segment是段，是段定义的伪指令。

### 定义
```asw
ASSUME CS:CODE, DS:DATA, SS:STACK       ; 定义代码段，标号为code；定义数据段，标号为data；定义栈段，标号为stack
```

### 段开始/端结束
```asw
DATA   SEGMENT
PRINT  DB "Hello World!", 0AH, 0DH, '$'
DATA   ENDS
```
data segment 表示数据段的开始，data ends 表示数据段结束。在他们之间则是数据段的内容：我们使用db申请了"Hello World!" 和 0AH, 0DH, '$' 的对应大小的空间并做了初始化。

### 使用数据段
```asw
MOV AX, DATA                         
MOV DS, AX
```
这两条指令，就是设置DS段的段地址。标号 data 就是我们在源程序开头使用 assume 将数据段DS与之绑定的标号，而标号 data 经过编译器编译后，则会变成一个地址，该地址是我们定义的数据段的段地址。那么指令语句：mov ax,data，含义就是将源程序中定义的数据段的段地址送入AX寄存器中。下面的指令语句：mov ds,ax，大家就很熟悉了，就是设置数据段的段地址，以便我们可以在程序中访问数据段。


## 参考
- [汇编语言里的数据定义伪指令助记符DB DD DQ DT以及全称](https://blog.csdn.net/weixin_44756776/article/details/108283586)
- [汇编学习教程：定义不同的段](https://blog.csdn.net/qq_34149335/article/details/124110122)
- [MASM汇编中伪指令ASSUME的作用](https://www.cnblogs.com/meizhouxiang/p/17668056.html)