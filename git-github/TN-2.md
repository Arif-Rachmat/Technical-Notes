# Note 2: SSH (Secure Shell)

> **Tujuan:** SSH untuk remote server, key-based authentication, `ssh-agent`, GitHub, transfer file, tunneling, konfigurasi multi-host, dan troubleshooting.
>
> **Cakupan:** OpenSSH pada Linux/macOS/Windows (PowerShell/OpenSSH atau Git Bash) dan GitHub over SSH.

---

## 1. Apa itu SSH?

**SSH (Secure Shell)** adalah protokol jaringan untuk membuat koneksi terenkripsi ke host lain melalui jaringan yang tidak sepenuhnya dipercaya.

Implementasi yang umum digunakan adalah **OpenSSH**, yang menyediakan `ssh` (client), `sshd` (server), `ssh-keygen`, `ssh-agent`, `ssh-add`, `scp`, dan berbagai utilitas terkait.

SSH dapat digunakan untuk:

- remote shell/login ke server;
- menjalankan command tanpa membuka interactive shell;
- autentikasi berbasis cryptographic key;
- transfer file (`scp`, `sftp`, `rsync` via SSH);
- port forwarding/tunneling;
- menjadi transport untuk Git, termasuk GitHub.

### Model sederhana

![Model Sederhana SSH](/git-github/Resource/ssh-01.png)

**Poin penting:** SSH mengenkripsi komunikasi, tetapi autentikasi dan otorisasi tetap harus dikonfigurasi dengan benar.

### Client

Program yang dijalankan dari komputer kita untuk memulai koneksi.

```
ssh user@server
```

### Server

Program `sshd` yang menerima koneksi SSH dan menangani autentikasi.

Contoh pada Ubuntu:

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl status ssh
```

Client saja cukup bila kebutuhan kita hanya **keluar menuju server** atau menggunakan GitHub. Server diperlukan jika komputer kita sendiri harus menerima koneksi SSH.

---

## 2. Instalasi / Setup Client

### Linux (Debian/Ubuntu)

```bash
sudo apt update
sudo apt install openssh-client
ssh -V
```

### macOS

OpenSSH biasanya sudah tersedia.

```bash
ssh -V
```

### Windows

Pada Windows modern, `OpenSSH Client` tersedia sebagai `optional/installed feature` tergantung konfigurasi sistem.

![mengaktifkan windows feature untuk OpenSSH](/git-github/Resource/ssh-02.png)

Cek dari PowerShell:

```powershell
ssh -V
```

Git Bash juga umumnya menyediakan `ssh`, tetapi pastikan `ssh -V` mengarah ke client yang ingin digunakan.

---

## 3. Command Dasar

Secara default SSH menggunakan TCP port **22**.

```powershell
ssh [options] [user@]host
```

Contoh:

```powershell
ssh arif@192.168.1.10
ssh arif@example.com
ssh -p 2222 arif@example.com
```

Makna:

- `user` = account pada host remote.
- `host` = hostname/IP remote.
- `-p` = port TCP SSH jika bukan 22.

>**NOTE:** Pada koneksi SSH pertama ke suatu server, Anda akan diminta mengonfirmasi host key fingerprint seperti pada gambar berikut: 
![Fingerprint Konfirmasi](/git-github/Resource/ssh-04.png)
Hal ini karena host tersebut belum ada di `~/.ssh/known_hosts`. Verifikasi fingerprint terlebih dahulu sebelum memilih `yes`; setelah dikonfirmasi, fingerprint akan disimpan untuk membantu mendeteksi perubahan identitas server pada koneksi berikutnya.

---

## 3. Struktur Dasar `~/.ssh`

Contoh:

```text
~/.ssh/
├── config
├── known_hosts
├── id_ed25519          # PRIVATE KEY — rahasia
├── id_ed25519.pub      # PUBLIC KEY — boleh dibagikan
└── authorized_keys     # hanya di sisi server
```

Fungsi utama:

| File | Fungsi |
|---|---|
| `id_ed25519` | private key client; jangan dibagikan |
| `id_ed25519.pub` | public key pasangan private key |
| `authorized_keys` | daftar public key yang diizinkan login ke account tersebut |
| `known_hosts` | host keys server yang pernah dikenal client |
| `config` | aturan koneksi SSH per host |

---




## 4. “Passwordless” dengan *Key-Based Authentication*

Dengan **Passwordless SSH** kita tidak perlu memasukkan **password account remote** setiap kali login (Tapi bukan berarti koneksi tidak memiliki kredensial).

![Key-Based authentication](/git-github/Resource/ssh-03.png)

Private key **tetap di client**. Server hanya perlu public key yang sesuai.

Pada server, public key biasanya ditempatkan di:

```text
~/.ssh/authorized_keys
```

---

### Mengkonfigurasi Passwordless Login

#### Step 1 – Membuat SSH Key Pair

Untuk penggunaan modern, standarnya adalah menggunakan **Ed25519**:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Secara default akan menghasilkan:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```
>**NOTE:** Saat `ssh-keygen` meminta passphrase, **gunakan passphrase** kecuali ada alasan operasional yang jelas untuk tidak menggunakannya. Passphrase melindungi private key ketika file tersebut dicuri.

Ubuntu saat ini juga merekomendasikan `Ed25519` karena ukuran key lebih kecil dan komputasi efisien; `RSA 4096` masih relevan untuk kompatibilitas sistem lama.

```bash
# Algoritma Alternatif
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

#### Step 2 – Mengaktifkan kunci SSH Key Pair ke Server

##### **Cara 1** — `ssh-copy-id` (Linux/macOS)

Setelah memiliki key:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server
```

Kemudian:

```bash
ssh user@server
```

Password account biasanya hanya diperlukan saat proses instalasi/copy key; setelah public key terpasang, client dapat login dengan key.

##### **Cara 2** — copy manual

Di server:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat >> ~/.ssh/authorized_keys
```

Paste **satu baris penuh** dari `id_ed25519.pub`, lalu `Ctrl-D`.

Kemudian:

```bash
chmod 600 ~/.ssh/authorized_keys
```

Pastikan owner file/direktori benar.

##### Windows → Linux

Dengan PowerShell, bila password login masih tersedia:

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | 
  ssh user@server "umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

Jika server menggunakan konfigurasi yang berbeda, copy key secara manual lebih aman dan mudah untuk troubleshooting.

---

## 5. Menyambungkan SSH ke GitHub

GitHub mendukung Git melalui SSH. Koneksi SSH ke GitHub ini digunakan untuk autentikasi/transport Git.

### Langkah 1 — Pastikan key ada

```bash
ls -la ~/.ssh
```

atau Windows PowerShell:

```powershell
Get-ChildItem $env:USERPROFILE\.ssh
```

### Langkah 2 — Pastikan agent memegang key

```bash
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

### Langkah 3 — Tambahkan public key ke GitHub

Tampilkan:

```bash
cat ~/.ssh/id_ed25519.pub
```

Windows PowerShell:

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```
Maka akan muncul sesuatu seperti:
```bash
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAnnJo3L8CifWtxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx myemail@gmail.com
```
Copy seluruh konten `id_ed25519.pub` tersebut, kemudian masukkan ke [pengaturan SSH GitHub](https://github.com/settings/keys):

![Tutorial mengatur SSH key pada GitHub](/git-github/Resource/ssh-05.png)

>**NOTE:** Pastikan memasukkan **public key**, dan bukan private key.

Referensi resmi: [GitHub Docs - Adding a new SSH key to your account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).

### Langkah 4 — Test koneksi

```bash
ssh -T git@github.com
```

Jika berhasil, GitHub akan menunjukkan pesan bahwa autentikasi berhasil tetapi GitHub tidak menyediakan shell access.

### Langkah 5 — Mengunakan SSH URL untuk repository

```text
git@github.com:OWNER/REPOSITORY.git
```

Contoh:

```bash
git clone git@github.com:Arif-Rachmat/Technical-Notes.git
```

Atau ubah remote existing:

```bash
git remote set-url origin git@github.com:Arif-Rachmat/Technical-Notes.git
```

Cek:

```bash
git remote -v
```

---

## 6. Rangkuman Command

```bash
# Login
ssh user@host
ssh -p 2222 user@host
ssh -i ~/.ssh/id_ed25519 user@host
ssh -v user@host

# Key management
ssh-keygen -t ed25519 -C "email@example.com"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh-keygen -lf ~/.ssh/id_ed25519.pub

# Copy key
ssh-copy-id user@host

# File transfer
scp file user@host:/path/
sftp user@host
rsync -avz -e ssh ./dir/ user@host:/path/

# Remote command
ssh user@host 'uname -a'
ssh user@host 'df -h'

# Tunneling
ssh -L 15432:127.0.0.1:5432 user@host
ssh -R 8080:127.0.0.1:3000 user@host
ssh -D 1080 user@host

# Jump host
ssh -J bastion user@internal

# GitHub
ssh -T git@github.com
git remote -v
git remote set-url origin git@github.com:OWNER/REPO.git

# Host key
ssh-keygen -F host.example.com
ssh-keygen -R host.example.com

# Server validation
sudo sshd -t
sudo systemctl status ssh
sudo journalctl -fu ssh.service
```

### Setup SSH GitHub 

Setup satu kali:

```bash
# 1. Generate key
ssh-keygen -t ed25519 -C "your_email@example.com"

# 2. Load ke agent
ssh-add ~/.ssh/id_ed25519

# 3. Tampilkan public key
cat ~/.ssh/id_ed25519.pub

# 4. Tambahkan public key ke GitHub
#    Settings → SSH and GPG keys → New SSH key

# 5. Test
ssh -T git@github.com
```

Set repository ke SSH:

```bash
git remote set-url origin git@github.com:Arif-Rachmat/Technical-Notes.git
```

---

## 7. Referensi Utama

1. [GitHub Docs — Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
2. [GitHub Docs — Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
3. [GitHub Docs — Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
4. [Ubuntu Server — OpenSSH server](https://ubuntu.com/server/docs/how-to/security/openssh-server/)
5. [OpenSSH / ssh manual](https://man.openbsd.org/ssh)
6. [OpenSSH / ssh_config manual](https://man.openbsd.org/ssh_config)