*你可能需要提前查询一下的概念：硬盘、柱面、磁头、扇区、进位*

**目标：为了以后能从硬盘加载kernel，我们现在需要学习一下如何从磁盘加载数据**

我们的操作系统不可能只有512字节大小，所以我们要从磁盘加载kernel。

幸运的是，我们不需要处理盘片的状态，我们只需要调用一些BIOS提供的中断就可以实现功能。要想实现从磁盘加载这个功能需要将`al`设置为`0x02`(可能还要设置其他寄存器来指定柱面、磁头、扇区等)然后使用`int 0x13`出发读磁盘中断。  

关注`0x13`中断你还可以参考这里：
```
AH = 02
AL = number of sectors to read	(1-128 dec.)
CH = track/cylinder number  (0-1023 dec., see below)
CL = sector number  (1-17 dec.)
DH = head number  (0-15 dec.)
DL = drive number (0=A:, 1=2nd floppy, 80h=drive 0, 81h=drive 1)
ES:BX = pointer to buffer


on return:
AH = status  (see INT 13,STATUS)
AL = number of sectors read
CF = 0 if successful
    = 1 if error


- BIOS disk reads should be retried at least three times and the
    controller should be reset upon error detection
- be sure ES:BX does not cross a 64K segment boundary or a
    DMA boundary error will occur
- many programming references list only floppy disk register values
- only the disk number is checked for validity
- the parameters in CX change depending on the number of cylinders;
    the track/cylinder number is a 10 bit value taken from the 2 high
    order bits of CL and the 8 bits in CH (low order 8 bits of track):

    |F|E|D|C|B|A|9|8|7|6|5-0|  CX
    | | | | | | | | | |	`-----	sector number
    | | | | | | | | `---------  high order 2 bits of track/cylinder
    `------------------------  low order 8 bits of track/cyl number

- see	INT 13,A

```

在这节课中，我们将第一次使用*进位*，它是每个寄存器上的一个额外的位，当当前操作溢出时会被置位:  

```nasm
mov ax, 0xFFFF
add ax, 1 ; ax = 0x0000 and carry = 1
```

进位是不能直接访问的，只能被其他操作符用作控制变量，比如`jc`(如果进位设置了就跳转)  

BIOS在读取磁盘完成之后还会把`al`设置为读取的扇区数，因此要将`al`与预期的扇区数进行比较以防出错。

代码
----

`boot_sect_disk.asm`里面就是用于从磁盘读取数据的完整程序。

`boot_sect_main.asm`文件里进行了一系列设置磁盘读取参数的操作，然后就`disk_load`函数来读取数据。请注意我们是如何写一些不属于引导扇区的空间的，因为大小超出了512字节。  

引导扇区实际上是硬盘0的第0个磁头的第0个柱面的第1个扇区(扇区编号从1开始)，因此，512字节之后的任何字节都对应于硬盘0的第0个磁头的第0个柱面的第2个扇区。

主程序将用样本数据填充两个扇区，然后让引导扇区的程序读取数据。

**请注意：如果你的代码一直报错，而且使用的是示例代码，那么就有可能是启动驱动器的问题，`dl`就是设置驱动器编号的寄存器**

BIOS在调用引导加载程序之前将`dl`设置为驱动器号。 然而，当从硬盘启动时，qemu可能存在一些问题。  

下面是两个解决方法：
1. 尝试使用`-fda`启动参数，就像这样：`qemu -fda boot_sect_main.bin`，设置这个参数之后qemu会把`dl`设置成`0x00`。
2. 显式使用`-boot`参数，比如这样：`qemu boot_sect_main.bin -boot c`，这样的话`dl`寄存器就会被设置成`0x80`。