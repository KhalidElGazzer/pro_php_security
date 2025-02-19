# Preventing XSS

Effective XSS prevention starts at the design stage of an application, not during final testing or after an exploit is discovered. Proper planning of user inputs, workflows, and security policies can mitigate risks significantly.

## SSL Does Not Prevent XSS
SSL encrypts data in transit but does not protect against XSS. Attackers can still inject malicious scripts that appear as legitimate markup. However, storing SSL session IDs in PHP sessions can help prevent some XSS exploits.

## Strategies for Preventing XSS
The best way to prevent XSS is to never allow user input to remain untouched. Below are some effective strategies:

### Encode HTML Entities in All Non-HTML Output
Using `htmlentities()` in PHP ensures that user input is displayed safely:

```php
function safe($value) {
    return htmlentities($value, ENT_QUOTES, 'utf-8');
}

$title = $_POST['title'];
$message = $_POST['message'];

echo "<h1>" . safe($title) . "</h1><p>" . safe($message) . "</p>";
```
>> Difference Between `htmlentities()` and `htmlspecialchars()`

Function	What it Does
`htmlspecialchars()`	Converts only a few special characters: `< > " & '.`
`htmlentities()`	Converts all characters that have an HTML entity equivalent `(e.g., © → &copy;, é → &eacute;)`.

### Sanitize All User-submitted URIs
User-submitted URIs should be validated to prevent `javascript:` or `vbscript:` injections. Use `parse_url()` to extract and verify URI components:

```php
$trustedHosts = ['example.com', 'another.example.com'];

function safeURI($value) {
    $uriParts = parse_url($value);
    global $trustedHosts;

    if (in_array($uriParts['host'], $trustedHosts)) {
        return $value;
    }
    return $value . ' [' . $uriParts['host'] . ']';
}

$uri = $_POST['uri'];
echo safeURI($uri);
```
> The `parse_url()` function in PHP is used to parse a URL and return its components as an associative array.

```
$url = "https://www.example.com:8080/path/to/page?query=123#section";
$parsed = parse_url($url);
print_r($parsed);
```
🖥 Output:

```
Array
(
    [scheme] => https
    [host] => www.example.com
    [port] => 8080
    [path] => /path/to/page
    [query] => query=123
    [fragment] => section
)
```

### Design a Private API for Sensitive Transactions
For actions like transactions, use a private API that only accepts `POST` requests. Check HTTP referrers to prevent unauthorized form submissions:

```php
if ($_SERVER['HTTP_REFERER'] != $_SERVER['HTTP_HOST']) {
    exit('Unauthorized form submission.');
}
```

### Escape All Output on Sensitive Pages
For sensitive pages, disable HTML markup in user inputs entirely to prevent injection attacks.

## Conclusion
XSS prevention requires a combination of encoding, filtering, and workflow design. By implementing these strategies early in development, you can significantly reduce the risk of XSS vulnerabilities in your application.
