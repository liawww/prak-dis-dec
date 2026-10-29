# Laporan Praktikum Modul 4 — Konsistensi dan Replikasi pada Sistem Terdistribusi

- **Nama**: Laesah Afenlia
- **NIM**: 255410019
- **Mata Kuliah**: Sistem Terdistribusi dan Terdesentralisasi
- **Prodi**: Informatika 1

Repo ini berisi hasil praktikum dua model replikasi:

1. **Streaming Replication PostgreSQL 18** (primary → replica, asinkron)
2. **Replikasi Master-Master dengan Apache Ignite 3.1.0** (cluster 3 node)

## Struktur Repo

```
.
├── README.md
├── postgresql/
│   └── src/
│       ├── 00_init.sql
│       ├── docker-compose.yaml
│       ├── env.sh
│       └── img/            <- screenshot bukti
└── ignite/
    └── src/
        ├── docker-compose.yaml
        ├── env.sh
        ├── sql/            <- schema.sql, current_catalog.sql, dst.
        └── img/            <- screenshot bukti
```

---

# Bagian 1 — Streaming Replication PostgreSQL 18

## 1.1 Konsep Singkat

*Streaming replication* mengirimkan **WAL (Write-Ahead Log)** dari server **primary** ke server **standby/replica** secara terus-menerus lewat koneksi TCP. Replica memutar ulang (*replay*) WAL tersebut sehingga datanya identik dengan primary.

- **Primary**: menerima operasi *read-write*.
- **Replica (hot standby)**: hanya *read-only*, selalu menyinkronkan diri dari primary.
- **Mode**: asinkron (`sync_state = async`). Primary tidak menunggu konfirmasi replica sebelum commit, sehingga cepat, tetapi ada kemungkinan kecil replica tertinggal (*replication lag*).
- **Replication slot**: menjamin primary tidak menghapus WAL yang belum diterima replica.

## 1.2 Penjelasan File

### `00_init.sql`
Dijalankan otomatis saat primary pertama kali diinisialisasi (karena di-mount ke `/docker-entrypoint-initdb.d/`).

```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');
```

- Baris 1: membuat user `replicator` yang punya hak `REPLICATION`, dipakai replica untuk menarik WAL.
- Baris 2: membuat *physical replication slot* bernama `replication_slot`.

### `docker-compose.yaml`

| Bagian | Fungsi |
|---|---|
| `x-postgres-common` | Template bersama (YAML anchor) untuk kedua service: image `postgres:18.3-alpine3.23`, user `postgres`, `restart: always`, dan *healthcheck* `pg_isready`. |
| `postgres_primary` | Port `5432`. User/DB/password: `zuser`/`zdb`/`zpass`. |
| `POSTGRES_HOST_AUTH_METHOD` | Mengatur `pg_hba.conf`: metode `scram-sha-256` untuk koneksi biasa, serta baris `host replication replicator 0.0.0.0/0 md5` agar user `replicator` boleh melakukan replikasi. |
| `command` (primary) | `wal_level=replica`, `hot_standby=on`, `max_wal_senders=10`, `max_replication_slots=10`, `hot_standby_feedback=on`. |
| `postgres_replica` | Port host `5433` → `5432` container. Memakai `PGUSER=replicator`. `PGDATA=/var/lib/postgresql/18/docker` (lokasi data PG 18 berbeda dari versi lama). |
| `command` (replica) | Menjalankan `pg_basebackup -R --slot=replication_slot -X stream` ke `postgres_primary` (diulang tiap 5 detik sampai berhasil), `chmod 0700` pada direktori data, lalu menjalankan `postgres`. Opsi `-R` otomatis membuat `standby.signal` dan `primary_conninfo`. |
| `depends_on` | Replica dijalankan setelah primary. |

### `env.sh`
Kumpulan alias agar perintah lebih singkat (opsional):

| Alias | Perintah |
|---|---|
| `dcu` | `sudo docker-compose up -d` |
| `dcd` | `sudo docker-compose down` |
| `dps` | `sudo docker ps` |
| `dep` / `der` | masuk bash ke container primary / replica |
| `dlp` / `dlr` | lihat log primary / replica |

> Nama container diawali nama direktori. Karena file disimpan di direktori `src`, namanya `src-postgres_primary-1` dan `src-postgres_replica-1`. **Jika direktori kamu bernama lain, sesuaikan `env.sh`.**

## 1.3 Langkah Menjalankan

```bash
cd postgresql/src
source env.sh
dcu          # docker-compose up -d
dps          # cek STATUS harus (healthy)
```

Jika STATUS belum `healthy`, cek log:

```bash
dlp   # log primary
dlr   # log replica
```

Log replica yang normal memuat pesan `Backup done, starting replica...` dan `started streaming WAL from primary`. Pesan `pg_basebackup failed. Retrying...` di awal itu wajar (primary belum siap).


<img width="949" height="104" alt="pg-01-dcu" src="https://github.com/user-attachments/assets/2fef3bef-5da3-447c-997b-a4c0c608a1d3" />

<img width="948" height="260" alt="pg-02-dps-healthy" src="https://github.com/user-attachments/assets/4300803b-debe-4e5a-b60a-2044b1a490a2" />

<img width="722" height="573" alt="pg-03-log-replica" src="https://github.com/user-attachments/assets/9ded38fb-e0dd-45cb-8a2c-66fc31a98907" />


## 1.4 Pengujian Replikasi

### a. Cek status replikasi di primary

```bash
dep
psql -U zuser -d zdb
```
```sql
\x
SELECT * FROM pg_stat_replication;
SELECT * FROM pg_replication_slots;
```

Yang diperhatikan:
- `pg_stat_replication`: `usename = replicator`, `state = streaming`, `sync_state = async`, dan `sent_lsn/write_lsn/flush_lsn/replay_lsn` bernilai sama (tidak ada lag).
- `pg_replication_slots`: `slot_name = replication_slot`, `slot_type = physical`, `active = t`.

<img width="977" height="631" alt="pg-04-stat-replication" src="https://github.com/user-attachments/assets/1803d07a-6579-4b34-a100-51b039def8f7" />

<img width="391" height="441" alt="pg-05-replication-slots" src="https://github.com/user-attachments/assets/aae3c5bd-2b3e-48a3-8239-325214c90a8d" />


### b. Cek di replica

```bash
der
psql -U zuser -d zdb
```
```sql
SELECT pg_is_in_recovery();
```
Hasil `t` (true) berarti server tersebut adalah standby/replica.

<img width="612" height="106" alt="pg-06-primary-before-table" src="https://github.com/user-attachments/assets/196920d4-4138-406b-b102-7da279977ceb" />
`

### c. Uji manipulasi data

**Sebelum membuat tabel** — jalankan `\dt` di primary dan replica, keduanya `Did not find any tables.`

<img width="573" height="118" alt="pg-07-replica-before-table" src="https://github.com/user-attachments/assets/666bf2a0-d514-473f-8ee1-b127fa005deb" />

<img width="630" height="292" alt="pg-08-primary-insert" src="https://github.com/user-attachments/assets/af83263e-f26a-4ad8-b153-95d639ae45ae" />


**Di primary:**
```sql
CREATE TABLE testrep (first INT, second VARCHAR(15));
INSERT INTO testrep (first, second) VALUES (21, 'Test pertama');
INSERT INTO testrep (first, second) VALUES (23, 'Test ke dua');
SELECT * FROM testrep;
```

<img width="581" height="285" alt="pg-09-replica-replicated" src="https://github.com/user-attachments/assets/fd92700d-b28c-44d5-91c8-536c3163a67f" />


**Di replica:**
```sql
\dt
SELECT * FROM testrep;
```
Tabel `testrep` beserta 2 baris datanya muncul otomatis di replica tanpa operasi apa pun, ini membuktikan replikasi berjalan.

<img width="594" height="345" alt="pg-10-high-availability" src="https://github.com/user-attachments/assets/586f29e2-f780-418f-aa91-af25a9acb3c5" />


> Coba juga `INSERT` langsung di replica. Akan gagal dengan error `cannot execute INSERT in a read-only transaction`, karena replica hanya *read-only*.

## 1.5 High Availability (Failover)

Jika primary mati, replica bisa **dipromosikan** menjadi primary.

**1. Matikan primary:**
```bash
sudo docker-compose stop postgres_primary
```
<img width="721" height="84" alt="pg-11-stop-primary" src="https://github.com/user-attachments/assets/d4a574b9-a13b-4c4e-9cad-327da8be0e44" />


**2. Promosikan replica** (masuk ke psql di replica):
```sql
SELECT pg_is_in_recovery();   -- t (masih replica)
SELECT pg_promote();          -- t (berhasil dipromosikan)
SELECT pg_is_in_recovery();   -- f (sekarang primary, bisa read-write)
```
<img width="822" height="372" alt="pg-12-pg-promote" src="https://github.com/user-attachments/assets/ba39f1ad-63e5-47b7-afa7-9a8f26c4ac3a" />


Setelah dipromosikan, node tersebut keluar dari mode recovery dan menerima operasi tulis.

> **Catatan:** `pg_promote()` bersifat manual. Untuk produksi biasanya dipakai tool otomatis seperti Patroni atau repmgr. Selain itu, primary lama tidak otomatis kembali jadi replica; harus di-*rebuild* dengan `pg_basebackup` atau `pg_rewind`.

**3. Matikan semuanya:**
```bash
dcd
dps
```
<img width="553" height="140" alt="pg-13-dcd" src="https://github.com/user-attachments/assets/0e7e21cc-4def-410c-9eac-971f1c036244" />

---

# Bagian 2 — Replikasi Master-Master dengan Apache Ignite 3.1.0

## 2.1 Konsep Singkat

Apache Ignite 3 membentuk **cluster** dari beberapa node. Data didistribusikan dan direplikasi antar node menggunakan protokol konsensus **Raft**. Dari sudut pandang pengguna, koneksi ke node mana pun bisa dipakai untuk membaca dan menulis, sehingga sering disebut replikasi "Master-Master". Perubahan yang dilakukan di satu node otomatis terlihat di node lain.

> Secara teknis, di balik layar tiap partisi data punya satu *leader* Raft yang memproses tulis, tetapi klien tidak perlu peduli node mana pemimpinnya karena request diteruskan otomatis.

## 2.2 Penjelasan File

### `docker-compose.yaml`
- Menjalankan **3 node** (`node1`, `node2`, `node3`) dari image `apacheignite/ignite:3.1.0`.
- `JVM_MAX_MEM` / `JVM_MIN_MEM` = `4g` per node.
- Konfigurasi jaringan lewat `configs`: port klaster `3344` dan `nodeFinder` berisi ketiga node agar saling menemukan.
- Pemetaan port:

| Node | REST API (admin/CLI) | Koneksi aplikasi |
|---|---|---|
| node1 | `10300` | `10800` |
| node2 | `10301` | `10801` |
| node3 | `10302` | `10802` |

### `env.sh`

| Alias | Fungsi |
|---|---|
| `dcu` / `dcd` / `dps` | up, down, daftar container |
| `dcli` | menjalankan Ignite CLI di container |
| `dclm` | sama seperti `dcli` tetapi me-*mount* `./sql` ke `/opt/ignite/sql` agar file SQL bisa dieksekusi |

### Folder `sql/`
`schema.sql`, `current_catalog.sql`, `media_and_genre.sql`, `tracks.sql`, `ee_and_cust.sql`, `invoices.sql`: skema dan data contoh (database musik *Chinook*).

## 2.3 Menjalankan Cluster

```bash
cd ignite/src
source env.sh
dcu
dps
```

Cek alamat IP tiap node:
```bash
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <CONTAINER_ID>
ping <IP>
```

<img width="1103" height="173" alt="pg-01-dcu" src="https://github.com/user-attachments/assets/ef172af6-2019-46aa-a403-8adc2ddae913" />

<img width="1095" height="94" alt="pg-02-dps" src="https://github.com/user-attachments/assets/72cc458a-cac8-4d4f-badc-0ee2986b5719" />

<img width="1091" height="412" alt="pg-03-dlr" src="https://github.com/user-attachments/assets/045699d0-6cc2-4390-b061-d1122ec2ea66" />



## 2.4 Inisialisasi Cluster

```bash
dcli
```
Pilih `Y` saat ditanya konek ke `http://localhost:10300`, lalu:
```
cluster init --name=ignite3 --metastorage-group=node1,node2,node3
cluster status
```
Hasil yang diharapkan: `status: active`, `nodes: 3`.

- `--metastorage-group` dan CMG: node-node yang menyimpan metadata cluster. Semua 3 node dilibatkan supaya tahan jika 1 node mati.
- Inisialisasi hanya dilakukan **sekali**. Jika keluar CLI (Ctrl-D) lalu masuk lagi, tidak perlu init ulang.

<img width="1063" height="452" alt="pg-04-stat-replication" src="https://github.com/user-attachments/assets/d4658933-0a24-4ee9-b85e-6d1e28ec9944" />


## 2.5 Eksekusi SQL

Gunakan `dclm` (karena butuh mount `./sql`), konek ke `http://localhost:10300` (node1):

```
sql --file=/opt/ignite/sql/schema.sql
sql --file=/opt/ignite/sql/current_catalog.sql
sql --file=/opt/ignite/sql/media_and_genre.sql
sql --file=/opt/ignite/sql/tracks.sql
sql --file=/opt/ignite/sql/ee_and_cust.sql
sql --file=/opt/ignite/sql/invoices.sql
```

Lihat tabel yang terbentuk:
```
sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```
<img width="909" height="379" alt="pg-05-replication-slots" src="https://github.com/user-attachments/assets/d73dca48-3bf0-4463-b665-17b189802765" />

<img width="634" height="169" alt="pg-06-replica-recovery" src="https://github.com/user-attachments/assets/3a80a64d-b589-4847-b586-970c0fc2941a" />

<img width="658" height="110" alt="pg-07-before-table" src="https://github.com/user-attachments/assets/750bcfc9-bc2a-4424-be78-68219ccf978f" />



## 2.6 Verifikasi Replikasi di Node Lain

Dari CLI, pindah ke node2:
```
connect http://localhost:10301
sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```
Lalu jalankan query analitik (ringkasan penjualan bulanan):

```sql
SELECT
    CAST(EXTRACT(YEAR FROM i.InvoiceDate) AS VARCHAR) || '-' ||
    CASE
        WHEN EXTRACT(MONTH FROM i.InvoiceDate) < 10
        THEN '0' || CAST(EXTRACT(MONTH FROM i.InvoiceDate) AS VARCHAR)
        ELSE CAST(EXTRACT(MONTH FROM i.InvoiceDate) AS VARCHAR)
    END AS YearMonth,
    COUNT(DISTINCT i.InvoiceId) AS InvoiceCount,
    COUNT(DISTINCT i.CustomerId) AS CustomerCount,
    SUM(i.Total) AS MonthlyRevenue,
    AVG(i.Total) AS AverageOrderValue
FROM Invoice i
GROUP BY EXTRACT(YEAR FROM i.InvoiceDate), EXTRACT(MONTH FROM i.InvoiceDate)
ORDER BY YearMonth DESC;
```

Ulangi untuk node3 (`connect http://localhost:10302`).

<img width="747" height="201" alt="pg-08-insert-primary" src="https://github.com/user-attachments/assets/352a7f17-c201-4a33-bd16-8929b8b57504" />

<img width="606" height="201" alt="pg-09-verify-replica" src="https://github.com/user-attachments/assets/dab2d107-9550-4460-b22a-39e01ea6803e" />



<img width="354" height="52" alt="pg-10-read-only" src="https://github.com/user-attachments/assets/23515e8a-f725-4464-b1ca-5f672240da5c" />


**Analisis:** data dimasukkan hanya lewat node1, tetapi node2 dan node3 menghasilkan data dan hasil query yang sama persis. Artinya data sudah direplikasi ke seluruh node dalam cluster, sehingga replikasi Master-Master berhasil.

Matikan semua node:
```bash
dcd
```
<img width="1090" height="93" alt="pg-11-final-status" src="https://github.com/user-attachments/assets/6e8e0ee9-5d76-4975-b891-5ecc1e7d898f" />


---

# Perbandingan Kedua Model

| Aspek | PostgreSQL Streaming Replication | Apache Ignite 3 |
|---|---|---|
| Topologi | Single-primary, banyak replica | Cluster multi-node |
| Tulis | Hanya di primary | Bisa lewat node mana pun |
| Replica | Read-only (hot standby) | Semua node setara bagi klien |
| Mekanisme | Pengiriman WAL (asinkron) | Konsensus Raft |
| Failover | Manual (`pg_promote()`) / tool eksternal | Otomatis (pemilihan leader Raft) selama mayoritas node hidup |
| Konsistensi | *Eventual* (ada lag kecil pada mode async) | Konsisten kuat per partisi |

# Kesimpulan

1. Streaming replication PostgreSQL berhasil: tabel dan data yang dibuat di primary muncul otomatis di replica, dan replica dapat dipromosikan menjadi primary saat primary mati (HA).
2. Cluster Apache Ignite 3.1.0 dengan 3 node berhasil diinisialisasi, dan data yang dimasukkan lewat satu node dapat dibaca dari node lain.
3. Kedua pendekatan memberi *availability* yang lebih baik daripada server tunggal. Pilihannya bergantung pada kebutuhan: PostgreSQL cocok untuk arsitektur read-replica yang sederhana, Ignite cocok jika dibutuhkan cluster terdistribusi tanpa satu titik tulis tunggal.

# Referensi

- Modul 4 Praktikum Sistem Terdistribusi dan Terdesentralisasi (Dr. Bambang Purnomosidi D. P.)
- https://medium.com/@eremeykin/how-to-setup-single-primary-postgresql-replication-with-docker-compose-98c48f233bbf
- https://www.postgresql.org/docs/current/warm-standby.html
- https://ignite.apache.org/docs/ignite3/latest/
