*一些你可能需要提前查询的概念：交叉编译*

**配置好一个可以编译我们自己内核的环境**

一旦我们要用一种高级语言(即C语言)进行开发，您将需要一个交叉编译器。  
[为什么需要一个交叉编译器？](http://wiki.osdev.org/Why_do_I_need_a_Cross_Compiler%3F)


配置环境变量
-----------------

我们将会从源码构建binutils和gcc，而且安装到`/usr/local/i386elfgcc`文件夹内。首先我们要配置一下环境变量：
```
export PREFIX="/usr/local/i386elfgcc"
export TARGET=i386-elf
export PATH="$PREFIX/bin:$PATH"
```

安装binutils
--------

```sh
mkdir /tmp/src
cd /tmp/src
curl -O http://ftp.gnu.org/gnu/binutils/binutils-2.24.tar.gz
tar xf binutils-2.24.tar.gz
mkdir binutils-build
cd binutils-build
../binutils-2.24/configure --target=$TARGET --enable-interwork --enable-multilib --disable-nls --disable-werror --prefix=$PREFIX 2>&1 | tee configure.log
sudo make all install 2>&1 | tee make.log
```

安装gcc
---
```sh
cd /tmp/src
curl -O http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-4.9.1/gcc-4.9.1.tar.bz2
tar xf gcc-4.9.1.tar.bz2
cd gcc-4.9.1 && ./contrib/download_prerequisites && cd ..
mkdir gcc-build
cd gcc-build
../gcc-4.9.1/configure --target=$TARGET --prefix="$PREFIX" --disable-nls --disable-libssp --enable-languages=c --without-headers
make all-gcc 
make all-target-libgcc 
sudo make install-gcc 
sudo make install-target-libgcc 
```

完成之后，你应该在`/usr/local/i386elfgcc/bin`看到了刚刚安装的binutils和gcc，命令是以`i386-elf-`为前缀的，这样的目的是避免与系统级的库发生冲突。  

你可以把`$PATH`添加到`~/.bashrc`文件中，这样就可以直接使用命令了而不至于每次都要加上全路径。
从现在开始，在本教程中，我们将在进行交叉编译时使用带有前缀的gcc命令。