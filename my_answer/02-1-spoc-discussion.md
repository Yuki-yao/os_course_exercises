## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS

- BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

  是主引导记录（MBR）。操作系统的内核映像由MBR负责加载到内存中。之所以不用BIOS直接加载，原因有两个：第一，BIOS运行时文件系统尚未建立，BIOS也不知道硬盘里存放着什么内容；第二，对于不同分区拥有不同操作系统的硬盘，BIOS无法判断应该从哪一个分区启动。

- 比较UEFI和BIOS的区别。

  UEFI的优势在于：系统启动文件与OS自身隔离，安全性更强；可以通过EFIShell选择启动文件，启动配置更灵活；支持容量更大，可以引导超过2TB的硬盘。

## 3.2 系统启动流程

- 分区引导扇区的结束标志是什么？

  0x55AA。

- 在UEFI中的可信启动有什么作用？

  在启动前进行数字签名检查，保证启动介质的安全性。

## 3.3 中断、异常和系统调用比较

- 什么是中断、异常和系统调用？

  中断：来自硬件设备的处理请求；

  异常：非法指令或其他原因导致当前指令执行失败

  系统调用：应用程序主动向OS发出服务请求

- 中断、异常和系统调用的处理流程有什么异同？

  - 源头：中断--外设，异常--应用程序意想不到的行为，系统调用--应用程序请求操作提供服务
  - 响应方式：中断--异步，异常--同步，系统调用--异步或同步
  - 处理机制：中断--持续、对应用程序透明，异常--杀死或重新执行意想不到的应用程序指令，系统调用--等待和持续

- 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？

  22个系统调用如下：

  ```c
  static int (*syscalls[])(uint32_t arg[]) = {
      [SYS_exit]              sys_exit,
      [SYS_fork]              sys_fork,
      [SYS_wait]              sys_wait,
      [SYS_exec]              sys_exec,
      [SYS_yield]             sys_yield,
      [SYS_kill]              sys_kill,
      [SYS_getpid]            sys_getpid,
      [SYS_putc]              sys_putc,
      [SYS_pgdir]             sys_pgdir,
      [SYS_gettime]           sys_gettime,
      [SYS_lab6_set_priority] sys_lab6_set_priority,
      [SYS_sleep]             sys_sleep,
      [SYS_open]              sys_open,
      [SYS_close]             sys_close,
      [SYS_read]              sys_read,
      [SYS_write]             sys_write,
      [SYS_seek]              sys_seek,
      [SYS_fstat]             sys_fstat,
      [SYS_fsync]             sys_fsync,
      [SYS_getcwd]            sys_getcwd,
      [SYS_getdirentry]       sys_getdirentry,
      [SYS_dup]               sys_dup,
  };
  ```

  功能大致可以分为进程管理(fork/exit/wait/exec/yield/kill/getpid/sleep)，文件操作(open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup)，内存管理(pgdir)，外设输出(putc)。

## 3.6 请分析函数调用和系统调用的区别

- 系统调用与函数调用的区别是什么？

  - 指令不同：系统调用对应指令为INT和IRET，函数调用在常规调用时则对应CALL和RET
  - 特权级不同：系统调用存在特权级转换的情况，常规调用不存在
  - 堆栈切换：系统调用有堆栈切换，常规调用没有

- 通过分析`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较函数调用与系统调用的堆栈操作有什么不同？

  函数调用比系统调用多一步SP压栈传参的操作。