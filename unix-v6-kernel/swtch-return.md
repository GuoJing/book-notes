### Swtch 返回位置

swtch() 的返回值为执行 savu() 的函数之声被调用的位置。

首先需要对函数调用时的进程有所了解。由于使用的编译器等原因，栈的行为会有所不同，此处以示例程序在 UNIX V6 上的运行情况为基础进行说明。

```
callee (a, b) {
    int c, d;
    c = a;
    d = b;
    return c + d;
}

caller () {
    int e, f, result;
    e = 1;
    f = 2;
    result = callee(e, f);
    return result;
}
```

上述代码在模拟器上生成以下汇编代码。

```
.globl _callee
.text
_callee:
~~callee:
~a=4
~b=6
~c=177770
~d=177766
// 2.
// callee 获得控制权后，首先执行 csv
// csv 函数用于向栈压入 r2, r3, r4的值
jsr         r5, csv
sub         $4, sp
mov         4(r5), -10(r5)
mov         6(r5), -12(r5)
mov         -10(r5), r0
add         -12(r5), r0
jbr         L1
L1:         jmp cret
.globl      _caller
.text

_caller:
~~caller:
~result=177764
~e=177770
~f=177766
jsr         r5, csv
sub         $6, sp
mov         $1, -10(r5)
mov         $2, -12(r5)
// 1.
// 将传递给 callee() 的参数压入栈之后，通过 jsr 指令跳转至callee()
// jsr 指令的第一个参数被压入栈，此处被压入栈的 pc 的值将成为从
// callee 返回的位置
// jsr 指令的第二个参数为 pc 的新值，控制权同时被移交给 callee
mov         -12(r5), (sp)
mov         -10(r5), -(sp)
jsr         pc, *$_callee
tst         (sp)+
mov         r0, -14(r5)
mov         -14(r5), 50
jbr         L2
L2:jmp  cret
.globl
.data
```