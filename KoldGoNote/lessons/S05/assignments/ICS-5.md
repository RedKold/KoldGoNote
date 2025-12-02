---
本科课程: ICS
date: 2025-11-10
author: 231275036-朱晗
completed: "true"
---

## 作业清单
第五章作业：教材第五章后习题中的第3、4、5、6（3）、8、9、11题。

### 3
```
|	符号		|	是否在`test.o`的符号表中	|	定义模块	|	符号类型	|	节		|
|	a		|	yes (extern refer it)	|	main.o	|	extern	|	UND		|
|	val		|	yes						|	test.o	|	global	|	.data	|
|	sum		|	yes						|	test.o	|	global	|	.text	|
|	i		|	no(in stack)			|	null	|	null	|	null	|	
```
### 4

```
| 符号		|	是否在`swap.o` 的符号表中	|	定义模块	|	符号类型	|	节		|
|	buf		|	yes						|	main.o	|	extern	|	UND		|
|	bufp0	|	yes						|	sum.o	|	global	|	.data	|
|	bufp1	|	yes						|	sum.o	|	local	|	.bss	|
|	incr	|	yes						|	sum.o	| 	local	|	.text	|
|	count	|	yes						| 	sum.o	|	local	|	.data	|
|	swap	|	yes						|	sum.o	|	global	|	.text	|
|	temp	|	no						|	~		|	~		|	~		|
```

### 5

假设一个 C 语言程序包含两个源文件 `main.c` 和 `proc1.c`，内容如图所示。回答上述问题
#### （1）上述两个文件哪些是强符号？哪些是 common 符号？
强符号：`x, y, z` in `main.c`, common 符号 `proc1` in `main.c`
强符号：`proc1` in `proc1.c`, common 符号 `x` in `proc1.c`

#### (2）程序执行后的打印结果是什么？请分别画出执行第 6 行的 `proc1` 函数前后，地址 `&x` 和 `&z` 中存放的内容。若第 3 行改为 `short y=1, z=2` 打印结果是什么？

打印结果 `x=0, z=0`

之前：
-  `&x: 0000 0101`
- `&z: 0000 0002`

改为 `short y=1, z=2`:
打印结果为 `x=0, z=-16392`


#### (3) 修改文件 `proc1.c` 使得 `main.c` 能输出正确的结果（即 `x=257, z=2`）。要求修改时不能改变任何变量的数据类型和名字。

本质是 `double` x 在 `proc1` 中 refer 了 main. c 中的 x，对后面两个 `short` 进行了 override
So, we let double `x`  = int (257) + short (0) +short (2) 这个字节就可以了。
对应是什么数呢？

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108170704.png)

实际上我们不关心这是什么数。我们直接硬修改其字节的值即可。
```c
double x;
#include <stdio.h>

typedef struct {
	unsigned x;
	short y;
	short z;
}s;
union {
    s a;
    double d;
} u;


void proc1()
{
	u.a = (s){
		257,0,2
	};
	double ans=u.d;
	printf("%lf\n",ans);
	x=ans;
}
```


### 6（3）
在模块 `mj` 中对符号 `x` 的任意引用于模块 `mi` 中定义的符号 `x` 关联记为 `REF(mj.x) -> DEF(mi.x)`. 请在下列空格处填写模块名和符号名，

(3)
```
/* m1.c */				/* m2.c	*/
int p1(void);			int x=10;
int p1;					int main;
int main()				int	p1()
{						{
	int x=p1();				main=1;
	return x;				return x;
}						}
```
**有链接问题**
- `m1.c` 中，变量不能和函数同名。所以 `int p1` 会导致链接错误。

`REF(m1.main)-> DEF(m1.main)`
`REF(m2.main)->DEF(m2.main)`
`REF(m1.p1)->DEF(m2.p1)` (if we delete `int p1`)
`REF(m1.x)->局部变量不存在关联`
`REF(m2.x)->DEF(m2.x)`
### 8
`.bss` 中的未初始化的全局变量等不需要占用空间。这可能节生了可执行文件中的一部分大小。
### 9 最短命令行链接静态库
`a->b` 表示 `b` 中定义了一个被 `a` 引用的符号

[[ICS#Using Static Libaries|Command line order matters]]
#### (1) `p.o -> libx.a -> liby.a`
```bash
gcc p.o libx.a liby.a 
```

#### (2) `p.o -> libx.a -> liby.a while liby.a -> libx.a`

// 出现了循环依赖
```bash
gcc p.o libx.a liby.a libx.a 
# liby.a rely on libx.a, but libx.a still have some ref in liby.a, so we scan liby.a again
```

#### (3) `p.o -> libx.a -> liby.a ->libz.a while liby.a -> libx.a -> libz.a`

```bash
gcc p.o libx.a liby.a libz.a libx.a liby.a 
```

`liby.a -> libx.a -> liby.a` 循环依赖，必须要 `read liby.a, read libx.a, read libxy.a` 才可以依赖建立完成。
- `x` 可能引用了  **`y` 中引用了 `x` 的符号。**

### 11 
图 5.20 给出了图 5.9b 中 `swap.c` 对应的 `swap.o` 中 `.text` 节和 `.rel.text` 节的内容，图中显示 `.text` 节中共有 6 处需重定位。假定链接后生成的可执行文件中 `buf` 和 `bufp0` 的存储地址分别是 `0x80495c8` 和 `0x80495d0`, `bufp1` 的存储地址位于 `.bss` 节的开始，为 `0x8049620`。根据对图 5.20 的分析，仿照例子填写表 5.3，**指出各个重定位**的符号名、相对于 `.text` 节起始位置的位移、指令所在行号、重定位类型、重定位前的内容、重定位后的内容。


![2d155b8473831c8c5ea9de344a6be917_720.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/2d155b8473831c8c5ea9de344a6be917_720.png)


```c
extern int buf[];
int *bufp0 = &buf[0]
int *bufp1;

void swap(){
	int temp;
	bufp1 = &buf[1];
	temp = *bufp0;
	*bufp0 =*bufp1;
	*bufp1 = temp;
}
```


| no. | symbol name    | offset | line  | type       | before       | after       |
| --- | -------------- | ------ | ----- | ---------- | ------------ | ----------- |
| 1   | `bufp1 (.bss)` | `0x8`  | 6-7   | `R_386_32` | `0x00000000` | `0x8049620` |
| 2   | `buf`          | `0xc`  | 6-7   | `R_386_32` | `0x00000000` | `0x80495c8` |
| 3   | `bufp0`        | `0x11` | 10-11 | `R_386_32` | `0x00000000` | `0x80495d0` |
| 4   | `bufp0`        | `0x1b` | 14-15 | `R_386_32` | `0x00000000` | `0x80495d0` |
| 5   | `bufp1 (.bss)` | `0x21` | 16-17 | `R_386_32` | `0x00000000` | `0x8049620` |
| 6   | `bufp1 (.bss)` | `0x2a` | 20-21 | `R_386_32` | `0x00000000` | `0x8049620` |



