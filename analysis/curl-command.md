Curl Commands - Case 1 User/Profile API

Base URL demo app: http://localhost:8088

Jalankan reset data terlebih dahulu:

curl -X POST http://localhost:8088/api/reset
1. GET - Daftar User
curl -i -X GET "http://localhost:8088/api/users"

Expected:

Status 200 OK
Body berisi count dan array data yang berisi seluruh user.
2. GET - Detail User
curl -i -X GET "http://localhost:8088/api/users/2"

Expected:

Status 200 OK
Body berisi satu object user berdasarkan ID.
Jika ID tidak ditemukan, akan mengembalikan 404 Not Found.
3. POST - Tambah User (Body Lengkap)
curl -i -X POST "http://localhost:8088/api/users" \
-H "Content-Type: application/json" \
-d '{"name":"Zaen Dinatussholihah","email":"zaen@gmail.com","username":"zaendina"}'

Expected:

Status 201 Created
Header Location: /api/users/{id-baru}
Body berisi data user yang baru ditambahkan.
4. POST - Uji Validasi Input (Body Tidak Lengkap)
curl -i -X POST "http://localhost:8088/api/users" \
-H "Content-Type: application/json" \
-d '{"name":"Zaen Dinatussholihah"}'

Expected:

Status 400 Bad Request
Body:
{
  "error": "Missing required fields",
  "missing": ["email", "username"]
}
5. POST - Update Profile User
curl -i -X POST "http://localhost:8088/api/users/1/profile" \
-H "Content-Type: application/json" \
-d '{"name":"Zaen D.","email":"zaenbaru@gmail.com"}'

Expected:

Status 200 OK
Body berisi data profile user yang telah diperbarui.
6. POST - Update Profile User Tidak Ditemukan
curl -i -X POST "http://localhost:8088/api/users/999/profile" \
-H "Content-Type: application/json" \
-d '{"name":"Test"}'

Expected:

404 Not Found

Body:

{
  "error": "User not found"
}
