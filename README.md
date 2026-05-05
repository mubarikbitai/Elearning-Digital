elearning/
│── config/
│   └── koneksi.php
│── assets/
│   └── style.css
│── auth/
│   ├── login.php
│   ├── logout.php
│── admin/
│   ├── dashboard.php
│   ├── materi.php
│   ├── nilai.php
│── siswa/
│   ├── dashboard.php
│   ├── tugas.php
│── uploads/
│── index.php
│── database.sql

<?php
$conn = mysqli_connect("localhost", "root", "", "elearning");

if (!$conn) {
    die("Koneksi gagal: " . mysqli_connect_error());
}
?>
CREATE DATABASE elearning;
USE elearning;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50),
    password VARCHAR(100),
    role ENUM('admin','guru','siswa')
);

CREATE TABLE materi (
    id INT AUTO_INCREMENT PRIMARY KEY,
    judul VARCHAR(100),
    deskripsi TEXT,
    file VARCHAR(255)
);

CREATE TABLE tugas (
    id INT AUTO_INCREMENT PRIMARY KEY,
    siswa VARCHAR(50),
    file VARCHAR(255),
    nilai INT
);

INSERT INTO users VALUES
(1,'admin','123','admin'),
(2,'guru','123','guru'),
(3,'siswa','123','siswa');

<?php
session_start();
include '../config/koneksi.php';

if(isset($_POST['login'])){
    $user = $_POST['username'];
    $pass = $_POST['password'];

    $data = mysqli_query($conn,"SELECT * FROM users WHERE username='$user' AND password='$pass'");
    $cek = mysqli_num_rows($data);

    if($cek > 0){
        $d = mysqli_fetch_assoc($data);
        $_SESSION['username'] = $user;
        $_SESSION['role'] = $d['role'];

        if($d['role']=="admin"){
            header("location:../admin/dashboard.php");
        }else{
            header("location:../siswa/dashboard.php");
        }
    }else{
        echo "Login gagal!";
    }
}
?>

<form method="post">
<h2>Login</h2>
<input type="text" name="username" placeholder="Username"><br>
<input type="password" name="password" placeholder="Password"><br>
<button name="login">Login</button>
</form>
<?php
session_start();
if($_SESSION['role']!="admin"){
    header("location:../auth/login.php");
}
?>

<h2>Dashboard Admin</h2>
<a href="materi.php">Kelola Materi</a> |
<a href="nilai.php">Input Nilai</a> |
<a href="../auth/logout.php">Logout</a>
<?php
include '../config/koneksi.php';

if(isset($_POST['upload'])){
    $judul = $_POST['judul'];
    $file = $_FILES['file']['name'];
    move_uploaded_file($_FILES['file']['tmp_name'], "../uploads/".$file);

    mysqli_query($conn,"INSERT INTO materi VALUES('', '$judul', '', '$file')");
}
?>

<form method="post" enctype="multipart/form-data">
<h3>Upload Materi</h3>
<input type="text" name="judul" placeholder="Judul"><br>
<input type="file" name="file"><br>
<button name="upload">Upload</button>
</form>

<h3>Daftar Materi</h3>
<?php
$data = mysqli_query($conn,"SELECT * FROM materi");
while($d = mysqli_fetch_array($data)){
    echo $d['judul']." - <a href='../uploads/".$d['file']."'>Download</a><br>";
}
?>
<?php
session_start();
include '../config/koneksi.php';

if(isset($_POST['kirim'])){
    $siswa = $_SESSION['username'];
    $file = $_FILES['file']['name'];
    move_uploaded_file($_FILES['file']['tmp_name'], "../uploads/".$file);

    mysqli_query($conn,"INSERT INTO tugas VALUES('', '$siswa', '$file', '0')");
}
?>

<form method="post" enctype="multipart/form-data">
<h3>Upload Tugas</h3>
<input type="file" name="file"><br>
<button name="kirim">Kirim</button>
</form>
<?php
include '../config/koneksi.php';

if(isset($_POST['nilai'])){
    $id = $_POST['id'];
    $nilai = $_POST['nilai'];

    mysqli_query($conn,"UPDATE tugas SET nilai='$nilai' WHERE id='$id'");
}

$data = mysqli_query($conn,"SELECT * FROM tugas");
while($d = mysqli_fetch_array($data)){
?>
<form method="post">
    <?= $d['siswa']; ?> -
    <a href="../uploads/<?= $d['file']; ?>">File</a>
    <input type="hidden" name="id" value="<?= $d['id']; ?>">
    <input type="number" name="nilai">
    <button name="nilai">Simpan</button>
</form>
<?php } ?>
<?php
session_start();
session_destroy();
header("location:login.php");
?>
