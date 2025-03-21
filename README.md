# MODULE 6 - RUST WEBSERVER

## REFLECTIONS
- [MILESTONE 1](#milestone-1)
- [MILESTONE 2](#milestone-2)
- [MILESTONE 3](#milestone-3)
- [MILESTONE 4](#milestone-4)
- [MILESTONE 5](#milestone-5)

## MILESTONE 1
### REFLECTION
```rust
let buf_reader = BufReader::new(&mut stream);
```
`buf_reader = BufReader::new(&mut stream)` membuat buffered reader dari `stream` untuk untuk meningkatkan kecepatan read calls yang berulang.


```rust
let http_request: Vec<_> = buf_reader.lines()
    .map(|result| result.unwrap())
    .take_while(|line| !line.is_empty())
    .collect();

```
buf_reader.lines() membaca stream baris per baris. .map(|result| result.unwrap()) mengambil hasil dari setiap baris (Result<String, io::Error>) dan menggunakan .unwrap() untuk mendapatkan string dari Ok(String). .take_while(|line| !line.is_empty()) menghentikan pembacaan saat menemukan baris kosong, yang menandakan akhir dari header HTTP. .collect() mengumpulkan baris-baris yang telah difilter ke dalam Vec<String>.membaca stream per baris lalu mengambil hasil dari setiap baris 

```rust
println!("Request: {:#?}", http_request);  
```
Mencetak http_request dengan format `{:#?} agar lebih mudah untuk proses debug yang memiliki banyak baris.
---

## MILESTONE 2
### REFLECTION

![Commit 2 screen capture](/assets/images/commit2.png)

Dalam implementasi baru method handle_connection(), http_request tidak lagi dicetak ke terminal,tetapi sekarang method meyusun HTTP response.

```rust
let status_line = "HTTP/1.1 200 OK"; 
```
Program membuat status response.

```rust
let contents = fs::read_to_string("hello.html").unwrap(); let length = contents.len();
```
program membaca isi file HTML untuk kemudian dikirim sebagai response.

```rust
let response = format!("{status_line}\r\nContent-Length:
      {length}\r\n\r\n{contents}");
```
program menyusun HTTP respose yang sesuai dengan struktur dispesifikasikan HTTP/1.1

```rust
stream.write_all(response.as_bytes()).unwrap();
```
Setelrah respons disusun, kemudian akan dikirim ke klien menggunakan potongan kode di atas.
---

## MILESTONE 3
### REFLECTION

![Commit 3 screen capture](/assets/images/commit3.png)

Before refactoring
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
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
}
```

Sebelum refactor, kode menggunakan if-else untuk menangani HTTP request. Jika klien pergi ke "/", maka server akan membaca hello.html dan mengembalikan status 200 OK. Jika permintaan selain "/", maka server akan membaca 404.html dan mengembalikan status 404 NOT FOUND.Namun, ada duplikasi kode dalam proses penyusunan respons, di mana kedua cabang if-else memiliki kode yang hampir sama.

After refactoring
```rust
fn handle_connection(mut stream: TcpStream) {
    // --snip--

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

Kode setelah refactoring menghilangkan duplikasi kdoe dan mempermudah proses jika ingin menambah halaman lain dengan cukup memperbarui mapping `status_lline` dan `filename`.
---

## MILESTONE 4
### REFLECTION

Pada kode ini, teradapat penambahan rute `/sleep` yang menyebabkan program "tidur" seelama 5 detik sebelum lannjut megembalikan respons. Jika kita membuka 2 tab dengan satu thread, misal `/sleep` lalu `/`, maka respons `/` akan menunggu `thread::sleep` selesai sebelum lanjut merespons.

Hal ini terjadi karena program saat ini merespons HTTP requeset sacara sinkronus. Jika satu permintaan membutuhkan waktu yang lama, maka permintaan lain perlu menunggu peermintaan sebelumnya selesai.
---

## MILESTONE 5
### REFLECTION

ThreeadPool memungkinkan untuk menjalankan beberapa task secara paralel dengan menggunakan thread-thread yang dibuat terlebih dahulu sebelumnya berdasarkan parameter size.

Threadpool memiliki dua komponen utama. Yaitu worker (kumpulan thread) yang mennympan daftar threead yang bisa digunakan untuk melakukan task dan sender yang digunakan untuk mengirim task ke thread yang tersedia.

