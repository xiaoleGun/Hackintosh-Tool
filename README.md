---
title: 不用刷BIOS来解锁MSR（CFG）
date: 2020-03-02
---
今天我们来学习不刷BIOS来解锁MSR(CFG)锁，本不想写这篇文章但没啥事干还是写了，而且云屋小站的那篇文章，有可能大部分人看不懂，我尽量把这个教程写的清楚明白一点让各位看的比较爽，但配图我还是借用[云屋小站](https://www.misonsky.cn/115.html)的配图，感谢了！！！下面我们开始我们的教程吧！！！
<!--more-->

我们需要提取BIOS，DELL用户请往下看部分机型BIOS是EXE格式的也需要往下看，如果是BIN或ROM格式的你就跳过吧!(PS:部分的AMI的BIOS可以提取BIOS，可以自己确定。)我下面先讲DELL机型的
。

(注意以下操作在Windows进行)首先进入[DELL官网](https://www.dell.com/support/home/cn/zh/cnbsd1?app=products&~ck=mn)搜索自己的机型并下载EXE文件的BIOS，然后在任意盘符创建一个文件夹，并取名叫做1把EXE格式的BIOS放到这个文件夹并把这个BIOS取名叫做1，然后打开CMD以管理员来运行，输入D:/进入D盘根据你的盘符来选择，之后CD 1进入文件夹，然后在输入1 –writeromfile之后你会发现在1的文件夹中多出一个ROM格式的文件这个就是我们的BIOS了，接下来你们可以跳过了，我要讲解AMI的方法了！！！

首先在win下载一个软件Universal BIOS Backup ToolKit 2.0.exe[点我下载](https://www.misonsky.cn/storge/?/resources/无需刷BIOS！使用setup_var命令解锁MSR%200xE2锁定，修改dvmt值，开启AHCI/Universal+BIOS+Backup+ToolKit+2.0.exe)下载之后直接提取就ok了![](1.jpg)
然后你们也会得到一个rom格式的文件。![](2.jpg)

接下来所有人看这里！！！我们现在需要下载一个软件，此软件是UEFITool工具它可以读取BIOS此软件有win、mac、Linux版本，这里我以mac版进行演示，mac版[点我下载](https://github.com/LongSoft/UEFITool)

打开UEFITool软件，菜单栏选择File>Open image file…![](3.jpg)
选择你的rom格式的bios文件或bin格式然后会出现以下界面![](4.jpg)同时按下alt+F（我这里是PS2键盘，如果你是USB键盘，那就按Win+F）打开搜索框，选择Text，输入CFG Lock，点OK。![](5.jpg)
得到如下信息![](6.jpg)双击这条信息，软件会自动定位到含有CFG Lock信息的模块。![](7.jpg)
右键该模块，选择extractbody，提取出该模块。
![](8.png) ![](9.jpg)
接下来，我们需要把该模块转换成我们能读懂的信息。我们需要借助ifrextract这个工具。官网点我打开，[点我下载](https://www.misonsky.cn/storge/?/resources/无需刷BIOS！使用setup_var命令解锁MSR%200xE2锁定，修改dvmt值，开启AHCI/ifrextract_v0.3.6.osx.zip)。把上面提取出来的模块更名为1.bin（为了方便操作），把ifrextract文件放到桌面上。打开终端，依次执行以下操作：cd ~/desktop 进入桌面这个目录。转换信息 ./ifrextract 1.bin 1.txt ![](10.jpg)
这样，我们会看到桌面上生成了一个叫1.txt的文件，里面就是转换出来的我们能读懂的BIOS模块信息。![](11.jpg)
## 寻找需要的偏移量
打开上一步转换出来的1.txt文件，同时按下alt+F（这是ps2键盘的，USB键盘使用Win+F），输入我们想要修改的东西名称进行查找。例如，我们想要解锁MSR 0xE2，那就输入CFG Lock进行搜索![](12.jpg)提取整理出有用的信息：

One Of: CFG lock, VarStoreInfo(VarOffset/VarName): 0x62, VarStore: 0x1
One Of Option: Disabled, Value (8 bit): 0x0
One Of Option: Enabled, Value (8 bit): 0x1

其中，VarStoreInfo值以及Option中的Value值就是我们要找的。VarStoreInfo值是CFG Lock这个选项的地址，也可以说其在BIOS中的偏移量。后面的VarStore是当前值（BIOS默认值），为0x1      0x1对应下面的Option里就是Enabled，意思就是CFGLock这个选择默认被打开了，也就是说MSR 0xE2默认被锁定了。而我们想要解锁它就需要把该选择修改成0x0的Disabled，就是要把0x62这个地址上的数值修改为0x0
## 修改BIOS隐藏设置
要修改BIOS隐藏设置，我们需要在UEFI shell里使用setup_var命令。datasone修改了一份支持setup_var命令的grub引导，我们可以借助这个工具。项目[点我下载](https://www.misonsky.cn/storge/?/resources/无需刷BIOS！使用setup_var命令解锁MSR%200xE2锁定，修改dvmt值，开启AHCI/modGRUBShell.efi)使用磁盘工具，把U盘抹掉，方案用主引导记录，格式选择MS-DOS（FAT）然后在U盘分区里新建一个EFI文件夹，EFI文件夹里新建一个BOOT文件夹，把下载的modGRUBShell.efi更名为BOOTX64.efi，放入BOOT文件夹里面。就这样，一个支持setup_var命令的grub引导启动盘就制作完成了。进BIOS里设置第一启动项为U盘（UEFI模式启动）。启动grub后，我们可以开始使用setup_var命令了。

我们开始来解锁MSR 0xE2输入:
setup_var_3 0x62
该命令是查询0x62这个地址（偏移量）的数值，0x62是我们在上面一步中查找到的CFG-Lock选项的地址，从下图中我们可以看到该0x62地址数值为0x01，即对应Enabled。![](13.jpg)
输入:
setup_var_3 0x62 0x00
该命令是修改0x62这个地址的数值为0x00，也就是把CFG-Lock这个选项设置为Disabled。![](14.jpg)

每一个BIOS的设置选项地址（偏移量）都是不一样的，不检查BIOS设置地址而直接使用setup_var命令修改是十分危险的事情！！！！！！！！！！！！！！！！！一点要注意！！！！

## 验证结果
setup_var命令执行完毕后，我们怎么知道命令是否生效了呢？
其实不难，只要使用一些辅助软件看一下即可。
例如，CFG-Lock这个选项，我们可以在Mac里使用AppleIntelInfo驱动打印CPU相关状态信息，Hackintool工具已经集成了该驱动，所以我们可以很方便地获取相关信息。![](15.jpg)
由上图可知，CFG-Lock已经成功解锁了。那么，你就可以删掉KernelPM补丁（Haswell+）或AppleIntelCPUPM（Haswell以前的）补丁，直接使用原生电源管理了。（Tips：用以防止开启HWP后奔溃的MSR_0xE2__xcpm_idle_instant_reboot内核补丁也不再需要了哦！）


各位朋友们到了这一步我们的MSR（CFG）就已经解锁成功了！！！大家以后可以抛弃一些补丁了！！我制作的镜像马上就会发布在我的博客上！希望大家多多的支持并鼓励我！！拜拜！



# Hackintosh-Tool
