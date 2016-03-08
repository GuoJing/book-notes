### sleep and wakeup

```
/*
 * Give up the processor till a wakeup occurs
 * on chan, at which time the process
 * enters the scheduling queue at priority pri.
 * The most important effect of pri is that when
 * pri<0 a signal cannot disturb the sleep;
 * if pri>=0 signals will be processed.
 * Callers of this routine must be prepared for
 * premature return, and check that the reason for
 * sleeping has gone away.
 */
sleep(chan, pri)
{
    register *rp, s;

    s = PS->integ;
    rp = u.u_procp;
    if(pri >= 0) {
        if(issig())
            goto psig;
        spl6();
        rp->p_wchan = chan;
        rp->p_stat = SWAIT;
        rp->p_pri = pri;
        spl0();
        if(runin != 0) {
            runin = 0;
            wakeup(&runin);
        }
        swtch();
        if(issig())
            goto psig;
    } else {
        spl6();
        rp->p_wchan = chan;
        rp->p_stat = SSLEEP;
        rp->p_pri = pri;
        spl0();
        swtch();
    }
    PS->integ = s;
    return;

    /*
     * If priority was low (>=0) and
     * there has been a signal,
     * execute non-local goto to
     * the qsav location.
     * (see trap1/trap.c)
     */
psig:
    aretu(u.u_qsav);
}
```

```
/*
 * Wake up all processes sleeping on chan.
 */
wakeup(chan)
{
    register struct proc *p;
    register c, i;

    c = chan;
    p = &proc[0];
    i = NPROC;
    do {
        if(p->p_wchan == c) {
            setrun(p);
        }
        p++;
    } while(--i);
}
```