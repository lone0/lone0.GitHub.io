# Build UnixBench for OpenWrt

UnixBench is a benchmark program for Unix-based systems originally from Byte magazine. A copy of the source code can be found at https://git.steamr.com/others/byte-unixbench. If you want to measure the performance of an embedded Linux system running OpenWrt or Yocto, you may need to cross-compile Unixbench on a PC and then flash the binaries onto the target board. Here are the steps to build UnixBench for OpenWrt

## Seperation of Compilation and Execution

In the original UnixBench, there is a single entry point, `./Run`. It is a script that first compiles the code and then runs the generated binaries in-place to benchmark the building host. However, this approach does not work for embedded systems like OpenWrt.

## Cross-Compilation

To build UnixBench for the target board, we don't use `./Run`. Instead, we call `make` with `CC=<path-to-the-cross-compiler>`. For example:

```
make CC=/opt/tools/OPENWRT/ubuntu20/toolchain-aarch64/bin/aarch64-openwrt-linux-gcc LD=/opt/tools/OPENWRT/ubuntu20/toolchain-aarch64/bin/aarch64-openwrt-linux-ld
```

By using CC and LD, UnixBench is built against a specific target CPU architecture instead of the x86-64 architecture of the build host.

## Runtime Execution
As mentioned earlier, `./Run` is is used to build UnixBench first, which is not suitable for an OpenWrt embedded system. The `make` instructions must be commented out as follows:

```
# preChecks
```

## Installation
The original UnixBench is not designed for installation; it runs in the directory where it is compiled. However, for OpenWrt, we need to find a location to store the generated binaries. `/usr/lib/unixbench` seems like a good choice.

## Dependencies
UnixBench relies on many Unix commands during its runtime, and these commands may or may not be included in an OpenWrt build. To ensure they are available, we must explicitly list them:

```
+perlbase-posix +perl +perlbase-time +perlbase-io +perlbase-findbin +coreutils-od
```

## Packaging
At this point, we have collected everything needed to create a new OpenWrt package for UnixBench. Here is the package and its dependencies:
```
define Package/$(PKG_NAME)
  SECTION:=utils
  CATEGORY:=Utilities
  DEPENDS:=+perlbase-posix +perl +perlbase-time +perlbase-io +perlbase-findbin +coreutils-od
  TITLE:=Unix benchmark suite
  URL:=https://github.com/kdlucas/byte-unixbench/
endef
```

To compile, add the following:
```
define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR)/UnixBench \
		$(TARGET_CONFIGURE_OPTS) \
		prefix="$(PKG_INSTALL_DIR)/usr"
endef
```

To install, add the following:
```
define Package/$(PKG_NAME)/install
	$(INSTALL_DIR) $(1)/usr/lib/unixbench
	cp -pPR $(PKG_BUILD_DIR)/UnixBench/* $(1)/usr/lib/unixbench
endef
```

Afterwards, do the menuconfig, enable `coreutils-od` under the `Utilities` menu, then enable `byte-unixbench`. Save the configuration and build the package. Once the build finishes, the following file will be generated:

```
byte-unixbench_5.1.3-1_aarch64_cortex-a53.ipk
```

## Obtaining Results
Change directory to `/usr/lib/unixbench`, and run `./Run`. After a few minutes, the results will be displayed on the screen.. In my case, an ARM64 target board scored 365.9, which is not particularly impressive.

## Wrap-up
The package we just created could be find here: https://github.com/lone0/byte-unixbench-openwrt/blob/main/package/utils/byte-unixbench/Makefile

The bench scores could be find here:
https://lone0.github.io/2023/05/11/unixbench-score-of-rpi3b.html

Enjoy.

