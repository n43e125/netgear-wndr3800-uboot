# Netgear WNDR3800 U-Boot

A playground for U-Boot on the Netgear WNDR3800 router.

| | |
| --- | --- |
| **Version** | 1.1.4 |
| **Binary** | Matches the factory version, except for the build date; source code is unchanged. |

## Contents

- [Netgear WNDR3800 U-Boot](#netgear-wndr3800-u-boot)
  - [Contents](#contents)
  - [Device layout](#device-layout)
  - [Source](#source)
  - [Toolchain](#toolchain)
  - [Build](#build)
  - [Flashing](#flashing)

## Device layout

| Component | Size / location |
|:--|:--|
| NOR flash | 16 MiB |
| RAM | 128 MiB |
| U-Boot | 320 KiB at the start of flash |
| U-Boot environment (`uboot_env`) | 128 KiB, immediately after U-Boot |

See the [OpenWrt WNDR3800 device page](https://openwrt.org/toh/netgear/wndr3800) for additional device information.

## Source

The source comes from the U-Boot directory in Netgear's GPL release:
[`WNDR3800-V1.0.0.52_gpl_src/u-boot/`](https://www.downloads.netgear.com/files/GPL/WNDR3800-V1.0.0.52_gpl_src.tar.bz2.zip).
The archive contains both Linux and U-Boot source. The download link may no longer work.

## Toolchain

- GCC 3.4.4
- uClibc 0.9.28

The toolchain is built as part of the full `WNDR3800-V1.0.0.52_gpl_src/` project.
A prebuilt archive, `staging_dir_mips-gcc-3.4.4-uclibc-0.9.28.tar.xz`, is available in the `toolchains/` directory of the repository. Extract it outside this repository, for example to a sibling directory:

```text
../staging_dir_mips-gcc-3.4.4-uclibc-0.9.28
```

Add the toolchain's `bin` directory to `PATH` before building.

## Build

Run these commands from the repository root:

```sh
export BUILD_TOPDIR="$(pwd)"
export STAGING_DIR="$BUILD_TOPDIR/tmp"

make distclean
make hw29763654p16p128_config
make CROSS_COMPILE=mips-linux-

# Create a 320 KiB flash image padded with 0xFF.
IMAGE="$(basename "$PWD").bin"
head -c 327680 /dev/zero | tr '\000' '\377' > "$IMAGE"

# Copy U-Boot into the image without truncating the padded file.
dd conv=notrunc if=u-boot.bin of="$IMAGE"
```

The resulting image is named after the repository directory (for example, `netgear-wndr3800-uboot.bin`).

The repository also includes the prebuilt image, `netgear-wndr3800-uboot.bin`, and linker map, `u-boot.map` in `artifacts/` directory.

## Flashing

> **Warning:** Writing U-Boot to flash can brick the router if the image or flash address is wrong. Verify the target model and image, and keep a recovery plan before proceeding.

If a working U-Boot is available, it can be used to flash the new image. The following is an example session; it assumes the router is `192.168.1.1` and the TFTP server is `192.168.1.2`:

```text
ar7100> setenv ipaddr 192.168.1.1
ar7100> setenv serverip 192.168.1.2
ar7100> ping ${serverip}
Trying eth0
: unit 0 phy is up...RGMii 1000Mbps full duplex
#259:ag7100_set_mac_from_link
: pll reg 0x18050010: 0x11110000
: cfg_1: 0x1ff0000
: cfg_2: 0x3ff
: cfg_3: 0x8001ff
: cfg_4: 0xffff
: cfg_5: 0x2fffef
: done cfg2 0x7215 ifctl 0x40605060 miictrl 0x22
Using eth0 device
host 192.168.1.2 is alive
ar7100> tftp 0x80060000 netgear-wndr3800-uboot.bin
Trying eth0
: unit 0 phy is up...RGMii 1000Mbps full duplex
#259:ag7100_set_mac_from_link
: pll reg 0x18050010: 0x11110000
: cfg_1: 0x1ff0000
: cfg_2: 0x3ff
: cfg_3: 0x8001ff
: cfg_4: 0xffff
: cfg_5: 0x2fffef
: done cfg2 0x7215 ifctl 0x40605060 miictrl 0x22
Using eth0 device
TFTP from server 192.168.1.2; our IP address is 192.168.1.1
Filename 'netgear-wndr3800-uboot.bin'.
Load address: 0x80060000
Loading: #################################################################
done
Bytes transferred = 327680 (50000 hex)
ar7100> erase 0xbf000000 +50000
Erase Flash from 0xbf000000 to 0xbf04ffff in Bank # 1
First 0x0 last 0x4 sector size 0x10000
   4
Erased 5 sectors
ar7100> cp.b 0x80060000 0xbf000000 0x50000
Copy to Flash... write addr: bf000000
done
ar7100> reset
```
