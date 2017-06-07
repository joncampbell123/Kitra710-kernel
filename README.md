# Linux Kernel for ARTIK710/KITRA710
## Contents
1. [Introduction](#1-introduction)
2. [Build guide](#2-build-guide)
3. [Update guide](#3-update-guide)

## 1. Introduction
This is the branch for kernel 4.4, for 4.1 check the master branch.

---
## 2. Build guide
### 2.1 Install cross compiler
```
sudo apt-get install gcc-aarch64-linux-gnu
```
If you can't install the above toolchain, you can use linaro toolchain.
```
wget https://releases.linaro.org/components/toolchain/binaries/5.3-2016.05/aarch64-linux-gnu/gcc-linaro-5.3.1-2016.05-x86_64_aarch64-linux-gnu.tar.xz
tar xf gcc-linaro-5.3.1-2016.05-x86_64_aarch64-linux-gnu.tar.xz
export PATH=~/gcc-linaro-5.3.1-2016.05-x86_64_aarch64-linux-gnu:$PATH
```
You can the path permernently through adding it into ~/.bashrc

### 2.2 Install android-fs-tools
To generate modules.img which contains kernel modules, you can use the make_ext4fs.
```
sudo apt-get install android-tools-fsutils
```

### 2.2 Build the kernel

```
make ARCH=arm64 kitra710_defconfig
```
If you want to change kernel configurations,
```
make ARCH=arm64 menuconfig
```

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image -j4
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- dtbs
mkdir usr/modules
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- modules_install INSTALL_MOD_PATH=usr/modules INSTALL_MOD_STRIP=1
make_ext4fs -b 4096 -L modules \
	-l 32M usr/modules.img \
	usr/modules/lib/modules/
rm -rf usr/modules
```


## 3. Update Guide
Copy compiled binaries into your board.

```
scp arch/arm64/boot/Image root@{YOUR_BOARD_IP}:/root
scp arch/arm64/boot/dts/nexell/*.dtb root@{YOUR_BOARD_IP}:/root
scp usr/modules.img root@{YOUR_BOARD_IP}:/root
```

+ On your board
```
mount -o remount,rw /boot
cp /root/Image /boot
cp /root/*.dtb /boot
dd if=/root/modules.img of=/dev/mmcblk0p2
sync
reboot
```
