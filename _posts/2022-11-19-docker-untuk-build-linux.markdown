---
layout: post
title:  "Docker untuk build Linux"
date:   2022-11-19 04:00:32 +0000
categories: tutorial
---

Beberapa kali saya gagal mem-build Yocto karena ada dependensi yang hilang setelah meng-upgrade OS di komputer.
Saya menggunakan Fedora, walaupun upgrade-nya 6 bulan sekali, tetap membuat was-was kalau nantinya
akan ada masalah saat build. Dan saat saya menulis ini Fedora 37 baru saja release dan saya kemungkinan
akan mengupgrade ke Fedora 37.

Dengan concern yang sama, komputer saya yang lain pun sudah lama sekali OS-nya tidak saya upgrade. Tetap
setia menggunakan Ubuntu 18.04 karena sangat cocok untuk build apapun, baik Yocto maupun Android.

Suatu hari rekan kerja saya bilang kalau sepertinya ada solusi untuk masalah ini. Saya pun
diminta untuk mengeksplor docker untuk building Yocto. Karena saya seringnya menge-build Yocto
untuk IMX platform, IMX6 dan IMX8, dari NXP, lalu ketemulah saya 2 repo, [ini](https://github.com/hnakayam/imx-docker-clone),
dan dari NXP sendiri [ini](https://github.com/nxp-imx/imx-docker). 

Konsepnya sangat simple, saya build Yocto di dalam docker yang menge-mount sebuah direktori di host
komputer. Jika saya ingin mengedit sesuatu misalkan mengganti parameter A, hal itu bisa saya lakukan
dari host OS, atau bisa juga langsung dari dalam docker karena ada editornya juga di sana.

Saat ini repo building dalam docker saya masih support Yocto saja. Saya belum coba untuK Android ataupun yang lain karena
storage yang terbatas hehehe. Untuk Android 11 saja, merekomendasikan punya 450 GB kosong. Walaupun repo docker saya
mirip dengan kedua repo yang saya sebutkan di atas, silakan pergi ke [sini](https://github.com/rahmanuh/build-in-docker)
kalau tertarik untuk melihat-lihat :)

Terima kasih :)
