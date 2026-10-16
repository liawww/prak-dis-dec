# Praktikum Minggu 03

Laporan Praktikum Minggu 03

**Topik:** Sinkronisasi pada Sistem Terdistribusi (Sinkronisasi Waktu, Vector Clock, Race Condition, Deadlock, dan Algoritma Raft)

## A. TUJUAN PRAKTIKUM

1. Memahami perlunya sinkronisasi waktu pada sistem terdistribusi dan melakukan sinkronisasi waktu ke NTP server menggunakan Chrony.
2. Memahami cara kerja *vector clock* untuk mengurutkan *event* pada sistem terdistribusi.
3. Menjalankan program *vector clock*, membandingkan keluarannya dengan perhitungan manual, dan membuatnya menjadi modul Python.
4. Memahami masalah *data race / race condition* pada program *multithreaded* beserta cara mencegahnya dengan *lock*.
5. Memahami terjadinya *deadlock* dan cara menghindarinya dengan *timeout*.
6. Memahami cara kerja algoritma Raft dalam memilih koordinator (*leader*).
7. Membuat program simulasi Raft menjadi modul Python dan membuat contoh penggunaannya.

## B. DASAR TEORI

### 1. Sinkronisasi Waktu

Pada sistem terdistribusi setiap node memiliki jam (*clock*) sendiri, dan jam-jam tersebut tidak pernah benar-benar sama karena adanya *clock drift* (laju jam yang sedikit berbeda). Perbedaan waktu antar node dapat membuat urutan kejadian menjadi kacau, misalnya pada *log*, transaksi, atau sertifikat keamanan. Karena itu node perlu menyamakan jamnya dengan sumber waktu acuan. Dua protokol yang umum dipakai adalah **NTP (Network Time Protocol)** dan **PTP (Precision Time Protocol)**; PTP lebih presisi tetapi membutuhkan dukungan perangkat keras jaringan.

Pada NTP, node berperan sebagai *client* yang meminta waktu dari *NTP server*. Server diberi tingkat **stratum**: stratum 0 adalah sumber waktu fisik (jam atom atau GPS), stratum 1 adalah server yang terhubung langsung ke stratum 0, stratum 2 mengambil waktu dari stratum 1, dan seterusnya. Semakin kecil stratum, semakin dekat ke sumber waktu yang akurat. **Chrony** adalah implementasi NTP untuk Linux yang terdiri dari *daemon* `chronyd` dan alat kendali `chronyc`.

### 2. Vector Clock

*Vector clock* adalah mekanisme untuk mengurutkan *event* pada sistem terdistribusi tanpa bergantung pada jam fisik. Setiap proses memiliki sebuah vektor berisi satu angka untuk tiap proses. Aturannya:

- Setiap ada kejadian lokal, proses menambah angkanya sendiri (*increment*).
- Saat mengirim pesan, proses menambah angkanya sendiri lalu menyertakan vektornya pada pesan.
- Saat menerima pesan, proses mengambil nilai terbesar (*max*) untuk tiap elemen dari vektornya sendiri dan vektor pada pesan, lalu menambah angkanya sendiri.
- Kejadian **A terjadi sebelum B** (*happens before*) jika semua elemen vektor A lebih kecil atau sama dengan elemen vektor B dan ada setidaknya satu elemen yang lebih kecil. Jika tidak ada yang terjadi sebelum yang lain, kedua kejadian disebut **konkuren**.

### 3. Masalah Tanpa Sinkronisasi: Race Condition dan Deadlock

**Race condition (data race)** terjadi ketika beberapa *thread* mengakses dan mengubah data yang sama secara bersamaan tanpa pengaturan, sehingga hasil akhir bergantung pada urutan eksekusi yang tidak bisa diprediksi. Bagian kode yang mengakses data bersama disebut *critical section*, dan dapat dilindungi dengan *lock* agar hanya satu *thread* yang masuk pada satu waktu.

**Deadlock** terjadi ketika dua atau lebih *thread* saling menunggu sumber daya yang dipegang oleh yang lain sehingga tidak ada yang bisa melanjutkan. Deadlock membutuhkan empat kondisi sekaligus: *mutual exclusion*, *hold and wait*, *no preemption*, dan *circular wait*. Salah satu cara menghindarinya adalah memberi batas waktu (*timeout*) saat meminta *lock*.

### 4. Algoritma Raft

Raft adalah algoritma **konsensus**, yaitu cara agar sekumpulan node sepakat pada satu keputusan walaupun ada node yang gagal. Setiap node berada pada salah satu dari tiga keadaan: **FOLLOWER**, **CANDIDATE**, atau **LEADER**. Waktu dibagi ke dalam *term* (periode kepemimpinan) yang nomornya terus bertambah. LEADER mengirim *heartbeat* secara berkala ke FOLLOWER. Jika FOLLOWER tidak menerima *heartbeat* sampai *election timeout* (nilainya acak), ia menaikkan *term*, menjadi CANDIDATE, dan meminta suara dari node lain. CANDIDATE yang mendapat suara mayoritas menjadi LEADER yang baru.

## C. PEMBAHASAN LISTING & TAHAPAN PRAKTIK

> Seluruh praktik dikerjakan pada Ubuntu (WSL) dengan Python 3.14 dan editor `nano`.

### 3.1 SINKRONISASI WAKTU

Modul menyediakan dua pilihan sesuai sistem operasi. Karena praktik ini dilakukan di Linux (Ubuntu WSL), saya memakai pilihan pertama, yaitu **Chrony**.

#### 1. Menjalankan Chrony untuk sinkronisasi ke NTP server

Chrony (`chronyd` versi 4.8) sudah tersedia pada Ubuntu yang dipakai; jika belum terpasang, caranya `sudo apt install chrony`. Sinkronisasi ke NTP server dicoba dengan perintah berikut, lalu statusnya diperiksa dengan `chronyc tracking`:

```
sudo chronyd -q 'server pool.ntp.org iburst'
chronyc tracking
```

<img width="1174" height="364" alt="3 1 sinkronisasi waktu ke suatu NTP server" src="https://github.com/user-attachments/assets/40bc0e8c-8399-40ce-8d54-74e1301f63d6" />


**Analisis Output:**

- Perintah `chronyd -q 'server pool.ntp.org iburst'` menampilkan pesan `Fatal error: Another chronyd may already be running (pid=308)`. Artinya `chronyd` sudah berjalan sebagai *service* di latar belakang sehingga tidak bisa dijalankan dua kali. Ini bukan kegagalan sinkronisasi, karena `chronyd` yang sudah berjalan itulah yang mengurus sinkronisasi waktu secara terus-menerus.
- `chronyc tracking` menampilkan status sinkronisasi: sumber waktu acuan (*Reference ID*) adalah `ntp-nts-2.ps6.canonical.com` dengan **Stratum 3**, artinya server tersebut tiga tingkat dari sumber waktu fisik.
- **System time: 2,514 detik slow of NTP time** berarti jam di komputer ini tertinggal sekitar 2,5 detik dibanding waktu NTP.
- **Last offset: -0,0207 detik** dan **RMS offset: 0,0096 detik** adalah selisih hasil pengukuran terakhir dan rata-ratanya; nilainya sangat kecil (puluhan milidetik).
- **Frequency: 6,966 ppm slow** artinya jam perangkat berjalan sedikit lebih lambat dari seharusnya (sekitar 7 mikrodetik per detik), dan Chrony mencatatnya untuk mengoreksi.
- **Root delay: 0,33 detik** adalah total waktu tempuh bolak-balik ke sumber waktu acuan; **Update interval: 194,4 detik** adalah jarak waktu antar pembaruan; **Leap status: Normal** artinya tidak ada penyesuaian *leap second*.

#### 2. Penjelasan proses sinkronisasi

1. `chronyd` (sebagai *client*) mengirim permintaan waktu ke NTP server dan mencatat waktu kirimnya (T1).
2. Server mencatat waktu menerima permintaan (T2) dan waktu membalas (T3), lalu mengirimkannya kembali.
3. *Client* mencatat waktu menerima balasan (T4).
4. Dari empat waktu itu dihitung **delay** perjalanan jaringan, yaitu (T4 − T1) − (T3 − T2), dan **offset** (selisih jam *client* dengan server), yaitu ((T2 − T1) + (T3 − T4)) / 2.
5. Pengukuran dilakukan berulang kali dan dari beberapa server (opsi `iburst` mempercepat pengukuran awal), lalu Chrony memilih sumber terbaik dan menyaring data yang buruk.
6. Berdasarkan offset, Chrony mengoreksi jam. Selisih kecil dikoreksi perlahan dengan mempercepat atau memperlambat jam (*slew*) agar waktu tidak melompat mundur, sedangkan selisih besar dapat langsung diloncatkan (*step*). Chrony juga menghitung *frequency error* (nilai ppm di atas) agar jam tetap akurat di antara dua kali sinkronisasi.

*(Catatan: pada WSL jam Linux umumnya disinkronkan dari Windows (host), sehingga `chronyc tracking` di sini terutama berguna untuk memperlihatkan hasil pengukuran jam terhadap NTP server.)*

---

### 3.2 VECTOR CLOCK

#### A. Menjalankan program vclocks.py

Kode `vclocks.py` (diketik pada `nano`) berisi *class* `VectorClock` dan simulasi tiga proses (P0, P1, P2):

<img width="1362" height="689" alt="3 2 A  vclocks" src="https://github.com/user-attachments/assets/5daeaacc-ad78-4338-b570-cb5172c61102" />

<img width="1366" height="491" alt="3 2 A  lanjutan vclocks" src="https://github.com/user-attachments/assets/75ea25b0-aa45-4ab9-a25f-c8034d02f0be" />


Program dijalankan dengan `python3 vclocks.py` dan keluarannya:

<img width="683" height="349" alt="3 2 A  output vclocks" src="https://github.com/user-attachments/assets/8343bf5d-32f8-4316-a98c-b9f77bcb86ff" />


**Analisis Output dan perbandingan dengan perhitungan manual:**

Perhitungan manual dilakukan dengan aturan *vector clock* dari Dasar Teori, dimulai dari P0 = [0,0,0], P1 = [0,0,0], P2 = [0,0,0]:

| Langkah | Kejadian | Perhitungan manual | Hasil manual | Keluaran program |
|---|---|---|---|---|
| 0 | Kondisi awal | semua nol | P0 [0,0,0], P1 [0,0,0], P2 [0,0,0] | sama |
| 1 | P0 *increment* | angka P0 + 1 | P0 [1,0,0] | `P0: [1, 0, 0]` |
| 2 | P0 kirim pesan ke P1 | P0 naik dulu menjadi [2,0,0], pesan membawa [2,0,0] | P0 [2,0,0] | `P0: [2, 0, 0]` |
| 3 | P1 terima pesan | max([0,0,0],[2,0,0]) = [2,0,0], lalu angka P1 + 1 | P1 [2,1,0] | `P1: [2, 1, 0]` |
| 4 | P2 *increment* | angka P2 + 1 | P2 [0,0,1] | `P2: [0, 0, 1]` |
| 5 | P1 kirim pesan ke P2 | P1 naik menjadi [2,2,0], pesan membawa [2,2,0] | P1 [2,2,0] | `P1: [2, 2, 0]` |
| 6 | P2 terima pesan | max([0,0,1],[2,2,0]) = [2,2,1], lalu angka P2 + 1 | P2 [2,2,2] | `P2: [2, 2, 2]` |

Hasil perhitungan manual **sama persis** dengan keluaran program. Pemeriksaan hubungan sebab-akibat (*causality*) pada akhir program:

- `vc0 happens before vc1: True`. vc0 = [2,0,0] dan vc1 = [2,2,0]: semua elemen vc0 ≤ vc1 dan elemen kedua lebih kecil (0 < 2). Ini wajar, karena P1 menerima pesan dari P0.
- `vc1 happens before vc0: False`. Elemen kedua vc1 (2) lebih besar dari vc0 (0), jadi vc1 tidak mungkin terjadi sebelum vc0.
- `vc0 is concurrent with vc2: False`. vc0 = [2,0,0] dan vc2 = [2,2,2]: semua elemen vc0 ≤ vc2 dan ada yang lebih kecil, sehingga vc0 *happens before* vc2, **bukan konkuren**. Walaupun P0 tidak mengirim pesan langsung ke P2, informasinya sampai lewat P1 (P0 → P1 → P2). Sebaliknya, seandainya P2 belum menerima pesan dari P1 (nilainya [0,0,1]), P0 dan P2 akan konkuren karena tidak ada yang lebih kecil atau sama dengan yang lain pada semua elemen.

#### B. Membuat modul Python untuk VectorClock

Class `VectorClock` dipindahkan ke berkas tersendiri bernama `vector_clock_module.py` (tanpa kode simulasi), sehingga dapat di-*import* dari program lain:

<img width="1366" height="729" alt="3 2 (B) vectorClockModule" src="https://github.com/user-attachments/assets/ab7f78a2-9e44-45c4-aed1-83abb9ff7b14" />


Kemudian dibuat program contoh penggunaannya pada berkas `demo_modul.py`, yang memakai `from vector_clock_module import VectorClock`:

<img width="684" height="466" alt="3 2 (B) 02 vectorClockModule" src="https://github.com/user-attachments/assets/7c291349-0360-4e5f-9715-c029a88d1850" />

Dijalankan dengan `python3 demo_modul.py`:

<img width="679" height="341" alt="3 2 (B) OUTPUT" src="https://github.com/user-attachments/assets/5342b34e-8118-4712-946c-34729922e5d1" />


**Analisis Output:**

- Modul berhasil dipakai: program `demo_modul.py` cukup menulis `from vector_clock_module import VectorClock` tanpa menyalin isi *class*.
- Setelah P0 melakukan *increment* lokal, P0 = [1,0,0]. Setelah P0 mengirim pesan ke P1, P0 menjadi [2,0,0] dan P1 menjadi [2,1,0], sama dengan perhitungan pada bagian A.
- `P0 happens before P1? True` karena [2,0,0] ≤ [2,1,0] dengan satu elemen lebih kecil.
- `P0 concurrent dengan P2? False` karena P2 masih [0,0,0], sehingga P2 *happens before* P0 (semua elemen P2 ≤ P0). Hasil ini berbeda dari kondisi konkuren yang sebenarnya; untuk menunjukkan kondisi konkuren, P2 perlu melakukan kejadian lokal terlebih dahulu tanpa menerima pesan dari P0 atau P1.

---

### 3.3 PROBLEM TANPA SINKRONISASI

#### 0. Program multithreaded (multithreads-example.py)

Program membuat lima *thread* (A sampai E) yang masing-masing mensimulasikan pekerjaan selama 5 detik, lalu dijalankan bersamaan. Program dijalankan berulang kali dengan `python3 multithreads-example.py` sampai keluaran yang berbeda didapat.
```
![Run 1](multithread-run-1.png)
![Run 2](multithread-run-2.png)
```

<img width="607" height="257" alt="multithread-run-1" src="https://github.com/user-attachments/assets/62ae6b37-9700-4b4a-a2f4-64b358fbe754" />

<img width="530" height="207" alt="multithread-run-2" src="https://github.com/user-attachments/assets/b0f73adb-9c0b-4d54-92d9-eff9d826d456" />


**Analisis Output:**

- Pada eksekusi pertama dan kedua, urutan pesan `Task X starting...` maupun `Task X finished after 5s.` bisa berbeda (misalnya pertama A, B, C, D, E tetapi kedua A, C, B, E, D).
- Semua *thread* memiliki pekerjaan yang sama (5 detik), tetapi *thread* tidak dijalankan secara bergantian dengan urutan yang pasti. Sistem operasi dan *scheduler* yang menentukan kapan setiap *thread* mendapat giliran CPU, dan *thread* yang selesai lebih dahulu tidak bisa diprediksi.
- Karena itu pola pikir sekuensial ("A selesai, lalu B, lalu C") tidak bisa dipakai pada program *multithreaded*. Hanya `t1.join()` sampai `t5.join()` yang menjamin pesan `All tasks completed.` selalu tampil terakhir.

#### 3.3.1 Data Race / Race Condition

##### a. race-conditions-01.py (dengan *data race*)

Dua *thread* masing-masing mengambil Rp 80 dari saldo awal Rp 100.

<img width="1366" height="732" alt="3 3 1 race-conditions-01 " src="https://github.com/user-attachments/assets/17b03c54-1d7f-4c3b-a139-e30cd16b47fd" />

<img width="481" height="110" alt="3 3 2 race-conditions-01 " src="https://github.com/user-attachments/assets/765add39-a483-45f2-9f23-6753852cc45c" />


**Analisis Output:**

- Keluarannya `Saldo: Rp 20` kemudian `Saldo: Rp -60`. Hasil ini tidak masuk akal, karena saldo tidak boleh negatif; seharusnya hanya satu pengambilan yang berhasil karena saldo Rp 100 tidak cukup untuk dua kali pengambilan Rp 80.
- Penyebabnya: kedua *thread* mengecek `if balance >= amount` hampir bersamaan. Pada saat dicek, saldo masih 100 sehingga keduanya lolos. Di antara pengecekan dan pengurangan ada `time.sleep(0.01)` sehingga *thread* lain sempat masuk sebelum saldo dikurangi. Akibatnya keduanya mengurangi saldo (100 − 80 − 80 = −60). Inilah *race condition*: hasil bergantung pada urutan eksekusi *thread* yang tidak terkendali.

**Visualisasi (digambar manual):**
```
![Visualisasi Race Condition](gambar-race-condition.png)
```
<img width="817" height="534" alt="gambar-race-condition" src="https://github.com/user-attachments/assets/64f38565-7ad8-4cf9-ae18-014e486bf0d9" />


##### b. race-conditions-02.py (dengan *lock*)

<img width="1366" height="731" alt="3 3 1 race-conditions-02 " src="https://github.com/user-attachments/assets/268ad80c-9f44-4102-99b4-d5950448203a" />

<img width="484" height="97" alt="3 3 2 race-conditions-02 " src="https://github.com/user-attachments/assets/00d25766-b057-42d0-98e6-2431bb7a676d" />


**Analisis Output:**

- Hanya satu pengambilan yang tampil (`Saldo: Rp 20`) dan saldo tidak pernah negatif.
- Program ini menambahkan `counter_lock = threading.Lock()` dan membungkus seluruh pengecekan dan pengurangan saldo dengan `with counter_lock:`. Lock membuat bagian tersebut (*critical section*) hanya bisa dimasuki satu *thread* pada satu waktu.
- *Thread* pertama yang mendapat *lock* mengecek saldo (100 ≥ 80), mengurangi menjadi 20, lalu melepas *lock*. *Thread* kedua baru boleh masuk setelah itu; saat ia mengecek, saldo sudah 20, sehingga `balance >= amount` bernilai salah dan tidak ada pengurangan (tidak ada pesan yang dicetak). Dengan begitu *race condition* tidak terjadi.

#### 3.3.2 Deadlock

##### a. deadlock-01.py (terjadi *deadlock*)

<img width="1366" height="728" alt="3 3 1 deadlock-01" src="https://github.com/user-attachments/assets/4edb1349-6666-4101-82f5-c14840074f34" />

<img width="454" height="97" alt="3 3 2 deadlock-01" src="https://github.com/user-attachments/assets/6a0b9c00-fe54-46df-870e-f642dfe17f1e" />

Program berhenti (*hang*) dan tidak pernah selesai, sehingga harus dihentikan paksa dengan `Ctrl + C`:

<img width="534" height="138" alt="3 3 3 deadlock-01" src="https://github.com/user-attachments/assets/7da355a9-0c5f-4192-9f4d-70f45c0e39ac" />


**Analisis Output:**

- Hanya dua baris tercetak: `Thread 1: Locked A, waiting for B...` dan `Thread 2: Locked B, waiting for A...`. Pesan `Locked both` tidak pernah muncul.
- *Thread* 1 memegang `lock_a` lalu menunggu `lock_b`, sedangkan *thread* 2 memegang `lock_b` lalu menunggu `lock_a`. Keduanya saling menunggu (*circular wait*) dan tidak ada yang mau melepaskan *lock* yang sudah dipegang (*hold and wait*), sehingga tidak ada yang bisa maju. Inilah *deadlock*.
- Pada `t1.join()` program menunggu selamanya, sehingga setelah ditekan `Ctrl + C` muncul `KeyboardInterrupt`. Pemanggilan `time.sleep(1)` di tengah program sengaja dipasang untuk memperbesar peluang *deadlock* (`# Increase chance of deadlock`), karena memberi waktu bagi kedua *thread* untuk masing-masing mengambil *lock* pertama sebelum mencoba *lock* kedua.

**Visualisasi (digambar manual):**
```
![Visualisasi Deadlock](gambar-deadlock.png)
```
<img width="790" height="515" alt="gambar-deadlock" src="https://github.com/user-attachments/assets/93b72132-2975-4bba-bcce-a5d52918d021" />


##### b. deadlock-02.py (menghindari *deadlock* dengan *timeout*)

<img width="1366" height="728" alt="3 3 1 deadlock-02" src="https://github.com/user-attachments/assets/03d9f18a-cf7a-44c1-9c5b-3fb76256f820" />

<img width="442" height="176" alt="3 3 2 deadlock-02" src="https://github.com/user-attachments/assets/1b84169c-f8a2-40bb-832a-beb38a4c97f8" />

**Analisis Output:**

- Program kali ini selesai dan mencetak `Execution Finished`, tanpa berhenti di tengah jalan.
- Setiap *lock* diminta dengan `acquire(timeout=5)`, artinya *thread* hanya mau menunggu paling lama 5 detik. Pada awalnya kondisinya sama dengan sebelumnya: *thread* 1 memegang Lock A dan *thread* 2 memegang Lock B, lalu keduanya saling menunggu.
- Setelah 5 detik *thread* 1 menyerah menunggu Lock B, keluar dari blok `try`, dan `finally` melepaskan Lock A. Dengan begitu *thread* 2 akhirnya bisa mengambil Lock A (`Thread 2: Acquired Lock A`) dan menyelesaikan pekerjaannya.
- Dengan *timeout* pola saling menunggu selamanya (*circular wait*) terputus. Konsekuensinya, salah satu *thread* (di sini *thread* 1) gagal mendapat *lock* kedua dan pekerjaannya tidak selesai (tidak ada pesan `Thread 1: Acquired Lock B`), sehingga dalam program nyata perlu ada langkah mencoba ulang (*retry*).

---

### 3.4 ALGORITMA RAFT

Program `raft.py` mensimulasikan tiga node (Node 1, 2, dan 3) yang memilih LEADER. Simulasi berjalan 20 langkah, dengan jeda 0,5 detik tiap langkah.

<img width="1365" height="730" alt="3 4 Algoritma Raft" src="https://github.com/user-attachments/assets/c2b232b9-7fcd-4dde-9e38-feb8bb6d3644" />

Program dijalankan dengan `python3 raft.py`. Keluarannya (langkah 1-6, 7-14, dan 15-20):

<img width="451" height="640" alt="3 4 output 1 Algoritma Raft" src="https://github.com/user-attachments/assets/0ccca56f-35b2-48d6-ae29-73f5f6a8c429" />

<img width="422" height="643" alt="3 4 output 2 Algoritma Raft" src="https://github.com/user-attachments/assets/77d1ab2d-20a5-41fa-9a00-8cc0c0fa140c" />

<img width="356" height="507" alt="3 4 output 3 Algoritma Raft" src="https://github.com/user-attachments/assets/daa2fd61-2735-41a0-9a57-d108306eec7d" />

**Analisis Output (cara Raft memilih LEADER):**

1. **Langkah 1-2:** Ketiga node berstatus FOLLOWER dengan *term* 0 dan belum ada LEADER. Setiap node punya *election timeout* acak antara 1 sampai 2 detik.
2. **Langkah 3:** *Election timeout* habis dan tidak ada *heartbeat* dari LEADER, sehingga node memulai pemilihan: menaikkan *term* menjadi 1, berstatus CANDIDATE, memilih dirinya sendiri, lalu meminta suara node lain. Di program ini permintaan suara disederhanakan, yaitu semua node lain dianggap memberi suara. Karena itu suara selalu mayoritas (3 dari 3, lebih dari 3 // 2 = 1) dan node langsung menjadi LEADER. Pada keluaran ini ketiga node habis *timeout* pada langkah yang sama sehingga muncul tiga pesan `elected leader in term 1`.
3. **Langkah 4:** Pada langkah berikutnya, program memilih satu LEADER secara acak (dalam keluaran ini **Node 2**) yang mengirim *heartbeat* berisi *term* 1 ke node lain. Node yang menerima *heartbeat* dengan *term* yang sama atau lebih tinggi kembali menjadi FOLLOWER dan menunda *election timeout*-nya. Akibatnya Node 1 dan Node 3 menjadi FOLLOWER, dan hanya **Node 2 yang menjadi LEADER** untuk seterusnya.
4. **Langkah 5-20:** Node 2 tetap menjadi LEADER karena terus mengirim *heartbeat*, sehingga FOLLOWER tidak pernah memulai pemilihan baru (*term* tetap 1). Setiap 5 langkah LEADER menambahkan perintah ke log-nya (`command-5`, `command-10`, `command-15`), sehingga `Log len` Node 2 naik dari 0 menjadi 3.
5. **Catatan simplifikasi:** pada Raft yang sebenarnya, satu *term* hanya boleh punya satu LEADER dan CANDIDATE harus benar-benar mendapat suara mayoritas dari node lain (node hanya memberi satu suara per *term*). Itu sebabnya log FOLLOWER pada simulasi ini tetap 0: replikasi log ke FOLLOWER tidak disimulasikan (`# In real system, replicate to followers`). Simulasi ini hanya menunjukkan prinsip dasarnya: *election timeout* acak, *term*, suara mayoritas, dan *heartbeat* yang menjaga satu LEADER.

**Visualisasi (digambar manual):**
```
![Visualisasi Algoritma Raft](gambar-raft.png)
```
<img width="947" height="455" alt="gambar-raft" src="https://github.com/user-attachments/assets/04f81797-0ec6-4d4f-b12e-a806dc8843e9" />


#### Membuat modul Python untuk Raft dan contoh simulasinya

Program `raft.py` dipisah menjadi dua berkas:

- [`raft_module.py`](raft_module.py): berisi `class State` dan `class RaftNode` (tanpa kode simulasi).
- [`demo_raft.py`](demo_raft.py): program contoh yang memakai `from raft_module import RaftNode, State`, membuat 3 node, dan menjalankan simulasi 20 langkah.

Dijalankan dengan:

```
python3 demo_raft.py
```
```
![Output demo_raft.py](raft-modul-output.png)
```
<img width="541" height="232" alt="raft-modul-output" src="https://github.com/user-attachments/assets/d509248f-b1e5-429d-b8f1-4efaad526ab0" />


**Analisis Output:**

- Hasilnya berupa pemilihan LEADER yang sama dengan `raft.py`, tetapi kode simulasi kini terpisah dari definisi *class*, sehingga `RaftNode` dapat dipakai ulang oleh program lain.
- Karena *election timeout* dibuat acak, node yang menjadi LEADER dan *step* terjadinya pemilihan bisa berbeda tiap dijalankan, tetapi pola akhirnya sama: satu LEADER dan sisanya FOLLOWER dengan *term* yang sama.

## D. KESIMPULAN

1. **Sinkronisasi waktu:** Chrony mengukur selisih jam dengan NTP server (`chronyc tracking`: *offset* sekitar 0,02 detik dan stratum 3). Sinkronisasi dilakukan dengan menghitung *offset* dan *delay* dari empat cap waktu, lalu mengoreksi jam secara bertahap.
2. **Vector clock:** hasil program sama dengan perhitungan manual, dan dapat menentukan apakah dua kejadian berurutan (*happens before*) atau konkuren tanpa memakai jam fisik. Class `VectorClock` berhasil dibuat menjadi modul yang dapat dipakai ulang.
3. **Race condition:** tanpa sinkronisasi, dua *thread* dapat membuat saldo menjadi negatif (−60). Dengan `Lock`, bagian kritis dilindungi sehingga hasilnya benar (Rp 20).
4. **Deadlock:** dua *thread* yang saling menunggu *lock* akan berhenti selamanya. Dengan `acquire(timeout=5)`, *thread* menyerah setelah batas waktu dan *deadlock* terhindar, dengan konsekuensi salah satu pekerjaan harus diulang.
5. **Raft:** LEADER dipilih lewat *election timeout* acak, kenaikan *term*, dan suara mayoritas, lalu dipertahankan dengan *heartbeat*. Program simulasi berhasil dijadikan modul (`raft_module.py`) beserta contoh penggunaannya (`demo_raft.py`).
