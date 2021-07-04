*一些需要你提前查询的概念：C语言、链接器、目标代码、反汇编*

**目标：学习使用C语言重写low-level功能的代码**

编译
-------

让我们看看C编译器如何编译我们的代码，并将其与用汇编器生成的机器码进行比较。  

我们将开始在`function.c`实现一个简单的函数。

为了编译出系统无关代码，我们需要在编译的时候传入`-ffreestanding`参数给GCC，就像这样：`i386-elf-gcc -ffreestanding -c function.c -o function.o`

让我们来看一下编译器生成的机器码是什么样的：
`i386-elf-objdump -d function.o`


链接
----

最后，为了生成一个二进制文件，我们将使用链接器。这一步的一个重要部分是了解高级语言如何调用函数的。我们是不知道函数在内存中的偏移量的，在这个例子中，我们将把偏移量置为在`0x0`，并使用`binary`格式生成没有任何标签和/或元数据的机器码，使用的命令就像下面这样：
`i386-elf-ld -o function.bin -Ttext 0x0 --oformat binary function.o`

*注意：使用ld进行链接的时候会出现Warning，不要在意。*

现在你可以使用`xxd`以二进制的形式查看`function.o`和`function.bin`里面的内容。
可以得出结论：`.bin`是纯机器码而`.o`里面包含很多用于debug的信息。


反编译
---------

出于好奇我们可能想看看反汇编出来的代码是什么样子的，使用下面这个指令就可以：
`ndisasm -b 32 function.bin`


更多
----

建议对下面列出的文件都进行编译、反编译、反汇编尝试一下，并看一下生成的机器码
- 使用局部变量的文件：`localvars.c`
- 包含函数调用功能的文件：`functioncalls.c`
- 使用指针的文件：`pointers.c`

请参考os-guide.pdf进行解释:为什么`pointer .c`的反汇编不像你所期望的那样? 字符"Hello"的ASCII码"0x48656c6c6f"被保存在哪里?  