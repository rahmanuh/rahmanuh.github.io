---
layout: post
title:  "Membuat Kostum Layer Yocto"
date:   2022-10-08 04:00:32 +0000
categories: tutorial
---

Jika kita ingin mempunyai konfigurasi yang sama setiap kali kita menge-build Yocto, 
lebih baik kita membuat sebuah kostum layer sendiri untuk menyimpan konfigurasinya
sesuai dengan kebutuhan kita.
Paling cepet memang meletakkan konfigurasi di `local.conf`, tapi tidak ada jaminan
kita akan selalu mendapat konfigurasi yang sama jika local.conf tidak disimpan 
dengan baik. 

Sebuah convention tidak tertulis, atau paling tidak yang saya pelajari selama ini, 
bahwa semua konfigurasi yang berefek global disimpan di folder conf, sedangkan yang 
lokal disimpan di recipe. Sebagai contoh, versi Linux kernel yang akan dijalankan 
disimpan di dalam `conf/machine/nama_machine`. Sedangkan package yang akan ikut 
diinstall ke dalam system disimpan di image recipe.

Sekarang kita akan buat sebuah konfigurasi distro, machine, dan image recipe yang 
nantinya akan menyimpan semua konfigurasi system seperti menggunakan systemd sebagai init manager,
versi Linux kernel yang akan dipakai, dan package yang akan diinstall ke dalam image.

Sebelumnya, buat dulu layer kita dengan `bitbake-layers create-layer <nama_layer>`, lebih lengkapnya
bisa baca di [sini](https://docs.yoctoproject.org/dev-manual/common-tasks.html#creating-a-general-layer-using-the-bitbake-layers-script)
Contohnya:
```
$ bitbake-layers create-layer ../sources/meta-wortel
NOTE: Starting bitbake server...
Add your new layer with 'bitbake-layers add-layer ../sources/meta-wortel'
```
Seperti sudah diberitahukan oleh bitbake-layers di atas, kita harus menambahkan layer kita ke dalam `conf/bblayer.conf`,
secara manual atau dengan `bitbake-layers add-layer <nama_layer>`.

Kembali ke topik kostum distro, sekarang kita tulis konfigurasi distro kita.
Sebagai contoh buat file `conf/distro/wortel.conf` untuk kostum distro kita, dan kita isi dengan
konfigurasi berikut:
```
require conf/distro/poky.conf

DISTRO = "wortel"
DISTRO_NAME = "Wortel (Wortel Distro)"
DISTRO_VERSION = "1.0.0"
DISTRO_CODENAME = "Chantenay"

# Use systemd instead of sysvinit
DISTRO_FEATURES:append = " systemd "
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = ""
```

Lalu buat machine sendiri di `conf/machine/wortelqarm.conf` dengan qemuarm sebagai referensi. Atau
copy-paste saja `poky/meta/conf/machine/qemuarm.conf` dengan beberapa modifikasi konfigurasi. 
Sebagai referensi, berikut adalah isi dari wortelqarm.conf:
```
#@TYPE: Machine
#@NAME: Wortel Qemu Arm Cortex-A15 machine
#@DESCRIPTION: Machine configuration for running an ARMv7 system on QEMU

require conf/machine/include/arm/armv7a/tune-cortexa15.inc
#require wortelqemu.inc

KERNEL_IMAGETYPE = "zImage"

UBOOT_MACHINE ?= "qemu_arm_defconfig"

SERIAL_CONSOLES ?= "115200;ttyAMA0 115200;hvc0"
SERIAL_CONSOLES_CHECK = "${SERIAL_CONSOLES}"

# For runqemu
QB_SYSTEM_NAME = "qemu-system-arm"
QB_MACHINE = "-machine virt,highmem=off"
QB_CPU = "-cpu cortex-a15"
QB_SMP = "-smp 4"
# Standard Serial console
QB_KERNEL_CMDLINE_APPEND = "vmalloc=256"
# For graphics to work we need to define the VGA device as well as the necessary USB devices
QB_GRAPHICS = "-device virtio-gpu-pci"
QB_OPT_APPEND = "-device qemu-xhci -device usb-tablet -device usb-kbd"
# Virtio Networking support
QB_TAP_OPT = "-netdev tap,id=net0,ifname=@TAP@,script=no,downscript=no"
QB_NETWORK_DEVICE = "-device virtio-net-device,netdev=net0,mac=@MAC@"
# Virtio block device
QB_ROOTFS_OPT = "-drive id=disk0,file=@ROOTFS@,if=none,format=raw -device virtio-blk-device,drive=disk0"
# Virtio serial console
QB_SERIAL_OPT = "-device virtio-serial-device -chardev null,id=virtcon -device virtconsole,chardev=virtcon"
QB_TCPSERIAL_OPT = "-device virtio-serial-device -chardev socket,id=virtcon,port=@PORT@,host=127.0.0.1 -device virtconsole,chardev=virtcon"

KMACHINE:wortelqarm = "qemuarma15"

EXTRA_IMAGEDEPENDS += "qemu-system-native qemu-helper-native:do_addto_recipe_sysroot"
PREFERRED_PROVIDER_virtual/kernel ??= "linux-stable"
MACHINE_ESSENTIAL_EXTRA_RRECOMMENDS += "kernel-modules"
IMAGE_FSTYPES += "tar.bz2 ext4"
IMAGE_CLASSES += "qemuboot"
```
 
Selanjutnya kita buat sebuah resep image. Di dalam kostum layer kita, kita buat sebuah folder `recipe-core/image`.
Dan copy-paste saja `poky/meta/recipes-core/images/core-image-minimal.bb`. Sebagai referensi
berikut adalah kostum image (wortel-image.bb) untuk distro wortel:
```
SUMMARY = "Wortel image with some features"

IMAGE_FEATURES += "ssh-server-dropbear"

IMAGE_INSTALL = "packagegroup-core-boot ${CORE_IMAGE_EXTRA_INSTALL}"

IMAGE_LINGUAS = " "

LICENSE = "MIT"

inherit core-image

IMAGE_ROOTFS_SIZE ?= "8192"
IMAGE_ROOTFS_EXTRA_SPACE:append = "${@bb.utils.contains("DISTRO_FEATURES", "systemd", " + 4096", "", d)}"
```
Di dalam resep image wortel-image.bb, saya menambahkan package ssh server agar nantinya saya bisa mengakses
Linux saya via ssh. 

Setelah semua konfigurasi baru sudah ditulis, sebelum kita build image kita (i.e wortel-image.bb), buka `conf/local.conf`
lalu ganti `DISTRO` dari poky menjadi wortel, dan ganti `MACHINE` dari qemuarm menjadi wortelqarm.
Lalu build lagi dengan `bitbake wortel-image`.
```
DISTRO="wortel"
MACHINE="wortelqarm"
```

Setelah di-build, jalankan dengan `runqemu wortelqarm serialstdio nographic` dan distro Linux kita jalan seperti berikut:

```
Wortel (Wortel Distro) 1.0.0 qemuarm ttyAMA0

qemuarm login: root
root@qemuarm:~# cat /etc/os-release
ID=wortel
NAME="Wortel (Wortel Distro)"
VERSION="1.0.0 (Chantenay)"
VERSION_ID=1.0.0
PRETTY_NAME="Wortel (Wortel Distro) 1.0.0 (Chantenay)"
DISTRO_CODENAME="Chantenay"
root@qemuarm:~#
```

Sekian dan terima kasih :)

Catatan:
- Entah kenapa ketika menjalankan qemu dengan kostum machine (wortelqarm), saya tidak bisa mengakses systemnya tanpa argumen `serialstdio`.

