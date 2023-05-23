---
date: "2023-02-03T01:09:43+08:00"
draft: false
title: ArchLinux安装记录（2023.2.3）
---

# 前言
对安装Linux，我已经想过写教程很久了，一直没有时间，趁过年有空趁机写一篇，时间仓促，如有差错请敬请指正

# 安装前的准备
1. 一个大于4G的，最好是USB3.0的U盘
2. 一台x86_64的电脑，且用的是主流硬件
3. 尽量不用*英伟达*显卡  

先在 https://archlinux.org/download/ 下载一个Arch镜像，也可以在各大国内镜像站下载
![Arch Download](https://s1.ax1x.com/2023/02/08/pSRWy5j.png "Arch Download")
用工具烧录到U盘，推荐使用rufus或ventoy，然后进入bios选择第一启动项（截图丢失）  
**如果使用的是UEFI，请将安全启动关闭！**  
接着会看到这样的界面：
![Live grub](https://s1.ax1x.com/2023/02/08/pSRfVL8.png "live grub")
选择第一项回车，进入Live环境，在这里我们可以维护系统，类似于WindowsPE
![Live](https://s1.ax1x.com/2023/02/08/pSRfKij.png "Live")
恭喜你！已经进入了Live环境

# 正式安装
事实上，Arch安装并不难，只需要理解每一步是做什么的，我跪在这里拆解并解释每一步是干什么的

## 连接网络
先让我们连上网络，输入以下命令
```bash
iwctl   //会进入联网模式
[iwd]# help    # 可以查看帮助
[iwd]# device list    # 列出你的无线设备名称，一般以wlan0命名
[iwd]# station wlan0 scan    # 扫描当前环境下的网络
[iwd]# station wlan0 get-networks    # 会显示你扫描到的所有网络
[iwd]# station wlan0 connect <Wifi名>
password:输入密码
[iwd]# exit    # 退出当前模式，回到安装模式
```
确认一下有没有成功连上网络
```bash
ping bilibili.com
```

## 切换镜像源
为了速战速决，我直接使用工具来自动切换镜像源各位也可以在各大镜像源寻找他们的Arch镜像，保存在 /etc/pacman.d/mirrorlist
```bash
reflector --country China --age 72 --sort rate --protocol https --save /etc/pacman.d/mirrorlist //每72小时自动切换访问最快的镜像源，只保存https
```

## 更新系统时间
```bash
timedatectl set-ntp true # 启用NTP服务
timedatectl status # 检查服务状态
```

## 磁盘分区
Arch镜像内置了fdisk和cfdisk，我使用cfdisk以帮助各位理解  
先输入lsblk验证自己电脑内的硬盘，比如我的
![硬盘](https://s1.ax1x.com/2023/02/08/pSRfeeS.png "硬盘")
其中20G的sda就是我的硬盘，当然你的可能不同
然后输入`cfdisk /dev/sda `（sda可以换成你的硬盘名）
![cfdisk](https://s1.ax1x.com/2023/02/08/pSRfAQP.png "cfdisk")
选择GPT，回车
![sda的分区](https://s1.ax1x.com/2023/02/08/pSRfEsf.png "sda的分区")
这就是你的硬盘了！现在开始分区，选择new，回车
![第一个分区](https://s1.ax1x.com/2023/02/08/pSRfFzt.png "第一个分区")
输入256m，回车，选择Type，然后修改为EFI系统分区
![EFI system](https://s1.ax1x.com/2023/02/08/pSRfmdg.png "EFI system")
重复之前的步骤，建立swap分区和root分区，完成之后如下图
![分区完成](https://s1.ax1x.com/2023/02/08/pSRfnoQ.png "分区完成")
分区完成之后选择write，回车后输入yes确认修改，然后选择quit退出（可以输入`lsblk`确认分区完毕）

## 格式化分区
将EFI分区格式化为Fat32格式
```bash
mkfs.fat -F 32 /dev/sda1
```
root分区格式化为xfs格式
```bash
mkfs.xfs -f /dev/sda3
```
当然也可以格式化为别的格式，比如ext4  
创建swap分区
```bash
mkswap /dev/sda2
```
（此处可以再lsblk查看分区情况）

## 挂载分区
```bash
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
swapon /dev/sda2 # 挂载swap
```

## 安装内核和其他软件
```bash
pacstrap /mnt linux linux-firmware linux-headers base base-devel neovim git
# 需要安装amd-ucode或intel-ucode，取决于你用的是什么牌子的处理器
```
解释一下
linux和linux-headers为系统内核和头文件
base和base-devel为系统编译打包，包括了许多常用工具
neovim为文本编辑器，vim的分支，支持更多先进功能

## 生成fstap文件
```bash
genfstap -U /mnt >> /mnt/etc/fstap
```

## Chroot进入新系统
```bash
arch-chroot /mnt
```
现在系统已经算“安装完了”，但是还不能使用，我们还得做进一步的设置

# 系统设置
在这里，我们将会进行配置本地化、引导、网络等

## 设置时区
```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc # 硬件时间同步
```
## 设置语言
```bash
nvim /etc/locale.gen
# 将以下两行取消注释
en_US.UTF8 UTF-8
zh_CN.UTF8 UTF-8
# 退出neovim
# 执行
locale-gen
```
## 设置环境变量
```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf # 可以换成你想要的语言，但可能乱码
```

## 系统命名
```bash
echo archlinux > /etc/hostname # name // archlinux可以换成你想要的名字
```

## 生成hosts文件
```bash
nvim /etc/hosts
# 在文件内输入以下内容
127.0.0.1 localhost
::1 localhost
127.0.1.1 archlinux.localdomain archlinux # 这里的archlinux是系统名字
```

## 安装必备包
```bash
pacman -S grub efibootmgr efivar networkmanager
systemctl enable NetworkManager # 启用NetworkManager
```

## 配置grub
```bash
grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

## 给root用户创建密码
```bash
passwd # 回车
New password: # 在此处输入密码，不会显示
Retype new password: # 重新输入密码，不会显示
passwd: password updated successfully ## 设置完成
```
然后
```bash
exit
shutdown now
```
关机后拔掉U盘，重新开机
输入用户名root和root密码即可进入系统

# 优化
其实这一块不一定要跟着做，这里只是为了让你的电脑更方便更安全

## 创建一个本地普通用户
```bash
useradd -m bob # 创建名叫bob的普通用户
passwd bob # 给bob设置密码
usermod -aG wheel,users,storage,power,lp,adm,optical bob # 给bob设置权限
EDITOR=nvim visudo 
# 取消注释 %wheel ALL=(ALL:ALL) ALL
%wheel ALL=(ALL:ALL) ALL
```

## 添加ArchLinuxCN存储库
该仓库是由archlinux中文社区驱动的一个非官方的软件仓库。我们使用的很多软件都需要使用这个库去下载，各大镜像源都有提供，可以挑选您喜欢的进行添加
```bash
nvim /etc/pacman.conf
# 在最后添加
[archlinuxcn] 
Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux-cn/$arch
# 顺便找到这一行，取消注释
[multilib] 
Include = /etc/pacman.d/mirrorlist
# 退出neovim
pacman -Sy archlinuxcn-keyring # 安装GPD密匙
```

**如果安装失败或安装慢，可以选择安装haveged**
```bash
pacman -Syu haveged
systemctl enable haveged
systemctl start haveged
rm -rf /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinux
pacman-key --populate archlinuxcn
pacman -Syy
```

## 安装驱动
```bash
pacman -S mesa vulkan-intell # Intel核显驱动
pacman -S mesa nvidia(-lts) nvidia-settings nvidia-dkms nvidia-utils nvidia-prime # Nvidia显卡驱动
pacman -S mesa xf86-video-amdgpu vulkan-radeon # AMD显卡驱动（不是ATI）
pacman -S alsa-utils pulseaudio pulseaudio-bluetooth cups # 声卡驱动
```

## 安装桌面
现实服务有两个主流的，一个是xorg，一个是wayland

xorg兼容性好，性能较差，wayland性能较好，兼容较差（一般情况下选择xorg问题少点）
```bash
pacman -S xorg
pacman -S ttf-dejavu ttf-droid ttf-hack ttf-font-awesome otf-font-awesome ttf-lato ttf-liberation ttf-linux-libertine ttf-opensans ttf-roboto ttf-ubuntu-font-family ttf-hannom noto-fonts noto-fonts-extra noto-fonts-emoji noto-fonts-cjk adobe-source-code-pro-fonts adobe-source-sans-fonts adobe-source-serif-fonts adobe-source-han-sans-cn-fonts adobe-source-han-sans-hk-fonts adobe-source-han-sans-tw-fonts adobe-source-han-serif-cn-fonts wqy-zenhei wqy-microhei # 一堆字体
nvim /etc/profile.d/freetype2.sh
--------------------------------------------
# 取消注释最后一句
export FREETYPE_PROPERTIES="truetype:interpreter-version=40"
pacman -S xfce xfce-goodies lightdm lightdm-gtk-greeter
systemctl enable lightdm
reboot
```
恭喜你，成功入坑Arch邪教！