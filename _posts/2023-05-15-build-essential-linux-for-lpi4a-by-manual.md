# Build essential Linux components for TH1520 SoC based Lichee Pi 4A by manual

This guide covers the steps to build an essential Linux system that is able to run on TH1520 SoC based systems, such as Lichee Pi 4A. It is recommended to do these on Ubuntu 20.04.


## Prerequisites

Assuming the building host is running Ubuntu 20.04, it is required following prerequisites are installed.

    $ sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev

## Toolchain

First of all, build the toolchain for TH1520.

    git clone https://github.com/T-head-Semi/xuantie-gnu-toolchain.git
    cd xuantie-gnu-toolchain
    ./configure --prefix=/home/lonestar/riscv/th1520/toolchain-bin/
    make linux -j16

Adjust the number behind -j for your build host.

After the toolchain is availalbe, we build the following modules that make up the rootfs: `openspi`, `uboot`, `kernel` and `busybox`.

## Kernel

    git clone https://github.com/revyos/thead-kernel.git
    CROSS_COMPILE=~/riscv/th1520/toolchain-bin/bin/riscv64-unknown-linux-gnu- ARCH=riscv make revyos_defconfig
    CROSS_COMPILE=~/riscv/th1520/toolchain-bin/bin/riscv64-unknown-linux-gnu- ARCH=riscv make 


## uboot

    git clone https://github.com/revyos/thead-u-boot.git
    CROSS_COMPILE=~/riscv/th1520/toolchain-bin/bin/riscv64-unknown-linux-gnu- ARCH=riscv make light_lpi4a_defconfig
    CROSS_COMPILE=~/riscv/th1520/toolchain-bin/bin/riscv64-unknown-linux-gnu- ARCH=riscv make


## opensbi

    git clone https://github.com/revyos/thead-opensbi.git


## Busybox



## Yocto

    git clone https://gitee.com/thead-yocto/xuantie-yocto.git -b Linux_SDK_V1.1.2
    cd xuantie-yocto
    source openembedded-core/oe-init-build-env thead-build/light-fm
    MACHINE=light-lpi4a bitbake thead-image-linux


## ext4.img


