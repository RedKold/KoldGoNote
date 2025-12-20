# 作业

- [[ics-1]]
- [[ics-2]]
- [[ics-3]]
- [[ics-5]]
- [[ics-6]]
- [[ics-7]]
- [[ics-8]]

# pa
- [[ics-pa1-report]]
- [[ics-pa1 note]]
- [[ICS-PA2 report]]
- [[ics-pa2 note]]
- [[ics-pa3-note]] 
- [[ics-pa3-report]]
- [[ICS-lab-cachesim]]

# 授课
## [[ICS - Section 2 ：数据的机器级表示与处理]]
## [[ICS - Section 3 程序的转换和指令系统]]
## [[ICS - Section 4 程序的机器级表示]]

- from c to object code
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108123433.png)

##  [[ICS - Section 5 程序的链接和加载执行]]

##  [[ics - section 6 内存层次结构 memory hierarchy]]



##  [[ics - section 7 虚拟存储器]]




##  [[ics - section 8 异常处理控制流]]

## ics - section 9 I/O

### `stdio.h` 
系统调用开销很大。
- 系统级别 I/O 函数对文件的标识是**文件描述符**，C 标准 I/O 库函数对文件的标识是 **指向 `FILE*`** 结构的指针
- FILE 中定义了 1024 字节的缓存区
	- `stdio`: store in  ` buf ` then output
	- `stderr`: no buffer, directly **out**;
- `_fillbuf()` 一个在输出前填充缓冲区的函数

我们最终是通过 `syscall` 的 `write` 和 `read` 来实现的。
回忆一下 PA [[ics-pa3-report#必答题(需要在实验报告中回答) - hello程序是什么, 它从而何来, 要到哪里去|hello程序发生了什么]]


**内核空间**
- **三种方式**
- **会算效率**