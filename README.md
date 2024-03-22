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

Bukti ScreenShot konten `hello.html`
![Commit 2 screen capture](/assets/images/commit2.png)

### Reflection Commit 3
Sebelumnya, website hanya akan merespons setiap request dari user dengan mengirimkan konten HTML dari file `hello.html`. Respon ini tidak memperhatikan apa yang diminta oleh user, sehingga konten yang dikirimkan semua sama yaitu `hello.html`. Pada commit ini, saya mencoba untuk memisahkan response dengan beberapa langkah berikut:

1. Buat file baru dengan nama `404.html` untuk menjadi konten HTML jika path tidak tersedia pada server
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
  </body>
</html>
```

2. Modifikasi file `main.rs` agar function `handle_connection` dapat menghandle response
```rust
...
let buf_reader = BufReader::new(&mut stream);
let request_line = buf_reader.lines().next().unwrap().unwrap();

if request_line == "GET / HTTP/1.1" {
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
} else {
    let status_line = "HTTP/1.1 404 NOT FOUND";
    let contents = fs::read_to_string("404.html").unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
}
```
Pada kode di atas, terdapat pembagian respons menjadi dua kasus, yaitu untuk path "/" dan path lainnya. Jika path adalah "/", maka server akan merespons dengan HTML dari berkas `hello.html`, sedangkan jika path tidak "/", maka respons akan menggunakan HTML dari berkas `404.html`. Untuk menentukan path tersebut, baris pertama dari HTTP request dibaca menggunakan `let request_line = buf_reader.lines().next().unwrap().unwrap();`. Pada bagian ini, method `.next()` digunakan untuk mengambil item pertama dari iterator. Kemudian, `.unwrap()` pertama digunakan untuk mengekstrak nilai dari Option yang dihasilkan, dan `.unwrap()` kedua digunakan untuk menangani hasil dari `Result` seperti yang digunakan sebelumnya.

Namun, setelah menerapkan logika seperti itu, ditemukan bahwa kode perlu direfaktor karena beberapa alasan:

1. **Pengurangan Duplikasi Kode**: Sebelum refaktor, blok kode yang berulang untuk membaca file dan mengirim respons terduplikasi di dalam kedua blok `if` dan `else`. Situasi ini tidak efisien karena mengharuskan perubahan dilakukan pada dua lokasi yang berbeda, meningkatkan potensi kesalahan. Dengan memperkenalkan logika bersama untuk menentukan status line dan nama file, kode menjadi lebih bersih dan kurang rentan terhadap kesalahan karena perubahan hanya perlu dilakukan satu kali.
   
2. **Peningkatan Keterbacaan Kode**: Dengan memisahkan logika untuk menentukan status line dan nama file dari proses membaca file dan mengirim respons, kode menjadi lebih mudah dimengerti. Pembagian tugas yang jelas membuat kode lebih terstruktur dan membantu pengembang dalam memahami fungsi masing-masing bagian.
   
3. **Fleksibilitas dalam Perubahan Masa Depan**: Jika ada kebutuhan untuk mengubah cara server membaca file atau mengirim respons di masa mendatang, perubahan hanya perlu dilakukan sekali. Pendekatan ini secara signifikan membantu dalam pemeliharaan kode dan pengembangan fitur baru, karena pengembang tidak perlu mencari dan memperbarui setiap tempat yang terpengaruh.

[REFACTOR] fungsi `handle_connection` menjadi berikut:

```rust
...
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

Bukti ScreenShot konten jika path tidak sesuai/tersedia
![Commit 3 screen capture](/assets/images/commit3failed.png)