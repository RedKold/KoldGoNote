---
completed: "true"
course: 计算机系统
tags:
  - assignment
author: 231275036-朱晗
---
作业清单：
- 第三章课后习题
- `3,4,5,7,8,10`
## 3 对于以下 AT&T 指令，根据操作数的长度确定对应指令助记符中的长度后缀，并说明操作数的寻址方式

```
mov		8(%ebp, %ebx, 4), %ax
mov 	%al, 12(%ebp)
add		(, %ebx, 4), %eax
or 		(%ebx), %dh
push	%rcx
mov 	$0xFFF0, %eax
test 	%rax,	%rax
lea		(%ebx, %esi),	%eax
```
- `mov`
	- `l`
	- 源操作数：基址+比例变址+位移
	- 目的操作数：基址+偏移量
- `mov`
	- `b`
	- 寄存器寻址
	- 基址+偏移
- `add`
	- `l`
	- 比例变址
	- 寄存器寻址 
- `or`
	- `l`
	- 间接寻址 
	- 寄存器寻址 
- `push`
	- `q
	- 寄存器寻址
- `mov`
	- `l`
	- 立即数寻址
	- 寄存器寻址
- `test`
	- `q`
	- 寄存器寻址
	- 寄存器寻址
- `lea`
	- `l`
	- 变址寻址

## 4 使用汇编器处理以下 IA-32/x86-64 中 AT&T 格式代码时都会出现错误，请说明每一行代码存在什么错误

1. `movl   0xFF,  (%eax)`
	1.  `0xFF` 无法解读，需要加括号作为直接寻址，或者加 `$` 作为立即数寻址
2. `movb   %ax, 12(%eax)`
	1. `movb`  is `byte` opr cannot deal ` %eax ` or ` %ax `.
3. `addl   %ecx, $0xF0`
	1. `$0xF0` is a imm value, you cannot add a value to a imm. it must be a regiter or memory addr
4. `orw   $0xFFFF0, (%ebx)`
	1. `orw` is a `word(16bit)` operation, but `$0xFFFF0>0xFFFF`,
5. `addb   $0xF8, (%dl)`
	1. can't visit memory addr from a `byte` register `%dl`
6. `movl   %bx, %eax`
	1. src oprand is `word`, but operator is `movl`, `long`
7. `andl   %esi, %esx`
	1. register `%esx` doesn't exist
8. `movq   8(%ebp, , 4), %rax`
	1. the second argument cannot be omitted
	2. `the index register` is missing!
9. `leaq   20(%rdi, %rsi), %eax`
	1. the dst oprand is `32bit`, but src oprand is a `64bit` address. `64bit` cannot store in `32bit`

## 5 汇编指令表达赋值
假设变量 `x` 和 `ptr` 的类型声明如下
```c
src_type x;
dst_type *ptr;

*ptr=(dst_type) x;
```
若 `x` 在寄存器 `EAX` 或 `AX` 或 `AL` 中，`ptr` 在寄存器 `EDX` 中，则对于表给出的 `src_type` 和 `dst_type` 类型组合，写出实现上述赋值语句的 IA-32 机器级代码 (AT&T) 格式

- 本题需要注意
	- 位数问题
	- 符号拓展问题
		- `MOVS`，低位向高位带符号拓展
		- `MOVZ`，无符号拓展。高位 0

| `src_type`    | `dst_type` | 机器级表示                 |
| ------------- | ---------- | --------------------- |
| char          | int        | `movsbl  %AL, (%EDX)` |
| int           | char       | `movb  %AL, (%EDX)`   |
| int           | unsigned   | `movl  %EAX, (%EDX)`  |
| short         | int        | `movswl  %AX, (%EDX)` |
| unsigned char | unsigned   | `movzbl  %AL, (%EDX)` |
| char          | unsigned   | `movsbl  %AL, (%EDX)` |
| int           | int        | `movl   %EAX, (%EDX)` |



## 7 假设在 IA-32 系统中以下地址以及寄存器中存放的机器数如表所示


1. `addl   (%eax), %edx`
	1. `%EDX: 0x0804 9380`
2. `subl   (%eax, %ebx), %ecx`
	1. `src oprand`: `0x0804 9300 + 0x0000 0100=0x0804 9400`
	2. 读地址，地址存储为 `0x8000 0008`
	3. 做减法，`%ecx: 0x0000 0010`, 得到结果为 `0x8000 0008`
	4. `%ecx: 0x8000 0008`
	5. 发生溢出 `OF=1` 
3. `orw   4(%eax, %ecx, 8), %bx`
	1. `src oprand`： `%eax+8*%ecx+4= 0x8049384`
	2. 读地址，存储 `0x80f7 ff00`
	3. `%bx` 存储低位 `0x0100`
	4. 做 `orw` 得到 `0xff00`
	5. `%ebx: 0x0000 ff00`
	6. 奇偶：`PF=1`
4. `testb   $0x80, %dl`
	1. 结果为 `0`
	2. `ZF=0`
5. `imull  $32, (%eax, %edx), %ecx` 
	1. `%eax+%edx =0x08049380`
	2. 读地址，存储, `0x908f 12a8`
	3. `32=2^5, 32*0x908f 12a8=0x11e25500`
	4. 发生溢出，`OF=1`
	5. 发生进位，`CF=1`
	6. `%ecx` 存储 `0x11e25500`
6. `mulw %bx`
	1. 另一个操作数隐含在累加器 `AX=0x9300`
	2. `%bx=0x0100`
	3. 乘积结果 `0x9300 * 0x0100 = 0x0093 0000（溢出）=0x0000` (32bit, store in dx-ax)
	4. `%edx=0x0000 0093`
	5. `%eax = 0x0000 0000`
	6. `OF=1, CF=1`
7. `decw  %cx`
	1. `%cx = 0x0010`
	2. `%cx -1 = 0x0010-1=0x0001`
	3. `%ecx = 0x0000 0001`
## 8 已知 IA-32 采取小端方式，根据给出的 IA-32 代码反汇编结果（部分信息用 `x...x` 表示）回答问题。

`1`. 已知 `je` 指令的操作码为 `0111 0100` `je` 指令的跳转目标地址是什么？`call` 指令中的跳转目标地址为 `0x80483b1` 是如何反汇编出来的？
- `0x804838c: 74 08`, `74` 即操作码，则 `08` 为相对偏移量 `0x08`, 目标地址 =` 0x804838e + 0x08 = 0x8048396`

- `call`： `e8 1e 00 00 00`
- `e8` 即操作码，`1e 00 00 00 ` 是小端序存储的跳转偏移，大小为 `0x00 00 00 1e`
- 下一条地址：`0x804838e+5=0x8048393`
- `0x8048393 + 0x1e = 0x80483b1`
- 至此，反汇编成功。

`2`. 已知 `jb` 指令的操作码为 `0111 0010`， `jb` 指令的跳转目标地址是什么？`movl` 指令中的目的地址是如何反汇编出来的？

`8048390: 72 f6, jb`
`0xf6: 8bit有符号偏移量`. `0xf6=-(0x0a)=-10`
下一条指令是 `8048392 movl`, 所以跳转 `0x8048392 -10=0x804830= 0x8048388`
跳转目标地址为 `0x8048388`
`movl` 指令反汇编：
`8048392: c6 05 00 a8 04 08 01 movl`
- `c6`: 将一个 `8bit` 立即数写入寄存器单元
- `05`：标记接下来是一个 `32` 位的绝对地址
- 小端序：`0x0804 a800` 即地址
- `0x01` ：存入的立即数

`3`. `jle` 指令的操作码为 `0111 1110`，`jle` 和 `mov` 指令的地址分别是什么？
`jle` 的相对跳转偏移为 `0x16`，设 `jle` 地址为 `A`, `mov` 地址为 `B`, 则 `A+2=B`, `B+0x16=0x80492e0`
`B=0x80492ca`, `A=0x80492c8`


`4`. 已知 `jmp` 指令的跳转目标地址采取相对寻址方式，`jmp` 指令操作码为 `1110 1001`，其跳转地址是什么？

`8048296: e9 00 ff ff ff jmp xxxxxxx`
`804829b: 29 c2 sub %eax, %edx`

`jmp` 指令相对寻址，相对偏移为 `0xffff ff00`, 其大小为 `-256=-0x100`
跳转到 `0x804829b - 256= 0x804819b`



## 10 汇编指令练习
假设 `x86-64` 系统某程序中变量 `x` 和 `ptr` 的类型声明如下：

```
src_type x;
dst_type *ptr;
```
这里 `src_type` 和 `dst_type` 是用 `typedef` 声明的数据类型。有以下一个 C 语言赋值语句
`*ptr=(dst_type) x;`
若 `x` 存储在寄存器 `RAX` 或 `EAX` 或 `AX` 或 `AL` 中，`ptr` 存储在寄存器 `RDX` 中，则对于表 `3.13` 给出的 `src_type` 和 `dst_type` 的类型组合，写出实现上述赋值语句对应的汇编指令。


| `src_type`      | `dst_type`      | AT&T                                  |
| --------------- | --------------- | ------------------------------------- |
| `char`          | `long`          | `movsbq %al, %rax; movq %rax (%rdx)`  |
| `int`           | `char`          | `movb， %al, (%rdx)`                   |
| `int`           | `unsigned long` | `movslq %eax, %rax; movl %rax (%rdx)` |
| `short`         | `int`           | `movswl %ax, %eax; movl %eax, (%rdx)` |
| `unsigned char` | `unsigned`      | `movzbl %al, %eax; movl %eax, (%rdx)` |
| `char`          | `unsigned long` | `movsbq %al, %rax; movq %rax, (%rdx)` |
| `unsigned long` | `int`           | `movl %eax, (%rdx)`                   |

如果 `src_type` 是有符号的，需要进行有符号拓展，C 语言标准要求我们要保证负数转化为无符号值，数值上等价于负数的补码表示。