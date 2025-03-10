*你可能需要事先查询的概念：C语言结构体、C语言数据类型、头文件保护、异常，还有一些关键字：packed、extern, volatile*

**目标：设置中断向量表（Interrupt Descriptor Table，IDT）来处理CPU产生的中断**

数据操作
----------

首先，我们将在`cpu/types.h`中定义一些特殊的数据操作`low_16`和`high_16`，这将帮助我们从char和int中分离出低位和高位，它被放在`cpu/`文件夹中，从现在开始我们将在这里放置一些机器相关的代码，bootsector虽然也是针对于x86编写的，但是仍然在`boot/`中，我们先不去管它。

一些之前的代码里面都已经使用了我们这节定义的`u8`、`u16`和`u32`类型，从这节之后我们的头文件都会有`ifdef`这种头文件保护机制。


中断
----------

中断是内核需要处理的主要事情之一，我们将实现它，以便能够在未来的课程中接受键盘输入。还有其它常见的中断:除数为0、越界、无效操作码、页面错误等。中断是在一个向量上处理的，结构和GDT类似，然而这次我们将用C来代替汇编编写IDT。


`cpu/idt.h`定义了idt的组成单元`idt_gate`的结构(需要256个`idt_gate`，可以为null，但如果数量不够cpu可能会产生panic)中的，还定义了由BIOS加载的idt结构，`idt_register`只包含一个内存地址和大小，结构很类似于GDT。最后，我们定义了`idt`和`idt_reg`变量，这两个变量将在加载的时候用到，`cpu/idt.c`里面`set_idt_gate`函数的功能只是用一个handler填充每个结构体，而`set_idt`函数只是用来组装传入给`lidt`指令的结构体，`lidt`指令可以从给定的地址加载idt。

ISRs
----

每当CPU检测到一个中断时，中断服务程序（Interrupt Service Routines，ISR）就会运行，中断通常是致命的。我们将编写足够的代码来处理它们，包括打印错误消息、停止CPU等。
在`cpu/isr.h`中，我们手动定义了32个中断服务程序，它们被声明为`extern`是因为它们将在`cpu/interrupt.asm`中实现。
现在你可以看看`cpu/isr.c`文件，如你所见，我们定义了一个函数`isr_install`来安装中断服务程序、加载IDT，还定义了错误消息列表、高级处理程序，你可以自定义`isr_handler`来做任何你想做的事情，我们这里只是简单的打印一些字符。

现在来看低层，底层代码将每个`idt_gate`的低层和高层处理程序绑定在一起，在文件`cpu/interrupt.asm`中我们定义了一个low level的ISR代码，它的功能就是保存/恢复状态并调用C代码，然后是实际的ISR的汇编实现，在`cpu/isr.h`上引用

注意`registers_t`结构体是如何表示我们在`interrupt.asm`中push进栈的所有寄存器的。

基本上是这样，现在我们需要在我们的Makefile中添加`cpu/interrupt.asm`，然后让内核安装isr并主动触发中断，请注意CPU是如何不停止的，即使在一些中断之后停止是一个好习惯。


cli指令
----
CLI汇编指令全称为Clear Interupt，该指令的作用是禁止中断发生，在CLI起效之后，所有外部中断都被屏蔽，这样可以保证当前运行的代码不被打断，起到保护代码运行的作用。STI汇编指令全称为Set Interupt，该指令的作用是允许中断发生，在STI起效之后，所有外部中断都被恢复，这样可以打破被保护代码的运行，允许硬件中断转而处理中断的作用。


函数栈
----
一般来说，我们将 %ebp 到 %esp 之间区域当做栈帧（也有人认为该从函数参数开始，不过这不影响分析）。并不是整个栈空间只有一个栈帧，每调用一个函数，就会生成一个新的栈帧。在函数调用过程中，我们将调用函数的函数称为“调用者(caller)”，将被调用的函数称为“被调用者(callee)”。在这个过程中，1）“调用者”需要知道在哪里获取“被调用者”返回的值（好像是存在eax通用寄存器中）；2）“被调用者”需要知道传入的参数在哪里，3）返回的地址在哪里。同时，我们需要保证在“被调用者”返回后，%ebp, %esp 等寄存器的值应该和调用前一致。因此，我们需要使用栈来保存这些数据。、
要注意函数参数应该是从后往前入栈，比如说以`registers_t`为参数的函数`isr_handler`首先压栈的是`u32 eip, cs, eflags, useresp, ss;`这些参数