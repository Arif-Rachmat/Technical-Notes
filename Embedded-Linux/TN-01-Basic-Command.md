# Cheat Sheet — Operasional Dasar Linux

## 1. Navigasi Filesystem

| Command | Fungsi                          | Contoh    |
| ------- | ------------------------------- | --------- |
| `pwd`   | Menampilkan directory saat ini  | `pwd`     |
| `ls`    | Melihat isi directory           | `ls -lah` |
| `cd`    | Berpindah directory             | `cd /etc` |
| `cd ..` | Naik satu directory             | `cd ..`   |
| `cd ~`  | Ke home directory               | `cd ~`    |
| `cd -`  | Kembali ke directory sebelumnya | `cd -`    |

### Argumen navigasi

```bash
.       # current directory
..      # parent directory
~       # home directory
/       # root directory
-       # previous directory (dengan cd)
```

---

## 2. File & Directory

| Command          | Fungsi                             |
| ---------------- | ---------------------------------- |
| `touch file`     | Membuat file kosong                |
| `mkdir dir`      | Membuat directory                  |
| `mkdir -p a/b/c` | Membuat nested directory           |
| `cp src dst`     | Menyalin file                      |
| `cp -r src dst`  | Menyalin directory                 |
| `mv src dst`     | Memindahkan / rename               |
| `rm file`        | Menghapus file                     |
| `rm -r dir`      | Menghapus directory beserta isinya |
| `cat file`       | Menampilkan isi file               |
| `less file`      | Membaca file secara interaktif     |
| `head file`      | Melihat bagian awal file           |
| `tail file`      | Melihat bagian akhir file          |
| `tail -f file`   | Memantau file secara real-time     |

> **Hati-hati dengan `rm -r` dan terutama `rm -rf`.** Linux umumnya tidak menyediakan recycle bin untuk operasi ini.

---

## 3. Mencari File dan Data

### Mencari file

```bash
find . -name "file.txt"
find /etc -type f -name "*.conf"
```

### Mencari teks

```bash
grep "pattern" file.txt
grep -r "pattern" .
```

### Alternatif cepat

```bash
which gcc
whereis gcc
```

`which` mencari executable yang akan dijalankan dari `PATH`, sedangkan `whereis` mencari binary, source, dan manual yang terkait.

---

## 4. Melihat dan Memproses Output

### Pipe

```bash
command1 | command2
```

Contoh:

```bash
ps aux | grep ssh
```

### Redirect

```bash
command > file.txt
```

Menimpa file.

```bash
command >> file.txt
```

Menambahkan ke akhir file.

```bash
command < file.txt
```

Menggunakan file sebagai standard input.

### Standard streams

```text
stdin   → 0
stdout  → 1
stderr  → 2
```

Contoh:

```bash
command > output.txt 2> error.txt
```

Gabungkan stdout dan stderr:

```bash
command > output.txt 2>&1
```

Bentuk umum:

```bash
command &> output.txt
```

---

## 5. Permissions

Melihat permission:

```bash
ls -l
```

Contoh:

```text
-rwxr-xr-- 
```

```text
r = read
w = write
x = execute
```

Mengubah permission:

```bash
chmod +x script.sh
chmod 755 script.sh
```

Mengubah owner:

```bash
sudo chown user:group file
```

Menjalankan command dengan privilege root:

```bash
sudo command
```

Melihat identity:

```bash
whoami
id
```

---

## 6. Process

Melihat process:

```bash
ps aux
```

Monitor process secara interaktif:

```bash
top
```

atau:

```bash
htop
```

Mencari process:

```bash
pgrep firefox
```

Menghentikan process:

```bash
kill PID
```

Memaksa menghentikan:

```bash
kill -9 PID
```

Menjalankan program di background:

```bash
./program &
```

---

## 7. System Information

### Kernel

```bash
uname -r
uname -a
```

### OS / Distribution

```bash
cat /etc/os-release
```

### CPU

```bash
lscpu
```

### Memory

```bash
free -h
```

### Storage

```bash
lsblk
df -h
du -sh directory/
```

### Hardware

```bash
lspci
lsusb
```

### Kernel messages

```bash
dmesg
dmesg | less
```

---

## 8. Disk & Filesystem

Melihat disk dan partition:

```bash
lsblk
```

Melihat penggunaan filesystem:

```bash
df -h
```

Melihat ukuran directory:

```bash
du -sh /path/to/directory
```

Melihat mount:

```bash
mount
```

Informasi block device:

```bash
blkid
```

---

## 9. Networking

Melihat interface:

```bash
ip addr
```

atau:

```bash
ip link
```

Melihat routing:

```bash
ip route
```

Test koneksi:

```bash
ping 8.8.8.8
```

Melihat port / socket:

```bash
ss -tuln
```

DNS lookup:

```bash
getent hosts example.com
```

Transfer / request sederhana:

```bash
curl https://example.com
```

---

## 10. Archive dan Compression

### tar

Membuat archive:

```bash
tar -cf archive.tar directory/
```

Extract:

```bash
tar -xf archive.tar
```

Membuat gzip archive:

```bash
tar -czf archive.tar.gz directory/
```

Extract:

```bash
tar -xzf archive.tar.gz
```

Melihat isi archive:

```bash
tar -tf archive.tar.gz
```

### zip

```bash
zip -r archive.zip directory/
unzip archive.zip
```

---

## 11. Package Management

Package manager bergantung pada distribution.

### Debian / Ubuntu

```bash
sudo apt update
sudo apt install package
sudo apt remove package
```

### Arch Linux

```bash
sudo pacman -Syu
sudo pacman -S package
sudo pacman -R package
```

### Fedora

```bash
sudo dnf install package
sudo dnf remove package
```

> Jangan menganggap semua distro menggunakan package manager yang sama.

---

## 12. Services

Pada sistem yang menggunakan `systemd`:

Melihat status:

```bash
systemctl status service
```

Menjalankan:

```bash
sudo systemctl start service
```

Menghentikan:

```bash
sudo systemctl stop service
```

Restart:

```bash
sudo systemctl restart service
```

Aktifkan saat boot:

```bash
sudo systemctl enable service
```

Lihat log service:

```bash
journalctl -u service
```

Log terbaru:

```bash
journalctl -u service -f
```

---

## 13. Environment & PATH

Melihat environment:

```bash
env
```

Melihat variable:

```bash
echo $PATH
echo $HOME
```

Menentukan executable yang digunakan:

```bash
which gcc
```

Menambahkan directory ke `PATH` sementara:

```bash
export PATH="$PATH:/path/to/bin"
```

---

## 14. Text Processing Dasar

| Command | Fungsi                         |
| ------- | ------------------------------ |
| `grep`  | Mencari pola                   |
| `sort`  | Mengurutkan                    |
| `uniq`  | Menghilangkan duplikasi        |
| `wc`    | Menghitung baris/kata/byte     |
| `cut`   | Mengambil field/kolom          |
| `sed`   | Transformasi text              |
| `awk`   | Processing text berbasis field |

Contoh:

```bash
cat access.log | grep ERROR | sort | uniq -c
```

---

## 15. SSH

Connect ke remote machine:

```bash
ssh user@host
```

Menyalin file ke remote:

```bash
scp file user@host:/path/
```

Menyalin dari remote:

```bash
scp user@host:/path/file .
```

Untuk Embedded Linux, SSH sering menjadi cara utama untuk mengakses board selama tahap development.

---

## 16. Help & Documentation

Hampir setiap command memiliki dokumentasi bawaan.

```bash
man ls
man chmod
man systemctl
```

Help singkat:

```bash
command --help
```

Mencari command:

```bash
apropos keyword
```

Contoh:

```bash
apropos network
```

---

## 17. Command yang Sangat Sering Dipakai

```bash
pwd
ls -lah
cd
cp
mv
rm
mkdir
cat
less
grep
find
chmod
chown
sudo
ps
top
kill
df -h
du -sh
free -h
ip addr
ss
dmesg
journalctl
systemctl
man
```

---


