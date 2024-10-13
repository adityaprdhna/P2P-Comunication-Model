
### Disusun Oleh

Aditya Eka Nanda 320220401001

I Made Aditya Pradana Putra 320220401007

Nisrina Labiba Sarwoko 320220401020


# P2P-Communication-Model
Proyek ini merupakan implementasi sistem Peer-to-Peer (P2P) sederhana menggunakan Python, yang memungkinkan beberapa komputer atau **node** untuk saling berkomunikasi dengan mengirim pesan dan file secara langsung. Setiap node dapat berperan sebagai **server** sekaligus **klien**, sehingga sistem ini tidak membutuhkan server pusat untuk berfungsi.

## Fitur Utama

- **Pengiriman Pesan Peer-to-Peer**: Node bisa mengirim dan menerima pesan satu sama lain secara langsung.
- **Pengiriman File**: Node dapat mengirim dan menerima file antar satu sama lain.
- **Mekanisme Penyiaran (Flooding)**: Untuk menghindari pengiriman ulang pesan atau file yang sama, sistem mencatat pesan atau file yang telah diterima sebelumnya.
- **Multi-threading**: Setiap node dapat menangani beberapa koneksi sekaligus menggunakan thread, memungkinkan node untuk terus menerima dan memproses pesan dari beberapa node secara bersamaan.

## Cara Kerja

1. **Setiap Node Berperan Sebagai Server**:  
   Setiap node menjalankan server yang mendengarkan koneksi dari node lain. Saat node lain mencoba terhubung, server akan membuat thread terpisah untuk menangani koneksi tersebut.

2. **Penanganan Koneksi dan Pengiriman Data**:  
   Server menerima dua jenis data: **pesan** atau **file**. Saat pesan atau file diterima, sistem memeriksa apakah pesan atau file tersebut sudah pernah diterima. Jika belum, pesan akan disebarkan (di-flood) ke node lain.

3. **Penyebaran Pesan (Flooding)**:  
   Setelah menerima pesan, node menyebarkannya ke node tetangga yang terhubung, sehingga seluruh jaringan akhirnya menerima pesan tersebut. Sistem mencatat pesan yang sudah diterima untuk mencegah duplikasi.

4. **Pengiriman File**:  
   Saat file diterima, file tersebut disimpan secara lokal dengan nama asli, dan sistem mencatat ukuran file untuk memastikan penerimaan file yang lengkap.

## Struktur Kode

### 1. Memulai Server di Node

Node memulai server menggunakan fungsi berikut untuk mendengarkan koneksi masuk:
```python
def start_server(host, port):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(5)
    print(f"Node berjalan sebagai server di {host}:{port}")
```

### 2. Menangani Koneksi dari Node Lain

Saat node terhubung, fungsi berikut menangani data yang diterima:
```python
def handle_client(conn, addr):
    global received_messages
    while True:
        try:
            header = conn.recv(1024).decode()
            if header.startswith("MESSAGE"):
                message = header[len("MESSAGE "):]
                if message not in received_messages:
                    received_messages.add(message)
                    flood_message(message)
            elif header.startswith("FILE"):
                file_name, file_size = header.split()[1:3]
                receive_file(conn, file_name, int(file_size))
        except Exception as e:
            print(f"Error menerima pesan dari {addr}: {e}")
            break
```

### 3. Flooding Pesan

Fungsi berikut menyebarkan pesan ke node tetangga:
```python
def flood_message(message):
    for neighbor in neighbors:
        send_message_to_neighbor(neighbor, message)
```

### 4. Menerima File

Node menerima file menggunakan fungsi ini, yang menulis file ke disk:
```python
def receive_file(conn, file_name, file_size):
    with open(file_name, 'wb') as f:
        bytes_received = 0
        while bytes_received < file_size:
            data = conn.recv(1024)
            f.write(data)
            bytes_received += len(data)
        print(f"File {file_name} berhasil diterima.")
```

## Cara Menggunakan

### 1. Menjalankan Node
Setiap node dalam jaringan harus menjalankan, dengan mengganti dan menambahkan masing-masing port pada device lain
```python
   # Set host dan port untuk node ini
    host = '10.2.55.88'    # Port Sendiri
    port = 5001

    # Tetangga yang akan terhubung (alamat IP dan port node tetangga)
    neighbors = [('10.2.55.46', 15006)]  # Port tetangga
```
Ganti `'host = '10.2.55.88'` dengan alamat IP yang ada dalam device masing masing dan kemudian memasukkan `'port = '5001'` sesuai dengan yang diinginkan, dan pada `neighbors` masukkan alamat IP dan Port device lain agar dapat terhubung satu sama lain.

### 2. Menghubungkan ke Node Lain
Node dapat terhubung ke node tetangga dengan menambahkan node tersebut ke variabel `neighbors`. Setelah terhubung, node dapat mengirim pesan atau file.

### 3. Mengirim Pesan atau File
Node dapat mengirim pesan atau file ke node lain yang terhubung. Sistem akan secara otomatis menyebarkan pesan ke seluruh jaringan dan mencegah duplikasi.


## Note
Dalam melakukan praktek Peer to Peer ini harus dilakukan menggunakan jaringan yang sama ataupun Local Area agar saling terhubung dan dapat melakukan tugasnya.
