# Archlinux 安装向导

### 1、联网

```
ip link	#查看网卡设备
ip link	set wlan0 up	#启用无线网卡
rfkill list	#查看射频开关状态
rfkill unblock wifi	#启用wifi
iwctl	#进入iwd交互界面
	station wlan0 scan	#扫描周围wifi
	station wlan0 get-networks	#显示周围wifi列表
	station wlan0 connect <ssid>	#连接你的wifi,之后输入密码（不显示）
	exit	#退出iwd
dhcpcd	#给无线网卡分配ip地址
ping www.baidu.com	#测试一下网络连接
timedatectl set-ntp true	#同步网络时间，并自动更新镜像源
```

### 2、分区

此为纯linux单系统或先Linux后Windows。

若是先Windows后Linux，EFI System和Mircrosoft reserved分区则在安装Windows时会自动生成。

```
fdisk -l	#列出所有磁盘，查找你需要安装系统的盘，如/dev/sda或/dev/nvme0n1（附带数字为其分区，p1为partition1）
fdisk /dev/sda	#进入对sda分区的交互界面
	g	#创建新的GPT分区表
	p	#显示该硬盘中已有分区（目前还没有，之后随时可以看）
	n	#新建分区（EFI）
		Partition number	#默认回车 1
		First sector	#默认回车
		Last sector,+/-sectors or +/-size{K,M,G,T,P}:+512M	#选择EFI分区的大小，为避免与其他操作系统的互操作问题，大小最好不小于300M
		Do you want to remove the signature? : Y	#去除原有格式
	n	#新建分区（LVM）
		Partition number	#默认回车 2
		First sector	#默认回车
		Last sector,+/-sectors or +/-size{K,M,G,T,P}:+380G	#选择Linux分区的大小，包括了swap和主分区等，也可以默认回车使用所有空间
	w	#写入并退出
mkfs.fat -F32 /dev/sda1	#格式化EFI分区为fat32
vgcreate vg /dev/sda2	#建立卷组vg（Volume Group），可用vgdisplay查询
lvcreate -L 380G vg -n root	#建立逻辑卷root（Logical Volume）
mkfs.ext4 /dev/vg/root	#格式化主分区为ext4
```

### 3、安装系统

```
vim /etc/pacman.conf	#配置pacman，可以反注释掉color，下一行可以加上ILoveCandy，进度条会变成吃豆人
vim /etc/pacman.d/mirrorlist	#将mirrorlist里的中国服务器剪切到最上端用以加速下载
pacman -Syy	#强制更新pacman
mount /dev/vg/root /mnt	#将主分区挂载到u盘系统的mnt中
mkdir /mnt/boot	#新建文件夹boot
mount /dev/sda1 /mnt/boot	#挂载EFI引导分区到boot文件夹
pacstrap /mnt base base-devel linux linux-firmware amd-ucode vim git grub efibootmgr	os-prober lvm2	#安装linux内核及各种基础软件，intel的cpu则安装intel-ucode而不是amd-ucode
genfstab -U /mnt >> /mnt/etc/fstab	#生成fstab（File System TABle）
```

### 4、本地化

```
arch-chroot /mnt	#以u盘身份进入电脑系统
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime	#设置时区
hwclock --systohc	#同步硬件时间
vim /mnt/etc/locale.gen	#本地化设置，反注释掉en_US.UTF-8 UTF-8及zh_CN.UTF-8 UTF-8
locale-gen	#生成本地化
echo LANG=en_US.UTF-8 >> /etc/locale.conf	#设置系统语言
echo HOSTNAME >> /etc/hostname	#设置你想要的机器名称，如HOSTNAME
vim /etc/hosts	#更改host
	# Static table lookup for hostnames.
	# See hosts(5) for details.
	127.0.0.1       localhost
	::1             localhost
	127.0.0.1       HOSTNAME.localdomain	HOSTNAME
systemctl enable NetworkManager	gdm	#开机自启
vim /mnt/etc/mkinitcpio.conf	#找到HOOK行，在block于filesystem之间插入lvm2
mkinitcpio -p linux	#生成内存镜像（for LVM）
passwd	#设置root密码，输入两遍，不显示
useradd -m -g wheel USER	#新建用户USER
passwd USER	#设置USER密码
visudo #反注释 %wheel ALL=(ALL) ALL 后可以用sudo免密码提权
mkdir /boot/grub
uname -m	#查看cpu架构，一般为x86_64
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB	#安装grub引导
grub-mkconfig -o /boot/grub/grub.cfg	#生成grub配置文件
exit	#退出chroot
umount -R /mnt	#解除挂载
reboot	#重启后进入系统
```

