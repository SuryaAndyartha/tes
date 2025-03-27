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

chmod 400 "$log_file" #Revisi
```
Penjelasan:

```bash
#!/bin/bash
```
Baris pertama _script_/program berisi shebang/hashbang yang berfungsi untuk memberi tahu sistem cara menjalankan _script_ tersebut, yaitu dengan dieksekusi langsung atau melalui _Bash_.

```bash
log_directory="/home/ubuntu/metrics"
```
Deklarasi variabel `log_directory` digunakan untuk menentukan lokasi penyimpanan _file log_, serta mempermudah pengelolaan dan perubahan _path_ dalam _script_.

```bash
mkdir -p "$log_directory"
```
Perintah ini akan membuat direktori yang ditentukan dalam `log_directory` jika belum ada, dengan opsi `-p` yang memastikan tidak terjadi eror jika direktori sudah ada.

```bash
time_format=$(date +"%Y%m%d%H%M%S")
```
Bagian ini akan mengambil waktu saat ini dalam format `YYYY/MM/DD/HH/MM/SS` menggunakan perintah `date`, lalu hasil atau _output_ dari perintah tersebut akan disimpan dalam variabel `time_format`.

```bash
log_file="$log_directory/metrics_$time_format.log"
```
Di sini, potongan _script_ tersebut akan memberikan nama _file log_ dengan menggabungkan _path_ dari `log_directory` yaitu pada kasus ini adalah `/home/ubuntu/metrics`, _string "metrics_"_, _timestamp_ dari `time_format`, dan ekstensi `.log`, sehingga setiap file log memiliki nama unik berdasarkan waktu pembuatannya. Sehingga di akhir, _file log_ yang dibuat akan memiliki format nama `path/metrics_YYYY/MM/DD/HH/MM/SS.log`.

```bash
if [ ! -f $log_file ]; then
        ...
fi
```
Perintah kondisi atau `if` ini memeriksa apakah _file_ yang disimpan dalam variabel `log_file` tidak ada, yang dituliskan dengan ekspresi `(! -f)`, sehingga perintah di dalam blok `then` hanya akan dijalankan jika _file_ tersebut belum dibuat. Dalam _Bash_, perintah `if` ditutup menggunakan `fi`.

Bagian di dalam perintah `if` akan dijelaskan sebagai berikut: 
   - ```bash
     touch $log_file
     ```
     Perintah `touch` akan membuat _file log_ yang diinginkan sesuai dengan ketentuan dan syarat yang sudah ditetapkan sebelumnya.
   - ```bash
     echo "mem_total, mem_used, mem_free, mem_shared, mem_buff, mem_available, swap_total, swap_used, swap_free, path, path_size" >$log_file
     ```
     Pada bagian di atas, `echo` akan mencetak bagian _header_ pada _file log_ yang akan dibuat, sehingga isi dari _file_ tersebut akan memiliki keterangan pada baris pertama mengenai informasi apa saja yang tertera di dalamnya. Setelah itu, _output_ dari perintah `echo` akan ditulis ke dalam _file log_ yang sudah dibuat oleh `>$log_file`.
   - ```bash
     chmod 600 "$log_file"
     ```
     Saat ini, _file log_ yang dibuat akan diberikan akses membaca (_read_) dan menulis (_write_) kepada pemilik, sehingga bisa dibuat dan dibaca dengan lancar.

```bash
mem_total=$(free -m | awk 'NR==2 {print $2}')
mem_used=$(free -m | awk 'NR==2 {print $3}')
mem_free=$(free -m | awk 'NR==2 {print $4}')
mem_shared=$(free -m | awk 'NR==2 {print $5}')
mem_buff=$(free -m | awk 'NR==2 {print $6}')
mem_available=$(free -m | awk 'NR==2 {print $7}')
swap_total=$(free -m | awk 'NR==3 {print $2}')
swap_used=$(free -m | awk 'NR==3 {print $3}')
swap_free=$(free -m | awk 'NR==3 {print $4}')
```
Bagian _script_ ini berperan dalam menjalankan perintah `free -m` untuk mendapatkan informasi yang diinginkan yaitu memori, lalu menggunakan _pipeline_ `(|)` untuk mengoper hasilnya ke `awk`, yang memproses teks secara baris per baris. Dalam `awk`, `NR==2` berarti program akan memilih baris kedua, dan dengan `{print $n}` kolom ke-n dari baris tersebut akan dicetak (misal yang dibutuhkan adalah `mem_total`, maka akan terdapat pada kolom ke-2, dst.). Hasil akhirnya disimpan dalam variabel `mem_total` menggunakan `$()`.

```bash
path="/home/ubuntu/"
```
Untuk bagian ini, khusus variabel `path` akan diisi secara manual, yaitu menunjuk kepada `/home/ubuntu`.

```bash
path_size=$(du -sh /home/ubuntu/ | awk '{print $1}')
```
Bagian _script_ ini berperan dalam menjalankan perintah `du -sh <target_path>` untuk mendapatkan informasi yang diinginkan yaitu _path size_, lalu menggunakan _pipeline_ `(|)` untuk mengoper hasilnya ke `awk`, yang memproses teks secara baris per baris. Dalam `awk`, `{print $1}` akan mencetak kolom ke-1 dari baris tersebut karena di sana letak dari informasi _path size_. Hasil akhirnya disimpan dalam variabel `path_size` menggunakan `$()`.

```bash
echo "$mem_total, $mem_used, $mem_free, $mem_shared, $mem_buff, $mem_available, $swap_total, $swap_used, $swap_free, $path, $path_size" >>$log_file
```
Sama seperti proses pencetakan _header_ pada barisan kode sebelumnya, bagian ini juga akan mencetak isi dari setiap variabel yang sudah ditentukan nilainya menggunakan perintahnya masing-masing. Setelah dicetak, _output_ akan ditambahkan (_append_) ke dalam _file log_ yang sudah dibuat dengan menggunakan `>>$log_file`, sehingga nilai dari setiap informasi yang dibutuhkan akan terletak di bawah _header_ tanpa menghapus isi _file_ yang sudah ada. 

 - Revisi
   ```bash
   chmod 400 "$log_file" #Revisi
   ```
   Sebelum melakukan demonstrasi, _script_ ini belum berhasil dalam memastikan pemilik untuk hanya mendapatkan akses membaca. Maka dari itu, _script_ sudah direvisi dengan tambahan `chmod 400` yang membuat _file log_ hanya bisa dibaca, tanpa ditulis (_write_).

   Sekarang akses ke pemilik sudah diperbarui.
   
   ![image alt](https://github.com/SuryaAndyartha/tes/blob/main/Screenshot%20from%202025-03-27%2007-02-13.png?raw=true)

### Foto Hasil Output

![image alt](https://github.com/SuryaAndyartha/tes/blob/main/Screenshot%20from%202025-03-27%2007-01-47.png?raw=true)

b. agg_5min_to_hour.sh; Code Lengkap:

```bash
#!/bin/bash

log_directory="/home/ubuntu/metrics"
mkdir -p "$log_directory"

time_format=$(date +"%Y%m%d%H")
agg_file="$log_directory/metrics_agg_$time_format.log"

previous_hour=$(date -d "1 hour ago" +"%Y%m%d%H")
log_files=$(ls $log_directory/metrics_$previous_hour*.log)

awk -F ',' '
NR == 1 { next }  
{
    if(NR == 2){
        for(i = 1; i <= NF; i++){
            min[i] = max[i] = $i
            sum[i] = 0
            count[i] = 0
        }
    }

    for(i = 1; i <= NF; i++){
        if(i == 10){
            path_value = $i  
            continue
        } 
	else if(i == 11){
            gsub(/M/, "", $i)
        }

        if($i+0 == $i){
            if($i < min[i]){ min[i] = $i }
            if($i > max[i]){ max[i] = $i }
            sum[i] += $i
            count[i]++
        }
    }
}
END {
    print "type,mem_total,mem_used,mem_free,mem_shared,mem_buff,mem_available,swap_total,swap_used,swap_free,path,path_size"

    printf "minimum"
    for(i = 1; i <= NF; i++){
        if(i == 10){
            printf ",%s", path_value
        } 
	else if(i == 11){
            printf ",%sM", min[i]
        } 
	else{
            printf ",%s", min[i]
        }
    }
    printf "\n"

    printf "maximum"
    for(i = 1; i <= NF; i++){
        if(i == 10){
            printf ",%s", path_value
        } 
	else if(i == 11){
            printf ",%sM", max[i]
        } 
	else{
            printf ",%s", max[i]
        }
    }
    printf "\n"

    printf "average"
    for(i = 1; i <= NF; i++){
        if(i == 10){
            printf ",%s", path_value
        } 
	else if(i == 11){
            printf ",%sM", (count[i] > 0 ? sum[i] / count[i] : min[i])
        } 
	else{
            printf ",%.1f", (count[i] > 0 ? sum[i] / count[i] : min[i])
        }
    }
    printf "\n"
}' $log_files > $agg_file

chmod 400 "$agg_file" #Revisi
```

```bash
#!/bin/bash
```
Baris pertama _script_/program berisi shebang/hashbang yang berfungsi untuk memberi tahu sistem cara menjalankan _script_ tersebut, yaitu dengan dieksekusi langsung atau melalui _Bash_.

```bash
log_directory="/home/ubuntu/metrics"
```
Deklarasi variabel `log_directory` digunakan untuk menentukan lokasi penyimpanan _file log_, serta mempermudah pengelolaan dan perubahan _path_ dalam _script_.

```bash
mkdir -p "$log_directory"
```
Perintah ini akan membuat direktori yang ditentukan dalam `log_directory` jika belum ada, dengan opsi `-p` yang memastikan tidak terjadi eror jika direktori sudah ada.

```bash
time_format=$(date +"%Y%m%d%H")
```
Bagian ini akan mengambil waktu saat ini dalam format `YYYY/MM/DD/HH` menggunakan perintah `date`, lalu hasil atau _output_ dari perintah tersebut akan disimpan dalam variabel `time_format`.

```bash
agg_file="$log_directory/metrics_agg_$time_format.log"
```
Di sini, potongan _script_ tersebut akan memberikan nama _file aggregation_ dengan menggabungkan _path_ dari `log_directory` yaitu pada kasus ini adalah `/home/ubuntu/metrics`, _string "metrics_agg_"_, _timestamp_ dari `time_format`, dan ekstensi `.log`, sehingga setiap file log memiliki nama unik berdasarkan waktu pembuatannya. Sehingga di akhir, _file aggregation_ yang dibuat akan memiliki format nama `path/metrics_agg_YYYY/MM/DD/HH.log`.
