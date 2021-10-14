# Cosmos

我的主机环境是 macOS。在 VirtualBox 上建立两个虚拟机。一个是 Ubuntu，用来生成虚拟硬盘。另一个是 HelloOS，用来启动 HelloOS 操作系统。Ubuntu 上需要安装 openssh-server 和 virtualbox。

在 Ubuntu 中生成虚拟硬盘。

## 生产虚拟硬盘

```shell
dd bs=512 if=/dev/zero of=hd.img count=204800
# bs: 表示块大小，这里是 512 字节
# if: 表示输入文件，/dev/zero 就是 Linux 下专门返回 0 数据的设备文件，读取它就返回 0
# of: 表示输出文件，即我们的硬盘文件
# count: 表示输出多少块
```

## 格式化虚拟硬盘

```shell
# 用 losetup -l 命令查看当前回环设备，选择一个空闲的回环设备，如 /dev/loop6
sudo losetup /dev/loop0 hd.img
```

```shell
sudo mkfs.ext4 -q /dev/loop0
```

```shell
sudo mount -o loop ./hd.img ./hdisk/  # 挂载硬盘文件
sudo mkdir ./hdisk/boot/  # 建立 boot 目录
```

## 安装 GRUB

```shell
# 第一步挂载虚拟硬盘文件为 loop0 回环设备
sudo losetup /dev/loop0 hd.img
sudo mount -o loop ./hd.img ./hdisk/  # 挂载硬盘文件
# 第二步安装 GRUB
sudo grub-install --boot-directory=./hdisk/boot/ --force --allow-floppy /dev/loop0
# --boot-directory: 指向先前我们在虚拟硬盘中建立的 boot 目录
# --force --allow-floppy: 指向我们的虚拟硬盘设备文件 /dev/loop0
```

在 ./hdisk/boot/grub/grub.cfg 中写入如下内容。

```
menuentry 'HelloOS' {
    insmod part_msdos
    insmod ext2
    set root='hd0'  # 我们的硬盘只有一个分区所以是 'hd0'
    multiboot2 /boot/HelloOS.bin  # 加载 boot 目录下的 HelloOS.eki 文件
    boot  # 引导启动
}
set timeout_style=menu
if [ "${timeout}" = 0 ]; then
  set timeout=10  # 等待 10 秒钟自动启动
fi
```

## 转换虚拟硬盘格式

```shell
VBoxManage convertfromraw ./hd.img --format VDI ./hd.vdi
# convertfromraw: 指向原始格式文件
# --format VDI: 表示转换成虚拟机需要的 VDI 格式
```

生成虚拟硬盘后，把 Ubuntu 的网络设置成 Bridged Adapter，以使用 scp 命令把 hd.vdi 传回主机。scp 命令的格式如下。

```shell
# username 和 ip 替换成自己的
scp username1@ip:/home/username2/hd.vdi ~/VirtualBox\ VMs/HelloOS
```

在 macOS 上安装虚拟硬盘。

## 安装虚拟硬盘

```shell
# 第一步
# SATA 的硬盘其控制器是 intelAHCI
VBoxManage storagectl HelloOS --name "SATA" --add sata --controller IntelAhci --portcount 1
# 第二步
# 删除虚拟硬盘 UUID 并重新分配
VBoxManage closemedium disk ./hd.vdi
# 将虚拟硬盘挂到虚拟机的硬盘控制器
VBoxManage storageattach HelloOS --storagectl "SATA" --port 1 --device 0 --type hdd --medium ./hd.vdi
```

## 启动虚拟机

```shell
VBoxManage startvm HelloOS  # 启动虚拟机
```
