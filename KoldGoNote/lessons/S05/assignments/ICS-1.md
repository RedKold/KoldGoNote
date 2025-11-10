---
completed: "true"
course: 计算机系统
tags:
  - assignment
author: 231275036-朱晗
---

### 13 异或程序探讨
```c
void xor_swap(int *x, int *y)
{
	*y=*x ^ *y;
	*x=*x ^ *y;
	*y=*x ^ *y;
}
```

1. `*x = a, *y = a ^ b`
2. `*x = a ^ (a ^ b) = a ^ a ^ b = 0 ^ b =b`
3. `*y = b ^ (a ^ b) = 0 ^ a = a`

### 14 调用 `xor_swap`
- `len` 为奇数的时候，头尾两两配对之后，最后中间的元素会是 `left == right` 时，会将数组中间的元素 `a[mid]`  会执行 `xor_swap(&a[mid], &a[mid])` 而，导致中间元素变为 `0`，导致错误。
- 将 `left <= right` 改为 `left < right`

### 15 十六进制玩法

`x`, `y` is `char` variable


| `x`    | `y`    | `x^y`  | `x&y`  | `x\|y` | `~x\|~y` | `x&!y` | `x&&y` | `x \|\| y` | `!x \|\| !y` | `x&&~y`    |
| ------ | ------ | ------ | ------ | ------ | -------- | ------ | ------ | ---------- | ------------ | ---------- |
| `0x3E` | `0xAB` | `0x95` | `0x2A` | `0xBF` | `0x54`   | `0x3E` | `0x00` | `0x01`     | `0x01`       | `0x01`     |
| `0xC8` | `0xF0` | `0x38` | `0xC0` | `0xF8` | `0x3F`   | `0x00` | `0x01` | `0x01`     | `0x00`       | `0x00`     |
| `0x8F` | `0x70` | `0xff` | `0x00` | `0xff` | `0xff`   | `0x00` | `0x01` | `0x01`     | `0x00`       | `0x00`     |
| `0x09` | `0x55` | `0x5c` | `0x01` | `0x5d` | `0xfe`   | `0x00` | `0x01` | `0x01`     | `0x00`       | `0x00`<br> |

### 16 一个 `n, n>=8` 位的变量 `x`，根据 C 语言按位运算的定义，写出满足下列要求的 C 语言表达式
- `x` 的最高有效字节不变，其余各位全变为 `0`
	`x = x & (0xFFULL << ((sizeof(x) - 1) * 8));`
- `x` 的最低有效字节不变，其余各位全变为 `0`
	- `x = x & 0xFF`
- `x` 的最低有效字节全变为 `0`，其余各位取反
	- `x = ~x & ~0xFF`
- `x` 的最低有效字节全为 `1`，其余各位不变。
	- `x = x | 0xFF`

### 17 反汇编器生成的小端处理器的机器级代码表示文本

- `b8 01 00 00` -  `-1207894016`
- `14` - `20`
- `58 fe ff ff` -  `1493106687`
- `74 fe ff ff` - `1962868735`
- `44` - `68`
- `c8 fe ff ff` - `-922812417`
- `10` - `16`
- `0c` - `12`
- `ec fe ff ff` - `-318832641`
- `20` - `32`

### 18 C 语言 `compare_str_len` 的问题
```c
int compare_str_len(char *str1, char *str2){
	return strlen(str1) - strlen(str2) > 0;
}
```

- 标准库声明 `strlen` 是 `size_t strlen(const char*s)`, 则可能发生溢出问题。即在 `strlen(str1) - strlen(str2)` 的结果 `>2147483647`，则可能导致被解读为负数而错误返回
- 修改建议：
	- 用更长的 `long int` 来声明函数。避免无符号整数溢出问题。

### 21 反汇编猜原参数

- 推测 `M=15`, `N=4`
- 汇编将乘法优化为位移和减法，除法先用 `if(y<0) y+=3` 优化负数移位的异常行为（因为负数除法是向靠近 0 取整。简单右移是向远离 0 取整）



### 30 在字长为 `32` bit 的计算机上，有一个 C 函数原型声明为 `int ch_mul_overflow(int x, int y);` ，该函数用于对两个 `int` 变量 `x` 和 `y` 的乘积判断是否溢出，若溢出返回 `1`，否则返回 `0`。请使用 `64` 位整型 `long long` 来编写该函数



```c
#include <limits.h>

int ch_mul_overflow(int x, int y) {

    long long res = (long long)x * y;

    // 检查乘积是否超出 int 类型的范围
    if (res > INT_MAX || res < INT_MIN) {
        return 1; // 溢出
    } else {
        return 0; // 未溢出
    }
}
```

### 34 无符号整数变量 `ux` 和 `uy`，判断表达式真假
声明初始化
- `unsigned ux=x;`, `unsigned uy=y`
- 若 `sizeof(int)=4`, for every `int` variable `x`, `y`, judge whether expression here is always true. if not, give the value of x and y when it's false.
- `(x*x) >= 0`
	- `not always`, `x=65535=0x0000ffff, y=-131071=0xfffe0001
- `(x-1<0) || x>0`
	- `not always`, `x=INT_MIN=-2147483648`
- `x<0 || -x <=0`
	- true
- `x>0 || -x >=0`
	- `not always`, let `x=INT_MIN`, `-x` is also `INT_MIN`, so both `x,-x` <0
- `x & 0xf!=15 || (x<<28)<0`
	- the first expr is: x's low 4bit is not `1111_b`
	- the second expr means: `left-move` `x` `28` bit, `the 31-th bit` is `1`. (负数)
	- 这意味着 `3rd bit of x` is `1`.
	- IF the first expr is false, then the second must be true.
	- IF the second expr is false, the first must be true
	- **true**
- `x>y==(-x<-y)`
	- 参考第四条，`-(INT_MIN)=INT_MIN`，
	- `not always`, `x=1,y=INT_MIN`
- `~x+~y==~(x+y)`
	- `not always`
	- 注意到，`~k=-k-1`
	- the original equation transfom to `-x-1 + (-y-1)`, obviously it's not equal to `~(x+y)=-(x+y)-1`
- `(int)(ux-uy) == -(y-x)`
	- `true`
- `((x>>2)<<2) <= x`
	- `true`
- `x*4+y*8==(x<<2) + (y<<3)`
	- `true`
- `x/4+y/8 ==(x>>2)+(y>>3)`
	- `not always`
	- `x=-3,y=-7`
- `x*y==ux*uy`
	- `true`
		- but may depends on interpreter's behavior
- `x+y==ux+uy`
	- `false`
	- `x=INT_MAX=2147483647,y=1`
- `x*~y+ux*uy==-x`
	- 根据前面提到的 `x+y == uy+ux`
	- `x*~y + y*x == -x`
	- `~y=-y-1`，代入即证明。


### 35 变量 `dx` `dy` `dz` 的声明和初始化如下

```c
double dx = (double) x;
double dy = (double) y;
double dz = (double) z
```

- 若 `float` 和 `double` 分别采用 `IEEE 754`..., `sizeof(int)=4`，对于任意 `int` 变量 `x,y,z`，下列表达式是否永真


- `dx*dy>=0`
	- `true`
- `(double)(float) x == dx`
	- `false`
	- 有精度损失。`float` 不能表达 `10` 位数以上的 `INT`。
	- `x=2000000000`
- `dx+dy == (double) (x+y)`
	- `false`
	- `x=2147483647`
	- 则左边不会溢出，右边溢出
- `(dx+dy)+dz == dx+(dy+dz)`
	- `false
	- 存在大数吃小数
	- - `dx = 1e8`
	- 令 `dy = -1e8`
	- 令 `dz = 1`
	1. **左边：`(dx + dy) + dz`**
	    - `dx + dy = 1e8 + (-1e8) = 0.0`
	    - `(dx + dy) + dz = 0.0 + 1.0 = 1.0`
	    - **左边结果是 `1.0`**。
	2. **右边：`dx + (dy + dz)`**
	    - `dy + dz = -1e8 + 1`
	    - 在浮点数运算中，`1` 相对 `1e8` 太小，`dy+dz` 的结果会被舍入为 `-1e8`。
	    - `dx + (dy + dz) = 1e8 + (-1e8) = 0.0`
	    - **右边结果是 `0.0`**。
- `dx/dx == dy/dy`
	- `if x==0, dx/dx would be NaN`
