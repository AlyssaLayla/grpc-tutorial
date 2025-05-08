**Nama**: Alyssa Layla Sasti  <br /> 
**Kelas**: AdPro B  <br />
**NPM**: 2306152052 <br />

## REFLECTION MODULE 8
1. What are the key differences between unary, server streaming, and bi-directional streaming RPC (Remote Procedure Call) methods, and in what scenarios would each be most suitable?
    Menurut saya, perbedaan utama terletak pada arah dan jumlah data yang dikirim:
    - Unary adalah komunikasi satu-ke-satu antara client dan server. Cocok untuk request sederhana seperti login, validasi data, atau transaksi sekali jalan. Contohnya adalah PaymentService dalam tutorial, yang hanya mengirim satu PaymentRequest dan menerima satu PaymentResponse.
    - Server Streaming memungkinkan client mengirim satu request, dan server merespons dengan banyak data secara bertahap. Ini cocok untuk kasus data besar seperti histori transaksi. Tutorial mengimplementasikan ini dalam TransactionService.
    - Bi-Directional Streaming memungkinkan client dan server bertukar pesan secara simultan dan terus menerus. Ini paling cocok untuk komunikasi real-time seperti aplikasi chat (ChatService). Tantangannya lebih besar karena perlu sinkronisasi dua arah secara bersamaan.

2. What are the potential security considerations involved in implementing a gRPC service in Rust, particularly regarding authentication, authorization, and data encryption?
    Menurut saya, ada beberapa potensi security dalam gRPC:
    - Autentikasi (Authentication): Sistem harus bisa memverifikasi identitas client, misalnya menggunakan token seperti JWT atau OAuth.
    - Otorisasi (Authorization): Setelah terautentikasi, perlu dicek apakah client punya izin untuk melakukan aksi tertentu, misalnya mengakses data pembayaran orang lain.
    - Enkripsi (Data Encryption): Data yang dikirim melalui jaringan harus dienkripsi, biasanya dengan TLS (Transport Layer Security). gRPC sendiri mendukung komunikasi terenkripsi secara default menggunakan HTTP/2 over TLS.

3. What are the potential challenges or issues that may arise when handling bidirectional streaming in Rust gRPC, especially in scenarios like chat applications?
    Saya menyadari bahwa bidirectional streaming seperti di ChatService menghadirkan tantangan tersendiri:
    - Kompleksitas asynchronous: Harus bisa menerima dan mengirim secara paralel tanpa blocking. Dalam tutorial, digunakan tokio::spawn untuk mengelola task menerima dan mengirim secara terpisah.
    - Manajemen channel dan error: Harus hati-hati dalam menangani error saat kirim/terima pesan. Kalau salah satu stream mati, aplikasi bisa crash atau kehilangan pesan.
    - Pengaturan buffer dan sinkronisasi: Buffer terlalu kecil bisa menyebabkan data menumpuk, terlalu besar bisa boros memori.

4. What are the advantages and disadvantages of using the tokio_stream::wrappers::ReceiverStream for streaming responses in Rust gRPC services?
    Penggunaan ReceiverStream sangat membantu dalam mengubah data dari channel ke stream yang bisa dibaca oleh tonic:
    - Kelebihan: Integrasi mudah dengan gRPC karena langsung kompatibel dengan Response::new(...).
    - Kekurangan: Jika sender tertutup atau error tidak ditangani, stream bisa terputus diam-diam tanpa error eksplisit.

5. In what ways could the Rust gRPC code be structured to facilitate code reuse and modularity, promoting maintainability and extensibility over time?
    Agar kode mudah dipelihara dan dikembangkan, saya menyadari pentingnya:
    - Memisahkan service berdasarkan file/modul, misalnya payment_service.rs, chat_service.rs.
    - Menggunakan trait agar implementasi tetap fleksibel dan mudah diuji.
    - Menghindari hardcoded string/url, gunakan konfigurasi via .env atau file.

6. In the MyPaymentService implementation, what additional steps might be necessary to handle more complex payment processing logic?
    Dalam tutorial, logikanya cukup sederhana. Namun di dunia nyata, saya menyadari bahwa:
    - Perlu ada validasi ketat atas nominal, identitas pengguna, dan status rekening.
    - Perlu mekanisme rollback jika transfer dana gagal (misalnya menggunakan database transaction atau compensating action).
    - Bisa ditambah pencatatan log transaksi, pengiriman notifikasi, atau integrasi API pihak ketiga (payment gateway).

7. What impact does the adoption of gRPC as a communication protocol have on the overall architecture and design of distributed systems, particularly in terms of interoperability with other technologies and platforms?
    Menurut saya, adopsi gRPC mengubah cara sistem berkomunikasi:
    - Menawarkan kinerja tinggi berkat Protobuf dan HTTP/2
    - Mendukung polyglot communication, karena gRPC bisa digunakan lintas bahasa (Rust, Go, Python, dll).
    - gRPC mendorong kita untuk berpikir lebih eksplisit (via .proto), tidak seperti REST yang lebih fleksibel tapi rawan kesalahan karena kurang standar.

8. What are the advantages and disadvantages of using HTTP/2, the underlying protocol for gRPC, compared to HTTP/1.1 or HTTP/1.1 with WebSocket for REST APIs?
    - HTTP/2 mendukung multiplexing dan streaming yang efisien, membuat gRPC unggul untuk komunikasi intensif.
    - HTTP/1.1 sederhana dan lebih kompatibel dengan tool lama, tapi kurang efisien.
    - WebSocket + HTTP/1.1 bisa digunakan untuk streaming, tapi implementasinya lebih kompleks dan tidak se-native gRPC.

9. How does the request-response model of REST APIs contrast with the bidirectional streaming capabilities of gRPC in terms of real-time communication and responsiveness?
    REST hanya cocok untuk komunikasi statis: client minta, server jawab. Ini menyulitkan kalau perlu update real-time. Sebaliknya, gRPC bisa stream data dua arah secara natural, membuatnya cocok untuk aplikasi chat, notifikasi, atau monitoring real-time.

10. What are the implications of the schema-based approach of gRPC, using Protocol Buffers, compared to the more flexible, schema-less nature of JSON in REST API payloads?
    Dengan Protobuf, struktur data lebih aman, perubahan terdeteksi saat compile-time, dan bisa menghasilkan kode otomatis untuk berbagai bahasa.
    - Protobuf (gRPC): format biner, cepat, efisien, dan punya definisi skema eksplisit. Sangat cocok untuk sistem besar dan komunikasi internal antar layanan.
    - JSON (REST): fleksibel, mudah dibaca manusia, cocok untuk API publik, tapi bisa menimbulkan error karena kurang validasi skema.