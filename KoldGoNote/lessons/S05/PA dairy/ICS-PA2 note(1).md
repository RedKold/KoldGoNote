
- 在 `include/cpu/decode.h` 中定义了结构体 `Decode`，其包含解码信息的必要信息

```c
typedef struct Decode {
  vaddr_t pc;
  vaddr_t snpc; // static next pc
  vaddr_t dnpc; // dynamic next pc
  ISADecodeInfo isa;
  IFDEF(CONFIG_ITRACE, char logbuf[128]);
} Decode;

```


### 尝试 NEMU 的第一个程序
在参考讲义试图编译运行 `dummy` 程序，发现 `python: Permission denied`，报错
`
```
/bin/sh: 1: python: Permission denied
make[1]: *** [/home/kasumi/PAs/ics2024/abstract-machine/scripts/platform/nemu.mk:22: insert-arg] Error 127
test list [1 item(s)]: dummy
[         dummy] ***FAIL***j

```

由于本机没有 `/usr/bin/python`，将这个访问链接到 `python3`
```bash
sudo ln -s /usr/bin/python3 /usr/bin/python`
```

解决了该问题

- 编译运行 `dummy` 程序，在 `/ics2024/am-kernels/tests/cpu-tests` 下执行
```bash
make ARCH=$ISA-nemu ALL=dummy run
```

RTFSC ，读该目录下的 `Makefile`



运行 `dummy` 程序报错，是因为 `instruction at PC = 0x8000000 is not implemented`
- yzh 打出了大大的 see **risc-v Manual**
- 我用 `gdb` 的 `bt` 功能打印了整个调用栈
![image.png|700](https://kold.oss-cn-shanghai.aliyuncs.com/20250917233231.png)

 - `nemu` 运行程序会首先试图 `load_image` ,如果没有则加载默认的 `image`
	 - 这里通过阅读 Makefile，得知我相当于用 `dummy` 编译后的作为 `image`。而由于命令我没有实现，所以报错。
- ` //nemu/include/generated/antoconf.h #define CONFIG_MBASE 0x80000000`
这是 `risc-v` 的基础地址 `0x80000000`
```c
// src/memory/paddr.c

uint8_t* guest_to_host(paddr_t paddr) { return pmem + paddr - CONFIG_MBASE; }
paddr_t host_to_guest(uint8_t *haddr) { return haddr - pmem + CONFIG_MBASE; }
```

这两个函数
`guest_to_host` 负责将客户地址转化为 host 地址。



```

/home/kasumi/PAs/ics2024/am-kernels/tests/cpu-tests/build/dummy-riscv32-nemu.elf:     file format elf32-littleriscv


Disassembly of section .text:

80000000 <_start>:
80000000:	00000413          	li	s0,0
80000004:	00009117          	auipc	sp,0x9
80000008:	ffc10113          	addi	sp,sp,-4 # 80009000 <_end>
8000000c:	00c000ef          	jal	ra,80000018 <_trm_init>

80000010 <main>:
80000010:	00000513          	li	a0,0
80000014:	00008067          	ret

80000018 <_trm_init>:
80000018:	ff010113          	addi	sp,sp,-16
8000001c:	00000517          	auipc	a0,0x0
80000020:	01c50513          	addi	a0,a0,28 # 80000038 <_etext>
80000024:	00112623          	sw	ra,12(sp)
80000028:	fe9ff0ef          	jal	ra,80000010 <main>
8000002c:	00050513          	mv	a0,a0
80000030:	00100073          	ebreak
80000034:	0000006f          	j	80000034 <_trm_init+0x1c>
```
- 这是 `dummy` 程序的汇编结果。可以看到，第一条指令 `0x80000000` 存储的命令是 `00000413` 指令，我们没有实现，所以 `nemu` 报错并让我们对照手册取实现。
- 有趣的是，如果你去读 `nemu-log.txt`，你会发现和编译器给出的命令名不一样：
	- `0x80000000: 00 00 04 13 mv  s0, zero`
		- `zero` 寄存器是 RISC-V 体系一个特殊的寄存器，值永远是 `0`
	- `li` 实际是 load immediate 的意思
	- `mv` 也是一个别名和 `li` 一样，（伪指令），是 `assembler` 为了让代码更易读取的别名。
	- `mv rd rs` 会把 `rs` 的值复制到 `rd` 中
	- `mv s0 zero` 即把 `s0` 寄存器清零。
	- `mv rd, rs` = `addi rd, rs, 0` 

### 尝试理解如何解析程序
框架代码定义了如下程序，用来解析。

```c
//src/isa/riscv32/inst.c

static void decode_operand(Decode *s, int *rd, word_t *src1, word_t *src2, word_t *imm, int type) {
  uint32_t i = s->isa.inst;
  
  // we can do so, because risc-v arrange that, rs1 and rs2 is always in same bits
  
  int rs1 = BITS(i, 19, 15);
  int rs2 = BITS(i, 24, 20);
  *rd     = BITS(i, 11, 7);
  
  // some RISC-V format
  switch (type) {
    case TYPE_I: src1R();          immI(); break;
    case TYPE_U:                   immU(); break;
    case TYPE_S: src1R(); src2R(); immS(); break;
    case TYPE_N: break;
    default: panic("unsupported type = %d", type);
  }
}
```
- 这里的 `R`，代表寄存器 `register` 的意思
- `immI, immU, immS` 是分别定义的宏，作用是按照 `RISC-V` 不同的格式 `I, U, S` 读取立即数 ` imm `
- `rs1` `rs2` 在任何指令中，位置不变。（source register）
- 我们可用 `opcode` `func3` **来唯一确定一条指令**
	- **`0010011`** 表示这是一个 **“立即数-整数运算”** 指令大类（I-Type）。
	- **`0110011`** 表示这是一个 **“寄存器-寄存器运算”** 指令大类（R-Type）。
	- **`0000011`** 表示这是一个 **“加载”** 指令大类。
	- `0100011` 表示这是一个 “**存储**” (store) 指令大类
- 附录：
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250919142513.png)

```c
#define immI() do { *imm = SEXT(BITS(i, 31, 20), 12); } while(0)
// 看的更清楚一点
do { *imm = SEXT(BITS(i, 31, 20), 12); } while(0)
// SEXT是宏，作用是位数拓展。`BITS`是位提取。
``` 

- 一些细节，查阅手册，结合 AI，可以得到
- 例如，我们对于 `immI`, 手册规定立即数是一个有符号数，但在我们机器，需要正确拓展其位数。以让符号位（对于 `imm` 在原指令，是 `31-20` 的 12bit 数的首位表达符号位）正确表示。（符号位移到机器的首位）。符号拓展-这就是 `SEXT` 宏的作用。位截断则是 `BITS` 宏的作用。

### RISC-V 的汇编代码很多伪汇编代码 `li, ret`，如何快速知道它们的意思？
`li` 、`ret` 均不是 `riscv` 的标准指令。`li` 其实是 `addi`， `ret` 即 `return` 是 `jalr` (jump and link register)
- 要快速认识这一点，可以查看二进制指令标识，搜索其末尾的 `optcode` 标识来查找手册。

### 手册介绍
- 基本格式
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250919202705.png)

> [!Note] ***关于立即数位数的理解***
> - **你可能有疑问**，为什么 `imm[11:0], imm[31:12]` 这样乱七八糟的？这代表什么意思？
> - **首先** `index from 0`.  比如 `B-Type`，其没有规定最低位是多少，我们就**默认其是 `0`**
> 	- **这其实**保证了分支目标一定是对齐到 2 字节（半字）的地址。
> 	- 实际上 RV32I 是 4 字节对齐的。
> 	- **所以不需要编码在指令里**。（**这节省了一部分空间**!）
> - 然后，`RISC-V` 为了能统一字长，**做出了这个设计**。这样，`imm` 立即数通过存储在不同的段来达到更好的兼容性。
> - 所以，例如 `J-Type`，`31bit` 处存储的数值应该左移 `20` 位，再和 `30-21` 存储的 bit 左移 `1` 位.... 取或得到最终的结果
> Similarly, the only difference between the U and J formats is that the 20-bit immediate is shifted left by 12 bits to form U immediates and by 1 bit to form J immediates.



### 解决: 实现了 `addi` 但是执行错误（思考：执行指令失败程序都做了什么？）

这记录我解决问题的过程，同时也回答思考题
##### 为什么执行了未实现指令会出现上述报错信息 
> RTFSC, 理解执行未实现指令的时候, NEMU 具体会怎么做.
- 这是解码代码的节选
```c
// decode
INSTPAT_START();
// ...
INSTPAT("0000000 00001 00000 000 00000 11100 11", ebreak , N, NEMUTRAP(s->pc, R(10))); // R(10) is $a0
// note: N is not really a type. It means Not
INSTPAT("??????? ????? ????? ??? ????? ????? ??", inv    , N, INV(s->pc));
INSTPAT("??????? ????? ????? 000 ????? 00100 11", addi   , I, R(rd) = src1 + imm);
INSTPAT_END();
```
- 可以看到 `INSTPAT("??????? ????? ????? ??? ????? ????? ??", inv    , N, INV(s->pc));` 的 `pattern` 是一个全 `?` 的串。如果你了解展开宏 `INSTPAT` 的内容，你会认识到写在这一行匹配规则之后的 `addi` 以及其他所有匹配规则都不会生效。
- 这类似于 `switch` 语句的 `default`，请务必放在最后写，以保证指令生效。

- 我能快速定位问题，得益于 `gdb` 的帮助。**和对程序执行未实现指令时行为的进一步理解**。
用 `grep` 查找了打印错误信息的函数，并用 `gdb` 调试打断点。
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250919181058.png)

调用栈如图所示

- 可以看到，调用栈在 `decode_exec:71line` 之后转到了打印错误信息的 `invalid_inst`。而 `71line` 正是 `INSTPAT("??????? ????? ????? ??? ????? ????? ??", inv    , N, INV(s->pc));` 
- ` #define INV(thispc) invalid_inst(thispc)`
	- 而这个宏的内容，就是调用 `invalid_inst(thispc)`

### 更新 PC
```c
INSTPAT("??????? ????? ????? 000 ????? 11000 11", beq    , B, if(src1 == src2) s->dnpc = s->pc +imm);
```

- `s->dnpc` 是动态 `dynamic` 的意思，所以我们应显式维护的是这个 `pc`。对于顺序执行的程序，`dnpc` 和 `stpc` 完全一样。而如果有跳转，则需要 `dnpc` 指明。

```c
// nemu/src/cpu/cpu-exec.c
static void exec_once(Decode *s, vaddr_t pc) {
	s->pc = pc;
	s->snpc = pc;
	isa_exec_once(s);
	cpu.pc = s->dnpc;
	/* 
		some codes
	*/
}
```
## Different Type Optcode
### J
RISC-V 的 **J-Type** 指令用于**无条件跳转和链接**。这种指令的格式非常简单，它将一个 20 位的立即数编码在指令字中，以便实现长距离的跳转。

---

#### J-Type 指令格式

J-Type 指令只有一个，那就是 `jal`（Jump and Link）。它的 32 位编码格式如下：

| 31 位      | 30-21 位     | 20 位      | 19-12 位      | 11-7 位 | 6-0 位    |
| --------- | ----------- | --------- | ------------ | ------ | -------- |
| `imm[20]` | `imm[10:1]` | `imm[11]` | `imm[19:12]` | `rd`   | `opcode` |
- 立即数的存储需要我们重新组合。因为不同位藏在不同地方。这主要是为了 **兼容** ，兼容 RV32C 的 16 位压缩指令，从而简化指令解码的硬件逻辑。
- 例如 J-Type，我们需要做如下处理

```c
#define immJ() do { \
	*imm = SEXT( \
		(BITS(i,31,31) << 20) | /* imm[20] */ \
		(BITS(i,19,12) << 12) | /* imm[19:12] */ \
		(BITS(i,20,20) << 11) | /* imm[11] */ \
		(BITS(i,30,21)) << 1, /* imm[10:1] */ \
		21 \
	); Log("The value is %#x", *imm);\
} while(0)

```


### 思考：`riscv32` 是如何实现 32bit 大数字的读取的？

`addi, lui`

1. **加载大立即数**：由于 RISC-V 指令集的固定长度限制，无法直接加载 32 位立即数。通过 LUI 指令加载高 20 位，再结合其他指令（如 ADDI）加载低 12 位，可以构造完整的 32 位数值。
### 代码细节 
- 请特别注意大小比较和计算涉及 `溢出`、`无符号、有符号` 、等情况的行为
- `slt, sltu, mulh` 利益相关
- `lui` 的写法
```c
// wrong
INSTPAT("??????? ????? ????? ??? ????? 01101 11", lui    , U, R(rd) =(SEXT(imm << 12,20))); 
// right
INSTPAT("??????? ????? ????? ??? ????? 01101 11", lui    , U, R(rd) = imm); 
```

### 思考题-如何批处理测试？
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250924102643.png)

## Monitor Bugs
- 发现 `w <expr>` 添加监视点的时候，如果添加一个监视点，`diff` 会没有触发。有两个，就可以了。
	- `while (h->next)` 改为 `while(h)`



## 基础设施

- `iringbuf`
	- 这部分实现在 `nemu/src/utils/itrace.c`
- `mtrace`
	- 这部分偷懒了，函数仍然写在 `nemu/src/utils/itrace.c` 中
	- 在 `Kconfig` 中添加一个宏，在项目中名为 `CONFIG_MTRACE`
	- 放在 `paddr_read` 和 `paddr_write` 函数中调用打印方法。
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251018155808.png)
	- 大概的效果


### `ftrace`
追踪函数调用：这部分需要阅读 `elf` 文件。

该命令可以读 elf
```bash
riscv64-linux-gnu-readelf -a add-riscv32-nemu.elf
```

RTFM by `man 5 elf`

一个典型的 `elf` 文件组织
```
+-------------------+
| ELF Header        |  → 文件总体信息（魔数、架构、偏移量等）
+-------------------+
| Program Header(s) |  → 程序运行相关（加载段信息，给操作系统用）
+-------------------+
| Section(s)        |  → 代码段、数据段、符号表、字符串表等
+-------------------+
| Section Header(s) |  → 每个 Section 的描述信息（给链接器/调试器用）
+-------------------+

```

### 如何判别 `elf`
```
ELF header (Ehdr)
      The ELF header is described by the type Elf32_Ehdr or Elf64_Ehdr:

          #define EI_NIDENT 16

          typedef struct {
              unsigned char e_ident[EI_NIDENT];
              uint16_t      e_type;
              uint16_t      e_machine;
              uint32_t      e_version;
              ElfN_Addr     e_entry;
              ElfN_Off      e_phoff;
              ElfN_Off      e_shoff;
              uint32_t      e_flags;
              uint16_t      e_ehsize;
              uint16_t      e_phentsize;
              uint16_t      e_phnum;
              uint16_t      e_shentsize;
              uint16_t      e_shnum;
              uint16_t      e_shstrndx;
          } ElfN_Ehdr;

```

`elf` 文件提供了魔数，即一个数组 `e_ident`，它的前几个字节是固定的：

```
// elf.h
#define EI_MAG0         0       /* e_ident[] indexes */
#define EI_MAG1         1
#define EI_MAG2         2
#define EI_MAG3         3
#define ELFMAG0         0x7f    /* e_ident[EI_MAG0] */
#define ELFMAG1         'E'     /* e_ident[EI_MAG1] */
#define ELFMAG2         'L'     /* e_ident[EI_MAG2] */
#define ELFMAG3         'F'     /* e_ident[EI_MAG3] */
#define ELFMAG          "\177ELF"  /* 也就是 {0x7f, 'E', 'L', 'F'} */
#define SELFMAG         4          /* 魔数长度是4字节 */
```

故可以通过内存检测的办法，打开一个 `elf` 文件，检测其 `e_ident` 的内存是否等于 `ELFMAG`（在长度 `SELFMAG` 意义），具体来说可以用这个 C 语句

```c
if (memcmp(elf_header.e_ident, ELFMAG, SELFMAG) != 0) {
    fprintf(stderr, "Not an ELF file\n");
    exit(EXIT_FAILURE);
}
```


同时，`elf header` 还有一个定义了表头的大小的成员 `elf section header entry size, e_shentsize`。

```
e_shentsize
	This member holds a sections header's size in bytes.  A section header is one entry in the section header table; all entries are the same size.
```
### 如何找到符号表？
#### find the `section header table`

```
e_phoff
	 This member holds the program  header  table's  file  offset  in
	 bytes.   If  the  file  has no program header table, this member
	 holds zero.

e_shoff
	 This member holds the section  header  table's  file  offset  in
	 bytes.   If  the  file  has no section header table, this member
	 holds zero.
```

这是 `ELF Header` 的两个成员。分别定义了 `program header table's file offset` 和 `section header table's file offset`

我们暂时关心 `section header`，

通过 `fseek(fp, e_shoff, SEEK_SET` 我们可以把文件指针定位到 `section` 节。

#### find the 字符节
当我们已经到达 `Section Table`，接下来就是去找字符表部分或其他我们关心的部分

> _sh_offset_
              This member's value holds the byte offset from the
              beginning of the file to the first byte in the section.
              One section type, **SHT_NOBITS**, occupies no space in the
              file, and its _sh_offset_ member locates the conceptual
              placement in the file.


`Section Header` 有以下成员：

```c
typedef struct {
   uint32_t   sh_name;
   uint32_t   sh_type;
   uint32_t   sh_flags;
   Elf32_Addr sh_addr;
   Elf32_Off  sh_offset;
   uint32_t   sh_size;
   uint32_t   sh_link;
   uint32_t   sh_info;
   uint32_t   sh_addralign;
   uint32_t   sh_entsize;
} Elf32_Shdr;

```

其中
- `sh_type` 代表符号表类型-
	- `SHT_STRTAB`
	- `SHT_SYMTAB`
- 我们可以通过 `fread(str_table, sizeof(Elf32_Shdr), 1, fp)` 的方法，遍历 `section header table` 的所有表头，比对：`strtab_header.sh_type == SHT_STRTAB` ，然后找到 `string table` 或其他想要的表


实际上，`Section Header Table` 中的不同表单 `entry` 是叠放在一起的结构体，他们的大小都是 `sizeof(Elf32_Shdr)`

```
+---------------------------+
| ELF Header (Elf64_Ehdr)   |  ←  文件开头
+---------------------------+
| Program Header Table      |  （可选，用于可执行/共享库）
+---------------------------+
| Section 1 (.text)         |
+---------------------------+
| Section 2 (.data)         |
+---------------------------+
| Section 3 (.bss)          |
+---------------------------+
| Section 4 (.symtab)       |
+---------------------------+
| Section 5 (.strtab)       |
+---------------------------+
| ...                       |
+---------------------------+
| Section Header Table       | ← e_shoff 指向这里
| [0] Elf64_Shdr             |
| [1] Elf64_Shdr             |
| [2] Elf64_Shdr             |
| [3] Elf64_Shdr             |
| ... 共 e_shnum 个节头项
+---------------------------+
```


### 阅读符号表，对照字符表

我们实际需要找到这两个表。建议保存字符表为一个哈希表便于查找。

|对象|文件段|内容|被谁引用|
|---|---|---|---|
| `.shstrtab` |节名字符串表|各个 Section 的名字|Section Header (`sh_name`)|
| `.strtab` |符号名字符串表|各个符号的名字|符号表项 (`st_name`)|
| `.symtab` |符号表|符号的地址、类型、大小|程序、链接器、调试器|
| `.dynsym` |动态符号表|动态链接符号|动态链接器|
在 `符号表` 中的偏移量大小，就可以对应的在 `字符串表` **找到其表达的字符串名字**


> // PA 的要求
	现在我们就可以把一个给定的地址翻译成函数名了: 由于函数的范围是互不相交的, 我们可以逐项扫描符号表中 `Type` 属性为 `FUNC` 的每一个表项, 检查给出的地址是否落在区间 `[Value, Value + Size)` 内, 若是, 则根据表项中的 `Name` 属性在字符串表中找到相应的字符串, 作为函数名返回. 如果没有找到符合要求的符号表表项, 可以返回字符串"???", 不过这很可能是你的实现错误导致的, 你需要再次检查你的实现.


### 增加对 `elf文件的支持`

- 在 `parse_arg()` 中增加对 `elf` 文件解析的内容
- 在 `nemu.mk（abstract-machine/scripts/platform/nemu.mk）` 中增加参数 `-e` 的内容
```
NEMUFLAGS += -l $(shell dirname $(IMAGE).elf)/nemu-log.txt
NEMUFLAGS += -b
NEMUFLAGS += -e $(IMAGE).elf
```


从 `gdb` 可知道，实际是在运行 `$ISA-interpreter` 时候加上了这些参数

```
(gdb) run
Starting program: /home/kasumi/PAs/ics2024/nemu/build/riscv32-nemu-interpreter -l /home/kasumi/PAs/ics2024/am-kernels/tests/cpu-tests/build/nemu-log.txt -b -e /home/kasumi/PAs/ics2024/am-kernels/tests/cpu-tests/build/add-riscv32-nemu.elf /home/kasumi/PAs/ics2024/am-kernels/tests/cpu-tests/build/add-riscv32-nemu.bin
```


**需要特别注意的是**， `Unix` 解析命令行的方法
```c
#include <unistd.h>

int main(int argc, char *argv[]) {
    int opt;
    while ((opt = getopt(argc, argv, "bhl:d:p:e:")) != -1) {
        switch (opt) {
            case 'b': /* 处理 -b 选项 */ break;
            case 'h': /* 处理 -h 选项 */ break;
            case 'l': /* 处理 -l 参数 */ break;
            case 'd': /* 处理 -d 参数 */ break;
            case 'p': /* 处理 -p 参数 */ break;
            case 'e': /* 处理 -e 参数 */ break;
            default: /* 处理未知选项 */ break;
        }
    }
    return 0;
}

```

所以，我们需要在 `src/monitor/monitor.c` 中的 `parse_arg()` 函数中增加一个 `e:` 
`:` **表示这是个需要参数的选项**。

> Without it, you won't get the `o`. then your switch would not  get `e`.


```c
while ( (o = getopt_long(argc, argv, "-bhl:d:p:e:", table, NULL)) != -1) {
	switch (o) {
		case 'e': /* ... */
		/* 
			...
		*/
	}
}

```

函数追踪功能的开启，我们定义他为一个宏 `CONFIG_FTRACE`，同样在 `Kconfig` 中修改，并在 `make menuconfig` 中进行设置

### 初步测试：可以阅读符号表
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251023152352.png)


比对 `add` 函数的用 linux 工具阅读的作为对照：

```
 riscv64-linux-gnu-readelf -a add-riscv32-nemu.elf
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           RISC-V
  Version:                           0x1
  Entry point address:               0x80000000
  Start of program headers:          52 (bytes into file)
  Start of section headers:          5604 (bytes into file)
  Flags:                             0x0
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         3
  Size of section headers:           40 (bytes)
  Number of section headers:         9
  Section header string table index: 8

Section Headers:
  [Nr] Name              Type            Addr     Off    Size   ES Flg Lk Inf Al
  [ 0]                   NULL            00000000 000000 000000 00      0   0  0
  [ 1] .text             PROGBITS        80000000 001000 000128 00  AX  0   0  4
  [ 2] .rodata           PROGBITS        80000128 001128 000040 00   A  0   0  4
  [ 3] .data             PROGBITS        80000168 001168 000120 00  WA  0   0  4
  [ 4] .comment          PROGBITS        00000000 001288 00002b 01  MS  0   0  1
  [ 5] .riscv.attributes RISCV_ATTRIBUTE 00000000 0012b3 00001f 00      0   0  1
  [ 6] .symtab           SYMTAB          00000000 0012d4 000220 10      7  15  4
  [ 7] .strtab           STRTAB          00000000 0014f4 0000a6 00      0   0  1
  [ 8] .shstrtab         STRTAB          00000000 00159a 00004a 00      0   0  1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  D (mbind), p (processor specific)

There are no section groups in this file.

Program Headers:
  Type           Offset   VirtAddr   PhysAddr   FileSiz MemSiz  Flg Align
  RISCV_ATTRIBUT 0x0012b3 0x00000000 0x00000000 0x0001f 0x00000 R   0x1
  LOAD           0x001000 0x80000000 0x80000000 0x00168 0x00168 R E 0x1000
  LOAD           0x001168 0x80000168 0x80000168 0x00120 0x00120 RW  0x1000

 Section to Segment mapping:
  Segment Sections...
   00     .riscv.attributes
   01     .text .rodata
   02     .data

There is no dynamic section in this file.

There are no relocations in this file.

The decoding of unwind sections for machine type RISC-V is not currently supported.

Symbol table '.symtab' contains 34 entries:
   Num:    Value  Size Type    Bind   Vis      Ndx Name
     0: 00000000     0 NOTYPE  LOCAL  DEFAULT  UND
     1: 80000000     0 SECTION LOCAL  DEFAULT    1 .text
     2: 80000128     0 SECTION LOCAL  DEFAULT    2 .rodata
     3: 80000168     0 SECTION LOCAL  DEFAULT    3 .data
     4: 00000000     0 SECTION LOCAL  DEFAULT    4 .comment
     5: 00000000     0 SECTION LOCAL  DEFAULT    5 .riscv.attributes
     6: 00000000     0 FILE    LOCAL  DEFAULT  ABS start.o
     7: 80000000     0 NOTYPE  LOCAL  DEFAULT    1 $x
     8: 00000000     0 FILE    LOCAL  DEFAULT  ABS add.c
     9: 80000010     0 NOTYPE  LOCAL  DEFAULT    1 $x
    10: 80000028     0 NOTYPE  LOCAL  DEFAULT    1 $x
    11: 00000000     0 FILE    LOCAL  DEFAULT  ABS trm.c
    12: 800000fc     0 NOTYPE  LOCAL  DEFAULT    1 $x
    13: 80000108     0 NOTYPE  LOCAL  DEFAULT    1 $x
    14: 80000128    64 OBJECT  LOCAL  DEFAULT    2 mainargs
    15: 80000108    32 FUNC    GLOBAL HIDDEN     1 _trm_init
    16: 80009000     0 NOTYPE  GLOBAL DEFAULT    3 _stack_pointer
    17: 80000128     0 NOTYPE  GLOBAL DEFAULT    1 _etext
    18: 80000000     0 NOTYPE  GLOBAL DEFAULT  ABS _pmem_start
    19: 80000288     0 NOTYPE  GLOBAL DEFAULT    3 _bss_start
    20: 80000288     0 NOTYPE  GLOBAL DEFAULT    3 edata
    21: 80009000     0 NOTYPE  GLOBAL DEFAULT    3 _heap_start
    22: 80001000     0 NOTYPE  GLOBAL DEFAULT    3 _stack_top
    23: 80009000     0 NOTYPE  GLOBAL DEFAULT    3 end
    24: 80000010    24 FUNC    GLOBAL HIDDEN     1 check
    25: 80000128     0 NOTYPE  GLOBAL DEFAULT    1 etext
    26: 80000000    16 FUNC    GLOBAL DEFAULT    1 _start
    27: 00000000     0 NOTYPE  GLOBAL DEFAULT  ABS _entry_offset
    28: 80000028   212 FUNC    GLOBAL HIDDEN     1 main
    29: 80000288     0 NOTYPE  GLOBAL DEFAULT    3 _data
    30: 80000168   256 OBJECT  GLOBAL HIDDEN     3 ans
    31: 80009000     0 NOTYPE  GLOBAL DEFAULT    3 _end
    32: 800000fc    12 FUNC    GLOBAL HIDDEN     1 halt
    33: 80000268    32 OBJECT  GLOBAL HIDDEN     3 test_data

No version information found in this file.
Attribute Section: riscv
File Attributes
  Tag_RISCV_arch: "rv32i2p0_m2p0"

```

比对，可见符号表是正常解析的。**接下来就需要读取函数了**。


### `riscv` 如何判别 `call` 和 `ret`？

首先明确一下寄存器结构：
RISC-V 架构提供了 32 个通用寄存器，编号为 x0 到 x31。每个寄存器都有特定的用途：
- **x0 (zero)**: 硬编码为 0，读出总是 0，写入无效。
- **x1 (ra)**: 返回地址寄存器，用于保存函数返回地址。
- **x2 (sp)**: 栈指针寄存器，指向栈的地址。
- **x3 (gp)**: 全局指针寄存器，用于链接器松弛优化。
- **x4 (tp)**: 线程指针寄存器，常用于保存指向进程控制块的指针。
- **x5-x7, x28-x31 (t0-t6)**: 临时寄存器，用于存储临时数据。
- **x8 (s0/fp)**: 帧指针寄存器，用于函数调用时保存数据。
- **x9 (s1)**: 保存寄存器，用于函数调用时保存数据。
- **x10-x17 (a0-a7)**: 用于函数调用，传递参数和返回值。
- **x18-x27 (s2-s11)**: 保存寄存器，用于函数调用时保存数据。

From the view of **code**:
```c
const char* regs[] = 
{
	"$0", "ra", "sp", "gp", "tp",  "t0",  "t1", "t2", "s0","s1", "a0",
	"a1", "a2", "a3", "a4", "a5",  "a6",  "a7", "s2", "s3", "s4", "s5",
	"s6", "s7", "s8", "s9", "s10", "s11", "t3", "t4", "t5", "t6"
};
```

**对于 `CALL`**
- `jal` 命令返回的地址应该存入 `x1(ra)` 中
```c
	// in jal, jalr
	// 返回地址存入`ra`
	if( rd == 1){ // `1` is actually a index of `ra`
		ftrace_call(s->pc, s->dnpc);
	}
```

**对于 `RET`**
- 根据汇编器的记号，也可以推断出是 `jalr x0, 0(ra)` 的别名
- `rd=x0` - 不保存返回地址
- `rs1 = ra(x1)` - 从返回地址寄存器读取目标地址
- `imm = 0 ` 偏移量为 `0`

```c
// in jalr

```



### 维护调用栈：`call` 进 `ret` 出。

一个细节：
`call` 会增加递归深度，`ret` 会减少。但是 `call` 是增加了再 `call`，`ret` 是 `ret` 之后才减少。

所以，打印信息的顺序不一样。
`call` 应该先更新深度在打印，而 `ret` 相反
- 否则你的打印可能会很丑，**不对齐**。

```c
if (type == TYPE_CALL)
{
	call_depth++;
	ftrace_write(type, call_depth, i < sym_count ? name : "???", pc, target);
}

else if (type == TYPE_RET)
{
	ftrace_write(type, call_depth, i < sym_count ? name : "???", pc, target);
	call_depth--;
}

```

- `cputests/recurion.c`
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251024001741.png)


### Difftest
这部分的实现：
```c
bool isa_difftest_checkregs(CPU_state *ref_r, vaddr_t pc)
{ 
 	int reg_num = ARRLEN(cpu.gpr); 
	for (int i = 0; i < reg_num; i++) 
 	{ 
 		if (ref_r->gpr[i] != cpu.gpr[i]) {
 			return false; 
		} 
	} 
 	if (ref_r->pc != cpu.pc) {
  		return false;
  	} 
  	return true;
}
```



> [!Note] **感激 Difftest 的理由**
> 在跑 `benchmark` 测试的时候，新实现了 `ori` 指令，这是一个 `I-type` 的但我手滑输入成了 `R` 导致 `imm` 解析不正确，`coremark` 测试集 `hit bad trap` 而难以追踪。最后，使用 `Difftest`，我迅速定位到了导致程序这个状态机（主要是寄存器存储值）不一样的指令 `ori`，从而迅速解决了问题。（困扰了两个小时，再想到 `Difftest` 之后迎刃而解）



## IO

进行 `hello` 测试前，不要忘记在 `menuconfig` 中勾选 `Device`，否则 `paddr.c` 模块的 `paddr_read()` 和 `paddr_write()` 不能正确处理 `addr` 是物理内存还是设备空间。**从而报错。**

---

### 实现 IOE
```c
bool ioe_init();
void ioe_read(int reg, void *buf);
void ioe_write(int reg, void *buf);
```

## 串口
### 时钟
8253计时器初始化时会分别注册 `0x48` 处长度为8个字节的端口, 以及 `0xa0000048` 处长度为8字节的MMIO空间, 它们都会映射到两个32位的RTC寄存器. CPU可以访问这两个寄存器来获得用64位表示的当前时间.
```
#define RTC_ADDR        (DEVICE_BASE + 0x0000048)
```

在 `amstract-machine/am/include/amdev.h` 为时钟定义了两个抽象寄存器


```c
#define AM_DEVREG(id, reg, perm, ...) \
  enum { AM_##reg = (id) }; \
  typedef struct { __VA_ARGS__; } AM_##reg##_T;

AM_DEVREG( 1, UART_CONFIG,  RD, bool present);
AM_DEVREG( 2, UART_TX,      WR, char data);
AM_DEVREG( 3, UART_RX,      RD, char data);
AM_DEVREG( 4, TIMER_CONFIG, RD, bool present, has_rtc);
AM_DEVREG( 5, TIMER_RTC,    RD, int year, month, day, hour, minute, second);
AM_DEVREG( 6, TIMER_UPTIME, RD, uint64_t us);

```
这是一个学习宏的好机会。
`AM_##reg` 的 `##` 起到一个拼接的功能，会将字符串 `"AM_"` 和 `reg` 这个宏给定的参数连成一个新的标识符。

`__VA_ARGS__;` 则是所有可变宏参数的集合

```c
AM_DEVREG(1, UART_CONFIG, RD, bool present);
// 展开为：
enum { AM_UART_CONFIG = 1 };
typedef struct { bool present; } AM_UART_CONFIG_T;

AM_DEVREG(4, TIMER_CONFIG, RD, bool present, has_rtc);
// 展开为：
enum { AM_TIMER_CONFIG = 4 };
typedef struct { bool present, has_rtc; } AM_TIMER_CONFIG_T;

AM_DEVREG(5, TIMER_RTC, RD, int year, month, day, hour, minute, second);
// 展开为：
enum { AM_TIMER_RTC = 5 };
typedef struct { int year, month, day, hour, minute, second; } AM_TIMER_RTC_T;
```


---

` am/src/platform/nemu/include/nemu.h` 中有一些相关的输入输出函数可以使用

```c
// abstract-machine/am/src/$ISA/$ISA.h

static inline uint8_t  inb(uintptr_t addr) { return *(volatile uint8_t  *)addr; } 
static inline uint16_t inw(uintptr_t addr) { return *(volatile uint16_t *)addr; } 
static inline uint32_t inl(uintptr_t addr) { return *(volatile uint32_t *)addr; }
```
我们是 `32` 机，想获得 `32` 的地址，可以用 `inl` 这个函数。


> [!Note] 如何进行 `real-time clock test` 测试？
> 这需要我们阅读源代码。
> 在 `/am-kernel/tests/am-tests/src/main.c` 中定义了测试的入口。
> 
> ```c
> #include <amtest.h>
> 
> void (*entry)() = NULL; // mp entry
> 
> static const char *tests[256] = {
>   ['h'] = "hello",
>   ['H'] = "display this help message",
>   ['i'] = "interrupt/yield test",
>   ['d'] = "scan devices",
>   ['m'] = "multiprocessor test",
>   ['t'] = "real-time clock test",
>   ['k'] = "readkey test",
>   ['v'] = "display test",
>   ['a'] = "audio test",
>   ['p'] = "x86 virtual memory test",
> };
> 
> int main(const char *args) {
>   switch (args[0]) {
>     CASE('h', hello);
>     CASE('i', hello_intr, IOE, CTE(simple_trap));
>     CASE('d', devscan, IOE);
>     CASE('m', mp_print, MPE);
>     CASE('t', rtc_test, IOE);
>     CASE('k', keyboard_test, IOE);
>     CASE('v', video_test, IOE);
>     CASE('a', audio_test, IOE);
>     CASE('p', vm_test, CTE(vm_handler), VME(simple_pgalloc, simple_pgfree));
>     case 'H':
>     default:
>       printf("Usage: make run mainargs=*\n");
>       for (int ch = 0; ch < 256; ch++) {
>         if (tests[ch]) {
>           printf("  %c: %s\n", ch, tests[ch]);
>         }
>       }
>   }
>   return 0;
> }
> ```
> 可见我们需要这样一个参数输入给 main 函数, 如果要执行 `real-time clock test`。
> 由此，我们再阅读 `am-kernel/tests/am-tests` 中的 `Makefile`
> 
> 综上所述，你可以用类似这样的命令运行不同的 `am_test`：
> ```bash
> make ARCH=$ISA-nemu run mainargs=h
> ```
> - 该命令会运行 `hello` test

运行 `rtc_test` 之后，我们来用 `ftrace` 看看调用栈。
```
0x80001044  call [_trm_init]@0x80000e4c 
0x80000ff4   call [main]@0x800010e8 
0x80001128     call [ioe_init]@0x800011c4   
0x800011c4     ret [$x]@0x8000112c  
0x8000112c     call [ioe_init]@0x8000117c   
0x8000117c     ret [$x]@0x80001130  
0x80001130     call [ioe_init]@0x80001204   
0x80001204     ret [$x]@0x80001134  
0x80001140   ret [ioe_init]@0x80000ff8  
0x80000ff8   call [main]@0x80000aec 
0x80000b4c     call [rtc_test]@0x80001144   
0x80001194     ret [__am_timer_uptime]@0x80000b50   
0x80000b60     call [rtc_test]@0x80001858   
0x800015ac     ret [__udivmoddi4]@0x80000b64    
0x80000b4c     call [rtc_test]@0x80001144   
/.../
```

该程序就是不断的调用 `rtc_test`，最终调用到 `__am_timer_uptime` 来更新信息的。
> [!Tip] **yzh 先生埋得雷**
> 根据 pa manuel 和群友讨论，先更新低位或者高位似乎会影响跑分结果，这是 yzh 老师对大家的考验。



> [!Note] `__am_timer_uptime` 是怎么调用 `nemu` 这个系统来获得时间的？
> 我们在 `am` 层的访存最后也会翻译成机器指令交给 `nemu` 执行，
> 可以看一下 `__am_timer_uptime` ，其访问时钟本质也是访存，用的是 `inl`
> ```c
> // /abstract-machine/am/src/riscv/riscv.h
> static inline uint32_t inl(uintptr_t addr) 
> { return *(volatile uint32_t *)addr; }
> ```
> 这里我们使用 `volatile` 关键字避免 **编辑器优化**，其实也是保证 `nemu` 能够执行我们想要的访存命令。
> 
> 所以 `lw,lb,lh` 等访存指令中有相关的信息。继续阅读 `nemu` 代码
> 
>  	
> 
> 上述翻译成机器代码，对时钟的访问则被 `lw` 出发 `paddr_read`，同时检测到是一个 `DEVICE` 就去 `DEVICE` 的内存空间 (`mmio_read`)访问相应的设备（通过 `map_read`）
> **那么**，我们访问到时钟，就会触发回调函数。对于**时钟**来说就是下面的 `rtc_io_handler`
> ```c
> // 在nemu中，主函数会执行`init_device`初始化所有设备，会分配空间映射，绑定回调函数
> // nemu/src/device/timer.c 
> static void rtc_io_handler(uint32_t offset, int len, bool is_write) {
>    assert(offset == 0 || offset == 4);
>    if (!is_write && offset == 4) {
>      uint64_t us = get_time();
>      rtc_port_base[0] = (uint32_t)us;
>      rtc_port_base[1] = us >> 32;
>    }
>  }
> ```
> 所以，如果先**更新低位**，则 `offset==0`，则 `get_time` 不会被执行，从而导致跑分异常（你的低位时间获取的是上一次的低位，**而不是系统时间**）。
> 解决方案可以是先更新高位，或者把 `offset == 4 ` 改为 `offset == 0` 并保留原来的先更新低位。


> [!Summary] **总结一下**
> 现在 `nemu` 作为我们的系统，其会运行我们写的 `am` 层中的程序，阅读代码理解两方面的行为，合理使用 `ftrace` **和汇编代码等工具是很有必要的**。










![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251101140512.png)
- 正常运行了。
> 如果你的实现正确, 你将会看到程序每隔1秒往终端输出一行信息. 由于我们没有实现 `AM_TIMER_RTC`, 测试总是输出1900年0月0日0时0分0秒, 这属于正常行为, 可以忽略.





#### 跑分测试

- 在 `Aliyun - Ubuntu 22.04, 2g 2核` 上跑分结果
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251101150454.png)
	- 阿里云学生服务器的配置比较拉胯，跑分 97 应当是正确的结果。
	- 克隆项目到 `Ubuntu` 虚拟机测试

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251101205246.png)
	- `microbench`	


**现在我们把 `am-kernels/kernels/demo/include/io.h`**中的 `HAS_GUI` 注释掉了，来看字符。