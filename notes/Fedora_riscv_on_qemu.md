# 在 QEMU 上运行 Fedora RISC-V

2022年9月6日 by jwy

## 环境准备

首先，下载  `qemu 7.0.0` 的源码，并进行编译：

```bash
wget https://download.qemu.org/qemu-7.0.0.tar.xz
tar -xvf qemu-7.0.0.tar.xz
cd qemu-7.0.0/
./configure --target-list=riscv64-softmmu,riscv64-linux-user
make -j

```



如果编译时提示缺少依赖，以 Ubuntu 20.04 为例，则需要安装如下的包：

```bash
sudo apt install -y build-essentials xz-utils pkg-config libglib2.0-dev libpixman-1-dev lsb-core
```



## 下载 Fedora RISC-V 镜像

从 [Index of /pub/alt/risc-v/repo/virt-builder-images/images](https://dl.fedoraproject.org/pub/alt/risc-v/repo/virt-builder-images/images/) 中下载对应的 elf 和 raw 镜像到一个新的目录，并将 raw 镜像的压缩包解压。

```bash
wget https://dl.fedoraproject.org/pub/alt/risc-v/repo/virt-builder-images/images/Fedora-Minimal-Rawhide-20200108.n.0-fw_payload-uboot-qemu-virt-smode.elf
wget https://dl.fedoraproject.org/pub/alt/risc-v/repo/virt-builder-images/images/Fedora-Minimal-Rawhide-20200108.n.0-sda.raw.xz
tar -xvf Fedora-Minimal-Rawhide-20200108.n.0-sda.raw.xz
```



## 编写启动脚本

在放置镜像的目录下编写 `start_vm.sh`脚本，调整对应的路径、文件名、CPU数量、内存大小等参数。脚本的样例如下：

```bash
#! /bin/bash

# The script is created for starting a riscv64 qemu virtual machine with specific parameters.

RESTORE=$(echo -en '\001\033[0m\002')
YELLOW=$(echo -en '\001\033[00;33m\002')
QEMUPATH="/home/osexp/qemu-7.0.0/build" # 你的 QEMU binary 位置

## Configuration
vcpu=8
memory=8
memory_append=`expr $memory \* 1024`
drive="Fedora-Minimal-Rawhide-20200108.n.0-sda.raw" # raw 磁盘镜像
fw="Fedora-Minimal-Rawhide-20200108.n.0-fw_payload-uboot-qemu-virt-smode.elf" # ELF 内核
ssh_port=10000 # ssh 端口号

cmd="$QEMUPATH/qemu-system-riscv64 \
  -bios none \
  -nographic \
  -machine virt \
  -smp "$vcpu" -m "$memory"G \
  -kernel  "$fw"\
  -object rng-random,filename=/dev/urandom,id=rng0 \
  -device virtio-rng-device,rng=rng0 \
  -device virtio-blk-device,drive=hd0 \
  -drive file="$drive",format=raw,id=hd0 \
  -device virtio-net-device,netdev=usernet \
  -netdev user,id=usernet,hostfwd=tcp::"$ssh_port"-:22 \
  -device qemu-xhci -usb -device usb-kbd -device usb-tablet \
  -append 'root=/dev/vda1 rw console=ttyS0 swiotlb=1 loglevel=3 systemd.default_timeout_start_sec=600 selinux=0 highres=off mem="$memory_append"M earlycon' "

echo ${YELLOW}:: Starting VM...${RESTORE}
echo ${YELLOW}:: Using following configuration${RESTORE}
echo ""
echo ${YELLOW}vCPU Cores: "$vcpu"${RESTORE}
echo ${YELLOW}Memory: "$memory"G${RESTORE}
echo ${YELLOW}Disk: "$drive"${RESTORE}
echo ${YELLOW}SSH Port: "$ssh_port"${RESTORE}
echo ${YELLOW}VNC Port: "$vnc_port"${RESTORE}
echo ""

sleep 2

eval $cmd

```

## 启动 QEMU 并通过 SSH 连接到虚拟机

在终端中运行 `bash start_vm.sh` 即可启动虚拟机。

```
osexp@osexp-VirtualBox:Fedora-qemu$ bash start_vm.sh
:: Starting VM...
:: Using following configuration

vCPU Cores: 8
Memory: 8G
Disk: Fedora-Minimal-Rawhide-20200108.n.0-sda.raw
SSH Port: 10000
VNC Port:


OpenSBI v0.5 (Jan  7 2020 00:00:00)
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name          : QEMU Virt Machine
Platform HART Features : RV64ACDFHIMSU
Platform Max HARTs     : 8
Current Hart           : 3
Firmware Base          : 0x80000000
Firmware Size          : 120 KB
Runtime SBI Version    : 0.2
...
...
...
Welcome to the Fedora/RISC-V disk image
https://fedoraproject.org/wiki/Architectures/RISC-V

Build date: Wed Jan  8 15:13:59 UTC 2020

Kernel 5.5.0-0.rc5.git0.1.1.riscv64.fc32.riscv64 on an riscv64 (ttyS0)

The root password is 'fedora_rocks!'.
root password logins are disabled in SSH starting Fedora 31.
User 'riscv' with password 'fedora_rocks!' in 'wheel' group is provided.

To install new packages use 'dnf install ...'

To upgrade disk image use 'dnf upgrade --best'

If DNS isn’t working, try editing ‘/etc/yum.repos.d/fedora-riscv.repo’.

For updates and latest information read:
https://fedoraproject.org/wiki/Architectures/RISC-V

Fedora/RISC-V
-------------
Koji:               http://fedora.riscv.rocks/koji/
SCM:                http://fedora.riscv.rocks:3000/
Distribution rep.:  http://fedora.riscv.rocks/repos-dist/
Koji internal rep.: http://fedora.riscv.rocks/repos/
fedora-riscv login:

```



使用`ssh -p 10000 -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -o PreferredAuthentications=password -o PubkeyAuthentication=no riscv@localhost` 登录到虚拟机中，密码为`fedora_rocks!`。由于默认 root 账户无法使用 ssh 密码登录，故而可以先使用普通账户登录后，再使用`su`切换到 root 账户，并配置 ssh 等设置。

## 关闭虚拟机

在 root 用户下执行 `poweroff` 即可关闭虚拟机，也可以使用 QEMU 的快捷键`C-a x`（先按 Ctrl+A ，再按 X），强制关闭 QEMU。

## 参考资料

https://zhuanlan.zhihu.com/p/440896294

[Architectures/RISC-V/Installing - Fedora Project Wiki](https://fedoraproject.org/wiki/Architectures/RISC-V/Installing#Boot_under_QEMU)
