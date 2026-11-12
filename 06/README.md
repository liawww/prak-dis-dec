# Modul 6 - Distributed File System (HDFS)

Nama : Laesah Afenlia<br>
NIM : 255410019<br>
Kelas : IF 1<br>

## 0. Persyaratan Software

- **JDK 17 dan/atau 21**. Jika akan mengkompilasi Apache Hadoop, gunakan JDK 17. JDK 17 dipakai untuk server, sedangkan client bisa memakai JDK 17 atau 21.
- **Apache Hadoop 3.5.0**
- **pdsh**
- **ssh**
- **sshd** sudah berjalan:
  - Artix Linux + dinit: `sudo dinitctl start sshd`
  - systemd: `sudo systemctl start ssh`

## 1. Unduh Apache Hadoop

Ambil distribusi Apache Hadoop di <https://hadoop.apache.org/releases.html> (pilih versi 3.5.0, binary).

<img src="img/hadoop-releases.html.png" width="700"><br>

## 2. Instalasi Apache Hadoop

<img src="img/hadoop-3.5.0.png" width="700"><br>

<img src="img/java -version.png" width="700"><br>

<img src="img/hadoop.png" width="700"><br>

Ekstrak file hasil download (asumsi berada di `~/master/data-engineering/apache-hadoop/hadoop-3.5.0.tar.gz`), lalu buat symlink:

```bash
cd ~/software/storage/
tar -xvf ~/master/data-engineering/apache-hadoop/hadoop-3.5.0.tar.gz
ln -s hadoop-3.5.0 hadoop
```

Lokasi instalasi selanjutnya disebut `$HADOOP_HOME`, yaitu `$HOME/software/storage/hadoop`.

Buat file env `~/env/bash/storage/hadoop.sh` berisi:

```bash
export PATH="$HOME/software/storage/hadoop/bin:$HOME/software/storage/hadoop/sbin:$PATH"
```

Setiap akan memakai Hadoop di sebuah shell/terminal, jalankan (cukup sekali per shell):

```bash
source ~/env/bash/storage/hadoop.sh
```

Cek instalasi:

```bash
hadoop
```

Jika berhasil, akan tampil usage Hadoop beserta daftar subcommand.

## 3. Instalasi pdsh

Paket `pdsh` diinstal menggunakan *package manager* `apt` pada lingkungan WSL (Ubuntu):

```bash
sudo apt install pdsh -y
export PDSH_RCMD_TYPE=ssh
```

<img src="img/pdsh.png" width="700"><br>

Untuk distro lain, sesuaikan dengan package manager yang digunakan.

Cek pdsh:

```bash
pdsh -q -w localhost
```

Jika `Rcmd type` bukan `ssh`, atur env variable (misal di Bash):

```bash
export PDSH_RCMD_TYPE=ssh
```

Atur agar ssh ke localhost tidak perlu login (passwordless):

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

Uji dengan `ssh localhost` (jawab `yes` saat pertama kali), lalu `logout`. Pada percobaan berikutnya tidak boleh meminta password.

<img src="img/pdsh-test.png" width="700"><br>
<img src="img/pdsh-keygen.png" width="700"><br>

## 4. Konfigurasi Apache Hadoop

Semua file berada di `$HADOOP_HOME/etc/hadoop/`.

### hdfs-site.xml
<img src="img/hdfs-site-xml.png" width="700"><br>

### core-site.xml
<img src="img/core-site-xml.png" width="700"><br>

### mapred-site.xml
<img src="img/mapred-site-xml.png" width="700"><br>

### yarn-site.xml
<img src="img/yarn-site-xml.png" width="700"><br>

## 5. Format NameNode

> **Peringatan:** proses ini dilakukan **sekali saja**. Jika diulang, berbagai file yang telah diproses di HDFS akan terhapus.

```bash
java -version
hdfs namenode -format
```
<img src="img/NameNode01.png" width="700"><br>
<img src="img/NameNode02.png" width="700"><br>

## 6. Atur JAVA_HOME

Edit `$HADOOP_HOME/etc/hadoop/hadoop-env.sh` dan isi `JAVA_HOME` sesuai lokasi JDK di komputer/server/node Anda, contoh:

```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```
<img src="img/JAVA_HOME.png" width="700"><br>

## 7. Jalankan Daemon HDFS

```bash
start-dfs.sh
```
<img src="img/jalankan_daemon_hdfs.png" width="700"><br>

Pemantauan dapat dilakukan melalui web UI NameNode: <http://localhost:9870> (pada modul, tampilan dibuka melalui `http://localhost:9870/dfshealth.html#tab-overview`).

<img src="img/web_hdfs.png" width="700"><br>

## 8. Selesai

Jika sampai tahap ini berhasil, Hadoop dan HDFS siap digunakan. Contoh operasi file pada HDFS:

```bash
   hdfs dfs -mkdir -p /user/afenlia/datasets
   hdfs dfs -put file1.csv file2.csv file3.csv /user/afenlia/datasets/
   hdfs dfs -ls /user/afenlia/datasets
```

Daftar lengkap perintah HDFS: <https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/FileSystemShell.html>

Untuk menghentikan HDFS:

```bash
stop-dfs.sh
```


## Tugas

1. Buat direktori `/user/<nama-user>/datasets` di HDFS Anda.
2. Cari 3 file `.csv` di Internet, lalu salin ketiganya ke direktori yang baru dibuat pada tugas 1.


<img src="img/tugas_hdfs_datasets.png" width="700"><br>

