<?php

// Placement Cell Management System

// Project Structure:
// 1. index.php - Home Page
// 2. login.php - Admin and Student Login
// 3. register.php - Student Registration
// 4. dashboard.php - Admin Dashboard
// 5. student_dashboard.php - Student Dashboard
// 6. database.php - Database Connection
// 7. admin_functions.php - Admin Specific Functions
// 8. student_functions.php - Student Specific Functions
// 9. style.css - CSS Styles
------------------------------------------
// database.php
$host = 'localhost';
$dbname = 'placement_cell';
$username = 'root';
$password = '';

try {
    $pdo = new PDO("mysql:host=$host;dbname=$dbname", $username, $password);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (PDOException $e) {
    die("Database connection failed: " . $e->getMessage());
}
-----------------------------------------
// index.php
?>
<!DOCTYPE html>
<html>
<head>
    <title>Placement Cell Management</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>Welcome to the Placement Cell Management System</h1>
    <a href="login.php">Login</a>
    <a href="register.php">Student Registration</a>
</body>
</html>

<?php
-----------------------------------------
// login.php
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    session_start();
    $email = $_POST['email'];
    $password = $_POST['password'];

    $stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
    $stmt->execute(['email' => $email]);
    $user = $stmt->fetch();

    if ($user && password_verify($password, $user['password'])) {
        $_SESSION['user_id'] = $user['id'];
        $_SESSION['role'] = $user['role'];
        
        if ($user['role'] === 'admin') {
            header('Location: dashboard.php');
        } else {
            header('Location: student_dashboard.php');
        }
    } else {
        echo "Invalid credentials!";
    }
}
?>
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <form method="POST">
        <label>Email:</label>
        <input type="email" name="email" required>
        <label>Password:</label>
        <input type="password" name="password" required>
        <button type="submit">Login</button>
    </form>
</body>
</html>

<?php
-----------------------------------------
// register.php
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $name = $_POST['name'];
    $email = $_POST['email'];
    $password = password_hash($_POST['password'], PASSWORD_BCRYPT);

    $stmt = $pdo->prepare('INSERT INTO users (name, email, password, role) VALUES (:name, :email, :password, "student")');
    $stmt->execute(['name' => $name, 'email' => $email, 'password' => $password]);

    echo "Registration successful!";
}
?>
<!DOCTYPE html>
<html>
<head>
    <title>Student Registration</title>
</head>
<body>
    <form method="POST">
        <label>Name:</label>
        <input type="text" name="name" required>
        <label>Email:</label>
        <input type="email" name="email" required>
        <label>Password:</label>
        <input type="password" name="password" required>
        <button type="submit">Register</button>
    </form>
</body>
</html>

<?php
-----------------------------------------
// dashboard.php
session_start();
if ($_SESSION['role'] !== 'admin') {
    header('Location: login.php');
    exit;
}

echo "Welcome to Admin Dashboard!";
// List all students and placement opportunities

$stmt = $pdo->query('SELECT * FROM users WHERE role = "student"');
$students = $stmt->fetchAll();
?>
<!DOCTYPE html>
<html>
<head>
    <title>Admin Dashboard</title>
</head>
<body>
    <h2>Students List</h2>
    <ul>
        <?php foreach ($students as $student): ?>
            <li><?= htmlspecialchars($student['name']) ?> (<?= htmlspecialchars($student['email']) ?>)</li>
        <?php endforeach; ?>
    </ul>
</body>
</html>

<?php
-----------------------------------------
// student_dashboard.php
session_start();
if ($_SESSION['role'] !== 'student') {
    header('Location: login.php');
    exit;
}

echo "Welcome to Student Dashboard!";
?>
<!DOCTYPE html>
<html>
<head>
    <title>Student Dashboard</title>
</head>
<body>
    <h2>Your Dashboard</h2>
    <p>View available placement opportunities here!</p>
</body>
</html>
