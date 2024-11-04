# Project Ujian Tengah Semester 7
Nama  : Salsa Bila Naqiyyah  
NIM   : 21.01.55.0018
# Deskripsi Project
Project Ujian Tengah Semester ini yaitu membuat Web Service REST untuk sistem manajemen sesuai objek yang ditentukan menggunakan PHP dan MySQL. Web Service yang mendukung operasi CRUD (Create, Read, Update, Delete) dan diuji menggunakan Postman. Tujuan project ini yaitu untuk memenuhi tugas Ujian Tengah Semester 7.
# Alat Yang Dibutuhkan
1. XAMPP (atau server web lain dengan PHP dan MySQL)  
2. Text editor (misalnya Visual Studio Code, Notepad++, dll)  
3. Postman
# Cara Instalsasi dan Penggunaan
## 1. Persiapan Lingkungan
1. Jalankan XAMPP Control Panel dan aktifkan Apache dan MySQL  
2. Buat folder baru bernama tugas_uts di dalam direktori htdocs XAMPP Anda.  
## 2. Membuat Database
1. Buka phpMyAdmin http://localhost/phpmyadmin   
2. Buat database baru bernama courses  
3. Pilih database courses, lalu buka tab SQL  
4. Jalankan query SQL berikut untuk membuat tabel dan menambahkan data sampel:
```sql
CREATE TABLE courses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    instructor VARCHAR(255) NOT NULL,
    duration INT NOT NULL,
    price FLOAT (10, 2) NOT NULL
);

INSERT INTO courses (title, instructor, duration, price) VALUES 
('Dasar-Dasar Basis Data', 'Dr. Budi Santoso', 40, 500000.00), 
('Pengembangan Web Dasar', 'Ibu Siti Aminah', 60, 350000.00), 
('Ilmu Data dengan Python', 'Dr. Ani Wulandari', 50, 750000.00), 
('Pembelajaran Mesin untuk Pemula', 'Bapak Joko Widodo', 45, 600000.00), 
('Keamanan Siber untuk Pemula', 'Ibu Rina Sari', 35, 450000.00);
```
## 3. Membuat File PHP untuk Web Service
1. Buka text editor Anda seperti Visual Studio Code
2. Buat file baru dan simpan sebagai courses_api.php di dalam folder tugas_uts.
3. Salin dan tempel kode berikut ke dalam courses_api.php:
```php
<?php
header("Content-Type: application/json");

$host = 'localhost';
$db_name = 'courses';
$username = 'root';
$password = '';

$conn = new mysqli($host, $username, $password, $db_name);

if ($conn->connect_error) {
    die(json_encode(["status" => "error", "message" => "Koneksi gagal: " . $conn->connect_error]));
}

$method = $_SERVER['REQUEST_METHOD'];

switch ($method) {
    case 'GET':
        if (isset($_GET['id'])) {
            $id = intval($_GET['id']);
            $query = "SELECT * FROM courses WHERE id = ?";
            $stmt = $conn->prepare($query);
            $stmt->bind_param("i", $id);
            $stmt->execute();
            $result = $stmt->get_result();
            if ($result->num_rows > 0) {
                $course = $result->fetch_assoc();
                echo json_encode(["status" => "success", "message" => "Data ditemukan", "data" => $course]);
            } else {
                http_response_code(404);
                echo json_encode(["status" => "error", "message" => "Data tidak ditemukan", "data" => null]);
            }
            $stmt->close();
        } else {
            $search = isset($_GET['title']) ? $_GET['title'] : '';
            $query = "SELECT * FROM courses";
            if ($search) {
                $search = $conn->real_escape_string($search);
                $query .= " WHERE title LIKE '%$search%'";
            }
            $result = $conn->query($query);
            if ($result->num_rows > 0) {
                $courses = [];
                while ($row = $result->fetch_assoc()) {
                    $courses[] = $row;
                }
                echo json_encode(["status" => "success", "message" => "Data ditemukan", "data" => $courses]);
            } else {
                echo json_encode(["status" => "success", "message" => "Tidak ada data", "data" => []]);
            }
        }
        break;

    case 'POST':
        $data = json_decode(file_get_contents("php://input"), true);
        if (empty($data['title']) || empty($data['instructor']) || !isset($data['duration']) || !isset($data['price'])) {
            http_response_code(400);
            echo json_encode(["status" => "error", "message" => "Semua field harus diisi.", "data" => null]);
            exit();
        }
        $title = $conn->real_escape_string($data['title']);
        $instructor = $conn->real_escape_string($data['instructor']);
        $duration = intval($data['duration']);
        $price = floatval($data['price']);
        $query = "INSERT INTO courses (title, instructor, duration, price) VALUES (?, ?, ?, ?)";
        $stmt = $conn->prepare($query);
        $stmt->bind_param("ssdi", $title, $instructor, $duration, $price);
        if ($stmt->execute()) {
            http_response_code(201);
            echo json_encode(["status" => "success", "message" => "Data berhasil ditambahkan.", "data" => null]);
        } else {
            http_response_code(500);
            echo json_encode(["status" => "error", "message" => "Gagal menambahkan data.", "data" => null]);
        }
        $stmt->close();
        break;

    case 'PUT':
        if (isset($_GET['id'])) {
            $id = intval($_GET['id']);
            $query = "SELECT * FROM courses WHERE id = ?";
            $stmt = $conn->prepare($query);
            $stmt->bind_param("i", $id);
            $stmt->execute();
            $result = $stmt->get_result();
            if ($result->num_rows > 0) {
                $data = json_decode(file_get_contents("php://input"), true);
                if (empty($data['title']) || empty($data['instructor']) || !isset($data['duration']) || !isset($data['price'])) {
                    http_response_code(400);
                    echo json_encode(["status" => "error", "message" => "Semua field harus diisi.", "data" => null]);
                    exit();
                }
                $title = $conn->real_escape_string($data['title']);
                $instructor = $conn->real_escape_string($data['instructor']);
                $duration = intval($data['duration']);
                $price = floatval($data['price']);
                $queryUpdate = "UPDATE courses SET title = ?, instructor = ?, duration = ?, price = ? WHERE id = ?";
                $stmtUpdate = $conn->prepare($queryUpdate);
                $stmtUpdate->bind_param("ssdii", $title, $instructor, $duration, $price, $id);
                if ($stmtUpdate->execute()) {
                    http_response_code(200);
                    echo json_encode(["status" => "success", "message" => "Data berhasil diupdate.", "data" => null]);
                } else {
                    http_response_code(500);
                    echo json_encode(["status" => "error", "message" => "Gagal mengupdate data.", "data" => null]);
                }
                $stmtUpdate->close();
            } else {
                http_response_code(404);
                echo json_encode(["status" => "error", "message" => "Data tidak ditemukan", "data" => null]);
            }
            $stmt->close();
        } else {
            http_response_code(400);
            echo json_encode(["status" => "error", "message" => "ID harus diisi.", "data" => null]);
        }
        break;

    case 'DELETE':
        if (isset($_GET['id'])) {
            $id = intval($_GET['id']);
            $queryCheck = "SELECT * FROM courses WHERE id = ?";
            $stmtCheck = $conn->prepare($queryCheck);
            $stmtCheck->bind_param("i", $id);
            $stmtCheck->execute();
            $resultCheck = $stmtCheck->get_result();

            if ($resultCheck->num_rows > 0) {
                $queryDelete = "DELETE FROM courses WHERE id = ?";
                $stmtDelete = $conn->prepare($queryDelete);
                $stmtDelete->bind_param("i", $id);
                $stmtDelete->execute();
        
                if ($stmtDelete->affected_rows > 0) {
                    http_response_code(200); 
                    echo json_encode(["status" => "success", "message" => "Data Berhasil Dihapus", "data" => null]);
                } else {
                    http_response_code(500);
                    echo json_encode(["status" => "error", "message" => "Gagal menghapus data.", "data" => null]);
                }
                $stmtDelete->close();
            } else {
                http_response_code(404);
                echo json_encode(["status" => "error", "message" => "Data tidak ditemukan", "data" => null]);
            }
            $stmtCheck->close();
        } else {
            http_response_code(400);
            echo json_encode(["status" => "error", "message" => "ID harus diisi.", "data" => null]);
        }
        break;
        
    default:
        http_response_code(405);
        echo json_encode(["status" => "error", "message" => "Metode tidak diizinkan.", "data" => null]);
        break;
}

$conn->close();
?>
```
## 4. Pengujian Menggunakan Postman
1. Buka Postman
2. Buat request baru untuk setiap operasi berikut:
## a. GET All Courses & Tittle
### Menampilkan semua data
- Method: GET
- URL: http://localhost/tugas_uts/courses_api.php
- Klik "Send"
### Mendukung pencarian berdasarkan nama/title
- Method: GET
- URL: http://localhost/tugas_uts/courses_api.php?title=Dasar-Dasar
- Klik "Send"
## b. GET Specific Courses
### Menampilkan detail data berdasarkan ID
- Method: GET
- URL: http://localhost/tugas_uts/courses_api.php?id=5 (untuk courses dengan ID 5)
- Klik "Send"
### Response 404 jika data tidak ditemukan
- Method: GET
- URL: http://localhost/tugas_uts/courses_api.php?id=8 (untuk courses dengan ID 8)
- Klik "Send"
## c. POST New Courses
### Menambah data baru Courses
- Method: POST
- URL: http://localhost/tugas_uts/courses_api.php
- Headers:
  - Key: Content-Type
  - Value: application/json
- Body:
  - Pilih "raw" dan "JSON"
  - Masukan :
    ```php
      {
        "id": "7",
        "title": "Belajar PHP untuk Pemula",
        "instructor": "Dr. Salsa Bila",
        "duration": "60",
        "price": "300000.00"
      }
- Klik "Send"
## d. PUT (Update) Courses
### Mengupdate data berdasarkan ID
- Method: PUT
- URL: http://localhost/tugas_uts/courses_api.php?id=2 (asumsikan ID courses baru adalah 2)
- Headers:
  - Key: Content-Type
  - Value: application/json
- Body:
- Pilih "raw" dan "JSON"
- Masukkan
```php
{
        "id": "2",
        "title": "Pengembangan Web Dasar",
        "instructor": "Ibu Siti Aminah",
        "duration": "60",
        "price": "350000.00"
}
```
### Response 404 jika data tidak ditemukan
- Method: PUT
- URL: http://localhost/tugas_uts/courses_api.php?id=9 (asumsikan courses dengan ID adalah 9)
- Klik "Send"
## e. DELETE Book
### Menghapus data berdasarkan ID
- Method: DELETE
- URL: http://localhost/tugas_uts/courses_api.php?id=6 (untuk menghapus courses dengan ID 6)
- Klik "Send"
### Response 404 jika data tidak ditemukan
- Method: DELETE
- URL: http://localhost/tugas_uts/courses_api.php?id=10 (untuk menghapus courses dengan ID 10)
- Klik "Send"

