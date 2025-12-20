---
本科课程: ICS
author: 231275036-朱晗
date: 2025-12-12
---
# 实验概况
完成了实验要求的所有基础功能，可以正常运行 **仙剑奇侠传**

**我将实验中比较有趣的部分和遇到问题的部分摘录在下面**

# 实验过程

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
> 我们可以通过判断 segment 的 `Type` 属性是否为 `PT_LOAD` 来判断一个 segment 是否需要加载.

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
> `.bss` 部分全局变量等默认为 0，其不需要让上层一个个遍历内存来做，只需要约定好：把这一块设置为 0 即可
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
### 实现自陷

这里遇到了一个 mepc 更新的问题，**在手册中也有提到**。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251125203208.png)

在循环执行。说明 pc 可能没有恢复好。

> 对于 mips 和 risc-v, mepc 保存的是自陷指令的 PC, 因此软件需要在适当的地方对保存的 PC 加上 4, **使得将来返回到自陷指令的下一条指令.**

由于我们尽可能想实现**硬件无关**的设计，所以我们不在 `NEMU` 层做修改，而是在 `AM` 层，也就是 `am/src/riscv/nemu/cte.c` 层做出 ISA 层面的修改即可
```c
Context* __am_irq_handle(Context *c) {
	display_context(c);

   // we deal the dnpc here. for riscv-v;
	c->mepc +=4;
	
	/* more code */
}

```

### Implement strace
这个很简单。追踪系统调用，而我们的系统调用都是在 `nanos-lite` 中的 `do_syscall` 解决的。在这里追踪一下就好了

我定义了一个字符串数组，这样能追踪名字，可读性更好。
> [!Note] 用 Vim 快速从 `enum` 获取这样一个字符数组
> 题外话：从枚举类型快速获得字符数组实际就是每一项左右添加 `"` 即可。用 vim 命令很容易实现
> `:'<,'>s/\(^[^,]*\)/"\1"/`


## 展示你的批处理系统

**实现**`SYS_execve` 系统调用
细节不再赘述。`man execve` 即可。值得注意的是，如果成功，该系统调用是没有返回值的（以免干扰栈，影响打开应用）


- 支持 `NTERM`
	在 `builtin-sh.cpp` 中增加对 `cmd` 的处理即可。
	注意，如果你修改了游戏文件，**你需要重新安装游戏文件**。`make ISA=$ISA fsimg` 来完成修改！


仙剑奇侠传运行发生色彩异常，人物脸色发蓝。而正常人应该是黄色脸

猜测可能是因为颜色映射问题
我们使用的编码是 `AA RR GG BB` 格式，但是 `SDL_Color` 联合体是 `r g b a | val`
我们之前代码直接访问联合体的 `val`，这样等价于颜色是 `RR GG BB AA` 格式，不符合要求。

直观来说，我们应该是将 `AA` 映射到了 BB ，导致发蓝。

修正后，色彩正常
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251212153832.png)

---


# 必答题


###  必答题(需要在实验报告中回答) - 理解上下文结构体的前世今生
> [!Question] 理解上下文结构体的前世今生
> 
> 你会在 `__am_irq_handle()` 中看到有一个上下文结构指针 `c`, `c` 指向的上下文结构究竟在哪里? 这个上下文结构又是怎么来的? 具体地, 这个上下文结构有很多成员, 每一个成员究竟在哪里赋值的? `$ISA-nemu.h`, `trap.S`, 上述讲义文字, 以及你刚刚在NEMU中实现的新指令, 这四部分内容又有什么联系?

**上下文结构**如下定义：

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
	- **我们暂时不关心**

**该上下文结构**在我们切换态的时候（比如中断处理，从用户态要系统态）**负责保存现场**。
- `mcause, mstatus, mepc` 在**中断**处理中
	- `mcause` 在框架代码中始终为 `11, 0xb`，这是 RISC-V约定的 `ecall from m-mode`, 我们具体是在 `NEMU` 执行 `isa_raise_intr()` 而赋值的
	- `mepc` 也交给 `NEMU` 赋值：即机器层面执行指令出现异常/中断，记录下 `mepc，然后跳到` trap 
- **不过这里**我们保存的仍然是寄存器中的值，还没有赋给结构体

我们有一个 `trap.S`, 其完成了对寄存器和 csr 的**压栈**进入结构体，此时 `Context` 开始有血有肉，可以当作**上下文**来工作了。
```asm
addi sp, sp, -CONTEXT_SIZE

MAP(REGS, PUSH)

csrr t0, mcause
csrr t1, mstatus
csrr t2, mepc

STORE t0, OFFSET_CAUSE(sp)
STORE t1, OFFSET_STATUS(sp)
STORE t2, OFFSET_EPC(sp)
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
`NEMU` 作为硬件，会执行这两条汇编指令，

```c
INSTPAT("0000000 00000 00000 000 00000 11100 11",   ecall,   I,
		// 0xb: ecall from m-mode
        s->dnpc = isa_raise_intr(0xb, s->pc));
 // mret is a fixed inst, all imm is set as 0;
 // but it used the last 7bit act as func1, so it's more like a system-R type;
```

我们按部分理了一下，现在按时间顺序理一下：

- **中断前准备**
	- `yield-test` 注册上下文环境，绑定回调函数(这里是输出 `y` 的 `simple_trap`)，设置异常处理 `trap`
		- `asm ("csrw mtvec, %0" : : "r"(__am_asm_trap))`
			- 这设置了 `mtvec` 是函数 `__am_asm_trap` 的地址
---
- 调用 `ecall` 中断 (`NEMU` 层处理)
	- 完成上述准备后，我们在 `AM` 层调用了 `yield()`，接着执行 `ecall`
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
	- 恢复现场



###  必答题(需要在实验报告中回答) - hello程序是什么, 它从而何来, 要到哪里去
> [!Question]
> 到此为止, PA中的所有组件已经全部亮相, 整个计算机系统也开始趋于完整. 你也已经在这个自己创造的计算机系统上跑起了hello这个第一个还说得过去的用户程序 (dummy是给大家热身用的, 不算), 好消息是, 我们已经距离运行仙剑奇侠传不远了(下一个阶段就是啦).
> 
> 不过按照PA的传统, 光是跑起来还是不够的, 你还要明白它究竟怎么跑起来才行. 于是来回答这道必答题吧:
> 
> > 我们知道 `navy-apps/tests/hello/hello.c` 只是一个C源文件, 它会被编译链接成一个ELF文件. 那么, hello程序一开始在哪里? 它是怎么出现内存中的? 为什么会出现在目前的内存位置? 它的第一条指令在哪里? 究竟是怎么执行到它的第一条指令的? hello程序在不断地打印字符串, 每一个字符又是经历了什么才会最终出现在终端上?
> 
> 上面一口气问了很多问题, 我们想说的是, 这其中蕴含着非常多需要你理解的细节. 我们希望你能够认真整理其中涉及的每一行代码, 然后用自己的语言融会贯通地把这个过程的理解描述清楚, 而不是机械地分点回答这几个问题.
> 
> 同样地, 上一阶段的必答题"理解穿越时空的旅程"也已经涵盖了一部分内容, 你可以把它的回答包含进来, 但需要描述清楚有差异的地方. 另外, C库中 `printf()` 到 `write()` 的过程比较繁琐, 而且也不属于PA的主线内容, 这一部分不必展开回答. 而且你也已经在PA2中实现了自己的 `printf()` 了, 相信你也不难理解字符串格式化的过程. 如果你对Newlib的实现感兴趣, 你也可以RTFSC.
> 
> 总之, 扣除C库中 `printf()` 到 `write()` 转换的部分, 剩下的代码就是你应该理解透彻的了. 于是, 努力去理解每一行代码吧!

`hello.c` 作为一个 C 源文件，我们在 `navy-apps` 里编译实际是生成了一个 ELF 镜像，`hello.c` 就在其中

之后，我们可以选择 `cp` 这个镜像到 `nanos-lite/build/ramdisk.img` （在实现文件系统之后，也可以通过 `fopen` 来打开）

`ramdisk.img` 有一个 `ramdisk_start, ramdisk_end` 的标记，我们通过这个标记知道内存的位置。

之后，我们通过 `naive_load` 调用我们的 `loader`，来解析 `ramdisk.img` 中存储的 `ELF` 文件。获取其程序头，**从而知道程序的各部分**在文件中的 `offset`，从而把程序加载到内存的正确位置。
```c
void init_ramdisk()
{
    Log("ramdisk info: start = %p, end = %p, size = %d bytes", &ramdisk_start, &ramdisk_end,
        RAMDISK_SIZE);
}
```

完成 `load` 之后，我们实际就知道怎么执行了。我们也已经把 `PC` 跳到了用户程序的第一个指令处。即 `loader` 中的 ` return eh.e_entry;

之后，我们具体执行 `hello` 程序，如果打开 `etrace`，可以看到 `SYS_brk` 请求，说明 `newlib` 中的 `printf` 程序视图分配缓存区，但是触发了异常（因为此刻还没实现）

然后转而直接系统调用 `write` 到串口来打印。每一个字符触发了 `nanos-lite` 层的系统调用 `write`

然后我们调用 `io_write` 来写到串口设备（`AM` 层）

之后 `io_write` 进一步到 `NEMU` 中内存访问串口来实现设备读写到事先定义好的串口位置。
至此，**计算机**的各抽象层正在向我们徐徐展开...

### 必答题 - 仙剑奇侠传的运行
> [!Question] 仙剑奇侠传究竟如何运行
> 运行仙剑奇侠传时会播放启动动画, 动画里仙鹤在群山中飞过. 这一动画是通过 `navy-apps/apps/pal/repo/src/main.c` 中的 `PAL_SplashScreen()` 函数播放的. 阅读这一函数, 可以得知仙鹤的像素信息存放在数据文件 `mgo.mkf` 中. 请回答以下问题: 库函数, libos, Nanos-lite, AM, NEMU是如何相互协助, 来帮助仙剑奇侠传的代码从 `mgo.mkf` 文件中读出仙鹤的像素信息, 并且更新到屏幕上? 换一种PA的经典问法: 这个过程究竟经历了些什么? (Hint: 合理使用各种trace工具, 可以帮助你更容易地理解仙剑奇侠传的行为)


这里我们实现了渐变效果，是通过根据**时间推移**不断加深颜色（具体而言是扩大 `rgb` 值）来做的。

```c
if (dwTime < 15000)
{
   for (i = 0; i < 256; i++)
   {
      rgCurrentPalette[i].r = (BYTE)(palette[i].r * dwTime / 15000);
      rgCurrentPalette[i].g = (BYTE)(palette[i].g * dwTime / 15000);
      rgCurrentPalette[i].b = (BYTE)(palette[i].b * dwTime / 15000);
   }
}
```



- 读取绘图文件: 用 `MKFReadChunk` 实现
-  `<抽象层次>`

```c
INT
PAL_MKFReadChunk(
   LPBYTE          lpBuffer,
   UINT            uiBufferSize,
   UINT            uiChunkNum,
   FILE           *fp
);

PAL_MKFReadChunk <pal> 
-> open/lseek/write <libos>
-> do_syscall <nanos-lite>
-> memcpy <abstract-machine>
```


- 计时器
```c
SDL_GetTicks() <PAL>
-> NDL_GetTicks()	<PAL> // still nvay layer
-> gettimeofday		<libos>
-> do_syscall	<nanos-lite>
-> sys_read		<nanos-lite>
-> io_read		<abstract-machine>
-> read from memory...	<nemu>
```


- 画图
```c
Some Draw Function in PAL <PAL>
-> Finally call some NDL function to Draw <PAL>
-> syscall to draw / write <libos>
-> do_syscall	<nanos-lite>
-> sys_write	<nanos-lite>	// write to VGA
-> io_write		<abstract-machine>
-> nemu process write instructions: nemu will found the address is mapped into device: then it will write the data out to SCREEN to display <nemu>
```


- 处理按键事件
- 和画图同理。只是读取的是键盘。不再赘述。

仙剑奇侠传作为一款游戏，需要处理画面的输出、更新，用户输入（键盘）的读取，我们通过 `nanos-lite - abstract-machine - nemu` 的层次结构，各层抽象，完成了这个伟大而有趣的任务。

由于本笔记存放 obsidian 仓库异常，丢失了 Stage3 的部分笔记。有些遗憾，以后可能补上。

---


> This ends the PA3
> **夜を越える**





