## Section 2

- 对连续信息采样->以使信息离散化
- 对离散样本用 0 和 1 进行编码
- **转化**
	- ISA: 指令系统能识别的基本类型数据。
- ISA 中：整数、浮点数的计算完全不一样


- Pre Course
	- 计算机的工作流程？
		- 根据` PC` 取指令
		- 指令译码
		- 按**地址**取操作数
		- 执行指令
		- 回写地址
		- 修改 `PC` 的值
	- **信息在计算机里是什么？**
		- 二进制串。指令和数据

### About Number Data
- **真值和机器数**
	- **机器数**：用 0 和 1 编码的计算机内部的 0/1 序列
	- **真值**：机器数真正的值，如：现实中带正负的数。
- Decimal / Binary
	- we often use a index to express **radix/ base** of a number
	- `B-binary`, `H-hexadecimal` (or prefix `ox`), `O-octonary`
- use binary to express hexadecimal or octal is very convenient
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20250903142600.png)
	- 小数部分不是特别直观，可以说一下
		- 为什么 **取整**？
		- 你可以假设小数部分 `
$$
x=\sum_{k=1}^{n}\left( \frac{1}{2} \right)^{k}\times 1_{\text{the $k$ th bit of binary pattern is 1}}
$$

我们考虑该求和列下标 `i` 的项，其为 $(1/2)^{i}$, 如果它乘了 $i$ 次 `2` 除了一个 1，就说明该二进制串的第 `i` 位（从低位到高位数）是 `1`
（这非常自然）。

所以我们从高位到低位依次确认即可。


- 八进制和十六进制完全一致（转化方法）


### Expression of value data
**数值数据** **Three Elements**
	- **进位计数制**
	- **定、浮点表示**
	- **如何用二进制编码**
		- 补码？
		- 反码？
		- 原码？
		- 移码
- 三要素齐全，才可确定 $\text{machine number}\to\text{true value}$
	- You can' t tell the true value of `1011001` without knowing 3 elements above


#### 编码方式

可见 ICS 教学。

[[ICS#Section 2 ：数据的机器级表示与处理]]


- 归纳一些有趣的事情
- Unsigned Integer
	- 没有符号位，。也无需使用原码补码移码
	- 能表示更大的最大值
	- 总是整数。

关于浮点数，也可以看 ICS 的。

#### BCD 码

- 编码思想：每个十进制数位（0-9）至少有 4 位二进制表示。而 4 位二进制位可组合位 16 种状态
- 6 个冗余，可利用起来


##### 十进制有权码
- 每个十进制数字的 4 个二进制位（称为基 2 码）都有确定的权
- `8421` 码是最常用的十进制码
	- 自然 BCD 码（AKA）
- BCD（Binary-Coded Decimal，二-进制编码十进制）是一种用二进制表示十进制数字的方法。它的核心思想是：**每个十进制数字（0~9）用固定的 4 位二进制表示**，而不是把整个数字直接转换成二进制。
### Some Note

> [!Note]  思考：`10` 在计算机中有几种可能的表示？
> 原码（只有一个情况：浮点数的尾数部分，未经过规格化）、补码（最常见）、BCD 码、**无符号数**(unsigned，很不一样！)
> 

- 现代计算机不再用原码表示整数
