## Section 4 程序的机器级表示
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027210140.png)

- 举个例子：
### 函数调用的栈帧

非常好！这是你在真正理解函数调用底层机制的关键一步。下面我帮你详细解释：**过程（函数）是如何通过 `EBP` 和偏移量访问参数的**。

---

### 🧩 一、背景：函数调用时的栈结构

在 **IA-32（x86 32 位）** 调用约定中（如 `cdecl`），函数参数和返回地址都是放在栈上的。  
当一个函数被调用时，栈大致如下：

```
高地址 →
+-----------------+
| 参数2 (arg2)    |  ← [ebp + 12]
+-----------------+
| 参数1 (arg1)    |  ← [ebp + 8]
+-----------------+
| 返回地址 (ret)  |  ← [ebp + 4]
+-----------------+
| 上一个 EBP 值   |  ← [ebp]
+-----------------+
| 局部变量1       |  ← [ebp - 4]
| 局部变量2       |  ← [ebp - 8]
↓ 低地址
```

---

### 🧠 二、函数栈帧建立过程（标准序言）

编译器通常会在函数开头生成如下代码：

```asm
push   %ebp          ; ① 保存调用者的 EBP
mov    %esp, %ebp    ; ② 建立新的栈帧基址
sub    $N, %esp      ; ③ 为局部变量预留空间
```

执行完这三步后：
- `%esp`:  means Stack Pointer
	- **指向栈顶**

- `%ebp` 固定指向当前函数的栈帧基准位置；
	- Base Pointer 
- 参数、返回地址、局部变量的相对位置都固定了。
    

---

### 🧩 三、通过 `EBP + 偏移` 访问参数

|访问内容|地址表达式|示例说明|
|---|---|---|
|返回地址|`[ebp + 4]`|`call` 指令自动压入|
|第一个参数|`[ebp + 8]`|`push arg1` 后第一个参数|
|第二个参数|`[ebp + 12]`|第二个参数|
|第三个参数|`[ebp + 16]`|以此类推|
|局部变量1|`[ebp - 4]`|栈往下（低地址）|
|局部变量2|`[ebp - 8]`|同理|

---

### 🧩 四、举个例子

C 代码：

```c
int add(int a, int b) {
    int sum = a + b;
    return sum;
}
```

对应汇编（典型 IA-32）：

```asm
add:
    push   %ebp
    mov    %esp, %ebp
    sub    $4, %esp        ; 为局部变量 sum 分配空间

    mov    8(%ebp), %eax   ; eax = a
    add    12(%ebp), %eax  ; eax = a + b
    mov    %eax, -4(%ebp)  ; sum = eax
    mov    -4(%ebp), %eax  ; return sum

    leave
    ret
```

📘 对照理解：

- `8(%ebp)` → 第一个参数 `a`
    
- `12(%ebp)` → 第二个参数 `b`
    
- `-4(%ebp)` → 局部变量 `sum`
    
- `leave` 相当于：
    
    ```asm
    mov %ebp, %esp
    pop %ebp
    ```
    

---

### 🧠 五、为什么要用 EBP？（ESP 会变）

因为在函数执行过程中：

- `ESP` 会因为 `push`、`pop`、`sub`、`call` 等操作而变化；
	- **作为一个栈顶**pointer，`push` 会让其减少一个字节。
    
- 而 `EBP` 是**固定基准点**，可以稳定地定位参数和局部变量。
	- **当前函数栈帧的基址**.
- **函数开始时 (Prologue):**
    
    - `pushl %ebp`：保存调用者 (caller) 的 `%ebp`。
        
    - `movl %esp, %ebp`：将 `%esp` 的当前值复制给 `%ebp`，建立**当前函数**的新"基准点"。
        
- **函数执行中：**
    
    - `%ebp` 保持不变。
        
    - 函数参数可以通过**正向偏移**访问 (如 `8(%ebp)`, `12(%ebp)`)。
        
    - 局部变量可以通过**负向偏移**访问 (如 `-4(%ebp)`, `-8(%ebp)`)。
        
- **函数结束时 (Epilogue):**
    
    - `leave` (或 `movl %ebp, %esp` + `popl %ebp`)：恢复调用者的 `%esp` 和 `%ebp`。
    

---

✅ **一句话总结：**

> 函数通过 `EBP` 建立一个稳定的参考点，然后用固定的偏移量访问参数（`[EBP+8]`, `[EBP+12]`）和局部变量（`[EBP-4]`, `[EBP-8]`）。

---

### `x86-64` 的过程调用

- 在 `IA-32` 中通常使用帧指针寄存器 `EBP` 指向栈帧底部。但是在 `x86-64` 中不再使用帧指针寄存器 `RBP` 访问入口参数和自动变量，而是使用 **帧**指针寄存器 `RSP` 作为 **基址**寄存器来栈帧中的信息，`RBP` 作为普通寄存器使用。
- 传递入口参数的寄存器依次为 `RDI, RSI, RDX, RCX, R8 和 R9`

| **参数顺序**    | **寄存器 (64-bit)** | **寄存器 (32-bit)** | **对应 C 语言参数** |
| ----------- | ---------------- | ---------------- | ------------- |
| **第 1 个参数** | $\mathbf{\%rdi}$ | $\%edi$          | `a`           |
| **第 2 个参数** | $\mathbf{\%rsi}$ | $\%esi$          | `b`           |
| **第 3 个参数** | $\mathbf{\%rdx}$ | $\%edx$          | `c`           |
| **第 4 个参数** | $\mathbf{\%rcx}$ | $\%ecx$          | `d`           |
| **第 5 个参数** | $\mathbf{\%r8}$  | $\%r8d$          | `e`           |
| **第 6 个参数** | $\mathbf{\%r9}$  | $\%r9d$          | `f`           |

**性能优势**：相比于 `IA-32`，通用寄存器 `GPRs` 的个数增加到 16，所以前六个参数可以直接通过 **寄存器**传递，而不需要压栈。这样提高了效率。

- **传递参数**：如果参数是 `32/16/8` 位，则参数被置于对应宽度的寄存器部分。例如，第一个入口参数是 `char`，则放在 `RDI` 对应字节宽度的 `DIL` 中。若入口参数是 `short`，则放在 `RAX` 对应 16 位宽度的 `AX` 中。

- 超过 `6` 个的参数，用栈传递。
**对于存放最终结果的 `RAX`** 寄存器，在返回结果之前，也可以当做普通的寄存器使用。

- `call` 和 `ret`
	- 在 `x86-64` 中，函数调用 `call` 指令讲一个 `64` 位的返回地址保存在栈中。故包含 `R[rsp] <- R[rsp]-8` 操作
	- 返回指令 `ret` 也是从栈中取回 `64` 位返回地址，故包含执行 `R[rsp] <- R[rsp]+8` 操作。 

#### example: `x86-64 + linux`

这里有一个简单的 C 语言程序

```c
// simple.c
#include <stdio.h>

// 这是一个简单的函数：计算两个整数的和
int add(int a, int b) {
    return a + b;
}

int main() {
    int x = 10;
    int y = 5;
    
    int result = add(x, y);
    
    printf("The sum of %d and %d is: %d\n", x, y, result);
    
    return 0;
}
```


汇编代码如下 (`gcc -S simple.c -o simple.s`)
```asm
.file	"simple.c"
	.text
	.globl	add
	.type	add, @function
add:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movl	%edi, -4(%rbp)
	movl	%esi, -8(%rbp)
	movl	-4(%rbp), %edx
	movl	-8(%rbp), %eax
	addl	%edx, %eax
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	add, .-add
	.section	.rodata
.LC0:
	.string	"The sum of %d and %d is: %d\n"
	.text
	.globl	main
	.type	main, @function
main:
.LFB1:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$16, %rsp
	movl	$10, -12(%rbp)
	movl	$5, -8(%rbp)
	movl	-8(%rbp), %edx			# 将rbp -8 即第2个参数`b`入寄存器
	movl	-12(%rbp), %eax
	movl	%edx, %esi				# x86-64的特色，用寄存器而不是栈传递参数给子过程。第二个参数给`%esi`
	movl	%eax, %edi				# 第一个参数给`%edi`
	call	add						# 子过程调用
	movl	%eax, -4(%rbp)
	movl	-4(%rbp), %ecx
	movl	-8(%rbp), %edx
	movl	-12(%rbp), %eax
	movl	%eax, %esi
	leaq	.LC0(%rip), %rax
	movq	%rax, %rdi
	movl	$0, %eax
	call	printf@PLT
	movl	$0, %eax
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size	main, .-main
	.ident	"GCC: (Ubuntu 11.4.0-1ubuntu1~22.04.2) 11.4.0"
	.section	.note.GNU-stack,"",@progbits
	.section	.note.gnu.property,"a"
	.align 8
	.long	1f - 0f
	.long	4f - 1f
	.long	5
0:
	.string	"GNU"
1:
	.align 8
	.long	0xc0000002
	.long	3f - 2f
2:
	.long	0x3
3:
	.align 8
4:
```



---
### Caller 和 Callee 保存寄存器
"Callee-saved"（被调用者保存）和 "Caller-saved"（调用者保存）是关于函数调用时，为保证程序状态（特别是寄存器中的数据）的完整性而建立的两种**寄存器管理规则**。

这些规则是**调用约定 (Calling Convention)** 的核心部分，它定义了函数之间如何安全地传递控制权和数据。

#### 1. Caller-Saved（调用者保存）寄存器

定义：
这些寄存器由调用函数 (Caller) 负责保存。当函数 A 调用函数 B 时，如果函数 A 在调用 B 之后还需要使用这些寄存器中的值，那么 A 必须在调用 B 之前将这些寄存器的**当前值保存到栈上（通过 push 指令），并在 B 返回后将它们恢复（通过 pop 指令）。**

函数的责任（Callee）：

被调用的函数（Callee）**可以自由地使用和修改这些寄存器，无需担心会破坏调用者的数据。**

目的：

允许被调用的函数自由地使用这些寄存器作为临时工作空间，而无需担心保存和恢复的开销。如果调用者不需要在调用后保留这些值，则可以节省栈操作。

**典型的 Caller-Saved 寄存器（x86/x86-64）：**

- **IA-32:** `%eax`, `%ecx`, `%edx`
	- 回忆一下：入口参数位置，正是 `Caller` 压栈的这三个 `Caller-Saved` 三个寄存器
- **x86-64 (System V ABI):** `%rax`, `%rcx`, `%rdx`, `%rsi`, `%rdi`, `%r8` 到 `%r11` (用于参数传递和返回值)
---

#### 2. Callee-Saved（被调用者保存）寄存器

定义：

这些寄存器由被调用函数 (Callee) 负责保存。如果被调用的函数要修改这些寄存器中的值，它必须在函数入口处（序言部分）将它们保存到栈上，并在函数返回前（尾声部分）将它们恢复。

函数的责任（Callee）：

被调用的函数在修改前必须保存这些寄存器，确保当控制权返回给调用者时，这些寄存器的值与调用前保持一致。

目的：

主要用于保存长期变量或帧指针。这样，调用者可以确信这些寄存器中的值在函数调用前后保持不变，从而减少调用者必须进行的栈操作。

**典型的 Callee-Saved 寄存器（x86/x86-64）：**

- **IA-32:** `%ebx`, `%esi`, `%edi`, `%ebp`, `%esp`
	- 这些用于函数内修改。
- **x86-64 (System V ABI):** `%rbx`, `%rbp`, `%rsp`, `%r12` 到 `%r15`
    

---

#### 区别总结对比

|**特性**|**Caller-Saved（调用者保存）**|**Callee-Saved（被调用者保存）**|
|---|---|---|
|**谁负责保存？**|**调用函数 (Caller)**|**被调用函数 (Callee)**|
|**何时保存/恢复？**|Caller 在调用前/后。|Callee 在函数入口/返回前。|
|**为什么存在？**|供 Callee 作为临时工作空间使用。|供 Caller 保存跨函数调用的稳定数据。|
|**保存的条件？**|仅当 Caller 在函数调用后还需要这些值时。|仅当 Callee 计划在函数内部修改它们时。|

#### 举例说明

假设函数 $A$ 调用函数 $B$：

1. **Caller-Saved 寄存器（如 `%eax`）：**
    
    - 如果 $A$ 需要保留 `%eax` 的值：$A$ 执行 `push %eax` -> 调用 $B$ -> $B$ 随意修改 `%eax` -> $B$ 返回 -> $A$ 执行 `pop %eax`。
        
    - 如果 $A$ 不需要保留 `%eax` 的值：$A$ 直接调用 $B$。$B$ 随意修改 `%eax`。**更快。**
        
2. **Callee-Saved 寄存器（如 `%ebx`）：**
    
    - 如果 $B$ 需要使用 `%ebx`：$B$ 在入口处执行 `push %ebx` -> $B$ 使用 `%ebx` -> $B$ 在返回前执行 `pop %ebx`。
        
    - 如果 $B$ 不需要使用 `%ebx`：$B$ 不进行任何操作。**更快。**
        

通过这种机制，编译器和程序员可以根据具体情况选择最有效率的方法来管理寄存器状态，从而优化程序的执行速度。


### 选择语句的机器级表示

#### `switch`

`switch` 会维护一个跳转表。
**每个跳转表的表项中存放一个某分支对应的标号**（4 字节地址，`.long` 表示）。

通过各跳转值中**最小的一个**当偏移量 （归一化从 `0` 开始 index）, 然后还要规定每个表子项的大小即 `align`, 地址偏移量为 `index * align`
`jmp *.L8 ( , %eax, 4)` 就是一个间接跳转到各个表项的语句（间隔为 `4` 字节。）

`switch` 语句通过**跳转表（Jump Table）**机制来根据不同的值决定跳转目标，这是一个比使用一系列 `if/else if` 语句效率高得多的方法。

当编译器将 C/C++ 等高级语言中的 `switch` 语句翻译成汇编时，如果 `case` 值是密集且范围较小的整数，它通常会生成这个跳转表结构。


跳转表机制的实现依赖于三个核心步骤：

##### 步骤 1: 规范化索引 (Normalization)

首先，程序需要将 `switch` 语句的控制变量 $V$ 转换为一个从 $0$ 开始的、可以安全地用作数组索引的**索引 $N$**。

**代码示例：** `movl %edi, %eax`

1. **加载变量 $V$：** 将 `switch` 变量的值加载到一个寄存器中（例如 `%edi`）。
    
2. 处理最小值 (Min Case)： 如果 case 值是从非零数字（例如 case 5:）开始的，编译器会先执行一次减法，将 $V$ 减去最小的 case 值，得到索引 $N$。
    
    $$N = V - \text{MinCase}$$
    
    （在您提供的例子中，movl %edi, %eax 假设 $\text{MinCase}=0$ 或这个检查在前面已经完成。）
    

##### 步骤 2: 边界检查 (Range Check)

为了防止变量 $V$ 超出 `case` 值的有效范围，程序必须执行边界检查。如果 $V$ 太大，程序将跳转到 `default` 分支（如果没有 `default`，则跳过 `switch`）。

- **检查：** $\text{If } N > \text{MaxIndex } \text{ then jump to Default}$
    
- **汇编指令：** 通常使用 `cmp` 和无符号跳转指令 `ja` (Jump if Above)。
    

##### 步骤 3: 间接跳转 (Indirect Jump)

这是核心步骤，使用索引 $N$ 来查找并执行跳转。

**代码示例：** `jmp *.L7(,%eax,8)`

1. 计算目标地址： CPU 根据跳转表的起始地址（.L7）、索引 $N$（在 %eax 中）和每个地址条目的大小（$S$ 字节，在 x86-64 中 $S=8$），计算出要跳转的地址所在的内存位置：
    
    $$\text{Address of Target} = \text{Start of .L7} + (N \times S)$$
    
2. **读取地址并跳转：** CPU 从内存的 $\text{Address of Target}$ 处读取存储的地址（例如 `AddrN`），然后立即跳转到该地址。
    

跳转表 `.L7` 本身就是一个**地址数组**，每个地址对应一个 `case` 分支的代码位置。

|**数组索引 N**|**内存内容 (地址)**|**对应的 switch 代码块**|
|---|---|---|
|0|`Addr0`|`case MinCase:`|
|1|`Addr1`|`case MinCase + 1:`|
|...|...|...|
|$N$|`AddrN`|`case MinCase + N:`|

#### 总结

`switch` 语句就是通过将**控制变量值**转换为一个**数组索引**，然后利用该索引**直接查表**获取目标地址，最终通过**一次间接跳转**实现分支。这种机制将 $O(N)$ 的线性查找时间（`if/else if` 链）变成了 $O(1)$ 的常数时间查找，效率极高。

- if your case is `0` to `10000000`, it actually will build a **huge** table.
- But if you don't want waste such a huge space, you may use `if-else` .  A well-designed `if-else` tree can reduce the complexity from `O(n)` to `O(log n)`


## 联合体 `union` 的内存表示
在 C 语言中，`union` 的定义决定了所有成员是**从同一内存地址开始**存储的。它们共享同一块内存空间。


```c
typedef union{
	struct { // s1
		int x;
		short y;
		short z;
	}	s1;
	struct{ // s2
		short a[2];
		int b;
		char *p;
	}	s2;
} utype;
```

结构体的总大小等于最大成员的大小，**（加上 padding 以保证对齐）**



### 对齐：拿结构体举例

- 习题：[[ICS-3#21 分别给出在 `IA-32+Linux` , `x86-64+linux` 平台上，下列各个结构体类型中每个成员的偏移量、结构体总大小以及结构体起始位置的对齐要求。|对齐习题]]

 - Linux 平台相对宽松
	 - `IA-32` 最多对齐 `4` 字节，按数据类型大小确定对齐规则。`char` 不需要对齐（可以补空）
	- `x86-64` 最多对齐 `8` 字节。




## 内存：x86-64 Linux Memory Layout

### Memory Layout
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108133932.png)

- **Stack**
	- Runtime stack (**8MB** limit)
		- try type `limit` in your `Linux` machine
	- E.g: Local variables
- **Heap**
	- Dynamically allocated as needed
	- When call `malloc()`, `calloc()`, `new()`
		- varies **dynamically**
- **Data**
	- Statically allocated data
	- E.g., global vars, `static` vars, **string constants**
- Text / **Shared Libraries**
	- Executable machine instructions
	- Read-only
	- **Dynamic Linking**


- Memory Allocation **Example**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108134100.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108134111.png)

### Buffer Overflow 

- Generally called a "buffer overflow"
	- When exceeding the memory size allocated for an array
- Most common form
	- Unchecked lengths on string inputs
	- Particularly for bounded characters arrays on the stack.
		- sometimes referred to as stack smashing



### Code Injection Attacks
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108135619.png)
- Your code may include `gets(buf)` like that, and as a **attacker**, you can offer a **long** string ,it actually your **injection** code. 
- You may need to **add** some padding whose value doesn't **matter**, In order to then get a number back into the **position** where the return pointer is **suppose** to be.
- Then, you can rewrite the return pointer, to return to the function you inject. for that, you need to check your `touch` function address.


> [!Note] Attack Lab
> Interested in this? Try CMU Attack Lab then.
> [Zhihu helper](https://zhuanlan.zhihu.com/p/476396465)
