# Modul 6

### Reflection Commit 1

1. Mendengarkan Koneksi TCP:
+ Kode tersebut merupakan implementasi dasar dari sebuah server Rust yang mendengarkan koneksi TCP yang masuk.
+ Dari modul standar (std::net::TcpListener), kita memulai dengan mengimpor modul yang diperlukan.
+ Pada fungsi main, kita membuat instance dari TcpListener dengan mengikatnya ke alamat 127.0.0.1:7878 menggunakan TcpListener::bind. Ini berarti server akan mendengarkan koneksi masuk pada port 7878 dari localhost.
+ Metode unwrap digunakan untuk menangani kesalahan yang mungkin terjadi selama proses pengikatan.

2. Menangani Koneksi Masuk:
+ Kode memasuki sebuah loop dengan menggunakan listener.incoming(), yang mengembalikan iterator atas koneksi masuk.
+ Untuk setiap koneksi masuk, yang direpresentasikan oleh stream, fungsi handle_connection dipanggil.

3. Menangani Setiap Koneksi:
+ Fungsi handle_connection didefinisikan untuk mengambil referensi mutable ke TcpStream.
+ Di dalam handle_connection, sebuah BufReader dibuat untuk membaca data dari stream dengan efisien.
+ Baris-baris permintaan HTTP dibaca dari stream menggunakan buf_reader.lines() dan dikumpulkan ke dalam sebuah vektor (http_request).
+ Kumpulan baris permintaan HTTP diprint ke konsol untuk inspeksi menggunakan format debug yang rapi ({:#?}).

4. Menjalankan Server:
+ Ketika server dijalankan (cargo run), ia mulai mendengarkan koneksi masuk.
+ Ketika sebuah browser mengirim permintaan ke 127.0.0.1:7878, server menerima permintaan tersebut, memprintnya, dan melanjutkan mendengarkan koneksi masuk lainnya.

5. Interpretasi Permintaan:
+ Permintaan yang diprint mengandung beberapa baris, termasuk metode HTTP (misalnya, GET), jalur permintaan (misalnya, /), versi HTTP, header, dan informasi user-agent.

### Reflection Commit 2
Pada commit ini, fungsi `handle_connection` diperluas dengan beberapa tambahan kode. Sekarang, fungsi ini tidak hanya dapat membaca permintaan melalui stream TCP, tetapi juga dapat meresponsnya. Proses ini dilakukan dengan mengirim sebuah respons HTTP kepada client yang berisi konten `hello.html`.

Proses respon dibuat dengan membuat status line respons yang menyatakan bahwa permintaan berhasil diproses dengan kode "200 OK". Kemudian, isi file `hello.html` dibaca dan panjang kontennya dihitung untuk disertakan dalam header respons. Respons HTTP yang lengkap, termasuk status line, header, dan konten, kemudian dibuat dan dikirim kembali ke client melalui stream yang sama.