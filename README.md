**Tutorial 11 Pemrograman Lanjut (Advanced Programming) 2024/2025 Genap**
* Nama    : Muhammad Almerazka Yocendra
* NPM     : 2306241745
* Kelas   : Pemrograman Lanjut - A

## Hello Minikube
### 1. Compare the application logs before and after you exposed it as a Service.
- Try to open the app several times while the proxy into the Service is running.
- What do you see in the logs? Does the number of logs increase each time you open the app?

### Before expose
![1](https://github.com/user-attachments/assets/a92c351d-59a0-4081-a9b7-974e3a2d2b8c)

### After expose
![2](https://github.com/user-attachments/assets/ceb92c50-28da-4b5f-bc4b-7b831c839285)
![3](https://github.com/user-attachments/assets/758567fa-a01e-4e11-a10c-e78d50f3efef)

Berdasarkan gambar yang ditampilkan, terdapat perbedaan yang signifikan pada aplikasi **logs** sebelum dan sesudah aplikasi di-_expose_ sebagai **Service**. 
- Sebelum di-_expose_, log aplikasi hanya menampilkan pesan _startup_ dasar yaitu `"Started HTTP server on port 8080"` dan `"Started UDP server on port 8081"` pada _timestamp_ `08:43:12`, tanpa ada aktivitas _request_ tambahan karena aplikasi belum dapat diakses dari luar _pod_.
- Setelah aplikasi di-_expose_ sebagai **Service** dan diakses melalui _browser_, log aplikasi mengalami peningkatan aktivitas yang terlihat dari munculnya entri log baru berupa HTTP **GET** requests ke path root ("/"). Pada gambar kedua terlihat **GET** _requests_ pada _timestamp_ `08:45:19`, sedangkan pada gambar ketiga terlihat **GET** _requests_ pada _timestamp_ `09:05:11`, menunjukkan bahwa setiap kali aplikasi diakses pada waktu yang berbeda, log baru akan tercatat.

Jumlah log meningkat setiap kali aplikasi dibuka karena **Service** berfungsi sebagai _gateway_ yang meneruskan _request_ eksternal ke pod aplikasi. Setiap kali _browser_ mengakses aplikasi melalui _minikube_ service `hello-node`, browser mengirimkan HTTP **GET** _request_ yang kemudian diterima dan diproses oleh _pod_, sehingga tercatat dalam _application logs_. Hal ini membuktikan bahwa _expose_ aplikasi sebagai **Service** tidak hanya memungkinkan akses eksternal yang sebelumnya tidak tersedia, tetapi juga memungkinkan kita untuk memantau aktivitas aplikasi melalui log yang mencatat setiap _request_ yang masuk. Perbedaan _timestamp_ pada setiap akses menunjukkan bahwa log aplikasi secara _real-time_ mencatat aktivitas pengguna ketika mengakses aplikasi melalui **Service**.

### 2. Notice that there are two versions of kubectl get invocation during this tutorial section. 
- The first does not have any option, while the latter has -n option with value set to kube-system.
- What is the purpose of the -n option and why did the output not list the pods/services that you explicitly created

Opsi -n dalam perintah `kubectl` get digunakan untuk memilih _namespace_ tertentu yang ingin kita lihat. Bayangkan _namespace_ seperti folder atau ruangan yang berbeda dalam sebuah gedung - setiap ruangan memiliki isi yang berbeda-beda. Ketika kita menjalankan kubectl _get_ tanpa tambahan -n, kita hanya melihat isi dari ruangan `"default"` dimana semua aplikasi yang kita buat biasanya disimpan. Di sinilah _pod_ dan _service_ `hello-node` yang kita buat tadi berada. Namun ketika kita menambahkan `-n kube-system`, kita sedang mengintip ke ruangan khusus bernama `"kube-system"` yang berisi peralatan dan sistem penting untuk menjalankan **Kubernetes** itu sendiri

Alasan mengapa output dari `kubectl get -n kube-system` tidak menampilkan _pod_ atau _service_?
Jawabannya sederhana karena kita sedang melihat ruangan yang salah, _resource_ yang kita buat berada di _namespace_ `"default"`, bukan di _namespace_ `"kube-system"`. _Namespace_ `"kube-system"` merupakan _namespace_ khusus yang digunakan oleh **Kubernetes** untuk menyimpan komponen sistem internal, sedangkan _resource_ yang dibuat pengguna secara normal akan ditempatkan di _namespace_ `"default"` kecuali secara eksplisit ditentukan _namespace_ lain. Penggunaan opsi `-n` ini sangat berguna dalam lingkungan _production_ dimana terdapat banyak _namespace_ yang berbeda untuk memisahkan aplikasi, _environment_, atau tim yang berbeda, sehingga _administrator_ dapat mengelola dan memonitor _resource_ secara lebih spesifik sesuai kebutuhan operasional.

## Rolling Update & Kubernetes Manifest File
### 1. What is the difference between Rolling Update and Recreate deployment strategy? 
Perbedaan utama antara strategi _deployment_ **Rolling Update** dan **Recreate** terletak pada cara mereka menangani proses pembaruan aplikasi dan dampaknya terhadap ketersediaan layanan. 

- Strategi **Rolling Update** melakukan pembaruan secara bertahap dan terkontrol, dimana _pod_ baru dengan versi terbaru dibuat satu per satu sementara _pod_ lama masih tetap berjalan dan melayani _traffic_. Proses ini memastikan bahwa selalu ada sejumlah _pod_ yang tersedia untuk menangani permintaan pengguna, sehingga aplikasi tetap dapat diakses tanpa mengalami _downtime_ selama proses _update_ berlangsung.
- Sebaliknya, strategi **Recreate** menggunakan pendekatan yang lebih sederhana namun berisiko, dimana seluruh _pod_ dari versi lama dihentikan terlebih dahulu sebelum _pod_ baru dengan versi terbaru dibuat dan dijalankan.

> Konsekuensi dari perbedaan pendekatan ini adalah **Rolling Update** memberikan pengalaman yang lebih baik bagi pengguna karena tidak ada gangguan layanan, membuatnya ideal untuk aplikasi _production_ yang membutuhkan _high availability_. Namun, strategi ini memerlukan lebih banyak _resource_ sementara karena _pod_ lama dan baru berjalan bersamaan selama transisi. Di sisi lain, **Recreate** menyebabkan _downtime_ yang dapat mengganggu pengguna tetapi lebih efisien dalam penggunaan _resource_ dan cocok untuk skenario dimana versi lama dan baru tidak dapat berjalan bersamaan karena konflik _database_, _shared storage_, atau _dependency_ tertentu. Pilihan strategi _deployment_ ini bergantung pada kebutuhan aplikasi, toleransi terhadap _downtime_, dan ketersediaan _resource_ dalam _cluster_.

### 2. Try _deploying_ the **Spring Petclinic** **REST** using **Recreate** _deployment strategy_ and document your _attempt_
![image](https://github.com/user-attachments/assets/571d3355-3215-432e-88df-bdc70c4fda11)

Untuk mencoba _strategi deployment_ **Recreate** pada **Spring Petclinic REST**, pertama saya perlu memodifikasi file _deployment_ yang sudah ada atau membuat file _deployment_ baru dengan strategi **Recreate**. Saya membuat file bernama `deployment-recreate.yaml` dan mengubah bagian `spec.strategy` dari **RollingUpdate** menjadi **Recreate**. Dalam file ini, saya menghapus konfigurasi _rollingUpdate_ karena parameter tersebut tidak diperlukan untuk strategi **Recreate**. Setelah menyiapkan file _manifest_, saya menjalankan perintah `kubectl apply -f deployment-recreate.yaml` untuk menerapkan _deployment_ dengan strategi **Recreate**. Selama proses ini, saya mengamati bahwa semua _pod_ yang sedang berjalan dihentikan terlebih dahulu sebelum _pod_ baru dibuat, berbeda dengan **Rolling Update** yang melakukan transisi bertahap. Meskipun proses _deployment_ berhasil, aplikasi mengalami _downtime_ singkat selama transisi dari _pod_ lama ke _pod_ baru

### 3. Try _deploying_ the **Spring Petclinic** **REST** using **Recreate** _deployment strategy_ and document your _attempt_
Untuk menerapkan strategi **Recreate** _deployment_, saya membuat file _manifest_ baru bernama `deployment-recreate.yaml` dengan memodifikasi bagian _strategy_ dari file _deployment_ sebelumnya. Perubahan utama yang dilakukan adalah mengganti `type: RollingUpdate` menjadi `type: Recreate` dan menghapus konfigurasi **rollingUpdate** yang tidak diperlukan untuk strategi **Recreate**.
```yaml
spec:
  strategy:
    type: Recreate
```

File _manifest_ ini kemudian dapat digunakan dengan perintah `kubectl apply -f deployment-recreate.yaml` untuk menjalankan _deployment_ menggunakan strategi **Recreate** yang akan menghentikan semua _pod_ _existing_ sebelum membuat _pod_ baru.

### 4. What do you think are the benefits of using Kubernetes manifest files?
Menurut saya, keuntungan utama menggunakan **Kubernetes** `manifest files` adalah kemudahan dan konsistensi dalam _deployment_. Ketika saya melakukan _deployment_ secara manual, saya harus mengetik banyak perintah _kubectl_ satu per satu dan mudah sekali salah atau lupa langkah tertentu. Dengan `manifest files`, semua konfigurasi sudah ditulis sekali dalam file **YAML** dan bisa langsung diterapkan dengan perintah `kubectl apply -f`, jadi tidak perlu mengingat atau mengetik ulang perintah yang panjang. File _manifest_ juga bisa disimpan dan dibagikan dengan tim lain, sehingga semua orang bisa melakukan _deployment_ yang sama persis tanpa takut ada perbedaan konfigurasi. Selain itu, jika ada masalah, kita bisa dengan mudah melihat kembali konfigurasi yang digunakan atau bahkan mengembalikan ke versi sebelumnya, berbeda dengan _deployment_ manual yang sulit dilacak dan diulang dengan tepat.