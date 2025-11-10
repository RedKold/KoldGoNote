
## 自陷

实现指令
```
80001410:   30571073            csrw    mtvec,a4
```

其在 `abstract-machine/am/src/$ISA/nemu/`

```c
bool cte_init(Context*(*handler)(Event, Context*)) {
  // initialize exception entry
  asm volatile("csrw mtvec, %0" : : "r"(__am_asm_trap));

  // register event handler
  user_handler = handler;

  return true;
}
```
定义。

我们需要增加对这条指令的支持。