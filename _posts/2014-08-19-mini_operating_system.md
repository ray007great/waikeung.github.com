---
layout: post
title: "“最简单“的操作系统"
description: "mini OS"
category: 操作系统
tags: [操作系统,引导扇区]
---

# 最小的"操作系统"

下学期就要学操作系统这门课，也由于离开学还有一个月，所以突发奇想，想要自己写一个简单的操作系统。就找了于渊的《一个操作系统的实现》这本书，这看的同时也学习了很多，在这里记下笔记，也算是给自己一点学下去的动力吧。


## 最简单的“操作系统“

其实这个算不上一个操作系统，它只是一段在引导扇区里面的小程序。虽然只是显示一段字符，但这段程序能够裸机运行，不依靠其他软件。

一个引导扇区是512个字节，并且以0xAA55为结束标识的扇区。下面就是这个引导扇区的源码。


    org 07C00h                  ; 告诉编译器程序加载到07C00处
        mov ax, cs
        mov ds, ax
        mov es, ax
        call DispStr          	; 调用显示字符串例程
        jmp $              		; 无限循环
    DispStr:
        mov ax, BootMessage
        mov bp, ax             	; es:bp = 串地址
        mov cx, 16            	; cx = 串长度
        mov ax, 01301h       	; ah = 13, al = 01h
        mov bx, 000Ch          	; 页号为0(bh = 0) 黑底红字 (bl = 0Ch,高亮)
        mov dl, 0
        int 10h                 ; 10h号中断
        ret
    BootMessage:  db "Hello,OS world!"
    times 510-($-$$)   db   0  	; 填充剩下的空间，使生成的二进制代码恰好为512字节
    dw 0xaa55                   ; 结束标志

### org 07C00h

org 伪指令用于指出其后的程序段或数据块存放在内存的起始地址的偏移量。编译器在汇编时对指令的地址计算是相对地址，对于引导扇区来说，要求的是绝对地址。引导扇区的程序一般都要求固定装载到内存的07C00h处。

所以`org 07C00h` 的意思就是将程序和数据装载到07C0h处。

### 寄存器

程序中的ax,cs等这些都是CPU的一些寄存器。这里所用到的寄存器都是16位的，能存放2个字节的内容。

* ax : 累加寄存器
* bx : 基址寄存器
* cx : 计数寄存器
* cs : 代码段寄存器(code segment)
* ds : 数据段寄存器(data segment)
* es : 附加寄存器器(extra segment)

### jmp $

$ 表示当前被汇编后的地址。`jmp $`表示跳到当前地址，以至于进入一个无限循环。

### int 10h

int 10h 是由BIOS对屏幕和显示器所提供的服务程序。

要在屏幕上显示一段字符，需要先设置好寄存器的值，然后调用int 10h 中断。

指定AH为13，表示调用显示字符串的功能。ES:BP = 串地址，CX=串长度，DH，DL=起始行列，BH=页号。当AL=01h时，光标会跟随显示移动，BL=0CH，表示属性，即黑底红字高亮。

关于10号中断的其他信息，参考:[INT 10中断功能](http://blog.csdn.net/yes_life/article/details/6778834)

### times 510-($-$$)

`times` 重复指令或数据。 `$$`表示该程序的初始代码段的地址，故该指令将会被执行510-($-$$)次。也就是用0来填充剩下的空间，达到510字节。

## 如何运行该“操作系统”

### 编译boot.asm

将上述代码保存为`boot.asm`,然后用`nasm`命令编译该文件。

	nasm -o boot.bin boot.asm

### 制作虚拟软盘
3.5寸 1.44M软盘结构：

* 2面、80道/面、18扇区/道、512字节/扇区
* 扇区总数=2面 * 80道/面 * 18扇区/道  =  2880扇区
* 存储容量= 512字节/扇区 * 2880扇区 =  1440 KB =1474560B
	
创建虚拟软盘镜像文件

	dd if=/dev/zero of=floppy.img bs=1474560 count=1
	dd if=/dev/zero of=floppy.img bs=512 count=2880
	dd if=/dev/zero of=floppy.img bs=1024 count=1440
这三条命令任意一条都可以建立一个虚拟的软盘镜像文件。

### 将boot.bin 写入虚拟软盘

	dd if=boot.bin of=floppy.img bs=512 count=1
	
### 创建虚拟机

用virtualBox创建一个虚拟机，将floppy.img挂载到软盘控制器下，然后设置为软盘启动。

![pic1](/assets/images/OS/pic1.png)
![pic2](/assets/images/OS/pic2.png)

启动该系统后，得出结果图：

![pic3](/assets/images/OS/pic3.png)

##本操作系统启动过程

1. 打开计算机电源；加电自检；
2. 寻找启动盘，该系统设置从软盘启动，计算机检查软盘的0面0磁道1扇区，如果发现它以0xAA55结束，则BIOS会认为它是一个引导扇区；
3. BIOS将该512字节扇区内容加载到内存地址0000:7c00；跳转到0000:7C00出将控制权交给这段引导代码。
4. 到此为止，计算机不再由BIOS中固有的程序来控制，而变成由操作系统的一部分来控制。