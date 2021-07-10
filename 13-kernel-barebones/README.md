*你可能需要先查询一下的概念：内核、ELF文件格式、makefile*

**目标：创建一个简单的内核然后从之前写的bootsector中启动它**

内核
----------

我们的内核会在屏幕的左上角打印字符'X'，你可以打开`kernel.c`文件看看是怎样实现的。

您将注意到`kernel.c`中包含一个不执行任何操作的函数，增加该函数的原因是因为我们不想使用默认的0x00地址去寻找启动函数，使用默认地址将失去一些便利性，我们将使用链接器完成从汇编里调用C语言的函数的功能，具体语法如下：
```
[extern main] ; 告诉链接器：main函数定义在外部文件中
call main ; 调用main函数，链接器知道main函数在二进制文件中的位置。
```

使用下面的命令来编译`kernel.c`文件：
`i386-elf-gcc -ffreestanding -c kernel.c -o kernel.o`

阅读`kernel_entry.asm`文件中的代码，你将看到如何在程序中使用`[extern]`声明。
和之前直接生成二进制文件不同，我们这次将生成一个`elf`格式的文件，稍后将把它和`kernel.o`链接。  
`nasm kernel_entry.asm -f elf -o kernel_entry.o`

ELF文件：Executable and Linkable File

链接：链接的过程实际上是为了解决多个文件之间符号引用的问题(symbol resolution)。编译时编译器只对单个文件进行处理，如果该文件里面需要引用到其他文件中的符号（例如全局变量或者函数），那么这时在这个文件中该符号的地址是没法确定的，只能等链接器把所有的目标文件连接到一起的时候才能确定最终的地址。这个填写地址的过程，根据填写的地址的类型和时机，可以分为解决程序内部跨文件引用的链接时重定位、引用外部库文件的装载时重定位和引用外部库文件时为了加快加载速度而引入的延迟绑定。


链接器
----------

使用如下命令将之前生成的两个ELF文件进行链接生成一个二进制文件：
`i386-elf-ld -o kernel.bin -Ttext 0x1000 kernel_entry.o kernel.o --oformat binary`

我们的内核不会被放到内存的`0x0`位置而是被放到`0x1000`位置，我们稍后还会在bootsector中指定这个参数。


Bootsector
--------------

这里面的Bootsector程序和第10个教程里面的代码很相似，具体可以打开`bootsect.asm`文件看看。

使用下面的命令编译Bootsector：
`nasm bootsect.asm -f bin -o bootsect.bin`


合并文件
-----------------------

现在我们有两个文件：`bootsect.bin`和`kernel.bin`，我们怎么能把他们合并到一起呢？
我们可以直接使用`cat`命令来合并两个文件：
`cat bootsect.bin kernel.bin > os-image.bin`


Run!
----

You can now run `os-image.bin` with qemu.
现在你可以使用qemu来运行`os-image.bin`文件了

如果在加载的时候出现了错误你需要设置qemu的参数(软盘 = `0x0`, 硬盘 = `0x80`)，或者可以使用`qemu-system-i386 -fda os-image.bin`。

你会看到以下4条消息：
- "Started in 16-bit Real Mode"
- "Loading kernel into memory"
- (左上角) "Landed in 32-bit Protected Mode"
- (左上角, 会覆盖上一条的部分消息) "X"



Makefile
--------
在最后一步，我们将使用Makefile来简化编译流程。你可以看看`Makefile`中的内容，如果有什么不懂得请使用搜索引擎。
