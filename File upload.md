

## 1. Unrestricted File Upload

### Vulnerable Code
```php
if(isset($_FILES['file'])){
    move_uploaded_file($_FILES['file']['tmp_name'], "uploads/" . $_FILES['file']['name']);
    echo "File uploaded successfully!";
}
```

### Issues
- No extension filtering, allowing any file type.
- No MIME type verification.
- No file size check.

### Secure Fix
```php
$allowed_ext = ['jpg', 'png', 'jpeg'];
$file_name = $_FILES['file']['name'];
$file_ext = strtolower(pathinfo($file_name, PATHINFO_EXTENSION));

if (in_array($file_ext, $allowed_ext)) {
    $safe_name = md5(uniqid()) . "." . $file_ext;  
    move_uploaded_file($_FILES['file']['tmp_name'], "uploads/" . $safe_name);
    echo "File uploaded successfully!";
} else {
    echo "Invalid file type!";
}
```

## 2. Double Extension Bypass

### Vulnerable Code
```php
if(isset($_FILES['file'])){
    $ext = end(explode('.', $_FILES['file']['name']));
    if($ext == "jpg" || $ext == "png") {
        move_uploaded_file($_FILES['file']['tmp_name'], "uploads/" . $_FILES['file']['name']);
    } else {
        echo "Invalid file type!";
    }
}
```

### Issues
- Allows `shell.php.jpg` upload.

### Secure Fix
```php
$file_ext = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));
if (in_array($file_ext, $allowed_ext)) {
    $safe_name = md5(uniqid()) . "." . $file_ext;  
    move_uploaded_file($_FILES['file']['tmp_name'], "uploads/" . $safe_name);
    echo "File uploaded successfully!";
} else {
    echo "Invalid file type!";
}
```

## 3. MIME Type Bypass

### Vulnerable Code
```php
if($_FILES['file']['type'] == "image/png" || $_FILES['file']['type'] == "image/jpeg") {
    move_uploaded_file($_FILES['file']['tmp_name'], "uploads/" . $_FILES['file']['name']);
}
```

### Issues
- MIME types can be spoofed.

### Secure Fix
```php
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['file']['tmp_name']);   // finfo_file() reads the actual file content and detects its true MIME type
finfo_close($finfo);

$allowed_mime = ['image/jpeg', 'image/png'];

if (in_array($mime, $allowed_mime)) {
    $safe_name = md5(uniqid()) . ".jpg";
    move_uploaded_file($_FILES['file']['tmp_name'], "uploads/" . $safe_name);
    echo "File uploaded!";
} else {
    echo "Invalid file type!";
}
```

## 4. PHP Execution in Uploads

### Vulnerable Code
```php
move_uploaded_file($_FILES['file']['tmp_name'], "uploads/" . $_FILES['file']['name']);
```

### Issues
- Allows execution of uploaded PHP files.

### Secure Fix
#### **1. Disable PHP Execution in Uploads Folder**
Create a `.htaccess` file in `uploads/`:
```apache
<FilesMatch "\.php$">
    Deny from all
</FilesMatch>
```

#### **2. Store Uploads Outside Web Root**
```php
$upload_dir = __DIR__ . '/../secure_uploads/';
move_uploaded_file($_FILES['file']['tmp_name'], $upload_dir . $safe_name);
```

## 5. Directory Traversal

### Vulnerable Code
```php
$target = "uploads/" . $_POST['filename'];
unlink($target);
```

### Issues
- Attackers can delete critical files using `../../index.php`.

### Secure Fix
```php
$target = realpath("uploads/" . $_POST['filename']);
$uploads_dir = realpath("uploads/");

if (strpos($target, $uploads_dir) === 0 && file_exists($target)) {
    unlink($target);
} else {
    echo "Invalid file!";
}
```


## 6. Lack of CSRF Protection

### Vulnerable Code

```php
if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $target_dir = "uploads/";
    $target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);
    move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file);
}
```
### Issues

- The code is vulnerable to Cross-Site Request Forgery (CSRF) attacks, allowing an attacker to upload files on behalf of a user.

### Secure Fix

- Implement CSRF protection using tokens.
```php
session_start();

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    if (!isset($_POST["csrf_token"]) || $_POST["csrf_token"] !== $_SESSION["csrf_token"]) {
        die("CSRF token validation failed.");
    }

    $target_dir = "uploads/";
    $target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);
    move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file);
}

// Generate CSRF token
$_SESSION["csrf_token"] = bin2hex(random_bytes(32));
```


## Final Secure File Upload Code
```php
$allowed_ext = ['jpg', 'jpeg', 'png'];
$upload_dir = __DIR__ . '/../secure_uploads/';

$file_name = $_FILES['file']['name'];
$file_tmp = $_FILES['file']['tmp_name'];
$file_ext = strtolower(pathinfo($file_name, PATHINFO_EXTENSION));

$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $file_tmp);
finfo_close($finfo);

$allowed_mime = ['image/jpeg', 'image/png'];

if (in_array($file_ext, $allowed_ext) && in_array($mime, $allowed_mime)) {
    $safe_name = md5(uniqid()) . "." . $file_ext;
    move_uploaded_file($file_tmp, $upload_dir . $safe_name);
    echo "File uploaded!";
} else {
    echo "Invalid file!";
}
```

## Key Security Fixes
- Sanitizes file names using `md5(uniqid())`.
- Validates file extension with `pathinfo()`.
- Verifies MIME type using `finfo_file()`.
- Moves uploaded files outside the web root.
- Disables PHP execution in the uploads folder.
- Uses `realpath()` to prevent directory traversal.

