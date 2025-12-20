## Section 5 程序的链接和加载执行

本文结合了 CMU CSAPP 课程和 NJU ICS 的内容
由于 CSAPP 英文讲授~~且英文打起来更方便~~, 多数用英文书写。
### ELF

Executable and Linkable Format
关于 ELF 表可以阅读 [[ICS-PA2 note#增加对 `elf文件的支持`|ELF C语言理解]]



### **链接的本质**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027204556.png)


合并不同的 `Section`，形成一个大的表。

### 可执行文件的内存映像

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027204711.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251027204741.png)


### 符号和符号解析

每一个 **可重定位目标模块**m 都有一个符号表，定义了在 `m` 中定义的符号。有三种链接器符号。

- **Global symbols**（全局符号）
	- 由模块 `m` 定义并能被其他模块引用的模块。
	- 非 Static 的函数名和非 static 的全局变量名。
- **External symbols**(外部符号)
	- 由其他模块定义并被模块 m 引用的全局符号
		- 如 `main.c`，函数名 `swap`
- **Local symbols**(本地符号)
	- 由模块 `m` 定义和引用的带 `static` 的函数名和变量名。因其生存期为整个程序运行过程，故并不分配在栈中，而是分配在 `static data` 区（静态数据区），即在 `.data` 或 `.bss` 节中分配空间。
		- 如 `swap.c` 中的 `static` 变量名 `bufp1`
		- `.data` 中的有初值，`.bss` 一般默认为 0
### 目标文件 `ELF` 中的符号表

可以参考阅读[[ICS-PA2 note#阅读符号表，对照字符表|如何阅读符号表]]



### What Do Linkers Do

#### Step 1: Symbol resolution
- Programs define and reference *symbols* (global variables and functions)
	- `void swap(){...}` `/* define symbol swap */` 
	- `swap();` `/* reference symbol swap */`
	- `int *xp = &x;` `/* define symbol xp, reference x */`
- **Symbol definitions** are stored in object file (by assembler) in *symbol table*
	- Symbol table is an array of `struct`
	- Each entry includes name, size, and location of symbol.


#### Step 2: Relocation （重定位）
- Merges separate code and data sections into single sections
- Relocates symbols from their relative locations in the `.o` files to their final absolute memory locations in the *executable*
- Update all references to these symbols to reflect their new positions
---

#### Three Kinds of Object Files (Modules)
- **Relocatable** object file (`.o` file )
	- Contains code and data in a form that can be combined with other relocatable object files to form executable object file
	- each `.o` file is produced from exactly one source (`.c`) file
- **Executable** object file (`a.out` file)
	-  Contains code and data in a from that can be copied directly into memory and then executed
- Share object file (`.so` file)
	- Special type of relocatable object file that can be loaded into memory and linked dynamically, at eithe**r load time or run-time**
	- Called ***Dynamic Link Libraries*** by Windows


#### Executable and Linkable Format (ELF)
- Standard binary format for object files
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108143659.png)

- Elf header
	- Word size, byte ordering, file type, machine type, etc.
- Segment header table
	- Page size, virtual addresses memory segments (sections), segment sizes
- `.text` section
	- **Code**
	- function store in here too.
- `.rodata` section
	- Read only data: jump tables(in `switch`), 
	- some const **number**
- `.data` section
	- **Initialized global variables**
- `.bss` section
	- **Uninitialized global variables** or **static variables**
	- back to 60s. IBM: *block started by symbol*
	- "Better Save Space" (BSS)
		- because **uninitialized** global variables no need to be in `.o`, it save occupied space.

- `.symtab` section
	- Symbol table
	- *Procedure* and static *variable* names
	- Section *names* and ***locations***
- `.rel` `.text` section
	- *Relocation* info for `.text` section
	- *Addresses* of instructions that will need to be modified in the executable
- `.rel` `.data` section
	- *Relocation* info for `.data` section
	- *Addresses* of pointer data that will need to be modified in the merged executable
- `.debug` section
	- Info for symbolic *debugging* (`gcc -g`)
	- provide **Information** that relates line numbers to in the source code to line **numbers** in the machine code.
	- that why we can use `gdb`
- `Section header table`
	- `Offset` and `Size` of each section

#### Linker Symbols
- **Global symbols**
	- Symbols defined by module `m` that can be referenced by other modules.
	- non-**static** C functions and on-**static** global variables
- **External symbols**
	- Global symbols that are **referenced** by module `m` but defined by some other module
- **Local symbols**
	- Symbols that are defined and referenced exclusively by module `m`
	- E.g.: C **functions** and **global variables** defined with the **`static`** attribute
	- **Local linker symbols are not local program variables**


##### Local Symbols
- **Local non-static C variables vs. local static C variables**
	- local non-static C variables: stored on the **stack**
	- local static C variables: store in either `.bss` or `.data`
```c
	// Compiler allocates space in .data for each definition of x
	// Create local symbols in the symbol table with unique names, e.g. x.1 and x.2
int f()
{
	static int x = 0;
	return x;
}

int g()
{
	static int x = 1;
	return x;	
}
```

##### How Linker Resolves Duplicate Symbol Definitions?
- Program symbols are **either *strong* or *weak***
	- ***Strong***: procedures and initialized globals
	- ***Weak***: uninitialized globals
	- 新标准：
	- ***Common***: unallocated uninitialized globals
		- Linker will tell how to link this common symbol later.
	- **Weak***: GCC 拓展的属性指示符 `__attribute__(week)`
		- 这个 `weak` 会被更强的所链接。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108145559.png)


- **Linker's Symbol Rules**
	- **Rule** 1 **Multiple strong symbols are not allowed.**
		- Each item can be defined only once
		- Otherwise: linker's error
	- **Rule 2: Given a strong symbol and multiple weak symbols, choose the strong symbol**
		- References to the weak symbol *resolve* to the **strong** symbol
	- **Rule 3: If there are multiple weak symbols, pick an arbitrary one**
		- Can override this with `gcc -fno-common`
			- Or `-Werror`
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108150157.png)

#### Relocated `.text` section
**什么是重定位**：
- 重定位的目的：在符号解析的基础上，将所有互相关联的目标模块合并，**并确定运行时每个定义符号**在虚拟地址空间中的**地址** 。在定义符号的引用处，**重定位**引用的地址。
- `临时地址` 到 `有效的绝对地址` 的修补。![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108151937.png)
- **重定位类型**
	- `R_386_PC32`：引用处采用 **PC 相对**寻址方式
		- 如上图的例子
	- `R_386_32`：引用处使用**绝对地址**方式



#### Using Static Libaries
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108152744.png)
- Linker 对于外部引用 (external references)
	- Scan `.o` files and `.a` files in the command line order
	- During scan, keep a list of the current unresolved references
	- As each new `.o` or `.a` (static libraries) , is encountered, try to resolve each unresolved reference in the list against the symbols defined in `obj.`
	- If any entries in the unresolved list at end of scan, then **error**
- **Command line order matters!**
	- affect how you scan



> [!Tip] How to do **risky** link and compile?
> To study link and compile, you may have to do risky **link**. However, gcc may prevent you from doing so by error your linkage. You can type following command to avoid it.
> ```bash
> gcc <your .c> -fcommon
> ```


