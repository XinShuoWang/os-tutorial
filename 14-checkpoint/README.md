*你可能需要事先百度的概念：宏内核、微内核、debugger、dgb*

**目标：重新组织一下代码，还要学习使用gdb来调试内核**

我们的内核功能目前很少，只是打印一个`X`。但是为了以后工程的简洁和编写`Makefile`的方便，现在是时候把代码归类放到文件夹里。因为从现在开始我们将主要使用C来编写代码，所以我们将使用gdb连接到qemu来进行调试，不过首先让我们安装一个交叉编译的`gdb`。  

```sh
cd /tmp/src
curl -O http://ftp.rediris.es/mirror/GNU/gdb/gdb-7.8.tar.gz
tar xf gdb-7.8.tar.gz && mkdir gdb-build && cd gdb-build
../gdb-7.8/configure --ta rget="$TARGET" --prefix="$PREFIX" --program-prefix=i386-elf-
make && sudo make install
```

Makefile中有一个目标`debug`，这个目标可以构建出`kernel.elf`，这是一个目标文件(不是二进制文件)，包含了我们在内核中生成的所有符号，在gcc上开启`-g`参数就可以记录这些符号。你也可以使用`xxd`以16进制看一下里面的具体内容，你将会在里面看到一些字符串。实际上，检查对象文件中的字符串的正确方法是通过`strings kernel.elf`。  


我们可以使用`make debug`命令开启调试，然后在`gdb`命令行上输入以下内容：
- 在`kernel.c:main()`处打个断点: `b main`
- 让操作系统运行起来: `continue`
- 连续运行两步: `next`紧接着`next`. 你将会看到我们正要将`X`打印到屏幕上，但是在`qemu`上并没有开始打印。
- 让我们看看在VGA内存里面有什么内容: `print *video_memory`，打印出的`L`来自于`Landed in 32-bit Protected Mode`
- 让我们看看`video_memory`指向的地址: `print video_memory`
- 使用`next`命令继续复制将`X`放入VGA内存
- 让我们看看成功了没有: `print *video_memory`

现在最好去学习一下GDB的用法，比如`info registers`命令，这将让我们省很多事。


您可能会注意到，由于这是一个教程，我们还没有讨论我们将编写哪种内核，它可能是一个宏内核，因为它们更容易设计和实现，毕竟这是我们的第一个操作系统。
