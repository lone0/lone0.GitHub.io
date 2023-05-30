# Replace the u-boot and kernel for Lichee Pi 4A

*Lichee Pi 4A* is a RISC-V based single-board computer with reasonable performance. By default, it runs a customized build of Debian 11. If you want to replace the stock u-boot and kernel with your own custom versions without changing the rest of the Linux system, follow this tutorial.

## Build your own u-boot and kernel

There are several reasons why you might be interested in building your own u-boot or kernel. It could be to enable/disable certain features or make small changes to the kernel or bootloader.

To begin, you'll need a decent build machine. Typically, an x86 Linux workstation with 8GB of memory and 1TB of available storage space is sufficient. Additionally, you'll need a dedicated cross-compiler for the SoC TH1520 that powers Lichee Pi 4A.

## Build the toolchain

Before you can start building, you'll need a toolchain for the Xuantie C910 core. The upstream GCC cannot build the u-boot and kernel for TH1520 due to some extended Control and Status Register issues. However, there is a dedicated GCC from T-head-Semi that works:

```bash
git clone https://github.com/T-head-Semi/xuantie-gnu-toolchain.git
cd xuantie-gnu-toolchain
git submodule update --init
./configure --prefix=`realpath ~/riscv/xuantie-gnu-toolchain-bin/`
make linux
```

Building the toolchain will take some time. Once it's finished, you'll find the generated GCC in `~/riscv/xuantie-gnu-toolchain-bin/bin/`.

```bash
$ ~/riscv/xuantie-gnu-toolchain-bin/bin/riscv64-unknown-linux-gnu-gcc -v
Using built-in specs.
COLLECT_GCC=/home/zhenxi/riscv/xuantie-gnu-toolchain-bin/bin/riscv64-unknown-linux-gnu-gcc
COLLECT_LTO_WRAPPER=/home/zhenxi/riscv/xuantie-gnu-toolchain-bin/libexec/gcc/riscv64-unknown-linux-gnu/10.2.0/lto-wrapper
Target: riscv64-unknown-linux-gnu
Configured with: /home/zhenxi/riscv/th1520/xuantie-gnu-toolchain/riscv-gcc/configure --target=riscv64-unknown-linux-gnu CXXFLAGS='-g -O2 ' --prefix=/home/zhenxi/riscv/xuantie-gnu-toolchain-bin --with-sysroot=/home/zhenxi/riscv/xuantie-gnu-toolchain-bin/sysroot --with-system-zlib --enable-shared --enable-tls --enable-languages=c,c++,fortran --disable-libmudflap --disable-libssp --disable-libquadmath --enable-libsanitizer --disable-nls --disable-bootstrap --src=.././riscv-gcc --disable-multilib --with-abi=lp64d --with-arch=rv64imafdc 'CFLAGS_FOR_TARGET=-O2   -mcmodel=medlow' 'CXXFLAGS_FOR_TARGET=-O2   -mcmodel=medlow'
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 10.2.0 (GCC)
```

## Build the u-boot

Lichee Pi 4A requires the u-boot-spl to boot up. Currently, it can only be obtained from the following source code:

```bash
git clone https://github.com/revyos/thead-u-boot.git
CROSS_COMPILE=~/riscv/xuantie-gnu-toolchain-bin/bin/riscv64-unknown-linux-gnu
ARCH=riscv make light_lpi4a_defconfig
```

Make any necessary changes, and then build the u-boot:

```bash
CROSS_COMPILE=~/riscv/xuantie-gnu-toolchain-bin/bin/riscv64-unknown-linux-gnu- ARCH=riscv make -jN
```

Once the build is complete, the generated file will be `u-boot-with-spl.bin`. Keep this file as it will be needed soon.

## Build the kernel

Currently, Lichee Pi 4A support has not reached the mainstream Linux kernel. Therefore, we have to rely on a dedicated kernel from T-Head:

```bash
git clone https://github.com/revyos/thead-kernel
CROSS_COMPILE=~/riscv/xuantie-gnu-toolchain-bin/bin/riscv64-unknown-linux-gnu- ARCH=riscv make revyos_defconfig
CROSS_COMPILE=~/riscv/xuantie-gnu-toolchain-bin/bin/riscv64-unknown-linux-gnu- ARCH=riscv make -jN
```

After the build is finished, you'll find the generated files `./arch/riscv/boot/Image` and `./arch/riscv/boot/dts/thead/light-lpi4a.dtb`. Keep these files as they will be needed soon.

## Build OpenSBI

SBI is specific to RISC-V, and the Linux kernel requires it to boot Lichee Pi 4A. OpenSBI provides a usable SBI implementation, so let's build it:

```bash
git clone https://github.com/revyos/thead-opensbi.git
CROSS_COMPILE=~/riscv/xuantie-gnu-toolchain-bin/bin/riscv64-unknown-linux-gnu- ARCH=riscv make PLATFORM=generic
```

Save the output `./build/platform/generic/firmware/fw_dynamic.bin` for later use.

## Partition table and fastboot

Before flashing the u-boot and kernel, let's take a look at the partitioning of Lichee Pi 4A.

```
mmcblk0      179:0    0  7.3G  0 disk
|- mmcblk0p1  179:1    0    2M  0 part
|- mmcblk0p2  179:2    0  500M  0 part /boot
\- mmcblk0p3  179:3    0  6.8G  0 part /
```

The internal storage consists of an eMMC chip with three partitions. `mmcblk0p1` is a binary block storing u-boot-spl. `mmcblk0p2` holds an ext4 filesystem containing the Linux kernel and DTB files. `mmcblk0p3` is the rootfs, which we won't touch.

You can directly flash the u-boot-spl to `mmcblk0p1`. However, for the kernel, we need to create a filesystem image using a tool called `make_ext4fs`.

To flash the device, the fastboot tool is provided, which is widely used in Android phones for low-level communication such as flashing firmware.

To use fastboot, connect your device to your computer using a USB-to-TypeC cable. The computer must run Ubuntu or a similar system with the fastboot package installed via `apt-get` or equivalent. As mentioned in the Sipeed wiki:

> 2. Entering the programming mode
>
> Please note that different versions of hardware have slightly different methods to enter programming mode. Refer to the following sections.
> 
> 2.1. Beta version hardware
>
> Press and hold the BOOT button on the board, then
>
> connect the USB-C cable to power on (the other end of the cable should be connected to a PC) to enter USB programming mode.

Put the above information in mind as it will be needed soon.

## Create an ext4 image for the kernel

First, get the code of make_ext4fs and build the binary:

```bash
git clone https://git.openwrt.org/project/make_ext4fs.git
cd make_ext4fs
make
```

You will get `./make_ext4fs`. This tool packs a directory into a filesystem image.

Create a new directory, let's say `~/boot-partition/`. Remember the previously generated files? Copy the following three files to `~/boot-partition/`:

- `thead-kernel/arch/riscv/boot/Image`
- `thead-kernel/arch/riscv/boot/dts/thead/light-lpi4a.dtb`
- `thead-opensbi/build/platform/generic/firmware/fw_dynamic.bin`

Then build the ext4 image:

```bash
./make_ext4fs -l 32000000 ~/boot.ext4 ~/boot-partition/
```

The output will be `~/boot.ext4`.

## Flash everything with fastboot

Connect Lichee Pi 4A to your computer with a USB Type-C cable while pressing the BOOT button. On your computer, check if the device is recognized:

```bash
root@osboxes:~# fastboot devices
????????????    fastboot
```

Execute the following commands, with `u-boot-with-spl.bin` coming from u-boot:

```bash
root@osboxes:~# fastboot flash ram ~/u-boot-with-spl.bin
Sending 'ram' (935 KB)                             OKAY [  0.313s]
Writing 'ram'                                      OKAY [  0.007s]
Finished. Total time: 0.345s
root@osboxes:~# fastboot reboot
Rebooting                                          OKAY [  0.007s]
Finished. Total time: 1.592s
root@osboxes:~# sleep 10
root@osboxes:~# fastboot flash uboot ~/u-boot-with-spl.bin
Sending 'uboot' (935 KB)                           OKAY [  0.233s]
Writing 'uboot'                                    OKAY [  0.044s]
Finished. Total time: 0.318s
root@osboxes:~# fastboot flash boot ~/boot.ext4
Sending 'boot' (31248 KB)                          OKAY [  7.770s]
Writing 'boot'                                     OKAY [  0.690s]
Finished. Total time: 9.060s
root@osboxes:~#
```

That's it! Now reboot it and you can enjoy your fresh new Lichee Pi 4A!

The console output for your reference:

```
U-Boot SPL 2020.01-gb5768043 (May 28 2023 - 15:25:48 +0000)
FM[1] lpddr4x dualrank freq=3733 64bit dbi_off=n sdram init
ddr initialized, jump to uboot
image has no header

U-Boot 2020.01-gb5768043 (May 28 2023 - 15:25:48 +0000)

CPU:   rv64imafdcvsu
Model: T-HEAD c910 light
DRAM:  8 GiB
C910 CPU FREQ: 750MHz
AHB2_CPUSYS_HCLK FREQ: 250MHz
AHB3_CPUSYS_PCLK FREQ: 125MHz
PERISYS_AHB_HCLK FREQ: 250MHz
PERISYS_APB_PCLK FREQ: 62MHz
GMAC PLL POSTDIV FREQ: 1000MHZ
DPU0 PLL POSTDIV FREQ: 1188MHZ
DPU1 PLL POSTDIV FREQ: 1188MHZ
MMC:   sdhci@ffe7080000: 0, sd@ffe7090000: 1
Loading Environment from MMC... OK
Error reading output register
Warning: cannot get lcd-en GPIO
LCD panel cannot be found : -121
splash screen startup cost 16 ms
In:    serial
Out:   serial
Err:   serial
Saving Environment to MMC... Writing to MMC(0)... OK
Net:   ethernet@ffe7070000 (eth0) using MAC address - 9e:76:5b:1d:f9:0c
eth0: ethernet@ffe7070000ethernet@ffe7070000:0 is connected to ethernet@ffe7070000.  Reconnecting to ethernet@ffe7060000
ethernet@ffe7060000 (eth1) using MAC address - 9e:76:5b:1d:f9:0d
, eth1: ethernet@ffe7060000
Hit any key to stop autoboot:  0 
50340 bytes read in 1 ms (48 MiB/s)
16388 bytes read in 0 ms
85856 bytes read in 1 ms (81.9 MiB/s)
69232 bytes read in 1 ms (66 MiB/s)
25210880 bytes read in 78 ms (308.2 MiB/s)
## Flattened Device Tree blob at 01f00000
   Booting using the fdt blob at 0x1f00000
   Using Device Tree in place at 0000000001f00000, end 0000000001f13e6f

Starting kernel ...

[    0.000000] Linux version 5.10.113-gb4474f77378a (zhenxi/.ccache/github.revyos.thead-kernel@builder2) (riscv64-unknown-linux-gnu-gcc (GCC) 10.2.0, GNU ld (GNU Binutils) 2.35) #4 SMP PREEMPT Sun May 28 15:50:58 UTC 2023
[    0.000000] OF: fdt: Ignoring memory range 0x0 - 0x200000
[    0.000000] earlycon: uart0 at MMIO32 0x000000ffe7014000 (options '115200n8')
[    0.000000] printk: bootconsole [uart0] enabled
[    0.000000] efi: UEFI not found.
[    0.000000] Reserved memory: created CMA memory pool at 0x00000000e4000000, size 320 MiB
[    0.000000] OF: reserved mem: initialized node linux,cma, compatible id shared-dma-pool
[    0.000000] Zone ranges:
[    0.000000]   DMA32    [mem 0x0000000000200000-0x00000000ffffffff]
[    0.000000]   Normal   empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000000200000-0x000000000fffffff]
[    0.000000]   node   0: [mem 0x0000000010000000-0x00000000166fffff]
[    0.000000]   node   0: [mem 0x0000000016700000-0x0000000016ffffff]
[    0.000000]   node   0: [mem 0x0000000017000000-0x0000000018ffffff]
[    0.000000]   node   0: [mem 0x0000000019000000-0x000000001bffffff]
[    0.000000]   node   0: [mem 0x000000001c000000-0x000000001dffffff]
[    0.000000]   node   0: [mem 0x000000001e000000-0x000000001fffffff]
[    0.000000]   node   0: [mem 0x0000000020000000-0x00000000207fffff]
[    0.000000]   node   0: [mem 0x0000000020800000-0x00000000ffffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000000200000-0x00000000ffffffff]
[    0.000000] software IO TLB: mapped [mem 0x00000000e0000000-0x00000000e4000000] (64MB)
[    0.000000] SBI specification v0.3 detected
[    0.000000] SBI implementation ID=0x1 Version=0x9
[    0.000000] SBI v0.2 TIME extension detected
[    0.000000] SBI v0.2 IPI extension detected
[    0.000000] SBI v0.2 RFENCE extension detected
[    0.000000] SBI v0.2 HSM extension detected
[    0.000000] riscv: ISA extensions acdfimsuv
[    0.000000] riscv: ELF capabilities acdfimv
[    0.000000] percpu: Embedded 27 pages/cpu s73496 r8192 d28904 u110592
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 1031688
[    0.000000] Kernel command line: console=ttyS0,115200 root=PARTUUID=80a5a8e9-c744-491a-93c1-4f4194fd690a rootfstype=ext4 rootwait rw earlycon clk_ignore_unused loglevel=7 eth=9e:76:5b:1d:f9:0c rootrwoptions=rw,noatime rootrwreset=yes init=/lib/systemd/systemd
[    0.000000] Dentry cache hash table entries: 524288 (order: 10, 4194304 bytes, linear)

```
