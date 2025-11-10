---
completed: "true"
course: 计算机系统
tags:
  - assignment
author: 231275036-朱晗
---
作业清单：
- 3，4，8，10，11，12

## 3 `func` 理解

### （1）
IA-32 系统。因为使用的是 `ebp`, `eax` 等寄存器，是 `32bit` 的。

### （2）
依次为 `%eax, %ebx, %ecx`

### (3)
```c
void func(int *xptr, int *yptr, int *zptr)
{
	// *zptr: %ecx
	// *yptr: %ebx
	// *xptr: %eax*
	
	// 保存临时值
	int a = *yptr;	// a: %edx
	int b = *zptr;	// b: %esi
	int c = *xptr;	// c: %edi
	
	// 修改值
	*yptr = c;
	*zptr = a;
	*xptr = b;
		
}
```

## 4 阅读 `operate()` 代码完成问题

### （1）写出汇编指令的注释，填写 `operate()` 函数中缺失的部分。

```
1	movl	12(%ebp),	%ecx 	# load y into %ecx
2	sall	$8,			%ecx	# y = y << 8
3 	movl	8(%ebp),	%eax	# load x into %eax
4	movl	20(%ebp),	%edx	# load k into %edx
5	imull	%edx,		%eax	# x = x * k;
6 	movl	16(%ebp),	%edx	# load z into %edx
7	andl	$65520,		%edx	# z = z & 0xFFF0
8 	addl 	%ecx,		%edx	# z = z + y
9	subl	%edx,		%eax	# return ans=x-z;	
```

将汇编代码总结起来，C 语言代码为

```c
int operate(int x,	int y,	int z,	int k)
{
	int v = x*k - (z & 0xFFF0 + y<<8)
	return v;
}
```


### (2) 给出 `x86-64` 格式的汇编代码，比较 `IA-32` 的性能；

| **IA-32 代码 (原始)**                          | **x86-64                                     | **解释 (x86-64)**                    |
| ------------------------------------------ | -------------------------------------------- | ---------------------------------- |
| **预处理:** (将传入参数复制到通用寄存器，模拟IA-32代码的内存加载)    |                                              |                                    |
| (x = %edi)                                 | `movl %edi, %r10d # r10d = x (临时)`           | 复制 x                               |
| (y = %esi)                                 | `movl %esi, %r11d # r11d = y (临时)`           | 复制 y                               |
| (z = %edx)                                 | `movl %edx, %r9d # r9d = z (临时)`             | 复制 z                               |
| (k = %ecx)                                 | `movl %ecx, %r8d # r8d = k (临时)`             | 复制 k                               |
|                                            |                                              |                                    |
| `1 movl 12(%ebp), %ecx # load y into %ecx` | `movl %r11d, %ecx # load y (r11d) into %ecx` | `y` 的值进入 `%ecx`。                   |
| `2 sall $8, %ecx # y = y << 8`             | `sall $8, %ecx # y = y << 8`                 | `%ecx = y << 8`。                   |
| `3 movl 8(%ebp), %eax # load x into %eax`  | `movl %r10d, %eax # load x (r10d) into %eax` | `x` 的值进入 `%eax`。                   |
| `4 movl 20(%ebp), %edx # load k into %edx` | `movl %r8d, %edx # load k (r8d) into %edx`   | `k` 的值进入 `%edx`。                   |
| `5 imull %edx, %eax # x = x * k;`          | `imull %edx, %eax # eax = eax * edx`         | `%eax = x * k`。                    |
| `6 movl 16(%ebp), %edx # load z into %edx` | `movl %r9d, %edx # load z (r9d) into %edx`   | `z` 的值重新进入 `%edx`。                 |
| `7 andl $65520, %edx # z = z & 0xFFF0`     | `andl $65520, %edx # edx = edx & 0xFFF0`     | `%edx = z & 0xFFF0`。               |
| `8 addl %ecx, %edx # z = z + y`            | `addl %ecx, %edx # edx = edx + ecx`          | `%edx = (z & 0xFFF0) + (y << 8)`。  |
| `9 subl %edx, %eax # return ans=x-z;`      | `subl %edx, %eax # eax = eax - edx`          | `%eax = (x*k) - z`。结果在 `%eax` 中返回。 |
**优点:** x86-64版本用**寄存器到寄存器**的移动（`movl %r11d, %ecx`）取代了IA-32版本中的**内存到寄存器**的加载（`movl 12(%ebp), %ecx`）。**这是最大的性能优势。** 避免了缓慢的**内存访问**


## 8.  `do_loop` 分析
### (1) 分析汇编代码写注释

#Tracking 可以注意一下 `div` 命令的存储位置，和 `decw` 和 `testw` 指令的用法

```
1	movw	8(%ebp),	%bx		# load short x into %bx
2	movw	12(%ebp),	%si		# load short y into %si
3	movw	16(%ebp),	%cx		# load short k into %cx 
4	.L1:						# the start of loop body
5	movw	%si,		%dx		# get value of y, store in %dx
6	movw	%dx,		%ax		# store the value of y into %ax
7	sarw	$15,		%dx		# shift arithmeticly 15 for value of y, store in %dx
8	idiv	%cx					# calculate %ax / %cx, that is y / k. the remainder store in %dx, and the quotient store in %ax
9	imulw	%dx,		%bx		# calculate x = x * (y%k)
10	decw	%cx					# k--;
11	testw	%cx,		%cx		# check if k == 0 (testw: do AND, and set ZF flags)
12	jle		.L2					# if k == 0, hit the end loop condition, jump to .L2
13	cmpw	%cx,		%si		# check if y == k
14	jg		.L1					# if y > k, jump back to loop body .L1
15	.L2	:						# the outside of loop body
16	movswl	%bx,		%eax	# return the ans to %eax register
```

###  (2) **过程分析**：寄存器保存到栈中

- Callee： 被调用者
	- `%ebp`: the pointer to frame
	- `%si`: load the value of parameter

- Caller - Saved
	- `%bx` 
	- `%cx`
	- `%dx`
**准备阶段必须保存到栈中**：
- `%ebp`, `Callee-Saved`: `%bx, %cx, %dx`
### (3) 为什么第 7 行的 DX 寄存器需要算数右移 15 位？
我们后面执行有符号除法 `idiv` 的时候需要符号拓展，`y` 是一个 `16` 位的数，被除数需要一个 `32` 位的数，存在 `DX-AX` 中，我们需要对 `DX` 做有符号拓展，表现为算数右移 15 位。（用 `y` 的最高位符号位填充空位）

### (4) 给出 `do_loop` 函数对应的 `x86-64` 汇编代码

```
# System V AMD64 ABI 
# 参数: x (%rdi), y (%rsi), k (%rdx)
# 结果: %eax

# 1-3. 初始化工作寄存器 (x->r8w, y->r9w, k->r10w)
movw %di, %r8w      
movw %si, %r9w      
movw %dx, %r10w     

# x86-64寄存器：传递参数依次为`rdi, rsi, rdx, rcx, r8, r9`

.L1:
    # 5-7. 准备 idiv 的被除数 (y)
    # y 已经是 16 位，直接移到 AX
    movw %r9w, %ax      # AX = y
    cwd                 # 符号扩展：DX = sign_extend(AX)。相当于 IA-32 的 sarw $15, %dx(convert-word-to-double-word)

    # 8. 除法 y / k
    idivw %r10w         # k 在 %r10w。 商 -> %ax，余数 (y%k) -> %dx

    # 9. 计算 x = x * (y%k)
    imulw %dx, %r8w     # r8w = r8w * (y%k)

    # 10. k--
    decw %r10w          

    # 11-12. 检查循环结束条件 (k <= 0)
    testw %r10w, %r10w  
    jle .L2             

    # 13-14. 检查循环继续条件 (y > k)
    cmpw %r10w, %r9w    # cmp k, y
    jg .L1              # if y > k, jump

.L2:
    # 16. 返回结果
    movswl %r8w, %eax   # 符号扩展 x 到 %eax
    ret
```

## 10 分析函数 `sw()`

已知 `sw()` 的 C 语言框架
```c
int sw(int x)
{
	int v=0;
	switch (x){
		/* some sentences.. */
	}
	return v;
}
```


### (1) 函数 `sw()` 的 `switch` 语句处理部分标号的取值情况如何？

实际是 `judge` `x+3` 和 `7` 的无符号大小关系。
在 `x+3 > 7 (unsigned)` 情况下，会执行 `L7`. 
So, iff `0<= x+3 <=7` , won't jump to `L7`

`jmp *.L8( , %eax, 4)`, `align` is `4`.

`x+3` 是对应的索引
所以
`x+3=0` jump `.L7` (default)
`x+3=1`, jump `.L2`
`x+3=2` jump `.L2`
`x+3=3` jump `.L3`
`x+3=4` jump `.L4`
`x+3=5` jump `.L5`
`x+3=6` jump `.L7`  (default)
`x+3=7` jump `.L6`

所以总结下来，有

- `-2 (.L2)`
- `0 (.L3)`
- `1 (.L4)`
- `2 (.L5)`
- `4 (.L6)`
- `default`
五个分支。


### (2) 标号的取值在什么情况下执行 `default` 分支？那些标号的取值会执行同一个分支？
上面已经给出答案。


## 11 分析 C 语言函数 `test()`

### (1)

```c
unsigned int test(short a, unsigned b, unsigned c, short *p);
```

### (2)

`x86-64`

```
# unsigned int test(short a, unsigned b, unsigned c, short *p);
# 参数: a=%rdi (16位在%di), b=%rsi (32位在%esi), c=%rdx (32位在%edx), p=%rcx (64位)
# 返回值在 %eax 中 (32位)

    # *p = a;
    # 将 a (16位) 存储到 p (%rcx) 指向的地址
    movw %di, (%rcx)     # movw: 移动一个字 (Word, 16位)

    # return b * c;
    # 1. 准备乘法操作数：将 b 移入 %eax (mull指令的隐式被乘数)
    movl %esi, %eax      # movl: 移动一个长字 (Long, 32位)。EAX = b

    # 2. 执行无符号乘法：EAX = EAX * c (%edx)
    # mull %edx 计算 EDX:EAX = EAX * EDX
    mull %edx            # EDX:EAX = b * c

    # 3. 返回
    # 结果的低 32 位）已在 %eax 中，符合 unsigned int 的返回要求。
    ret
```


## 12 `func()` 的 C 语言代码分析
```c
#include <stdio.h>
int funct(void){
	int x, y;
	scanf("%d %d", &x, &y);
	return x-y;
}
```


`IA-32`

```asm
1 funct:
2 	pushl 	%ebp
3	movl	%esp,		%ebp
4	subl	$40,		%esp
5	leal	-8(%ebp),	%eax
6	movl	%eax,		8(%esp)
7	leal	-4(%ebp),	%eax
8	movl	%eax,		4(%esp)	
9	movl	%.LC0,		(%esp)	# Push the pointer to "%d %d" into stack
10	call	scanf				# Assume after scanf run, x=15, y=20
11	movl	-4(%ebp),	%eax	
12	subl	-8(%ebp),	%eax	
13	leave
14	ret
```


假设函数 `funct()` 开始执行后，`R[esp]=0xbc00 0020`, `R[ebp] = 0xbc00 0030`，指向字符串 `"%d %d"` 的指针为 `0x804 c000` 回答下列问题

### （1）执行 3、10 和 13 行的指令后，寄存器 EBP 中的内容分别是？

- `3` 
	- `pushl`：`R[esp] = 0xbc00 0020 -0x4=0xbc00 001c`
	- `%ebp: =%esp `, `R[ebp] = 0xbc00 001c`
- `10`
	- `call scanf`, the caller will push `%ebp` into stack.  
	- But when `scanf` end, the `%ebp` stay the same
	- `R[ebp] = 0xbc00 001c`
- `13`
	- `leave` is used to recover the stack frame of caller
	- this will 
		- `movl %ebp %esp`
			- 恢复 `%esp`, `R[%esp] = 0xbc 001c`
		- `popl %ebp`
			- `R[esp]=0xbc 001c + 0x4 = 0xbc00 0020`
			- pop 出的是 `R[ebp]` 的旧值 `0xbc00 0030` 
			- 赋给 `%ebp`，即 `R[ebp]=0xbc00 0030`


### (2) 执行第 3、10 和 13 行的指令后，寄存器 ESP 的内容分别是？

- `3`
	- `R[esp] = 0xbc00 0020 -0x4=0xbc00 001c`
- `10`
	- `4 subl	$40,		%esp` 
		- `R[esp] = 0xbc00 001c - 0x28 =  0xbbfffff4`
	- 执行 `call scanf`
		- **会将返回地址**（第 11 行）压入栈中
		- 将 `%esp -4`
		- `R[esp] = 0xbbff fff4 -4=0xbbff fff0`
- `13`
	- `leave`
	- do `movl %ebp %esp`
	- `R[esp] = 0xbc00 001c`
	- `popl`,
		- `R[esp] = 0xbc00 001c +4 =0xbc00 0020` 

### (3) 局部变量 `x` 和 `y` 所在存储单元的地址分别是？

**计算地址：**

1. 局部变量 $y$ 的地址 ($\text{EBP} - 4$)：
    $$A_y = 0\text{xbc00 001c} - 4 = 0\text{xbc00 0018}$$
2. 局部变量 $x$ 的地址 ($\text{EBP} - 8$)：
    
    $$A_x = 0\text{xbc00 001c} - 8 = 0\text{xbc00 0014}$$
			
### (4) 画出执行第 10 行指令后的 `func` 的栈帧，给出栈帧中的内容和地址


```
        内存地址 (Hex)       内容 (Hex)       内容描述 (4 字节/行)
|----------------------|------------------|---------------------------------------|
| 0xbc00 0024          | (Caller Data)    | 调用者数据或下一个栈帧 (高地址)       |
| 0xbc00 0020          | (Return Addr)    | 调用 funct 的返回地址 (行 14 的地址)  |
|======================|==================|=======================================|
| 0xbc00 001c <-- %EBP | 0xbc00 0030      | 旧 %EBP (调用者的帧基址)         |
| 0xbc00 0018          | 0x0000 0014      | 局部变量 y 的值 (20) |
| 0xbc00 0014          | 0x0000 000f      | 局部变量 x 的值 (15) |
| 0xbc00 0010          | 0xbc00 0018      | scanf 参数 3: 变量 y 的地址 (&y)      |
| 0xbc00 000c          | 0xbc00 0014      | scanf 参数 2: 变量 x 的地址 (&x)      |
| 0xbc00 0008          | 0x804c 0000      | scanf 参数 1: 格式字符串地址 (%.LC0)  |
| 0xbc00 0004          | (未使用/Padding) | funct 为局部变量/参数预留的栈空间     |
| 0xbc00 0000          | (未使用/Padding) | funct 为局部变量/参数预留的栈空间     |
| 0xbcf ffffc          | (未使用/Padding) | funct 为局部变量/参数预留的栈空间     |
| 0xbcf ffff8          | (未使用/Padding) | funct 为局部变量/参数预留的栈空间     |
| 0xbcf ffff4          | (未使用/Padding) | funct 为局部变量/参数预留的栈空间     |
| 0xbcf ffff0 <-- %ESP | (Return Addr)    | scanf 函数的返回地址 (行 11 的地址) |
|----------------------|------------------|---------------------------------------|
| 0xbcf fffec          | (Callee Data)    | scanf 函数使用的栈空间 (低地址)       |
```


- 注意分清层次：`func` 作为 `scanf` 的 `caller`, 自己也可能是某个函数的 `callee`，所以其 `%EBP` 上部 `+4` 是自己的返回地址。，同时 ` %EBP+8 ` 可以获得自己的调用者的**数据**。


## 17 函数 `st_ele()` 的分析（`L,M,N` 是 `#define` 的常数）

### (1) 得到 LMN 的值
```c
int a[L][M][N];

int st_ele(int i, int j, int k, int *dst){
	*dst = a[i][j][k];
	return sizeof(a);
}
```


注意这条命令
```asm
leal	(%edx,	%edx,	8),	%edx 
```

实际是一种编译器便捷实现乘法的方法。
`R[EDX]=EDX + 8*EDX=9*EDX`

对 ASM 代码逐行注释理解，同时认识到多维数组其实是分别计算各维度偏移量再叠起来即可。
然后 `L` 可以用 `sizeof` 得到的元素总数求得
int 是 `4` 字节，故 `L=`
`L=72/sizeof(int)=18, M=7, N=9`

![IMG_9291.JPG|400](https://kold.oss-cn-shanghai.aliyuncs.com/IMG_9291.JPG)

### （2）写出函数对应的 `x86-64` 代码

```asm
# %rdi=i, %rsi=j, %rdx=k, %rcx=dst
leaq 	(%rsi, %rsi, 8), 	%r8		# R8 = 9j
salq 	$6,					%rdi 	# %rdi = 64i
decq		%rdi						# %rdi-- = 63i
addq	%rdi,				%r8		# r8 = 9j + 63i
addq 	%r8,				%rdx	# rdx = k + 9j + 63i
movslq	a( , %edx, 4)		%rax	# 符号拓展数组的元素，使用movslq
movq	%rax,				(%rcx)	# *dst = R[rax]
movq	%4536,				%rax	# return sizeof()
ret
```

## 20 假设联合体类型 `utype` 的定义如下：
```c
typedef union{
	struct {
		int x;
		short y;
		short z;
	}	s1;
	struct{
		short a[2];
		int b;
		char *p;
	}	s2;
} utype;
```

假设存在具有如下形式的一组函数

```c
void getvalue(utype	*uptr,	TYPE *dst){
	*dst = EXPR;
}
```


该组函数用于计算不同的表达式 `EXPR` 的值。
返回值的数据类型根据表达式的类型确定。`getvalue` 的入口参数 `uptr` 和 `dst` 分别装入寄存器 `EAX` 和 `EDX` 中，仿照例子在不同表达式的 TYPE 类型以及表达式对应的 `IA-32` 序列。


| 表达式 `EXPR`               | `TYPE` 类型 | 汇编指令序列                                                                       |
| ------------------------ | --------- | ---------------------------------------------------------------------------- |
| `uptr->s1.x`             | `int`     | `movl (%eax), %eax;`, `movl %eax, (%edx)`                                    |
| `uptr->s1.y`             | `short`   | `movw 4(%eax), %ax`, `movw %ax, (%edx)`                                      |
| `&uptr->s1.z`            | `short`   | `leal 6(%eax), %eax`, `movl %eax, (%edx)`                                    |
| `uptr->s2.a`             | `short*`  | `movl (%eax), %eax`, `movl %eax, (%edx)`                                     |
| `uptr->s2.a[uptr->s2.b]` | `short`   | `movl 4(%eax), %ecx`, `movswl (%eax, %ecx, 2), %eax`, `movw %ax, (%edx)`<br> |
| `*uptr->s2.p`            | `char *`  | `movl 8(%eax), %ecx`, `movb (%ecx) %al`, `movb %al, (%edx)`                  |
>  **注意**：最后一条，你不能直接使用 `movb (%ecx), (%edx)`，这是因为不允许直接对内存的操作，必须通过寄存器作为媒介。
>  注意取地址存储的内存值要用 `(%edx)` 而不是 `(%dx)` ,因为我们使用的内存地址是 `32bit` 的


## 21 分别给出在 `IA-32+Linux` , `x86-64+linux` 平台上，下列各个结构体类型中每个成员的偏移量、结构体总大小以及结构体起始位置的对齐要求。
```c
{1} struct S1 {short s; char c; int i; char d;};
{2} struct S2 {int i; short s; char c; char d;};
{3} struct S3 {char c; short s; int i; char d;};
{4} struct S4 {short s[3]; char c;};
{5} struct S5 {char c[3]; short* s; int i; char d; double e;};
{6} struct S6 {struct S1 c[3]; struct S2 *s; char d;}
```


### IA-32
IA-32 最大对齐策略为 `4` 字节
```
IA-32和x86064在最大对齐不超过4字节的情况是相同的这里略去。

研究S5:S6

--- 结构体 S5 信息 ---
结构体总大小 (sizeof): 24 字节
结构体对齐要求 (Max Align): 4 字节
-------------------------------
c[0] 偏移量: 0
s* 偏移量: 4
i 偏移量: 8
d 偏移量: 12
e 偏移量: 16

--- 结构体 S6 信息 ---
结构体总大小 (sizeof): 44 字节
# sizeof(S6) = sizeof(S1)*3 + 4  + 1 + padding(3)=36+4+4=44
结构体对齐要求 (Max Align): 4 字节
-------------------------------
c[0] 偏移量: 0
s* 偏移量: 36
d 偏移量: 40
```

### x86-64
在我的 `ubuntu 22.04 x86-64` 上做实验，得到
```
C 语言结构体内存布局分析 (当前平台):
======================================

--- 结构体 S1 信息 ---
结构体总大小 (sizeof): 12 字节
结构体对齐要求 (Max Align): 4 字节
-------------------------------
s 偏移量: 0
c 偏移量: 2
i 偏移量: 4
d 偏移量: 8

--- 结构体 S2 信息 ---
结构体总大小 (sizeof): 8 字节
结构体对齐要求 (Max Align): 4 字节
-------------------------------
i 偏移量: 0
s 偏移量: 4
c 偏移量: 6
d 偏移量: 7

--- 结构体 S3 信息 ---
结构体总大小 (sizeof): 12 字节
结构体对齐要求 (Max Align): 4 字节
-------------------------------
c 偏移量: 0
s 偏移量: 2
i 偏移量: 4
d 偏移量: 8

--- 结构体 S4 信息 ---
结构体总大小 (sizeof): 8 字节
结构体对齐要求 (Max Align): 2 字节
-------------------------------
s[0] 偏移量: 0
c 偏移量: 6

--- 结构体 S5 信息 ---
结构体总大小 (sizeof): 32 字节
结构体对齐要求 (Max Align): 8 字节
-------------------------------
c[0] 偏移量: 0
s* 偏移量: 8
i 偏移量: 16
d 偏移量: 20
e 偏移量: 24

--- 结构体 S6 信息 ---
结构体总大小 (sizeof): 56 字节
结构体对齐要求 (Max Align): 8 字节
-------------------------------
c[0] 偏移量: 0
s* 偏移量: 40
d 偏移量: 48
```


### code for test
```c
#include <stdio.h>
#include <stddef.h> // 包含 offsetof 宏定义
#include <stdalign.h>

// 预先定义所有结构体

// {1}
struct S1 {
    short s;
    char c;
    int i;
    char d;
};

// {2}
struct S2 {
    int i;
    short s;
    char c;
    char d;
};

// {3}
struct S3 {
    char c;
    short s;
    int i;
    char d;
};

// {4}
struct S4 {
    short s[3];
    char c;
};

// {5}
struct S5 {
    char c[3];
    short* s;
    int i;
    char d;
    double e;
};

// {6} (修正了 'strcut' 为 'struct')
struct S6 {
    struct S1 c[3];
    struct S2 *s;
    char d;
};

// 宏定义：用于获取结构体对齐要求（非标准但常用）
// 原理：结构体中第一个成员的偏移量即是结构体本身的对齐要求。
//       但在有填充的情况下，更准确的对齐要求应取结构体中最大对齐成员的对齐要求。
//       这里我们使用一个辅助函数/宏来获取编译器报告的对齐值（如果可用）
//       或者简化为最大基本类型对齐。为了跨平台，我们打印结构体大小和偏移量。
//       在许多 Unix/Linux 平台上，对齐要求约等于 (sizeof(Struct) % 最大对齐成员大小) == 0。
// 鉴于跨平台差异，我们只打印最精确的偏移量和总大小。

void print_struct_info(const char *name, size_t size, size_t max_align) {
    // 假设结构体的对齐要求是其成员的最大对齐要求（通常正确）
    printf("\n--- 结构体 %s 信息 ---\n", name);
    printf("结构体总大小 (sizeof): %zu 字节\n", size);
    printf("结构体对齐要求 (Max Align): %zu 字节\n", max_align);
    printf("-------------------------------\n");
}

int main() {
    printf("C 语言结构体内存布局分析 (当前平台):\n");
    printf("======================================\n");

    // {1} struct S1
    size_t s1_max_align = (sizeof(int) > sizeof(short)) ? sizeof(int) : sizeof(short);
    print_struct_info("S1", sizeof(struct S1), s1_max_align);
    printf("s 偏移量: %zu\n", offsetof(struct S1, s));
    printf("c 偏移量: %zu\n", offsetof(struct S1, c));
    printf("i 偏移量: %zu\n", offsetof(struct S1, i));
    printf("d 偏移量: %zu\n", offsetof(struct S1, d));

    // {2} struct S2
    size_t s2_max_align = (sizeof(int) > sizeof(short)) ? sizeof(int) : sizeof(short);
    print_struct_info("S2", sizeof(struct S2), s2_max_align);
    printf("i 偏移量: %zu\n", offsetof(struct S2, i));
    printf("s 偏移量: %zu\n", offsetof(struct S2, s));
    printf("c 偏移量: %zu\n", offsetof(struct S2, c));
    printf("d 偏移量: %zu\n", offsetof(struct S2, d));

    // {3} struct S3
    size_t s3_max_align = (sizeof(int) > sizeof(short)) ? sizeof(int) : sizeof(short);
    print_struct_info("S3", sizeof(struct S3), s3_max_align);
    printf("c 偏移量: %zu\n", offsetof(struct S3, c));
    printf("s 偏移量: %zu\n", offsetof(struct S3, s));
    printf("i 偏移量: %zu\n", offsetof(struct S3, i));
    printf("d 偏移量: %zu\n", offsetof(struct S3, d));

    // {4} struct S4
    size_t s4_max_align = sizeof(short);
    print_struct_info("S4", sizeof(struct S4), s4_max_align);
    printf("s[0] 偏移量: %zu\n", offsetof(struct S4, s)); // 数组的首元素偏移量
    printf("c 偏移量: %zu\n", offsetof(struct S4, c));

    // {5} struct S5
    // 注意：这里的 max_align 应该取 double 或 short* (指针) 中较大的对齐要求
    size_t s5_max_align = (sizeof(double) > sizeof(short*)) ? alignof(double) : alignof(short*);
    print_struct_info("S5", sizeof(struct S5), s5_max_align);
    printf("c[0] 偏移量: %zu\n", offsetof(struct S5, c));
    printf("s* 偏移量: %zu\n", offsetof(struct S5, s));
    printf("i 偏移量: %zu\n", offsetof(struct S5, i));
    printf("d 偏移量: %zu\n", offsetof(struct S5, d));
    printf("e 偏移量: %zu\n", offsetof(struct S5, e));

    // {6} struct S6
    // 注意：这里的 max_align 应该取 struct S2 *s 或 struct S1 的对齐要求中较大的
    // 假设 struct S2 *s 的对齐要求是 alignof(struct S2 *)，即指针的对齐要求
    size_t s6_max_align = (alignof(struct S1) > alignof(struct S2 *)) ? alignof(struct S1) : alignof(struct S2 *);
    print_struct_info("S6", sizeof(struct S6), s6_max_align);
    printf("c[0] 偏移量: %zu\n", offsetof(struct S6, c));
    printf("s* 偏移量: %zu\n", offsetof(struct S6, s));
    printf("d 偏移量: %zu\n", offsetof(struct S6, d));
    
    return 0;
}

/*
注意：
1. 为了打印结构体对齐要求，我使用了 C11 标准的 alignof 运算符。如果您的编译器不支持 C11，
   请将代码中的 alignof(...) 替换为 sizeof(int) 或 4（对于 32 位）/ 8（对于 64 位）来近似。
2. 运行此代码将输出您**当前编译环境**下的实际布局，这可能与理论分析（IA-32/x86-64 Linux 默认）略有不同，
   但通常会匹配其中一个平台。
*/
```


##  24 假定函数 `abc()` 的入口参数有 `a, b, c`，每个参数都可能是带符号整数或无符号整数类型，而且它们的长度也可能不同。该函数具有如下过程体
```c
*b += c;
*a += *b;
```

在 `x86-64` 机器上编译后的汇编代码如下
```asm
1	abc:
2	addl	(%rdx),	%edi
3	movl	%edi,	(%rdx)
4	movslq	%edi,	%rdi
5	addq	%rdi,	(%rsi)
6	ret
```

分析以上汇编代码以确定三个入口参数的顺序和可能的数据类型，写出函数 `abc` 可能的函数原型


### Solution
阅读此代码，`addl (%rdx), %edi`,  `%rdx` 通常是第 3个参数的存储 `reg`，我们对其取值，所以可能这是一个指针型参数，从 `addl` 知道应该是一个 `32` 位的

`(%rdx)`，在源代码中应该是 `b`，那么 `%edi` 就代表 `c`，后面用到了 `%rdi`，属于是寄存器复用。

后面有 `movslq %edi %rdi` 的操作，说明 `c` 是有符号类型的 32 位。

`%rsi` 应该是一个指针类型，且是最后一步操作结果存入 (`%rsi`)，显然代表 `a`

在回忆：[[ICS - Section 4 程序的机器级表示#`x86-64` 的过程调用|x86-64的过程调用]]知识。`%rdi -> first param, %rsi -> second param, %rdx -> 3rd param`

则不难写出

```c
void abc(int c, long *a, long *b);

// or we can change unsigned or not of a and b. Then we get totally 4 possible answers.

void abc(int c, unsigned long *a, long *b);

void abc(int c, unsigned long *a, unsigned long *b);

void abc(int c,  long *a, unsigned long *b);
```

## 26 假设你要维护一个大型 C 语言程序，其部分代码如下

```c
typedef struct {
	unsigned	 l_data;
	line_struct	 x[LEN];
	unsigned	 r_data; 
} str_type;

void proc(int i, str_type *sptr){
	unsigned val = sptr->l_data +sptr->r_data;
	line_struct *xptr = &sptr->x[i];
	xptr->a[xptr->idx] = val;
}
```

编译时常量 LEN 以及结构类型 `line_struct` 的声明都在一个你无权访问的文件中，但是有代码的 `.o` 版本（可重定位目标）文件，通过 OBJDUMP 反汇编该文件后，得到函数 `proc` 对应的 IA-32 反汇编结果如图所示，根据反汇编结果推断常量 `LEN` 的值以及结构类型 `line_struct` 的完整声明。（假设其中只有成员 `a` 和 `idx`）


![ca20e0771d04ccca550ea7d2de91bb4e_720.png|700](https://kold.oss-cn-shanghai.aliyuncs.com/ca20e0771d04ccca550ea7d2de91bb4e_720.png)


解
```asm
- the first 6 lines, is preparing.
- i -> %eax, sptr -> %ecx
7	imul 	$0x1c, 	%eax, 	%ebx 			# R[ebx] = 28i
8	lea		0x0( , %eax, 8),		%edx	# R[edx]=8R(eax)=8i
9	sub		%eax,	%edx					# R[edx]=7i
10	add		0x4(%ecx, %ebx, 1), 	%edx	# R[edx]=sptr + R[ebx] + 4 + R[edx]=sptr + 28i +7i +4  
// sptr->x[i].a[sptr->x[i].idx]
// so the 7i is after get x[i], then get the idx.
// It means in the struct, the `idx` is behind the `a`, and a is `7` numbers.
11	mov		0xc8(%ecx),		%eax			# (R[eax])=Mr(R[ecx]+0xc8=sptr+200 )// address of r_data, now R[eax]=sptr->r_data
12 	add		(%ecx),	%eax					# R[eax]=Mr(R[ecx]) + R(eax), (val = l_data + r_data), value in %eax
13	mov		%eax,	0x8(%ecx, %edx, 4)		# xptr->a[xptr->idx] = val; 
											# note that idx is by xptr->a[xptr->idx],  so %edx is actually the idx.
```
---

- **Specially** Note the line `10` , we get a strange `28i + 7i`. Here is a brief analyse
	**内存地址 $\mathbf{M}[\ldots]$:**
	
	- `%ecx` = $\text{sptr}$
	    
	- `%ebx` = $28i$ ( $\text{Offset}(\text{x}[\text{i}])$ )
	    
	- `0x4` = $\text{Offset}(\text{l\_data})$
	    
	- 所以，$\mathbf{M}[4 + \text{ECX} + \text{EBX}]$ 访问的地址是：
	    
	    $$\text{sptr} + 4 + 28i$$
---

- from line `10` we know `sizeof(line_struct)=28`,  and `sizeof(x)=LEN*sizeof(line_struct) = 200 - 4=196`, so `LEN` would be `196/28 = 7`
- from  line `13` we know we need a `offset = offset(l_data) + offset(idx)`  to visit `a`,
	- SO `int idx` is the first member in `line_struct`
	- and there is a `scale=4`, means `a is int a[]` 
- 

so:
```c
struct line_struct
{
	int idx;
	int a[6];
}
```

`int idx` is `4` byte, so `a[N], N=(28-4)/4=6`