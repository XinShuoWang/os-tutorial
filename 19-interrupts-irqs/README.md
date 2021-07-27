*你可能需要提前查询的概念：IRQs, PIC, polling*

IRQ：An IRQ is an interrupt request from a device.
PIC：Programmable Interrupt Controller

**目标：实现完整的中断**

PIC会在CPU启动时将硬件中断的0-7号映射到0x8-0xF, 硬件中断的8-15号映射到0x70-0x77，可以看到，这与我们上节课编写的0-31号ISR中断冲突，所以我们需要将IRQ重新映射到Interrupt Service Routines的32-47号。PIC通过I/O端口进行通信，主PIC的命令端口为0x20，数据端口为0x21，从PIC命令端口为0xA0，数据端口为0xA1。

这是重映射PIC的代码，详情可以参考下面的注释，重映射完成之后我们会把IRQ加入到`idt_gate_t`数组里面。
```
    // Remap the PIC
    // ICW stands for "Initialization Commands Words"
    port_byte_out(0x20, 0x11); /* write ICW1 to PICM, we are gonna write commands to PICM */
    port_byte_out(0xA0, 0x11); /* write ICW1 to PICS, we are gonna write commands to PICS */
    port_byte_out(0x21, 0x20); /* remap PICM to 0x20 (32 decimal) */
    port_byte_out(0xA1, 0x28); /* remap PICS to 0x28 (40 decimal) */
    port_byte_out(0x21, 0x04); /* IRQ2 -> connection to slave */ 
    port_byte_out(0xA1, 0x02);
    port_byte_out(0x21, 0x01); /* write ICW4 to PICM, we are gonna write commands to PICM */
    port_byte_out(0xA1, 0x01); /* write ICW4 to PICS, we are gonna write commands to PICS */
    port_byte_out(0x21, 0x0);  /* enable all IRQs on PICM */
    port_byte_out(0xA1, 0x0);  /* enable all IRQs on PICS */
```

现在我们来看看汇编程序，在`interrupt.asm`文件中，第一步是为我们刚刚在C代码中定义的的IRQ符号添加全局定义（类似这样`global irq0`）以方便链接器进行链接。第二步是在文件最后添加IRQ处理程序，注意它们都会跳转到一个新的段:`irq_common_stub`，这个段的处理程序与`isr_common_stub`非常相似，它位于`interrupt.asm`文件顶部，它还定义了一个新的`[extern irq_handler]`。

现在来看看C代码，在`isr.c`文件中有一个`irq_handler()`函数，它向PIC发送一些指令并调用适当的处理程序，该处理程序存储在名为`interrupt_handlers`的数组中，这个数组定义在文件顶部。新的结构体定义在`isr.h`文件中，我们还将使用一个简单的函数来注册中断处理程序。

现在我们可以定义我们的第一个IRQ处理程序了！因为`kernel.c`文件没有变化，所以没有新的东西可以运行。