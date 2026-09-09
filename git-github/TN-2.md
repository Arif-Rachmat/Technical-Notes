# Note 2: SSH (Secure Shell)

> **Purpose:** catatan praktis dan konseptual tentang SSH untuk remote server, key-based authentication, `ssh-agent`, GitHub, transfer file, tunneling, konfigurasi multi-host, dan troubleshooting.
>
> **Scope:** OpenSSH pada Linux/macOS/Windows (PowerShell/OpenSSH atau Git Bash) dan GitHub over SSH.

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

```text
+----------------------+           Encrypted SSH           +----------------------+
| SSH Client           |  --------------------------------> | SSH Server           |
| laptop/workstation   |        TCP, default port 22       | sshd                 |
|                      |                                   |                      |
| private key          |                                   | authorized_keys      |
| ~/.ssh/id_ed25519    |                                   | ~/.ssh/authorized_keys
+----------------------+                                   +----------------------+
```

**Poin penting:** SSH mengenkripsi komunikasi, tetapi autentikasi dan otorisasi tetap harus dikonfigurasi dengan benar.

Referensi: [GitHub - Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) dan [Ubuntu Server - OpenSSH server](https://ubuntu.com/server/docs/how-to/security/openssh-server/).

---

## 2. SSH Client vs SSH Server

### Client

Program yang dijalankan dari komputer kita untuk memulai koneksi.

```bash
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

Pada sistem Unix-like, permission yang terlalu longgar pada `.ssh` atau `authorized_keys` dapat menyebabkan `sshd` menolak key authentication.

---

## 4. Port dan Format Command

Secara default SSH menggunakan TCP port **22**.

```bash
ssh [options] [user@]host
```

Contoh:

```bash
ssh arif@192.168.1.10
ssh arif@example.com
ssh -p 2222 arif@example.com
```

Makna:

- `user` = account pada host remote.
- `host` = hostname/IP remote.
- `-p` = port TCP SSH jika bukan 22.

Port bukan mekanisme autentikasi. Memindahkan SSH dari port 22 dapat mengurangi noise dari scanner, tetapi **bukan pengganti security controls**.

---

## 5. Instalasi / Setup Client

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

Pada Windows modern, OpenSSH Client tersedia sebagai optional/installed feature tergantung konfigurasi sistem.

Cek dari PowerShell:

```powershell
ssh -V
```

Git Bash juga umumnya menyediakan `ssh`, tetapi pastikan `ssh -V` mengarah ke client yang ingin digunakan.

---

## 6. Membuat SSH Key Pair

Untuk penggunaan modern, **Ed25519** adalah pilihan utama:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

Secara default akan menghasilkan:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

### Private vs public key

```text
                key pair
                   │
          ┌────────┴────────┐
          │                 │
    PRIVATE KEY         PUBLIC KEY
 id_ed25519          id_ed25519.pub
    SECRET              SHAREABLE
          │                 │
          │                 └──> server/GitHub
          │
          └──> TIDAK BOLEH dibagikan
```

Saat `ssh-keygen` meminta passphrase, **gunakan passphrase** kecuali ada alasan operasional yang jelas untuk tidak menggunakannya. Passphrase melindungi private key ketika file tersebut dicuri.

Ubuntu saat ini juga merekomendasikan Ed25519 karena ukuran key lebih kecil dan komputasi efisien; RSA 4096 masih relevan untuk kompatibilitas sistem lama.

```bash
# Alternatif kompatibilitas
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

---

## 7. Mengapa Key-Based Authentication Bisa “Passwordless”?

Istilah **passwordless SSH** biasanya berarti kita tidak perlu memasukkan **password account remote** setiap kali login. Bukan berarti koneksi tidak memiliki kredensial.

Alurnya secara konseptual:

```text
Client                                      Server
------                                      ------
private key                                public key
    │                                          │
    │      connection + key authentication    │
    ├─────────────────────────────────────────>│
    │                                          │
    │<──────── cryptographic challenge ────────┤
    │                                          │
    │  prove possession of private key         │
    ├─────────────────────────────────────────>│
    │                                          │
    │<──────────── session accepted ───────────┤
```

Private key **tetap di client**. Server hanya perlu public key yang sesuai.

Pada server, public key biasanya ditempatkan di:

```text
~/.ssh/authorized_keys
```

---

## 8. Mengaktifkan Passwordless Login ke Server

### Cara 1 — `ssh-copy-id` (Linux/macOS)

Setelah memiliki key:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server
```

Kemudian:

```bash
ssh user@server
```

Password account biasanya hanya diperlukan saat proses instalasi/copy key; setelah public key terpasang, client dapat login dengan key.

### Cara 2 — copy manual

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

### Windows → Linux

Dengan PowerShell, bila password login masih tersedia:

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub |
  ssh user@server "umask 077; mkdir -p ~/.ssh; cat >> ~/.ssh/authorized_keys"
```

Jika server menggunakan konfigurasi yang berbeda, copy key secara manual lebih aman untuk troubleshooting.

---

## 9. `ssh-agent`: Passwordless Tanpa Mengorbankan Passphrase

Ada dua hal yang sering tercampur:

1. **Tidak mengetik password account remote** → key-based authentication.
2. **Tidak mengetik passphrase private key berulang kali** → `ssh-agent`.

Rekomendasi umum:

```text
Account password   → tidak dipakai untuk login SSH
Private key        → tetap encrypted dengan passphrase
ssh-agent          → menyimpan key yang sudah unlock selama session
```

### Linux/macOS

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```

### Windows PowerShell

Cek service:

```powershell
Get-Service ssh-agent
```

Aktifkan dan mulai:

```powershell
Set-Service -Name ssh-agent -StartupType Manual
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519
ssh-add -l
```

`ssh-agent` memungkinkan aplikasi seperti `ssh`/Git menggunakan key tanpa meminta passphrase setiap command.

**Security note:** jangan menganggap `ssh-agent` sebagai vault tanpa risiko. Siapa pun yang telah memperoleh kontrol cukup atas account/session Anda dapat berpotensi menggunakan agent sesuai kebijakan agent/socket.

---

## 10. `~/.ssh/config`: Konfigurasi SSH yang Rapi

File:

```text
~/.ssh/config
```

Contoh:

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

Host myserver
    HostName 203.0.113.10
    User arif
    Port 22
    IdentityFile ~/.ssh/id_ed25519_server
    IdentitiesOnly yes
```

Setelah itu:

```bash
ssh myserver
```

bukan lagi:

```bash
ssh -p 22 arif@203.0.113.10 -i ~/.ssh/id_ed25519_server
```

### Kenapa `IdentitiesOnly yes` berguna?

Jika agent memiliki banyak key, SSH dapat mencoba beberapa identity. `IdentitiesOnly yes` membantu memaksa penggunaan identity yang dinyatakan untuk host tersebut dan mengurangi masalah seperti:

```text
Too many authentication failures
```

---

## 11. Menyambungkan SSH ke GitHub

GitHub mendukung Git melalui SSH. **GitHub bukan server shell umum untuk account kita**; koneksi SSH ke GitHub digunakan untuk autentikasi/transport Git, bukan untuk mendapatkan terminal Unix GitHub.

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

Di GitHub:

```text
Profile → Settings → SSH and GPG keys → New SSH key
```

Pilih **Authentication** untuk key yang akan dipakai login/autentikasi Git melalui SSH. Masukkan **public key**, bukan private key.

Referensi resmi: [GitHub Docs - Adding a new SSH key to your account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).

### Langkah 4 — Test koneksi

```bash
ssh -T git@github.com
```

Jika berhasil, GitHub akan menunjukkan pesan bahwa autentikasi berhasil tetapi GitHub tidak menyediakan shell access.

### Langkah 5 — Gunakan SSH URL untuk repository

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

## 12. HTTPS vs SSH untuk Git

```text
Git over HTTPS
    └── https://github.com/owner/repo.git
        └── authentication via supported HTTPS credentials/token flow

Git over SSH
    └── git@github.com:owner/repo.git
        └── authentication via SSH key
```

SSH sangat nyaman untuk developer workstation/server karena key dapat dipakai berulang tanpa memasukkan credential repository pada setiap `push`.

Namun, SSH bukan otomatis “lebih aman dalam semua konteks”. Security tetap bergantung pada private-key protection, host-key verification, account security, agent hygiene, access controls, dan endpoint security.

---

## 13. SSH Host Key dan `known_hosts`

Saat pertama kali mengakses host:

```bash
ssh user@example.com
```

SSH dapat menampilkan fingerprint dan bertanya apakah host dipercaya.

Setelah disetujui, identitas host disimpan di:

```text
~/.ssh/known_hosts
```

Tujuannya adalah membantu mendeteksi perubahan identitas host, misalnya pada skenario **man-in-the-middle**.

### Jangan membiasakan ini

```text
yes
```

secara buta tanpa memverifikasi fingerprint pada sistem sensitif.

### Lihat fingerprint/host key

```bash
ssh-keygen -lf ~/.ssh/known_hosts
```

Untuk troubleshooting/inspection yang lebih spesifik, gunakan:

```bash
ssh-keygen -F example.com
```

Jika host memang telah diganti secara sah, entri lama dapat dikelola dengan:

```bash
ssh-keygen -R example.com
```

**Jangan gunakan `ssh-keygen -R` untuk “menghilangkan warning” tanpa terlebih dahulu memahami kenapa host key berubah.**

---

## 14. Command SSH Sehari-hari

### Login interactive

```bash
ssh user@host
```

### Jalankan satu command remote

```bash
ssh user@host 'uname -a'
ssh user@host 'df -h'
ssh user@host 'systemctl status nginx --no-pager'
```

### Port berbeda

```bash
ssh -p 2222 user@host
```

### Identity berbeda

```bash
ssh -i ~/.ssh/id_ed25519_server user@host
```

### Debug verbose

```bash
ssh -v user@host
ssh -vv user@host
ssh -vvv user@host
```

Gunakan `-v` → `-vv` → `-vvv` sesuai kebutuhan; jangan selalu gunakan `-vvv` untuk koneksi normal karena output sangat ramai.

### Logout

```bash
exit
```

atau `Ctrl-D`.

---

## 15. Transfer File

### `scp`

Local → remote:

```bash
scp file.txt user@server:/tmp/
```

Remote → local:

```bash
scp user@server:/tmp/file.txt .
```

Directory recursive:

```bash
scp -r ./project user@server:/opt/
```

### `sftp`

Interactive file transfer:

```bash
sftp user@server
```

Command umum di dalam `sftp`:

```text
ls
pwd
cd /path
lpwd
lcd /local/path
get file.txt
put file.txt
exit
```

### `rsync` via SSH

Untuk sinkronisasi directory yang efisien:

```bash
rsync -avz -e ssh ./project/ user@server:/opt/project/
```

**Catatan:** `rsync` perlu tersedia di client dan server untuk mode remote shell.

---

## 16. Menjalankan Background Job di Server

SSH session terputus dapat menghentikan process yang bergantung pada terminal/session.

Untuk pekerjaan interaktif yang panjang, gunakan **tmux** atau **screen**:

```bash
tmux new -s work
```

Detach:

```text
Ctrl-b d
```

Reconnect:

```bash
tmux attach -t work
```

Ini berbeda dari daemon/service manager. Untuk service production, gunakan `systemd`/service manager yang sesuai.

---

## 17. Port Forwarding / Tunneling

SSH tidak hanya memberi remote shell; ia juga dapat membuat tunnel TCP.

### Local port forwarding (`-L`)

Format:

```bash
ssh -L LOCAL_PORT:DEST_HOST:DEST_PORT user@ssh-host
```

Contoh akses PostgreSQL yang hanya listen di server:

```bash
ssh -L 15432:127.0.0.1:5432 user@server
```

Lalu dari local machine:

```text
localhost:15432 → SSH tunnel → server:5432
```

### Remote port forwarding (`-R`)

```bash
ssh -R REMOTE_PORT:DEST_HOST:DEST_PORT user@server
```

### SOCKS proxy (`-D`)

```bash
ssh -D 1080 user@server
```

Aplikasi yang mendukung SOCKS5 dapat diarahkan ke proxy lokal `127.0.0.1:1080`.

**Security:** forwarding dapat memperluas reachability jaringan. Jangan membuka tunnel yang tidak dipahami pada production environment.

---

## 18. Jump Host / Bastion

Jika target tidak dapat diakses langsung:

```bash
ssh -J bastion user@internal-server
```

Contoh konfigurasi:

```sshconfig
Host internal
    HostName 10.0.0.20
    User arif
    ProxyJump bastion

Host bastion
    HostName bastion.example.com
    User arif
```

Kemudian cukup:

```bash
ssh internal
```

Model ini umum pada jaringan production yang memisahkan internet-facing bastion dari private subnet.

---

## 19. Keepalive untuk Koneksi Tidak Stabil

Jika network sering idle timeout, client dapat mengirim keepalive:

```sshconfig
Host myserver
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

Interpretasi praktis:

- kirim keepalive sekitar setiap 60 detik;
- setelah sejumlah kegagalan berturut-turut, client menganggap koneksi mati.

Jangan menganggap keepalive sebagai perbaikan semua masalah jaringan.

---

## 20. Setup SSH Server yang Lebih Aman

Contoh baseline pada Ubuntu/Linux:

1. Buat user non-root untuk administrasi.
2. Install OpenSSH server.
3. Pasang public key.
4. Uji login key **sebelum** menonaktifkan password login.
5. Validasi konfigurasi dengan `sshd -t`.
6. Baru apply perubahan konfigurasi.

File konfigurasi utama biasanya:

```text
/etc/ssh/sshd_config
```

Ubuntu juga menggunakan snippet:

```text
/etc/ssh/sshd_config.d/*.conf
```

Contoh directive yang umum dipertimbangkan setelah key authentication terbukti bekerja:

```text
PubkeyAuthentication yes
PasswordAuthentication no
PermitRootLogin no
```

Setelah edit:

```bash
sudo sshd -t
sudo systemctl restart ssh.service
```

**Peringatan kritis:** salah konfigurasi SSH pada remote-only server dapat mengunci Anda keluar. Selalu pertahankan satu session administrasi aktif saat menguji perubahan dan pastikan jalur recovery tersedia.

Ubuntu secara eksplisit menyarankan `sudo sshd -t` sebelum restart karena konfigurasi SSH yang salah dapat membuat server tidak dapat diakses. Lihat [Ubuntu OpenSSH documentation](https://ubuntu.com/server/docs/how-to/security/openssh-server/).

---

## 21. Permission yang Penting di Linux

Baseline umum:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Private key:

```text
-rw-------  id_ed25519
```

Public key:

```text
-rw-r--r--  id_ed25519.pub
```

`known_hosts` dan beberapa file konfigurasi dapat memiliki permission berbeda sesuai distro/konfigurasi, tetapi prinsipnya tetap: private credential harus terlindungi dan `.ssh` tidak boleh writable oleh user lain.

---

## 22. Troubleshooting Cepat

### A. `Permission denied (publickey)`

Checklist:

```bash
ssh-add -l
ssh -v user@server
```

Lalu periksa di server:

```bash
ls -ld ~ ~/.ssh
ls -l ~/.ssh/authorized_keys
```

Pastikan:

- public key memang ada di `authorized_keys`;
- username benar;
- private key yang benar digunakan;
- `ssh-agent` memegang key bila diperlukan;
- permission dan ownership benar;
- `sshd` mengizinkan public-key authentication.

### B. `Too many authentication failures`

Biasanya terlalu banyak identity ditawarkan oleh agent/client.

Coba:

```bash
ssh -o IdentitiesOnly=yes -i ~/.ssh/id_ed25519 user@server
```

atau atur pada `~/.ssh/config`.

### C. `Connection refused`

Kemungkinan:

- `sshd` tidak aktif;
- port salah;
- firewall/security group menolak;
- service listen pada address/port berbeda.

Di server:

```bash
sudo systemctl status ssh
sudo ss -lntp | grep ssh
```

### D. `Connection timed out`

Lebih sering menunjukkan masalah network path/firewall/security group daripada masalah credential.

### E. Host key changed / `REMOTE HOST IDENTIFICATION HAS CHANGED!`

**Jangan langsung hapus `known_hosts`.** Verifikasi terlebih dahulu bahwa host memang diganti/reinstalled/migrated. Baru, bila perubahan sah:

```bash
ssh-keygen -R host.example.com
```

Lalu reconnect dan verifikasi fingerprint baru.

### F. Server menolak key walaupun file benar

Lihat log SSH:

```bash
sudo journalctl -fu ssh.service
```

atau pada distro tertentu:

```bash
sudo journalctl -u sshd
```

Nama unit dapat berbeda (`ssh.service` vs `sshd.service`).

---

## 23. Identity Management untuk Banyak Environment

Jangan gunakan satu key file tanpa struktur untuk seluruh kebutuhan.

Contoh:

```text
~/.ssh/
├── id_ed25519_github
├── id_ed25519_server_dev
├── id_ed25519_server_prod
└── config
```

Config:

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes

Host dev
    HostName dev.example.com
    User arif
    IdentityFile ~/.ssh/id_ed25519_server_dev
    IdentitiesOnly yes

Host prod
    HostName prod.example.com
    User arif
    IdentityFile ~/.ssh/id_ed25519_server_prod
    IdentitiesOnly yes
```

Keuntungan:

- blast radius lebih kecil ketika satu key bocor;
- identitas lebih jelas;
- rotasi/revocation lebih mudah;
- debugging lebih sederhana.

---

## 24. `ssh-keygen` Commands yang Berguna

Generate key:

```bash
ssh-keygen -t ed25519 -C "comment"
```

Fingerprint:

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

Ubah passphrase private key:

```bash
ssh-keygen -p -f ~/.ssh/id_ed25519
```

Lihat lokasi/default supported identities:

```bash
ssh-keygen -l -f ~/.ssh/id_ed25519.pub
```

### Public key yang hilang

Jika private key masih tersedia, public key dapat direkonstruksi untuk algoritma/key format yang didukung, misalnya:

```bash
ssh-keygen -y -f ~/.ssh/id_ed25519 > ~/.ssh/id_ed25519.pub
```

Namun, perlakukan operasi ini sebagai recovery dan tetap verifikasi hasilnya.

---

## 25. `ssh-add` Commands yang Berguna

List key di agent:

```bash
ssh-add -l
```

Add key:

```bash
ssh-add ~/.ssh/id_ed25519
```

Remove one key:

```bash
ssh-add -d ~/.ssh/id_ed25519
```

Remove all identities dari agent:

```bash
ssh-add -D
```

Gunakan `ssh-add -D` dengan hati-hati bila ada aplikasi lain yang masih membutuhkan identity dalam agent.

---

## 26. GitHub Workflow Ringkas

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

Workflow harian:

```bash
git pull
git status
git add .
git commit -m "docs: update notes"
git push
```

Tidak perlu mengirim private key ke GitHub. Git menggunakan SSH client pada local machine untuk membuktikan kepemilikan key.

---

## 27. Security Rules — Hafalkan Ini

### Jangan pernah

```text
❌ commit private key ke Git
❌ upload `id_ed25519` ke GitHub
❌ mengirim private key melalui chat/email
❌ menyalin private key ke server hanya agar login berhasil
❌ mematikan host-key checking secara permanen
❌ menonaktifkan password SSH sebelum key login teruji
❌ menghapus warning host key tanpa investigasi
```

### Lakukan

```text
✅ gunakan Ed25519 untuk deployment modern
✅ gunakan passphrase pada private key
✅ gunakan ssh-agent untuk kenyamanan
✅ gunakan key berbeda untuk environment berbeda jika sesuai threat model
✅ verifikasi host fingerprint pada sistem sensitif
✅ backup/recovery access dirancang sebelum remote-only changes
✅ rotasi/revoke key jika compromise dicurigai
```

---

## 28. Mental Model yang Perlu Diingat

```text
                SSH CLIENT
                    │
       ┌────────────┴────────────┐
       │                         │
  private key                ssh config
       │                         │
       └────────────┬────────────┘
                    │
              SSH protocol
                    │
            network / TCP 22
                    │
       ┌────────────┴────────────┐
       │                         │
      sshd                  known host key
       │
       └── authorized_keys
                    │
              authenticated
                    │
       ┌────────────┼────────────┐
       │            │            │
     shell        command      forwarding
```

### Tiga konsep yang jangan tertukar

| Konsep | Pertanyaan yang dijawab |
|---|---|
| **Host key** | “Apakah saya berbicara dengan server yang benar?” |
| **User authentication key** | “Apakah client ini memiliki credential untuk account ini?” |
| **Authorization** | “Setelah autentikasi, apa yang boleh dilakukan user ini?” |

---

## 29. Cheat Sheet

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

---

## 30. Gambar / Diagram yang Disarankan

Untuk menjaga repo tetap ringan, diagram konseptual sederhana di atas dapat dipertahankan langsung dalam Markdown. Untuk gambar eksternal, gunakan sumber yang kredibel dan hindari screenshot/diagram yang lisensinya tidak jelas.

### Gambar 1 — SSH Key Pair

**Deskripsi:** diagram laptop di kiri dengan `private key` dan `public key`, lalu public key dikirim ke server sementara private key tetap di laptop.

**Search keywords:**

```text
SSH public key private key authentication diagram
OpenSSH key based authentication diagram
SSH key pair public private key diagram
```

### Gambar 2 — SSH Authentication Flow

**Deskripsi:** sequence/flow client → server yang menunjukkan koneksi, host-key verification, public-key authentication, challenge/signature, lalu session established.

**Search keywords:**

```text
SSH public key authentication flow diagram
SSH authentication handshake diagram
OpenSSH authentication flow public key
```

### Gambar 3 — GitHub SSH Setup

**Deskripsi:** screenshot GitHub Settings → SSH and GPG keys → Add/New SSH key, dengan contoh public key yang ditempel ke field Key.

**Search keywords:**

```text
GitHub Settings SSH and GPG keys add SSH key
GitHub add new SSH authentication key screenshot
```

Sumber primer yang cocok untuk screenshot/prosedur: [GitHub Docs - Adding a new SSH key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account).

### Gambar 4 — SSH Remote Architecture

**Deskripsi:** workstation → internet/LAN → firewall/router → SSH server (`sshd`) → user shell/process.

**Search keywords:**

```text
SSH client server architecture diagram
OpenSSH remote login architecture
SSH bastion host architecture diagram
```

### Gambar 5 — SSH Tunneling

**Deskripsi:** browser/app di localhost menuju local port, melewati encrypted SSH tunnel, lalu keluar ke service internal; cocok untuk menjelaskan `ssh -L`.

**Search keywords:**

```text
SSH local port forwarding diagram
SSH tunneling local forwarding architecture
SSH port forwarding diagram -L -R
```

---

## 31. Referensi Utama

1. [GitHub Docs — Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
2. [GitHub Docs — Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
3. [GitHub Docs — Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
4. [Ubuntu Server — OpenSSH server](https://ubuntu.com/server/docs/how-to/security/openssh-server/)
5. [OpenSSH / ssh manual](https://man.openbsd.org/ssh)
6. [OpenSSH / ssh_config manual](https://man.openbsd.org/ssh_config)

---

## 32. Inti yang Harus Dikuasai

Setelah mempelajari note ini, minimal harus bisa menjelaskan dan melakukan hal berikut tanpa tutorial:

```text
1. Apa beda ssh client dan sshd.
2. Apa beda private key, public key, host key, dan authorized_keys.
3. Membuat Ed25519 key pair.
4. Login ke server dengan public-key authentication.
5. Menggunakan ssh-agent + ssh-add.
6. Menulis ~/.ssh/config untuk banyak host.
7. Menguji koneksi GitHub dengan `ssh -T git@github.com`.
8. Mengubah Git remote dari HTTPS ke SSH.
9. Transfer file dengan scp/sftp/rsync.
10. Membaca `ssh -v` untuk debugging.
11. Memahami host-key warning dan known_hosts.
12. Membuat local port forwarding sederhana.
13. Melakukan hardening SSH tanpa mengunci akses sendiri.
```
