### 选择执行进程的算法

swtch() 用来选择下一个将被执行的进程，它从起始位置遍历 proc[], 并将满足下列条件的进程选择为执行进程。

1. 处于可执行状态 SRUN
2. 拥有最高优先级 proc.p_pri 的值最小

setpri() 函数用来随时调整各种进程的执行优先级。占用 CPU 时间越长的进程，其执行优先级将会之间降低。

内核中有很多处理都是从最起始位置遍历 proc[] 的，而 swtch() 是从代表当前执行进程的元素位置开始便利 proc[]。如果存在两个以上具有最高执行优先级的进程，位于对应执行进程元素的后方，并且离它最近的那个会被优先选择。

选择了需要执行的进程后，下列处理进行进程切换。

1. 修改内核 APR，使得被选择进程的 user 结构体可以通过 u 进行访问
2. 恢复在被选择进程的 user 结构体中保存的 r5，r6 的值
3. 恢复在被选择进程的 user 结构体中保存的用户 APR 的值

swtch 执行结束后，被选择的进程将返回到调用 savu() 以保存 r5，r6 的函数自身被调用的位置。

ken/slp.c

```
swtch(){
    // 保存着上次执行 swtch() 时选择的进程，第一次执行时为 NULL
    static struct proc *p;
    register i, n;
    register struct proc *rp;

    if(p==NULL)
        p = &proc[0];

    // 将 r5, r6 的当前值保存于待中断进程的 user.u_rsav 之中
    // 等到该进程恢复时再次执行
    savu(u.u_rsav);
    // 执行 retu() 切换至调度器进程，proc[0] 是供调度器使用的系统进程
       // 在系统启动时被创建
    retu(proc[0].p_addr);
}

loop:
    runrun = 0;
    rp = p;
    p = NULL;
    n = 128;

    i = NPROC;

    /*
    遍历 proc[]，寻找满足下述条件的进程：

        1. 可执行状态 SRUN
        2. 存在于内存中 SLOAD
        3. 执行优先级最高，proc.p_pri 具有最小值
    */
    do {
        rp++;
        if (rp >= &proc[NPROC]) {
            rp = &proc[0];
        if (rp->p_stat == SRUN && (rp->p_flag&SLOAD) != 0) {
            if (rp->p_pri<n) {
                p = rp;
                n = rp->p_pri;
            }
        }
        } while (--i)

    // 如果不存在这样的进程则调用 idle()，等待发生中断而出现的可执行进程
    if (p ==NULL) {
        p = rp;
        idle();
        goto loop;
    }

    // 有可执行的进程
    rp = p;
    curpri = n;

    retu(rp->p_addr);
    sureg();
        /*
    如果SSWAP标识位为1，将其清0并从user.u_ssav中恢复 r5, r6 的值，之所以这样做
    是因为当由于调度器以外的原因导致数据被换出内存时，将SSWAP标识为设置为1。

    这意味着首先将调用xswap()进行换出处理，随后执行swtch()切换执行进程。
    在 swtch() 的第 10 行执行 savu(u.u_rsav) 时，因为 user 结构体已被换出至交换空间
    所以此处的 savu() 将失去意义，所以解决办法是在叫唤处理前首先执行 savu(u.u_rsav)
    */
    if (rp->p_flag&SSWAP) {
        rp->p_flag = & ~SSWAP;
        aretu(u.u_ssav);
    }

    return(1);
    }
```