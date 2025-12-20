
## 自陷

实现指令
```
80001410:   30571073            csrw    mtvec,a4
```

其在 `abstract-machine/am/src/$ISA/nemu/`

```c
bool cte_init(Context*(*handler)(Event, Context*)) {
  // initialize exception entry
  asm volatile("csrw mtvec, %0" : : "r"(__am_asm_trap));

  // register event handler
  user_handler = handler;

  return true;
}
```
定义。

我们需要增加对这条指令的支持。

各指令的 `opcode` 固定位 `1110011` `0x73`

**各指令的 funct3 和 opcode 值**  
CSRRW（CSR Read and Write）  
funct3：`001`  
opcode：`1110011`（`0 x73 `）

CSRRS（CSR Read and Set）  
funct3：`010`  
opcode：`1110011`（`0 x73 `）

CSRRC（CSR Read and Clear）  
funct3：`011`  
opcode：`1110011`（`0 x73 `）

CSRRWI（CSR Read and Write Immediate）  
funct3：`101`  
opcode：`1110011`（`0 x73 `）

CSRRSI（CSR Read and Set Immediate）  
funct3：`110`  
opcode：`1110011`（`0 x73 `）

CSRRCI（CSR Read and Clear Immediate）  
funct3：`111`  
opcode：`1110011`（`0 x73 `）

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251110231633.png)

前面 `31-20` 标志访问的控制和状态寄存器。（12 位立即数）

手册  *The RISC-V Instruction Set Manual: Volume II: PrivilegedArchitecture* 可能有用。pp17


- **8**: Environment call from U-mode（用户模式ecall）
    
- **9**: Environment call from S-mode（监管模式ecall）
    
- **11**: Environment call from M-mode（机器模式ecall）
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251111003127.png)



```
[src/isa/riscv32/system/intr.c:24 isa_raise_intr] MTVEC: 0x80001438
// nemu 的log

阅读elf符号表

189: 80001438     0 NOTYPE  GLOBAL DEFAULT    1 __am_asm_trap
```

应当没问题。

- **再理一下**：

```c

#define CONTEXT_SIZE  ((NR_REGS + 3) * XLEN)
#define OFFSET_SP     ( 2 * XLEN)
#define OFFSET_CAUSE  ((NR_REGS + 0) * XLEN)
#define OFFSET_STATUS ((NR_REGS + 1) * XLEN)
#define OFFSET_EPC    ((NR_REGS + 2) * XLEN)
```
对于我们的 `riscv32`，这里的 `XLEN = 4`


```c
// abstract-machine/am/src/riscv/nemu/trap.S
__am_asm_trap:
  addi sp, sp, -CONTEXT_SIZE

  MAP(REGS, PUSH)

  csrr t0, mcause
  csrr t1, mstatus
  csrr t2, mepc

  STORE t0, OFFSET_CAUSE(sp)
  STORE t1, OFFSET_STATUS(sp)
  STORE t2, OFFSET_EPC(sp)

  # set mstatus.MPRV to pass difftest
  li a0, (1 << 17)
  or t1, t1, a0
  csrw mstatus, t1

  mv a0, sp
  call __am_irq_handle

  LOAD t1, OFFSET_STATUS(sp)
  LOAD t2, OFFSET_EPC(sp)
  csrw mstatus, t1
  csrw mepc, t2

  MAP(REGS, POP)

  addi sp, sp, CONTEXT_SIZE
  mret
```
- 汇编代码函数 `__am_asm_trap` 做了：
### 1. 保存现场 (Saving Context)

在 CPU 从用户态或 Supervisor 态陷入（Trap）到 Machine 态（M-Mode）时，首先要做的是保存当前任务的状态，以便之后能正确恢复。
- `addi sp, sp, -CONTEXT_SIZE`: **在栈上分配空间**，用于保存完整的 CPU 上下文。
- `MAP(REGS, PUSH)`: 这是一个宏，用于将所有的 **通用寄存器（GPRs，即 $x1$ 到 $x31$）** 依次推入栈中。
- `csrr t0, mcause`: 读取 **`mcause`**（异常/中断原因）寄存器。
- `csrr t1, mstatus`: 读取 **`mstatus`**（机器状态）寄存器。
- `csrr t2, mepc`: 读取 **`mepc`**（异常发生时的程序计数器）寄存器。
- `STORE ...`: 将上面读取的这三个重要的 **控制和状态寄存器（CSRs）** 的值保存到栈上的上下文结构体中。
### 2. 处理中断 (Handling Interrupt)
保存完所有硬件状态后，将控制权转移给高级语言（C 语言）编写的中断处理函数。
- `# set mstatus.MPRV to pass difftest`: 这一段是针对特定调试环境（如 NEMU 或 Difftest）的**环境设置**。它将 `mstatus` 寄存器的 **`MPRV`**（Memory Privilege）位设置为 1，这通常是为了让 M-Mode 代码能以低权限（如 S-Mode 或 U-Mode）的权限访问内存，便于调试和测试。
- `mv a0, sp`: 将当前栈指针 `sp`（它指向保存的上下文结构体的起始地址）作为 **第一个参数** 放入 `a0` 寄存器。
- `call __am_irq_handle`: **调用 C 语言的中断处理函数**。这个 C 函数（在 Abstract Machine 中）会根据 `mcause` 判断是哪种中断或异常，并执行相应的处理逻辑（如系统调用分发、时钟中断处理等）。

### 3. 恢复现场并返回 (Restoring Context and Return)
C 函数处理完毕后，将恢复 CPU 状态，使任务从陷阱点继续执行。
- `LOAD t1, OFFSET_STATUS(sp)` / `LOAD t2, OFFSET_EPC(sp)`: 从栈上加载之前保存的 `mstatus` 和 `mepc` 的值。
- `csrw mstatus, t1` / `csrw mepc, t2`: **写回** `mstatus` 和 `mepc` 寄存器。
- `MAP(REGS, POP)`: **恢复** 所有 **通用寄存器（GPRs）** 的值。
- `addi sp, sp, CONTEXT_SIZE`: **释放** 栈上分配的上下文空间。
- `mret`: **机器模式返回指令**（Machine Return）。它会将程序计数器（PC）设置为 `mepc` 的值，并根据 `mstatus` 中的 `MPIE` 和 `MPP` 位来设置中断使能和切换到正确的特权模式（通常是 S-Mode 或 U-Mode），从而使被中断的任务得以继续运行。

看看 Context 的结构：
```c
struct Context {
   // TODO: fix the order of these members to match trap.S
   // uintptr_t mepc, mcause, gpr[NR_REGS], mstatus;

   // new order to match trap.S
   uintptr_t gpr[NR_REGS],mcause, mstatus, mepc; 
   void *pdir;
};
```

这里参照 `trap.S` 修改了结构体的顺序。
什么是现场呢？
- 我们需要所有寄存器的值，包括 `gpr` 和 `csrs` 的值
- 这就是 `Context` 的内容

Event 的内容
```c
// An event of type @event, caused by @cause of pointer @ref
typedef struct {
  enum {
    EVENT_NULL = 0,
    EVENT_YIELD, EVENT_SYSCALL, EVENT_PAGEFAULT, EVENT_ERROR,
    EVENT_IRQ_TIMER, EVENT_IRQ_IODEV,
  } event;
  uintptr_t cause, ref;
  const char *msg;
} Event;
```




- `yield` 函数的汇编相当简单：
```asm
8000147c <yield>:
8000147c:   fff00893            li  a7,-1
80001480:   00000073            ecall
80001484:   00008067            ret
```


- `abstract-machine/am/src/riscv/nemu/cte.c` 中是我们的异常处理的重要 src
```c
Context* __am_irq_handle(Context *c) {
  display_context(c);
  if (user_handler) {
   Event ev = {0};
    switch (c->mcause) {
     default: ev.event = EVENT_ERROR; break;
    }
   c = user_handler(ev, c);
   assert(c != NULL);
  }
  return c;
 }
```

这个函数中，需要我们对 `Context` 的 `mcause` 进行处理，传给 `event`。需要阅读手册的 `mcause` 部分

```c
// event的定义
// am/include/am.h 
typedef struct {
  enum {
    EVENT_NULL = 0,
    EVENT_YIELD, EVENT_SYSCALL, EVENT_PAGEFAULT, EVENT_ERROR,
    EVENT_IRQ_TIMER, EVENT_IRQ_IODEV,
  } event;
```



- **实现新指令**：
	- 实现 Trap-Return 指令
	- `mret` is always provided
	- To return after handling a trap.
> [!Note] 手册中对 `mret` 的说明
> - The MRET instruction is used to return from a **trap taken** into M-mode. MRET first determines what the **new privilege mode will be** according to the values of MPP and MPV in **mstatus or mstatush**, as encoded in Table 35. MRET then in mstatus/mstatush sets MPV=0, MPP=0, MIE=MPIE, and MPIE=1. Lastly, **MRET sets the privilege mode as previously determined, and sets pc=mepc.**

我们 PA 现在还不关心状态。直接改 PC 到 `mepc` 即可


在 *The RISC-V Instruction Set Manual: Volume II: PrivilegedArchitecture*  的 164 页可以查到指令格式。


添加几条指令之后，正常通过 `yield test`

**新指令**如下
```c
	INSTPAT("0000000 00000 00000 000 00000 11100 11", 	ecall,   I,
		   bool ok=true; 
	          // word_t _mcause = isa_reg_str2val("a7",&ok);
		   // 0xb is Environment call from M-mode
		   if(ok) s->dnpc = isa_raise_intr(0xb, s->pc));  
	// mret is a fixed inst
	// but it use the last 7bit act as func1, so it's more like a system-R type;
	INSTPAT("0011000 00010 00000 000 00000 11100 11",   mret,    R,		
		   // recover the pc to mepc;
		   s->dnpc = *CSR_MEPC
	);
	   INSTPAT("??????? ????? ????? 001 ????? 11100 11", 	csrrw,	 I, 	R(rd) = CSR(imm); CSR(imm) = src1);
	   INSTPAT("??????? ????? ????? 010 ????? 11100 11", 	csrrs,	 I, 	R(rd) = CSR(imm); CSR(imm) |= src1);

```


### 必答题：理解 `yield test` 的行为：发生了什么？

在 `am-tests` 的主函数中，执行 `yield test` 的分支
其除了注册 `IOE`，还注册了 `CTE`
```c
CASE('i', hello_intr, IOE, CTE(simple_trap));
```

```c
#define CTE(h) ({ Context *h(Event, Context *); cte_init(h); })
// 这个带参数宏用`simple_trap`注册上下文环境。
```
`CTE(simple_trap)` 用 `simple_trap` 调用 `cte_init(h)` 注册了上下文环境，并将 `simple_trap` 作为出现异常的函数，执行**汇编代码段函数**(`__am_asm_trap` )

这段函数会保存现场，调用 `__am_irq_handle` 进行异常处理。然后再恢复现场并返回。


```c
bool cte_init(Context*(*handler)(Event, Context*)) {
   // initialize exception entry
   asm volatile("csrw mtvec, %0" : : "r"(__am_asm_trap));

   // register event handler
   user_handler = handler;

   return true;
 }
```


熟悉一下 `Context` 的组织
```c
struct Context {
   // uintptr_t mepc, mcause, gpr[NR_REGS], mstatus;
   // new order to match trap.S
   uintptr_t gpr[NR_REGS],mcause, mstatus, mepc; 
   void *pdir;
};
```

`Context` 即一个存储上下文的结构体：
什么能体现程序某一刻的状态呢？
- **所有寄存器的信息**
- `void *pdir`
	- **PDIR** 是 **Page DIRectory** 的缩写，它存储了当前 **进程的页表基地址**。
	- 在 RISC-V 架构中，当 **S-mode** (Supervisor Mode) 或 **U-mode** (User Mode) 启用 **虚拟内存** 时，内核需要将这个地址加载到 **`satp` 寄存器**（Supervisor Address Translation and Protection Register）。
	- **我们暂时不关心**

这里的 `simple_trap` 是 `tests/intr.c` 中的一个函数
```c
Context *simple_trap(Event ev, Context *ctx)
```
其会对 `ctx` 做一些修改，然后返回这个 `ctx`。根据 `Event ev` 的不同事件，会回调不同的分支 (`yield` 就是 `putch('y')` 到输出) 


`yield test` 接下来的代码在 `am/tests/am-tests/src/tests/intr.c` 中，其在 `hello_intr()` 中循环调用 `yield()`

```c
while (1) {
	for (volatile int i = 0; i < 10000000; i++) ;
    	yield();
}
```

`yield` 实际写作一个汇编代码
```c
asm volatile("li a7, -1; ecall");
```


其首先加载一个 `-1`, 到 `a7`，然后调用 `ecall`， 
**解释**：在我们的约定中，用 `a7` 寄存器即 `x17` 作为存储**异常号**的寄存器。

`NEMU` 作为硬件，会执行这两条汇编指令，

```c
INSTPAT("0000000 00000 00000 000 00000 11100 11",   ecall,   I,
        bool ok=true;
		word_t NO = isa_reg_str2val("a7",&ok);
        if(ok) s->dnpc = isa_raise_intr(NO, s->pc));
 // mret is a fixed inst, all imm is set as 0;
 // but it used the last 7bit act as func1, so it's more like a system-R type;
```

其根据寄存器 `a7` 获得异常号，并完成下一条指令的设置。

我们按部分理了一下，现在按时间顺序理一下：

- **中断前准备**
	- `yield-test` 注册上下文环境，绑定回调函数(这里是输出 `y` 的 `simple_trap`)，设置异常处理 `trap`
		- `asm ("csrw mtvec, %0" : : "r"(__am_asm_trap))`
			- 这设置了 `mtvec` 是函数 `__am_asm_trap` 的地址
---
- 调用 `ecall` 中断 (`NEMU` 层处理)
	- 完成上述准备后，我们在 `AM` 层调用了 `yield()`，这个函数实际是汇编的，其将异常号 `mcause` 装入 `a7` 寄存器，接着执行 `ecall`
	- `NEMU` 层将异常号写入 `csr`，记录异常 pc，并跳转到 `mtvec` (machine trap-vector base-address register)处
	- 跳转到 `mtvec` 即设置好的 `trap` 处。
---
- **中断处理**
	- 跳转到 `trap` 处就开始真正中断处理（`__am_asm_trap`）进行处理
	- 保存现场 (即保存 `csr` 各项数据到栈上)
	- 执行回调函数 `__am_irq_handle`
		- 其根据上下文 `Context *c` 的内容，设置 `event` 。
		- 我们约定 `-1` 是 `yield`，可以从 `yield` 函数的 `li` 指令了解到。
			- `case -1: ev.event = EVENT_YIELD; break;`
		- 对于本测试来说，我们绑定的回调函数是 `simple_trap`，其会根据我们的 `EVENT` 类型输出字符，这里是 `y`
	- 恢复现场。


## Nano-lite

> [!TODO] 添加第一个事件分发
> 在 `src/irq.c` 添加如下函数即可
> ```c
> static Context* do_event(Event e, Context* c) {
>   switch (e.event) {
> 	case EVENT_YIELD: printf("Handle event ID = %d, yiled!",e.event); break;
>     default: panic("Unhandled event ID = %d", e.event);
>   }
>   return c;
> }
> ```



### 加载第一个用户程序
> [!Note] 阅读 elf
> 之前，我们在 PA2 和 ELF 的交手，主要是从链接的角度，阅读 `section`。
> 但实际上，elf 也提供了 ` segment ` 视角，从程序装载的角度。二者存在一个映射。
> ```
> There are no section groups in this file.
> 
> Program Headers:
>   Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
>   RISCV_ATTRIBUT 0x009a8b 0x00000000 0x00000000 0x00023 0x00000 R   0x1
>   LOAD           0x001000 0x80000000 0x80000000 0x016f9 0x016f9 R E 0x1000
>   LOAD           0x0026fc 0x800016fc 0x800016fc 0x07364 0x0fd0c RW  0x1000
> Section to Segment mapping:
> 
>  Segment Sections...
>   00     .riscv.attributes
>   01     .text .rodata .srodata.__func__.0
>   02     .data .sdata.heap .bss
> ```
> 我们可以通过判断segment的 `Type` 属性是否为 `PT_LOAD` 来判断一个segment是否需要加载.

Elf 文件在 ramdisk 中，框架代码提供了一些 API
```
// 从ramdisk中`offset`偏移处的`len`字节读入到`buf`中
size_t ramdisk_read(void *buf, size_t offset, size_t len);

// 把`buf`中的`len`字节写入到ramdisk中`offset`偏移处
size_t ramdisk_write(const void *buf, size_t offset, size_t len);

// 返回ramdisk的大小, 单位为字节
size_t get_ramdisk_size();
```


> [!Question] 如何 assert 一个 `ELF` 文件？
> ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251119151034.png)
>  `ELF` 文件提供了 `Magic number` 来完成这件事。
>  我们是小端机。所以存储字节为 `0x464c457f`



> [!Question] 为什么要清零？
> 为什么需要将 `[VirtAddr + FileSiz, VirtAddr + MemSiz)` 对应的物理区间清零?
> - 回答：`man 5 elf`,  手册里有描述
> If the segment's memory  size  `p_memsz`  is  larger than  the file size `p_filesz`, the "extra" bytes are defined to hold the value 0 and to follow the segment's initialized area. 
> `MemSiz` 一般大于等于 `FileSiz`，这部分 `[VirtAddr + FileSiz, VirtAddr + MemSiz)` 映射到 `.bss` 
> `.bss` 部分 全局变量等默认为 0，其不需要让上层一个个遍历内存来做，只需要约定好：把这一块设置为 0 即可
> **示例**：
> ```
> 内存地址 (VirtAddr)
> 0x03000000  +---------------------------+
>             |  .text / .data            |  <-- 这一部分从 ELF 文件里拷贝
>             |  (代码和已初始化变量)       |      (长度 = FileSiz)
> 0x0301D600  +---------------------------+
>             |  .bss                     |  <-- 这一部分由系统填充 0
>             |  (未初始化变量)             |      (长度 = MemSiz - FileSiz)
> 0x03027240  +---------------------------+
> ```



在 `loader` 中，我们最后返回 `e_entry`
> `e_entry`
> This  member  gives  the virtual address to which the system first transfers control, thus starting the process.  If the file has no associated entry point, this member holds zero.



### 系统调用

我们首先需要查看系统调用的寄存器约定：`man syscall`
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251125102901.png)

The  first  table lists the instruction used to transition to kernel mode (which might not be the fastest or best way to transition to the kernel, so you might have to refer to `vdso (7)`), the register used to indicate the system call number, the register (s) used to return the system call result, and the register used to signal an error.


再来看看系统调用函数 `syscall` 的内容
```c
intptr_t _syscall_(intptr_t type, intptr_t a0, intptr_t a1, intptr_t a2) { 
 register intptr_t _gpr1 asm (GPR1) = type;
 register intptr_t _gpr2 asm (GPR2) = a0;
 register intptr_t _gpr3 asm (GPR3) = a1;
 register intptr_t _gpr4 asm (GPR4) = a2;
 register intptr_t ret asm (GPRx);
 asm volatile (SYSCALL : "=r" (ret) : "r"(_gpr1), "r"(_gpr2), "r"(_gpr3), "r"(_gpr4));
 return ret;
}
```

我们根据手册和 syscall 的提示，设定 riscv 的各个寄存器的下标。

#### 新系统事件的注册（AM 层）

```c
// abstract-machine/am/include/am.h
typedef struct {
   enum {
     EVENT_NULL = 0,
     EVENT_YIELD, EVENT_SYSCALL, EVENT_PAGEFAULT, EVENT_ERROR,
     EVENT_IRQ_TIMER, EVENT_IRQ_IODEV,
   } event;
   uintptr_t cause, ref;
   const char *msg;
 } Event;
```

#### 事件处理分发（AM 层）
`abstract-machine/am/src/riscv/nemu/cte.c`
有
```c
Context* __am_irq_handle(Context *c) {
   display_context(c);
   if (user_handler) {
    Event ev = {0};
     switch (c->mcause) {
      case -1: ev.event = EVENT_YIELD; break;
      default: ev.event = EVENT_ERROR; break;
     }

    c = user_handler(ev, c);
    assert(c != NULL);
   }

   return c;
 }

```

#### 系统调用（Nano-lite 层）

```c
void do_syscall(Context *c) {
   uintptr_t a[4];
   a[0] = c->GPR1;

   switch (a[0]) {
     default: panic("Unhandled syscall ID = %d", a[0]);
   }
}
```
我们在 Nano-lite 中提取系统调用号。

修改的路径：
- 在 `abstract-machine` 的 `cte` 模块添加新的 SYSCALL 支持，这里 `c->mcause =1` 时，设置 `EVENT_SYSCALL`。增加对这个事件的分发
- AM 设置事件完成，在 `nanos-lite` 中接受到这个事件，增加对应的处理

```c
void do_syscall(Context* c)
{
    printf("do syscall\n");
    uintptr_t a[4];
    a[0] = c->GPR1;
    a[1] = c->GPR2;
    a[2] = c->GPR3;
    a[3] = c->GPR4;
 14
    switch (a[0])
    {
    case SYS_yield:
        sys_yield(c);
        break;
    default:
        panic("Unhandled syscall ID = %d", a[0]);
    }
}

```

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251125203208.png)

在循环执行。说明 pc 可能没有恢复好。

> 对于 mips 和 risc-v, mepc保存的是自陷指令的PC, 因此软件需要在适当的地方对保存的PC加上4, **使得将来返回到自陷指令的下一条指令.**

由于我们尽可能想实现**硬件无关**的设计，所以我们不在 `NEMU` 层做修改，而是在 `AM` 层，也就是 `am/src/riscv/nemu/cte.c` 层做出 ISA 层面的修改即可
```c
Context* __am_irq_handle(Context *c) {
	display_context(c);

   // we deal the dnpc here. for riscv-v;
	c->mepc +=4;
	
	/* more code */
}
```

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251125210140.png)
- Now it's ok.
- Then we have to deal with another syscall. its mcause is `0x0`, read the friendly manuel, it's a `SYS_exit`
- 实际上，`SYS_exit` 应该接收退出状态的参数. 为了方便测试，这里先用 `halt()`

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251125215914.png)
- 之前铸币了，忘记在 `nano-lite` 中调用 `do_syscall` 了

- 总结调用链：
	- 程序触发 `syscall`，保存现场到 `csr` 寄存器， 控制权交给系统，即 ` nano-lite `
	- `AM` 读取寄存器数据，根据 ` mcause ` 分发不同的事件。
	- 操作系统层 `nanos-lite` 处理事件 (`do_event`) ，根据不同的事件号分发不同的处理。在 `EVENT_SYSCALL` 分支执行 `do_syscall` 来进行进一步处理
	- `do_syscall` 会利用 `GPR?` 宏读取 `Context` 的值。通过第一个参数 `a[0] = c->GPR1` 来获得具体的 `syscall` 类型。执行不同的 `sys_xxx` 操作。
	- 设定返回值给用户。
- 结束中断处理后，恢复现场，继续执行。



#### Implement strace
这个很简单。追踪系统调用，而我们的系统调用都是在 `nanos-lite` 中的 `do_syscall` 解决的。在这里追踪一下就好了

我定义了一个字符串数组，这样能追踪名字，可读性更好。
> [!Note] 用 Vim 快速从 `enum` 获取这样一个字符数组
> 题外话：从枚举类型快速获得字符数组实际就是每一项左右添加 `"` 即可。用 vim 命令很容易实现
> `:'<,'>s/\(^[^,]*\)/"\1"/`



```c
// nanos-lite/src/syscall.c
#ifdef CONFIG_STRACE
	printf("STRACE: syscall ID = %d, syscall name is %s\n", a[0], sys2str[a[0]])      ;
#endif

```