ubuntu 自动安装
===

启动安装 CD, 
参考: https://help.ubuntu.com/community/Installation/FromLinux#Alternate_CD

从 `ubuntu-releases` 和 `ubuntu` 镜像网站下载安装 CD 和硬盘安装文件.
安装 CD 必须放在分区根目录,
要求安装程序可识别分区文件系统, `fat32`, `ntfs`, `ext3`, `ext4` 应该都可以, lvm 未测试.
硬盘安装文件任意目录均可, 建议放到 install 目录.
硬盘安装文件会尝试在所有分区根目录查找光盘镜像.

```sh
cd /
#wget --continue 'http://ftp.cuhk.edu.hk/pub/Linux/ubuntu-releases/trusty/ubuntu-14.04.1-server-amd64.iso'
wget --continue 'http://mirrors.ustc.edu.cn/ubuntu-releases/trusty/ubuntu-14.04.1-server-amd64.iso'
mkdir /install
cd /install
#wget --continue 'http://ftp.cuhk.edu.hk/pub/Linux/ubuntu/dists/trusty/main/installer-amd64/current/images/hd-media/vmlinuz'
#wget --continue 'http://ftp.cuhk.edu.hk/pub/Linux/ubuntu/dists/trusty/main/installer-amd64/current/images/hd-media/initrd.gz'
wget --continue 'http://mirrors.ustc.edu.cn/ubuntu/dists/trusty/main/installer-amd64/current/images/hd-media/vmlinuz'
wget --continue 'http://mirrors.ustc.edu.cn/ubuntu/dists/trusty/main/installer-amd64/current/images/hd-media/initrd.gz'
```

`/etc/grub.d/40_custom` 添加如下配置:

```sh
menuentry 'hd-inst' {
        load_video
        gfxmode $linux_gfx_mode
        insmod gzio
        insmod part_msdos
        insmod ext2
        insmod loopback
        insmod lvm
        set root='hd1,msdos1'
        search --no-floppy --fs-uuid --set=root 4ed1920a-e87b-4b91-b4da-e1e4d5401578
        linux /install/vmlinuz root=/dev/ram1 ramdisk_size=1048576 rw preseed/file=/hd-media/my.seed
        initrd /install/initrd.gz
}
```

配置 `my.seed`,
参考: https://help.ubuntu.com/14.04/installation-guide/amd64/apbs04.html

```
d-i debian-installer/locale string en_US.UTF-8
# Optionally specify additional locales to be generated.
#d-i localechooser/supported-locales zh_CN.UTF-8

# Keyboard selection.
# Disable automatic (interactive) keymap detection.
d-i console-setup/ask_detect boolean false
#d-i keyboard-configuration/modelcode string pc105
d-i keyboard-configuration/layoutcode string us

# Disable network configuration entirely. This is useful for cdrom
# installations on non-networked devices where the network questions,
# warning and long timeouts are a nuisance.
#d-i netcfg/enable boolean false

# netcfg will choose an interface that has link if possible. This makes it
# skip displaying a list if there is more than one interface.
d-i netcfg/choose_interface select auto

# To pick a particular interface instead:
#d-i netcfg/choose_interface select eth1

# If you prefer to configure the network manually, uncomment this line and
# the static network configuration below.
d-i netcfg/disable_autoconfig boolean true

# If you want the preconfiguration file to work on systems both with and
# without a dhcp server, uncomment these lines and the static network
# configuration below.
#d-i netcfg/dhcp_failed note
#d-i netcfg/dhcp_options select Configure network manually

# Static network configuration.
d-i netcfg/get_nameservers string 8.8.8.8
d-i netcfg/get_ipaddress string 118.193.215.39
d-i netcfg/get_netmask string 255.255.255.128
d-i netcfg/get_gateway string 118.193.215.1
d-i netcfg/confirm_static boolean true

# Any hostname and domain names assigned from dhcp take precedence over
# values set here. However, setting the values still prevents the questions
# from being shown, even if values come from dhcp.
d-i netcfg/get_hostname string hk
d-i netcfg/get_domain string oolap.com

# Disable that annoying WEP key dialog.
d-i netcfg/wireless_wep string
# The wacky dhcp hostname that some ISPs use as a password of sorts.
#d-i netcfg/dhcp_hostname string radish

# If non-free firmware is needed for the network or other hardware, you can
# configure the installer to always try to load it, without prompting. Or
# change to false to disable asking.
d-i hw-detect/load_firmware boolean false

# Use the following settings if you wish to make use of the network-console
# component for remote installation over SSH. This only makes sense if you
# intend to perform the remainder of the installation manually.
d-i anna/choose_modules string network-console
#d-i network-console/password password r00tme
#d-i network-console/password-again password r00tme
# Use this instead if you prefer to use key-based authentication
d-i network-console/authorized_keys_url /authorized_keys

# If the system has free space you can choose to only partition that space.
# This is only honoured if partman-auto/method (below) is not set.
# Alternatives: custom, some_device, some_device_crypto, some_device_lvm.
d-i partman-auto/init_automatically_partition select biggest_free

# Grub is the default boot loader (for x86). If you want lilo installed
# instead, uncomment this:
d-i grub-installer/skip boolean true
# To also skip installing lilo, and install no bootloader, uncomment this
# too:
d-i lilo-installer/skip boolean true
```

如果硬盘安装失败, 最简单的办法是使用网络安装, 首先从镜像网站下载对于版本的网络安装镜像:

```
wget --continue 'http://mirrors.ustc.edu.cn/ubuntu/dists/precise/main/installer-amd64/current/images/netboot/ubuntu-installer/amd64/linux'
wget --continue 'http://mirrors.ustc.edu.cn/ubuntu/dists/precise/main/installer-amd64/current/images/netboot/ubuntu-installer/amd64/initrd.gz'
```

使用 grub4dos 或 grub2 加载安装镜像, 为简化操作, 可直接指定一些预设参数.
关键配置为语言、键盘、镜像、网络、主机名.

```
title ubuntu netboot
find --set-root --ignore-floppies --ignore-cd /netboot/linux
kernel /netboot/linux locale=en_US.UTF-8 console-setup/ask_detect=false keyboard-configuration/layoutcode=us mirror/protocol=http mirror/country=manual mirror/http/hostname=mirrors.aliyun.com mirror/http/directory=/ubuntu mirror/http/proxy= netcfg/choose_interface=eth1 netcfg/disable_autoconfig=true netcfg/get_ipaddress=121.42.46.16 netcfg/get_netmask=255.255.252.0 netcfg/get_gateway=121.42.47.247 netcfg/get_nameservers=8.8.8.8 netcfg/confirm_static=true netcfg/get_hostname=cn netcfg/get_domain=oolap.com 
initrd /netboot/initrd.gz
```
