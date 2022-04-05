# NetBSD 1.0

## 问题

1. **telnetd: All network ports in use**

>You are out of ptys. By default older versions of the Installer only create four of them. There could be maximum of 64 ptys (/dev/ptys0 through /dev/ptysf besides the ones you listed above). To create more of them, go to the /dev directory and (as root) type:
>```shell
>    sh MAKEDEV pty0
>```
>That'll make 12 more ptys (ptyp4 through ptypf). The default number of ptys for GENERIC configuration is 16, so it may not help even if you created more ptys by typing:
> 
>```shell
>sh MAKEDEV pty1
>```
> (which will make devices ptyq0 through ptyqf).
>If you need more than 16 ptys, you will have to compile your own kernel after having increased both the maxusers line and the pseudo-device pty line in the kernel configuration file. 
>Note: The versions 1.1e or later of the Installer do create all 16 ptys that configured into a GENERIC kernel by default. However, a bug was found in this routine that wasn't fixed until 1.1f, so please use that version or later.

2. **root login refused on this terminal.**

>修改 /etc/ttys 改成 secure

>1) Open /etc/ttys as root.

>2) Append "off secure" to the lines beginning with "tty" as follows.

>ttyp0 none network off secure

>..........

>1) Restart the telnet daemon using the following command (as root).

>```shell
># /etc/rc.d/inetd restart
>```

## 安装

在bochs上安装NetBSD1.0 学习TCP/IP详解卷二

想学习一下《TCP/IP详解 卷2：实现》，因为书里使用的是4.4BSD-Lite的代码，所以就想找一个这样的环境。能够对着实际的代码学习。上网搜索，找到一篇这样的帖子

在VMware 10.0上安装NetBSD 1.0([https://www.cnblogs.com/StupidTortoise/p/3715186.html).](https://www.cnblogs.com/StupidTortoise/p/3715186.html).)

我刚开始就是按照这个帖子的步骤，一步一步的也在VMWare上安装完成。 但是这篇教程有个问题， 就是NetBSD的不支持PCI的网卡，

这篇文章的作者，自己把NetBSDB1.3的网卡驱动移植到了 NetBSD上。而且看文章后，应该ping和ftp没有问题。其他的还不确定，

如果我也按他这个思路，后续都用虚拟机的小窗口进行操作，用vi进行代码操作，那实在是太痛苦了。

因此我最主要需求就是：

能够直接支持网络，可以通过telnet登陆到NetBSD上，用putty终端进行操作。 我需要一块ISA网卡。 所以这时候就想到了bochs

很多操作系统的实验都是在这个上面操作的。

开始如下的安装过程:

一,软盘镜像,光盘镜像 用VMware已经做好的.(具体制作过程，参考上面那个帖子)

二,使用bximage 制作了一个磁盘镜像文件。这里卡了挺长时间，因为磁盘镜像做出来以后，系统可以正常安装，但是启动的时候报错。

bochs打印的日志显示每磁道扇区数(17)不支持。最后猜想应该是NetBSD的驱动太老，bximage默认制作出来的扇区数是63.这个时候

就转去研究bximage怎么能指定扇区数，网上也没有相关信息，最后看bximage代码，发现程序里就是写死扇区63。磁头17.

这个问题最后通过fdisk解决.修改了扇区数17. 重新安装，发现磁头也不对，报磁头数7不支持。用fdisk把磁头改成7.还是不对，

又看了一下代码，磁头数需要-1(不知道什么原因).把磁头数修改成8.磁盘就做好了。

1)运行bximage 进入交互式安装，选择hd->flat 选择大小1G(1024),文件名(d.img).生成磁盘文件

2)运行fdisk d.img . 选x 然后依次修改H(磁头 修改为8) S(扇区 17) C(柱面 12000)。然后r w 修改d.img

三。重新安装，安装过程也参考那篇教程，安装过程是一样的。

四。最关键的网络部分。遇到如下问题：

1)bochs的默认配置文件，是不带网卡的。上网搜索后，NE2K是他支持的ISA网卡

2)调整NE2K的配置， 参考了bochs的文档，基本上每个模式的试了一遍。各种重启。试到slirp模式 终于能从NetBSD连到本机了。

但是还差一点，我需要能从本机连到NetBSD. 最后是使用win32模式。终于好了

ne2k: enabled=true, ioaddr=0x300, irq=10, mac=b0:c4:20:00:00:01, ethmod=win32, ethdev=\Device\NPF_{6AEE20DC-22D5-407C-80C9-14AA96130B69}

上面那个是配置。 win32需要安装wincap. irq=10如果启动的时候报冲突，就换一个，ethdev 用安装bochs 有个NIS LISTER查看，

找一个再用的网卡，比如我这里使用的是vwmare的虚拟网卡，网段是192.168.61.XX

3)配置好启动后， 需要把NetBSD里的网卡起来配置成和刚才的网段一致。ifconfig ed2 up 192.168.61.10 netmask 255.255.255.0

4)ping ftp都好使了，试试telnet。报错telnetd: All network ports in use。 查看NetBSD telnetd的代码， 发现报错的是getty

报错。他这个port是tty的port，不是网络的端口。上网查找怎么增加pty

cd /dev sh MAKEDEV pty0

5)再次telnet 发现root用户不允许登陆， 修改/etc/ttys 增加 ttyp0 none network on secure

终于可以登陆了。

6)把网卡信息写到启动脚本里 ifconfig ed2 up 192.168.61.10 netmask 255.255.255.0 把这个加到/etc/netstart里。

7)重启，出现最后一个小问题。报错。getty none无法执行。网上搜索到如下一句话，不理解

> I had to turn on this particular tty because this machine sits headless
> under like 15 other machines :) so I needed to be able to telnet to it,
You do *not* need to activate ptty entries in "/etc/ttys" to be able
to telnet into a box. Just launch "inetd". You have to mark them as
"secure" if you want to be able to login as "root".

最后是网上找到一个这样的配置 ttyp0 none network off secure 。把on改成off. 好了。这也就明白了

他上面那句话的意思了。

重新编译内核:

NetBSD-1.0的BPF支持

提示没有 /dev/bpf0 这个文件。直接创建 /dev/bpf0 这个文件后 tcpdump 命令还是提示错误：

结合网上搜索的结果，最后确定是由于内核没有添加 BPF 支持导致的。为内核添加 BPF 支持，并重新编译：

SENSERBSD 新增

/usr/src/sys/arch/i386/conf 将原始的GENERICAHA复制一份进行修改

Cp GENERICAHA SENSERBSD

pseudo-device bpfilter 4

NetBSD-1.0的ktrace

option KTRACE

重新编译内核

config SENSERBSD

cd ../compile/SENSERBSD

make depend && make 会生成 netbsd

将原始的/netbsd 备份 cp /netbsd /netbsd.std

cp netbsd /netbsd

重启