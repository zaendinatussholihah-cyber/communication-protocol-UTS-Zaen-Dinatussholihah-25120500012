# Analisis Encoding - Case 1 User/Profile API

## 1. Format Encoding

API User/Profile ini menggunakan format **JSON (JavaScript Object Notation)** sebagai satu-satunya format encoding untuk request body maupun response body. Hal ini terlihat dari header `Content-Type: application/json; charset=utf-8` yang konsisten muncul pada seluruh response dari server.

JSON dipilih karena beberapa alasan yang relevan dengan konteks API ini:

* **Mudah dibaca manusia** — struktur key-value plaintext memudahkan debugging langsung dari terminal atau Postman.
* **Ringan** — tidak ada tag pembungkus seperti XML, sehingga ukuran payload lebih kecil.
* **Didukung luas** — hampir semua bahasa pemrograman dan HTTP client (termasuk Postman, curl, browser) mendukung parsing JSON secara native.
* **Cocok untuk data semi-terstruktur** — data user memiliki field dengan tipe data yang beragam (string dan number) yang terwakili langsung dalam JSON.

---

# 2. Struktur Data per Response

## 2.1 GET /api/users — Daftar User

**Response body (200 OK):**

```json
{
  "count": 3,
  "data": [
    {
      "id": 1,
      "name": "Andi Saputra",
      "email": "andi@gmail.com",
      "username": "andi01"
    },
    {
      "id": 2,
      "name": "Budi Santoso",
      "email": "budi@gmail.com",
      "username": "budi02"
    },
    {
      "id": 3,
      "name": "Citra Lestari",
      "email": "citra@gmail.com",
      "username": "citra03"
    }
  ]
}
```

**Struktur root** adalah sebuah JSON object dengan dua field:

* `count` (number) — jumlah total user dalam response.
* `data` (array of object) — daftar user.

Setiap object di dalam array memiliki field:

* `id` (number) — identitas unik user.
* `name` (string) — nama lengkap user.
* `email` (string) — alamat email user.
* `username` (string) — nama pengguna untuk login.

---

## 2.2 GET /api/users/{id} — Detail User

**Response body (200 OK):**

```json
{
  "id": 2,
  "name": "Budi Santoso",
  "email": "budi@gmail.com",
  "username": "budi02"
}
```

**Struktur root** adalah sebuah object tunggal (bukan array). Field-fieldnya identik dengan satu elemen di dalam `data` pada response daftar.

**Perbedaan dengan response daftar:** Response detail tidak memiliki wrapper `count` dan `data`, karena client sudah menentukan ID user yang ingin diambil.

---

## 2.3 POST /api/users — Tambah User (body valid)

**Request body yang dikirim:**

```json
{
  "name": "Zaen Dinatussholihah",
  "email": "zaen@gmail.com",
  "username": "zaen12"
}
```

**Response body (201 Created):**

```json
{
  "id": 4,
  "name": "Zaen Dinatussholihah",
  "email": "zaen@gmail.com",
  "username": "zaen12",
  "createdAt": "2026-06-22T10:13:58+0000"
}
```

**Field tambahan yang muncul di response:**

* `id` (number) — ID baru yang digenerate server secara otomatis.
* `createdAt` (string, format ISO 8601) — waktu pembuatan data user.

**Header tambahan di response:**

```http
Location: /api/users/4
```

Header ini memberitahu client lokasi resource baru yang berhasil dibuat.

---

## 2.4 POST /api/users — Uji Validasi Input (body tidak lengkap)

**Request body yang dikirim:**

```json
{
  "name": "Zaen Dinatussholihah"
}
```

**Response body (400 Bad Request):**

```json
{
  "error": "Missing required fields",
  "missing": [
    "email",
    "username"
  ]
}
```

**Struktur pesan error:**

* `error` (string) — penjelasan singkat mengenai kesalahan.
* `missing` (array of string) — daftar field yang belum diisi.

---

# 3. Analisis HTTP Method

### Mengapa GET dipakai untuk membaca data?

GET digunakan untuk mengambil daftar user dan detail user tanpa mengubah data pada server. GET bersifat **safe** dan **idempotent**, sehingga pemanggilan berulang tidak mengubah kondisi server.

### Mengapa POST dipakai untuk menambah user?

POST digunakan untuk membuat resource baru di server. Setiap request POST yang berhasil akan menghasilkan data user baru dengan ID yang berbeda sehingga POST tidak bersifat idempotent.

### Perbedaan POST (create) vs PATCH/PUT (update)

* `POST /api/users` → membuat user baru.
* `PUT /api/users/{id}` → mengganti seluruh data user.
* `PATCH /api/users/{id}` → memperbarui sebagian data user.
* `POST /api/users/{id}/profile` → memperbarui profil user pada demo aplikasi.

---

# 4. Analisis Status Code

| Request                         | Status Code     | Arti                          | Mengapa server memberikan status ini       |
| ------------------------------- | --------------- | ----------------------------- | ------------------------------------------ |
| GET /api/users                  | 200 OK          | Permintaan berhasil           | Server berhasil mengembalikan daftar user  |
| GET /api/users/2                | 200 OK          | Permintaan berhasil           | Server berhasil menemukan user dengan id=2 |
| POST /api/users (valid)         | 201 Created     | Resource baru berhasil dibuat | Server berhasil menyimpan data user baru   |
| POST /api/users (tidak lengkap) | 400 Bad Request | Request tidak valid           | Ada field wajib yang tidak dikirim         |

Status code tambahan:

* **404 Not Found** — muncul jika `GET /api/users/999`.
* **500 Internal Server Error** — muncul jika terjadi kesalahan tak terduga di sisi server.

---

# 5. Analisis Header Penting

### `Content-Type: application/json; charset=utf-8`

Menjelaskan bahwa isi body menggunakan format JSON dengan encoding UTF-8.

### `Location: /api/users/4`

Menunjukkan URL resource user yang baru dibuat.

### `Content-Length`

Memberitahu ukuran body response dalam satuan byte.

### `Access-Control-Allow-Origin: *`

Mengizinkan request dari domain manapun.

### `Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS`

Menentukan metode HTTP yang diizinkan server.

---

# 6. Validasi Input & Error Handling

Field yang wajib diisi:

* `name`
* `email`
* `username`

Jika ada field yang kosong:

```http
400 Bad Request
```

Response:

```json
{
  "error": "Missing required fields",
  "missing": [
    "email",
    "username"
  ]
}
```

Jika user tidak ditemukan:

```http
404 Not Found
```

Response:

```json
{
  "error": "User not found"
}
```

Format error seperti ini memudahkan client mengetahui penyebab kegagalan request.

---

# 7. Kesimpulan

Berdasarkan pengujian User/Profile API, metode GET digunakan untuk mengambil data pengguna, sedangkan metode POST digunakan untuk menambahkan dan memperbarui data pengguna. Response server menggunakan format JSON sehingga mudah dibaca dan diproses oleh aplikasi.

Status code HTTP membantu pengguna mengetahui apakah request berhasil atau terjadi kesalahan. Header seperti Content-Type dan Location berperan penting dalam proses pertukaran data. Dari praktikum ini dapat dipahami cara kerja komunikasi client-server, penggunaan metode HTTP, struktur response JSON, serta pentingnya analisis protocol dalam pertukaran data.
