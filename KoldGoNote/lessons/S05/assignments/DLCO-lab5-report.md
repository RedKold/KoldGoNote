
# 实验 5 存储器及数据通路设计 
## 1. 只读存储器实验
### 实验内容：
利用 ASCII 码可显示字符点阵字库，在 Logisim 的 LED 点阵组件上显示 ASCII 码字符形状。
### 基本原理
LED 组件每一排可以读取一个数，绘出图样
我们的 LED ROM 中存储 ASCII 字库，在对应 row 绘出图形，就可以显示字。
### 设计电路
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215221109.png)


要注意一些细节：
- 首行是标识，不存储
- 有效地址要减去 `0x20`，
- 每个点阵数据 16 字节，所以字库的起始位置需要作差左移 4 位。

### 仿真测试
- `0` (48)
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215221240.png)

-  `p`
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215221259.png)


## 2. 数据存储器实验
### 基本原理
Logisim 中的 RAM 组件的数据接口 Data Interface 中有三种不同的工作模式，若设置为分离加载和存储端口模式 Separate load and store ports，则分别显示输入数据端口和输出数据端口；其它两种模式则只使用同一个数据端口进行同步或异步读写操作，区别在于是否有时钟输入端口。

我们按照 `byte` `halfword` `word` 来进行数据读写，用 `MemOp` 来表示当前指令读取数据的字节长度。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215221414.png)


注意，**由于对齐要求**，所以事实上读取 3 字节的请求是不存在的。
### 设计电路
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215221459.png)

实现了功能，通过了 OJ 读写要求 

## 3 取指令部件实验
### 基本原理
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215221947.png)
单周期 CPU 设计如上。这个实验实现的是取指令部分，即根据 PC 实现 IM

指令地址的计算有多种情形：
1. 顺序执行指令，则 `PC=PC+4`。
2. 无条件跳转指令，`jal` 指令，`PC = PC + 立即数 imm`；`jalr` 指令，`PC = R[rs1] + 立即数 imm`。
3. 分支转移指令，根据比较运算的结果和 Zero 标志位来判断，如果条件成立则 `PC = PC +立即数 imm`，否则 `PC=PC+4`


### 基本电路
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215222140.png)
根据此表，构建电路。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215222910.png)

### 测试数据

实际和 OJ 表格已知，这里摘录如下：

| 序号  | 输入    |                 |            |            |         | 输出（边沿信号有效前） |            |
| --- | ----- | --------------- | ---------- | ---------- | ------- | ----------- | ---------- |
|     | Reset | Initial Address | Imm        | BusA       | NPCASrc | NPCBSrc     | PC         |
| 1   | 1     | 0x0400          | 0x00000000 | 0x00000000 | 0       | 0           | 0x00000000 |
| 2   | 0     | 0x0000          | 0x00000000 | 0x00000000 | 0       | 0           | 0x00000400 |
| 3   | 0     | 0x0000          | 0x00000000 | 0x00000000 | 0       | 0           | 0x00000404 |
| 4   | 0     | 0x0000          | 0x000007f8 | 0x00000000 | 0       | 1           | 0x00000408 |
| 5   | 0     | 0x0000          | 0x0000020c | 0x00000000 | 0       | 1           | 0x00000c00 |
| 6   | 1     | 0x1500          | 0x00000000 | 0x00000000 | 0       | 0           | 0x00000e0c |
| 7   | 0     | 0x0000          | 0x00000000 | 0x00000000 | 0       | 0           | 0x00001500 |
| 8   | 0     | 0x0000          | 0x00000000 | 0x00000000 | 0       | 0           | 0x00001504 |
| 9   | 0     | 0x0000          | 0x00000000 | 0x0003fbc0 | 1       | 1           | 0x00001508 |
| 10  | 0     | 0x0000          | 0x00000000 | 0x00000000 | 0       | 0           | 0x0003fbc0 |
| 11  | 0     | 0x0000          | 0x000001fc | 0x00000000 | 0       | 1           | 0x0003fbc4 |
| 12  | 0     | 0x0000          | 0x00000000 | 0x0000150c | 1       | 1           | 0x0003fdc0 |
| 13  | 0     | 0x0000          | 0x00000c00 | 0x00000000 | 1       | 1           | 0x0000150c |
| 14  | 0     | 0x0000          | 0x00000000 | 0x00000000 | 0       | 0           | 0x00000c00 |
| 15  | 0     | 0x0000          | 0x00000000 | 0x00000000 | 0       | 0           | 0x00000c04 |
| 16  | 0     | 0x0000          | 0xfffff9c8 | 0x00000000 | 0       | 1           | 0x00000c08 |

## 4 取操作数部件 IDU 实验

### 基本原理
取操作数的过程，对于不同类型指令大致相同。RV32I 是 32 位定长指令，所以指令译码可以用 splitter 方便的进行。


据图 5.7 所示，ALUAsrc 和 ALUBsrc 用来控制两个多路选择器的输出数据， ALUASsrc 控制 1 个两路选择器，当 ALUASsrc=0 时，选择 BusA 输出到 ALU 的操作数 A 口，当 ALUASsrc=1 时输出 PC。ALUBSrc 控制 1 个四路选择器，当 ALUBSsrc=00 时选择 BusB 输出到 ALU 的操作数 B 口，当 ALUBSsrc=01 时选择输出常数 4，当 ALUBSsrc=10 时选择输出 32 位立即数 Imm。

此外，我们还要进行正确的立即数拓展

```
immI = {20{Instr[31]}, Instr[31:20]};
immU = {Instr[31:12], 12'b0};
immS = {20{Instr[31]}, Instr[31:25], Instr[11:7]};
immB = {19{Instr[31]},Instr[31], Instr[7], Instr[30:25], Instr[11:8], 1'b0};
immJ = {11{Instr[31]}, Instr[31], Instr[19:12], Instr[20], Instr[30:21], 1'b0};
```

### 电路设计
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215223615.png)


- **立即数拓展**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215223633.png)

### 测试数据表

|   |   |   |   |   |   |   |
|---|---|---|---|---|---|---|
|PC|IR|BusW|控制信号|指令功能|汇编语句|DataA、DataB输出值|
|0|fedca2b7|fedca000|ExtOp=1 RegWr=1 ALUASrc=0 ALUBSrc=2|LUI指令，将立即数加载到寄存器高位|lui a0, 0xfedca|DataA=00000000, DataB=fedca000|
|4|f9c28293|fedc9f9c|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=2|ADDI指令，寄存器加立即数并写入目标寄存器|addi a1, a0, 0xfffffff9c|DataA=fedca000, DataB=ffffff9c|
|8|01006013|00000010|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=2|ADDI指令，寄存器加立即数并写入目标寄存器|addi zero, zero, 0x10|DataA=00000000, DataB=00000010|
|c|06502223|00000064|ExtOp=2 RegWr=0 ALUASrc=0 ALUBSrc=2|SW指令，将寄存器数据存储到内存|sw zero, 0x64(a1)|DataA=00000000, DataB=00000064|
|10|06400303|ffffff9c|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=2|LW指令，从内存加载数据到寄存器|lw a2, 0x64(zero)|DataA=00000000, DataB=00000064|
|14|06601423|00000068|ExtOp=2 RegWr=0 ALUASrc=0 ALUBSrc=2|SW指令，将寄存器数据存储到内存|sw zero, 0x68(a2)|DataA=00000000, DataB=00000068|
|18|06405383|00009f9c|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=2|LW指令，从内存加载数据到寄存器|lw a3, 0x64(zero)|DataA=00000000, DataB=00000064|
|1c|06701623|0000006c|ExtOp=2 RegWr=0 ALUASrc=0 ALUBSrc=2|SW指令，将寄存器数据存储到内存|sw zero, 0x6c(a3)|DataA=00000000, DataB=0000006c|
|20|4042d413|ffedc9f9|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=2|ADDIW指令，寄存器加立即数（32位）并写入目标寄存器|addiw a4, a1, 0x404|DataA=fedc9f9c, DataB=00000404|
|24|006444b3|00123665|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=0|ADD指令，两个寄存器相加并写入目标寄存器|add a5, a4, a2|DataA=ffedc9f9, DataB=ffffff9c|
|28|00649533|50000000|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=0|SLL指令，寄存器逻辑左移并写入目标寄存器|sll a6, a5, a2|DataA=00123665, DataB=ffffff9c|
|2c|008505b3|4fedc9f9|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=0|XOR指令，两个寄存器异或并写入目标寄存器|xor a7, a6, a4|DataA=50000000, DataB=ffedc9f9|
|30|00b2a633|00000001|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=0|SLT指令，比较两个寄存器大小（小于则置1）并写入目标寄存器|slt s0, a1, a7|DataA=fedc9f9c, DataB=4fedc9f9|
|34|00b2b6b3|00000000|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=0|SLTU指令，无符号比较两个寄存器大小（小于则置1）并写入目标寄存器|sltu s1, a1, a7|DataA=fedc9f9c, DataB=4fedc9f9|
|38|40b287b3|aeeed5a3|ExtOp=0 RegWr=1 ALUASrc=0 ALUBSrc=0|ADDW指令，两个寄存器32位相加并写入目标寄存器|addw s2, a1, a7|DataA=fedc9f9c, DataB=4fedc9f9|
|3c|06f02823|00000070|ExtOp=2 RegWr=0 ALUASrc=0 ALUBSrc=2|SW指令，将寄存器数据存储到内存|sw zero, 0x70(s2)|DataA=00000000, DataB=00000070|
|40|0067c263|00000001|ExtOp=3 RegWr=0 ALUASrc=0 ALUBSrc=0|BLTU指令，无符号小于则分支跳转|bltu s2, a2, 0x4|DataA=aeeed5a3, DataB=ffffff9c|
|44|0067d263|00000001|ExtOp=3 RegWr=0 ALUASrc=0 ALUBSrc=0|BGEU指令，无符号大于等于则分支跳转|bgeu s2, a2, 0x4|DataA=aeeed5a3, DataB=ffffff9c|
|48|0040086f|0000004c|ExtOp=4 RegWr=1 ALUASrc=1 ALUBSrc=1|JAL指令，跳转并链接（保存返回地址）|jal s3, 0x4|DataA=00000048, DataB=00000004|
|4c|004808e7|00000050|ExtOp=0 RegWr=1 ALUASrc=1 ALUBSrc=1|JALR指令，寄存器间接跳转并链接|jalr s4, s3, 0x4|DataA=0000004c, DataB=00000004|
|50|00001917|00001050|ExtOp=1 RegWr=1 ALUASrc=1 ALUBSrc=2|AUIPC指令，将程序计数器加立即数并写入寄存器|auipc s5, 0x1000|DataA=00000050, DataB=00001000|
## 5 数据通路实验

### 基本原理
**数据通路**是完成数据存取、运算的部件。
Different instruction type has different data transport process:
- IFU
- IDU
- EX (Execute)
- M (Access Memory)
- WB (Write Back)

在之前的实现，IFU IDU 和 EX (ALU, in lab 4.5) 和 M 都已经做完了。**这里我们来实现跳转控制部件**。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215224520.png)

然后，我们把数据通路的各部分串联起来即可


### 电路设计
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215224627.png)

## 思考题
### 如何利用 ROM 实验实现滚动显示的功能，在 3 个 LED 点阵矩阵中，**左右滚动**显示 5 个 ASCII 字符，如“NJUCS”
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215225253.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215225307.png)

制作一个包含 `NJUCS` 的 image, 然后 load 进 `lab5` 中的 rom
之后，LED matrix 横向排列，添加一个自增的计数器，以 `0x10` 这一点阵一行的**间距**作为步长，依次显示。
这样我们就能获得“移动”的效果
### 分析说明如果 PC 寄存器、寄存器堆和数据存储器写入数据的时钟信号触发边沿不一致，则对程序执行结果有什么影响？

因为我们是按 PC 取指令。如果 PC 的触发边沿和别的不一致是很麻烦的。
- 如果 PC 上升触发，寄存器下降更新。则可能我们更新 PC 时候 拿到的是寄存器的旧值，从而错误判断跳转逻辑，造成程序混乱。
- 寄存器和数据存储器写入触发边沿不一致，可能导致写入混乱或者写冲突。
- 不同边沿触发可能导致同一时钟周期内出现数据 “读写冲突”，例如寄存器堆写入的数据被同时读取

### 在 CPU 启动执行后，如何实现在当前程序结束后，CPU 不再继续往下执行？ 

可以添加一个 `halt` 指令，功能是当执行了 `halt` 之后就把 `PC` 设置为不再自增。


具体实现可以是在 IFU **元件中**，对 PC 寄存器做一个使能端 halt

halt 指令的 opcode 是 `0x73`

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251215230852.png)


## 验收表格：
验收表格用 Excel 重新制作，这里附上图片。你也可以查看压缩包里的 excel 附件 

![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251217215253.png)

