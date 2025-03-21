## Task 4 - Proxy Terbaik di New Eridu _(Best Proxy in New Eridu)_

### Soal

Belle dan Wise adalah Proxy dengan nama Phaethon di Kota New Eridu yang menyamar sebagai warga biasa dengan membuka Toko Video di Sixth Street. Suatu hari, Wise meminta Belle untuk memantau setiap server yang berjalan di PC mereka karena biaya listrik bulanan yang tinggi. Karena Belle terlalu sibuk mengelola Toko Video, ia meminta bantuan kalian (Proxy yang Hebat) untuk membuat program monitoring ini.

Buatlah program untuk memantau sumber daya pada setiap server. Program ini hanya perlu memantau:

- **Penggunaan RAM** menggunakan perintah `free -m`.
- **Ukuran suatu direktori** menggunakan perintah `du -sh <target_path>`.
- **Uptime** menggunakan perintah `uptime` dan ambil bagian yang menunjukkan waktu berjalan.
- **Load average** menggunakan perintah `cat /proc/loadavg` dan ambil tiga nilai pertama (1, 5, dan 15 menit).

Catat semua metrics yang diperoleh dari hasil `free -m`. Untuk hasil `du -sh <target_path>`, catat ukuran dari path direktori tersebut. Direktori yang akan dipantau adalah `/home/{user}/`.

**Persyaratan**

- **Masukkan semua metrics** ke dalam sebuah file log bernama `metrics_{YmdHms}.log`. `{YmdHms}` adalah waktu saat script Bash dijalankan. Contoh: jika dijalankan pada `2025-03-17 19:00:00`, maka file log yang akan dibuat adalah `metrics_20250317190000.log`.

- Script untuk **mencatat metrics** di atas harus berjalan secara otomatis **setiap 5 menit**.

- Kemudian, buat satu script untuk membuat **aggregasi** file log ke satuan jam. Script agregasi akan memiliki info dari file-file yang tergenerate tiap menit. Dalam hasil file aggregasi tersebut, terdapat nilai **minimum, maximum, dan rata-rata** dari tiap-tiap metrics. File aggregasi akan ditrigger untuk dijalankan setiap jam secara otomatis. Berikut contoh nama file hasil aggregasi. Script agregasi akan dijalankan secara otomatis setiap jam. Contoh nama file hasil agregasi adalah `metrics_agg_2025031719.log` dengan format `metrics_agg_{YmdH}.log`.

- Buat script untuk memantau **uptime dan load average server** setiap jam dan menyimpannya dalam file log bernama `uptime_{YmdH}.log`. Uptime harus diambil dari output perintah uptime, sedangkan load average diambil dari `cat /proc/loadavg`.

- Terakhir, untuk menghemat storage, buatlah script untuk **menghapus** file log agregasi yang lebih lama dari **12 jam pertama** setiap hari. Script ini harus dijalankan setiap hari pada pukul 00:00.

- Karena file log bersifat sensitif, pastikan semua file log hanya dapat dibaca oleh **pemilik file**.

**Nama File Script**

| No  | Nama File Script      | Fungsi                                               |
| --- | --------------------- | ---------------------------------------------------- |
| 1   | `minute5_log.sh`      | Script pencatatan metrics setiap 5 menit             |
| 2   | `agg_5min_to_hour.sh` | Script agregasi log per jam                          |
| 3   | `uptime_monitor.sh`   | Script monitoring uptime dan load average setiap jam |
| 4   | `cleanup_log.sh`      | Script penghapusan log lama setiap hari              |

**Lokasi Penyimpanan Log**

Semua file log disimpan di `/home/{user}/metrics`.

**Format Log**

1. **Log Per 5 Menit (`metrics_{YmdHms}.log`)**

   ```
   mem_total,mem_used,mem_free,mem_shared,mem_buff,mem_available,swap_total,swap_used,swap_free,path,path_size
   15949,10067,308,588,5573,4974,2047,43,2004,/home/$USER/test/,74M
   ```

2. **Log Agregasi Per Jam (`metrics_agg_{YmdH}.log`)**

   ```
   type,mem_total,mem_used,mem_free,mem_shared,mem_buff,mem_available,swap_total,swap_used,swap_free,path,path_size
   minimum,15949,10067,223,588,5339,4626,2047,43,1995,/home/$USER/test/,50M
   maximum,15949,10387,308,622,5573,4974,2047,52,2004,/home/$USER/test/,74M
   average,15949,10227,265.5,605,5456,4800,2047,47.5,1999.5,/home/$USER/test/,62M
   ```

3. **Log Uptime Per Jam (`uptime_{YmdH}.log`)**

   ```
   uptime,load_avg_1min,load_avg_5min,load_avg_15min
   17:29:06  up  2:41,0.17,0.12,0.10
   ```

**Contoh Log yang Akan Dihapus pada 2025-03-18 Pukul 00:00**

```
metrics_agg_2025031700.log
metrics_agg_2025031701.log
metrics_agg_2025031702.log
metrics_agg_2025031703.log
metrics_agg_2025031704.log
metrics_agg_2025031705.log
metrics_agg_2025031706.log
metrics_agg_2025031707.log
metrics_agg_2025031708.log
metrics_agg_2025031709.log
metrics_agg_2025031710.log
metrics_agg_2025031711.log
metrics_agg_2025031712.log
```

**Konfigurasi Cron**

Konfigurasi cron untuk menjalankan script ini disimpan dalam file **crontabs**.

Dengan mengikuti instruksi di atas, kalian akan membantu Belle dan Wise dalam mengelola penggunaan sumber daya server mereka dan mengoptimalkan biaya listrik yang digunakan!

---

### Penyelesaian

a. minute5_log.sh; Code Lengkap:

```bash
#!/bin/bash

log_directory="/home/ubuntu/metrics"
mkdir -p "$log_directory"

time_format=$(date +"%Y%m%d%H%M%S")
log_file="$log_directory/metrics_$time_format.log"


if [ ! -f $log_file ]; then
        touch $log_file
        echo "mem_total, mem_used, mem_free, mem_shared, mem_buff, mem_available, swap_total, swap_used, swap_free, path, path_size" >$log_file

        chmod 600 "$log_file"
fi

mem_total=$(free -m | awk 'NR==2 {print $2}')
mem_used=$(free -m | awk 'NR==2 {print $3}')
mem_free=$(free -m | awk 'NR==2 {print $4}')
mem_shared=$(free -m | awk 'NR==2 {print $5}')
mem_buff=$(free -m | awk 'NR==2 {print $6}')
mem_available=$(free -m | awk 'NR==2 {print $7}')
swap_total=$(free -m | awk 'NR==3 {print $2}')
swap_used=$(free -m | awk 'NR==3 {print $3}')
swap_free=$(free -m | awk 'NR==3 {print $4}')
path="/home/ubuntu/"
path_size=$(du -sh /home/ubuntu/ | awk '{print $1}')

echo "$mem_total, $mem_used, $mem_free, $mem_shared, $mem_buff, $mem_available, $swap_total, $swap_used, $swap_free, $path, $path_size" >>$log_file
```
Penjelasan:

```bash
#!/bin/bash
```
Baris pertama _script_/program berisi shebang/hashbang yang berfungsi untuk memberi tahu sistem cara menjalankan _script_ tersebut, yaitu dengan dieksekusi langsung atau melalui _Bash_.

