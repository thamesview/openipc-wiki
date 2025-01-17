# OpenIPC Wiki
[Table of Content](../../README.md)

VH-NM-H33C
----------

- Manufacturer: [Vanhua](https://vanhua.en.alibaba.com/)
- Board: T31_GC2083_MINIONS_V1_1
- SoC: Ingenic T31L
- Sensor: Galaxycore GC2083
- Flash: XMC 25QH64CHIQ
- Dimensions: 38 × 53 mm (1.50 × 2.09 inch)

![](images/T31_GC2083_MINIONS_V1_1_FRONT.png)
![](images/T31_GC2083_MINIONS_V1_1_BACK.png)

### Ports

#### HD port (Horizontal motion motor)
5-pin Molex PicoBlade

#### VD port (Vertical motion motor)
5-pin Molex PicoBlade

#### IRCUT port
2-pin Molex PicoBlade

#### Ethernet port + Power
8-pin Molex PicoBlade

#### IR/White light port
5-pin Molex PicoBlade

#### Speaker port
2-pin Molex PicoBlade, 90 deg.

#### Microphone port
2-pin Molex PicoBlade

#### USB port (unpopulated)
Unpopulated USB port also has contacts for a USB Wi-Fi module.

#### MMC port (unpopulated)
Unpopulated MMC port also has contacts for an SD card slot.

#### Reset button (unpopulated)

#### UART
UART TX and RX contacts are located in the top left corner, between the microphone terminal
and the flash memory chip, next to a mounting hole. GND contact is located on the opposite
side of the module, but it is easier to connect to ground at the mounting hole.


### ipctool
```
---
chip:
vendor: Ingenic
model: T31L
board:
vendor: OpenIPC
version: 2.3.09.03
rom:
- type: nor
  block: 32K
  partitions:
    - name: boot
      size: 0x40000
      sha1: fce73c18
    - name: env
      size: 0x10000
      sha1: 53c6776e
      contains:
        - name: uboot-env
          offset: 0x0
    - name: kernel
      size: 0x200000
      sha1: 3daffccb
    - name: rootfs
      size: 0x500000
      path: /,squashfs
      sha1: 1da73e38
    - name: rootfs_data
      size: 0xb0000
      path: /overlay,jffs2,rw
      size: 8M
      ram:
      total: 64M
      media: 24M
      firmware:
      kernel: "3.10.14__isvp_swan_1.0__ (PREEMPT Sun Sep 3 20:01:59 UTC 2023)"
      toolchain: buildroot-gcc-12.2.0
      main-app: /usr/bin/majestic
      sensors:
- vendor: GalaxyCore
  model: GC2083
  control:
  bus: 0
  type: i2c
  addr: 0x6e
```
