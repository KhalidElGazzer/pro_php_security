# Ex 1: Login Bypass (Classic SQL Injection)

**Vulnerable Code (Unsafe Login Form)**

```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=testdb", "username", "password");

$username = $_POST['username'];
$password = $_POST['password'];

$sql = "SELECT * FROM users WHERE username = '$username' AND password = '$password'";
$result = $pdo->query($sql);

if ($result->rowCount() > 0) {
    echo "Login successful!";
} else {
    echo "Invalid username or password!";
}
?>
```

If an attacker enters this as the username:
```
admin' -- 
```
The SQL query becomes:
```php
SELECT * FROM users WHERE username = 'admin' --' AND password = ''
```
The -- turns the rest of the query into a comment.
The query logs in as admin without needing a password.

**Fixed Code (Using Prepared Statements)**

```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=testdb", "username", "password");

$username = $_POST['username'];
$password = $_POST['password'];

$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->execute([$username, $password]);

if ($stmt->rowCount() > 0) {
    echo "Login successful!";
} else {
    echo "Invalid username or password!";
}
?>
```
✔ Why is this secure?

Uses placeholders (?) – Prevents direct SQL injection.
User input is treated as data, not SQL commands.


# EX 2: Vulnerable Code: Using User Input in an ORDER BY Clause

**Vulnerable Code:**
```php
<?php
$conn = new mysqli("localhost", "username", "password", "database");

// User input
$order = $_GET['order'];

// Vulnerable query
$query = "SELECT * FROM users ORDER BY $order";
$result = $conn->query($query);

// Display results
while ($row = $result->fetch_assoc()) {
    echo "User: " . $row['username'] . "<br>";
}
?>
```
Exploit:

An attacker can inject malicious SQL by passing a payload like:
```
http://example.com/?order=username; DROP TABLE users; --
```
The resulting query will be:
```
SELECT * FROM users ORDER BY username; DROP TABLE users; --
```
This will drop the users table.

**Fixed Code:**

Validate the order parameter against a whitelist of allowed columns:
```php
<?php
$conn = new mysqli("localhost", "username", "password", "database");

// User input
$order = $_GET['order'];

// Whitelist of allowed columns
$allowed_columns = ["username", "email", "created_at"];
if (!in_array($order, $allowed_columns)) {
    $order = "username"; // Default to a safe column
}

// Safe query
$query = "SELECT * FROM users ORDER BY $order";
$result = $conn->query($query);

// Display results
while ($row = $result->fetch_assoc()) {
    echo "User: " . htmlspecialchars($row['username']) . "<br>";
}

// Close the connection
$conn->close();
?>
```


# EX 3: ```mysql_real_escape_string()``` (Deprecated)

**Vulnerable Code**
```php
<?php
$pdo = mysql_connect("localhost", "username", "password");
mysql_select_db("testdb");

$username = $_GET['username'];
$query = "SELECT * FROM users WHERE username = '$username'";
$result = mysql_query($query);

while ($row = mysql_fetch_assoc($result)) {
    echo "User: " . $row['username'];
}
?>
```
Exploit

Attacker inputs:
```
admin' OR '1'='1
```
Query becomes:
```
SELECT * FROM users WHERE username = 'admin' OR '1'='1'
```
This returns all users in the database.
**Fixed Code:**
```php
<?php
$pdo = mysql_connect("localhost", "username", "password");
mysql_select_db("testdb");

$username = mysql_real_escape_string($_GET['username']); // Escaping input
$query = "SELECT * FROM users WHERE username = '$username'";
$result = mysql_query($query);

while ($row = mysql_fetch_assoc($result)) {
    echo "User: " . $row['username'];
}
?>
```
Why This Is Bad?

❌ Still allows certain injection attacks.
❌ Deprecated since PHP 7.0 (do not use this in new projects).


# Final Thoughts

Never use ```mysql_real_escape_string()``` or `addslashes()` 

Prefer mysqli or PDO prepared statements

Use a secure wrapper function for legacy applications













