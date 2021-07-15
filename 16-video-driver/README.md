*你可能需要提前查询的概念：VGA character cells, screen offset*

**目标：在屏幕上打印字符串**

这节教程的代码量有点小多，但是不要紧，我们一步一步来就好了，在最后我们将在屏幕上打印字符串。

文件`drivers/screen.h`里面定义了一些用于VGA显示的常量和对外暴露的接口，其中`void clear_screen();`函数是用来重绘屏幕的，`void kprint_at(char *message, int col, int row);`和`void kprint(char *message);`是用来向屏幕打印字符的，`kprint`的意思是："kernel print"。

文件`drivers/screen.c`开头就放了几个函数的声明，这几个函数是私有的（我们不在头文件中暴露接口外面的程序就不可能使用到我们的私有函数），这几个函数的功能是用来帮助我们实现`kprint`API。这里面还包含了我们上节教程的`get`和`set_cursor_offset()`函数。`print_char()`函数是用来直接操作VGA显存的。最后的几个函数用来辅助我们在`col`、`row`和`offset`之间进行转换的。


kprint_at
---------

调用`kprint_at`函数的时候`col`和`row`的值可以为`-1`，这表示我们将在当前光标位置打印字符串。
`kprint_at`首先声明col、row、offset三个变量，然后它遍历`char*`并调用`print_char()`进行打印直到字符串的结束。
注意，`print_char`本身返回下一个游标位置的偏移量，我们将在下一个循环中重用它。`kprint`函数基本上是`kprint_at`函数的封装。


print_char
----------

像`kprint_at`函数一样，`print_char`也允许cols/rows为`-1`，在这种情况下，它将调用`ports.c`里面的函数从I/O Port中获得cursor的位置，`print_char`还会换行，在这种情况下，我们将光标偏移量定位到下一行的第0列。
请记住，VGA单元格使用两个字节，一个用于字符本身，另一个用于颜色。


kernel.c
--------

我们的新内核终于能够打印字符串了！
它能进行字符定位、跨行打印，最后还会尝试在屏幕边界之外写入，但是会出现异常：在右下角打印一个红色的`E`。
在下一课中，我们将学习如何滚动屏幕。