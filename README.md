## *Refleksi*

### *1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?*  
- Unary: Client mengirim satu request, server membalas dengan satu response. Cocok untuk operasi standar seperti mengambil satu data spesifik atau autentikasi user.
- Server Streaming: Client mengirim satu request, server mengirim aliran (stream) data secara bertahap. Cocok untuk data yang besar atau real-time updates seperti feed berita atau harga saham.
- Bi-directional Streaming: Client dan server saling mengirim pesan secara terus-menerus (two-way). Paling pas untuk aplikasi interaktif berkelanjutan, seperti aplikasi chat atau game real-time.

### *2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?*  
- Autentikasi & Autorisasi: gRPC bisa menggunakan interceptors untuk memvalidasi token (seperti JWT) di setiap request.
- Enkripsi Data: Berjalan di atas HTTP/2, gRPC sangat dianjurkan (dan kadang diwajibkan) menggunakan TLS/SSL. Kamu bisa menerapkan mutual TLS (mTLS) agar client dan server saling memvalidasi sertifikat kriptografi mereka sebelum berkomunikasi.

### *3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?*  
Tantangan utamanya adalah manajemen state dan konkurensi. Pada aplikasi chat, server harus mengelola banyak koneksi secara bersamaan. Di Rust, jika koneksi terputus tiba-tiba, kamu harus memastikan channel komunikasi (seperti mpsc) ditutup dengan benar agar tidak terjadi memory leak. Sinkronisasi state antar task asinkron menggunakan Tokio juga butuh kehati-hatian agar tidak terjadi deadlock.

### *4. What are the advantages and disadvantages of using the `tokio_stream::wrappers::ReceiverStream` for streaming responses in Rust gRPC services?*  
- Kelebihan: Sangat memudahkan kita mengonversi receiver dari mpsc::channel milik Tokio menjadi Stream yang dikenali oleh framework Tonic untuk gRPC.
- Kekurangan: Membutuhkan alokasi memori untuk buffer (saat mendefinisikan kapasitas channel). Jika buffer penuh karena client lambat menerima data, proses di server bisa ikut tertahan (backpressure).

### *5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity promoting maintainability and extensibility over time?*  
Sama seperti saat merancang arsitektur menggunakan Spring Boot atau Django, kode gRPC sebaiknya memisahkan layer:
- Layer Transport: Khusus berisi handler gRPC dari Tonic.  
- Layer Layanan/Bisnis: Logika inti aplikasi dipisah dalam struct atau module tersendiri.  
- Layer Data: Mengelola koneksi ke database.
Memisahkan kode hasil generate Protobuf ke dalam modul khusus juga akan membuat codebase lebih rapi.

### *6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?*  
Di dunia nyata, service pembayaran tidak langsung merespons "sukses". Langkah tambahan yang diperlukan:  
- Menerapkan Idempotency Key agar pembayaran tidak terproses dua kali jika koneksi terputus.
- Integrasi ke payment gateway pihak ketiga.
- Pencatatan (logging) dan audit trail yang ketat, serta mekanisme rollback database jika terjadi kegagalan di tengah transaksi.

### *7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?*  
gRPC memaksa kita memiliki kontrak yang ketat antar layanan melalui file Protobuf. Ini sangat mempermudah lingkungan polyglot (multi-bahasa pemograman), karena stub client/server bisa di-generate secara otomatis untuk bahasa apa pun. Dampaknya, microservices menjadi lebih terikat secara kontrak, tapi komunikasinya jauh lebih efisien.

### *8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?*  
- Kelebihan HTTP/2: Memiliki fitur multiplexing (mengirim banyak request/response secara paralel di satu koneksi TCP tanpa head-of-line blocking), kompresi header (HPack) yang menghemat bandwidth, dan server push.
- Kekurangan: Implementasi dan debugging lebih sulit karena datanya berbentuk biner (binary framing), bukan teks biasa seperti HTTP/1.1 yang mudah dibaca.

### *9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?*  
REST mengikuti siklus satu request untuk satu response. Jika butuh data real-time, REST harus melakukan polling (bertanya ke server terus-menerus), yang mana sangat membebani jaringan. Sebaliknya, bi-directional streaming gRPC menahan satu koneksi tetap terbuka, sehingga server atau client bisa melempar data baru kapan pun secara instan tanpa perlu overhead koneksi baru.

### *10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?*  
- Protobuf: Karena berbasis skema biner, ukurannya jauh lebih kecil dan parsing-nya sangat cepat, sangat menghemat CPU. Validasi tipe data terjadi otomatis sesuai skema.  
- JSON: Bersifat plain-text sehingga human-readable dan sangat fleksibel (mudah ditambahkan field baru tanpa merusak skema). Kekurangannya, ukuran datanya lebih besar dan butuh langkah validasi ekstra saat di-parsing.

