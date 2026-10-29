# Laporan Praktikum Modul 4 — Konsistensi dan Replikasi pada Sistem Terdistribusi

Nama : Laesah Afenlia

NIM : 255410019

Prodi : IF 1

---
**TUJUAN**
Memahami cara kerja streaming replication pada PostgreSQL.
Menjalankan server primary dan replica PostgreSQL 18 menggunakan Docker Compose.
Menguji bahwa perubahan data di primary otomatis muncul di replica.
Memahami konsep High Availability (HA) dengan mempromosikan replica menjadi primary.
Mengonfigurasi dan menginisialisasi cluster Apache Ignite 3.1.0 dengan tiga node.
Menguji replikasi Master-Master dengan menjalankan perintah SQL dan memeriksa datanya di setiap node.
DASAR TEORI
**1. Replikasi dan Konsistensi Data**

Replikasi adalah proses menyalin data dari satu server ke server lain supaya data tetap tersedia walaupun salah satu server bermasalah. Tantangan utamanya adalah menjaga konsistensi, yaitu memastikan semua salinan data memiliki isi yang sama atau setidaknya segera menjadi sama.

**2. Streaming Replication PostgreSQL**

Pada streaming replication, server primary mengirim catatan WAL (Write-Ahead Log) ke server replica secara terus-menerus melalui koneksi jaringan. Replica memutar ulang catatan tersebut sehingga datanya mengikuti primary. Primary menerima operasi baca dan tulis, sedangkan replica hanya melayani pembacaan (hot standby). Pada praktikum ini replikasi berjalan secara asinkron, artinya primary tidak menunggu konfirmasi replica sebelum menyelesaikan transaksi. Replication slot dipakai agar primary tidak menghapus WAL yang belum diterima replica.

**3. High Availability (HA)**

High Availability adalah upaya menjaga layanan tetap tersedia ketika terjadi gangguan. Pada PostgreSQL, jika primary mati maka replica dapat dipromosikan menjadi primary baru dengan fungsi pg_promote().

**4. Replikasi Master-Master Apache Ignite**

Apache Ignite 3 membentuk cluster dari beberapa node, dan data dikelola serta direplikasi antar node menggunakan protokol konsensus Raft. Dari sisi pengguna, koneksi ke node mana pun dapat dipakai untuk membaca dan menulis, sehingga modelnya sering disebut Master-Master.

**5. Docker dan Docker Compose**

Docker menjalankan aplikasi di dalam container yang terisolasi, sedangkan Docker Compose mengatur beberapa container sekaligus lewat satu file konfigurasi. Keduanya dipakai untuk menjalankan PostgreSQL dan Apache Ignite pada praktikum ini.

# 4.1 Streaming Replication Menggunakan PostgreSQL

Membuat dua server PostgreSQL 18 yang saling terhubung, yaitu primary dan replica. Setiap perubahan data di primary akan direplikasi ke replica.<br>

## 4.1.1 Prasyarat
Memastikan Docker dan Docker Compose sudah terpasang serta layanan Docker sudah aktif:<br>

<img width="393" height="125" alt="pg-00-prasyarat" src="https://github.com/user-attachments/assets/8ec6e52f-ae31-4d4f-a980-c60552ca1041" />


<img src="postgresql/src/img/prasyarat-docker.png" width="700"> <br>

Pembahasan : <br>

Sebelum mengonfigurasi replikasi, dipastikan dulu bahwa Docker dan Docker Compose sudah terpasang dan daemon dockerd sudah berjalan. Pengecekan dilakukan dengan docker --version dan docker compose version. Petunjuk modul memakai Linux, jadi pada sistem operasi lain perlu penyesuaian seperlunya. <br>

## 4.1.2 Struktur Direktori dan File

Membuat 3 file di dalam direktori postgresql/src:<br>

<img width="239" height="219" alt="docker-compose" src="https://github.com/user-attachments/assets/a3e822d7-c72d-4551-b14f-b4629380c507" />


Pembahasan : <br>

Pada tahap ini disiapkan tiga berkas utama untuk streaming replication. Berkas 00_init.sql dijalankan otomatis saat primary pertama kali dibuat, isinya membuat user replicator yang berhak melakukan replikasi dan physical replication slot bernama replication_slot. Berkas docker-compose.yaml mendefinisikan dua service dengan image postgres:18.3-alpine3.23. Service postgres_primary berjalan di port 5432 dengan user zuser, database zdb, dan konfigurasi WAL seperti wal_level=replica, hot_standby=on, dan max_wal_senders=10, serta mengizinkan user replicator untuk koneksi replikasi. Service postgres_replica berjalan di port host 5433 dan menjalankan pg_basebackup ke primary, diulang tiap 5 detik sampai berhasil. Opsi -R pada perintah itu membuat standby.signal dan primary_conninfo otomatis sehingga server langsung berjalan sebagai standby. Berkas ketiga, env.sh, berisi alias seperti dcu, dcd, dps, dep, der, dlp, dan dlr supaya perintah Docker lebih singkat. Nama container diawali nama direktori, karena direktorinya src maka namanya src-postgres_primary-1 dan src-postgres_replica-1. <br>

## 4.1.3 Menjalankan docker-compose

Perintah dijalankan dari direktori `postgresql/src`. Alias dimuat dulu dengan `source env.sh`, lalu `dcu` dijalankan. Karena image belum ada di komputer, Docker Compose mengambilnya dulu dari DockerHub, kemudian membuat network dan menjalankan kedua container.

```bash
cd postgresql/src
source env.sh
dcu
```

<img src="https://github.com/user-attachments/assets/2fef3bef-5da3-447c-997b-a4c0c608a1d3" width="700"> <br>

Kemudian periksa status container:<br>

<img src="https://github.com/user-attachments/assets/4300803b-debe-4e5a-b60a-2044b1a490a2" width="700"> <br>

Periksa juga log replica:<br>

<img src="https://github.com/user-attachments/assets/9ded38fb-e0dd-45cb-8a2c-66fc31a98907" width="700"> <br>

Pembahasan : <br>

Perintah dcu menjalankan dua container, yaitu src-postgres_primary-1 dan src-postgres_replica-1. Pada hasil docker ps, kolom STATUS awalnya masih health: starting, lalu berubah menjadi healthy setelah beberapa saat, artinya kedua server sudah lolos pengecekan pg_isready. Pada log replica terlihat pesan pg_basebackup failed. Retrying in 5 seconds ... di awal. Ini wajar karena primary belum siap menerima koneksi. Setelah primary siap muncul Backup done, starting replica..., lalu log menunjukkan replica masuk ke standby mode dan started streaming WAL from primary. Dengan begitu replica sudah mulai menerima WAL dari primary. <br>

## 4.1.4 Pengujian

Untuk melihat apakah replikasi berfungsi, saya melihat tabel diagnostik PostgreSQL dan mencoba memanipulasi data tabel.

Pertama, masuk ke container primary lalu buka `psql`, kemudian lihat isi `pg_stat_replication`.

```bash
dep
psql -U zuser -d zdb
```

```sql
\x
SELECT * FROM pg_stat_replication;
```

Hasilnya menunjukkan `usename` bernilai `replicator`, `state` bernilai `streaming`, dan `sync_state` bernilai `async`, artinya replikasi berjalan secara asinkron. Nilai `sent_lsn`, `write_lsn`, `flush_lsn`, dan `replay_lsn` sama, yang berarti replica tidak tertinggal dari primary.

<img width="1096" height="186" alt="pg-08-primary-stat-replication" src="https://github.com/user-attachments/assets/61137c2d-8c83-4f6d-a07d-0d66afbbc1c8" />

Selanjutnya lihat `pg_replication_slots`.

```sql
SELECT * FROM pg_replication_slots;
```

Terlihat slot `replication_slot` dengan `slot_type` bernilai `physical` dan `active` bernilai `t`, jadi slot sedang dipakai oleh replica.

<img width="1096" height="158" alt="pg-09-primary-replication-slots" src="https://github.com/user-attachments/assets/59d21dcb-0c70-41e8-b03c-125e8abf8ed5" />

Pada server replica, masuk ke container replica dan cek statusnya.

```bash
der
psql -U zuser -d zdb
```

```sql
SELECT pg_is_in_recovery();
```

Hasil `t` (true) menunjukkan bahwa server tersebut adalah replica atau standby yang hanya mereplikasi data dan menyediakan akses baca.

Untuk melihat efek replikasi, saya membuat tabel lalu mengisinya di primary, setelah itu melihat kondisi di replica.

<img width="612" height="106" alt="pg-06-primary-before-table" src="https://github.com/user-attachments/assets/196920d4-4138-406b-b102-7da279977ceb" />

**Sebelum pembuatan tabel.** `\dt` dijalankan di primary dan di replica, keduanya menampilkan `Did not find any tables.` karena belum ada tabel.

<img width="635" height="114" alt="pg-07-replica-before-table" src="https://github.com/user-attachments/assets/6fae6054-9d45-454e-a607-ba69e9b21a03" />

<img width="698" height="142" alt="pg-07-primary-before-table" src="https://github.com/user-attachments/assets/93169d14-2167-4472-b664-ba44116d2cb3" />



**Manipulasi data pada primary.** Saya membuat tabel `testrep`, mengisi dua baris data, lalu menampilkannya.

```sql
CREATE TABLE testrep (first INT, second VARCHAR(15));
INSERT INTO testrep (first, second) VALUES (21, 'Test pertama');
INSERT INTO testrep (first, second) VALUES (23, 'Test ke dua');
SELECT * FROM testrep;
```
**Berikut adalah hasil pengecekan status replikasi dan slot replikasi pada server primary:**

<img width="1096" height="186" alt="pg-08-primary-stat-replication" src="https://github.com/user-attachments/assets/61137c2d-8c83-4f6d-a07d-0d66afbbc1c8" />

<img width="1096" height="158" alt="pg-09-primary-replication-slots" src="https://github.com/user-attachments/assets/59d21dcb-0c70-41e8-b03c-125e8abf8ed5" /> 


<img width="630" height="292" alt="pg-08-primary-insert" src="https://github.com/user-attachments/assets/af83263e-f26a-4ad8-b153-95d639ae45ae" />

**Hasil di replica.** Tanpa melakukan apa pun di replica, tabel `testrep` dan kedua barisnya sudah muncul. Ini menunjukkan bahwa manipulasi data di primary otomatis direplikasi ke replica.

```sql
\dt
SELECT * FROM testrep;
```

<img width="581" height="285" alt="pg-09-replica-replicated" src="https://github.com/user-attachments/assets/fd92700d-b28c-44d5-91c8-536c3163a67f" />

## 4.1.5 High-Availability

Model replikasi ini bisa dipakai untuk kebutuhan *High Availability* (HA). Jika server primary mati, salah satu replica bisa menggantikannya.

**Primary server mati.** Saya mematikan container primary dengan perintah berikut.

```bash
sudo docker-compose stop postgres_primary
```

<img width="721" height="84" alt="pg-11-stop-primary" src="https://github.com/user-attachments/assets/d4a574b9-a13b-4c4e-9cad-327da8be0e44" />

Pembahasan : <br>

Container primary dihentikan untuk mensimulasikan primary yang mengalami gangguan. Pada kondisi ini tidak ada server yang bisa menerima operasi tulis, sehingga replica perlu dipromosikan. <br> 

**Mempromosikan replica.** Setelah primary diketahui mati, replica dipromosikan menjadi primary, dari yang awalnya hanya bisa dibaca menjadi bisa dibaca dan ditulis. Di `psql` pada replica, `pg_is_in_recovery()` pertama kali bernilai `t`, lalu `pg_promote()` dijalankan dan hasilnya `t`, kemudian `pg_is_in_recovery()` dicek lagi dan sekarang bernilai `f`. Nilai `f` berarti server sudah keluar dari mode recovery dan berperan sebagai primary.

```sql
SELECT pg_is_in_recovery();
SELECT pg_promote();
SELECT pg_is_in_recovery();
```
<img width="624" height="173" alt="pg_stat_replication" src="https://github.com/user-attachments/assets/ce3520d2-02b5-4e91-b000-3ac62c993751" />

<img width="628" height="179" alt="pg_promote()" src="https://github.com/user-attachments/assets/68010c5a-90c9-4c35-892f-bca8785d01f6" />

Pembahasan : <br>

Sebelum promosi, pg_is_in_recovery() bernilai t yang berarti server masih replica. Setelah pg_promote() dijalankan dan mengembalikan t, pengecekan berikutnya bernilai f. Nilai f menunjukkan server sudah keluar dari mode recovery dan kini berperan sebagai primary yang dapat membaca dan menulis. <br>

**Mematikan semua server.** Terakhir, primary dan replica dimatikan dengan `dcd`, lalu `dps` dijalankan untuk memastikan sudah tidak ada container yang berjalan.

```bash
dcd
dps
```

<img width="553" height="140" alt="pg-13-dcd" src="https://github.com/user-attachments/assets/0e7e21cc-4def-410c-9eac-971f1c036244" />

Pembahasan : <br>

Perintah dcd menghapus kedua container dan network, dan dps yang kosong memastikan sudah tidak ada container yang berjalan. <br>
---

# 4.2 Replikasi Master-Master Menggunakan Apache Ignite

Membuat cluster Apache Ignite 3.1.0 dengan tiga node. Semua node dapat dipakai untuk membaca dan menulis, dan perubahan di satu node akan terlihat di node lain.<br>

## 4.2.1 Prasyarat

Prasyaratnya sama dengan bagian sebelumnya.

1. Docker sudah terpasang.
2. Docker Compose sudah terpasang.
3. Daemon `dockerd` sudah aktif.
4. Petunjuk modul memakai Linux, jadi kalau memakai sistem operasi lain perlu ada penyesuaian seperlunya.
   
<img src="postgresql/src/img/prasyarat-docker.png" width="700"> <br>
Pembahasan : <br>

Kebutuhan lingkungan sama seperti sebelumnya, sehingga cukup dipastikan Docker dan Docker Compose masih dapat dipakai dan layanan Docker dalam keadaan aktif. <br>

## 4.2.2 Struktur Direktori dan File

Menyiapkan direktori ignite/src berisi docker-compose.yaml, folder sql, dan env.sh:<br>

<img src="ignite/src/img/struktur-folder.png" width="700"> <br>

Pembahasan : <br>

Bagian ini hanya memerlukan satu berkas docker-compose.yaml dan kumpulan file SQL dari dokumentasi Apache Ignite. Berkas compose menjalankan tiga node (node1, node2, node3) dari image apacheignite/ignite:3.1.0, dengan port klaster 3344 dan nodeFinder berisi ketiga node supaya saling menemukan. Folder sql berisi skema dan data contoh database musik Chinook, yaitu schema.sql, current_catalog.sql, media_and_genre.sql, tracks.sql, ee_and_cust.sql, dan invoices.sql. Berkas env.sh berisi alias dcu, dcd, dan dps seperti sebelumnya, ditambah dcli yang menjalankan Ignite CLI di container, dan dclm yang sama seperti dcli tetapi me-mount folder ./sql ke /opt/ignite/sql agar file SQL bisa dibaca dari dalam container. <br>

## 4.2.3 Menjalankan docker-compose
**1. Menjalankan Cluster**

Dari direktori `ignite/src`, alias dimuat dengan `source env.sh` lalu cluster dijalankan dengan `dcu`. Saat pertama kali, image Apache Ignite diambil dulu dari DockerHub, kemudian network dan ketiga container (`node1`, `node2`, `node3`) dibuat dan dijalankan.

```bash
cd ignite/src
source env.sh
dcu
```

<img width="1103" height="173" alt="ignite-01-dcu" src="https://github.com/user-attachments/assets/ef172af6-2019-46aa-a403-8adc2ddae913" />

**2. Memeriksa Status Container**

Setelah itu `dps` dipakai untuk memastikan ketiga node sudah berjalan.

```bash
dps
```
**3. Memeriksa IP Address Masing-Masing Node**
```bash
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <CONTAINER ID>
ping <IP>
```

<img width="1091" height="412" alt="ignite-03-inspect-ping" src="https://github.com/user-attachments/assets/045699d0-6cc2-4390-b061-d1122ec2ea66" />

Pembahasan : <br>

Perintah dcu mengambil image Apache Ignite dari DockerHub lalu menjalankan tiga container node. Pada hasil dps, ketiganya berstatus aktif. Port 10300 sampai 10302 adalah REST API untuk administrasi (http://localhost:10300 untuk node1, 10301 untuk node2, dan 10302 untuk node3), sedangkan port 10800 sampai 10802 dipakai untuk koneksi aplikasi. Perintah docker inspect menampilkan alamat IP tiap node di jaringan Docker, dan ping dipakai untuk memastikan ketiganya bisa dijangkau. <br>


## 4.2.4 Inisialisasi Cluster

Ignite CLI dijalankan dengan alias `dcli`. Saat ditanya apakah ingin terhubung ke node default `http://localhost:10300`, saya menjawab `Y`. Kemudian cluster diinisialisasi dengan perintah berikut.

```
dcli
cluster init --name=ignite3 --metastorage-group=node1,node2,node3
cluster status
```
<img width="1063" height="452" alt="ignite-04-cluster-init" src="https://github.com/user-attachments/assets/d4658933-0a24-4ee9-b85e-6d1e28ec9944" />

Pembahasan : <br>

CLI terhubung ke node default http://localhost:10300. Perintah cluster init membuat cluster bernama ignite3 dengan ketiga node sebagai anggota metastorage group, supaya metadata cluster tetap aman bila satu node mati. Perintah cluster status kemudian menunjukkan cluster berstatus active dengan 3 node. Inisialisasi hanya perlu dilakukan sekali, jadi setelah keluar dari CLI dengan Ctrl-D dan masuk lagi, tidak perlu init ulang. <br>

## 4.2.5 Eksekusi Perintah-Perintah SQL
**1. Eksekusi Skema dan Dataset di Node 1**

File SQL perlu bisa diakses dari container, karena itu CLI dijalankan dengan `dclm` yang me-*mount* folder `sql`. Setelah terhubung ke `http://localhost:10300`, file SQL dieksekusi satu per satu.

```
sql --file=/opt/ignite/sql/schema.sql
sql --file=/opt/ignite/sql/current_catalog.sql
sql --file=/opt/ignite/sql/media_and_genre.sql
sql --file=/opt/ignite/sql/tracks.sql
sql --file=/opt/ignite/sql/ee_and_cust.sql
sql --file=/opt/ignite/sql/invoices.sql
```

<img width="909" height="379" alt="ignite-05-sql-files" src="https://github.com/user-attachments/assets/d73dca48-3bf0-4463-b665-17b189802765" />

<img width="634" height="169" alt="ignite-06-sql-files" src="https://github.com/user-attachments/assets/3a80a64d-b589-4847-b586-970c0fc2941a" />

Pembahasan : <br>

File schema.sql membuat tabel-tabel, lalu file lainnya mengisi data katalog, tracks, karyawan, pelanggan, dan invoice. Keluaran Updated ... rows menunjukkan jumlah baris yang berhasil dimasukkan oleh tiap perintah. Seluruh perintah ini hanya dijalankan di node1. <br>

**2. Melihat Tabel yang Terbentuk**
Kemudian masuk ke mode SQL dan lihat tabel yang sudah terbentuk.

```
sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```

<img width="658" height="110" alt="ignite-07-system-tables" src="https://github.com/user-attachments/assets/750bcfc9-bc2a-4424-be78-68219ccf978f" />

**3. Verifikasi di Node 2**
Untuk memeriksa apakah perubahan di `http://localhost:10300` sudah dipropagasi ke node lain, saya terhubung ke node2 di `http://localhost:10301`, lalu menjalankan query yang sama.

```
connect http://localhost:10301
sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```

<img width="747" height="201" alt="ignite-08-node2" src="https://github.com/user-attachments/assets/352a7f17-c201-4a33-bd16-8929b8b57504" />

Hasilnya node2 sudah berisi tabel-tabel seperti yang tadi dibuat di node1. Selanjutnya saya menjalankan query analitik berupa ringkasan penjualan bulanan.

```sql
-- Monthly sales summary for the last 12 months
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
FROM
    Invoice i
GROUP BY
    EXTRACT(YEAR FROM i.InvoiceDate), EXTRACT(MONTH FROM i.InvoiceDate)
ORDER BY
    YearMonth DESC;
```

<img width="606" height="201" alt="ignite-09-query-node2" src="https://github.com/user-attachments/assets/dab2d107-9550-4460-b22a-39e01ea6803e" />

**4. Verifikasi di Node 3**
Hal yang sama dilakukan di node3 (`http://localhost:10302`).

```
connect http://localhost:10302
```

<img width="354" height="52" alt="ignite-10-node3" src="https://github.com/user-attachments/assets/23515e8a-f725-4464-b1ca-5f672240da5c" />
Pembahasan : <br>

Data hanya dimasukkan lewat node1, tetapi saat terhubung ke node2 tabel-tabelnya sudah ada dan query analitik langsung bisa dijalankan. Hal yang sama dilakukan di node3. Hasil query di kedua node sama dengan data yang dimasukkan di node1. Dengan demikian data sudah direplikasi ke seluruh node dalam cluster, sehingga replikasi Master-Master pada Apache Ignite 3.1.0 berjalan. <br>

**5. Mematikan Semua Node**
Data hanya dimasukkan lewat node1, tetapi node2 dan node3 ternyata juga sudah berisi data dan memberi hasil query yang sama. Dengan demikian terlihat bahwa telah terjadi replikasi Master-Master pada cluster Apache Ignite 3.1.0.

Terakhir, semua node dimatikan dengan `dcd`.

```bash
dcd
```

<img width="1090" height="93" alt="ignite-11-dcd" src="https://github.com/user-attachments/assets/6e8e0ee9-5d76-4975-b891-5ecc1e7d898f" />

Pembahasan : <br>

Perintah dcd menghapus ketiga container node beserta network-nya sehingga praktikum selesai dan lingkungan kembali bersih. <br>
