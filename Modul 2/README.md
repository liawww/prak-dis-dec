# Praktikum Minggu 02

Laporan Praktikum Minggu 02

**Topik:** Komunikasi Antar Proses pada Sistem Terdistribusi

## A. TUJUAN PRAKTIKUM

1. Memahami apa itu proses dan bagaimana proses dikelola oleh sistem operasi.
2. Mampu menampilkan, menjalankan, dan mematikan proses pada komputer sendiri.
3. Memahami perbedaan komunikasi antar proses pada satu node dengan komunikasi antar proses pada sistem terdistribusi.
4. Membuat GraphQL server menggunakan Python dengan paket Strawberry.
5. Membuat client yang dapat mengakses GraphQL server tersebut.

## B. DASAR TEORI

### 1. Proses

Proses adalah hasil dari menjalankan sebuah program atau aplikasi. Setiap kali kita membuka aplikasi, sistem operasi membuat satu proses baru. Proses ini terdiri dari kode program yang dijalankan (*executable code*), data, *resources* yang dipakai, dan informasi tentang kondisi proses (*state*) seperti *stack* dan *heap*. Semua proses dikelola oleh sistem operasi, mulai dari pembagian memori dan CPU sampai komunikasi antar proses. Pengguna biasanya tidak perlu tahu karena semuanya berjalan di belakang layar.

### 2. Komunikasi Antar Proses pada Satu Node

Pada satu komputer (satu node), semua proses berada di bawah kendali satu sistem operasi. Semua proses memakai *clock* yang sama dan bisa memakai *shared memory*. Karena itu urutan kerja proses mudah dikendalikan dan tidak perlu sinkronisasi yang rumit, selama sistem operasinya mengelola proses dengan baik.

### 3. Komunikasi Antar Proses pada Sistem Terdistribusi

Pada sistem terdistribusi, proses berjalan di lebih dari satu node. Setiap node punya *clock* sendiri dan mengelola memorinya sendiri. Node yang satu tidak boleh mengakses memori node lain karena alasan keamanan, sehingga *shared memory* tidak bisa dipakai. Karena itu dibutuhkan cara khusus agar proses di node yang berbeda bisa saling berkomunikasi, misalnya dengan mengirim pesan lewat jaringan.

### 4. GraphQL dan Strawberry

GraphQL adalah spesifikasi yang bisa dipakai untuk komunikasi antara *client* (yang meminta layanan) dan *server* (yang melayani permintaan). Client menulis *query* untuk menyebutkan data apa saja yang dibutuhkan, lalu server mengirim kembali data tepat sesuai query itu. Keuntungannya, client dan server bisa dibuat dengan bahasa pemrograman yang berbeda. Pada praktikum ini server dibuat dengan Python menggunakan paket **Strawberry**, sedangkan *client* dibuat dengan Python juga.

## C. PEMBAHASAN & TAHAPAN PRAKTIK

### PRAKTIK 1 – Proses pada Windows

#### 1. Menampilkan proses di komputer

Untuk melihat proses yang sedang berjalan di komputer, saya membuka **Task Manager** dengan menekan tombol `Ctrl + Shift + Esc`. Di tab *Processes* terlihat daftar aplikasi yang sedang dibuka (*Apps*) dan proses yang berjalan di latar belakang (*Background processes*), lengkap dengan penggunaan CPU, memori, disk, dan jaringan masing-masing. Dari sini saya jadi tahu bahwa walaupun saya hanya membuka beberapa aplikasi, sebenarnya ada banyak proses lain yang dijalankan oleh sistem operasi tanpa saya sadari.

![Daftar proses di Task Manager](01-task-manager.png)

#### 2. Menjalankan satu aplikasi dan melihat prosesnya

Saya menjalankan satu aplikasi, yaitu **Notepad**, lalu kembali ke Task Manager untuk mencari prosesnya. Setelah Notepad dibuka, muncul entri baru bernama Notepad di daftar *Apps*, lengkap dengan penggunaan CPU dan memorinya. Ini menunjukkan bahwa setiap aplikasi yang dijalankan akan menjadi proses yang dikelola oleh sistem operasi, sesuai dengan teori di modul.

![Proses Notepad di Task Manager](02-proses-notepad.png)

#### 3. Mematikan proses lewat perintah (bukan tombol close)

Untuk mematikan proses tanpa memakai tombol *close* atau perintah keluar dari aplikasi, saya memakai perintah di Command Prompt. Pertama saya cek dulu proses Notepad dengan `tasklist | findstr notepad`, lalu mematikannya dengan `taskkill /IM notepad.exe /F`. Setelah perintah dijalankan, jendela Notepad langsung tertutup dan prosesnya hilang dari Task Manager. Cara lain yang hasilnya sama adalah klik kanan proses di Task Manager lalu memilih *End task*. Untuk me-*restart* proses, caranya adalah mematikan prosesnya terlebih dahulu, kemudian menjalankan aplikasinya kembali sehingga muncul proses baru.

```
tasklist | findstr notepad
taskkill /IM notepad.exe /F
```

![Mematikan proses dengan taskkill](03-kill-proses.png)

### PRAKTIK 2 – GraphQL Server dengan Strawberry

#### 1. Mempelajari uv

Langkah pertama adalah mempelajari **uv**, yaitu alat untuk mengelola proyek, versi Python, dan paket Python dengan cepat. Saya membaca catatan yang diberikan di repositori NEO-X-School untuk tahu perintah dasarnya. Jika uv belum ada di komputer, saya memasangnya dengan perintah di bawah, lalu memastikan uv terpasang dengan mengecek versinya.

```
winget install astral-sh.uv
uv --version
```

![Versi uv](04-uv-version.png)

#### 2. Membuat workspace dan menentukan versi Python

Saya membuat workspace dengan nama `workspace-01` memakai uv, lalu masuk ke folder tersebut. Workspace ini adalah folder tempat semua pekerjaan praktikum diletakkan supaya rapi dan terpisah dari proyek lain. Di dalamnya saya menentukan memakai Python versi 3.14, sesuai petunjuk modul, dan uv akan mengunduhnya sendiri jika belum ada di komputer.

```
uv init workspace-01
cd workspace-01
uv python install 3.14
```

![Membuat workspace](05-workspace.png)

#### 3. Membuat dan mengaktifkan environment

Selanjutnya saya membuat *virtual environment*, yaitu lingkungan Python yang terpisah untuk proyek ini. Tujuannya agar paket yang dipasang untuk praktikum ini tidak bercampur dengan paket lain di komputer. Setelah dibuat, environment harus diaktifkan; tandanya, di awal baris terminal muncul tulisan `(workspace-01)` atau `(.venv)`.

```
uv venv --python 3.14
.venv\Scripts\activate
```

![Environment aktif](06-environment.png)

#### 4. Instalasi paket yang diperlukan

Saya memasang paket **Strawberry GraphQL** beserta fitur CLI-nya ke dalam environment tadi. Perintah ini juga ikut memasang paket-paket pendukung seperti `graphql-core`, `starlette`, dan `uvicorn` (server web untuk menjalankan aplikasi). Setelah selesai, terminal menampilkan daftar paket yang berhasil terpasang.

```
uv pip install "strawberry-graphql[cli]"
```

![Instalasi Strawberry](07-install-strawberry.png)

#### 5. Menjalankan server dari schema.py

Saya menyiapkan file `schema.py` di folder workspace. File ini berisi definisi data dan cara server menjawab permintaan: ada tipe `Book` dengan atribut `title` dan `author`, fungsi yang mengembalikan daftar buku, dan `Query` bernama `books` yang dapat dipanggil oleh client. Server dijalankan dengan `strawberry dev schema`, dan terminal menampilkan bahwa server aktif di alamat `http://0.0.0.0:8000/graphql`. Selama server berjalan, terminal ini jangan ditutup.

```
strawberry dev schema
```

![Server berjalan](08-server-jalan.png)

Setelah itu saya membuka alamat `http://localhost:8000/graphql` di browser. Muncul tampilan GraphiQL, yaitu antarmuka untuk mengirim query ke server. Bagian kiri untuk menulis query dan bagian kanan menampilkan hasilnya.

![Tampilan GraphiQL di browser](09-graphiql.png)

#### 6. Menuliskan query

Pada sisi kiri GraphiQL saya menulis query berikut. Query ini meminta server mengirim daftar `books`, dan dari setiap buku saya hanya meminta field `title` dan `author`. Inilah ciri khas GraphQL: client bisa menentukan sendiri data apa yang diminta, tidak lebih dan tidak kurang.

```
{
  books {
    title
    author
  }
}
```

![Query di GraphiQL](10-query.png)

#### 7. Menjalankan query dan melihat hasil

Saya menekan tombol *run* (segitiga pink) untuk mengirim query ke server. Hasilnya muncul di sisi kanan dalam format JSON, berisi data buku "The Great Gatsby" karya F. Scott Fitzgerald. Ini membuktikan bahwa komunikasi client dan server berhasil: browser sebagai client mengirim permintaan, dan server Strawberry yang berjalan sebagai proses terpisah menjawab dengan data yang sesuai.

![Hasil query](11-hasil-query.png)

#### 8. Mematikan server

Setelah selesai, server dimatikan dengan menekan `Ctrl + C` pada terminal tempat `strawberry dev schema` dijalankan, sehingga proses server berhenti dan alamat `localhost:8000` tidak bisa diakses lagi. *(Catatan: langkah ini dilakukan setelah tugas client di bawah selesai, karena client membutuhkan server yang masih menyala.)*

![Server dimatikan](12-server-mati.png)

### PRAKTIK 3 – Membuat Client untuk GraphQL Server

Pada tugas terakhir saya membuat sebuah client memakai bahasa Python, yaitu file `client.py`. Client ini tidak memakai browser, tetapi langsung mengirim permintaan HTTP `POST` ke alamat `http://localhost:8000/graphql` dengan isi berupa query yang sama seperti sebelumnya. Server membalas dengan JSON, lalu client membacanya dan menampilkan daftar buku di terminal. Client memakai pustaka bawaan Python (`urllib` dan `json`), jadi tidak perlu memasang paket tambahan. Supaya bisa berjalan, server harus dinyalakan dulu di satu terminal, kemudian client dijalankan di terminal lain.

```
python client.py
```

Kode lengkap ada pada file [client.py](client.py).

![Hasil client](13-hasil-client.png)

## D. KESIMPULAN

Dari praktikum ini saya belajar bahwa setiap aplikasi yang dijalankan menjadi proses yang dikelola sistem operasi dan bisa dilihat serta dimatikan lewat Task Manager maupun perintah. Saya juga memahami bahwa pada sistem terdistribusi proses tidak bisa berbagi memori, sehingga komunikasi dilakukan lewat jaringan, salah satunya memakai GraphQL. Dengan Strawberry saya berhasil membuat server GraphQL sederhana, mengaksesnya lewat browser, dan membuat client Python yang mengambil data dari server tersebut.
