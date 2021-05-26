# Kode yang sudah dirubah

## Makefile
```diff
-CS333_PROJECT ?= 1
+CS333_PROJECT ?= 2
PRINT_SYSCALLS ?= 0
CS333_CFLAGS ?= -DPDX_XV6
ifeq ($(CS333_CFLAGS), -DPDX_XV6)
CS333_UPROGS +=	_halt _uptime
endif

ifeq ($(CS333_PROJECT), 2)
CS333_CFLAGS += -DCS333_P1 -DUSE_BUILTINS -DCS333_P2
-CS333_UPROGS += _date _time _ps
+CS333_UPROGS += _date _ps _time
CS333_TPROGS += _testsetuid  _testuidgid _p2-test
endif
```

## defs.h
```diff
+#ifdef CS333_P2
+#include "uproc.h"
+#endif

struct buf;
struct context;
struct file;
struct inode;
struct pipe;
struct proc;
struct rtcdate;
struct spinlock;
struct sleeplock;
struct stat;
struct superblock;

// proc.c
int             cpuid(void);
void            exit(void);
int             fork(void);
int             growproc(int);
int             kill(int);
struct cpu*     mycpu(void);
struct proc*    myproc();
void            pinit(void);
void            procdump(void);
void            scheduler(void) __attribute__((noreturn));
void            sched(void);
void            setproc(struct proc*);
void            sleep(void*, struct spinlock*);
void            userinit(void);
int             wait(void);
void            wakeup(void*);
void            yield(void);

+#ifdef CS333_P2
+int             getprocs(uint max, struct uproc* upTable);
+#endif

#ifdef CS333_P3
void            printFreeList(void);
void            printList(int);
void            printListStats(void);
#endif // CS333_P3
```
