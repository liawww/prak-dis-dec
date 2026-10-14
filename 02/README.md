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
<img width="1362" height="727" alt="01-task-manager" src="https://github.com/user-attachments/assets/531799c1-0462-4cee-b663-8427e1f3cc21" />


Task Manager dibuka dengan kombinasi tombol `Ctrl + Shift + Esc`, kemudian dipilih tab **Processes**. Pada tab ini ditampilkan berbagai proses yang sedang berjalan, lengkap dengan penggunaan CPU, memori, disk, jaringan, dan konsumsi daya (*power usage*). Daftar diurutkan berdasarkan penggunaan CPU tertinggi.

**Analisis Output:**

- Saat pengamatan, penggunaan CPU tercatat 74%, memori 91%, disk 6%, dan jaringan 0%.
- Proses dengan penggunaan CPU tertinggi adalah rsEngineSvc (29,1%) dengan power usage Very high, disusul Search (15,1%, power usage High) dan Desktop Window Manager (6,1%) yang mengelola tampilan jendela.
- Proses Google Chrome (19) menggunakan memori terbesar, yaitu 1.024,5 MB. Angka (19) menunjukkan bahwa satu aplikasi Chrome memunculkan beberapa proses sekaligus.
- Penggunaan memori sistem yang mencapai 91% menunjukkan bahwa memori komputer hampir terpakai seluruhnya oleh proses-proses yang berjalan.
- Tampak pula berbagai proses lain seperti Rainmeter, Windows Explorer, System, NVIDIA Container, Lively Wallpaper, Runtime Broker, CTF Loader, dan beberapa Service Host, yang berjalan di latar belakang tanpa disadari pengguna.
- Banyaknya proses yang tampil menunjukkan bahwa sistem operasi mengelola banyak proses secara bersamaan, sesuai dengan sifat transparan pengelolaan proses pada satu node.

#### 2. Menyiapkan Linux dengan WSL dan Ubuntu

<img width="648" height="119" alt="02-wsl-welcome" src="https://github.com/user-attachments/assets/f48ac690-7a2f-475c-932f-2dc40704b734" />


<img width="664" height="462" alt="03-ubuntu-akun" src="https://github.com/user-attachments/assets/03acce07-dc7e-40a8-8299-63e4b1f512e8" />


Agar proses pada Linux dapat diamati, digunakan WSL. Dari Command Prompt Windows dijalankan perintah wsl, sehingga prompt berubah menjadi afenlia@DESKTOP-UUH2NR5:/mnt/c/Users/ASUS$. Prompt ini menandakan bahwa shell Linux (Ubuntu) sudah aktif, sedangkan /mnt/c adalah drive C: milik Windows yang dipasang (mount) di dalam WSL.

Pada pembukaan pertama, Ubuntu melakukan penyiapan instance baru (Provisioning the new WSL instance Ubuntu) dan meminta pembuatan akun pengguna. Diisi nama pengguna afenlia beserta password, kemudian dipilih y pada pertanyaan pengumpulan data metrik platform. Setelah tahap ini selesai, terminal Ubuntu siap digunakan.

#### 3. Memasang `htop` dan Menampilkan Proses pada Linux

<img width="1366" height="724" alt="04-htop-install" src="https://github.com/user-attachments/assets/7d7cb9c6-ada8-43e8-b76b-76c18dc53c0f" />


<img width="1366" height="729" alt="05-htop-proses" src="https://github.com/user-attachments/assets/1d7d305b-9c52-48a3-954b-5521041a98ce" />


Agar proses dapat diamati dengan htop, aplikasi tersebut dipasang terlebih dahulu. Daftar paket diperbarui dan htop dipasang dalam satu perintah, kemudian htop dijalankan:

```
sudo apt update && sudo apt install htop -y
htop
```

Perintah sudo meminta password pengguna ([sudo: authenticate] Password:). Operator && membuat perintah kedua hanya dijalankan jika apt update berhasil, sedangkan opsi -y menjawab "ya" secara otomatis pada konfirmasi pemasangan. Pada tangkapan layar terlihat apt mengambil (Get) daftar paket dari repositori Ubuntu (resolute, resolute-updates, resolute-security, dan resolute-backports).

**Analisis Output:**

- `htop` menampilkan daftar proses lengkap dengan PID, pengguna (*USER*), penggunaan CPU dan memori, serta perintah yang dijalankan.
- Pada bagian atas terlihat keempat core CPU (0 sampai 3) yang hampir tidak terpakai (0,0%), penggunaan memori 344 MB dari 1,81 GB, swap 0K dari 1,00 GB, serta jumlah proses (Tasks) sebanyak 27 dengan 12 thread. Load average tercatat 0,05 0,10 0,09 dan uptime 00:13:50.
- Proses pertama adalah `/sbin/init` dengan PID 1, yaitu proses induk yang menjalankan proses-proses lainnya pada sistem Linux.

#### 4. Menjalankan Aplikasi dan Melihat Proses yang Dimunculkan

<img width="1364" height="728" alt="06-nano-jalan" src="https://github.com/user-attachments/assets/2b26f018-a89c-4771-b698-5aca0c7fe8ea" />


<img width="1363" height="723" alt="07-nano-di-htop" src="https://github.com/user-attachments/assets/d49919b8-ea03-420c-a526-5106386ef46f" />


Aplikasi yang dijalankan adalah editor teks nano (versi 8.7.1) dengan membuka berkas bernama tugas:

```
nano tugas
```

Setelah nano terbuka pada satu jendela terminal, dibuka jendela terminal Ubuntu kedua, lalu dijalankan htop untuk memeriksa daftar proses. Kedua jendela ditampilkan berdampingan, htop di sebelah kiri dan nano di sebelah kanan.

**Analisis Output:**

- Pada daftar proses htop muncul baris baru dengan perintah nano tugas, pengguna afenlia, dan PID 961 (penggunaan memori 0,2%, status S atau sleeping karena nano sedang menunggu masukan dari pengguna).
- Jumlah proses (Tasks) naik menjadi 31 dibandingkan 27 sebelumnya, sebagian karena terminal kedua dan proses htop itu sendiri ikut berjalan.
- Hal ini membuktikan bahwa setiap aplikasi yang dijalankan akan menjadi proses yang tercatat dan dikelola oleh sistem operasi dengan PID tertentu.

#### 5. Mematikan Proses Melalui Perintah (tanpa Keluar dari Aplikasi)

<img width="672" height="711" alt="08-kill-htop-f9" src="https://github.com/user-attachments/assets/1ba01101-1620-49cb-abf6-fda1baaca7e3" />


<img width="1365" height="727" alt="09-pkill-nano" src="https://github.com/user-attachments/assets/376f8d83-9763-403d-8d2c-c5a97a2b2308" />


Proses nano dimatikan dari luar aplikasi, bukan dengan perintah keluar bawaan nano (`Ctrl + X`). Terdapat dua cara yang dicoba:

**a. Melalui `htop`.** Proses nano dipilih pada daftar, kemudian ditekan tombol **F9 (Kill)**, Muncul daftar sinyal pada panel Send signal, dipilih sinyal **SIGTERM** (sinyal 15), dan ditekan Enter untuk mengirim.
**b. Melalui perintah `pkill`.** Proses dimatikan berdasarkan namanya:

```
pkill nano
```

Padanan lain yang dapat digunakan adalah mematikan berdasarkan PID, yaitu `kill 961`.

**Analisis Output:**

- Pada terminal tempat nano dijalankan muncul pesan `Received SIGHUP or SIGTERM`, kemudian aplikasi langsung tertutup dan prompt shell kembali tampil.
- Hal ini menandakan bahwa proses dihentikan oleh sistem operasi melalui sinyal, bukan karena pengguna keluar dari aplikasi.
- Proses nano tidak lagi muncul pada daftar `htop` setelah perintah dijalankan.

#### 6. Me-*restart* Proses

<img width="875" height="74" alt="10-restart-nano" src="https://github.com/user-attachments/assets/4818f796-d642-4509-86e8-d75cc3f5318d" />


Pada Linux tidak terdapat satu perintah khusus untuk me-*restart* proses aplikasi biasa. Restart dilakukan dengan mematikan proses terlebih dahulu, kemudian menjalankannya kembali:

```
pkill nano
nano tugas
```

Untuk jenis proses lain, tersedia cara yang menyesuaikan sifat proses tersebut:

- Program yang mendukung *reload* konfigurasi dapat diminta membaca ulang konfigurasinya tanpa dimatikan dengan mengirim sinyal SIGHUP: `kill -HUP <PID>`.
- Proses yang berjalan sebagai *service* dapat di-*restart* dengan `sudo systemctl restart <nama_service>`.

**Analisis Output:**

- Setelah `nano tugas` dijalankan kembali, `htop` menampilkan baris `nano tugas` dengan PID baru, yaitu *757* (pengguna `afenlia`, status `S`).
- Hal ini menunjukkan bahwa *restart* sebenarnya membuat proses baru dengan PID baru, bukan menghidupkan kembali proses yang sama.

#### 7. Ringkasan Pekerjaan Praktik 1

Secara berurutan, pekerjaan pada Praktik 1 adalah menampilkan proses pada Windows melalui Task Manager, menyiapkan Ubuntu pada WSL, memasang dan menjalankan `htop`, menjalankan aplikasi nano dan menemukan prosesnya (PID 961), mematikan proses tersebut melalui `htop` dan `pkill` tanpa keluar dari aplikasi, serta me-*restart* proses dengan menjalankan kembali aplikasinya (PID baru 757). Praktik ini menunjukkan bahwa pada satu node seluruh siklus hidup proses ditangani oleh sistem operasi.

---

### PRAKTIK 2 – GRAPHQL SERVER MENGGUNAKAN STRAWBERRY

#### 1. Menyiapkan Lingkungan dengan uv (Memasang uv, Membuat Workspace, Environment, dan Instalasi Paket)

<img width="1360" height="728" alt="11-install-strawberry" src="https://github.com/user-attachments/assets/3c08da0c-c77b-443e-9d81-5bbbb5a7a70e" />


Langkah pertama adalah mempelajari penggunaan `uv` berdasarkan catatan pada repositori NEO-X-School yang disebutkan di modul. Seluruh langkah penyiapan pada praktik ini dilakukan di terminal Ubuntu (WSL), sehingga perintah yang dipakai adalah perintah Linux.

#### a. Memasang uv. 
uv.dipasang menggunakan skrip instalasi resmi:

``
curl -LsSf https://astral.sh/uv/install.sh | sh
``
Setelah selesai, installer menampilkan pesan `everything's installed!` beserta daftar program yang terpasang (`uv` dan `uvx`). Installer juga memberi tahu bahwa direktori `$HOME/.local/bin` perlu ditambahkan ke `PATH`. Agar `uv` langsung dikenali pada terminal yang sedang berjalan tanpa membuka terminal baru, dijalankan:

``
source $HOME/.local/bin/env
``

#### b. Membuat workspace.
Workspace dengan nama `workspace-01` dibuat sebagai folder kerja agar pekerjaan praktikum tertata dan terpisah dari proyek lain, kemudian terminal berpindah ke folder tersebut:

```
uv init --python 3.14 workspace-01
cd workspace-01
uv python install 3.14
```

#### c. Menentukan versi Python, membuat, dan mengaktifkan environment.
Pada workspace tersebut digunakan Python versi 3.14 sesuai petunjuk modul. *Virtual environment* dibuat agar paket yang dipasang untuk praktikum ini tidak bercampur dengan paket lain di komputer, lalu diaktifkan:

```
uv venv --python 3.14
.venv\Scripts\activate
```

Opsi --python 3.14 menetapkan versi Python environment sehingga tidak bergantung pada versi Python bawaan lain di sistem.

**Analisis Output:**

- uv menampilkan Using CPython 3.14.4 interpreter at: /usr/bin/python3.14, artinya Python 3.14.4 sudah tersedia di sistem sehingga langsung dipakai tanpa perlu diunduh.
- Muncul pula Creating virtual environment at: .venv dan petunjuk aktivasi Activate with: source .venv/bin/activate.
- Setelah environment aktif, awal baris terminal diberi tambahan nama environment, yaitu (workspace-01). Hal ini menandakan paket yang dipasang berikutnya hanya berlaku di dalam environment ini.

#### d. Instalasi Paket yang Diperlukan
Paket Strawberry GraphQL beserta fitur CLI-nya dipasang ke dalam environment dengan perintah berikut. Tanda petik tunggal dipakai agar tanda kurung siku [cli] tidak ditafsirkan oleh shell:

```
uv pip install "strawberry-graphql[cli]"
```

**Analisis Output:**

- `uv` me-*resolve* dan memasang 24 paket sekaligus, di antaranya `strawberry-graphql`, `graphql-core`, `starlette`, `uvicorn`, `websockets`, `click`, `typer`, dan `rich`.
- Paket `graphql-core` menyediakan implementasi inti spesifikasi GraphQL, `starlette` dan `uvicorn` menyediakan lapisan server web, sedangkan `typer`, `click`, dan `rich` mendukung perintah `strawberry` pada terminal.
- Proses instalasi berlangsung sangat cepat karena `uv` memanfaatkan *cache*.

#### 2. Menyiapkan `schema.py`

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

#### 3. Menjalankan Server
<img width="1366" height="717" alt="12-server-strawberry" src="https://github.com/user-attachments/assets/64e8a96b-446a-4380-97a7-f8f423a6ced9" />

Dari folder `workspace-01`, environment diaktifkan kembali kemudian Server dijalankan dengan perintah:

```
source .venv/bin/activate
strawberry dev schema
```
Catatan: perintah `wsl` pada awal tangkapan layar menghasilkan pesan `Command'wsl' not found` karena terminal sudah berada di dalam Ubuntu, sehingga perintah tersebut tidak diperlukan.

**Analisis Output:**

Terminal menampilkan Running strawberry on http://0.0.0.0:8000/graphql. Artinya server GraphQL mendengarkan permintaan pada port 8000 dengan endpoint /graphql. Selama server berjalan, terminal ini tidak boleh ditutup karena server adalah proses yang terus berjalan (foreground).

#### 4. Mengakses GraphiQL melalui Browser

<img width="943" height="391" alt="15-graphql-server" src="https://github.com/user-attachments/assets/aec6c3d2-42bb-4973-8186-a431125cba64" />


Browser dibuka, kemudian diakses alamat `http://localhost:8000/graphql.` Sesuai modul, alamat `0.0.0.0` dapat diganti dengan `localhost` atau `127.0.0.1` karena ketiganya mengarah ke server yang sama. Muncul antarmuka *GraphiQL* yang terbagi menjadi dua bagian, yaitu sisi kiri untuk menuliskan query dan sisi kanan untuk menampilkan hasil query. Tombol run (segitiga merah muda) digunakan untuk menjalankan query.

Pada bilah alamat terlihat parameter `?q=...` yang berisi query dalam bentuk terkodekan (misalnya `%7B` untuk `{` dan `%0A` untuk baris baru). GraphiQL menyimpan isi query pada URL sehingga query yang sama dapat dibuka kembali.

#### 5. Menuliskan dan Menjalankan Query

<img width="562" height="285" alt="17-graphql-query" src="https://github.com/user-attachments/assets/2c2e601e-7f59-4b81-90e9-548739ccb4e5" />


Pada sisi kiri GraphiQL dituliskan query berikut. Query ini meminta server mengirim daftar `books`, dan dari setiap buku hanya diminta field `title` dan `author`:

```graphql
{
  books {
    title
    author
  }
}
```
Saat query diketik, GraphiQL menampilkan saran otomatis (autocomplete) seperti `query`, `mutation`, `subscription`, dan `fragment` yang diambil dari schema server.

<img width="710" height="243" alt="18-graphql-response" src="https://github.com/user-attachments/assets/b55ac19a-0146-4c32-adee-74d239c93147" />


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
Query yang sama juga diuji melalui alamat `http://127.0.0.1:8000/graphql` dengan penulisan lengkap menggunakan kata kunci `query { ... }`. Hasilnya identik, karena penulisan `{ ... }` hanyalah bentuk singkat dari `query { ... }`.

<img width="1366" height="722" alt="13-graphql-query" src="https://github.com/user-attachments/assets/0702ee61-4dec-4e7e-9e05-192125a8069c" />

**Analisis Output:**

- Struktur hasil JSON sama dengan query yang ditulis: hanya field `title` dan `author` yang dikembalikan, berada di dalam objek `data` > `books`.
- Hal ini membuktikan bahwa client menentukan sendiri bentuk data yang diminta, sedangkan server memenuhinya melalui resolver.
- Browser bertindak sebagai client yang mengirim permintaan, sementara server Strawberry yang berjalan sebagai proses terpisah menjawab dengan data yang sesuai. Inilah bentuk komunikasi antar proses melalui jaringan.

---

### PRAKTIK 3 – MEMBUAT CLIENT UNTUK GRAPHQL SERVER

#### 1. Membuat Program Client
<img width="1195" height="693" alt="14-graphql-client" src="https://github.com/user-attachments/assets/63c27203-f7b9-4e5d-8dfc-202f81c2a168" />

Tugas terakhir adalah membuat client menggunakan bahasa pemrograman bebas yang mengakses GraphQL server di atas. Bahasa yang dipilih adalah **Python**, dengan modul bawaan `urllib.request` dan `json` sehingga tidak diperlukan instalasi paket tambahan. Client dibuat pada berkas `client.py`:

```python
import urllib.request
import json

# URL GraphQL Server
URL = "http://localhost:8000/graphql"

QUERY = """
{
  books {
    title
    author
  }
}
"""

# Membentuk request HTTP POST berisi query dalam format JSON
data = json.dumps({"query": query}).encode("utf-8")
req = urllib.request.Request(
    url,
    data=data,
    headers={"Content-Type": "application/json"},
)

# Mengirim request dan membaca respons dari server
with urllib.request.urlopen(req) as response:
    result = json.loads(response.read().decode("utf-8"))

print("=== HASIL QUERY DARI GRAPHQL CLIENT ===")
print(json.dumps(result, indent=2))
```

**Penjelasan Kode:**

- Query GraphQL dikirim sebagai JSON berbentuk `{"query": "..."}` dengan metode **HTTP POST** ke endpoint `/graphql`.
- Respons server dibaca dan di-*parse* dari JSON menjadi *dictionary* Python.
- Hasil ditampilkan ke terminal dalam format JSON yang rapi menggunakan `json.dumps(..., indent=2)`.

Kode lengkap tersedia pada berkas [client.py](client.py).

#### 2. Menjalankan Server dan Client

<img width="926" height="440" alt="19-hasil-client" src="https://github.com/user-attachments/assets/5ceb5bc7-3f2a-416a-88bf-a5d4b1461c4e" />


Server harus berjalan terlebih dahulu pada satu terminal (`strawberry dev schema`), kemudian client dijalankan pada terminal lain, yaitu terminal PowerShell di Visual Studio Code, dengan environment yang aktif:
```
Set-ExecutionPolicy Unrestricted -Scope Process
.\Scripts\Activate.ps1
python client.py
```
Perintah `Set-ExecutionPolicy Unrestricted -Scope Process` diperlukan karena PowerShell secara bawaan memblokir eksekusi skrip, sehingga skrip aktivasi environment tidak dapat dijalankan. Opsi `-Scope Process` membuat pelonggaran ini hanya berlaku pada sesi terminal yang sedang dibuka. Setelah itu `Activate.ps1` mengaktifkan environment (prompt menjadi `(workspace-01) PS ...`), lalu `python client.py` menjalankan client.

**Analisis Output:**
Client menampilkan judul === HASIL QUERY DARI GRAPHQL CLIENT === diikuti hasil JSON dari server:


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

- Data yang ditampilkan sama dengan hasil pada GraphiQL.
- Terdapat dua proses berbeda, yaitu proses server (`strawberry dev`) dan proses client (`python client.py`), yang berkomunikasi melalui jaringan menggunakan format pesan yang telah disepakati (GraphQL di atas HTTP), bukan melalui *shared memory*.
- Karena yang disepakati adalah spesifikasinya, client dapat dibuat dengan bahasa lain tanpa mengubah server.

#### 3. Menghentikan Server

<img width="737" height="270" alt="20-server-mati" src="https://github.com/user-attachments/assets/f372cf8c-d28d-42e1-a370-b85d5b4695cc" />


Setelah seluruh pengujian selesai, server dihentikan dengan menekan `Ctrl + C` pada terminal tempat `strawberry dev schema` dijalankan, sesuai petunjuk modul. Pada tangkapan layar tampak tanda `^C` pada terminal setelah tombol tersebut ditekan. Setelah server berhenti, alamat `localhost:8000` tidak lagi dapat diakses.

## D. KESIMPULAN

1. Proses adalah hasil eksekusi program yang dikelola oleh sistem operasi dan memiliki identitas unik berupa PID. Setiap aplikasi yang dijalankan menjadi satu atau lebih proses, yang dapat diamati melalui Task Manager pada Windows maupun `htop` pada Linux.
2. Proses dapat dikendalikan dari luar aplikasi. Pada Linux, proses nano berhasil dimatikan melalui `htop` (F9, sinyal SIGTERM) dan perintah `pkill` tanpa keluar dari aplikasi, yang ditandai dengan munculnya pesan `Received SIGHUP or SIGTERM`.
3. *Restart* proses dilakukan dengan mematikan proses lalu menjalankannya kembali sehingga terbentuk proses baru dengan PID yang berbeda. Untuk kebutuhan tertentu, tersedia sinyal SIGHUP untuk membaca ulang konfigurasi dan `systemctl restart` untuk *service*.
4. Pada satu node, komunikasi antar proses relatif sederhana karena seluruhnya dikelola oleh sistem operasi, memakai *clock* yang sama, dan dapat memanfaatkan *shared memory*. Pada sistem terdistribusi, antar node tidak berbagi *clock* maupun memori sehingga komunikasi harus dilakukan melalui pertukaran pesan lewat jaringan, salah satunya dengan GraphQL.
5. `uv` memudahkan pembuatan workspace, penentuan Python 3.14, pembuatan environment, dan instalasi paket secara cepat. Strawberry memungkinkan pembuatan GraphQL server dengan beberapa baris kode Python serta menyediakan GraphiQL untuk pengujian.
6. GraphQL memungkinkan client meminta tepat field data yang dibutuhkan melalui satu endpoint, sedangkan server memenuhinya melalui schema dan resolver, sehingga client dan server dapat dibangun dengan peranti pengembangan yang berbeda.
7. Client berbasis Python berhasil mengakses GraphQL server melalui HTTP POST ke `/graphql` dan menampilkan data yang sama dengan hasil pada GraphiQL. Hal ini membuktikan bahwa komunikasi antar proses antara client dan server berjalan dengan baik.
