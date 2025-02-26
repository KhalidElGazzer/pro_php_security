

## 1. Vulnerable Code: Session Fixation

### Vulnerable Code:

```php
<?php
// Start the session
session_start();

// Accept session ID from URL (vulnerable to fixation)
if (isset($_GET['PHPSESSID'])) {
    session_id($_GET['PHPSESSID']);
}

// Simulate user login
if (isset($_POST['username']) && isset($_POST['password'])) {
    // Authenticate user
    $_SESSION['authenticated'] = true;
    $_SESSION['username'] = $_POST['username'];
}
?>
```

### Vulnerability:
- The script accepts a session ID from the URL ($_GET['PHPSESSID']), making it vulnerable to session fixation.
- An attacker can force a user to use a known session ID and hijack the session after the user logs in.

### Mitigation:
- Regenerate the session ID after login.
- Disable URL-based session IDs by setting session.use_only_cookies to 1.

```php
<?php
// Force cookies-only sessions
ini_set('session.use_only_cookies', TRUE);
ini_set('session.use_trans_sid', FALSE);

// Start the session
session_start();

// Simulate user login
if (isset($_POST['username']) && isset($_POST['password'])) {
    // Authenticate user
    $_SESSION['authenticated'] = true;
    $_SESSION['username'] = $_POST['username'];

    // Regenerate session ID to prevent fixation
    session_regenerate_id(true);
}
?>
```

---

## 2. Vulnerable Code: Session Hijacking via Unencrypted Connection

### Vulnerable Code:
```php
<?php
// Start the session over HTTP (vulnerable to hijacking)
session_start();

// Store user data in session
$_SESSION['user_id'] = 123;
?>
```

### Vulnerability:
- The session ID is transmitted over HTTP, making it vulnerable to eavesdropping and session hijacking.

### Mitigation:
- Use HTTPS (SSL/TLS) to encrypt all communication between the client and server.
- Ensure the Secure flag is set on session cookies.

```php
<?php
// Force HTTPS
if ($_SERVER['HTTPS'] !== 'on') {
    header('Location: https://' . $_SERVER['HTTP_HOST'] . $_SERVER['REQUEST_URI']);
    exit();
}

// Start the session
session_start();

// Set secure cookie attributes
session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'domain' => 'example.com',
    'secure' => true,    // Only send over HTTPS
    'httponly' => true,  // Prevent JavaScript access
    'samesite' => 'Strict' // Prevent cross-site requests
]);

// Store user data in session
$_SESSION['user_id'] = 123;
?>
```

---

## 3. Vulnerable Code: No Session Timeout

### Vulnerable Code:
```php
<?php
// Start the session with no timeout (vulnerable to hijacking)
session_start();

// Store user data in session
$_SESSION['user_id'] = 123;
?>
```

### Vulnerability:
- The session remains valid indefinitely, increasing the risk of session hijacking if the session ID is compromised.

### Mitigation:
- Set a session timeout to limit the session’s lifetime.
- Use session.cookie_lifetime and session.gc_maxlifetime to control session duration.

```php
<?php
// Set session timeout to 20 minutes
ini_set('session.cookie_lifetime', 1200); // 20 minutes
ini_set('session.gc_maxlifetime', 1200);  // 20 minutes

// Start the session
session_start();

// Store user data in session
$_SESSION['user_id'] = 123;
?>
```

---

## 4. Vulnerable Code: No Session ID Regeneration

### Vulnerable Code:
```php
<?php
// Start the session without regenerating ID (vulnerable to fixation/hijacking)
session_start();

// Simulate user login
if (isset($_POST['username']) && isset($_POST['password'])) {
    // Authenticate user
    $_SESSION['authenticated'] = true;
    $_SESSION['username'] = $_POST['username'];
}
?>
```

### Vulnerability:
- The session ID is not regenerated after login, making it vulnerable to session fixation and hijacking.

### Mitigation:
- Regenerate the session ID after login or privilege changes.

```php
<?php
// Start the session
session_start();

// Simulate user login
if (isset($_POST['username']) && isset($_POST['password'])) {
    // Authenticate user
    $_SESSION['authenticated'] = true;
    $_SESSION['username'] = $_POST['username'];

    // Regenerate session ID to prevent fixation
    session_regenerate_id(true);
}
?>
```

---

## 5. Vulnerable Code: Session Data Exposure

### Vulnerable Code:
```php
<?php
// Start the session
session_start();

// Store sensitive data in session without validation
$_SESSION['credit_card'] = '1234-5678-9012-3456';
?>
```

### Vulnerability:
- Sensitive data (e.g., credit card numbers) is stored in the session without encryption or validation, making it vulnerable to data theft.

### Mitigation:
- Encrypt sensitive data before storing it in the session.
- Validate and sanitize all session data.

```php
<?php
// Start the session
session_start();

// Encrypt sensitive data before storing
$credit_card = '1234-5678-9012-3456';
$encrypted_card = openssl_encrypt($credit_card, 'AES-128-CBC', 'encryption_key');

// Store encrypted data in session
$_SESSION['credit_card'] = $encrypted_card;
?>
```

---

## 6. Vulnerable Code: No Session Integrity Check

### Vulnerable Code:
```php
<?php
// Start the session without integrity checks (vulnerable to hijacking)
session_start();

// Store user data in session
$_SESSION['user_id'] = 123;
?>
```

### Vulnerability:
- The session is not bound to the user’s IP address or user agent, making it vulnerable to hijacking.

### Mitigation:
- Bind the session to the user’s IP address or user agent.

```php
<?php
// Start the session
session_start();

// Validate session integrity
if (isset($_SESSION['user_ip']) && $_SESSION['user_ip'] !== $_SERVER['REMOTE_ADDR']) {
    session_destroy();
    die("Session hijacking detected!");
} else {
    $_SESSION['user_ip'] = $_SERVER['REMOTE_ADDR'];
}

// Store user data in session
$_SESSION['user_id'] = 123;
?>
```
```

