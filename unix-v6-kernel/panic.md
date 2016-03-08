### panic

ken/prf.c

panic 通过循环调用 idle 进行玄幻调用以阻止程序继续运行。

```
panic(s)
char *s;
{
    panicstr = s'
    update();
    printf("panic: %s\n", s);
    for (;;)
        idle();
}
```

idle m40.s

```
.globl  _idle
_idle:
    mov     PS, -(sp)
    bic     $340, PS
    wait
    mov     (sp)+, PS
    rts     pc
```