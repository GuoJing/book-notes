### New Proc

[http://minnie.tuhs.org/cgi-bin/utree.pl?file=V6/usr/sys/ken/slp.c](http://minnie.tuhs.org/cgi-bin/utree.pl?file=V6/usr/sys/ken/slp.c)

```
newproc()
{
    int a1, a2;
    struct proc *p, *up;
    register struct proc *rpp;
    register *rip, n;

    p = NULL;
    /*
     * First, just locate a slot for a process
     * and copy the useful info from this process into it.
     * The panic "cannot happen" because fork has already
     * checked for the existence of a slot.
     */
retry:
    mpid++;
    if(mpid < 0) {
        mpid = 0;
        goto retry;
    }
    for(rpp = &proc[0]; rpp < &proc[NPROC]; rpp++) {
        if(rpp->p_stat == NULL && p==NULL)
            p = rpp;
        if (rpp->p_pid==mpid)
            goto retry;
    }
    if ((rpp = p)==NULL)
        panic("no procs");

    /*
     * make proc entry for new proc
     */

    rip = u.u_procp;
    up = rip;
    rpp->p_stat = SRUN;
    rpp->p_flag = SLOAD;
    rpp->p_uid = rip->p_uid;
    rpp->p_ttyp = rip->p_ttyp;
    rpp->p_nice = rip->p_nice;
    rpp->p_textp = rip->p_textp;
    rpp->p_pid = mpid;
    rpp->p_ppid = rip->p_pid;
    rpp->p_time = 0;

    /*
     * make duplicate entries
     * where needed
     */

    for(rip = &u.u_ofile[0]; rip < &u.u_ofile[NOFILE];)
        if((rpp = *rip++) != NULL)
            rpp->f_count++;
    if((rpp=up->p_textp) != NULL) {
        rpp->x_count++;
        rpp->x_ccount++;
    }
    u.u_cdir->i_count++;
    /*
     * Partially simulate the environment
     * of the new process so that when it is actually
     * created (by copying) it will look right.
     */
    savu(u.u_rsav);
    rpp = p;
    u.u_procp = rpp;
    rip = up;
    n = rip->p_size;
    a1 = rip->p_addr;
    rpp->p_size = n;
    a2 = malloc(coremap, n);
    /*
     * If there is not enough core for the
     * new process, swap out the current process to generate the
     * copy.
     */
    if(a2 == NULL) {
        rip->p_stat = SIDL;
        rpp->p_addr = a1;
        savu(u.u_ssav);
        xswap(rpp, 0, 0);
        rpp->p_flag =| SSWAP;
        rip->p_stat = SRUN;
    } else {
    /*
     * There is core, so just copy.
     */
        rpp->p_addr = a2;
        while(n--)
            copyseg(a1++, a2++);
    }
    u.u_procp = rip;
    return(0);
}
```