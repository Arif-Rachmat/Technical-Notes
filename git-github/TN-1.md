# Note 1: Git dan Repository Lokal



## 1. Arsitektur Dasar Git & File States (*Optional, nice to know*)

Git bekerja dengan mengambil **snapshot** (foto kondisi seluruh proyek), bukan mencatat perbedaan baris file (*delta*). Git membagi direktori lokal menjadi 3 area utama:

1. **Working Directory:** Tempat file diedit secara fisik oleh developer.
2. **Staging Area (Index):** Area transit untuk mendaftarkan file yang siap masuk ke commit berikutnya (`git add`).
3. **Local Repository (HEAD):** Database lokal yang menyimpan seluruh riwayat commit (`git commit`).

![3 Stage of Git: CWD, Staging Area, Repository (Lokal)](/git-github/Resource/3StageOfGit.png)


### Pointer Utama:

* **HEAD:** Pointer yang menunjuk ke commit/branch yang sedang aktif di Working Directory saat ini.
* **Branch:** Pointer yang menunjuk ke commit paling ujung dari jalur pengembangan tertentu.

---

## **2. Konfigurasi dan Penggunaan Sehari-hari**

### **a. Konfigurasi Utama**
Saat baru pertama kali menjalankan Git, jalankan:
```bash
# Identitas Wajib (Global)
git config --global user.name "Nama User"
git config --global user.email "developer@email.com"

# Default Branch & Line Endings
git config --global init.defaultBranch main
git config --global core.autocrlf input # Linux/macOS (Gunakan 'true' untuk Windows)

# Cek Konfigurasi Aktif
git config --list --show-origin

```

### **b. Workflow Harian & Inspeksi State**

```bash
# Cek status area kerja secara ringkas
git status -s

# Pindahkan file ke Staging Area
git add src/index.js           # Spesifik file
git add .                      # Seluruh file/subfile dan folder/subfolder dari CWD
git add -p                     # Interaktif per-hunk/bagian kode

# Commit ke repository lokal / version control
git commit -m "Nama/Deskripsi Commit"

# Visualisasi Log Ringkas & Linear
git log --graph --oneline --decorate -n 10

```
**NOTES:** Sebuah commit sebaiknya bersifat **Atomic** (fokus pada satu perubahan logis yang dapat berfungsi mandiri). Gunakan standar format Conventional Commits:

`<type>(<scope>): <description>`

- `feat`: Fitur baru.
- `fix`: Perbaikan bug.
- `refactor`: Restrukturisasi kode tanpa mengubah fungsionalitas.
- `docs`: Dokumentasi.
- `test`: Pengujian/Unit Test.
 
**Contoh:**
```bash
git add src/auth.py
git commit -m "feat(auth): add JWT token generation on login"
```

### **c. Mengabaikan File atau Folder dengan `.gitignore`**
File `.gitignore` digunakan untuk menjaga repositori Git tetap bersih dengan mengabaikan file atau folder yang tidak perlu atau bersifat sementara, seperti file hasil build, dependensi pihak ketiga, file log, konfigurasi editor, atau data sensitif seperti API key. File ini biasanya ditempatkan di direktori root repositori, tetapi bisa juga ada di sub-folder untuk pengaturan pengabaian yang lebih spesifik

Berikut adalah contoh isi file ignore:
```gitignore
# Dependencies & Environment
node_modules/
.venv/
.env

# Build Output & Logs
dist/
*.log
```
Maka file dan folder yang akan dilacak (✅) dan yang tidak (❌) oleh `git` adalah sebagai berikut:
```text
my-awesome-project/
├── .env                          <-- Sensitive: Environment variables (API keys, DB secrets) (❌)
├── .gitignore                    <-- Git configuration: Specifies files to ignore (✅)
├── README.md                     <-- Tracked: Project documentation (✅)
├── package.json                  <-- Tracked: Project manifest and dependencies (✅)
├── package-lock.json             <-- Tracked: Exact dependency versions lockfile (✅)
├── src/                          <-- Source Code Directory (✅)
│   ├── index.js                  <-- Main entry point (✅)
│   ├── app.js                    <-- Express application setup (✅)
│   └── styles.css                <-- Main stylesheet (✅)
├── public/                       <-- Public assets directory (✅)
│   ├── index.html                <-- HTML template (✅)
│   └── favicon.ico               <-- Site icon (✅)
├── node_modules/                 <-- Heavy: Installed npm packages (❌)
│   ├── express/                  <-- Express package files (❌)
│   └── debug/                    <-- Debug package files (❌)
├── dist/                         <-- Output: Distribution folder (❌)
│   └── app.min.js                <-- Minified distribution file (❌)
└── logs/                         <-- Runtime logs directory
    ├── error.log                 <-- Error log file (❌)
    └── combined.log              <-- Combined log output (❌)
```


---

## **3. Branching, Merging, & Rebase Strategy**

### **Navigasi Branch**

Gunakan `git switch` dan `git restore` untuk menggantikan perintah `git checkout` (*Legacy*):

```bash
# Buat dan pindah ke branch baru
git switch -c feature/payment-gateway

# Pindah ke branch yang sudah ada
git switch main

# Membuang perubahan di Working Directory (Batal Edit)
git restore src/app.js

# Mendorong file keluar dari Staging Area kembali ke Working Directory
git restore --staged src/app.js
```

### **Merge vs Rebase**
![Git Merge VS Rebase Visualization](/git-github/Resource/GitMergeVSRebase.jpg)
| **Parameter** | `git merge` | `git rebase` |
| --- | --- | --- |
| **Mekanisme** | Menggabungkan 2 branch dengan membuat *3-way merge commit*. | Memindahkan basis commit branch fitur ke puncak branch target. |
| **Bentuk Histori** | Non-linear (bercabang dan mempertahankan konteks historis). | Linear (lurus tanpa merge commit tambahan). |
| **Kasus Penggunaan** | Penggabungan *feature branch* ke `main` (Publik). | Merapikan *feature branch* lokal sebelum di-merge ke `main`. |

```bash
# Melakukan Rebase branch fitur di atas main
git switch feature/payment-gateway
git rebase main

# Menggabungkan ke main via Merge
git switch main
git merge feature/payment-gateway
```

### Resolusi Konflik

Saat terjadi konflik kode:

1. Buka file berkonflik, pilih bagian kode yang benar, lalu hapus marker (`<<<<<<<`, `=======`, `>>>>>>>`).
2. Tandai file selesai diperbaiki: `git add <file>`
3. Lanjutkan proses: `git rebase --continue` atau `git commit` (jika merge).

---

## **4. Undo & Emergency Recovery**

### Guide Pemulihan Kode atau Versi

| Perintah | Efek pada `HEAD` | Staging Area | Working Directory | Skenario Penggunaan |
| --- | --- | --- | --- | --- |
| **`git reset --soft HEAD~1`** | Mundur 1 commit | **Tetap Ada** | **Tetap Ada** | Salah pesan commit atau ingin memecah commit. |
| **`git reset --mixed HEAD~1`** | Mundur 1 commit | **Dibersihkan** | **Tetap Ada** | Ingin menata ulang file yang dimasukkan ke staging. |
| **`git reset --hard HEAD~1`** | Mundur 1 commit | **Dihapus** | **Dihapus** | Membuang total commit terakhir beserta seluruh kodenya. |
| **`git revert <commit_id>`** | Maju (Buat Commit Baru) | Unchanged | Unchanged | Batalkan commit yang **sudah di-push ke server/public branch**. |


### Menyelamatkan Commit yang Hilang (`git reflog`)

Jika tidak sengaja melakukan `git reset --hard` atau kehilangan commit:

```bash
# 1. Buka catatan aktivitas HEAD lokal
git reflog

# Output:
# a1b2c3d HEAD@{0}: reset: moving to HEAD~1
# e5f6g7h HEAD@{1}: commit: feat: important feature lost

# 2. Kembalikan commit yang hilang ke branch baru
git switch -c recovered-work e5f6g7h

```

### Detached HEAD State

Kondisi saat `HEAD` menunjuk langsung ke sebuah *commit hash*, bukan nama *branch*.

<div align="center">

**contoh `HEAD` yang normal:**

![Normal HEAD State Visualization](/git-github/Resource/NormalHead.png)

**Contoh Detached `HEAD`:**

![Detached HEAD State Visualization](/git-github/Resource/Detached%20HEAD.jpg)
</div>

Cara memperbaiki Detached `HEAD`:

```bash
# Keluar dari Detached HEAD tanpa membuang perubahan
git switch -c fix-experiment
```

---

## **5. Stashing & Context Switching (Multi-Tasking Lokal)**

Saat terjadi kondisi darurat di mana kita harus segera berpindah ke *branch* lain (misal: ada bug yang harus *patch* mendadak), tetapi perubahan di *Working Directory* saat ini belum siap di-commit:

### Manajemen Stash

`git stash` menyimpan perubahan lokal (*tracked files*) ke dalam tumpukan (*stack*) sementara dan mengembalikan *Working Directory* ke kondisi `HEAD`.

```bash
# Simpan perubahan berlabel pesan spesifik (disarankan)
git stash save "WIP: alur validasi formulir belum selesai"

# Simpan termasuk file baru yang belum ter-track (untracked files)
git stash -u

# Lihat daftar seluruh simpanan stash
git stash list
# Output: stash@{0}: On feature/auth: WIP: alur validasi formulir...

# Terapkan stash terbaru DAN hapus dari stack
git stash pop

# Terapkan stash spesifik TANPA menghapusnya dari stack
git stash apply stash@{0}

# Hapus stash tertentu atau bersihkan seluruh isi stack
git stash drop stash@{0}
git stash clear
```

### Alternatif: `git worktree`

Jika perlu mengerjakan dua *branch* secara bersamaan tanpa perlu bolak-balik, gunakan `git stash` atau `git switch`:

```bash
# Buat direktori paralel baru terpisah untuk mengerjakan hotfix
git worktree add ../project-hotfix main

# Pindah direktori terminal ke area kerja hotfix
cd ../project-hotfix

# Setelah selesai dan di-merge, hapus worktree tersebut
cd ../project-main
git worktree remove ../project-hotfix
```

---

## **6. Code Inspection & Debugging Tools**

### 1. Line-by-Line Attribution (`git blame`)

Digunakan untuk melihat siapa yang terakhir kali mengubah baris kode tertentu, pada *commit* berapa, dan kapan.

```bash
# Lacak baris 15 hingga 30 pada file tertentu
git blame -L 15,30 src/services/auth.service.js
```

### 2. Pickaxe Search (`git log -S`)

Mencari *commit* spesifik berdasarkan string atau kode yang **ditambahkan atau dihapus** dari basis kode sepanjang riwayat proyek.

```bash
# Cari commit yang menambah atau menghapus teks "JWT_SECRET_KEY"
git log -S "JWT_SECRET_KEY" --oneline
```

### 3. Binary Search Debugging (`git bisect`)

Metode paling efisien untuk melacak *commit* spesifik mana yang menyebabkan *bug* atau *regression* menggunakan algoritma *binary search*.

```bash
# 1. Mulai sesi bisect
git bisect start

# 2. Tandai bahwa versi saat ini mengalami bug (Bad)
git bisect bad

# 3. Tandai commit lama yang diketahui masih berjalan normal (Good)
git bisect good v1.0.0

# Git akan checkout ke commit tengah secara otomatis.
# Jalankan pengujian/unit test Anda, lalu beri masukan ke Git:
git bisect good # Jika kode normal
# ATAU
git bisect bad  # Jika bug ditemukan

# 4. Setelah Git menemukan commit penyebab utama (First Bad Commit), akhiri sesi:
git bisect reset
```

---

## 7. Emergency Cheat Sheet

| Masalah / Skenario Krisis | Solusi Perintah CLI | Penjelasan Efek |
| --- | --- | --- |
| **Salah commit ke `main` (seharusnya di *feature branch*)** | `git reset --soft HEAD~1` `git switch -c feature/my-work` | Commit dibatalkan tanpa menghapus kode, lalu dipindahkan ke branch baru. |
| **Salah tulis pesan pada commit terakhir** | `git commit --amend -m "pesan baru"` | Timpa pesan commit terakhir (hanya jika belum di-push ke remote). |
| **Tidak sengaja `git reset --hard` & kehilangan kode** | `git reflog``git switch -c recovered <HASH>` | Cari hash commit sebelum reset di reflog, lalu buat branch pemulihan. |
| **Commit rusak/bug sudah terlanjur di-push ke server** | `git revert <COMMIT_ID>` | Buat commit balik aman yang membatalkan perubahan tanpa merusak histori publik. |
| **Lupa meletakkan file baru saat commit terakhir** | `git add file_tertinggal.js``git commit --amend --no-edit` | Masukkan file baru ke dalam commit terakhir tanpa mengubah pesan commit. |

---