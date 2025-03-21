# MODULE 6 - RUST WEBSERVER

## REFLECTIONS
- [MILESTONE 1](#milestone-1)
- [MILESTONE 2](#milestone-)

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