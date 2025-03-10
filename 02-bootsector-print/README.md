*一些可能需要你查询的概念: 中断, 寄存器*

**目标：在上一个实验的基础之上让OS可以打印一些字符**

我们将改进我们的上一个无限循环引导扇区，并在屏幕上打印一些东西，我们将使用中断实现这项工作。  

在这个例子中我们要把“Hello”的每一个字符写入`al`寄存器，`al`寄存器是属于`ax`的寄存器的低地址部分，还要把`0x0e`写入到`ah`寄存器之中，`ah`寄存器是`ax`寄存器的高地址部分，之后就触发一个`0x10`中断，这个中断在视频输出中是一个比较常用的中断。

`ax`寄存器解释：
```
AX BX CX DX是CPU内部的通用寄存器中的数据寄存器,数据寄存器一般用于存放参与运算的数据或运算的结果,每一个数据寄存器都是16位的(即16个二进制位), 但又可以将高,低8位分别作为两个独立的8位寄存器使用.它们的高8位记作AH,BH,CH,DH,低8位记作AL,BL,CL,DL.这种灵活的使用方 法给编程带来极大的方便,既可以处理16位数据,也能处理8位数据.
数据寄存器除了作为通用寄存器使用外,它们还有各自的习惯用法
AX 称为累加器,常用于存放算术逻辑运算中的操作数,另外所有的I/O指令都使用累加器与外设接口传送信息
BX 称为基址寄存器,常用来存放访问内在时的基地址,
CX 称为计数寄存器,在循环和串操作指令中用作计数器
DX 称为数据寄存器,在寄存器间接寻址中的I/O指令中存放I/O端口的地址
另外,在做双字长乘除法运算时,DX 与AX合起来存放一个双字长数(32位),其中DX存放高16位,AX存放低16位.
```

将`0x0e`存放入`ah`寄存器是要告诉显示中断：我们想把`al`里的内容以tty模式写到屏幕上

我们在此例中将只设置tty模式一次，尽管在现实世界中我们不能确定`ah`寄存器内的内容是不变的，因为当我们的进程休眠的时候，一些其他进程可能在这个CPU上运行，这个进程结束之后也没有进行正确清理，这样就会留下垃圾数据在`ah`寄存器中。在这个例子中我们并不需要考虑这个事情，因为我们的进程是唯一一个在运行的程序。


新的启动引导扇区代码：
```nasm
mov ah, 0x0e ; tty mode
mov al, 'H'
int 0x10
mov al, 'e'
int 0x10
mov al, 'l'
int 0x10
int 0x10 ; 'l' is still on al, remember?
mov al, 'o'
int 0x10

jmp $ ; jump to current address = infinite loop

; padding and magic number
times 510 - ($-$$) db 0
dw 0xaa55 
```

代码解释：
```
mov al, 'l'
int 0x10
int 0x10 ; 'l' is still on al, remember?
```
这部分代码是这个样子的原因是因为要重复显示字符`l`，而且之前`al`寄存器里就已经存放了`l`这个值，所以不需要再进行一遍对`al`寄存器的赋值，从这里也可以看出来：调用`int 0x10`就会显示一次`al`寄存器里面的值。

你可以在编译完成之后通过`xxd file.bin`这个命令来查看文件的十六进制表示。

编译：
`nasm -fbin boot_sect_hello.asm -o boot_sect_hello.bin`

运行：
`qemu boot_sect_hello.bin`

结果：
引导扇区会在屏幕上打印"Hello"这个字符，然后就处于无限循环之中。
