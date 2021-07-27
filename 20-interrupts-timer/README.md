*你可能需要事先查询的概念：定时器、键盘中断、scancode*

**目标：实现CPU定时器和键盘输入处理程序**

定时器
-----

定时器很容易配置，首先我们将在`cpu/timer.h`中声明一个`init_timer()`函数，并在`cpu/timer.c`中实现，函数的功能也只是计算时钟频率并将字节发送到适当的端口。我们还要做的就是修复`kernel/utils.c`里面的`int_to_ascii()`函数，使其以正确的顺序打印数字，为此，我们需要实现`reverse()`和`strlen()`函数。最后，回到`kernel/kernel.c`并做两件事：再次启用中断、初始化计时器中断。这时候你就可以使用`make run`命令来看到效果了！

键盘
--------

键盘操作更简单，但有一个缺点：PIC并没有向我们发送按下键的ASCII码，而是发送“按下”和“松开”操作的的scancode，所以我们需要翻译它们。在`drivers/keyboard.c`文件中有两个函数：回调函数和配置中断回调的初始化函数。`keyboard.h`文件中放的是定义，`keyboard.c`中是函数实现，在其中有一个很长的`switch`匹配，它是用来将扫描码转换为ASCII码的。目前，我们将只实现美国键盘输入的一个简单子集，你可以在这里阅读更多关于[扫描码](http://www.win.tue.nl/~aeb/linux/kbd/scancodes-1.html)的内容。
