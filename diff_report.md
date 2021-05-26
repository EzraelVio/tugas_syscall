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
```

```diff
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
```

```diff
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

## proc.c
```diff
#include "types.h"
#include "defs.h"
#include "param.h"
#include "memlayout.h"
#include "mmu.h"
#include "x86.h"
#include "proc.h"
#include "spinlock.h"

#ifdef CS333_P2
  #include "uproc.h"
#endif
```
```diff
allocproc(void)
{
  struct proc *p;
  char *sp;

  acquire(&ptable.lock);
  int found = 0;
  for(p = ptable.proc; p < &ptable.proc[NPROC]; p++)
    if(p->state == UNUSED) {
      found = 1;
      break;
    }
  if (!found) {
    release(&ptable.lock);
    return 0;
  }
  p->state = EMBRYO;
  p->pid = nextpid++;
  release(&ptable.lock);

  // Allocate kernel stack.
  if((p->kstack = kalloc()) == 0){
    p->state = UNUSED;
    return 0;
  }
  sp = p->kstack + KSTACKSIZE;

  // Leave room for trap frame.
  sp -= sizeof *p->tf;
  p->tf = (struct trapframe*)sp;

  // Set up new context to start executing at forkret,
  // which returns to trapret.
  sp -= 4;
  *(uint*)sp = (uint)trapret;

  sp -= sizeof *p->context;
  p->context = (struct context*)sp;
  memset(p->context, 0, sizeof *p->context);
  p->context->eip = (uint)forkret;

  p->start_ticks = ticks;

 + #ifdef CS333_P2
 +   p->cpu_ticks_total = 0;
 +   p->cpu_ticks_in = 0;
 + #endif // CS333_P2

  return p;
}
```


