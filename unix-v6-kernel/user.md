### User

[http://minnie.tuhs.org/cgi-bin/utree.pl?file=V6/usr/sys/user.h](http://minnie.tuhs.org/cgi-bin/utree.pl?file=V6/usr/sys/user.h)

User 用于管理当前打开的进程和目录。由于内核只需要当前执行进程的 user 结构体，因此当进程被换出至交换空间的时候，对应的 user 也会被交换出去。

```
/*
 * The user structure.
 * One allocated per process.
 * Contains all per process data
 * that doesn't need to be referenced
 * while the process is swapped.
 * The user block is USIZE*64 bytes
 * long; resides at virtual kernel
 * loc 140000; contains the system
 * stack per user; is cross referenced
 * with the proc structure for the
 * same process.
 */
```

```
/*
关于 effective user_id 和 real user_id
执行操作的时候，例如 sudo -usa bash
effective user_id is sa
real user_id is guojing
*/
```

```
struct user
{
    int u_rsav[2];
    /* save r5,r6 when exchanging stacks （进程切换保存至 r5, r6） */
    int u_fsav[25];                 /* save fp registers */
                                    /* rsav and fsav must be first in structure */
    char    u_segflg;               /* flag for IO; user or kernel space */
    char    u_error;                /* return error code */
    char    u_uid;                  /* effective user id */ sudo sa，sa is effective user
    char    u_gid;                  /* effective group id */ sa group is effective group
    char    u_ruid;                 /* real user id */ guojing is real user
    char    u_rgid;                 /* real group id */ dev is real user id
    int u_procp;                    /* pointer to proc structure */
    char    *u_base;                /* base address for IO */
    char    *u_count;               /* bytes remaining for IO */
    char    *u_offset[2];           /* offset in file for IO */
    int *u_cdir;                    /* pointer to inode of current directory */
    char    u_dbuf[DIRSIZ];         /* current pathname component */
    char    *u_dirp;                /* current pointer to inode */
    struct  {                       /* current directory entry */
        int u_ino;
        char    u_name[DIRSIZ];
    } u_dent;
    int *u_pdir;                    /* inode of parent directory of dirp */
    int u_uisa[16];                 /* prototype of segmentation addresses */
    int u_uisd[16];                 /* prototype of segmentation descriptors */
    int u_ofile[NOFILE];            /* pointers to file structures of open files */
    int u_arg[5];                   /* arguments to current system call */
    int u_tsize;                    /* text size (*64) */
    int u_dsize;                    /* data size (*64) */
    int u_ssize;                    /* stack size (*64) */
    int u_sep;                      /* flag for I and D separation */
    int u_qsav[2];                  /* label variable for quits and interrupts */
    int u_ssav[2];                  /* label variable for swapping */
    int u_signal[NSIG];             /* disposition of signals */
    int u_utime;                    /* this process user time */
    int u_stime;                    /* this process system time */
    int u_cutime[2];                /* sum of childs' utimes */
    int u_cstime[2];                /* sum of childs' stimes */
    int *u_ar0;                     /* address of users saved R0 */
    int u_prof[4];                  /* profile arguments */
    char    u_intflg;               /* catch intr from sys */
                                    /* kernel stack per user
                                     * extends from u + USIZE*64
                                     * backward not to reach here
                                    */
} u;
```

```
/* u_error codes */
#define EFAULT      106     用户空间和内核空间传送数据失败
#define EPERM       1       当前用户不是超级用户
#define ENOENT      2       指定文件不存在
#define ESRCH       3       信号的目标进程不存在或者已经结束
#define EINTR       4       系统调用处理中对信号做了处理
#define EIO         5       IO错误
#define ENXIO       6       设备编号所示的设备不存在
#define E2BIG       7       系统通过调用 exec 向程序传送了超过512字节的参数
#define ENOEXEC     8       无法识别待执行程序的格式
#define EBADF       9       试图操作未打开的文件，包括只读的
#define ECHILD      10      执行系统调用wait的时候，找不到子进程
#define EAGAIN      11      系统执行fork的时候，无法在 proc[] 中找到空元素
#define ENOMEM      12      试图向进程分配超过可使用容量的内存
#define EACCES      13      没有对文件或目录的访问权限
#define ENOTBLK     15      不是块设备或特殊文件
#define EBUSY       16      执行 mount umount 时，目标对象仍然在使用中
#define EEXIST      17      执行系统调用 link 时对象文件已经存在
#define EXDEV       18      试图对其他设备上的文件创建连接
#define ENODEV      19      设备编号所示的设备不存在
#define ENOTDIR     20      不是目录
#define EISDIR      21      试图对目录进行写入操作
#define EINVAL      22      参数有误
#define ENFILE      23      数组 file[] 溢出
#define EMFILE      24      数组 user.u_ofile[] 溢出
#define ENOTTY      25      不是代表终端的特殊文件
#define ETXTBSY     26      准备加载至代码段的程序文件曾被其他进程当作数据文件
#define EFBIG       27      文件尺寸过大
#define ENOSPC      28      块设备容量不足
#define ESPIPE      29      对管道文件执行了系统调用 seek
#define EROFS       30      试图更新只读块设备上的文件或目录
#define EMLINK      31      文件连接数过多
#define EPIPE       32      损坏的管道文件
```

内核可以通过全局变量 u 访问执行进程的 user 结构体。

conf/m40.s

```
.globl  _u
_u      =   140000
```
