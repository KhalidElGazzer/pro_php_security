htmlspecialchars() to prevent the xss attackes  -->

 <!-- $user_input = '<script>alert("Hacked!");</script>';
$safe_output = htmlspecialchars($user_input);

echo $safe_output;

🔹 Output:

&lt;script&gt;alert("Hacked!");&lt;/script&gt; -->





<!-- htmlspecialchars() Flags -->

<!-- You can use different flags to modify how characters are encoded:
Flag	Description
ENT_QUOTES =>	   Converts both single (') and double (") quotes. (Recommended for security)
ENT_NOQUOTES =>	   Leaves quotes unchanged.
ENT_HTML401	=>     Uses HTML 4.01 encoding rules.
ENT_HTML5 =>	   Uses HTML5 encoding rules.
🔹 Example: Using Flags

$text = '"Double" and \'Single\' quotes';
echo htmlspecialchars($text, ENT_QUOTES);

🔹 Output:

&quot;Double&quot; and &#039;Single&#039; quotes

    Both " and ' are converted. -->





=====================================================================================



stripslashes() => Is a function that delete the \ that could added to prevent the sql attackes while retrieving data from the database.
addslashes()   => This function adds backslashes (\) before characters that need to be escaped in SQL or other contexts, such as(', ", \).
strip_tags()   => This function removes all HTML and PHP tags from the string ("<script>hi</script>" => hi).






=====================================================================================

**Some examples:**
--
1.
vulnerable code: 
```
<?php
if (isset($_POST['comment'])) {
    $comment = $_POST['comment'];
    file_put_contents("comments.txt", $comment . PHP_EOL, FILE_APPEND);
}

$comments = file("comments.txt", FILE_IGNORE_NEW_LINES);
foreach ($comments as $comment) {
    echo "<p>$comment</p>";
}
?>
```
Here simple payload will be fired. Like ```<script>alert('Stored XSS')</script>```

**mitigation**

```
<?php
if (isset($_POST['comment'])) {
    $comment = htmlspecialchars($_POST['comment'], ENT_QUOTES, 'UTF-8');
    file_put_contents("comments.txt", $comment . PHP_EOL, FILE_APPEND);
}

$comments = file("comments.txt", FILE_IGNORE_NEW_LINES);
foreach ($comments as $comment) {
    echo "<p>$comment</p>";
}
?>

```
2.
vulnerable
```
<?php
header('Content-Type: application/json');
$data = [
    "message" => $_GET['msg']
];
echo json_encode($data);
?>
   ```
Payload:
```
http://example.com/?msg="><script>alert(1)</script>
```


**mitigation**
```
<?php
header('Content-Type: application/json');
$data = [
    "message" => htmlspecialchars($_GET['msg'], ENT_QUOTES, 'UTF-8')
];
echo json_encode($data);
?>

```
3-
vulnerable code:
```
<?php
header('Content-Type: application/json');
echo json_encode(["message" => $_GET['msg']]);
?>
```
mitigation
```
<?php
header('Content-Type: application/json');
echo json_encode(["message" => htmlspecialchars($_GET['msg'], ENT_QUOTES, 'UTF-8')]);
?>
```

4-
```
<?php
$conn = new mysqli("localhost", "root", "", "test");
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $comment = $_POST["comment"];
    $conn->query("INSERT INTO comments (text) VALUES ('$comment')");
}

$result = $conn->query("SELECT * FROM comments");
while ($row = $result->fetch_assoc()) {
    echo "<p>" . $row["text"] . "</p>";
}
?>
```
mitigation
```
<?php
$conn = new mysqli("localhost", "root", "", "test");
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $comment = htmlspecialchars($_POST["comment"], ENT_QUOTES, 'UTF-8');
    $stmt = $conn->prepare("INSERT INTO comments (text) VALUES (?)");
    $stmt->bind_param("s", $comment);
    $stmt->execute();
}

$result = $conn->query("SELECT * FROM comments");
while ($row = $result->fetch_assoc()) {
    echo "<p>" . htmlspecialchars($row["text"], ENT_QUOTES, 'UTF-8') . "</p>";
}
?>
```































