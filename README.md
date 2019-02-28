## Modul 2 Praktikum Sistem Operasi

# Proses dan Daemon

Requirements:
1. Linux
2. gcc / g++

Tutorial compiling C code: [here](https://github.com/raldokusuma/compile-c-program)

## Daftar Isi
- [Proses dan Daemon](#proses-dan-daemon)
  - [Daftar Isi](#daftar-isi)
  - [1. Proses](#1-proses)
    - [1.1 Proses](#11-proses)
    - [1.2 Proses ID](#12-proses-id)
      - [Process ID (PID)](#process-id-pid)
      - [Parent Process ID (PPID)](#parent-process-id-ppid)
      - [Parent Process](#parent-process)
      - [Child Process](#child-process)
    - [1.3 Melihat Proses](#13-melihat-proses)
    - [1.4 Membunuh Proses](#14-membunuh-proses)
    - [1.5 Membuat Proses](#15-membuat-proses)
      - [fork](#fork)
      - [exec](#exec)
      - [wait](#wait)
      - [system](#system)
    - [1.6 Jenis-Jenis Proses](#16-jenis-jenis-proses)
      - [Zombie Process](#zombie-process)
      - [Orphan Process](#orphan-process)
      - [Daemon Process](#daemon-process)
  - [2. Daemon](#2-daemon)
    - [2.1 Daemon](#21-daemon)
    - [2.2 Proses Pembuatan Daemon](#22-proses-pembuatan-daemon)
      - [1. Fork Parent Process dan penghentian Parent Process](#1-fork-parent-process-dan-penghentian-parent-process)
      - [2. Mengubah mode file menggunakan `umask(0)`](#2-mengubah-mode-file-menggunakan-umask0)
      - [3. Membuat Unique Session ID (SID)](#3-membuat-unique-session-id-sid)
      - [4. Mengubah Directory Kerja](#4-mengubah-directory-kerja)
      - [5. Menutup File Descriptor Standar](#5-menutup-file-descriptor-standar)
      - [6. Membuat Loop utama](#6-membuat-loop-utama)
  - [Appendix](#appendix)
    - [Soal Latihan](#soal-latihan)
      - [Latihan 1](#latihan-1)
      - [Latihan 2](#latihan-2)
      - [Latihan 3](#latihan-3)
    - [References](#references)


## 1. Proses

### 1.1 Proses
Proses merupakan keadaan ketika sebuah program sedang di eksekusi. Setiap proses juga memiliki PID atau Process ID yang merupakan nomor unik yang dapat digunakan untuk berinteraksi dengan proses bersangkutan.

Jalankan `$ ps` untuk menampilkan proses atau `$ ps -u` untuk lebih detailnya.

### 1.2 Proses ID
#### Process ID (PID)
PID merupakan indentitas unik berupa angka yang digunakan dalam beberapa sistem operasi untuk mengidentifikasi suatu proses. Untuk mendapatkan PID gunakan perintah `System call getpid()`
#### Parent Process ID (PPID)
PPID merupakan induk dari PID dan merupakan creator dari suatu proses. Setiap proses memiliki satu induk proses (PPID). Untuk mendapatkan PPID gunakan perintah `System call getppid()`
#### Parent Process
Parent process merupakan proses yang menciptakan beberapa child process. Proses ini tercipta dengan mengeksekusi fungsi `fork`, lalu hasil pemanggilan fungsi `fork` tersebut menciptakan beberapa child process.
#### Child Process
Child process merupakan proses yang dibuat oleh parent process. Setiap proses dapat membuat banyak child process, namun setiap proses hanya memiliki satu parent process, kecuali untuk proses yang paling pertama, proses tersebut tidak memiliki parent. Proses pertama yang dipanggil unit dalam Linux dimulai oleh kernel pada saat boot dan tidak pernah berhenti.

### 1.3 Melihat Proses

### 1.4 Membunuh Proses
Membunuh proses menggunakan `$ kill {pid}`

Contoh :
```
$ kill 3789
```
terdapat beberapa macam signal yang digunakan dalam command kill, antara lain sebagi berikut :

| Signal name | Signal value  | Effect       |
| ------------|:-------------:| -------------|
| SIGHUP      | 1             | Hangup         |
| SIGINT      | 2             | Interrupt from keyboard  |
| SIGKILL     | 9             | Kill signal   |
| SIGTERM     | 15            | Termination signal
| SIGSTOP     | 17,19,23      | Stop the process

Secara default, `$ kill` menggunakan signal SIGTERM. Untuk menggunakan signal tertentu, gunakan 
```
$ kill -{signal value} {pid}
```
. Contoh, `$ kill -9 3789` untuk menggunakan SIGKILL.

### 1.5 Membuat Proses
#### fork
#### exec
#### wait
#### system

### 1.6 Jenis-Jenis Proses
#### Zombie Process
Zombie Process terjadi karena adaanya child process yang di exit namun parrent processnya tidak tahu bahwa child process tersebut telah di terminate, misalnya disebabkan karena putusnya network. Sehingga parent process tidak merelease process yang masih digunakan oleh child process tersebut walaupun process tersebut sudah mati.
#### Orphan Process
Orphan Process adalah sebuah proses yang ada dalam komputer dimana parent process telah selesai atau berhenti bekerja namun proses anak sendiri tetap berjalan.
#### Daemon Process
Daemon Process adalah sebuah proses yang bekerja pada background karena proses ini tidak memiliki terminal pengontrol. Dalam sistem operasi Windows biasanya lebih dikenal dengan sebutan service. Daemon adalah sebuah proses yang didesain supaya proses tersebut tidak mendapatkan intervensi dari user.

## 2. Daemon
### 2.1 Daemon
Daemon adalah proses yang berjalan di balik layar (background) dan tidak berinteraksi langsung dengan user melalui standard input/output.
### 2.2 Proses Pembuatan Daemon
#### 1.  Fork Parent Process dan penghentian Parent Process 

Langkah pertama adalah men*spawn* proses menjadi induk dan anak dengan melakukan *forking*,  kemudian membunuh proses induk. Proses induk yang mati akan menyebabkan sistem operasi mengira bahwa proses telah selesai.

```c
pid_t pid, sid;
pid=fork();
if (pid < 0){
    exit(EXIT_FAILURE);
}
if (pid > 0){
    //catat PIP proses anak ke sebuah file
    exit(EXIT_SUCCESS);
}   //jika pembuatan proses berhasil, maka parent proses akan dimatikan
```

#### 2. Mengubah mode file menggunakan `umask(0)`

Untuk menulis beberapa file (termasuk logs) yang dibuat oleh daemon, mode file harus diubah untuk memastikan bahwa file tersebut dapat ditulis dan dibaca secara benar. Pengubahan mode file menggunakan implementasi umask().

#### 3. Membuat Unique Session ID (SID)

Child Process harus memiliki unik SID yang berasal dari kernel agar prosesnya dapat berjalan. Child Process menjadi Orphan Process pada system. Tipe pid_t juga digunakan untuk membuat SID baru untuk Child Process. Setsid() digunakan untuk pembuatan SID baru. Fungsi setsid() memiliki return tipe yang sama dengan fork().

```c
sid = setsid();
if(sid<0){
    exit(EXIT_FAILURE);
}  
```

#### 4. Mengubah Directory Kerja

Directori kerja yang aktif harus diubah ke suatu tempat yang telah pasti akan selalu ada. Pengubahan tempat direktori kerja dapat dilakukan dengan implementasi fungsi `chdir()`. Fungsi `chdir()` mengembalikan nilai -1 jika gagal.

```c
if((chdir("/"))<0) {
    exit(EXIT_FAILURE);
}
```

#### 5. Menutup File Descriptor Standar

Langkah terakhir dalam men-set daemon adalah menutup file desciptor standard dengan menggunakan STDIN, STDOUT, dan STDERR. Dikarenakan oleh daemon tidak menggunakan terminal, maka file desciptor dapat terus berulang dan berpotensi berbahaya bagi keamanan. Untuk mengatasi hal tersebut maka harus menggunakan fungsi close().

```c
close(STDIN_FILENO);
close(STDERR_FILENO);
clode(STDOUT_FILENO);
```

#### 6. Membuat Loop utama 

Daemon bekerja dalam jangka waktu tertentu, sehingga diperlukan sebuah looping.

```c
while(1){
    sleep(5)
}
exit(EXIT_SUCCES);
```

## Appendix
### Soal Latihan
#### Latihan 1
#### Latihan 2
#### Latihan 3
### References
https://notes.shichao.io/apue/
http://www.linuxzasve.com/preuzimanje/TLCL-09.12.pdf