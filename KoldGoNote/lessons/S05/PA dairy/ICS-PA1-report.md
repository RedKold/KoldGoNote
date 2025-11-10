## 实验进度

我完成了 PA1 的**所有内容**，通过了**所有测试样例**，并为了调试方便，加入了逻辑与或非等运算符。

## 一些问题和思考
### 为什么 `printf()` 的输出要换行
- 事实上我经常忘记加换行，但我发现这样就会出现
- `<some word you print balabala> (nemu) `，这会使得终端页面异常丑陋，同时输出可能堆叠在一堆里，**不好看。**
- 同时，`printf()` 写到标准输出 `stdout` 并不是立刻刷新，而 `\n` 起到一个刷新的作用，使用**换行符**能及时显示，避免程序异常退出导致缓存区 log 信息没有输出。

### 再探表达式解析 - 分治法和栈
我在大一的 `程序设计基础实验` 曾经完成过一个基于 `Qt` 的图形化计算器，可以计算包含括号的、有小数和负数的四则运算计算器。
事实上，当时我对于 `-` 的多义性（`二目运算符 SUB`? `单目运算符 NEG` ?）的处理是采用了复杂的分类讨论。由于我用**栈**+**后缀表达式**实现算术式解析([我曾经参考的文章](https://blog.csdn.net/sjl20021006/article/details/117400261)) ，我的算式解析过程是 `流式` 的，即从左到右依次读取数字和操作符并压栈，对于多数加减乘除的 `二目操作符`，我使用这使得全局视角缺失，判断操作符 `token` 的过程异常繁琐。

```cpp
	// some source code...
	//栈外优先级高于OR等于栈内，直接入栈（包括括号）
	if (expr[i] == '(')Oprators.push(expr[i]);
	else if (Priority(expr[i]) >= Priority(Oprators.top())) {
		Oprators.push(expr[i]);
	}
	else if (expr[i] == ')') {
		while (Oprators.top() != '(') {
			float a = Numbers.top(); Numbers.pop();
			float b = Numbers.top(); Numbers.pop();
			char opr = Oprators.top(); Oprators.pop();
			//cout << "now the opr is: " << opr << endl;
			float c = SubCalc(a, b, opr);
			Numbers.push(c);
		}
		Oprators.pop();
	}
	//栈外优先级低于栈内，取出两个操作数完成操作，得到结果入栈。
	else {
		float a = Numbers.top(); Numbers.pop();
		float b = Numbers.top(); Numbers.pop();
		char opr = Oprators.top(); Oprators.pop();
		//cout << "now the opr is: " << opr << endl;
		float c = SubCalc(a, b, opr);
		Numbers.push(c);
		Oprators.push(expr[i]);
	}
```

该 Qt 计算器的源码可查看[RedKold/kold_calculator: first homework project](https://github.com/RedKold/kold_calculator)

而这次，我学习到另一个处理计算式解析的流派，基本思想是 `分而治之`。其给我的启发很大。我清楚的发现，代码变得更简洁了，更容易思考了。

同时震撼我的是正则表达式。其实无论在实习大语言模型预处理知识库的时候，还是大一预处理单词本来做“百词斩”小玩具的时候，我都曾经想尝试过 regex（~~终究败给懒惰~~），这次用了之后，一行解决复杂的 token 提取，且在 vim 中结合的很好，给我留下很深的印象。

**缺省的技不如人，终会找上门，不如早点拥抱新技术**

### About `static` keyword
> [!Note] **温故而知新**
> 框架代码中定义 `wp_pool` 等变量的时候使用了关键字 `static`, `static` 在此处的含义是什么? 为什么要在此处使用它?

先看看源码
```c
static WP wp_pool[NR_WP] = {};
// head is WP in using, free_ is WP that is free
static WP *head = NULL, *free_ = NULL;
```

`static` 关键词一般有两个意思：
- 如果在函数内部，**表达静态存储**。即函数调用后不销毁，值没有存在**栈**而是 **堆** 中
- 如果在文件作用域，**限制**该变量或函数的可见性仅限于当前源文件。

我觉得这是一种 **抽象设计**。使用 `static` 限制之后，外部只能通过 `src/sdb/watchpoint.c` 调用监视点管理函数。

> [!Tip]
> 事实上，我亲身体会了这种设计的好处。我写的 `wp_pool` 相关函数没有用 static 声明，我试图在别的文件中调用函数而不行。这是一种 `barrier`, 限制了我做出更多未测试行为... (~~and bomb my PA~~)



## **必答题**

原必答题链接 [PA1 必答题](https://nju-projectn.github.io/ics-pa-gitbook/ics2024/1.7.html)

- **Solution**
```
// PC: instruction    | // label: statement
0: mov  r1, 0         |  pc0: r1 = 0;
1: mov  r2, 0         |  pc1: r2 = 0;
2: addi r2, r2, 1     |  pc2: r2 = r2 + 1;
3: add  r1, r1, r2    |  pc3: r1 = r1 + r2;
4: blt  r2, 100, 2    |  pc4: if (r2 < 100) goto pc2;   // branch if less than
5: jmp 5              |  pc5: goto pc5;
```

初始状态为 `(0,x,x)`，`x` 表示未初始化，`PC=0` 处的指令是 `mov r1,0`, 执行完下一个状态是 `1,0,x`, 之后程序执行 `move`

```
(0, x, x) -> (1, 0, x) -> (2, 0, 0) -> (3, 0, 1) -> (4, 1, 1)->
(2, 1, 1) -> (3, 1, 2) -> (4, 3, 2) -> (2, 3, 2) -> (loop...)->
(2, 4851, 98) -> (3, 4851, 99) -> (4,4950, 99) -> 
(2, 4950, 99) -> (3, 4950, 100) -> (4,5050, 100) -> (5, 5050, 100) -> end;
```

---
- **Solution**
	- `500` 次编译，其中 `450` 次调试，假设我为了解决 `450` 个 bug，则每次通过 `gdb` 调试需要获得 `9000` 个信息，每个信息需要 `30s`，则调试花费了我 `30s*9000=4500min=75h`
		- ~~这足以通关 3次空洞骑士~~
	- 假如简易调试器只需要 `10s`，则整体时间减少了 `2/3`，节省了 `50h`

---
 - riscv32
	- riscv32有哪几种指令格式?
	- LUI指令的行为是什么?
	- mstatus寄存器的结构是怎么样的?
- **Solution**
	- 我选择 riscv32
		- riscv32 有哪几种指令格式可以参阅**The RISC-V Instruction Set Manual [Volume I](https://github.com/riscv/riscv-isa-manual/releases/download/riscv-isa-release-382fd8b-2024-04-11/unpriv-isa-asciidoc.pdf)** 的 **2.2 Base Instruction Formats**, 其位于 Page 23.
			- **四种指令格式**
				- `R-Type`
				- `I-Type`
				- `S-Type`
				- `U-Type`
		- LUI 指令的行为可以参阅 **The RISC-V Instruction Set Manual Volume I** 的 **2.4 Integer Computational Instructions**， Page 27
			- **LUI (load upper immediate)**, uses the U-type format. LUI places the 32-bit U-immediate value into the destination register `rd`
		- mstatus 寄存器的结构是怎么样的可以参阅**The RISC-V Instruction Set Manual** [Volume II]([priv-isa-asciidoc.pdf](file:///E:/BaiduSyncdisk/Semesters/S05/ICS/priv-isa-asciidoc.pdf)) 的 3.1.6, Machine Status Registers (mstatus and mstatush) 章节
			- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20250914150748.png)
---

- shell **命令统计代码行数**
	- `find . -name "*.c" -or -name "*.h" | xargs wc -l` 命令，其中 find 可以递归返回当前目录所有 `.c .h` 文件，将他们通过管道，再用 xargs 转化为命令行参数交给 `wc` 来计算行数 `-l` 
		- `wc - print newline, word, and byte count for each file`
	- 在 `pa1`，总共有 `267920` 行代码
	- 可以使用 git 切换到 pa0 分支来“回到过去”，
		- `git checkout pa0` 
		- 再次运行 `find . -name "*.c" -or -name "*.h" | xargs wc -l`, get `266813`
	- 我总共编辑了 `267920-266813=1107 lines` 代码
	- 如果不包括空行：`grep -r . --include="*.c" --include="*.h" | wc -l`
		- **原理**：`.` 在正则表达式表达匹配任意字符，但是不包括换行，这就过滤掉了空行
		- result: `231480`
---
- `CFLAGS  := -O2 -MMD -Wall -Werror $(INCLUDES) $(CFLAGS)`
	- 这是 `scripts/build.mk` 中的编译选项
	- 具体可以阅读 [Warning Options (Using the GNU Compiler Collection (GCC))](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
	- `-Wall`： Warning all (并非)
		- `-Wall` 引入了常用了一组警告 set
以下引用[官方文档]([Warning Options (Using the GNU Compiler Collection (GCC))(https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)，关于 `-Wall` 引如的警告参数可以详见。
> This enables all the warnings about constructions that some users consider questionable, and that are **easy to avoid** (or modify to prevent the warning), even in conjunction with macros.
- `-Werror`:  This flag promotes all warnings into errors
- **总结**
	- 使用 `-Wall` 有助于在编译的时候发现一些警告信息，**避免一些不好的编程习惯**导致潜在的 bug
	- 使用 `-Werror` 将逼迫程序员完成一个“干净”的项目，否则存在警告就将无法编译运行。这有助于避免一些潜在的 bug，养成良好的编程习惯。
- 总之，二者的使用可以降低维护成本，减少项目 bug 积累，降低“出错”的可能。
---

**This ends PA1**

**完成 PA1 的感觉相当好**：它是一段新旅程的开始，在这个新手村我熟悉了 linux 的基本操作，学习用以前从未体验过的方式编程和开发 (Features: `ssh+aliyun, vim, tmux, xfce+xrdp...`)。之后的日子，想必充满挑战也充满乐趣。

感谢 lj 老师的细心讲解和助教团队的帮助。

> **キボウの道を ジグザグ進もう**

