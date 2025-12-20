## FPGA 设计和硬件描述语言

可编程逻辑器件（Programmable Logic Device, **PLD**）。
- 介绍可编程逻辑元件
- 介绍 FPGA 和其设计


### 存储器阵列 ROM, RAM

- `ROM`：(Read-only Memory, ROM) **只读存储器。**
	- **非易失性**。
	- 断电也不会消失。
- `RAM`: （Random-access Memory, RAM）**随机存取存储器**
	- **易失性存储器**
	- 静态 RAM
	- 动态 RAM

#### ROM
- **常见实现方式**
	- ROM 存储阵列：MOS **晶体管的有无**来区分存储 `0` 和 `1`
#### RAM
- 静态 RAM (SRAM)
- 动态 RAM (DRAM)

#### 存储阵列表示
`16*8位存储阵列`
- 存储 `16` 个 `8bit` 机器数


### FPGA 设计概述
**FPGA**（Field Programmable Gate Array, FPGA）: **现场可编程门阵列**