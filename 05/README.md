
# LAPORAN PRAKTIKUM SISTEM TERDISTRIBUSI DAN TERDESENTRALISASI

## Modul 5 — Fault Tolerance: Load Balancing Aplikasi dan Failure Detection

| | |
|---|---|
| Nama | Laesah Afenlia |
| NIM | 255410019 |
| Kelas | IF 1 |
| Dosen Pengampu | Dr. Bambang Purnomosidi D. P. |
| Program Studi | Informatika, Universitas Teknologi Digital Indonesia |

---

## 1. Tujuan Praktikum

1. Memahami konsep *fault tolerance* dan *high availability* pada sistem terdistribusi.
2. Membuat aplikasi web berbasis ASGI Python menggunakan framework BlackSheep.
3. Melakukan *scaling* aplikasi menjadi lebih dari satu *instance* (2 instance) menggunakan Docker Compose.
4. Mengkonfigurasi nginx sebagai *reverse proxy* sekaligus *load balancer*.
5. Memahami dan mempraktikkan mekanisme *failure detection* menggunakan protokol *heartbeat*.
6. Memahami penerapan pola *retry* (library `tenacity`) dan pola *circuit breaker* untuk menangani kegagalan koneksi ke komponen eksternal.

---

## 2. Dasar Teori

### 2.1 Fault Tolerance
*Fault tolerance* (toleransi kesalahan) adalah kemampuan sistem untuk tetap berfungsi dengan benar meskipun sebagian komponennya mengalami kegagalan. Pada sistem terdistribusi, kegagalan adalah hal yang wajar (node mati, jaringan putus, proses *crash*), sehingga sistem harus dirancang agar kegagalan satu komponen tidak membuat seluruh layanan berhenti. Teknik yang umum digunakan antara lain replikasi (redundansi), *load balancing*, *failure detection*, *retry*, dan *circuit breaker*.

### 2.2 High Availability dan Scaling
*High availability* (HA) adalah kondisi ketika layanan tetap tersedia dalam waktu yang sangat tinggi. Salah satu cara mencapainya adalah *horizontal scaling*, yaitu menjalankan aplikasi lebih dari satu *instance*. Jika satu instance gagal, instance lain masih dapat melayani permintaan.

### 2.3 Load Balancing
*Load balancing* adalah teknik mendistribusikan permintaan (*request*) dari klien ke beberapa server/instance agar beban terbagi merata, waktu respons lebih baik, dan tidak ada *single point of failure* pada sisi aplikasi. Algoritma yang umum: *round robin*, *least connections*, dan *IP hash*. Pada nginx, algoritma bawaan untuk blok `upstream` adalah *round robin*.

### 2.4 nginx sebagai Reverse Proxy
nginx adalah web server yang juga banyak dipakai sebagai *reverse proxy* dan *load balancer*. Direktif `upstream` mendefinisikan kelompok server *backend*, sedangkan `proxy_pass` meneruskan permintaan klien ke kelompok tersebut. Klien hanya berkomunikasi dengan nginx (port 80), tanpa mengetahui ada berapa instance di belakangnya.

### 2.5 Docker dan Docker Compose
*Docker* membungkus aplikasi beserta dependensinya ke dalam *container* sehingga lingkungan eksekusi konsisten. *Docker Compose* memungkinkan pengelolaan beberapa container sekaligus melalui satu berkas `docker-compose.yml`. Opsi `--scale <service>=N` digunakan untuk menjalankan N instance dari suatu *service*. Di dalam jaringan Compose, container dapat saling menjangkau menggunakan nama *service* (misalnya `bs_app`) sebagai nama host.

### 2.6 BlackSheep
BlackSheep adalah *web framework* asinkron untuk Python yang berjalan di atas standar ASGI (dijalankan dengan server seperti Uvicorn). Pada praktikum ini BlackSheep hanya digunakan sebagai aplikasi contoh (template `mvc`) untuk keperluan pengujian *load balancing*.

### 2.7 Failure Detection
*Failure detection* adalah proses untuk menentukan apakah suatu komponen telah gagal. Deteksi yang akurat penting agar sistem dapat segera melakukan pemulihan, misalnya mengalihkan lalu lintas ke komponen lain.

### 2.8 Heartbeat
Protokol *heartbeat* memantau aktivitas suatu komponen. Jika komponen pemantau tidak menerima *heartbeat* dari komponen lain dalam jangka waktu tertentu, komponen tersebut diasumsikan telah gagal dan tidak responsif. Pada praktikum ini, pemeriksaan dilakukan dengan script `check-server.py` yang mencoba terhubung ke host dan port tertentu.

### 2.9 Retry dengan Tenacity
Koneksi ke komponen di luar aplikasi selalu berpotensi gagal (kegagalan sementara/*transient*). Pola *retry* mengulangi percobaan beberapa kali sebelum menyerah. Library `tenacity` menyediakan dekorator untuk mengatur jumlah percobaan, jeda antar percobaan (*wait*), dan kondisi berhenti (*stop*).

### 2.10 Circuit Breaker
*Circuit breaker* adalah pola desain pada *microservices* untuk mencegah kegagalan berantai (*cascading failure*) dengan memblokir sementara panggilan ke layanan yang gagal. Pola ini bekerja sebagai *state machine* dengan tiga status:

| Status | Perilaku |
|---|---|
| **Closed** | Permintaan diizinkan dan dipantau kegagalannya. Jika ambang batas kegagalan terpenuhi, beralih ke *open*. |
| **Open** | Permintaan langsung ditolak agar layanan punya waktu untuk pulih. Setelah batas waktu, beralih ke *half-open*. |
| **Half-open** | Sejumlah permintaan terbatas diizinkan sebagai uji. Jika berhasil kembali ke *closed*, jika gagal kembali ke *open*. |

---

## 3. Langkah Kerja

### 3.1 Persiapan Environment

```bash
mkdir workspace-05
cd workspace-05
uv venv
source .venv/bin/activate
uv pip install blacksheep blacksheep-cli
```
**uv pip install blacksheep**



**uv pip install blacksheep-cli**

**Pembahasan: 
1. **`mkdir workspace-05 && cd workspace-05`**: Membuat direktori kerja baru bernama `workspace-05` dan langsung berpindah ke dalam direktori tersebut.
2. **`uv venv`**: Membuat lingkungan virtual (*virtual environment*) Python terisolasi di dalam direktori `.venv` menggunakan tool `uv`.
3. **`source .venv/bin/activate`**: Mengaktifkan virtual environment yang telah dibuat, ditandai dengan munculnya nama environment `(workspace-05)` pada prompt terminal.
4. **`uv pip install blacksheep blacksheep-cli`**: Mengunduh dan menginstal framework web berbasis ASGI `blacksheep` beserta perkakas command line `blacksheep-cli` ke dalam virtual environment.

---

### 3.2 Membuat Aplikasi BlackSheep
2. Pembuatan Aplikasi Web
Setelah itu, buat aplikasi menggunakan cli:


```bash
$ blacksheep create
# Project name: blacksheep_lb | Template: mvc | OpenAPI: Yes
# Settings library: essentials-configuration | Format: TOML
$ cd blacksheep_lb
$ uv pip install -r requirements.txt
$ python dev.py
```
Aplikasi berjalan di `http://localhost:44777`. Dockerfile bawaan BlackSheep membuka **port 80**, bukan 44777, dan hanya diubah pada versi Python (`3.14.4-slim`).

### 3.3 Struktur Direktori
```
.
├── blacksheep_lb/        # aplikasi (punya Dockerfile sendiri)
├── nginx/
│   ├── Dockerfile
│   └── nginx.conf
├── docker-compose.yml
└── env.sh                # alias (opsional, Linux Bash)
```

### 3.4 Konfigurasi nginx

`nginx/Dockerfile`
```dockerfile
FROM nginx

# Override the default nginx configuration file
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/nginx.conf
```

`nginx/nginx.conf`
```nginx
events {
  worker_connections 4096;
}

http {
  # Definisi grup, lihat di docker-compose.yml
  upstream backend {
    server bs_app:80;
  }
  server {
    listen 80;
    location / {
      # sesuaikan dengan nama upstream di atas
      proxy_pass http://backend;
    }
  }
}
```

### 3.5 docker-compose.yml
```yaml
services:
  # aplikasi yang akan di-scale, ada pada direktori blacksheep_lb
  bs_app:
    build: ./blacksheep_lb
  # nginx dibangun dari nginx/Dockerfile
  nginx:
    container_name: nginx
    build: ./nginx
    ports:
      - 80:80
    depends_on:
      - bs_app
```

### 3.6 Alias (env.sh)
```bash
alias dcu="sudo docker-compose up --build -d --scale bs_app=2"
alias dcd="sudo docker compose down"
alias dps="docker ps --format '{{.Image}} {{.Names}}'"
alias dcln="docker logs nginx"
```

### 3.7 Menjalankan Sistem
```bash
$ sudo dinitctl start dockerd     # Systemd: sudo systemctl start docker
$ source env.sh
$ dcu                             # build + jalankan, bs_app di-scale jadi 2
$ dps                             # cek container
$ sudo docker logs load-balancing-bs_app-1
$ sudo docker logs load-balancing-bs_app-2
$ dcd                             # matikan semua container
```

### 3.8 Pengujian Failure Detection
```bash
# Heartbeat
$ python check-server.py localhost 44777
$ python check-server.py localhost 80

# Retry (tenacity)
$ python check-retry.py

# Circuit breaker
$ python check-circuit-breaker.py
```
Masing-masing script dijalankan pada dua kondisi: aplikasi BlackSheep **aktif** dan **non-aktif** di port 44777.

---

## 4. Hasil dan Pembahasan

### 4.1 Load Balancing
Setelah `dcu`, Docker membangun image `bs_app` dan `nginx`, lalu menjalankan 3 container. Perintah `dps` menampilkan:

```
load-balancing-nginx nginx
load-balancing-bs_app load-balancing-bs_app-1
load-balancing-bs_app load-balancing-bs_app-2
```

Awalan `load-balancing` berasal dari nama direktori proyek. Aplikasi dapat diakses melalui `http://localhost` (port 80), bukan lagi port 44777, karena klien sekarang berkomunikasi dengan nginx.

Pada log kedua instance terlihat permintaan `GET /` beserta aset statis (CSS, JS, gambar) tercatat di instance yang berbeda. Instance `bs_app-1` menerima 2 akses `GET /` terakhir dan `bs_app-2` menerima 1 akses. Hal ini membuktikan permintaan dari klien diteruskan nginx ke instance yang tersedia, sehingga beban terdistribusi. Jika salah satu instance mati, instance lainnya masih dapat melayani permintaan sehingga *availability* aplikasi meningkat.

### 4.2 Heartbeat (`check-server.py`)

| Kondisi | Perintah | Hasil |
|---|---|---|
| BlackSheep aktif di 44777 | `python check-server.py localhost 44777` | `Server up` |
| Tidak ada server di port 80 | `python check-server.py localhost 80` | `Server down` |

Script mencoba terhubung ke host dan port yang diberikan. Jika koneksi berhasil, komponen dianggap hidup (*up*); jika koneksi gagal atau ditolak, komponen dianggap gagal (*down*). Inilah prinsip dasar *failure detection*: respons dari komponen menjadi tanda ia masih aktif, sedangkan ketiadaan respons dianggap sebagai kegagalan.

### 4.3 Retry dengan Tenacity (`check-retry.py`)

| Kondisi | Hasil |
|---|---|
| BlackSheep aktif | `Attempting to access http://localhost:44777...` lalu `Successfully accessed http://localhost:44777` (berhasil pada percobaan pertama) |
| BlackSheep non-aktif | Pesan `Attempting to access ...` muncul **5 kali**, lalu `Failed to access URL after multiple retries: RetryError[... raised ConnectionError]` |

Saat server aktif, tidak diperlukan pengulangan. Saat server mati, `tenacity` mengulang percobaan beberapa kali. Setelah seluruh percobaan habis, *exception* `RetryError` dilempar dengan penyebab `ConnectionError`. Pola ini berguna untuk kegagalan sementara (misalnya server baru restart), tetapi untuk layanan yang benar-benar mati, pengulangan terus-menerus membuang waktu dan sumber daya.

### 4.4 Circuit Breaker (`check-circuit-breaker.py`)

| Kondisi | Hasil |
|---|---|
| BlackSheep aktif | `Attempting to access http://localhost:44777...` lalu `Successfully accessed http://localhost:44777` |
| BlackSheep non-aktif | `HTTPConnectionPool(host='localhost', port=44777): Max retries exceeded ... [Errno 111] Connection refused` |

Cara kerja secara konsep: selama layanan sehat, circuit breaker berada pada status *closed* dan permintaan diteruskan. Kegagalan dihitung, dan ketika melewati ambang batas, sirkuit berpindah ke status *open* sehingga permintaan berikutnya langsung ditolak tanpa mencoba menghubungi layanan yang gagal. Setelah jeda waktu, sirkuit masuk status *half-open* untuk menguji pemulihan. Jika uji berhasil maka kembali *closed*, jika gagal kembali *open*.

> Catatan: detail implementasi (library yang dipakai, nilai ambang kegagalan, dan lama *recovery timeout*) mengikuti isi file `check-circuit-breaker.py` pada modul praktikum. Sesuaikan bagian ini dengan kode yang kamu jalankan.

### 4.5 Perbandingan Retry vs Circuit Breaker

| Aspek | Retry | Circuit Breaker |
|---|---|---|
| Tujuan | Mengatasi kegagalan sementara | Mencegah kegagalan berantai dan melindungi layanan |
| Perilaku saat gagal | Mengulang percobaan beberapa kali | Memblokir panggilan sementara |
| Risiko | Membebani layanan yang sedang bermasalah | Permintaan ditolak walau layanan mungkin sudah pulih (sebelum half-open) |
| Penggunaan ideal | Gangguan singkat | Layanan gagal berkepanjangan |

Keduanya saling melengkapi dan sering dipakai bersamaan.

---

## 5. Kesimpulan

1. *Load balancing* dengan nginx dan Docker Compose berhasil mendistribusikan permintaan ke 2 instance aplikasi BlackSheep sehingga meningkatkan *high availability*.
2. Protokol *heartbeat* sederhana mampu membedakan kondisi server aktif (`Server up`) dan tidak aktif (`Server down`).
3. Pola *retry* (`tenacity`) menangani kegagalan sementara dengan mengulang percobaan, dan melaporkan `RetryError` bila semua percobaan gagal.
4. Pola *circuit breaker* mencegah kegagalan berantai dengan membatasi panggilan ke layanan yang gagal, melalui tiga status *closed*, *open*, dan *half-open*.
5. Kombinasi *load balancing*, *failure detection*, *retry*, dan *circuit breaker* menjadi fondasi penting untuk membangun sistem terdistribusi yang toleran terhadap kegagalan.

---