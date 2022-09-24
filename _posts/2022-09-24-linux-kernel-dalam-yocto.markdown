---
layout: post
title:  "Linux Kernel dalam Yocto project"
date:   2022-09-24 04:00:32 +0000
categories: tutorial
---

Bermain dengan Yocto project dan embedded Linux pasti akan berinteraksi juga dengan
Linux kernel. menurut saya, Linux kernel adalah inti dalam embedded Linux, 
tanpanya sebuah platform atau board tidak akan bisa booting.

Poky sudah membekali kita dengan Linux kernel yang sudah disiapkan dan dites 
oleh para developer Yocto Project agar kita sebagai user/developer tinggal pakai dan pilih versi 
Linux berapa yang mau kita pakai.

Jika kita lihat di `poky/meta/recipes-kernel/linux` akan ada banyak sekali resep Linux kernel yang sudah disediakan.
Ada yang standard (linux-yocto), real time (linux-yocto-rt), tiny atau super minimal kernel (linux-yocto-tiny),
dan dummy (linux-dummy) yang dipilih jika kita mau pakai external kernel yang bukan di-build di dalam Yocto build.

Poky versi kirkstone juga menyediakan 2 versi Linux kernel, versi 5.10 dan 5.15, keduanya adalah versi LTS. Secara default,
Yocto versi kirkstone akan memilih Linux versi 5.15. Namun kita boleh juga memilih versi 5.10. Nanti akan kita bahas
bagaimana cara menggantinya.

Sedikit mengenai convention nama resep dalam Yocto. Dalam Yocto, nama resep akan diikuti dengan nomor versinya. 
Sebagai contoh:
linux-yocto_5.15.bb --> PN (Package Name) = "linux-yocto", PV (Package Version) = "5.15". 
Penjelasan PN dan PV bisa dibaca di [sini](https://docs.yoctoproject.org/ref-manual/variables.html)

Setelah tutorial [menge-build Linux](https://rahmanuh.github.io/jekyll/update/2022/09/16/build-linux-dengan-yocto-project.html)
untuk qemuarm, kita akan punya Linux dengan versi kernel 5.15. Sekarang coba kita ganti menjadi 5.10.

Output Linux kernel Yocto standard:
```
root@qemuarm:~# uname -a
Linux qemuarm 5.15.62-yocto-standard #1 SMP PREEMPT Mon Aug 22 15:16:19 UTC 2022 armv7l GNU/Linux
```

Buka conf/local.conf lagi dan tambahkan deklarasi berikut di sana:
```
PREFERRED_VERSION_linux-yocto = "5.10.%"
```
Line di atas maksudnya adalah: saya pilih versi 5.10.apapun untuk linux-yocto.
Setelah selesai di bitbake, maka ketika dijalankan, Linux kita sudah jalan versi 5.10.

```
root@qemuarm:~# uname -a
Linux qemuarm 5.10.137-yocto-standard #1 SMP PREEMPT Tue Aug 23 02:15:05 UTC 2022 armv7l GNU/Linux
```

## Linux kernel modules
Secara default, kernel-modules tidak di-install ke dalam core-image-minimal. Untuk menginstall-nya tambahkan
line berikut ke dalam conf/local.conf.

```
MACHINE_ESSENTIAL_EXTRA_RRECOMMENDS += "kernel-modules"
```

Memodifikasi Linux kernel yang jalan pun sangat mudah. Sebagai contoh kita akan memasukkan modules wireless (cfg80211) ke dalam
Linux kernel kita sebagai module. Seara default, CFG80211 tidak aktif dalam Linux yocto standard. Bisa dicek 
dengan command di berikut:
```
$ cat tmp/work/qemuarm-poky-linux-gnueabi/linux-yocto/5.10.137+gitAUTOINC+e462ebf625_5666799db3-r0/linux-qemuarm-standard-build/.config | grep CFG80211
# CONFIG_CFG80211 is not set
# CFG80211 needs to be enabled for MAC80211
```

Jika kita cukup familiar dengan Linux kernel build, kita akan menggunakan `make menuconfig` untuk mengkonfigurasi Linux kernel.
Di dalam Yocto, caranya sedikit berbeda. Jalankan command menuconfig dari bitbake:
```
$ bitbake linux-yocto -c menuconfig
```
Maka akan muncul sebuah window terminal baru seperti bawah ini:
![menuconfig window](/pictures/menuconfig-window.png)

Lalu pergi ke `Networking support` -> `Wireless` -> `CFG80211` jadikan *M* (module). Setelah di save dan exit, pastikan di file .config,
module CFG80211 sudah diaktifkan sebagai module. 
```
$ cat tmp/work/qemuarm-poky-linux-gnueabi/linux-yocto/5.10.137+gitAUTOINC+e462ebf625_5666799db3-r0/linux-qemuarm-standard-build/.config | grep CFG80211
CONFIG_CFG80211=m
# CONFIG_CFG80211_DEVELOPER_WARNINGS is not set
CONFIG_CFG80211_REQUIRE_SIGNED_REGDB=y
CONFIG_CFG80211_USE_KERNEL_REGDB_KEYS=y
CONFIG_CFG80211_DEFAULT_PS=y
# CONFIG_CFG80211_DEBUGFS is not set
CONFIG_CFG80211_CRDA_SUPPORT=y
# CONFIG_CFG80211_WEXT is not set
```

Build lagi kernel dan image-nya dengan bitbake dan test pakai qemu. Kita bisa menge-load module cfg80211 dengan command `modprobe cfg80211`.

Mengganti konfigurasi Linux kernel dengan menuconfig lebih aman daripada secara manual karena dependensi-dependensi yang berhubungan
dengan suatu modul/driver seperti CFG80211 akan secara otomatis diaktifkan. Jika secara manual, ada kemungkinan dependensi-dependensi lain
tidak ikut aktif sehingga bisa mengalami compilation error atau bahkan tidak bisa booting. 

Kurang lebih begitulah yang bisa kita lakukan dengan cepat kalau ingin mengganti konfigurasi Linux kernel kita. 

Sampai jumpa lagi :)
