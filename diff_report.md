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

+#ifdef CS333_P2
+  #include "uproc.h"
+#endif

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
 +#endif // CS333_P2

  return p;
}

```
```diff

initproc = p;
  if((p->pgdir = setupkvm()) == 0)
    panic("userinit: out of memory?");
  inituvm(p->pgdir, _binary_initcode_start, (int)_binary_initcode_size);
  p->sz = PGSIZE;
  memset(p->tf, 0, sizeof(*p->tf));
  p->tf->cs = (SEG_UCODE << 3) | DPL_USER;
  p->tf->ds = (SEG_UDATA << 3) | DPL_USER;
  p->tf->es = p->tf->ds;
  p->tf->ss = p->tf->ds;
  p->tf->eflags = FL_IF;
  p->tf->esp = PGSIZE;
  p->tf->eip = 0;  // beginning of initcode.S

+  #ifdef CS333_P2
+    p->uid = DEFAULT_UID;
+    p->gid = DEFAULT_GID;
  +#endif
  ```

```diff
// Copy process state from proc.
  if((np->pgdir = copyuvm(curproc->pgdir, curproc->sz)) == 0){
    kfree(np->kstack);
    np->kstack = 0;
    np->state = UNUSED;
    return -1;
  }
  np->sz = curproc->sz;
  np->parent = curproc;
  *np->tf = *curproc->tf;

+  #ifdef CS333_P2
+    np->uid = curproc->uid;
+    np->gid = curproc->gid;
+  #endif
  ```

```diff
// Switch to chosen process.  It is the process's job
      // to release ptable.lock and then reacquire it
      // before jumping back to us.
#ifdef PDX_XV6
      idle = 0;  // not idle this timeslice
#endif // PDX_XV6
      c->proc = p;
      switchuvm(p);
      p->state = RUNNING;
+      #ifdef CS333_P2
+        p->cpu_ticks_in = ticks;
+      #endif // CS333_P2
      swtch(&(c->scheduler), p->context);
      switchkvm();
```

```diff
sched(void)
{
  int intena;
  struct proc *p = myproc();

  if(!holding(&ptable.lock))
    panic("sched ptable.lock");
  if(mycpu()->ncli != 1)
    panic("sched locks");
  if(p->state == RUNNING)
    panic("sched running");
  if(readeflags()&FL_IF)
    panic("sched interruptible");
  intena = mycpu()->intena;
+  #ifdef CS333_P2
+    p->cpu_ticks_total += (ticks - p->cpu_ticks_in);
+  #endif // CS333_P2
  swtch(&p->context, mycpu()->scheduler);
  mycpu()->intena = intena;
}
```
