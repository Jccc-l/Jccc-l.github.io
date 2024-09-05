---
title: Archlinux基本安装
date: 2024-05-06 18:54:51
description: Arch Linux的基础安装过程
categories:
- Linux
tags:
- Arch Linux
- Linux
mathjax: false
katex: false
---

# Archlinux

## 基础系统安装

ArchLinux是一个流行的GNU/Linux滚动发行

### 基础配置

#### 检查是否以UEFI启动

通过检查UEFI的位数来验证启动模式

```sh
cat /sys/firmware/efi/fw_platform_size
```

如果命令结果为`64`，则系统是以`UEFI`模式引导且使用`64`位`x64 UEFI`。如果命令结果为`32`，则系统是以`UEFI`模式引导且使用`32`位`IA32 UEFI`，虽然其受支持，但引导加载程序只能使用`systemd-boot`和`GRUB`。如果文件不存在，则系统可能是以`BIOS`模式（或`CSM`模式）引导。确保系统以UEFI模式启动。

#### 禁用reflector服务

禁用reflector服务，避免将需要的镜像源删除

```sh
systemctl stop reflector
```

查看系统时间

```sh
timedatectl
```

#### 连接网络

##### 无线连接

无线连接使用iwctl进行认证连接

```sh
iwctl
device list
station wlan0 scan
station wlan0 connect [essid]
exit
```

##### 有线连接

一般只要接上网线即可直接联网

#### 测试网络

```sh
ping baidu.com
```

稍等片刻，如果能看到数据返回，即说明已经联网，`ctrl+c`终止当前命令。若无法连接，使用`ip link set [device name] up`来激活对应网卡后再重新网络连接与测试。

#### 更新系统时钟

```sh
timedatectl set-ntp true    # 将系统时间与网络事件进行同步
timedatectl status          # 查看服务状态
```

> 在Live环境中`systemd-timesyncd`默认启用，系统连接网络后，系统时间会自动同步
> 使用`timedatectl`确认系统时间准确

#### 配置pacman

```sh
vim /etc/pacman.conf
```

开启pacman输出颜色和多线程下载

```conf /etc/pacman.conf
Color
ILoveCandy

ParallelDownloads = 8
```

编辑`/etc/pacman.d/mirrorlist`文件，选择China的https镜像源放到最上

#### 分区

##### 设置分区

> 注意：下面的nvmen1等分区修改为你对应的设备分区信息

使用`fdisk`分区，`p`输出分区表，`n`创建新的分区，`t`设置分区类型，`w`保存并退出，`q`不保存退出，`h`输出帮助信息

```sh
fdisk /dev/nvme0n1
```

分为两个区：

- /dev/nvme0n1p1: 300M
- /dev/nvme0n1p2: 剩余空间

##### 格式化分区

```sh
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.btrfs /dev/nvme0n1p2
```

##### 创建子卷

```sh
mount /dev/nvme0n1p2 /mnt/Arch --mkdir
cd /mnt/Arch
btrfs subvolume create @
btrfs subvolume create @boot
btrfs subvolume create @home
btrfs subvolume create @var
btrfs subvolume create @pkg
btrfs subvolume create @log
btrfs subvolume create @swapfile
umount /mnt/Arch
```

##### 挂载目录

```sh
mount /dev/nvme0n1p2 -o compress=zstd,noatime,ssd,discard=async,space_cache=v2,subvol=@  /mnt/Arch --mkdir
mount /dev/nvme0n1p2 -o compress=zstd,noatime,ssd,discard=async,space_cache=v2,subvol=@boot  /mnt/Arch/boot --mkdir
mount /dev/nvme0n1p2 -o compress=zstd,noatime,ssd,discard=async,space_cache=v2,subvol=@home  /mnt/Arch/home --mkdir
mount /dev/nvme0n1p2 -o compress=zstd,noatime,ssd,discard=async,space_cache=v2,subvol=@var  /mnt/Arch/var --mkdir
mount /dev/nvme0n1p2 -o compress=zstd,noatime,ssd,discard=async,space_cache=v2,subvol=@log  /mnt/Arch/var/log --mkdir
mount /dev/nvme0n1p2 -o compress=zstd,noatime,ssd,discard=async,space_cache=v2,subvol=@pkg  /mnt/Arch/var/cache/pacman/pkg --mkdir
mount /dev/nvme0n1p2 -o compress=zstd,noatime,ssd,discard=async,space_cache=v2,subvol=@swapfile  /mnt/Arch/swapfile --mkdir
mount /dev/nvme0n1p1 /mnt/Arch/boot/efi --mkdir
```

### 安装系统和必要工具

安装基础系统

```sh
pacstrap -K /mnt/Arch base linux linux-firmware     # -K选项在安装软件包时在目标中初始化一个空的pacman密钥
```

如果使用linux-zen内核，则使用以下命令

```sh
pacstrap -K /mnt/Arch base linux-zen linux-zen-headers linux-firmware
```

安装必要软件

```sh
pacstrap /mnt/Arch networkmanager sudo neovim
```

### 配置系统

#### 分区信息导入

生成目录配置文件fstab：

```sh
genfstab -U /mnt/Arch >> /mnt/Arch/etc/fstab
```

检查`fstab`文件

```sh
cat /mnt/Arch/fstab
```

#### Chroot

切换根系统

```sh
arch-chroot /mnt/Arch
```

#### 配置系统时间

设置时区

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

设置硬件时间，生成/etc/adjtime

```sh
hwclock --systohc
```

#### 地区设置

设置系统语言，编辑`/etc/locale.gen`和`/etc/locale.conf`

`/etc/locale.gen`删除`en_US.UTF-8 UTF-8`前面的`#`

```conf /etc/locale.gen
en_US.UTF-8 UTF-8
```

创建`/etc/locale.conf`，设置语言变量

```conf /etc/locale.conf
LANG=en_US.UTF-8
```

生成区域设置

```sh
locale-gen
```

#### 网络配置

##### 设置主机名

创建`/etc/hostname`，设置为你想为主机取的主机名

```hostname /etc/hostname
yourhostname
```

##### 设置hosts

Hosts文件(也称为etc/ Hosts)是Windows(和其他操作系统)用于将IP地址映射到主机名或域名的文本文件。此文件充当本地计算机的本地DNS服务，它覆盖计算机通过网络连接到的DNS服务器的映射。[^1](https://www.digitalcitizen.life/etc-hosts-file-windows/#google_vignette)

编辑`/etc/hosts`，添加以下内容

```hosts /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   yourhostname.localdomain    yourhostname
```

两个文件中的`yourhostname`要保持一致

#### Initramfs

创建Initramfs

```sh
mkinitcpio -P
```

#### 启动加载器

安装grub和efi管理器等

```sh
sudo pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory /boot/efi --bootloader-id=GRUB     # 安装grub，命名为GRUB
```

有些电脑的UEFI系统需要在一个特定位置上有一个可启动的文件，才会显示你想要的启动选项。但是有时候，即使你用grub-install命令安装了GRUB引导程序，它在VisualBIOS启动顺序里也可能看不到。解决方法就是用以下命令，把GRUB安装在一个默认的启动路径上：

```sh
grub-install --target=x86_64-efi --efi-directory /boot/efi --removable 
```

另一种方法是将已安装的GRUB EFI可执行文件移动到默认/回退路径：

```sh
mv /boot/efi/grub /boot/efi/BOOT
mv /boot/efi/EFI/BOOT/grubx64.efi /boot/efi/EFI/BOOT/BOOTX64.EFI
```

[参考Arch Linux官方文档](https://wiki.archlinux.org/title/GRUB#Default/fallback_boot_path)

编辑`/etc/default/grub`

```grub /etc/default/grub
GRUB_DEFAULT=0                                      # 默认启动项，默认为0，如果安装了Windows和Archlinux双系统，则0为Archlinux，2为Windows
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=5 nowatchdog"  # 内核参数，删除quiet，将loglevel设置为5以显示更详细的日志信息，方便查看调试信息。添加nowatchdog关闭watchdog服务，加快开关机速度
GRUB_SAVEDEFAULT=false                              # 将这次选择的启动项作为下次的默认启动项，由于grub无法写入btrfs文件系统，该选项无效，因此将其关闭
GRUB_DISABLE_OS_PROBER=false                        # false为启用os-prober，使grub调用os-prober检测其他操作系统
```

生成Grub配置文件


```sh
grub-mkconfig /boot/grub/grub.cfg
```

当输出类似以下信息，则生成成功

> Found linux image: /boot/vmlinuz-linux-zen
> Found initrd image: /boot/intel-ucode.img /boot/initramfs-linux-zen.img
> Found fallback initrd image(s) in /boot:  intel-ucode.img initramfs-linux-zen-fallback.img
> Adding boot menu entry for UEFI Firmware Settings ...
> done


如果使用了`--removable`参数，但是又想自定义Bootloader的ID，可以使用efibootmgr进行重命名

首先使用efibootmgr命令找到对应的启动项

```sh
efibootmgr
```

```
BootCurrent: 0002
Timeout: 0 seconds
BootOrder: 0002,0000,0007,0008
Boot0000* Windows Boot Manager	HD(1,GPT,08b3ed34-07fe-4e11-b9cd-44509ff2c405,0x800,0x32000)/\EFI\Microsoft\Boot\bootmgfw.efi57494e444f5753000100000088000000780000004200430044004f0042004a004500430054003d007b00390064006500610038003600320063002d0035006300640064002d0034006500370030002d0061006300630031002d006600330032006200330034003400640034003700390035007d00000061000100000010000000040000007fff0400
Boot0002* UEFI OS	HD(1,GPT,b17d053f-31c1-4aed-b5d5-f00c2397f26c,0x800,0x96000)/\EFI\BOOT\BOOTX64.EFI
```

我这里的启动项是`Boot0002`，记住后面的`\EFI\BOOT\BOOTX64.EFI`，以及你的EFI分区（我的分区是/dev/nvme0n1p1）

然后执行以下命令：

```sh
efibootmgr -c -d '/dev/nvme0n1p1' -L "Arch Linux" -l \\EFI\\BOOT\\BOOTX64.EFI
```

这样就会把UEFI启动项命名为`Arch Linux`

#### 设置密码

```sh
passwd
```

#### 安装完成

通过`exit`退出chroot环境，`umount -R /mnt/Arch`取消硬盘挂载（如果显示busy，则先cd到其他目录再取消），输入`reboot`重启电脑

### 常用系统配置

#### 创建普通用户

创建一个普通用户jack并为他创建home目录

```sh
useradd -m jack
```

给jack赋予sudo提权的能力

```sh
EDITOR=nvim visudo
```

找到对应位置并添加一行

```/etc/sudoers /etc/sudoers
## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL:ALL) ALL
jack ALL=(ALL:ALL) ALL
```

`:wq`保存退出

随后第一次使用`sudo`命令时，会显示以下内容，根据提示输入密码即可

> We trust you have received the usual lecture from the local System
> Administrator. It usually boils down to these three things:
> 
>     #1) Respect the privacy of others.
>     #2) Think before you type.
>     #3) With great power comes great responsibility.
> 
> For security reasons, the password you type will not be visible.
> 
> \[sudo\] password for jack:

#### Swap File配置

关闭交换文件存储目录的写时复制（CoW）功能

```sh
chattr +C /swapfile
```

创建交换文件

```sh
btrfs filesystem mkswapfile --size 16g --uuid clear /swapfile/swapfile
```

激活交换文件

```sh
swapon /swapfile/swapfile
```

编辑fstab文件，添加一个交换文件的条目

```fstab /etc/fstab
/swapfile/swapfile none swap defaults 0 0
```

#### 启用网络

启用`NetworkManager.service`服务

```sh
sudo systemctl start NetworkManager.service
```

输入`nmtui`打开网络管理器的TUI界面，使用方向键、回车键和ESC键进行网络连接配置

设置NetworkManager开机自启

```sh
sudo systemctl enable NetworkManager.service
```

#### 启用ssh

##### 安装启用

安装`openssh`，这个软件包包含了服务端和客户端

```sh
sudo pacman -S openssh
```

启用`sshd`服务

```sh
sudo systemctl start sshd
```

##### 设置ssh免密登陆

在客户端生成ssh密钥

```sh
ssh-keygen -t rsa
```

在`/home/jack/.ssh/`（Windows的目录在C:/Users/Username/.ssh）中可以看到两个文件`id_rsa`、`id_rsa.pub`，将id_rsa.pub文件的内容复制到远程服务器的/home/jack/.ssh/authorized_keys文件中

Linux使用以下命令自动复制

```sh
ssh-copy-id jack@remote_host
```

尝试SSH连接：

```sh
ssh jack@remote_host
```

若设置正确，即可免密登陆

#### 笔记本合盖不睡眠

编辑`/etc/systemd/logind.conf`，在`Login`下方添加`HandleLidSwitch=ignore`

重启`systemd-logind`服务使配置生效

## 常用工具配置

### AUR依赖

安装AUR包需要两个重要工具

```sh
sudo pacman -S git base-devel       # 版本管理工具  基础编译包组
```

### man手册分页显示工具

man命令指定语言

> 需要安装语言包`man-pages-zh_cn`

```sh
man -L zh_CN [文档]
```

<!-- ## Minecraft服务器搭建-->

## Reference

[Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide)

[现代化的 Archlinux 安装，Btrfs、快照、休眠以及更多。](https://sspai.com/post/78916)

[Archlinux在Btrfs分区上的安装（bios篇）——yafeng](https://www.cnblogs.com/yafengabc/p/5172862.html)

[archlinux 简明指南](https://arch.icekylin.online/guide/)

[Arch Linux 安装使用教程 - ArchTutorial - Arch Linux Studio](https://archlinuxstudio.github.io/ArchLinuxTutorial/#/)
