# REST

REST, or Representational State Transfer, is an architectural style for designing networked applications. It relies on a stateless, client-server, cacheable communications protocol -- the HTTP protocol is almost always used. In a typical REST architecture:

## Client-Server Interaction
A client sends a request to the server, which responds with a representation of the requested resource.

## Resources
A resource can be almost any informational object, such as a database entry, a document, or even a service. The representation of the resource is usually a formatted document (often in XML or JSON) that acts as a snapshot of its current or requested state.

## URLs and Verbs
REST resources are identified using meaningful URLs. These URLs accept different HTTP request "verbs" such as:

- **GET**: Retrieve data safely (idempotent, meaning it doesn’t change anything).
- **POST**: Create new data.
- **PUT**: Update existing data.
- **DELETE**: Remove data.

These verbs are analogous to the CRUD (Create, Retrieve, Update, Delete) model.

## Response
A RESTful service typically provides two components in its responses:

- **Response Body**: The actual data, often in JSON or XML format.
- **Status Code**: An HTTP status code indicating the success or failure of the request. For example:
  - `200 OK`: Successful request.
  - `201 CREATED`: Successfully created a resource.
  - `400 BAD REQUEST`: Client-side error.
  - `500 INTERNAL SERVER ERROR`: Server-side error.
  - `503 SERVICE UNAVAILABLE`: Server is temporarily unavailable.

## Response Formats
Many REST servers allow clients to specify the response format (e.g., JSON, XML) using an `ACCEPT` header or a file extension (e.g., `/api/users.json`). Some servers have hard-coded formats, which are acceptable as long as they are well-documented.

# JSON

JSON (JavaScript Object Notation) is a lightweight, text-based data interchange format that is easy for both humans and machines to read and write. It is widely used in RESTful services due to its simplicity and compatibility with many programming languages.

## Key Features of JSON

### Data Types
JSON supports various data types, including:

- **Strings**: `{ "name": "John" }`
- **Numbers**: `{ "age": 30 }`
- **Booleans**: `{ "isStudent": true }`
- **Arrays**: `{ "hobbies": ["reading", "traveling"] }`
- **Objects**: `{ "address": { "city": "New York", "zip": "10001" } }`
- **Null**: `{ "middleName": null }`

### Structure
JSON data is structured as key-value pairs, similar to a dictionary or hash map. For example:

```json
{
  "firstname": "Tom",
  "lastname": "Smith",
  "age": 40
}
```

### Usage in REST
JSON is often used as the response format in REST APIs because it is lightweight and easy to parse. For example, a PHP array can be easily converted to JSON using `json_encode`:

```php
$data = array(
  'firstname' => 'Tom',
  'lastname' => 'Smith',
  'age' => 40
);
echo json_encode($data);
```

Output:

```json
{
  "firstname": "Tom",
  "lastname": "Smith",
  "age": 40
}
```

### Parsing JSON
JSON can be decoded back into a data structure in most programming languages. For example, in PHP, you can use `json_decode` to convert a JSON string into an array or object.

# REST Security

REST (Representational State Transfer) is a lightweight and flexible approach to building APIs, but its simplicity can also introduce security challenges. Since REST relies on HTTP, it inherits the security limitations of the underlying protocol. To secure RESTful services, you need to implement additional measures to protect resources, authenticate users, and ensure data integrity.

## 1. Restricting Access to Resources and Formats

- **Limit Exposure**: Restrict the amount of data and the variety of formats returned by your API. Fewer options make the API easier to test, debug, and maintain.
- **Data Limitation**: Avoid returning all records in a single request (e.g., use pagination or limit results to the most recent 500 entries).
- **Input Validation**: Validate and sanitize all user inputs to prevent injection attacks.
  - Ensure integers are within expected ranges.
  - Use database escape functions like `mysql_real_escape_string()`.
  - Reject invalid inputs with appropriate error codes (e.g., `404 Not Found`).

## 2. Authenticating and Authorizing RESTful Requests

- **API Keys**: Require developers to register for an API key and include it in requests:

  ```http
  GET /properties/list?apiKey=123456
  ```

- **HMAC Authentication**: Use HMAC (Hash-based Message Authentication Code) for stronger security.

  **Client-Side Example:**
  ```php
  $time = time() + 5*60; // UNIX timestamp plus a few minutes
  $apikey = '1839390183ABC38910';
  $hash = hash_hmac('ripemd160', $time, $apikey);
  ```

  **Server-Side Validation:**
  ```php
  $time = $_GET['time'];
  $now = time();
  $apikey = // Retrieve from database
  $hash = $_GET['hash'];
  $myhash = hash_hmac('ripemd160', $time, $apikey);

  if ($myhash == $hash && $now <= $time) {
      // Request is valid
  }
  ```

## 3. Enforcing Quotas and Rate Limits

- **Prevent Abuse**: Limit the number of requests a user can make within a specific time frame to protect your resources from overuse or Denial of Service (DoS) attacks.

- **Implementation**:
  - Track requests in a database table with the API key and timestamp.
  - Check the number of requests made by a user within the allowed time frame.
  - Return an error (e.g., `429 Too Many Requests`) if the limit is exceeded.

  ```sql
  CREATE TABLE api_requests (
      id INT AUTO_INCREMENT PRIMARY KEY,
      api_key VARCHAR(255),
      request_time TIMESTAMP
  );
  ```

  **Use a cron job to periodically purge old records:**
  ```bash
  0 * * * * /path/to/script/purge_old_requests.php
  ```

- **Caching**: Cache responses for a short period to reduce database load and improve performance:
  ```php
  $cachefile = 'cache/'.$apikey.$path.'.json';
  $cachetime = 5*60; // 5 minutes
  if (file_exists($cachefile) && time() - $cachetime < filemtime($cachefile)) {
      // Serve cached response
  } else {
      // Generate new response and cache it
      ob_start();
      // Query database and generate response
      $fp = fopen($cachefile, 'w');
      fwrite($fp, ob_get_contents());
      fclose($fp);
      ob_end_flush();
  }
  ```

## 4. Using SSL to Encrypt Communications

- **Encrypt Data in Transit**: Use HTTPS (SSL/TLS) to encrypt all communication between the client and server. This prevents eavesdropping, tampering, and man-in-the-middle attacks.




# A Basic REST Server in PHP

What follows is a very basic REST server that you can use to create basic services. It is meant as a starter
for you to use, something you can build other functionality on top of (for example, any security
measures). First, let’s take a look at the two classes we want to use for our implementation, and then
we’ll walk through it.

```php
<?php

class RestUtilities {
    public static function processRequest() {
        // Determine the HTTP method
        $req_method = strtolower($_SERVER['REQUEST_METHOD']);
        $obj = new RestRequest(); // Create a new RestRequest object
        $data = array();

        // Handle request data based on the HTTP method
        switch ($req_method) {
            case 'get':
                $data = $_GET; // Use $_GET for GET requests
                break;
            case 'post':
                $data = $_POST; // Use $_POST for POST requests
                break;
            case 'put':
                // Parse raw input for PUT requests
                parse_str(file_get_contents('php://input'), $put_vars);
                $data = $put_vars;
                break;
            default:
                die(); // Unsupported method
                break;
        }

        // Set the method and request variables in the RestRequest object
        $obj->setMethod($req_method);
        $obj->setRequestVars($data);

        // Decode JSON data if present
        if (isset($data['data'])) {
            $obj->setData(json_decode($data['data']));
        }

        return $obj; // Return the RestRequest object
    }

    public static function sendResponse($status = 200, $body = '', $content_type = 'text/html') {
        // Set the HTTP status header
        $status_header = 'HTTP/1.1 ' . $status . ' ' . RestUtilities::getStatusCodeMessage($status);
        header($status_header);
        header('Content-type: ' . $content_type); // Set the content type
        header('Content-length: ' . strlen($body)); // Set the content length

        // Send the response body if provided
        if ($body != '') {
            echo $body;
            exit;
        } else {
            // Generate an error page if no response body is provided
            $msg = '';
            switch ($status) {
                case 401:
                    $msg = 'You must be authorized to view this page.';
                    break;
                case 404:
                    $msg = 'The requested URL was not found.';
                    break;
                case 500:
                    $msg = 'The server encountered an error processing your request.';
                    break;
                case 501:
                    $msg = 'The requested method is not implemented.';
                    break;
            }

            // Create an HTML error page
            $body = '<html><head>
                     <title>' . $status . ' ' . RestUtilities::getStatusCodeMessage($status) . '</title>
                     </head>
                     <body>
                     <h1>' . RestUtilities::getStatusCodeMessage($status) . '</h1>
                     <p>' . $msg . '</p>
                     </body></html>';

            echo $body;
            exit;
        }
    }

    public static function getStatusCodeMessage($status) {
        // Map HTTP status codes to their corresponding messages
        $codes = array(
            200 => 'OK',
            201 => 'Created',
            202 => 'Accepted',
            204 => 'No Content',
            301 => 'Moved Permanently',
            302 => 'Found',
            303 => 'See Other',
            304 => 'Not Modified',
            305 => 'Use Proxy',
            306 => '(Unused)',
            307 => 'Temporary Redirect',
            400 => 'Bad Request',
            401 => 'Unauthorized',
            402 => 'Payment Required',
            403 => 'Forbidden',
            404 => 'Not Found',
            500 => 'Internal Server Error',
            501 => 'Not Implemented',
            502 => 'Bad Gateway',
            503 => 'Service Unavailable',
            504 => 'Gateway Timeout',
            505 => 'HTTP Version Not Supported'
        );

        return (isset($codes[$status])) ? $codes[$status] : '';
    }
}

class RestRequest {
    private $request_vars; // Request parameters
    private $data; // Raw data (e.g., JSON)
    private $http_accept; // Preferred response format (json or xml)
    private $method; // HTTP method (get, post, put, etc.)

    public function __construct() {
        $this->request_vars = array();
        $this->data = '';
        $this->http_accept = (strpos($_SERVER['HTTP_ACCEPT'], 'json')) ? 'json' : 'xml'; // Determine response format
        $this->method = 'get'; // Default method
    }

    public function setData($data) {
        $this->data = $data;
    }

    public function setMethod($method) {
        $this->method = $method;
    }

    public function setRequestVars($request_vars) {
        $this->request_vars = $request_vars;
    }

    public function getData() {
        return $this->data;
    }

    public function getMethod() {
        return $this->method;
    }

    public function getHttpAccept() {
        return $this->http_accept;
    }

    public function getRequestVars() {
        return $this->request_vars;
    }
}

$data = RestUtilities::processRequest();
switch($data->getMethod){
case 'get':
//retrieve a listing of properties from the database
$properties = $db->getPropertiesList(); //assume an array is returned
RestUtilities::sendResponse(200, json_encode($properties), 'application/json');
break;
}
```


