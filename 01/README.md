# Praktikum Minggu 01
Laporan Praktikum Minggu 01

**Topik:** Pengenalan Git, GitHub, serta Konfigurasi Awal Lingkungan Pengembangan

## A. TUJUAN PRAKTIKUM
1. Memahami peranan Version Control System (VCS) dalam pengelolaan berkas kode sumber program.
2. Membedakan fungsi operasional antara sistem lokal Git dengan platform kolaborasi berbasis web GitHub.
3. Melakukan instalasi perangkat lunak Git pada sistem operasi Windows dengan pengaturan dependensi yang tepat.
4. Menyiapkan konfigurasi identitas dasar pengembang (username dan email) sebelum memulai commit.
5. Menginisiasi pembuatan repositori secara terpusat di GitHub maupun secara lokal di komputer kerja.
6. Membangun komunikasi data dua arah antara repositori lokal dan repositori remote.
7. Mempraktikkan siklus kerja dasar versioning: clone, add, commit, push, dan pull.
8. Menerapkan isolasi pengerjaan fitur baru menggunakan sistem percabangan (branching) agar berkas utama tidak mengalami kerusakan saat terjadi kesalahan penulisan program.
9. Memahami alur kerja kolaborasi berbasis izin akses, baik pada repositori personal, organisasi mahasiswa/tim, maupun pola kontribusi terbuka lewat mekanisme fork dan Pull Request (PR).
10. Menyelesaikan proses sinkronisasi berkas agar kondisi perubahan baris kode di penyimpanan lokal selalu selaras dengan repositori daring.

## B. DASAR TEORI

### 1. Konsep Dasar Git (Distributed Version Control System)
Dalam rekayasa perangkat lunak, pelacakan riwayat perubahan kode sumber merupakan kebutuhan krusial agar pengembangan sistem dapat diaudit kembali saat terjadi bug atau regresi performa. Git dirancang sebagai Distributed Version Control System (DVCS), artinya setiap pengembang menyimpan salinan riwayat repositori secara lengkap di mesin masing-masing, bukan sekadar mengambil berkas teranyar dari server terpusat.

Mekanisme internal Git bekerja dengan mencatat serangkaian snapshot keadaan berkas mini setiap kali perintah commit dieksekusi, bukan sekadar menyimpan daftar perbedaan teks (delta differences). Basis data lokal ini memungkinkan operasi pengecekan histori, pembuatan cabang kode baru, dan perbandingan perubahan berjalan tanpa membutuhkan koneksi internet aktif.

### 2. Platform GitHub dan Pola Kolaborasi Modern
Sementara Git menangani versioning di level mesin lokal, GitHub bertindak sebagai platform repositori cloud yang menyediakan infrastruktur kolaborasi berbasis jaringan. GitHub memadukan fungsionalitas Git dengan modul manajemen proyek terstruktur, seperti pelacakan kendala (issue tracking), peninjauan kualitas kode (code review), hingga persetujuan penggabungan cabang kode via mekanisme Pull Request.

Pola visibilitas repositori diatur ke dalam dua skema utama:
* **Public Repository:** Kode sumber dapat diakses, diinspeksi, dan di-clone oleh siapa saja di internet, umum digunakan untuk proyek sumber terbuka (open-source) atau materi perkuliahan bersama.
* **Private Repository:** Akses pembacaan maupun penulisan dibatasi hanya kepada pemilik akun dan pihak kolaborator terdaftar, lazim dipakai untuk pengerjaan tugas mandiri maupun proyek sistem yang belum siap dipublikasikan ke khalayak umum.

## C. PEMBAHASAN LISTING & TAHAPAN PRAKTIK

### PRAKTIK 1 – INSTALASI DAN KONFIGURASI GIT FOR WINDOWS
Pada praktikum minggu pertama ini, instalasi dilakukan pada sistem operasi Windows 64-bit menggunakan installer resmi Git for Windows versi 2.56.0. Seluruh tahapan konfigurasi disesuaikan dengan alur tangkapan layar praktikum berikut.

#### 1. Persiapan File Installer
<img width="658" height="88" alt="WhatsApp Image 2026-10-05 at 01 04 05" src="https://github.com/user-attachments/assets/433bbaf7-de3d-4bf6-b94d-dfd6a81c01e7" />


Langkah pertama diawali dengan mengunduh paket instalasi Git for Windows dari portal resmi git-scm.com. Hasil unduhan berupa berkas executable bernama `Git-2.56.0-64-bit.exe`. Pemasangan dimulai dengan membuka file installer tersebut.

#### 2. Persetujuan Lisensi (GNU General Public License)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.04.34.jpeg" width="700">

Jendela pertama menampilkan dokumen lisensi GNU General Public License versi 2 (Juni 1991). Bagian ini menerangkan aspek keterbukaan lisensi perangkat lunak Git sebelum pengguna melanjutkan ke langkah penentuan direktori berkas. Untuk melanjutkan, tombol **Next** ditekan.

#### 3. Pemilihan Direktori Instalasi (Destination Location)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.04.55.jpeg" width="700">

Lokasi pemasangan berkas program diarahkan ke direktori standar bawaan Windows, yaitu `C:\Program Files\Git`. Instalasi ini membutuhkan alokasi ruang penyimpanan minimal sekitar 341,5 MB pada partisi sistem. Setelah memastikan jalur folder sudah tepat, proses dilanjutkan dengan menekan **Next**.

#### 4. Pemilihan Komponen (Select Components)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.05.07.jpeg" width="700">

Pada menu opsi komponen fungsional yang akan dipasang ke sistem operasi, beberapa konfigurasi utama dicentang:

* **Windows Explorer integration:** Opsi *Open Git Bash here* dan *Open Git GUI here* diaktifkan agar terminal konsol Git dapat dipanggil secara langsung lewat klik kanan pada folder proyek mana pun di Windows Explorer.
* **Git LFS (Large File Support):** Mengaktifkan kapabilitas pelacakan berkas biner berukuran besar.
* **Asosiasi Ekstensi Berkas:** Memetakan berkas konfigurasi `.git*` ke teks editor bawaan serta mengaitkan skrip `.sh` agar dapat dieksekusi melalui Git Bash.
* **Scalar:** Mengaktifkan add-on manajemen repositori berskala besar.

Setelah verifikasi komponen selesai, klik **Next**.

#### 5. Pembuatan Folder Start Menu
<img src="WhatsApp%20Image%202026-10-05%20at%2001.05.21.jpeg" width="700">

Tahap ini menentukan letak pintasan program pada Start Menu Windows. Konfigurasi dibiarkan menggunakan nama default `Git` tanpa mencentang opsi penonaktifan folder, kemudian klik **Next**.

#### 6. Penentuan Teks Editor Default (Default Editor)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.05.47.jpeg" width="700">

Git memerlukan text editor eksternal untuk menangani pengisian pesan commit interaktif maupun resolusi konflik berkas secara manual. Pada menu drop-down, opsi yang dipilih adalah **Use Visual Studio Code as Git's default editor**. Pilihan ini diambil karena VS Code merupakan editor utama yang ringan, mudah digunakan untuk membaca kode program, dan memiliki tampilan visual pembanding baris kode (diff tool) yang nyaman bagi mahasiswa Informatika. Setelah memilih VS Code, klik **Next**.

#### 7. Penamaan Cabang Awal Repositori (Initial Branch Name)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.06.16.jpeg" width="700">

Pada pengaturan nama branch default saat perintah `git init` pertama kali dijalankan, dipilih opsi **Override the default branch name for new repositories** dengan memasukkan nama `main` ke dalam kolom teks. Langkah ini dilakukan agar repositori lokal selaras dengan konvensi repositori modern di GitHub yang sudah menjadikan `main` sebagai cabang standar pengganti penamaan lama `master`. Klik **Next** untuk beralih ke tahapan selanjutnya.

#### 8. Penyesuaian Environment PATH
<img src="WhatsApp%20Image%202026-10-05%20at%2001.06.29.jpeg" width="700">

Bagian ini mengatur integrasi perintah Git ke dalam variabel environment sistem operasi Windows. Opsi yang dipilih adalah:

**Git from the command line and also from 3rd-party software.**

Pilihan yang direkomendasikan ini menyuntikkan perintah `git` ke dalam system path secara fleksibel, sehingga Git dapat dipanggil tidak hanya melalui terminal Git Bash, melainkan juga lewat Command Prompt (CMD), Windows PowerShell, maupun terminal internal yang terpasang di dalam Visual Studio Code. Klik **Next**.

#### 9. Pemilihan Pustaka Transport HTTPS
<img src="WhatsApp%20Image%202026-10-05%20at%2001.07.46.jpeg" width="700">

Untuk mengamankan transmisi data repositori saat berkomunikasi dengan server remote via jalur protokol HTTPS, pada tangkapan layar ini dipilih opsi:

**Use the native Windows Secure Channel library.**

Konfigurasi ini menginstruksikan Git untuk memvalidasi sertifikat SSL/TLS menggunakan penyimpanan sertifikat bawaan Windows (Windows Certificate Stores), yang sangat berguna saat komputer terhubung dengan jaringan institusi atau kampus yang menerapkan filter sertifikat khusus. Klik **Next**.

#### 10. Konfigurasi Penanganan Baris Akhir Teks (Line Ending Conversions)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.08.05.jpeg" width="700">

Perbedaan format akhir baris (line ending) antara sistem Windows (CRLF) dan Unix/Linux (LF) dapat memicu konflik semu pada histori kode sumber. Pada opsi ini dipilih konfigurasi rekomendasi pertama:

**Checkout Windows-style, commit Unix-style line endings.**

Artinya, Git akan secara otomatis mengonversi karakter LF menjadi CRLF saat berkas di-checkout ke direktori kerja Windows, dan mengembalikannya ke format LF murni saat perubahan disimpan kembali (commit) ke repositori. Parameter internal `core.autocrlf` disetel pada nilai `true`. Klik **Next**.

#### 11. Pemilihan Terminal Emulator untuk Git Bash
<img src="WhatsApp%20Image%202026-10-05%20at%2001.08.21.jpeg" width="700">

Pada tahap penentuan antarmuka terminal pembaca konsol, dipilih opsi:

**Use MinTTY (the default terminal of MSYS2).**

MinTTY menyediakan jendela terminal yang fleksibel, mendukung pengaturan ukuran layar dinamis, seleksi blok teks non-persegi panjang, serta penanganan karakter font Unicode yang stabil untuk kebutuhan eksekusi instruksi berbasis Unix di lingkungan Windows. Klik **Next**.

#### 12. Penentuan Perilaku Default Perintah `git pull`
<img src="WhatsApp%20Image%202026-10-05%20at%2001.08.32.jpeg" width="700">

Ketika pengembang menarik pembaruan data dari repositori remote ke repositori lokal melalui perintah `git pull`, sistem perlu mengetahui strategi penggabungan berkas yang hendak diterapkan. Pada langkah ini dipilih opsi bawaan:

**Merge.**

Dengan opsi ini, Git akan membuat sebuah *merge commit* ketika menggabungkan cabang-cabang yang memiliki percabangan terpisah, sehingga integritas riwayat pekerjaan paralel tetap utuh tercatat pada log repositori. Klik **Next**.

#### 13. Pemilihan Credential Helper
<img src="WhatsApp%20Image%202026-10-05%20at%2001.09.01.jpeg" width="700">

Untuk memudahkan proses autentikasi akun GitHub tanpa harus berulang kali menginput nama pengguna dan token rahasia pada terminal, dipilih:

**Git Credential Manager.**

Aplikasi pembantu lintas platform ini akan mengamankan penyimpanan sesi login ke dalam sistem kredensial Windows secara terenkripsi. Klik **Next**.

#### 14. Konfigurasi Opsi Ekstra (Extra Options)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.09.15.jpeg" width="700">

Pada jendela fitur tambahan kinerja, dicentang opsi:

**Enable file system caching.**

Opsi `core.fscache = true` ini mempercepat pembacaan status berkas dan direktori proyek berukuran besar ke dalam memori kerja (RAM cache), sehingga respons perintah pelacakan seperti `git status` menjadi jauh lebih gesit. Opsi *symbolic links* dibiarkan nonaktif. Tombol **Install** kemudian ditekan untuk memulai eksekusi penulisan berkas sistem.

#### 15. Proses Pemasangan Berkas (Extracting Files)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.09.29.jpeg" width="700">

Program installer mulai mengekstrak seluruh pustaka, binari utilitas Git, serta dependensi pendukung seperti `git-lfs.exe` ke direktori tujuan yang telah ditentukan. Bilah kemajuan (progress bar) ditunggu hingga seluruh proses selesai sempurna.

#### 16. Penyelesaian Instalasi
<img src="WhatsApp%20Image%202026-10-05%20at%2001.10.47.jpeg" width="700">

Setelah proses transfer data berakhir, jendela *Completing the Git Setup Wizard* muncul menandakan Git telah sukses terpasang pada komputer. Opsi centang *View Release Notes* dibiarkan aktif bila ingin membaca catatan perubahan versi, lalu tombol **Finish** ditekan untuk menutup jendela instalasi secara tuntas.

#### 17. Uji Verifikasi Pemanggilan Git pada Command Prompt
<img src="WhatsApp%20Image%202026-10-05%20at%2001.17.06.jpeg" width="700">

Setelah proses instalasi selesai, tahap pengujian dilakukan guna memastikan berkas binari Git telah terdaftar dengan benar di dalam *environment variable* sistem operasi Windows dan dapat diakses secara global. Jendela terminal Command Prompt (CMD) dibuka, kemudian dieksekusi instruksi:

```bash
git
```

**Analisis Output:**

Ketika perintah `git` dijalankan tanpa argumen tambahan, konsol langsung merespons dengan menampilkan daftar parameter penggunaan (*usage format*) beserta ringkasan perintah dasar (*common Git commands*). Perintah tersebut dikelompokkan ke dalam beberapa fungsi kerja utama, antara lain:

* *Start a working area* (`clone`, `init`) untuk membuat atau mengambil repositori.
* *Work on the current change* (`add`, `mv`, `restore`, `rm`) untuk mengatur berkas ke *staging area*.
* *Examine the history and state* (`status`, `diff`, `log`, `show`) untuk memantau kondisi dan riwayat berkas.
* *Grow, mark and tweak your common history* (`branch`, `commit`, `merge`, `rebase`, `switch`, `tag`) untuk manajemen cabang dan pencatatan komit.
* *Collaborate* (`fetch`, `pull`, `push`) untuk sinkronisasi dengan server remote.

Respons ini membuktikan bahwa konfigurasi penyesuaian PATH (*Git from the command line and also from 3rd-party software*) berhasil diimplementasikan, sehingga terminal mengenali perintah `git` tanpa memicu pesan galat *command not recognized*.

#### 18. Pemeriksaan Versi Git Terpasang (`git --version`)
<img src="WhatsApp%20Image%202026-10-05%20at%2001.17.29.jpeg" width="700">

Untuk mengonfirmasi nomor rilis paket perangkat lunak Git yang aktif dan terpasang pada komputer lokal, dijalankan perintah spesifik berikut pada terminal CMD:

```bash
git --version
```

Output ini memvalidasi bahwa sistem telah berhasil memasang **Git for Windows versi 2.56.0** (arsitektur 64-bit) sesuai dengan berkas installer yang diunduh sebelumnya. Dengan munculnya informasi versi tersebut, lingkungan pengembangan lokal dinyatakan siap digunakan untuk konfigurasi identitas global (`git config`) serta pengerjaan repositori tugas praktikum selanjutnya.

#### 19. Konfigurasi Identitas Pengguna (`git config --global user.name`)
<img src="WhatsApp%20Image%202026-10-06%20at%2014.07.38.jpeg" width="700">

Sebelum melakukan commit pertama, identitas pengembang perlu didaftarkan ke Git. Nama pengguna diatur secara global pada terminal CMD dengan perintah:

```bash
git config --global user.name "liawww"
```

Konfigurasi global ini berlaku untuk seluruh repositori di komputer tersebut dan akan tercantum sebagai penulis (*author*) pada setiap commit.
