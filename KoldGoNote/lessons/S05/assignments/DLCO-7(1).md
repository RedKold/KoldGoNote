---
本科课程: DLCO
author: 231275036-朱晗
date: 2025-11-12
title: DLCO-assignment-7
completed: "true"
---
习题 2 (4)、2 (9)、3、6、7、8、10、11、13、15、17。


## 2 (4) 寻址方式理解
> 哪些寻址方式下的操作数在寄存器中？哪些寻址方式下的操作数在存储器中？

**在寄存器中**
- 寄存器直接寻址

**在存储器中**
- 寄存器间接寻址（访问寄存器存储的地址的内存）
- 直接寻址
- 间接寻址
- 变址寻址

此外，**立即寻址**的操作数直接在指令内部。
## 2 (9) `call` and `jump`
**转移跳转**和**调用指令**的区别是什么？返回指令是否需要有地址码字段？
	转移跳转是直接改变 `pc` 并不维护给上层的**返回地址**和参数的 **入口地址**，但是 **调用指令**则需要维护二者。也就是 `call` 指令执行后会把其下一条指令的地址作为返回地址，**压入栈**后其他寄存器管理。并在调用结束后用 `ret` 指令置 `pc` 为返回地址
	**返回指令**不需要有地址码字段（通常），因为 `caller` 会把 `ret address` 压入栈帧管理或者放入某特定寄存器。
	如果 ISA 设计放入某个通用寄存器，**则需要一个地址码**。

## 3 寻址练习
某转移指令，共占两个字节。`操作码` + `相对位移量(补码)`，CPU 每次从内存取 `1` 个字节。假设执行到某转移指令时 `PC` 的内容为 `258`，执行该转移指令要求转移到 `220` 开始的一段程序执行，则该转移指令第二字节的内容应该是多少？
执行后，`pc` 已经更新为了 `260`
`260 + x = 220`
`a= -40`, `40_DEC = 0x28_HEX `, 则 `-40_DEC =0xff -0x28+1=0xd8`

第二字节内容为 `0xd8`
## 6  分配不同地址指令
> 定长指令字格式，指令字长为 `16bit`，每个操作数的地址码长 `6` 位。指令分二地址、单地址和零地址三类。若二地址指令有 `k2` 条，零地址指令有 `k0` 条，则单地址指令最多可以有多少条？

二地址有 `16-6*2=4` 位置分配操作码
- 共 `2^4=16`
零地址有 `16` 位置分配操作码
单地址有 `10` 位置分配**操作码**
我们可以从二地址的未编码中取一些给 `k1` 用。在 `k1` 剩余的 `10-4=6` 的编码位获得 `k1` 的所有编码。同时，我们应该保证零地址能正常读取。所以还需要**一位**标记这个是不是 `0` 地址指令。

**总体来说**：
- **先分配二地址**。前 `4` bit 剩下 `2^4-k2` 可以给一地址和零地址用。
- **分配一地址**：
假设该 ISA **没有编码浪费**。则 `k2` `k1` 分配剩下位置都可以给 `k0`
$$
((2^{4}-k_{2})\times 2^{6}-k_{1})\times 2^{(16-4-6)}=k_{0}
$$
**总结一下**：
$$
k_{1}=(16-k_{2})\times {2}^{6}-\frac{k_{0}}{2^{6}}
$$
这里注意 $2^{x}-k_{i}$ **实际就是某一段编码位置**，**去掉已经编码给其他操作的**的码位之后的意思。**所以才能保证大家的前缀不冲突**。
## 7 ISA design 
某计算机字长为 `16` 位，每次存储器访问宽度为 `16` 位，CPU 中有 `8` 个 `16` 位通用寄存器。
设计 ISA:
- 指令长度为字长的整数倍
- 至多支持 64 种不同操作
- 每个操作数支持 4 种寻址方式
	- imm  **I**
	- register direct addressing **R**
	- register indirect addressing **S**
	- shift-offset addressing **X** 
存储器地址位数和立即数均为 `16` 位
任何一个通用寄存器可作为变址寄存器。支持以下 7 种二地址指令格式。
`RR RI RS RX XI SI SS` 
**设计**
- 编码通用寄存器，需要至少 `3` 比特。
- 编码不同指令格式：需要 `3` 比特
- 支持 `imm` 读取，需要 `16` 比特
- 支持不同的操作码，需要 `log(64)=6` 比特
该指令集可以用 `3+3+16+6 = 28 -字长整数倍-> 32 位` 的指令
RR:
**定长**指令设计：
`RR`
```
// RR type
|	zero	|	src1	|	src2	|	opcode	|	typecode	|
|	16		|	3		|	3		|	7		|	3			|	
// zero means 16 bit zero series. because imm is not used.

// RI type
|	imm		|	src1	|	000		|	opcode	|	typecode	|
|	16		|	3		|	3		|	7		|	3			|

// RS type
|	imm		|	src1	|	src2	|	opcode	|	typecode	|
|	16		|	3		|	3		|	7		|	3			|

// RX type
|	imm		|	src1	|	src2	|	opcode	|	typecode	|
|	16		|	3		|	3		|	7		|	3			|
// here we can use src1 at first register R, and use src2 to form as shift, imm is offset

// XI type
|	imm2	|	imm		|	src1	|	src2	|	opcode	|	typecode	|
|	16		|	16		|	3		|	3		|	7		|	3			|
// we need another 16bit to store new imm to act as  offset

// SI type
|	imm		|	src1	|	src2	|	opcode	|	typecode	|
|	16		|	3		|	3		|	7		|	3			|
// similar as RI type. but get R we use M[R(src1)]

// SS type
|	zero	|	src1	|	src2	|	opcode	|	typecode	|
|	16		|	3		|	3		|	7		|	3			|
// similar as RR type;
```
## 8  寻址方式
计算机字长为 `16` 位，主存地址空间大小为 `128KB`, 按字编址。**采用单字长定长**指令格式，指令各字段定义如下：

转移指令相对寻址方式。offset 补码。寻址方式定义如图表所示。
![IMG_9573(20251111-195906).JPG|600](https://kold.oss-cn-shanghai.aliyuncs.com/IMG_9573(20251111-195906).JPG)

### 该指令系统最多可有多少条指令？最多可有多少个通用寄存器？存储器地址寄存器（MAR）和存储器数据寄存器（MDR）至少各需要多少位？

The length of `op` is `4` , **so at most `2^4=16` instructions;**
`gpr` index is `3` bit, so at most `8` gprs.
The storage size is `128KB` `word length = 16bit =2B`. so there is `128KB/2B = 64K` storage units.
The `MAR` must mapping to these units. `64K=2^6*1024=2^16`, **so `MAR` at least need `16` bit**

For the word length is `16bit`, **so `MDR` need `16bit`**

### 转移指令的目标地址范围是多少？
PC and gpr are all `16bit`. a `16` bit `signed` number is `[-32768, 32767]`
So target address is ranging from `-32768` (backward) to `32767` (forward)
**Note**: the target address can't out of range of `PC`. (which is **also** a `16` bit number)

### 若操作码 `0010B` 表示加法操作（助记符 `add`），寄存器 R4 和 R5 的编号分别为 `100B`, 和 `101B`，R4 的内容为 `1234H`, R5 `5678H`，地址 `5678H` 存储 `1234H`，则汇编语句 `add (R4), (R5)+` 对应的机器码是什么？（HEX）该指令执行后，哪些寄存器和存储单元的内容会改变？改变后的内容是什么？

**机器码**：
注意字长为 16 位，小端
```asm
0010	001	100	010	101	
0010	0011	0001	0101
HEX:
0x2315
```
机器码为 `0x2315`
该指令执行后。
`1234H+5678H = 68acH`，则 `R5` 对应内存单元 `M(R(5))` 即地址 ` 5678H ` 的内容变为 ` 68acH `
同时 R5 自增。故 `R5` 内容改变为 `5679H`
同时 `PC` 需要改变。
## 10 `call offset`
对于远距离的过程调用，使用伪指令 `call offset` 作为调用指令，对应以下两条真实指令：
```
auipc 	x1,		offset[31:12]+offset[11] 
jalr	x1,	x1,	offset[11:0]
```
请说明为什么 `auipc` 指令中高 `20` bit 的位移量计算时 `offset[31:12]` 需要加上 `offset[11]`?

我们相当于用 `auipc` 和 `jalr` 拼出来了一个 `32` bit 的大 `OFFSET` **作为偏移量**。
但是 `auipc` 需要移位，所以我们需要做 **符号拓展**。
该符号拓展需要我们加上 `offset[11]` 一起做移位。
- why？
	- 如果 `offset[11]=0`，则 `offset[11:0]` 拓展到 `32bit` 高位填 `0`，无影响。
	- 如果 `offset[11]=1`，则 `offset[11:0]` 拓展到 `32bit` 时候高位会产生一个进位 `1`。我们为了维护这个被符号位拓展挤掉的 `1`，就在 `offset[31:12]` 后加一个 `1`
总体来看，就是 `auipc 	x1,		offset[31:12]+offset[11] `

> [!Note] 再说一点
> **换句话说**，`offset[11:0]` 做符号拓展在其高位 `[31:11]` 填的都是 `1`，拿移位后的 `offset[31:12]` 来看就是一个 `-1`, 但这不是我们想要还原的大 `offset` 的真实值。**为了修整这个不应该**的符号拓展导致差的 `-1`，我们在移位之前补一个 `1`。
## 11 更好的乘法汇编
指令条数最少得不含乘法指令的 rv32I 代码
```asm
sll		t1,		t0,		3		# t1 = t0<<3
sub		t1,		t1,		t0		# t1 = t1-t0
```


## 13 程序段理解
入口参数 `int a`, `int b` 分别置于 `a0` 和 `a1` 中。返回参数是该过程的结果，置于 `a0` 中。要求为以下 `RV32I` 指令序列加注释，并简单说明该过程的功能。
```asm
			add		t0,	zero,	zero		// t0 = 0
loop:		beq		a1,	zero,	finish		// if(a1==0) jump to finish	
			add		t0,	t0,		a0			// t0 = t0 + a0;
			addi	a1,	a1,		-1			// a1 = a1 -1; (a1--)
			j		loop					// jump to start of loop
			
finish:		addi	t0,	t0,		100			// t0 = t0 + 100
			add		a0,	t0,		zero		// a0 = t0 + 0
```
该函数的功能是：

```c
func:
	int x=0; 	// t0
	while(b!=0){
		x+=a;	
		b--;		// addi a1, a1, -1
	}
	
	// finish:
	x+=100;	
	return x;
```

其计算并返回了 `a*b + 100`

## 15 设计按位与指令
假定编译器将 `a` 和 `b` 分别分配到 `t0` 和 `t1` 中，用一条 RV32I 指令或最短的 RV32I 指令序列以实现以下 C 语言语句：`b=31&a`. 如果把 `31` 换成 `65535`，即 `b=65535 & a`，则用 RV32I 指令或指令序列如何实现？
关于如何处理长立即数可以阅读 [[ICS-PA2 note#U|PA2-Utype]]
具体来说是 `lui` 和 `addi` 组合使用。

**这里**注意一下 [[#10 `call offset`]] 中的问题：分别存储，由于 `addi` 会对 `imm` 做符号拓展，`0xFFF` 被解释为 `-1` ，符号拓展后相当于差了一个 `-1`，我们在高位 `+1` 以弥补这个差。即 `lui t1 0xF+1`, 即 `lui t1 16`

```asm
// b=31&a, 31=0x1F
andi	t1	t0	0x1F

// b=65535&a, 65535=0xFFFF, 16bit, cannot store in 12bit imm
// we store the over part of 12bit, that is the high 4bit number of 0xFFFF (0xF=15), into the t1.
// lui(load upper immediate) 
lui		t1		16	
addi	t1		0xFFF
and		t1	t1	t0
```



## 17 RV32I 指令含义解释 `beq`

请说明 RV32I 中 `beq` 指令的含义，并解释为什么汇编程序在对下列汇编语言源程序中的 `beq` 指令进行汇编时会遇到问题，应该如何修改该程序段？
```
here:	beq 	t0,	t2,	there
	......
there:	addi	t1,	a0,	4
```

`beq`: branch - equal, 即如果两个操作数 `src1 == src2` ,并令 `pc = pc + imm` （这里是我们想要的 there）
进行汇编会发生问题：
- `beq` 是一个 `B` type 的指令，其立即数有 `7 + 4 + 1 =12` 位, 再加上默认的最低位 `0`（没有在指令中给出，这是因为跳转指令 `imm` 一定是 `2` 的倍数。(RV32G: 4. RV32C: 2)）(相当于 `imm*2 (imm<<1)`)
跳转指令范围：
首先表达 `imm` 的范围为：`imm[12,1]` 补码表达最大数为 `2047`, 加上最低位最大表达 `4094` ，而每个指令是 `4` byte，**所以最多向后**跳 `4094/4=1023` 个指令
同理，向前最多跳 `1024` 个指令。

那么，如果 `there` 和 `here` 的距离过远，**就无法跳转**。

而 `J-type` 能实现更远的无符号跳转(20bit imm)。可以利用这个特性
我们用一个 `break` 代码块，其离 `here` 很近, 保证 `bne` 可以跳转到。

```
here: 	bne 	t0,	t2,	break
		jal		x0, there
break:	
		...
there:	addi	t1,	a0,	4
```


