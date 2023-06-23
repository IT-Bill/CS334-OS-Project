# CS334-OS-Project

如何跑通徐佬的代码：
```bash
git clone https://github.com/NeumoNeumo/Linux-0.11.git
cd Linux-0.11

make clean
make 
make start

# 如果提示缺少qemu-i386，执行以下命令安装qemu
sudo apt-get install qemu-kvm


```

#### 如果安装过程出错，参照[Ubuntu20.04更换国内镜像源（阿里、网易163、清华、中科大） | TARDIS (midoq.github.io)](https://midoq.github.io/2022/05/30/Ubuntu20-04更换国内镜像源/) 更换镜像源





`qemu-system-i386` 是在 x86 体系结构上运行的 QEMU 系统模拟器。使用该命令，可以在运行 QEMU 的主机上模拟一个 x86 架构的计算机，从而可以运行各种操作系统和应用程序。

该命令的一些常用参数包括：

- `-m <memory-size>`：设定虚拟机的内存大小，例如 `-m 512` 表示设定为 512MB 的内存。
- `-cdrom <iso-file>`：指定启动盘，例如 `-cdrom ubuntu.iso` 表示使用 Ubuntu 安装镜像作为启动盘。
- `-boot <boot-option>`：指定启动方式，例如 `-boot d` 表示从 CD-ROM 启动。
- `-hda <disk-image>`：指定硬盘镜像文件，例如 `-hda disk.img` 表示使用 disk.img 文件作为主硬盘。
- `-net nic,model=<model>`：指定网卡模型，例如 `-net nic,model=rtl8139` 表示使用 RTL8139 网卡模型。

通过使用 `qemu-system-i386` 命令，可以为测试、开发和学习目的创建虚拟计算机，同时还可以模拟一些传统的旧型号计算机，如 IBM PC 或 386。





# Difference

###### Makefile文件

源码：

```
LDFLAGS	+= -Ttext 0 -e startup_32
CFLAGS	+= $(RAMDISK) -Iinclude
CPP	+= -Iinclude
```

修改后的代码：

```
# TODO64
ifeq ($(TARGET), x86)
	LDFLAGS += -Ttext 0 -e startup_32
	CFLAGS  += $(RAMDISK) -Iinclude
	CPP     += -Iinclude
else
	LDFLAGS += -Ttext 0 -e startup_64
	CFLAGS  += $(RAMDISK) -Iinclude
	CPP     += -Iinclude
endif
```

这段代码是一个条件语句指令，根据 `TARGET` 变量的值来进行条件判断。如果变量 `TARGET` 的值等于 "x86"，则会执行 if 后面的代码块；如果不等于 "x86"，则会执行 else 后面的代码块。

在 if 代码块中，LDFLAGS 变量的值被设置为 "-Ttext 0 -e startup_32"，这是针对32位架构的连接器选项。CFLAGS 变量被设置为 $(RAMDISK) -Iinclude，表示编译器标志，其中 $(RAMDISK) 表示链接指定的 RAM DISK；CPP 变量被设置为 -Iinclude，表示C++编译器搜索头文件路径。

在 else 代码块中，LDFLAGS 变量的值被设置为 "-Ttext 0 -e startup_64"，这是针对64位架构的连接器选项。CFLAGS 和 CPP 变量的值与 if 代码块中的值相同。

除此之外，代码中还包含 "# TODO64" 注释，表示在64位架构上需要进行进一步的调整和修改。

###### 这段代码的作用是为不同类型的架构（32位或64位）设置不同的连接器选项，以便于正常链接。这是操作系统内核开发中常用的技巧。变量的使用可以方便地在不同的架构之间进行切换，提高了代码的可维护性和可扩展性。

源码：

```
start:
	@qemu -m 16M -boot a -fda Image -hda $(HDA_IMG)

bochs-start:
	@$(BOCHS) -q -f tools/bochs/bochsrc/bochsrc-hd.bxrc	

debug:
	@qemu -m 16M -boot a -fda Image -hda $(HDA_IMG) -s -S -nographic -serial '/dev/ttyS0'

```

修改后的代码：

```
start:
ifeq ($(TARGET), x86)
	@qemu-system-i386 -m 16M -boot a -drive format=raw,file=Image,if=floppy -drive format=raw,file=hdc-0.11.img,index=0,media=disk
else
	@qemu-system-x86_64 -m 16M -boot a -drive format=raw,file=Image,if=floppy -drive format=raw,file=hdc-0.11.img,index=0,media=disk
endif

debug:
ifeq ($(TARGET), x86)
	@qemu-system-i386 -m 16M -boot a -drive format=raw,file=Image,if=floppy -drive format=raw,file=hdc-0.11.img,index=0,media=disk -s -S
else
	@qemu-system-x86_64 -m 16M -boot a -drive format=raw,file=Image,if=floppy -drive format=raw,file=hdc-0.11.img,index=0,media=disk -s -S
endif
```

这段代码是一个 Makefile 规则，用于启动虚拟机来运行操作系统内核。

在 Makefile 规则中，`start` 和 `debug` 是指令的名称，使用 `make start` 或 `make debug` 命令可以分别运行这两个指令。这些指令是用于在虚拟机中启动操作系统内核的。

指令中的 `ifeq ($(TARGET), x86)` 判断 `TARGET` 变量是否等于 `x86`，如果是，则执行 if 代码块中的 qemu-system-i386 命令来启动32位虚拟机；否则则执行 else 代码块中的 qemu-system-x86_64 命令来启动64位虚拟机。这里使用了 qemu-system-i386 命令和 qemu-system-x86_64 命令来启动相应的虚拟机，用 `-m` 标志设置虚拟机内存大小，`-boot a` 为使虚拟机从软盘启动，`-drive` 为为虚拟机添加虚拟磁盘，在这里分别添加了Image文件和hdc-0.11.img文件；`-s` 和 `-S` 命令用于在虚拟机启动时连接 GDB 调试器。

`start` 指令是用于启动正常运行的虚拟机，`debug` 指令则是用于启动虚拟机并连接 GDB 调试器来进行调试。

这段代码的作用是在不同的架构（32位或64位）下启动相应的虚拟机，并为虚拟机连接 GDB 调试器，方便操作系统内核开发者进行调试和测试。

###### 增加：

```
lldb-as:
	@lldb --local-lldbinit

lldb-src:
	@lldb --local-lldbinit tools/system
```



这段代码是一个 Makefile 规则，用于启动LLDB调试器。

指令中的 `lldb-as` 和 `lldb-src` 是指令的名称，使用 `make lldb-as` 或 `make lldb-src` 命令可以分别运行这两个指令。这些指令主要用于在开发过程中调试操作系统内核的汇编和源码。

`lldb-as` 指令中，使用 `@lldb --local-lldbinit` 命令启动 LLDB 调试器，并在启动过程中加载 `local-lldbinit` 脚本。该脚本用于初始化 LLDB 并设置操作系统内核的汇编调试环境。

`lldb-src` 指令中，使用 `@lldb --local-lldbinit tools/system` 命令启动 LLDB 调试器，并在启动过程中加载 `tools/system` 目录下的 `local-lldbinit` 脚本。该脚本用于初始化 LLDB 并设置操作系统内核的 源码调试环境。

这段代码的作用是启动 LLDB 调试器，并为开发者提供方便的调试环境，以便快速识别和修复汇编或源码层面上的问题。



## setpu.s

###### 增加：

这是一段 x86 汇编代码，代码的作用是在屏幕上打印出消息。下面是对代码的解释：

```
复制代码_start:   ; 定义程序入口函数

  ; 将 cs 的值移动到 ax 中，再将 ax 的值移动到 ds 和 es 中，这样 ds 和 es 与 cs 相同
  mov %cs,%ax
  mov %ax,%ds
  mov %ax,%es
      
  ; 打印一些消息
  mov $0x03, %ah    ; 设置 BIOS 中断功能号为 0x03，代表向标准输出设备（屏幕）写入一个字符
  xor %bh, %bh      ; 用来指定显示器的页号，这里将其清零
  int $0x10         ; 调用 BIOS 中断进行输出

  ; 设置要输出的字符串并将值传递给相应的中断控制器
  mov $29, %cx      ; 将要输出的字符数量存储在 cx 寄存器中
  mov $0x000b, %bx  ; 设置显示器页面号为 0x000B，用于彩色字符输出
  mov $msg2, %bp    ; 将要输出的字符串地址存储在 bp 指针指向的内存中
  mov $0x1301, %ax  ; 设置 BIOS 中断功能号为 0x13，代表BIOS显示字符串
  int $0x10         ; 调用 BIOS 中断进行输出
```

因此，这段代码实现了向标准输出设备输出一些消息，然后在屏幕上打印出一个特定的字符串。



这是一段 16 位 x86 汇编代码，其作用是显示一些主引导记录（MBR）的信息。下面是对代码的解释：

```
复制代码## 修改 ds（数据段）和 es（附加段）寄存器的值，使其指向 INITSEG 和 SETUPSEG。
mov $INITSEG, %ax
mov %ax, %ds
mov $SETUPSEG, %ax
mov %ax, %es

## 显示鼠标光标位置
mov $0x03, %ah
xor %bh, %bh
int $0x10
mov $11, %cx
mov $0x000c, %bx
mov $cur, %bp
mov $0x1301, %ax
int $0x10

## 显示详细信息
mov %ds:0, %ax
call print_hex
call print_nl

## 显示内存大小
mov $0x03, %ah
xor %bh, %bh
int $0x10
mov $12, %cx
mov $0x000a, %bx
mov $mem, %bp
mov $0x1301, %ax
int $0x10

## 显示详细信息
mov %ds:2, %ax
call print_hex

## 显示柱面数
mov $0x03, %ah
xor %bh, %bh
int $0x10
mov $25, %cx
mov $0x000d, %bx
mov $cyl, %bp
mov $0x1301, %ax
int $0x10

## 显示详细信息
mov %ds:0x80, %ax
call print_hex
call print_nl

## 显示磁头数
mov $0x03, %ah
xor %bh, %bh
int $0x10
mov $8, %cx
mov $0x000e, %bx
mov $head, %bp
mov $0x1301, %ax
int $0x10

## 显示详细信息
mov %ds:0x82, %ax
call print_hex
call print_nl

## 显示扇区数
mov $0x03, %ah
xor %bh, %bh
int $0x10
mov $8, %cx
mov $0x000f, %bx
mov $sect, %bp
mov $0x1301, %ax
int $0x10

## 显示详细信息
mov %ds:0x8e, %ax
call print_hex
call print_nl
```

因此，这段代码实现了读取 MBR 中一些关键数据的信息，最后通过调用 `print_hex` 和 `print_nl` 函数，将这些数据以十六进制的形式打印在屏幕上。其中，每次调用 `int $0x10` 函数时，都是通过 BIOS 中断将信息打印在屏幕上的。





###### bootsect（启动扇区）

这段代码是在启动之前将 boot sector（启动扇区）加载到内存中。下面是对代码的解释：

```
复制代码_start:
	mov $BOOTSEG, %ax    ; 将要加载 boot sector 的内存地址赋值给 DS 寄存器
	mov %ax, %ds         
	mov $INITSEG, %ax    ; 将第一个模块将要被加载到的内存地址赋值给 ES 寄存器
	mov %ax, %es         

	mov $256, %cx        ; 设置加载 boot sector 的字节数
	sub %si, %si         ; 将源地址偏移量清0
	sub %di, %di         ; 将目的地址偏移量清0
	rep movsw            ; 使用 rep 指令重复执行 movsw 命令，将数据从 boot sector 加载到内存中

	ljmp $INITSEG, $go   ; 通过长跳转跳转到初始化的模块，开始执行主程序

go:
	mov %cs, %ax         ; 初始化 CS 和 DS 寄存器
	mov %ax, %ds
	mov %ax, %es

	# 将堆栈放在 0x9ff00 这个地址
	mov %ax, %ss
	mov $0xFF00, %sp     ; 设置堆栈指针的初始值为 0xFF00
```

因此，此代码片段实现了将 boot sector 加载到内存中并准备执行代码的任务。首先使用 `mov` 命令将要加载 boot sector 的内存地址分别赋值给 DS 和 ES 寄存器。然后，使用 `rep; movsw` 命令将 boot sector 中的数据复制到目的位置。最后，使用 `ljmp` 命令通过长跳转跳转到另一个初始化模块，间接地启动程序。在这个过程中，还将堆栈指针的初始值设置为 0xFF00。













































