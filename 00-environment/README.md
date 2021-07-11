*你可能需要事先搜索一下的概念：Linux、terminal、compiler、emulator、nasm、qemu*

**目标：配置这个项目所需的软件环境**

在Ubuntu上你这可以这样安装依赖：```sudo apt install build-essential texinfo qemu-system-x86 nasm```
qemu启动时可能不能直接使用```qemu binfile```这样的命令，你可能需要这样使用qemu：```qemu-system-x86_64 binfile```，你也可以在```~/.bashrc```里面配置一下，像这样```alias qemu="qemu-system-x86_64"```，这样的话就可以直接使用```qemu binfile```的命令了，我之后的教程都会这样来操作。

不需要纠结不会nasm汇编语法，语法可以慢慢学。


