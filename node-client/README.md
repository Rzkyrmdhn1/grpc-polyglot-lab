# gRPC Polyglot Lab

Praktikum komunikasi inter-service dengan gRPC. Satu server Go dipanggil oleh dua client berbeda bahasa (Python dan Node.js) dengan satu kontrak yang sama, yaitu `user.proto`.

## Struktur Proyek

- `user.proto`: kontrak API (Single Source of Truth)
- `go-server/`: gRPC server (Go), port 50051
- `python-client/`: gRPC client (Python)
- `node-client/`: gRPC client (Node.js)

## Analisis: Static Code Generation vs Dynamic Runtime Generation

Pada pendekatan static code generation (Go dan Python), file `user.proto` dikompilasi terlebih dahulu oleh `protoc` sebelum program dijalankan. Hasilnya adalah file kode sumber nyata (`user.pb.go` dan `user_grpc.pb.go` di Go, serta `user_pb2.py` dan `user_pb2_grpc.py` di Python) yang berisi struct/class pesan, stub client, dan interface server. File ini kemudian diimpor langsung oleh aplikasi, sehingga kesalahan tipe data atau nama field bisa terdeteksi lebih awal (terutama di Go pada saat kompilasi) dan IDE dapat memberi autocomplete. Konsekuensinya, setiap kali skema berubah, misalnya saat menambah field `phone`, kode harus di-generate ulang dan aplikasi dijalankan ulang agar perubahan terbaca. Sebaliknya, pada dynamic runtime generation (Node.js), tidak ada langkah `protoc` dan tidak ada file hasil generate. Library `@grpc/proto-loader` membaca dan mem-parsing `user.proto` saat program berjalan (`loadSync`), lalu membangun definisi pesan dan service di memori. Perubahan skema cukup diikuti dengan menjalankan ulang program tanpa proses kompilasi ulang, sehingga alur kerjanya lebih sederhana dan fleksibel. Kekurangannya, tidak ada pengecekan tipe saat kompilasi (kesalahan baru ketahuan saat runtime), ada sedikit biaya parsing saat startup, dan file `.proto` harus tersedia di lokasi yang bisa dibaca ketika aplikasi berjalan.
