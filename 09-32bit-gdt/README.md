*你可能需要事先查询的概念：Global Descriptor Table（GDT）*

**目标：编写GDT**

还记得第6课的segmentation吗？通过设置segmentation寄存器结合offset获得更大的寻址能力。  

在32位模式下，segmentation的工作方式不同。 现在，offset成为GDT中的索引，GDT是由段描述符(segment descriptor，SD)组成的数组，这个描述符定义了基地地址(32位)、大小(20位)和其它一些标志，如只读、权限等。前述的基址、大小等在SD中被切分存储，详情你可以查看os-dev.pdf文件的第34页上的图或GDT的Wikipedia页面。

编写GDT的最简单方法是定义两个段，一个用于代码，另一个用于数据。 这些可能会重叠，这意味着没有内存保护，但这已经足够启动之用了，我们稍后将使用一种更高级的语言来修复这个问题。

出于规定，第一个GDT的字节必须是`0x00`，以确保程序员在管理地址时没有犯任何错误。  

此外，CPU不能直接加载GDT的地址，它需要一个称为"GDT descriptor"的元结构来描述结构，元结构包含GDT的大小描述(16b)、GDT的地址描述(32b)，最后使用`lgdt`指令加载Global Descriptor Table。  

让我们直接跳到用汇编写得GDT代码。同样，要想理解SD里面标志的意思请参考os-dev.pdf文档。  

在下一课中，我们将尝试着进入到32位保护模式！ 第8、9节的代码都将在第10节里被用到。 