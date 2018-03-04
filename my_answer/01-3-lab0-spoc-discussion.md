## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

  至少需要内存、TLB、可持久化存储设备，以及CPU对特权级的支持。

  需要提供的特权指令包括Exceptions, Virtual Memory, Privilege Modes, Segmentation/Paging相关的指令。

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

  实模式下的系统程序和用户程序没有区别对待，且程序指针指向的是实在的物理地址，这就导致用户程序的指针一旦指向其他程序甚至系统程序的内存区域，并进行数据修改，将带来非常危险的后果。另外，实模式下可访问的物理内存不超过1MB。保护模式则不同，此时程序访问的地址为虚拟地址，物理内存得到了保护，同时可寻址范围扩展到4GB。

  物理地址是处理器提交到总线上的用于访问的最终地址，逻辑地址是程序指令访问时给出的地址。线性地址是二者之间的中间层，属于段式内存管理中变换形成的地址空间。

- 理解list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义

```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

这些数字对应每一个域在结构体中所占的位数。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```

如果在其他代码段中有如下语句，

```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```

请问执行上述指令后， intr的值是多少？

intr的值为0x0003.