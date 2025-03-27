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
Di sini, potongan _script_ tersebut akan memberikan nama _file log_ dengan menggabungkan _path_ dari `log_directory` yaitu pada kasus ini adalah `/home/ubuntu/metrics`, _string "metrics_"_, _timestamp_ dari `time_format`, dan ekstensi `.log`, sehingga setiap _file log_ memiliki nama unik berdasarkan waktu pembuatannya. Sehingga di akhir, _file log_ yang dibuat akan memiliki format nama `path/metrics_YYYY/MM/DD/HH/MM/SS.log`.

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
time_format=$(date +"%Y%m%d%H")
```
Bagian ini akan mengambil waktu saat ini dalam format `YYYY/MM/DD/HH` menggunakan perintah `date`, lalu hasil atau _output_ dari perintah tersebut akan disimpan dalam variabel `time_format`.

```bash
agg_file="$log_directory/metrics_agg_$time_format.log"
```
Di sini, potongan _script_ tersebut akan memberikan nama _file aggregation_ dengan menggabungkan _path_ dari `log_directory` yaitu pada kasus ini adalah `/home/ubuntu/metrics`, _string "metrics_agg_"_, _timestamp_ dari `time_format`, dan ekstensi `.log`, sehingga setiap _file aggregation_ memiliki nama unik berdasarkan waktu pembuatannya. Sehingga di akhir, _file aggregation_ yang dibuat akan memiliki format nama `path/metrics_agg_YYYY/MM/DD/HH.log`.

```bash
previous_hour=$(date -d "1 hour ago" +"%Y%m%d%H")
```
Bagian ini mengambil waktu satu jam yang lalu dalam format `YYYY/MM/DD/HH` menggunakan perintah `date`. `-d "1 hour ago"` memberi tahu `date` untuk menghitung waktu satu jam sebelum waktu saat ini. `+"%Y%m%d%H"` akan menetapkan format hasilnya agar hanya menampilkan tahun, bulan, hari, dan jam. `$()` menjalankan perintah dan menyimpan _output_ ke dalam variabel `previous_hour`.

```bash
log_files=$(ls $log_directory/metrics_$previous_hour*.log)
```
Di sini, program mencari dan mencocokkan semua _file log_ yang dibuat pada jam sebelumnya di dalam `log_directory` menggunakan perintah `ls`. Variabel `previous_hour` yang sudah diisi akan berguna untuk memastikan setiap _file log_ pada jam sebelumnya akan tercantum ke dalam format `YYYY/MM/DD/HH*`. Namun karena format tanggal pada _file log_ sampai ke menit dan detik (tidak hanya berhenti pada jam), maka kita membutuhkan tanda `*` yang menandakan bahwa format setelah jam (`HH`) dibebaskan atau _arbitrary_. Semua _file log_ yang cocok akan dimasukkan ke dalam variabel `log_files` menggunakan tanda `$()`.

```bash
awk -F ',' '
```
Penggunaan `awk -F ','` berfungsi untuk mengambil data dari _file log_. Bagian `-F ','` digunakan untuk menetapkan koma `(,)` sebagai pemisah kolom _(field separator)_, sehingga `awk` dapat membaca dan memproses setiap nilai dalam _file log_ sebagai kolom yang terpisah sesuai dengan yang diinginkan. Karakter `'` berfungsi sebagai pembuka `awk`.

```bash
NR == 1 { next } 
```
Bagian ini digunakan dalam `awk` untuk melewati baris pertama dalam _file log_, karena berisi _header_ yang tidak perlu diproses. `NR == 1` berarti jika baris yang sedang diproses adalah baris pertama, maka akan `next`, yaitu memberi tahu `awk` untuk langsung melanjutkan ke baris berikutnya tanpa menjalankan perintah lain.

```bash
if(NR == 2){
        ...
}
```
Untuk kondisi `if` di sini, program akan melakukan perintah yang telah ditentukan jika baris yang sedang diproses adalah baris kedua (`NR == 2`). 

Bagian di dalam perintah `if` akan dijelaskan sebagai berikut: 
   - ```bash
     for(i = 1; i <= NF; i++){
            ...
     }
     ```
     Bagian ini adalah perulangan atau `loop` menggunakan perintah `for` yang akan terus berjalan dari indeks ke-i sampai jumlah kolom yang ada (`i <= NF`). Variabel `i` akan _increment_.

     Isi dari perintah `for` akan dijelaskan sebagai berikut:

     - ```bash
       min[i] = max[i] = $i
       ```
       Setiap kolom ke-i diinisialisasi sebagai nilai minimum dan maksimum pertama kali.
     	  
     - ```bash
       sum[i] = 0
       count[i] = 0
       ```
       Variabel `sum` yang akan menjumlahkan setiap isi dari data yang ada di _file log_ diinisialisasikan sebagai 0, begitu pun dengan variabel `count` yang akan menghitung berapa kali masing-masing data muncul. Dengan kedua variabel ini, bisa didapatkan nilai rata-rata dari semua data yang ada. Diisi sesuai dengan indeks ke-i yang merepresentasikan kolom.

```bash
for(i = 1; i <= NF; i++){
        ...
    }
}
```
Bagian ini adalah perulangan atau `loop` menggunakan perintah `for` yang akan terus berjalan dari indeks ke-i sampai jumlah kolom yang ada (`i <= NF`). Variabel `i` akan _increment_.

Isi dari perintah `for` akan dijelaskan sebagai berikut:

 - ```bash
   if(i == 10){
      path_value = $i  
      continue
   } 
   ```
   Jika variabel `i` menunjukkan kolom ke-10 (kolom nama _path_), maka variabel `path_value` akan langsung diisi oleh apa yang tertulis pada kolom ke-10 tersebut, yaitu `/home/ubuntu`. Lalu program akan langsung mengabaikan pemrosesan lain akibat perintah `continue`.

 - ```bash
   else if(i == 11){
      gsub(/M/, "", $i)
   }
   ```
   Jika variabel `i` menunjukkan kolom ke-11 (kolom _path size_), maka akan terjadi pemisahan terlebih dahulu antara angka dengan karakter 'M' yang tertera pada kolom tersebut. Dengan `gsub`, program bisa mengganti karakter 'M' (`/M/`) dengan petik kosong (`""`). Dalam arti lain, karakter 'M' akan dihapus agar memudahkan perhitungan. Lalu `gsub` akan diaplikasikan ke kolom yang ditunjuk oleh `$i`, yaitu kolom ke-11.

 - ```bash
   if($i+0 == $i){
      if($i < min[i]){ min[i] = $i }
      if($i > max[i]){ max[i] = $i }
      sum[i] += $i
      count[i]++
   }
   ```
   Selain kolom ke-10 dan ke-11 yang sudah diantisipasi pada dua kondisi `if` sebelumnya, maka program akan melanjutkan serangkaian pemrosesan data untuk menentukan nilai maksimal, minimal, dan rata-rata. Pertama-tama, perintah `if` akan memastikan bahwa nilai yang terdapat pada kolom ke-i merupakan suatu angka dengan `$i+0 == $i`. Artinya adalah, jika nilai pada kolom tersebut berupa sebuah string, maka `(string)+0` tidak akan `==` nilai awal (`$i`) karena _string_ ditambah 0 akan menjadi 0. Di sisi lain, jika benar nilainya adalah angka, maka `(angka)+0` akan `==` nilai awal (`$i`). Dengan seperti ini, bisa dipastikan bahwa program akan mengoperasikan nilai yang berupa angka saja dan tidak ada _string_ yang tercampur.

    - `if($i < min[i]){ min[i] = $i }` adalah cara mencari nilai minimal yang sederhana. Jika nilai di kolom ke-i `($i)` lebih kecil dari `min[i]`, maka nilai tersebut menjadi nilai minimal yang baru.
    - `if($i > max[i]){ max[i] = $i }` adalah cara mencari nilai maksimal yang sederhana. Jika nilai di kolom ke-i `($i)` lebih besar dari `max[i]`, maka nilai tersebut menjadi nilai maksimal yang baru.
    - `sum[i] += $i` akan menambahkan variabel `sum` pada kolom ke-i `($i)` dengan nilai yang didapatkan dari _file log_ tersebut.
    - `count[i]++` akan menghitung berapa kali suatu data muncul. Dengan cara ini, bisa dihitung rata-rata dari tiap data.
  
```bash
END {
    print "type,mem_total,mem_used,mem_free,mem_shared,mem_buff,mem_available,swap_total,swap_used,swap_free,path,path_size"
```
`END` digunakan untuk menjalankan perintah setelah semua data selesai diproses. Dalam _script_ ini, akan dicetak semua hasil atau _output_ yang sudah didapatkan. Hal pertama yang akan dilakukan adalah mencetak _header_ bagi _file aggregation_ sesuai dengan format yang diinginkan. 

```bash
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
```
Pencetakan _output_ menggunakan perintah `printf`. Bagian ini akan mencetak kata "minumum" sebagai label di awal baris. Lalu ada perulangan menggunakan perintah `for` yang akan berjalan sebanyak jumlah kolom (`i <= NF`). 

Isi dari perintah `for` akan dijelaskan sebagai berikut:

 - ```bash
   if(i == 10){
      printf ",%s", path_value
   }
   ```
   Jika variabel `i` menunjukkan kolom ke-10 (kolom nama _path_), maka akan langsung mencetak isi dari variabel `path_value` sebagai _string_.

 - ```bash
   else if(i == 11){
      printf ",%sM", min[i]
   }
   ```
   Jika variabel `i` menunjukkan kolom ke-11 (kolom _path size_), maka akan mencetak nilai minimal (`min[i]`) dari _path size_ sebagai _string_, serta menambahkan karakter 'M' yang sebelumnya sudah dihilangkan/dipisah di awal. Dengan cara ini, penulisan data akan tetap sesuai format.

 - ```bash
   else{
      printf ",%s", min[i]
   }
   ```
   Jika bukan kolom ke-10 atau ke-11, maka program akan langsung mencetak nilai minimal (`min[i]`) dari tiap kolom data yang ada sebagai _string_.

 - ```bash
   printf "\n"
   ```
   Mencetak baris baru setelah selesai mencetak semua nilai.

```bash
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
```
Pencetakan _output_ menggunakan perintah `printf`. Bagian ini akan mencetak kata "maximum" sebagai label di awal baris. Lalu ada perulangan menggunakan perintah `for` yang akan berjalan sebanyak jumlah kolom (`i <= NF`). 

Isi dari perintah `for` akan dijelaskan sebagai berikut:

 - ```bash
   if(i == 10){
      printf ",%s", path_value
   }
   ```
   Jika variabel `i` menunjukkan kolom ke-10 (kolom nama _path_), maka akan langsung mencetak isi dari variabel `path_value` sebagai _string_.

 - ```bash
   else if(i == 11){
      printf ",%sM", max[i]
   }
   ```
   Jika variabel `i` menunjukkan kolom ke-11 (kolom _path size_), maka akan mencetak nilai maksimal (`max[i]`) dari _path size_ sebagai _string_, serta menambahkan karakter 'M' yang sebelumnya sudah dihilangkan/dipisah di awal. Dengan cara ini, penulisan data akan tetap sesuai format.

 - ```bash
   else{
      printf ",%s", max[i]
   }
   ```
   Jika bukan kolom ke-10 atau ke-11, maka program akan langsung mencetak nilai maksimal (`max[i]`) dari tiap kolom data yang ada sebagai _string_.

 - ```bash
   printf "\n"
   ```
   Mencetak baris baru setelah selesai mencetak semua nilai.

```bash
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
```
Pencetakan _output_ menggunakan perintah `printf`. Bagian ini akan mencetak kata "average" sebagai label di awal baris. Lalu ada perulangan menggunakan perintah `for` yang akan berjalan sebanyak jumlah kolom (`i <= NF`). 

Isi dari perintah `for` akan dijelaskan sebagai berikut:

 - ```bash
   if(i == 10){
      printf ",%s", path_value
   }
   ```
   Jika variabel `i` menunjukkan kolom ke-10 (kolom nama _path_), maka akan langsung mencetak isi dari variabel `path_value` sebagai _string_.

 - ```bash
   else if(i == 11){
      printf ",%sM", (count[i] > 0 ? sum[i] / count[i] : min[i])
   }
   ```
   Jika variabel `i` menunjukkan kolom ke-11 (kolom _path size_), maka akan dilakukan pengecekan terlebih dahulu untuk mencegah eror, yaitu pembagian dengan penyebut angka 0. Apakah data _path size_ muncul lebih dari 0 kali (`count[i] > 0 ?`)? Jika iya, maka lakukan perhitungan rata-rata, yaitu total nilai data dibagi banyaknya muncul (`sum[i] / count[i]`). Jika tidak, maka langsung cetak nilai minimalnya saja, dalam kasus ini pasti 0. Pencetakan nilai rata-rata dari _path size_ diperlakukan sebagai _string_, serta menambahkan karakter 'M' yang sebelumnya sudah dihilangkan/dipisah di awal. Dengan cara ini, penulisan data akan tetap sesuai format.


 - ```bash
   else{
      printf ",%.1f", (count[i] > 0 ? sum[i] / count[i] : min[i])
   }
   ```
   Jika bukan kolom ke-10 atau ke-11, maka akan dilakukan pengecekan terlebih dahulu untuk mencegah eror, yaitu pembagian dengan penyebut angka 0. Apakah data pada kolom ke-i muncul lebih dari 0 kali (`count[i] > 0 ?`)? Jika iya, maka lakukan perhitungan rata-rata, yaitu total nilai data dibagi banyaknya muncul (`sum[i] / count[i]`). Jika tidak, maka langsung cetak nilai minimalnya saja, dalam kasus ini pasti 0. Pencetakan nilai rata-rata dari tiap data pada kolom ke-i diperlakukan sebagai _float_ dengan ketelitian 1 angka di belakang koma.

 - ```bash
   printf "\n"
   ```
   Mencetak baris baru setelah selesai mencetak semua nilai.

```bash
}' $log_files > $agg_file
```
Tanda `}'` adalah penutup dari `END` yang merupakan bagian dari `awk`. `$log_files` `$log_files > $agg_file` berperan untuk mengalihkan (_redirect_) _output_ atau hasil pemrosesan `awk` ke _file_ agregasi.

- Revisi
   ```bash
   chmod 400 "$log_file" #Revisi
   ```
   Sebelum melakukan demonstrasi, _script_ ini belum berhasil dalam memastikan pemilik untuk hanya mendapatkan akses membaca. Maka dari itu, _script_ sudah direvisi dengan tambahan `chmod 400` yang membuat _file log_ hanya bisa dibaca, tanpa ditulis (_write_).

   Sekarang akses ke pemilik sudah diperbarui.
   
   ![image alt](https://github.com/SuryaAndyartha/tes/blob/main/Screenshot%20from%202025-03-27%2008-11-37.png?raw=true)

### Foto Hasil Output

![image alt](https://github.com/SuryaAndyartha/tes/blob/main/Screenshot%20from%202025-03-27%2008-10-46.png?raw=true)

c. uptime_monitor.sh; Code Lengkap:

```bash
#!/bin/bash

log_directory="/home/ubuntu/metrics"
mkdir -p "$log_directory"

time_format=$(date +"%Y%m%d%H")
uptime_file="$log_directory/uptime_$time_format.log"

if [ ! -f $uptime_file ]; then
        touch $uptime_file
        echo "uptime,load_avg_1min,load_avg_5min,load_avg_15min" >$uptime_file

	chmod 600 "$uptime_file"
fi

uptime=$(uptime | awk '{ print $1, $2, $3, $4}')
load_avg_1min=$(cat /proc/loadavg | awk '{ print $1 }')
load_avg_5min=$(cat /proc/loadavg | awk '{ print $2 }')
load_avg_15min=$(cat /proc/loadavg | awk '{ print $3 }')

echo "$uptime$load_avg_1min,$load_avg_5min,$load_avg_15min" >>$uptime_file

chmod 400 "$uptime_file" #Revisi
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
time_format=$(date +"%Y%m%d%H")
```
Bagian ini akan mengambil waktu saat ini dalam format `YYYY/MM/DD/HH` menggunakan perintah `date`, lalu hasil atau _output_ dari perintah tersebut akan disimpan dalam variabel `time_format`.

```bash
uptime_file="$log_directory/uptime_$time_format.log"
```
Di sini, potongan _script_ tersebut akan memberikan nama _file uptime_ dengan menggabungkan _path_ dari `log_directory` yaitu pada kasus ini adalah `/home/ubuntu/metrics`, _string "uptime_"_, _timestamp_ dari `time_format`, dan ekstensi `.log`, sehingga setiap _file uptime_ memiliki nama unik berdasarkan waktu pembuatannya. Sehingga di akhir, _file log_ yang dibuat akan memiliki format nama `path/metrics_YYYY/MM/DD/HH/MM/SS.log`.

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
