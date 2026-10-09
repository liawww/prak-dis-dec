# Praktikum Minggu 02

Laporan Praktikum Minggu 02

**Topik:** Komunikasi Antar Proses pada Sistem Terdistribusi (Manajemen Proses, GraphQL Server dengan Strawberry, dan Client GraphQL)

## A. TUJUAN PRAKTIKUM

1. Memahami pengertian proses sebagai hasil eksekusi program yang dikelola oleh sistem operasi.
2. Menampilkan dan mengamati daftar proses yang berjalan pada sistem operasi Windows maupun Linux (WSL Ubuntu).
3. Mengidentifikasi proses yang dimunculkan oleh sebuah aplikasi yang sedang dijalankan beserta Process ID (PID)-nya.
4. Mempraktikkan cara mematikan proses melalui perintah, tanpa menggunakan perintah keluar dari aplikasi yang bersangkutan.
5. Memahami dan mempraktikkan cara me-*restart* proses.
6. Membedakan karakteristik komunikasi antar proses pada satu node dengan komunikasi antar proses pada sistem terdistribusi.
7. Menyiapkan lingkungan pengembangan Python menggunakan `uv`, meliputi pembuatan workspace, penentuan versi Python, pembuatan environment, dan instalasi paket.
8. Membangun GraphQL server menggunakan paket Strawberry pada Python dan mengujinya melalui antarmuka GraphiQL.
9. Membuat program client yang mengakses GraphQL server melalui permintaan HTTP.
10. Memahami bahwa komunikasi antara client dan server merupakan bentuk komunikasi antar proses yang dilakukan melalui jaringan, bukan melalui *shared memory*.

## B. DASAR TEORI

### 1. Konsep Proses dan Process ID (PID)

Proses merupakan hasil dari eksekusi program atau aplikasi yang bersifat *executable*. Proses dikelola oleh sistem operasi dan terdiri atas *executable code*, data, *resources*, serta informasi tentang *state* yang berupa *stack* dan *heap*. Setiap aplikasi yang dijalankan akan menjadi proses, dan satu aplikasi dapat memunculkan lebih dari satu proses sekaligus.

Setiap proses yang berjalan diberi identitas unik oleh sistem operasi yang disebut **Process ID (PID)**. PID digunakan sebagai acuan ketika sistem operasi maupun pengguna hendak memantau, menghentikan, atau mengatur sebuah proses. Pada Linux, proses pertama yang dijalankan saat sistem hidup memiliki PID 1 (umumnya `init` atau `systemd`) dan menjadi induk dari proses-proses lainnya.

### 2. Pengelolaan Proses pada Satu Node

Pada satu node (satu komputer), seluruh aspek yang berkaitan dengan proses berada dalam kendali sistem operasi, mulai dari eksekusi program menjadi proses, alokasi *resources*, pengelolaan proses, hingga komunikasi antar proses. Seluruh mekanisme ini bersifat transparan terhadap pengguna, artinya pengguna tidak perlu melihat prosesnya karena di latar belakang semuanya telah dikelola oleh sistem operasi.

Untuk mengamati proses yang berjalan, tersedia perangkat bawaan maupun tambahan sesuai sistem operasi. Pada Windows digunakan **Task Manager**, sedangkan pada Linux digunakan **htop** yang menampilkan daftar proses secara interaktif beserta penggunaan CPU dan memori.

### 3. Sinyal pada Linux

Pada Linux, pengendalian proses dari luar dilakukan dengan cara mengirimkan **sinyal (*signal*)** ke proses tersebut. Beberapa sinyal yang umum digunakan:

- **SIGTERM (15):** permintaan agar proses berhenti secara baik-baik, sehingga proses masih sempat membersihkan sumber dayanya. Sinyal ini adalah bawaan perintah `kill` dan `pkill`.
- **SIGKILL (9):** pemaksaan penghentian proses oleh sistem operasi yang tidak dapat diabaikan oleh proses.
- **SIGHUP (1):** sinyal *hang up* yang pada sejumlah program digunakan sebagai tanda agar membaca ulang konfigurasi (*reload*).

Perintah `kill <PID>` mengirim sinyal berdasarkan PID, sedangkan `pkill <nama>` mengirim sinyal berdasarkan nama proses. Pada `htop`, pengiriman sinyal dilakukan melalui tombol **F9 (Kill)**.

### 4. Windows Subsystem for Linux (WSL)

WSL adalah fitur Windows yang memungkinkan lingkungan Linux dijalankan langsung di dalam Windows tanpa perlu menggunakan mesin virtual terpisah atau *dual boot*. Pada praktikum ini WSL digunakan dengan distribusi Ubuntu agar proses pada sistem operasi Linux dapat diamati menggunakan `htop`.

### 5. Komunikasi Antar Proses pada Sistem Terdistribusi

Komunikasi antar proses pada satu node relatif sederhana karena semua dikendalikan dan dikelola oleh sistem operasi. Semua proses berjalan pada *clock* yang sama dan dapat memanfaatkan *shared memory*, sehingga seluruh proses terurut dan terkendali dengan baik selama sistem operasi mengelola proses dengan baik.

Kondisi tersebut berbeda pada sistem terdistribusi yang terdiri atas lebih dari satu node:

- Antar node **tidak berada pada *clock* yang sama**.
- **Tidak memungkinkan penggunaan *shared memory***, karena setiap node mengelola memorinya sendiri dan tidak diperbolehkan mengakses memori node lain demi alasan keamanan.

Oleh karena itu, diperlukan cara khusus agar proses yang berada pada node berbeda dapat saling berkomunikasi, umumnya melalui pertukaran pesan lewat jaringan. Salah satu cara yang dapat digunakan adalah **GraphQL**.

### 6. GraphQL

GraphQL adalah spesifikasi *query language* untuk API. Dengan GraphQL, dapat dibuat server yang melayani *query* dari *client* sesuai spesifikasi tersebut, sehingga terjadi komunikasi antar proses antara client (pihak yang meminta layanan) dan server (pihak yang melayani permintaan). Karena komunikasi berlandaskan spesifikasi, peranti pengembangan pada sisi client dan server boleh berbeda.

Beberapa karakteristik penting GraphQL:

- **Schema bertipe:** server mendefinisikan tipe data dan *query* yang tersedia.
- **Client menentukan data yang diminta:** client menuliskan field yang dibutuhkan saja, dan server hanya mengembalikan field tersebut.
- **Resolver:** fungsi pada sisi server yang bertugas mengambil data untuk field yang diminta.
- **Satu endpoint:** seluruh permintaan dikirim ke satu alamat yang sama, yaitu `/graphql` pada praktikum ini.

### 7. Strawberry dan uv

**Strawberry** adalah pustaka Python untuk membuat GraphQL server dengan pendekatan *code-first*, yaitu schema ditulis menggunakan *class* Python dan *type hint*. Strawberry menyediakan perintah `strawberry dev` untuk menjalankan server pengembangan beserta antarmuka **GraphiQL**, yaitu UI berbasis browser untuk menulis dan menguji query.

**uv** adalah alat untuk mengelola proyek, versi Python, *virtual environment*, dan paket Python dengan kecepatan tinggi. Pada praktikum ini uv digunakan untuk menyiapkan seluruh lingkungan pengembangan server.

## C. PEMBAHASAN LISTING & TAHAPAN PRAKTIK

### PRAKTIK 1 – MANAJEMEN PROSES PADA WINDOWS DAN LINUX (WSL UBUNTU)

Pada praktik pertama, proses diamati pada dua sistem operasi, yaitu Windows menggunakan Task Manager dan Linux (Ubuntu pada WSL) menggunakan `htop`. Dengan demikian, cara kedua sistem operasi menampilkan dan mengelola proses dapat dibandingkan.

#### 1. Menampilkan Proses pada Windows (Task Manager)

![Task Manager](01-task-manager.png)

Task Manager dibuka dengan kombinasi tombol `Ctrl + Shift + Esc`, kemudian dipilih tab **Processes**. Pada tab ini ditampilkan berbagai proses yang sedang berjalan, lengkap dengan penggunaan CPU, memori, disk, jaringan, dan konsumsi daya (*power usage*). Daftar diurutkan berdasarkan penggunaan CPU tertinggi.

**Analisis Output:**

- Saat pengamatan, penggunaan CPU tercatat 67%, memori 78%, disk 3%, dan jaringan 0%.
- Proses dengan penggunaan CPU tertinggi adalah `rsEngineSvc` (35,3%), disusul `Desktop Window Manager` (9,7%) yang mengelola tampilan jendela.
- Proses `Google Chrome (19)` menggunakan memori terbesar, yaitu 698,8 MB. Angka (19) menunjukkan bahwa satu aplikasi Chrome memunculkan beberapa proses sekaligus.
- Tampak pula berbagai proses sistem seperti `System`, `Windows Explorer`, `Runtime Broker`, `CTF Loader`, dan beberapa `Service Host`, yang berjalan di latar belakang tanpa disadari pengguna.
- Banyaknya proses yang tampil menunjukkan bahwa sistem operasi mengelola banyak proses secara bersamaan, sesuai dengan sifat transparan pengelolaan proses pada satu node.

#### 2. Menyiapkan Linux dengan WSL dan Ubuntu

![Welcome to WSL](02-wsl-welcome.png)

![Akun Ubuntu](03-ubuntu-akun.png)

Agar proses pada Linux dapat diamati, digunakan WSL. Setelah WSL terpasang dan jendela *Welcome to WSL* muncul, Ubuntu dijalankan. Pada pembukaan pertama, Ubuntu meminta pembuatan akun pengguna sehingga diisi nama pengguna dan password, kemudian dipilih `y` pada pertanyaan pengumpulan data metrik. Setelah tahap ini selesai, terminal Ubuntu siap digunakan.

#### 3. Memasang `htop` dan Menampilkan Proses pada Linux

![Instalasi htop](04-htop-install.png)

![Tampilan htop](05-htop-proses.png)

Pada terminal Ubuntu dijalankan perintah `htop`, namun muncul pesan bahwa perintah tersebut belum terpasang. Oleh karena itu, daftar paket diperbarui dan `htop` dipasang, lalu dijalankan kembali:

```
htop
sudo apt update
sudo apt install htop
htop
```

**Analisis Output:**

- `htop` menampilkan daftar proses lengkap dengan PID, pengguna (*USER*), penggunaan CPU dan memori, serta perintah yang dijalankan.
- Pada bagian atas terlihat penggunaan tiap *core* CPU, penggunaan memori (sekitar 325 MB dari 1,81 GB), dan jumlah proses (*Tasks*).
- Proses pertama adalah `/sbin/init` dengan PID 1, yaitu proses induk yang menjalankan proses-proses lainnya pada sistem Linux.

#### 4. Menjalankan Aplikasi dan Melihat Proses yang Dimunculkan

![Nano berjalan](06-nano-jalan.png)

![Proses nano pada htop](07-nano-di-htop.png)

Aplikasi yang dijalankan adalah editor teks **nano** dengan membuka berkas bernama `tugas`:

```
nano tugas
```

Setelah nano terbuka pada satu jendela terminal, dibuka jendela terminal Ubuntu kedua, lalu dijalankan `htop` untuk memeriksa daftar proses.

**Analisis Output:**

- Pada daftar proses `htop` muncul baris baru dengan perintah `nano tugas` dan PID **961**.
- Hal ini membuktikan bahwa setiap aplikasi yang dijalankan akan menjadi proses yang tercatat dan dikelola oleh sistem operasi dengan PID tertentu.

#### 5. Mematikan Proses Melalui Perintah (tanpa Keluar dari Aplikasi)

![Kill melalui htop](08-kill-htop-f9.png)

![pkill nano](09-pkill-nano.png)

Proses nano dimatikan dari luar aplikasi, bukan dengan perintah keluar bawaan nano (`Ctrl + X`). Terdapat dua cara yang dicoba:

**a. Melalui `htop`.** Proses nano dipilih pada daftar, kemudian ditekan tombol **F9 (Kill)**, dipilih sinyal **SIGTERM** (sinyal 15), dan ditekan Enter.

**b. Melalui perintah `pkill`.** Proses dimatikan berdasarkan namanya:

```
pkill nano
```

Padanan lain yang dapat digunakan adalah mematikan berdasarkan PID, yaitu `kill 961`.

**Analisis Output:**

- Pada jendela nano muncul pesan `Received SIGHUP or SIGTERM`, kemudian aplikasi langsung tertutup.
- Hal ini menandakan bahwa proses dihentikan oleh sistem operasi melalui sinyal, bukan karena pengguna keluar dari aplikasi.
- Proses nano tidak lagi muncul pada daftar `htop` setelah perintah dijalankan.

#### 6. Me-*restart* Proses

![Restart nano](10-restart-nano.png)

Pada Linux tidak terdapat satu perintah khusus untuk me-*restart* proses aplikasi biasa. Restart dilakukan dengan mematikan proses terlebih dahulu, kemudian menjalankannya kembali:

```
pkill nano
nano tugas
```

Untuk jenis proses lain, tersedia cara yang menyesuaikan sifat proses tersebut:

- Program yang mendukung *reload* konfigurasi dapat diminta membaca ulang konfigurasinya tanpa dimatikan dengan mengirim sinyal SIGHUP: `kill -HUP <PID>`.
- Proses yang berjalan sebagai *service* dapat di-*restart* dengan `sudo systemctl restart <nama_service>`.

**Analisis Output:**

- Setelah `nano tugas` dijalankan kembali, `htop` menampilkan PID yang berbeda dari sebelumnya (PID 961 menjadi [PID baru]).
- Hal ini menunjukkan bahwa *restart* sebenarnya membuat proses baru dengan PID baru, bukan menghidupkan kembali proses yang sama.

#### 7. Ringkasan Pekerjaan Praktik 1

Secara berurutan, pekerjaan pada Praktik 1 adalah menampilkan proses pada Windows melalui Task Manager, menyiapkan Ubuntu pada WSL, memasang dan menjalankan `htop`, menjalankan aplikasi nano dan menemukan prosesnya (PID 961), mematikan proses tersebut melalui `htop` dan `pkill` tanpa keluar dari aplikasi, serta me-*restart* proses dengan menjalankan kembali aplikasinya. Praktik ini menunjukkan bahwa pada satu node seluruh siklus hidup proses ditangani oleh sistem operasi.

---

### PRAKTIK 2 – GRAPHQL SERVER MENGGUNAKAN STRAWBERRY

#### 1. Mempelajari dan Memasang `uv`

![Versi uv](11-uv-version.png)

Langkah pertama adalah mempelajari penggunaan `uv` berdasarkan catatan pada repositori NEO-X-School yang disebutkan di modul. Apabila `uv` belum tersedia, pemasangan dilakukan melalui `winget`, kemudian dilakukan pengecekan versi untuk memastikan `uv` terpasang dengan benar:

```
winget install astral-sh.uv
uv --version
```

#### 2. Membuat Workspace dan Menentukan Versi Python

![Membuat workspace](12-workspace.png)

Workspace dengan nama `workspace-01` dibuat sebagai folder kerja agar pekerjaan praktikum tertata dan terpisah dari proyek lain. Pada workspace tersebut digunakan Python versi 3.14 sesuai petunjuk modul. `uv` akan mengunduh Python tersebut apabila belum tersedia pada komputer.

```
uv init --python 3.14 workspace-01
cd workspace-01
uv python install 3.14
```

Opsi `--python 3.14` menetapkan versi Python proyek sehingga pemilihan versi tidak bergantung pada Python bawaan sistem.

#### 3. Membuat dan Mengaktifkan Environment

![Environment aktif](13-environment.png)

*Virtual environment* dibuat agar paket yang dipasang untuk praktikum ini tidak bercampur dengan paket lain di komputer, lalu diaktifkan:

```
uv venv --python 3.14
.venv\Scripts\activate
```

**Analisis Output:**

Setelah environment aktif, awal baris terminal diberi tambahan nama environment (`(workspace-01)` atau `(.venv)`). Hal ini menandakan paket yang dipasang berikutnya hanya berlaku di dalam environment ini.

#### 4. Instalasi Paket yang Diperlukan

![Instalasi Strawberry](14-install-strawberry.png)

Paket Strawberry GraphQL beserta fitur CLI-nya dipasang ke dalam environment dengan perintah berikut (pada Windows digunakan tanda petik ganda):

```
uv pip install "strawberry-graphql[cli]"
```

**Analisis Output:**

- `uv` me-*resolve* dan memasang 24 paket sekaligus, di antaranya `strawberry-graphql`, `graphql-core`, `starlette`, `uvicorn`, `websockets`, `click`, `typer`, dan `rich`.
- Paket `graphql-core` menyediakan implementasi inti spesifikasi GraphQL, `starlette` dan `uvicorn` menyediakan lapisan server web, sedangkan `typer`, `click`, dan `rich` mendukung perintah `strawberry` pada terminal.
- Proses instalasi berlangsung sangat cepat karena `uv` memanfaatkan *cache*.

#### 5. Menyiapkan `schema.py`

Berkas `schema.py` yang telah disediakan diletakkan pada folder `workspace-01`. Isi berkas ini kurang lebih sebagai berikut:

```python
import typing

import strawberry


@strawberry.type
class Book:
    title: str
    author: str


def get_books():
    return [
        Book(
            title="The Great Gatsby",
            author="F. Scott Fitzgerald",
        ),
    ]


@strawberry.type
class Query:
    books: typing.List[Book] = strawberry.field(resolver=get_books)


schema = strawberry.Schema(query=Query)
```

**Penjelasan Kode:**

- `class Book` yang diberi dekorator `@strawberry.type` mendefinisikan tipe data buku dengan dua field, yaitu `title` dan `author`.
- Fungsi `get_books()` berperan sebagai *resolver* yang mengembalikan daftar buku.
- `class Query` adalah tipe akar (*root*) yang memuat query `books`, yang dihubungkan dengan resolver `get_books`.
- `strawberry.Schema(query=Query)` membentuk schema akhir yang akan dilayani oleh server.

#### 6. Menjalankan Server

![Server berjalan](15-server-jalan.png)

Server dijalankan dengan perintah:

```
strawberry dev schema
```

**Analisis Output:**

Terminal menampilkan bahwa server aktif pada alamat `http://0.0.0.0:8000/graphql`. Artinya server GraphQL mendengarkan permintaan pada port 8000 dengan *endpoint* `/graphql`. Selama server berjalan, terminal ini tidak boleh ditutup.

#### 7. Mengakses GraphiQL melalui Browser

![Tampilan GraphiQL](16-graphiql.png)

Browser dibuka, kemudian diakses alamat `http://localhost:8000/graphql`. Muncul antarmuka **GraphiQL** yang terbagi menjadi dua bagian, yaitu sisi kiri untuk menuliskan query dan sisi kanan untuk menampilkan hasil query. Tombol *run* (segitiga merah muda) digunakan untuk menjalankan query.

#### 8. Menuliskan dan Menjalankan Query

![Query pada GraphiQL](17-query.png)

Pada sisi kiri GraphiQL dituliskan query berikut. Query ini meminta server mengirim daftar `books`, dan dari setiap buku hanya diminta field `title` dan `author`:

```graphql
{
  books {
    title
    author
  }
}
```

![Hasil query](18-hasil-query.png)

Setelah tombol *run* ditekan, hasil ditampilkan pada sisi kanan dalam format JSON:

```json
{
  "data": {
    "books": [
      {
        "title": "The Great Gatsby",
        "author": "F. Scott Fitzgerald"
      }
    ]
  }
}
```

**Analisis Output:**

- Struktur hasil JSON sama dengan query yang ditulis: hanya field `title` dan `author` yang dikembalikan, berada di dalam objek `data` > `books`.
- Hal ini membuktikan bahwa client menentukan sendiri bentuk data yang diminta, sedangkan server memenuhinya melalui resolver.
- Browser bertindak sebagai client yang mengirim permintaan, sementara server Strawberry yang berjalan sebagai proses terpisah menjawab dengan data yang sesuai. Inilah bentuk komunikasi antar proses melalui jaringan.

---

### PRAKTIK 3 – MEMBUAT CLIENT UNTUK GRAPHQL SERVER

#### 1. Membuat Program Client

Tugas terakhir adalah membuat client menggunakan bahasa pemrograman bebas yang mengakses GraphQL server di atas. Bahasa yang dipilih adalah **Python**, dengan modul bawaan `urllib` dan `json` sehingga tidak diperlukan instalasi paket tambahan. Client dibuat pada berkas `client.py`:

```python
import json
import urllib.error
import urllib.request

URL = "http://localhost:8000/graphql"

QUERY = """
{
  books {
    title
    author
  }
}
"""


def kirim_query(query: str) -> dict:
    """Mengirim query GraphQL ke server lewat HTTP POST dan mengembalikan hasilnya."""
    payload = json.dumps({"query": query}).encode("utf-8")
    request = urllib.request.Request(
        URL,
        data=payload,
        headers={"Content-Type": "application/json"},
        method="POST",
    )
    with urllib.request.urlopen(request, timeout=10) as response:
        return json.loads(response.read().decode("utf-8"))


def main():
    try:
        hasil = kirim_query(QUERY)
    except urllib.error.URLError as error:
        print(f"Gagal terhubung ke server: {error}")
        return

    if "errors" in hasil:
        print("Server mengembalikan error:", hasil["errors"])
        return

    print("Daftar buku dari GraphQL server:")
    for nomor, buku in enumerate(hasil["data"]["books"], start=1):
        print(f"{nomor}. {buku['title']} - {buku['author']}")


if __name__ == "__main__":
    main()
```

**Penjelasan Kode:**

- Query GraphQL dikirim sebagai JSON berbentuk `{"query": "..."}` dengan metode **HTTP POST** ke endpoint `/graphql`.
- Respons server dibaca dan di-*parse* dari JSON menjadi *dictionary* Python.
- Program memeriksa key `errors` untuk menangani kesalahan dari sisi GraphQL, serta menangkap `URLError` apabila server belum dijalankan.
- Data pada `hasil["data"]["books"]` ditampilkan satu per satu pada terminal.

Kode lengkap tersedia pada berkas [client.py](client.py).

#### 2. Menjalankan Server dan Client

![Hasil client](19-hasil-client.png)

Server harus berjalan terlebih dahulu pada satu terminal, kemudian client dijalankan pada terminal lain (dengan environment yang sama aktif):

```
strawberry dev schema
```

```
python client.py
```

**Analisis Output:**

Client menampilkan daftar buku:

```
Daftar buku dari GraphQL server:
1. The Great Gatsby - F. Scott Fitzgerald
```

- Data yang ditampilkan sama dengan hasil pada GraphiQL.
- Terdapat dua proses berbeda, yaitu proses server (`strawberry dev`) dan proses client (`python client.py`), yang berkomunikasi melalui jaringan menggunakan format pesan yang telah disepakati (GraphQL di atas HTTP), bukan melalui *shared memory*.
- Karena yang disepakati adalah spesifikasinya, client dapat dibuat dengan bahasa lain tanpa mengubah server.

#### 3. Menghentikan Server

![Server dimatikan](20-server-mati.png)

Setelah seluruh pengujian selesai, server dihentikan dengan menekan `Ctrl + C` pada terminal tempat `strawberry dev schema` dijalankan. Proses server berhenti dan alamat `localhost:8000` tidak lagi dapat diakses.

## D. KESIMPULAN

1. Proses adalah hasil eksekusi program yang dikelola oleh sistem operasi dan memiliki identitas unik berupa PID. Setiap aplikasi yang dijalankan menjadi satu atau lebih proses, yang dapat diamati melalui Task Manager pada Windows maupun `htop` pada Linux.
2. Proses dapat dikendalikan dari luar aplikasi. Pada Linux, proses nano berhasil dimatikan melalui `htop` (F9, sinyal SIGTERM) dan perintah `pkill` tanpa keluar dari aplikasi, yang ditandai dengan munculnya pesan `Received SIGHUP or SIGTERM`.
3. *Restart* proses dilakukan dengan mematikan proses lalu menjalankannya kembali sehingga terbentuk proses baru dengan PID yang berbeda. Untuk kebutuhan tertentu, tersedia sinyal SIGHUP untuk membaca ulang konfigurasi dan `systemctl restart` untuk *service*.
4. Pada satu node, komunikasi antar proses relatif sederhana karena seluruhnya dikelola oleh sistem operasi, memakai *clock* yang sama, dan dapat memanfaatkan *shared memory*. Pada sistem terdistribusi, antar node tidak berbagi *clock* maupun memori sehingga komunikasi harus dilakukan melalui pertukaran pesan lewat jaringan, salah satunya dengan GraphQL.
5. `uv` memudahkan pembuatan workspace, penentuan Python 3.14, pembuatan environment, dan instalasi paket secara cepat. Strawberry memungkinkan pembuatan GraphQL server dengan beberapa baris kode Python serta menyediakan GraphiQL untuk pengujian.
6. GraphQL memungkinkan client meminta tepat field data yang dibutuhkan melalui satu endpoint, sedangkan server memenuhinya melalui schema dan resolver, sehingga client dan server dapat dibangun dengan peranti pengembangan yang berbeda.
7. Client berbasis Python berhasil mengakses GraphQL server melalui HTTP POST ke `/graphql` dan menampilkan data yang sama dengan hasil pada GraphiQL. Hal ini membuktikan bahwa komunikasi antar proses antara client dan server berjalan dengan baik.
