---
layout: post
title:  "Build Linux dengan Yocto Project"
date:   2022-09-16 03:07:32 +0000
categories: tuorial
---

Halo di tulisan ini, dan (mungkin) blog ini, saya ingin berbagi mengenai Linux. Sekaligus
sebagai tempat untuk mendokumentasikan ketertarikan saya dengan Linux.
Demi kepraktisan dan familiaritas saya akan tetap menggunakan istilah-istilah dalam Bahasa Inggris
untuk istilah-istilah yang berkaitan dengan komputer seperti build, command, booting, dan lainnya.
Ini juga berdasarkan obrolan sehari-hari dengan kolega dan teman, walaupun berbahasa Indonesia,
kami tetap menggunakan beberapa istilah dalam Bahasa Inggris :)

# Pengenalan singkat Yocto Project

Kalau dari website resminya, Yocto Project bukan sebuah Linux distro, tetapi bisa digunakan sebagai salah satu cara untuk membuat distro.
Maksudnya adalah, Yocto Project bukan distro seperti Ubuntu, Fedora, atau Arch. Dengan Yocto Project, walaupun kita bisa ambisius membuat
sebuah distro seperti Ubuntu dan lainnya. Namun, biasanya Yocto Project digunakan untuk membuat distro sendiri untuk
embedded system pada use case yang spesifik. Sebagai contoh, daripada menggunakan Raspbian yang sangat generic
untuk sebuah Raspberry Pi yang hanya bertugas mengukur suhu ruangan, dengan Yocto Project kita bisa menge-build 
sebuah Linux seminimal mungkin untuk melakukan tugas itu. Ya itu memang sebuah contoh yang sangat ekstrim,
tetapi semoga cukup memberi gambaran tentang penggunakan Yocto Project.

# Build linux sendiri dengan Yocto Project

Untuk menge-build sebuah Linux image yang sangat minimal dengan Yocto Project, kita membutuhkan hal-hal dibawah ini:
- Sebuah komputer dengan OS Ubuntu, Fedora, atau Debian. Daftarnya di [sini](https://docs.yoctoproject.org/ref-manual/system-requirements.html#supported-linux-distributions)
- 50 GB free storage
- Git > 1.8.3.1
- tar > 1.28
- python > 3.6.0
- gcc > 5.0

Ya, menge-build sebuah distro Linux, bukan cuma menge-build Linux kernel dan package-package yang diperlukan saja,
tetapi juga proses menge-build-nya ada men-download source code package yang mau di-build. Makanya butuh storage besar.

# Persiapan build komputer
Pastikan kita sudah meng-install semua development package yang diperlukan untuk menge-buiid sebuah Linux system. Yocto Project
menyediakan panduan di [sini](https://docs.yoctoproject.org/ref-manual/system-requirements.html#required-packages-for-the-build-host).
Sebagai contoh, komputer saya Fedora 36, tinggal jalankan command berikut:

```
sudo dnf install gawk make wget tar bzip2 gzip python3 unzip perl patch diffutils diffstat git \
	cpp gcc gcc-c++ glibc-devel texinfo chrpath ccache perl-Data-Dumper perl-Text-ParseWords \
	perl-Thread-Queue perl-bignum socat python3-pexpect findutils which file cpio python python3-pip \
	xz python3-GitPython python3-jinja2 SDL-devel xterm rpcgen mesa-libGL-devel perl-FindBin \
	perl-File-Compare perl-File-Copy perl-locale zstd lz4
```

Sebelum menge-build, kita harus clone dulu repository Poky. Poky adalah sebuah distro acuan yang biasa dipakai oleh berbagai vendor
untuk membuat distro mereka sendiri. Biasanya selain butuh Poky sebagai base metadata, kita juga butuh meta-openembedded.

Kalau saya, biasanya membuat sebuah folder khusus yang saya gunakan untuk menyimpan semua metadata. Sebagai contoh, saya simpan semua metadata
di folder sources.

```
$ mkdir -p wortel/sources
$ cd wortel/sources
$ git clone git://git.yoctoproject.org/poky
```

Tulisan ini, dan mungkin selanjutnya, akan menggunakan release Kirkstone. Perlu juga diperhatikan bahwa syntax metadata mulai Honister ke atas (i.e Kirkstone)
dan release yang lebih tua (i.e hardknott, zeus, sumo, ...) berbeda.
Berikut adalah daftar semua release Yocto Project, di [sini](https://docs.yoctoproject.org/migration-guides/index.html)

Checkout git branch kirkstone dari Poky:
```
$ cd poky
$ git checkout kirkstone
```

Untuk memulai menge-build, kita pergi dulu ke folder `wortel`. Lalu load open-embedded build environment agar shell kita saat ini mengenal 
command-command open-embedded seperti `bitbake`, `bitbake-layers`, dan lainnya. Sangat disarankan untuk menggunakan `bash`.
Bukan berarti shell yang lain seperti `zsh` tidak bisa, tapi dulu saya pernah punya masalah dengan `zsh` kita menge-build Poky.
```
$ cd wortel
$ source sources/poky/oe-init-build-env build
You had no conf/local.conf file. This configuration file has therefore been
created for you with some default values. You may wish to edit it to, for
example, select a different MACHINE (target hardware). See conf/local.conf
for more information as common configuration options are commented.

You had no conf/bblayers.conf file. This configuration file has therefore been
created for you with some default values. To add additional metadata layers
into your configuration please add entries to conf/bblayers.conf.

The Yocto Project has extensive documentation about OE including a reference
manual which can be found at:
    https://docs.yoctoproject.org

For more information about OpenEmbedded see the website:
    https://www.openembedded.org/


### Shell environment set up for builds. ###

You can now run 'bitbake <target>'

Common targets are:
    core-image-minimal
    core-image-full-cmdline
    core-image-sato
    core-image-weston
    meta-toolchain
    meta-ide-support

You can also run generated qemu images with a command like 'runqemu qemux86'

Other commonly useful commands are:
 - 'devtool' and 'recipetool' handle common recipe tasks
 - 'bitbake-layers' handles common layer tasks
 - 'oe-pkgdata-util' handles common target package tasks
```
Sekarang kita sudah berada di dalam folder build. Seperti yang sudah diberitahukan di atas bahwa kita tidak punya `conf/local.conf`,
kita akan membuatnya dulu.

Buka conf/local.conf dengan text editor. Sebagai contoh:
```
$ vim conf/local.conf
```

Edit variabel-variabel berikut:
```
MACHINE ??= "qemuarm" 

DL_DIR = "${TOPDIR}/../../yocto-share/downloads"
SSTATE_DIR = "${TOPDIR}/../../yocto-share/sstate-cache"
```

Untuk kesempatan ini kita akan menge-build untuk qemuarm (`MACHINE ??= "qemuarm"`) dengan tujuan mensimulasikan platform dengan prosesor Arm.
Di kesempatan yang lain kita akan coba juga menjalankan image yang sudah di-build di Raspberry Pi.

Kembali mengenai local.conf, `DL_DIR` dan `SSTATE_DIR` adalah folder untuk menyimpan semua source code yang di download dan menyimpan `sstate-cache`. 
Folder `sstate-cache` digunakan oleh `bitbake` untuk menyimpan status-status yang berhubungan dengan compilation, linking, dan lain-lain. 
`bitbake` adalah script Python yang disediakan Yocto Project sebagai `make`-nya Yocto Project. Kedua folder tersebut dapat di-share jika di komputer kita,
kita menge-build Linux image untuk berbagai versi Yocto ataupun berbagai platform/cpu. Selain menghemat storage, waktu menge-build juga dapat dipercepat.

Sebagai tambahan catatan, variabel `TOPDIR` adalah build directory. Contoh `DL_DIR` di atas menuju ke folder yocto-share yang berada dua level di atas build directory.

Sekarang kita build Linux image paling minimal yang cuma bisa booting. Daftar image yang bisa di-build ada di [sini](https://docs.yoctoproject.org/ref-manual/images.html?highlight=image)
```
$ bitbake core-image-minimal
```

Proses ini bisa sangat lama tergantung spesifikasi komputer yang dipakai, siapkan cemilan, atau biarkan proses build-nya berjalan di malam hari saat kita tidur.
Sebagai gambaran, komputer saya 8 core dan 32 GB RAM, dan sebagian package sudah di build, masih butuh waktu sekitar 1 jam.

Selanjutnya kita tes menggunakan qemu. Cukup jalankan:
```
$ runqemu qemuarm
```
Di akhir booting, akan ada login prompt untuk root seperti di bawah:
```
INIT: Entering runlevel: 5
Configuring network interfaces... ip: RTNETLINK answers: File exists
Starting syslogd/klogd: done

Poky (Yocto Project Reference Distro) 4.0.3 qemuarm /dev/ttyAMA0

qemuarm login: root
root@qemuarm:~#
```

# Kostumisasi Linux image
Sekarang kita akan coba untuk mengkostumisasi core-image-minimal dengan mengganti sysvinit dengan systemd. Selain itu
kita juga akan menginstall ssh server agar kita bisa ssh ke dalam qemuarm.

Buka lagi conf/local.conf dan tambahkan variabel-variabel di bawah di paling bawah conf/local.conf.
```
# Use systemd instead of sysvinit
DISTRO_FEATURES:append = " systemd "
DISTRO_FEATURES_BACKFILL_CONSIDERED = "sysvinit"
VIRTUAL-RUNTIME_init_manager = "systemd"
VIRTUAL-RUNTIME_initscripts = ""

EXTRA_IMAGE_FEATURES += " ssh-server-dropbear"
```

Sebagai default, Poky menggunakan `sysvinit`. Namun untuk kesempatan ini kita akan gantikan `sysvinit` dengan `systemd`. Jika `systemd`
gagal kita akan fallback ke `sysvinit`. Dengan local.conf baru, jalankan `bitbake core-image-minimal` lagi. Saya membutuhkan 48 menit. Sekali lagi,
siapkan cemilan atau bisa juga ditunggu sambil menyetrika baju. kalau sudah selesai, tes lagi pakai qemu `runqemu qemuarm`.

Booting-nya jadi agak lebih lama karena systemd punya banyak service yang harus dijalankan. Sekarang kita coba ssh ke dalam qemuarm.
Biasanya alamat IP-nya 192.168.7.2, tapi untuk lebih yakin bisa dicek dulu menggunakan `ip a`. Untuk kepentingan development,
Poky sudah mengkonfigurasikan ssh ke root tanpa password. Lain kali akan kita bahas juga soal ini. Ada fitur mengenai hal ini.

Output ssh ke qemuarm:
```
$ ssh root@192.168.7.2
root@qemuarm:~#
```

OK, kita sudah bisa menge-build minimal Linux dan menambahkan beberapa fitur tambahan. Selanjutnya kita akan coba menambahkan layer sendiri.

Dadah... :)
