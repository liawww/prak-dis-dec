# Laporan Praktikum Modul 4 — Konsistensi dan Replikasi pada Sistem Terdistribusi

Nama : Laesah Afenlia

NIM : 255410019

Prodi : IF 1

---

# 4.1 Streaming Replication Menggunakan PostgreSQL

Pada bagian ini saya mengonfigurasi *streaming replication* di PostgreSQL 18 menggunakan Docker Compose. Prinsipnya sama seperti versi PostgreSQL sebelumnya, hanya saja direktori data tidak lagi berada di `/var/lib/postgresql/data` tetapi di `/var/lib/postgresql/18/docker`. Ada dua server yang dijalankan: satu server *primary* yang menerima operasi baca dan tulis, dan satu server *replica* (*standby*) yang terus menyalin data dari primary lewat pengiriman WAL (*Write-Ahead Log*) sehingga isinya selalu sama dengan primary.

## 4.1.1 Prasyarat

Sebelum mengerjakan, hal yang perlu sudah tersedia adalah sebagai berikut.

1. Docker sudah terpasang.
2. Docker Compose sudah terpasang.
3. Daemon `dockerd` sudah aktif.
4. Petunjuk modul memakai Linux, jadi kalau memakai sistem operasi lain perlu ada penyesuaian seperlunya.

## 4.1.2 Struktur Direktori dan File

Ada satu direktori yang berisi tiga file, yaitu `00_init.sql`, `docker-compose.yaml`, dan `env.sh`. Di repo ini direktorinya bernama `src` dan berada di dalam folder `postgresql`.

```
postgresql/
└── src/
    ├── 00_init.sql
    ├── docker-compose.yaml
    └── env.sh
```

**00_init.sql** berisi perintah SQL yang dijalankan otomatis ketika server primary pertama kali diinisialisasi. Fungsinya membuat user yang dipakai untuk replikasi dan membuat slot replikasinya.

```sql
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replicator_password';
SELECT pg_create_physical_replication_slot('replication_slot');
```

Baris pertama membuat user `replicator` dengan hak `REPLICATION`, yang nantinya dipakai replica untuk menarik data dari primary. Baris kedua membuat *physical replication slot* bernama `replication_slot` supaya primary tidak menghapus WAL yang belum diterima oleh replica.

**docker-compose.yaml** adalah konfigurasi yang dijalankan oleh `docker-compose`.

```yaml
x-postgres-common:
  &postgres-common
  image: postgres:18.3-alpine3.23
  user: postgres
  restart: always
  healthcheck:
    test: 'pg_isready -U zuser --dbname=zdb'
    interval: 10s
    timeout: 5s
    retries: 5

services:
  postgres_primary:
    <<: *postgres-common
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: zuser
      POSTGRES_DB: zdb
      POSTGRES_PASSWORD: zpass
      POSTGRES_HOST_AUTH_METHOD: "scram-sha-256\nhost replication replicator 0.0.0.0/0 md5"
      POSTGRES_INITDB_ARGS: "--auth-host=scram-sha-256"
    command: |
      postgres
      -c wal_level=replica
      -c hot_standby=on
      -c max_wal_senders=10
      -c max_replication_slots=10
      -c hot_standby_feedback=on
    volumes:
      - ./00_init.sql:/docker-entrypoint-initdb.d/00_init.sql

  postgres_replica:
    <<: *postgres-common
    ports:
      - 5433:5432
    environment:
      PGUSER: replicator
      PGPASSWORD: replicator_password
      PGDATA: /var/lib/postgresql/18/docker
    command: |
      bash -c "
      until pg_basebackup --pgdata=/var/lib/postgresql/18/docker -R --slot=replication_slot --host=postgres_primary --port=5432 -X stream; do
        echo 'pg_basebackup failed. Retrying in 5 seconds ...'
        sleep 5
      done
      echo 'Backup done, starting replica...'
      chmod 0700 /var/lib/postgresql/18/docker
      postgres
      "
    depends_on:
      - postgres_primary
```

Bagian `x-postgres-common` adalah template yang dipakai bersama oleh kedua service, isinya image `postgres:18.3-alpine3.23`, user `postgres`, kebijakan `restart: always`, dan *healthcheck* memakai `pg_isready`. Service `postgres_primary` berjalan di port 5432 dengan user `zuser`, database `zdb`, dan password `zpass`. Pengaturan `POSTGRES_HOST_AUTH_METHOD` mengatur `pg_hba.conf` agar koneksi biasa memakai `scram-sha-256` dan user `replicator` boleh melakukan koneksi replikasi. Pada `command` primary ada beberapa parameter, yaitu `wal_level=replica`, `hot_standby=on`, `max_wal_senders=10`, `max_replication_slots=10`, dan `hot_standby_feedback=on`. Service `postgres_replica` berjalan di port host 5433 yang diarahkan ke port 5432 container, memakai `PGUSER=replicator`, dan menyimpan data di `/var/lib/postgresql/18/docker`. Perintahnya menjalankan `pg_basebackup` ke primary dan mengulang tiap 5 detik sampai berhasil, lalu mengubah izin direktori data menjadi `0700`, kemudian menjalankan `postgres`. Opsi `-R` membuat file `standby.signal` dan `primary_conninfo` secara otomatis. Replica baru dijalankan setelah primary karena ada `depends_on`.

**env.sh** berisi alias shell untuk mempersingkat penulisan perintah, sifatnya opsional. Nama container diawali dengan nama direktori, karena file disimpan di direktori `src` maka nama containernya menjadi `src-postgres_primary-1` dan `src-postgres_replica-1`.

```bash
alias dcu="sudo docker-compose up -d"
alias dcd="sudo docker-compose down"
alias dps="sudo docker ps"
alias der="sudo docker exec -it src-postgres_replica-1 bash"
alias dep="sudo docker exec -it src-postgres_primary-1 bash"
alias dlr="sudo docker logs src-postgres_replica-1"
alias dlp="sudo docker logs src-postgres_primary-1"
```

`docker-compose up` dipakai untuk mengaktifkan beberapa container sekaligus berdasarkan `docker-compose.yaml`, `docker-compose down` untuk mematikannya, `docker ps` untuk melihat daftar container, `docker exec` untuk menjalankan proses (dalam hal ini bash) di container yang sedang berjalan, dan `docker logs` untuk menampilkan log container.

## 4.1.3 Menjalankan docker-compose

Perintah dijalankan dari direktori `postgresql/src`. Alias dimuat dulu dengan `source env.sh`, lalu `dcu` dijalankan. Karena image belum ada di komputer, Docker Compose mengambilnya dulu dari DockerHub, kemudian membuat network dan menjalankan kedua container.

```bash
cd postgresql/src
source env.sh
dcu
```

<img width="949" height="104" alt="pg-01-dcu" src="https://github.com/user-attachments/assets/2fef3bef-5da3-447c-997b-a4c0c608a1d3" />

Setelah itu saya cek dengan `dps`. Yang diperhatikan terutama kolom STATUS, kalau berjalan dengan baik akan muncul `healthy`.

```bash
dps
```

<img width="948" height="260" alt="pg-02-dps-healthy" src="https://github.com/user-attachments/assets/4300803b-debe-4e5a-b60a-2044b1a490a2" />

Kalau belum `healthy`, penyebabnya bisa dilacak lewat log dengan `dlp` untuk primary dan `dlr` untuk replica. Pada log replica terlihat pesan `pg_basebackup failed. Retrying in 5 seconds ...` di awal, ini wajar karena primary belum siap menerima koneksi. Setelah primary siap, muncul `Backup done, starting replica...` dan replica mulai melakukan streaming WAL dari primary.

```bash
dlr
```

<img width="722" height="573" alt="pg-03-log-replica" src="https://github.com/user-attachments/assets/9ded38fb-e0dd-45cb-8a2c-66fc31a98907" />

Pada titik ini sudah ada dua instance PostgreSQL 18.3, yaitu `src-postgres_primary-1` sebagai server utama yang menerima operasi *read-write*, dan `src-postgres_replica-1` sebagai *standby* yang menyinkronkan datanya dengan server utama. Semua operasi penulisan dan manipulasi data di server utama akan direplikasi ke replica.

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

<img width="977" height="631" alt="pg-04-stat-replication" src="https://github.com/user-attachments/assets/1803d07a-6579-4b34-a100-51b039def8f7" />

Selanjutnya lihat `pg_replication_slots`.

```sql
SELECT * FROM pg_replication_slots;
```

Terlihat slot `replication_slot` dengan `slot_type` bernilai `physical` dan `active` bernilai `t`, jadi slot sedang dipakai oleh replica.

<img width="391" height="441" alt="pg-05-replication-slots" src="https://github.com/user-attachments/assets/aae3c5bd-2b3e-48a3-8239-325214c90a8d" />

Pada server replica, masuk ke container replica dan cek statusnya.

```bash
der
psql -U zuser -d zdb
```

```sql
SELECT pg_is_in_recovery();
```

Hasil `t` (true) menunjukkan bahwa server tersebut adalah replica atau standby yang hanya mereplikasi data dan menyediakan akses baca.

<img width="612" height="106" alt="pg-06-primary-before-table" src="https://github.com/user-attachments/assets/196920d4-4138-406b-b102-7da279977ceb" />

Untuk melihat efek replikasi, saya membuat tabel lalu mengisinya di primary, setelah itu melihat kondisi di replica.

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

**Mempromosikan replica.** Setelah primary diketahui mati, replica dipromosikan menjadi primary, dari yang awalnya hanya bisa dibaca menjadi bisa dibaca dan ditulis. Di `psql` pada replica, `pg_is_in_recovery()` pertama kali bernilai `t`, lalu `pg_promote()` dijalankan dan hasilnya `t`, kemudian `pg_is_in_recovery()` dicek lagi dan sekarang bernilai `f`. Nilai `f` berarti server sudah keluar dari mode recovery dan berperan sebagai primary.

```sql
SELECT pg_is_in_recovery();
SELECT pg_promote();
SELECT pg_is_in_recovery();
```
<img width="624" height="173" alt="pg_stat_replication" src="https://github.com/user-attachments/assets/ce3520d2-02b5-4e91-b000-3ac62c993751" />

<img width="628" height="179" alt="pg_promote()" src="https://github.com/user-attachments/assets/68010c5a-90c9-4c35-892f-bca8785d01f6" />


**Mematikan semua server.** Terakhir, primary dan replica dimatikan dengan `dcd`, lalu `dps` dijalankan untuk memastikan sudah tidak ada container yang berjalan.

```bash
dcd
dps
```

<img width="553" height="140" alt="pg-13-dcd" src="https://github.com/user-attachments/assets/0e7e21cc-4def-410c-9eac-971f1c036244" />

---

# 4.2 Replikasi Master-Master Menggunakan Apache Ignite

Pada bagian ini saya mengonfigurasi *cluster* Apache Ignite 3 menggunakan Apache Ignite 3.1.0. Mode replikasi ini sering disebut Master-Master karena semua node bisa berperan sebagai primary, artinya bisa dipakai untuk membaca maupun menulis. Perubahan yang dilakukan di satu node akan terlihat di node lainnya dalam satu cluster.

## 4.2.1 Prasyarat

Prasyaratnya sama dengan bagian sebelumnya.

1. Docker sudah terpasang.
2. Docker Compose sudah terpasang.
3. Daemon `dockerd` sudah aktif.
4. Petunjuk modul memakai Linux, jadi kalau memakai sistem operasi lain perlu ada penyesuaian seperlunya.

## 4.2.2 Struktur Direktori dan File

Untuk bagian ini hanya diperlukan satu file `docker-compose.yaml` dan sekumpulan file perintah SQL yang diperoleh dari dokumentasi Apache Ignite (halaman *explore SQL*). Selain itu ada `env.sh` untuk shortcut perintah. Di repo ini semuanya ada di `ignite/src`.

```
ignite/
└── src/
    ├── docker-compose.yaml
    ├── env.sh
    └── sql/
        ├── schema.sql
        ├── current_catalog.sql
        ├── media_and_genre.sql
        ├── tracks.sql
        ├── ee_and_cust.sql
        └── invoices.sql
```

`docker-compose.yaml` menjalankan tiga node (`node1`, `node2`, `node3`) dari image `apacheignite/ignite:3.1.0`, masing-masing dengan memori JVM 4 GB. Konfigurasi jaringannya memakai port klaster 3344 dan `nodeFinder` berisi ketiga node supaya saling menemukan.

Folder `sql` berisi skema dan data contoh database musik Chinook, yaitu `schema.sql`, `current_catalog.sql`, `media_and_genre.sql`, `tracks.sql`, `ee_and_cust.sql`, dan `invoices.sql`.

`env.sh` berisi alias berikut.

```bash
alias dcu="sudo docker-compose up -d"
alias dcd="sudo docker-compose down"
alias dps="sudo docker ps"
alias dclm="sudo docker run --rm -it --network=host -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 -v ./sql:/opt/ignite/sql apacheignite/ignite:3.1.0 cli"
alias dcli="sudo docker run --rm -it --network=host -e LANG=C.UTF-8 -e LC_ALL=C.UTF-8 apacheignite/ignite:3.1.0 cli"
```

`dcu`, `dcd`, dan `dps` fungsinya sama seperti di bagian PostgreSQL. `dcli` menjalankan Ignite CLI di dalam container, sedangkan `dclm` sama seperti `dcli` tetapi folder `./sql` di komputer lokal di-*mount* ke `/opt/ignite/sql` di container supaya file SQL bisa dieksekusi dari CLI.

## 4.2.3 Menjalankan docker-compose

Dari direktori `ignite/src`, alias dimuat dengan `source env.sh` lalu cluster dijalankan dengan `dcu`. Saat pertama kali, image Apache Ignite diambil dulu dari DockerHub, kemudian network dan ketiga container (`node1`, `node2`, `node3`) dibuat dan dijalankan.

```bash
cd ignite/src
source env.sh
dcu
```

<img width="1103" height="173" alt="ignite-01-dcu" src="https://github.com/user-attachments/assets/ef172af6-2019-46aa-a403-8adc2ddae913" />

Setelah itu `dps` dipakai untuk memastikan ketiga node sudah berjalan.

```bash
dps
```

<img width="1095" height="94" alt="ignite-02-dps" src="https://github.com/user-attachments/assets/72cc458a-cac8-4d4f-badc-0ee2986b5719" />

Dari ketiga node tersebut ada port yang bisa dipakai. Port 10300 sampai 10302 adalah REST API untuk operasi administratif, yaitu `http://localhost:10300` untuk node1, `http://localhost:10301` untuk node2, dan `http://localhost:10302` untuk node3. Port-port ini bisa langsung dihubungkan saat menjalankan Ignite CLI. Port 10800 sampai 10802 dipakai untuk koneksi dari aplikasi.

Alamat IP masing-masing node bisa diketahui dengan perintah `docker inspect`, lalu konektivitasnya dicek memakai `ping`.

```bash
sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <CONTAINER ID>
ping <IP>
```

<img width="1091" height="412" alt="ignite-03-inspect-ping" src="https://github.com/user-attachments/assets/045699d0-6cc2-4390-b061-d1122ec2ea66" />

## 4.2.4 Inisialisasi Cluster

Ignite CLI dijalankan dengan alias `dcli`. Saat ditanya apakah ingin terhubung ke node default `http://localhost:10300`, saya menjawab `Y`. Kemudian cluster diinisialisasi dengan perintah berikut.

```
dcli
cluster init --name=ignite3 --metastorage-group=node1,node2,node3
cluster status
```

Hasil `cluster status` menunjukkan cluster bernama `ignite3` dengan 3 node dan status `active`, serta ketiga node ikut menjadi anggota CMG dan metastorage. Opsi `--metastorage-group` diisi ketiga node supaya metadata cluster tetap aman kalau satu node mati. Inisialisasi hanya perlu dilakukan sekali. Kalau keluar dari CLI dengan Ctrl-D lalu masuk lagi dengan perintah yang sama, tidak perlu inisialisasi ulang.

<img width="1063" height="452" alt="ignite-04-cluster-init" src="https://github.com/user-attachments/assets/d4658933-0a24-4ee9-b85e-6d1e28ec9944" />

Pada kondisi ini CLI sudah terhubung ke `http://localhost:10300`, dan semua yang dikerjakan di node ini akan direplikasi ke node lain dalam satu cluster.

## 4.2.5 Eksekusi Perintah-Perintah SQL

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

Kemudian masuk ke mode SQL dan lihat tabel yang sudah terbentuk.

```
sql
SELECT * FROM system.tables WHERE schema = 'PUBLIC';
```

<img width="658" height="110" alt="ignite-07-system-tables" src="https://github.com/user-attachments/assets/750bcfc9-bc2a-4424-be78-68219ccf978f" />

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

Hal yang sama dilakukan di node3 (`http://localhost:10302`).

```
connect http://localhost:10302
```

<img width="354" height="52" alt="ignite-10-node3" src="https://github.com/user-attachments/assets/23515e8a-f725-4464-b1ca-5f672240da5c" />

Data hanya dimasukkan lewat node1, tetapi node2 dan node3 ternyata juga sudah berisi data dan memberi hasil query yang sama. Dengan demikian terlihat bahwa telah terjadi replikasi Master-Master pada cluster Apache Ignite 3.1.0.

Terakhir, semua node dimatikan dengan `dcd`.

```bash
dcd
```

<img width="1090" height="93" alt="ignite-11-dcd" src="https://github.com/user-attachments/assets/6e8e0ee9-5d76-4975-b891-5ecc1e7d898f" />
