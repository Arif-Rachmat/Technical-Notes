# Technical Notes #0 — Fundamental Linux

> Catatan Dasar dan Fundamental mengenai linux secara umum sebelum masuk ke Embedded Linux, GPIO, I2C, Device Tree, driver, dan topik sistem lainnya.

---

## 1. Pendahuluan

Linux sering kali dianggap sebagai sebuah sistem operasi seperti Windows atau macOS. **Linux adalah sebuah kernel.** Kernel mengatur interaksi antara software dan hardware. Yang biasanya disebut sebagai sistem Linux umumnya terdiri dari kernel beserta banyak komponen lain seperti program sistem, library, shell, init system, package manager, dan aplikasi. Karena itu, istilah **Linux distribution (distro)** lebih tepat digunakan untuk menyebut keseluruhan sistem yang kita instal dan gunakan.

### 1.1 Kernel VS OS

#### Kernel

Kernel adalah software inti yang berjalan dengan privilege tinggi dan bertanggung jawab menyediakan layanan dasar sistem.

Beberapa tugas utama kernel:

- mengatur CPU dan scheduling

- mengelola memory

- mengelola process dan thread

- menyediakan filesystem

- mengatur network stack

- menangani interrupt

- mengelola device dan driver

- menyediakan interface untuk program melalui system call

- menerapkan mekanisme protection dan isolation

---

#### Operating System

Istilah **operating system (OS)** biasanya digunakan untuk menyebut keseluruhan software yang membuat komputer dapat digunakan.

![](/home/arif/Documents/markdown/Technical-Notes/Embedded-Linux/resource/fundamental-2.jpg)

Kernel adalah **inti OS**, tetapi OS bukan hanya kernel.

Sebuah **distribution** menggabungkan kernel dengan software lain sehingga menghasilkan sistem yang usable.

---

### 1.2 Linux Distribution

Sebuah distro pada dasarnya adalah sekumpulan komponen yang disusun menjadi satu sistem.

![](/home/arif/Documents/markdown/Technical-Notes/Embedded-Linux/resource/fundamental-3.jpg)

Karena komponennya modular, dua distro dapat menggunakan kernel Linux yang sama tetapi mempunyai Environment, package manager, konfigurasi, dan software yang sangat berbeda.

Contoh:

| Distribution       | Package Manager                   | Umum digunakan untuk             |
| ------------------ | --------------------------------- | -------------------------------- |
| Debian             | `apt`                             | Server, general-purpose          |
| Ubuntu             | `apt`                             | Desktop, server                  |
| Arch               | `pacman`                          | General-purpose, rolling release |
| Fedora             | `dnf`                             | Desktop, server                  |
| OpenWrt            | `opkg`/`apk` pada varian tertentu | Embedded/network appliance       |
| Yocto-based system | bergantung image/build system     | Embedded product                 |

---

### 1.3 User Space dan Kernel Space

Konsep fundamental dalam Linux dan Operating System pada umumnya. Konsep ini memisahkan memori virtual dalam sistem operasi menjadi dua wilayah dengan hak akses berbeda, di mana **kernel space** berfungsi sebagai area khusus untuk menjalankan inti sistem operasi dan mengendalikan perangkat keras secara langsung dengan hak istimewa tertinggi, sedangkan **user space** adalah area terbatas tempat aplikasi pengguna berjalan dengan aman tanpa risiko merusak stabilitas sistem secara keseluruhan jika terjadi kegagalan (*crash*). Hal ini tidak hanya diberlakukan secara software, CPU modern memiliki mekanisme privilege, dimana sistem menggunakan privilege tersebut untuk memisahkan software biasa dari komponen yang mempunyai akses penuh terhadap hardware dan resource sistem.

![](/home/arif/Documents/markdown/Technical-Notes/Embedded-Linux/resource/fundamental-4.jpg)

#### User Space

Program biasa dijalankan di user space.

```text
bash
python
gcc
vim
ssh
ls
cat
my_application
```

Program tersebut **tidak seharusnya mempunyai akses langsung terhadap seluruh hardware atau memory sistem**.

#### Kernel Space

Kernel berjalan dengan privilege yang jauh lebih tinggi.

Kernel dapat:

- mengakses device

- mengelola page table

- menangani interrupt

- mengatur process

- mengakses physical memory

- berkomunikasi dengan hardware melalui driver

Pemisahan ini ditujukan untuk Proteksi, isolasi, dan stabilitas sistem.

---

### 1.4 System Call (SYSCALL)

Program user-space tidak biasanya mengakses hardware secara langsung

```text
Application → GPIO register
```

Melainkan ia harus berbicara terhadap kernel melalui **system call (syscall)**.

System call adalah interface yang memungkinkan program user-space meminta layanan dari kernel. Linux mendokumentasikan berbagai interface user-space seperti system call dan `ioctl`.

Contoh operasi yang melibatkan kernel:

```c
open();
read();
write();
close();
mmap();
fork();
execve();
ioctl();
```

Contoh sederhana:

```c
int fd = open("/dev/example", O_RDWR);
```

Bila diuraikan prosesnya menjadi

![](/home/arif/Documents/markdown/Technical-Notes/Embedded-Linux/resource/fundamental-5.jpg)

---

## 2. Filosofi Unix/Linux

### 2.1. Do One Thing and Do It Well

Setiap program sebaiknya memiliki tanggung jawab yang jelas dan melakukan satu pekerjaan dengan baik.

```text
grep  → mencari pola
sort  → mengurutkan
wc    → menghitung
```

Daripada membuat satu program yang melakukan semuanya, fungsi dapat dipisahkan menjadi beberapa tool.

### 2.2 Make Programs Work Together

Program sebaiknya dapat **digabungkan** untuk menyelesaikan pekerjaan yang lebih kompleks.

```bash
cat log.txt | grep ERROR | sort
```

Setiap program hanya mengerjakan bagiannya, kemudian hasilnya diteruskan ke program berikutnya. 

### 2.3 Silence Is Golden

Program sebaiknya hanya menampilkan output yang memang diperlukan, terutama ketika operasi berhasil.

```text
Berhasil  → tidak perlu banyak output
Gagal     → tampilkan error yang relevan
```

Hal ini membuat output lebih mudah dibaca dan lebih mudah digunakan dalam script atau pipeline.

### 2.4 User Empowerment

Sistem pada dasarnya menganggap bahwa pengguna memahami apa yang sedang dilakukan dan memberikan **power serta kebebasan** sebesar mungkin, tanpa hambatan tersembunyi atau pembatasan yang dibuat-buat.

Pengguna dapat:

- mengakses sistem

- mengubah konfigurasi

- mengotomasi pekerjaan

- menggabungkan tools

- membangun software sendiri

---

## 3. File dalam Linux

### 3.1 "Everything is a File"

Salah satu slogan Unix/Linux yang paling terkenal adalah:

> "Everything is a file."

Maksudnya adalah Unix memberikan "*abstraction interface*" yang seragam untuk banyak resource.

```text
Regular file       → read/write
Device             → read/write/ioctl
Pipe               → read/write
Socket             → read/write
Terminal           → read/write
```

Karena banyak resource dapat direpresentasikan melalui `file descriptor`, programmer dapat menggunakan konsep yang serupa untuk berinteraksi dengan resource tersebut.

Namun Linux memiliki API khusus ketika model file biasa tidak cukup. Contohnya:

```text
read()
write()
ioctl()
mmap()
poll()
epoll()
socket()
```

Jadi slogan yang lebih tepat adalah:

> **Linux menggunakan file dan file descriptor sebagai abstraction penting untuk banyak jenis resource.**

---

### 3.2 Filesystem

Filesystem bukan sekadar tempat menyimpan file.

Linux menggunakan filesystem sebagai salah satu abstraction utama untuk mengorganisasi storage dan resource.![](/home/arif/Documents/markdown/Technical-Notes/Embedded-Linux/resource/fundamental-6.png)

Filesystem Hierarchy Standard (FHS) mendefinisikan konvensi penempatan file dan directory pada sistem Unix-like, walaupun implementasi modern dapat memiliki variasi tertentu.

| Path    | Fungsi umum                           |
| ------- | ------------------------------------- |
| `/`     | Root filesystem                       |
| `/home` | File user                             |
| `/etc`  | Konfigurasi sistem                    |
| `/usr`  | Program dan resource sistem           |
| `/var`  | Data yang berubah, log, cache         |
| `/tmp`  | Temporary files                       |
| `/dev`  | Device nodes                          |
| `/proc` | Interface informasi kernel/process    |
| `/sys`  | Interface kernel/device melalui sysfs |
| `/boot` | File yang diperlukan saat boot        |

---

### 3.3 `/dev`, `/proc`, dan `/sys`

Ketiga directory ini sangat penting dalam Embedded Linux.

#### `/dev`

Berisi **device nodes** yang menjadi interface user-space terhadap banyak device.

```text
/dev/ttyUSB0
/dev/i2c-1
/dev/spidev0.0
/dev/null
/dev/urandom
```

contoh mengakses I2c

![](/home/arif/Documents/markdown/Technical-Notes/Embedded-Linux/resource/fundamental-7.png)

#### `/proc`

`procfs` menyediakan interface terhadap informasi kernel dan process.

```bash
cat /proc/cpuinfo
cat /proc/meminfo
cat /proc/interrupts
```

#### `/sys`

`sysfs` mengekspos banyak informasi mengenai object, device, driver, dan hubungan hardware di kernel.

```bash
ls /sys/class/
ls /sys/bus/
ls /sys/devices/
```

---

### 3.4 File Descriptor

Salah satu konsep paling penting dalam Linux adalah **file descriptor (FD)**.

Ketika process membuka sebuah resource, kernel dapat memberikan sebuah integer sebagai handle.

Contoh:

```c
int fd = open("file.txt", O_RDONLY);
```

`fd` bukan file itu sendiri.

`fd` adalah **identifier** yang digunakan process untuk merujuk resource yang sedang dibuka.

Contoh umum:

```text
0 → stdin
1 → stdout
2 → stderr
```

Process juga dapat mempunyai FD untuk:

```text
regular file
socket
pipe
device
terminal
```

---

### 3.5 Permission dan User

Linux adalah sistem multi-user.

Setiap process berjalan dengan identity tertentu dan setiap file mempunyai permission.

```bash
ls -l
```

dapat menghasilkan:

```text
-rw-r--r-- 1 arif users 1234 file.txt
```

Secara sederhana:

![](/home/arif/Documents/markdown/Technical-Notes/Embedded-Linux/resource/fundamental-8.png)

```text
r = read
w = write
x = execute
```

---


