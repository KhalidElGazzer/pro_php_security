# **Preventing SQL Injection in PHP** 

SQL injection happens when an attacker manipulates user input to execute malicious database queries. To prevent this, PHP provides multiple techniques to sanitize input and secure queries.
1. **Use Prepared Statements (Best Practice)**

Using prepared statements with **PDO** ensures user input is treated as data, not executable SQL code.

**Example: Secure Query with PDO**
```php
<?php
$pdo = new PDO("mysql:host=localhost;dbname=testdb", "username", "password");
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$variety = $_GET['variety']; // User input

$stmt = $pdo->prepare("SELECT * FROM wines WHERE variety = ?");
$stmt->execute([$variety]);

$result = $stmt->fetchAll();
print_r($result);
?>
```    
✅ Why?

The `?` placeholder ensures the input is not executed as part of the SQL statement.
Prevents SQL injection even if the user inputs `lol' OR '1'='1`.

**2. Properly Format SQL Values**

When manually constructing queries (not recommended), always:

Enclose string values in single quotes (').
Validate numeric values to prevent syntax errors.

**Example: Unsafe Query**
```php
<?php
$variety = $_GET['variety'];
$query = "SELECT * FROM wines WHERE variety = $variety"; // ❌ Vulnerable to SQL Injection
?>
```
If a user enters lol' OR '1'='1, the query becomes:
```
SELECT * FROM wines WHERE variety = 'lol' OR '1'='1';
```
This grants access to all records! 
**Fixed Version**
```php
<?php
$variety = $_GET['variety'];
$query = "SELECT * FROM wines WHERE variety = '" . addslashes($variety) . "'"; //Still not the best solution
?>
```
⚠️ Warning: `addslashes()` is not enough. Use prepared statements instead.

>> Do not use the `magic_quotes_gpc` directive or its behind-the-scenes partner, the
`addslashes()` function, which is limited in its application, and requires the
additional step of the `stripslashes()` function.

**3. Validate User Input**

Before sending user input to the database, check:

* Data types (is_int(), gettype(), intval())
* String length (strlen())
* Date format (strtotime())

Example: Validate Numeric Input
```php
<?php
$vintage = $_GET['vintage'];

if (!is_numeric($vintage)) {
    exit("Invalid input!");
}

$query = "SELECT * FROM wines WHERE vintage = " . intval($vintage); // Now it's safe
?>
```
Why?

Ensures only numbers are allowed.
Prevents injection via input like `"2000; DROP TABLE wines;"` => the intval will output only 2000 and the rest of the query will be ignored.

**4. Escape Special Characters (If Not Using Prepared Statements)**

If prepared statements are not possible, escape input using `mysqli_real_escape_string()`.

**Example: Secure Query with Escaping**

```php
$connection = new mysqli("localhost", "username", "password", "database");

// User input
$input = "O'Reilly";

// Escape the input
$escaped_input = mysqli_real_escape_string($connection, $input);

// Use in a query
$query = "SELECT * FROM users WHERE name = '$escaped_input'";
echo $query;
```
Why?

Prevents SQL injection by escaping special characters like `'`.


>> `mysqli_real_escape_string()` escapes special characters in a string, making it safe to include in an SQL query.

It adds a backslash (\) before the following characters:

* Single quote (')
* Double quote (")
* Backslash (\)
* NUL (the \0 character)
* Newline (\n)
* Carriage return (\r)


**5. Abstract Validation into Functions**

To keep code clean and consistent, create a function to sanitize input.
Example: Safe Function

```php
<?php
function safeInput($string, $conn) {
    return "'" . mysqli_real_escape_string($conn, $string) . "'";
}

$conn = new mysqli("localhost", "username", "password", "testdb");
$variety = safeInput($_GET['variety'], $conn);

$query = "SELECT * FROM wines WHERE variety = $variety";
$result = $conn->query($query);
?>
```
Why?

Centralizes sanitization, making it easier to maintain.

**6. Retrofitting Legacy Applications**
For old applications, implement a wrapper function around database queries.

**Example: Retrofitting with a Secure Query Function**
```php
<?php
function executeSafeQuery($pdo, $sql, $params) {
    $stmt = $pdo->prepare($sql);
    $stmt->execute($params);
    return $stmt->fetchAll();
}

$pdo = new PDO("mysql:host=localhost;dbname=testdb", "username", "password");
$variety = $_GET['variety'];

$result = executeSafeQuery($pdo, "SELECT * FROM wines WHERE variety = ?", [$variety]);
print_r($result);
?>
```
Why?

Easy to apply across multiple queries.
Reduces risk of missing sanitization in different parts of the code.









>> Note

If ur forced to choose between `mysql_real_escape_string()` and prepared statements using PDO extension, you must choose prepared statements cuz `The mysql_real_escape_string()` and `mysql_query()` functions are deprecated as of PHP 5.5.0 and removed in PHP 7.0.0, Using these functions is not recommended because they are no longer supported and are inherently insecure.










