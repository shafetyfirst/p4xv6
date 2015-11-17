#include "types.h"
#include "stat.h"
#include "fcntl.h"
#include "user.h"
#include "x86.h"

#define PGSIZE (4096)
struct lock_t {
  uint locked;       // Is the lock held?

  // For debugging:
  char *name;        // Name of lock.
  //struct cpu *cpu;   // The cpu holding the lock.
  uint pcs[10];      // The call stack (an array of program counters)
                     // that locked the lock.
};

int
thread_create(void (*start_routine)(void*), void *arg)
{
  void *stack = malloc(PGSIZE);

  int pid;

   if(stack == NULL){
     printf(1, "thread create error");
   }
  if((uint)stack % PGSIZE)
    stack = stack + (4096 - (uint)stack % PGSIZE);

  pid = clone(start_routine, arg, stack);

  return pid;
}

int
thread_join(void)
{
  void *stack = malloc(PGSIZE);
  int pid;  
  if(stack == NULL){
     printf(1, "join error");
  }
  if((uint)stack % PGSIZE)
    stack = stack + (4096 - (uint)stack % PGSIZE);

  pid = join(&stack);
  
  free(stack);
  
  return pid;
  
}

void
lock_init(struct lock_t *lk, char *name)
{
  lk->name = name;
  lk->locked = 0;
//  lk->cpu = 0;
}

// Acquire the lock.
// Loops (spins) until the lock is acquired.
// Holding a lock for a long time may cause
// other CPUs to waste time spinning to acquire it.
void
lock_acquire(struct lock_t *lk)
{
 // pushcli(); // disable interrupts to avoid deadlock.
/*
  if(holding(lk))
    panic("acquire");
*/
  // The xchg is atomic.
  // It also serializes, so that reads after acquire are not
  // reordered before it. 
  while(xchg(&lk->locked, 1) != 0)
    ;

  // Record info about lock acquisition for debugging.
//  lk->cpu = cpu;
// getcallerpcs(&lk, lk->pcs);
}

// Release the lock.
void
lock_release(struct lock_t *lk)
{
/*
  if(!holding(lk))
    panic("release");
*/
  lk->pcs[0] = 0;
//  lk->cpu = 0;

  // The xchg serializes, so that reads before release are 
  // not reordered after it.  The 1996 PentiumPro manual (Volume 3,
  // 7.2) says reads can be carried out speculatively and in
  // any order, which implies we need to serialize here.
  // But the 2007 Intel 64 Architecture Memory Ordering White
  // Paper says that Intel 64 and IA-32 will not move a load
  // after a store. So lock->locked = 0 would work here.
  // The xchg being asm volatile ensures gcc emits it after
  // the above assignments (and after the critical section).
  xchg(&lk->locked, 0);

 // popcli();
}
/*
// Record the current call stack in pcs[] by following the %ebp chain.
void
getcallerpcs(void *v, uint pcs[])
{
  uint *ebp;
  int i;
  
  ebp = (uint*)v - 2;
  for(i = 0; i < 10; i++){
    if(ebp == 0 || ebp < (uint*)0x100000 || ebp == (uint*)0xffffffff)
      break;
    pcs[i] = ebp[1];     // saved %eip
    ebp = (uint*)ebp[0]; // saved %ebp
  }
  for(; i < 10; i++)
    pcs[i] = 0;
}

// Check whether this cpu is holding the lock.
int
holding(struct sweetLock *lock)
{
  return 1;//lock->locked && lock->cpu == cpu;
}


// Pushcli/popcli are like cli/sti except that they are matched:
// it takes two popcli to undo two pushcli.  Also, if interrupts
// are off, then pushcli, popcli leaves them off.

void
pushcli(void)
{
  int eflags;
  
  eflags = readeflags();
  cli();
  if(cpu->ncli++ == 0)
    cpu->intena = eflags & FL_IF;
}

void
popcli(void)
{
  if(readeflags()&FL_IF)
    panic("popcli - interruptible");
  if(--cpu->ncli < 0)
    panic("popcli");
  if(cpu->ncli == 0 && cpu->intena)
    sti();
}
*/
