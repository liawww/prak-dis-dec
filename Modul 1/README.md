# Instalasi Git

[ [Kembali](README.md) ]

Bagian ini merupakan seri tulisan tentang [Git](https://git-scm.com/). Silahkan ke [README.md](README.md) untuk memahami gambaran garis besar materi-materi yang dituliskan.

Git tersedia untuk berbagai sistem operasi. *Precompiled binaries* bisa diperoleh di [halaman dowbload Git](https://git-scm.com/downloads) untuk 3 sistem operasi utama: Linux, Mac OS X, dan Windows. Git bisa menggunakan antarmuka grafis (GUI) maupun CLI (*command line interface*). Pada materi ini, kita akan banyak menggunakan antarmuka CLI melalui shell (Linux / Mac OS X) atau *command prompt* / PowerShell di Windows. Setelah instalasi, periksa keberhasilan instalasi dengan menggunakan:

```
$ git --version
git version 2.30.1
```

Jika muncul versi (tergantung versi yang terinstall), maka kita bisa mulai menggunakan Git.

## Linux

Git untuk Linux biasanya sudah ada pada masing-masing distro dan bisa diinstall dengan *package manager* dari distro yang bersangkutan. Sebagai contoh:

1. OpenSuSE dan turunannya: sudo zypper in git
2. Arch Linux dan turunannya: sudo pacman -S git
3. Debian dan turunannya: sudo apt install git
4. RedHat dan turunannya: sudo dnf install git

Silahkan melihat pada manual dari *package manager* di distro yang bersangkutan. 

## Mac OS X

Git untuk Mac OS X juga sudah tersedia dan bisa diinstall menggunakan [Homebrew](http://brew.sh) (*package manager* di Mac OS X):

```
brew install git
```

## Windows

Sebelum install Git di Windows, anda harus sudah mempunyai editor teks yang didukung oleh Windws. Editor yang bisa dipilih banyak, tetapi disarankan menggunakan [Notepad++](https://notepad-plus-plus.org/) atau [Visual Studion Code](https://code.visualstudio.com/) atau [Vim](https://www.vim.org/). Keberadaan editor teks ini akan menentukan keberhasilan instalasi (lihat langkah 5).

1. Setelah download Git, double click pada file yang di-download. Akan dimunculkan lisensi. Klik **Next** untuk lanjut.

![01](images/01/install-01.jpg)

2. Setelah itu, pilih lokasi instalasi. Secara default akan terisi *C:\Program Files\Git*. Ganti lokasi jika memang anda menginginkan lokasi lain, klik **Next**

![02](images/01/install-02.jpg)

3. Pilih komponen. Tidak perlu diubah-ubah, sesuai dengan default saja. Klik pada **Next**.

![03](images/01/install-03.jpg)

4. Mengisi shortcut untuk menu Start. Gunakan default (Git), ganti jika ingin mengganti - misalnya Git VCS.

![04](images/01/install-04.jpg)

5. Pilih editor yang akan digunakan bersama dengan Git. Pada dasarnya anda bebas menggunakan editor teks apapun. 

![05-01](images/01/install-05-01.jpg)

Beberapa editor teks yang bisa anda gunakan adalah:

![05-02](images/01/install-05-02.jpg)

Anda juga bisa menggunakan editor pilihan anda sendiri selain di daftar dengan memilih pilihan terakhir dan kemudian mengisikan *executable file* dari editor teks yang akan digunakan.

![05-03](images/01/install-05-03.jpg)

6. Setiap melakukan inisialisasi repo Git, suatu nama branch akan diberikan. Default nama adalah **master** tetapi umumnya sekarang diganti dengan **main**. Ubahlah konfigurasi tersebut:

![06](images/01/install-06.jpg)

7. Pada saat instalasi, Git menyediakan akses git melalui Bash maupun command prompt. Pilih pilihan kedua supaya bisa menggunakan dari dua antarmuka tersebut. Bash adalah shell di Linux. Dengan menggunakan bash di Windows, pekerjaan di command line Windows bisa dilakukan menggunakan bash - termasuk ekskusi dari Git.

![07](images/01/install-07.jpg)

8. Pilih **native Windows Secure Channel library** HTTPS. Git menggunakan https untuk akes ke repo GitHub atau repo-repo lain (GitLab, Assembla).

![08](images/01/install-08.jpg)

9. Pilih pilihan pertama untuk konversi akhir baris (CR-LF).

![09](images/01/install-09.jpg)

10. Pilih MinTTY untuk terminal yang digunakan untuk mengakses Git Bash.

![10](images/01/install-10.jpg)

11. Tetapkan perilaku standar dari **git pull**. Pilih default saja yaitu **Fast-forward or merge**. Arti dari hal ini akan dipelajari pada proses pembelajaran lanjutan. 

![11](images/01/install-11.jpg)

12. Memilih **credential helper**.

![12](images/01/install-12.jpg)

13. Untuk opsi ekstra, pilih serta aktifkan *file system caching*.

![13](images/01/install-13.jpg)

14. Setelah itu proses instalasi akan dilakukan.

![014](images/01/install-14.jpg)

15. Jika selesai akan muncul dialog pemberitahuan. Klik pada **Finish**.

![015](images/01/install-15.jpg)

16. Untuk mencoba dari command prompt, masuk ke command prompt, setelah itu jalankan "git" untuk melihat apakah sudah terinstall atau belum. Jika sudah terinstall dengan benar, makan akan muncul hasil berikut:

```bash
C:\Program Files\Git\bin>git
usage: git [-v | --version] [-h | --help] [-C <path>] [-c <name>=<value>]
           [--exec-path[=<path>]] [--html-path] [--man-path] [--info-path]
           [-p | --paginate | -P | --no-pager] [--no-replace-objects] [--no-lazy-fetch]
           [--no-optional-locks] [--no-advice] [--bare] [--git-dir=<path>]
           [--work-tree=<path>] [--namespace=<name>] [--config-env=<name>=<envvar>]
           <command> [<args>]

These are common Git commands used in various situations:

start a working area (see also: git help tutorial)
   clone      Clone a repository into a new directory
   init       Create an empty Git repository or reinitialize an existing one

work on the current change (see also: git help everyday)
   add        Add file contents to the index
   mv         Move or rename a file, a directory, or a symlink
   restore    Restore working tree files
   rm         Remove files from the working tree and from the index

examine the history and state (see also: git help revisions)
   bisect     Use binary search to find the commit that introduced a bug
   diff       Show changes between commits, commit and working tree, etc
   grep       Print lines matching a pattern
   log        Show commit logs
   show       Show various types of objects
   status     Show the working tree status

grow, mark and tweak your common history
   backfill   Download missing objects in a partial clone
   branch     List, create, or delete branches
   commit     Record changes to the repository
   merge      Join two or more development histories together
   rebase     Reapply commits on top of another base tip
   reset      Set `HEAD` or the index to a known state
   switch     Switch branches
   tag        Create, list, delete or verify tags

collaborate (see also: git help workflows)
   fetch      Download objects and refs from another repository
   pull       Fetch from and integrate with another repository or a local branch
   push       Update remote refs along with associated objects

'git help -a' and 'git help -g' list available subcommands and some
concept guides. See 'git help <command>' or 'git help <concept>'
to read about a specific subcommand or concept.
See 'git help git' for an overview of the system.

C:\Program Files\Git\bin>
```

Lihat versi dari Git:

```bash
C:\Program Files\Git\bin>git --version
git version 2.53.0.windows.1

C:\Program Files\Git\bin>
```

Jika anda sudah terbiasa menggunakan **winget**, instalasi bisa dilakukan secara cepat dari Powershell / command prompt dengan perintah:

```bash
winget install --id Git.Git -e --source winget
```

Hasilnya akan sama dengan langkah-langkah di atas jika memilih kondisi default.
