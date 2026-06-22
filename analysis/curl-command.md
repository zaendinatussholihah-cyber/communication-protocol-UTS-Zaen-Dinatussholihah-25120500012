# Curl Commands - Case 1 User/Profile API

Base URL demo app: http://localhost:8088

Jalankan reset data:

```bash
curl -X POST http://localhost:8088/api/reset
```

## 1. GET - Daftar User

```bash
curl -i -X GET "http://localhost:8088/api/users"
```

Expected:
- 200 OK
- Body berisi `count` dan array `data` (seluruh pengguna).

---

## 2. GET - Detail User

```bash
curl -i -X GET "http://localhost:8088/api/users/2"
```

Expected:
- 200 OK
- Body berisi satu object user berdasarkan id.
- Jika id tidak ditemukan maka `404 Not Found`.

---

## 3. POST - Tambah User (Body Lengkap)

```bash
curl -i -X POST "http://localhost:8088/api/users" \
-H "Content-Type: application/json" \
-d '{"name":"Zaen Dinatussholihah","email":"zaen@gmail.com","username":"zaen12"}'
```

Expected:
- 201 Created
- Header `Location: /api/users/{id-baru}`
- Body berisi data user yang berhasil ditambahkan.

---

## 4. POST - Uji Validasi Input (Body Tidak Lengkap)

```bash
curl -i -X POST "http://localhost:8088/api/users" \
-H "Content-Type: application/json" \
-d '{"name":"Zaen Dinatussholihah"}'
```

Expected:
- 400 Bad Request
- Body:

```json
{
  "error": "Missing required fields",
  "missing": ["email", "username"]
}
```

---

## 5. POST - Update Profile User

```bash
curl -i -X POST "http://localhost:8088/api/users/1/profile" \
-H "Content-Type: application/json" \
-d '{"name":"Zaen D.","email":"zaenbaru@gmail.com"}'
```

Expected:
- 200 OK
- Body berisi data profil yang sudah diperbarui.

---

## 6. POST - Update Profile User Tidak Ditemukan

```bash
curl -i -X POST "http://localhost:8088/api/users/999/profile" \
-H "Content-Type: application/json" \
-d '{"name":"Test"}'
```

Expected:
- 404 Not Found

```json
{
  "error": "User not found"
}
