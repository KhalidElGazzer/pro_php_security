# Visibility Risks

Temporary files can inadvertently expose private data to the public or malicious actors. Attackers with shell or FTP access can exploit files with revealing names, such as `2011_Confidential_Sales_Strategies.tmp` or session files like `sess_95971078f4822605e7a18c612054f658`. Even without direct access, attackers can manipulate URIs to access previously checked files. For example, if a URI like `http://bad.example.com/spellcheck.php?tmp_file=spellcheck46` is used to access spellchecking output, an attacker might try `http://bad.example.com/spellcheck.php?tmp_file=spellcheck45` to access other temporary files.

# Execution Risks

Temporary files are not meant to be executable, but attackers can upload malicious scripts to temporary locations. For instance, a file containing `<?php exec('rm /*.*'); ?>` could be executed, potentially causing catastrophic damage by deleting files or executing arbitrary commands.

# Hijacking Risks

Attackers can hijack temporary files by replacing or appending data, leading to:

* Exposure of Confidential Data: Attackers might access sensitive files like the system password file.

* Data Erasure or Corruption: Files could be deleted or corrupted, preventing the completion of legitimate requests.

* Output Redirection: Attackers might redirect output to a location they control, potentially overwriting system files or exposing sensitive data.

Hijacking is related to race conditions, where simultaneous access to data can result in mixed or corrupted data. However, the primary security issue is the attacker’s ability to replace or modify the file, not the timing of access.




# Preventing Temporary File Abuse

**1. Make Locations Difficult:**

* Use hard-to-guess filenames and store files in default locations like `/tmp` or `/var/tmp`.

* Use PHP’s `tempnam()` function to create files with unique names, or manually create filenames with `fopen()` and `uniqid()` for greater randomness.
```php

$tempFilename = uniqid('skiResort', TRUE);
touch($tempFilename);
chmod($tempFilename, 0600);
```
This creates a file with a unique name and restricts permissions to the owner


**2. Make Permissions Restrictive:**

* Set file permissions to `600` (**read/write for owner only**) and directory permissions to `700` (**read/write/execute for owner only**).

* Use Apache configuration to block access to temporary files within the webroot:
```
    <Directory /var/www/myapp/tmp>
      <FilesMatch "\.ph(p(3|4)?|tml)$">
        order deny,allow
        deny from all
      </FilesMatch>
    </Directory>
```
This prevents Apache from serving files in the specified directory.





**3. Write to Known Files Only:**

* Verify files are empty before writing:
```php
if (filesize($tempFilename) === 0) {
      // Write to the file
} else {
      exit("$tempFilename is not empty.\nStart over again.");
    }
```
* Use checksums (e.g., `sha1_file()`) to detect tampering:
```php
$hashnow = sha1_file($tempFilename);
$_SESSION['hashnow'] = $hashnow;
```
* Before writing additional data, generate a new checksum and compare it to the stored value:
```php
$hashnow = sha1_file($tempFilename);
if ($hashnow === $_SESSION['hashnow']) {
      // Write to the file again
  $hashnow = sha1_file($tempFilename);
  $_SESSION['hashnow'] = $hashnow;
} else {
   exit("Temporary file contains unexpected contents.\nStart over again.");
}
```
**4. Read from Known Files Only:**

* Verify data integrity using checksums or signatures (e.g., OpenSSL certificates or random tokens).

* How it Works:

1. When writing the file, generate a digital signature using OpenSSL.
2. Store the signature alongside the file or in a database.
3. Before reading, verify that the stored signature matches the file’s content.






**Example: Signing & Verifying Data**
Writing & Signing the File
```php
$data = "Sensitive Information";
file_put_contents($tempFilename, $data);

// Generate digital signature
$private_key = openssl_pkey_get_private("file://private.pem");
openssl_sign($data, $signature, $private_key, OPENSSL_ALGO_SHA256);

// Store the signature in session (or database)
$_SESSION['signature'] = base64_encode($signature);
```
**Reading & Verifying the File**

```php
$data = file_get_contents($tempFilename);
$public_key = openssl_pkey_get_public("file://public.pem");
$signature = base64_decode($_SESSION['signature']);

if (openssl_verify($data, $signature, $public_key, OPENSSL_ALGO_SHA256)) {
    echo " File is authentic and safe to read!";
} else {
    exit("File integrity check failed! Possible tampering detected.");
}
```







**5. Check Uploaded Files:**

* Use `is_uploaded_file()` to validate files uploaded via HTTP POST:
```php
if (is_uploaded_file($_FILES['userfile']['tmp_name'])) {
   echo 'The file in temporary storage is the one that was uploaded.';
} else {
    echo 'There is some problem with the file in temporary storage!';
}
```


















