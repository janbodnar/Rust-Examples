# Reqwest


## Basic GET request

Simple synchronous HTTP GET request with error handling. This example  
demonstrates the most basic usage of reqwest's blocking client for  
making HTTP requests.

```rust
use reqwest::blocking::Client;
use reqwest::Error;

fn main() -> Result<(), Error> {
    let client = Client::new();
    let response = client.get("https://webcode.me").send()?;
    
    println!("Status: {}", response.status());
    println!("Content: {}", response.text()?);
    
    Ok(())
}
```

The blocking client provides a synchronous interface that's simple to use  
in traditional applications. The send() method executes the request and  
returns a Response object. The text() method reads the response body as  
a UTF-8 string. Both operations can fail, so we use the ? operator to  
propagate errors.


## Basic GET request async

Asynchronous HTTP GET request using tokio runtime. Async operations  
allow handling multiple requests concurrently without blocking threads.

```rust
use reqwest::Client;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();
    let response = client.get("https://webcode.me").send().await?;
    
    println!("Status: {}", response.status());
    println!("Content: {}", response.text().await?);
    
    Ok(())
}
```

The async version uses await at each async operation point. This allows  
other tasks to run while waiting for network I/O. The Client is reusable  
across multiple requests and handles connection pooling automatically.  
Tokio provides the async runtime needed to execute async functions.


## POST request with JSON

Sending JSON data in a POST request with proper headers. This pattern  
is common for REST API interactions where you need to send structured  
data to the server.

```rust
use reqwest::Client;
use serde_json::json;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();
    let json_data = json!({
        "name": "John Doe",
        "email": "john@example.com",
        "age": 30
    });
    
    let response = client
        .post("https://httpbin.org/post")
        .json(&json_data)
        .send()
        .await?;
    
    println!("Status: {}", response.status());
    println!("Response: {}", response.text().await?);
    
    Ok(())
}
```

The json() method automatically sets the Content-Type header to  
application/json and serializes the data. The serde_json::json! macro  
provides a convenient way to create JSON objects. HTTPBin.org is a  
testing service that echoes back the request data for verification.


## POST request with form data

Submitting form data using application/x-www-form-urlencoded encoding.  
This is the traditional HTML form submission format used by web browsers.

```rust
use reqwest::Client;
use std::collections::HashMap;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();
    let mut form_data = HashMap::new();
    form_data.insert("username", "johndoe");
    form_data.insert("password", "secret123");
    form_data.insert("remember_me", "on");
    
    let response = client
        .post("https://httpbin.org/post")
        .form(&form_data)
        .send()
        .await?;
    
    println!("Status: {}", response.status());
    println!("Response: {}", response.text().await?);
    
    Ok(())
}
```

The form() method automatically sets the Content-Type to  
application/x-www-form-urlencoded and URL-encodes the key-value pairs.  
HashMap provides a convenient way to structure form data. This format  
is widely supported by web servers and APIs for simple data submission.


## Custom headers

Adding custom headers to HTTP requests for authentication, content type,  
or other metadata. Headers provide additional context and control over  
request processing.

```rust
use reqwest::Client;
use reqwest::header::{HeaderMap, HeaderValue, AUTHORIZATION, USER_AGENT};

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();
    let mut headers = HeaderMap::new();
    headers.insert(AUTHORIZATION, HeaderValue::from_static("Bearer token123"));
    headers.insert(USER_AGENT, HeaderValue::from_static("MyApp/1.0"));
    headers.insert("X-Custom-Header", HeaderValue::from_static("custom-value"));
    
    let response = client
        .get("https://httpbin.org/headers")
        .headers(headers)
        .send()
        .await?;
    
    println!("Status: {}", response.status());
    println!("Response: {}", response.text().await?);
    
    Ok(())
}
```

HeaderMap stores HTTP headers with type-safe values. Standard headers  
like AUTHORIZATION and USER_AGENT are provided as constants. Custom  
headers can be added with string names. HeaderValue ensures headers  
contain only valid ASCII characters as required by HTTP specifications.


## Authentication with Bearer token

Implementing Bearer token authentication for API access. This is the  
standard authentication method for OAuth 2.0 and many modern APIs.

```rust
use reqwest::Client;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();
    let token = "your_api_token_here";
    
    let response = client
        .get("https://httpbin.org/bearer")
        .bearer_auth(token)
        .send()
        .await?;
    
    match response.status().as_u16() {
        200 => println!("Authentication successful"),
        401 => println!("Authentication failed - invalid token"),
        _ => println!("Unexpected status: {}", response.status()),
    }
    
    println!("Response: {}", response.text().await?);
    
    Ok(())
}
```

The bearer_auth() method is a convenience function that adds the  
Authorization header with "Bearer " prefix. Different status codes  
indicate different authentication states. A 401 status typically means  
the token is invalid or expired, while 200 indicates successful  
authentication and authorized access.


## Timeout configuration

Setting request timeouts to handle slow or unresponsive servers. Timeouts  
prevent applications from hanging indefinitely when network issues occur.

```rust
use reqwest::Client;
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::builder()
        .timeout(Duration::from_secs(10))
        .connect_timeout(Duration::from_secs(5))
        .build()?;
    
    match client.get("https://httpbin.org/delay/3").send().await {
        Ok(response) => {
            println!("Status: {}", response.status());
            println!("Request completed within timeout");
        }
        Err(e) if e.is_timeout() => {
            println!("Request timed out");
        }
        Err(e) => {
            println!("Other error: {}", e);
        }
    }
    
    Ok(())
}
```

The Client::builder() provides a fluent interface for configuration.  
timeout() sets the overall request timeout, while connect_timeout() sets  
the connection establishment timeout. The is_timeout() method helps  
distinguish timeout errors from other network errors for appropriate  
error handling.


## Response status handling

Comprehensive HTTP status code handling with different response patterns.  
Proper status code handling is essential for robust HTTP client  
implementations.

```rust
use reqwest::{Client, StatusCode};

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();
    let response = client.get("https://httpbin.org/status/404").send().await?;
    
    match response.status() {
        StatusCode::OK => {
            println!("Success! Response: {}", response.text().await?);
        }
        StatusCode::NOT_FOUND => {
            println!("Resource not found (404)");
        }
        StatusCode::UNAUTHORIZED => {
            println!("Authentication required (401)");
        }
        StatusCode::INTERNAL_SERVER_ERROR => {
            println!("Server error (500)");
        }
        status if status.is_success() => {
            println!("Success with status: {}", status);
        }
        status if status.is_client_error() => {
            println!("Client error: {}", status);
        }
        status if status.is_server_error() => {
            println!("Server error: {}", status);
        }
        _ => {
            println!("Unexpected status: {}", response.status());
        }
    }
    
    Ok(())
}
```

StatusCode provides constants for common HTTP status codes and helper  
methods for status categories. The is_success(), is_client_error(), and  
is_server_error() methods help categorize responses. HTTPBin's /status  
endpoint returns the specified status code for testing purposes.


## File download

Downloading files with progress tracking and proper error handling.  
This example shows how to download large files efficiently without  
loading everything into memory.

```rust
use reqwest::Client;
use std::fs::File;
use std::io::Write;
use tokio_stream::StreamExt;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();
    let url = "https://httpbin.org/bytes/1024";
    
    let response = client.get(url).send().await?;
    
    if !response.status().is_success() {
        println!("Failed to download: {}", response.status());
        return Ok(());
    }
    
    let mut file = File::create("downloaded_file.bin")?;
    let mut stream = response.bytes_stream();
    let mut downloaded = 0;
    
    while let Some(chunk) = stream.next().await {
        let chunk = chunk?;
        file.write_all(&chunk)?;
        downloaded += chunk.len();
        println!("Downloaded {} bytes", downloaded);
    }
    
    println!("Download completed: {} bytes total", downloaded);
    
    Ok(())
}
```

The bytes_stream() method returns a stream of byte chunks, allowing  
memory-efficient processing of large files. Each chunk is written  
immediately to disk rather than accumulating in memory. This approach  
scales to files of any size and provides real-time progress feedback.


## File upload

Uploading files using multipart form data. This is the standard way  
to upload files through HTTP, commonly used in web forms and APIs.

```rust
use reqwest::Client;
use reqwest::multipart::{Form, Part};
use std::fs::File;
use std::io::Read;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::new();
    
    // Read file content
    let mut file = File::open("upload_test.txt")?;
    let mut file_content = Vec::new();
    file.read_to_end(&mut file_content)?;
    
    // Create multipart form
    let file_part = Part::bytes(file_content)
        .file_name("upload_test.txt")
        .mime_str("text/plain")?;
    
    let form = Form::new()
        .text("description", "Test file upload")
        .text("user_id", "12345")
        .part("file", file_part);
    
    let response = client
        .post("https://httpbin.org/post")
        .multipart(form)
        .send()
        .await?;
    
    println!("Upload status: {}", response.status());
    println!("Response: {}", response.text().await?);
    
    Ok(())
}
```

The multipart Form allows mixing text fields with file uploads. Part  
represents individual form parts with content type and filename metadata.  
The mime_str() method sets the MIME type for proper content handling by  
the server. Text fields and file parts can be combined in a single form.


## Query parameters

Adding URL query parameters for filtering, pagination, and data selection.  
Query parameters are the standard way to pass optional data in GET  
requests.

```rust
use reqwest::Client;
use std::collections::HashMap;

#[tokio::main]
async fn main() -> Result<(), reqwest::Error> {
    let client = Client::new();
    
    // Method 1: Using query() with HashMap
    let mut params = HashMap::new();
    params.insert("page", "1");
    params.insert("limit", "10");
    params.insert("sort", "name");
    params.insert("filter", "active");
    
    let response1 = client
        .get("https://httpbin.org/get")
        .query(&params)
        .send()
        .await?;
    
    println!("Method 1 - HashMap approach:");
    println!("Final URL: {}", response1.url());
    
    // Method 2: Using query() with array of tuples
    let response2 = client
        .get("https://httpbin.org/get")
        .query(&[("search", "rust"), ("category", "programming")])
        .send()
        .await?;
    
    println!("Method 2 - Tuple array approach:");
    println!("Final URL: {}", response2.url());
    println!("Response: {}", response2.text().await?);
    
    Ok(())
}
```

The query() method accepts various parameter formats including HashMap  
and tuple arrays. Parameters are automatically URL-encoded for safety.  
The final URL shows how parameters are appended with proper encoding.  
Multiple query() calls accumulate parameters rather than replacing them.


## Cookie handling

Managing HTTP cookies for session persistence and authentication. Cookies  
maintain state across multiple requests, essential for authenticated  
sessions and user preferences.

```rust
use reqwest::{Client, cookie::Jar};
use std::sync::Arc;
use url::Url;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let cookie_jar = Arc::new(Jar::default());
    let client = Client::builder()
        .cookie_store(true)
        .cookie_provider(Arc::clone(&cookie_jar))
        .build()?;
    
    // First request - server sets cookies
    let response1 = client
        .get("https://httpbin.org/cookies/set/session_id/abc123")
        .send()
        .await?;
    
    println!("First response status: {}", response1.status());
    
    // Check cookies in the jar
    let url = Url::parse("https://httpbin.org")?;
    let cookies: Vec<_> = cookie_jar.cookies(&url)
        .unwrap_or_default()
        .to_str()
        .unwrap_or("")
        .split(';')
        .collect();
    
    println!("Stored cookies: {:?}", cookies);
    
    // Second request - cookies are automatically sent
    let response2 = client
        .get("https://httpbin.org/cookies")
        .send()
        .await?;
    
    println!("Second response: {}", response2.text().await?);
    
    Ok(())
}
```

The cookie_store(true) enables automatic cookie handling. The Jar stores  
cookies between requests according to HTTP cookie specifications. Cookies  
are automatically sent with subsequent requests to matching domains and  
paths. This enables stateful HTTP interactions without manual cookie  
management.


## Error handling patterns

Comprehensive error handling for different types of HTTP and network  
failures. Robust error handling is crucial for production HTTP clients  
that need to handle various failure scenarios.

```rust
use reqwest::{Client, Error};
use std::time::Duration;

#[tokio::main]
async fn main() {
    let client = Client::builder()
        .timeout(Duration::from_secs(5))
        .build()
        .unwrap();
    
    let urls = vec![
        "https://httpbin.org/get",           // Should succeed
        "https://httpbin.org/status/500",    // Server error
        "https://nonexistent.invalid",      // DNS failure
        "https://httpbin.org/delay/10",      // Timeout
    ];
    
    for url in urls {
        println!("\nTesting URL: {}", url);
        
        match make_request(&client, url).await {
            Ok(response) => println!("Success: {}", response),
            Err(e) => handle_error(e),
        }
    }
}

async fn make_request(client: &Client, url: &str) -> Result<String, Error> {
    let response = client.get(url).send().await?;
    
    if response.status().is_success() {
        Ok(format!("Status: {}", response.status()))
    } else {
        Ok(format!("HTTP Error: {}", response.status()))
    }
}

fn handle_error(error: Error) {
    if error.is_timeout() {
        println!("Error: Request timed out");
    } else if error.is_connect() {
        println!("Error: Failed to connect - check network/DNS");
    } else if error.is_request() {
        println!("Error: Request construction failed");
    } else if error.is_decode() {
        println!("Error: Response decoding failed");
    } else {
        println!("Error: Unknown error - {}", error);
    }
}
```

Different error types require different handling strategies. is_timeout()  
identifies timeout errors, is_connect() detects network connectivity  
issues, and is_decode() catches response parsing problems. Proper error  
classification enables appropriate retry logic and user feedback in  
production applications.


## Proxy configuration

Configuring HTTP and HTTPS proxies for network routing and security.  
Proxy support is essential in corporate environments and for traffic  
routing through specific gateways.

```rust
use reqwest::{Client, Proxy};
use std::env;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Method 1: System proxy (reads HTTP_PROXY, HTTPS_PROXY env vars)
    let client1 = Client::builder()
        .proxy(Proxy::all("http://proxy.example.com:8080")?)
        .build()?;
    
    // Method 2: HTTP/HTTPS specific proxies
    let client2 = Client::builder()
        .proxy(Proxy::http("http://proxy.example.com:8080")?)
        .proxy(Proxy::https("http://proxy.example.com:8080")?)
        .build()?;
    
    // Method 3: Authenticated proxy
    let client3 = Client::builder()
        .proxy(
            Proxy::all("http://proxy.example.com:8080")?
                .basic_auth("username", "password")
        )
        .build()?;
    
    // Method 4: Using environment variables
    if let Ok(proxy_url) = env::var("HTTP_PROXY") {
        let client4 = Client::builder()
            .proxy(Proxy::all(&proxy_url)?)
            .build()?;
        
        println!("Using proxy from environment: {}", proxy_url);
    }
    
    // Test request through proxy
    let response = client1
        .get("https://httpbin.org/ip")
        .send()
        .await?;
    
    println!("Response through proxy: {}", response.text().await?);
    
    Ok(())
}
```

Proxy::all() configures the proxy for both HTTP and HTTPS traffic.  
Protocol-specific proxies can be set separately. basic_auth() adds  
authentication credentials for proxy servers that require them. The  
HTTP_PROXY and HTTPS_PROXY environment variables are standard ways to  
configure proxy settings system-wide.


## Custom client configuration

Advanced client configuration with connection pooling, redirects, and  
security settings. Custom configuration allows fine-tuning performance  
and behavior for specific use cases.

```rust
use reqwest::{Client, redirect::Policy};
use std::time::Duration;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::builder()
        // Connection settings
        .timeout(Duration::from_secs(30))
        .connect_timeout(Duration::from_secs(10))
        .pool_idle_timeout(Duration::from_secs(60))
        .pool_max_idle_per_host(10)
        
        // Redirect policy
        .redirect(Policy::limited(5))
        
        // Security settings
        .danger_accept_invalid_certs(false)
        .danger_accept_invalid_hostnames(false)
        
        // HTTP/2 settings
        .http2_prior_knowledge()
        .http2_keep_alive_interval(Duration::from_secs(30))
        
        // User agent
        .user_agent("MyApp/1.0 (https://example.com)")
        
        // Default headers
        .default_headers({
            let mut headers = reqwest::header::HeaderMap::new();
            headers.insert("Accept", "application/json".parse().unwrap());
            headers
        })
        
        .build()?;
    
    // Test the configured client
    let response = client
        .get("https://httpbin.org/user-agent")
        .send()
        .await?;
    
    println!("Status: {}", response.status());
    println!("Response: {}", response.text().await?);
    
    // Client configuration info
    println!("\nClient configured with:");
    println!("- 30s total timeout, 10s connect timeout");
    println!("- Connection pool: 10 idle connections per host");
    println!("- Maximum 5 redirects");
    println!("- HTTP/2 enabled with keep-alive");
    println!("- Custom User-Agent and default headers");
    
    Ok(())
}
```

Client::builder() provides extensive configuration options. Connection  
pooling settings control resource usage and performance. Redirect policies  
prevent infinite redirect loops. Security settings control TLS validation.  
HTTP/2 settings enable modern protocol features. Default headers are  
automatically added to all requests made with this client instance.


## HEAD request

Retrieving only response headers without downloading the body content.  
HEAD requests are useful for checking resource availability, getting  
metadata, or testing endpoints without transferring large amounts of data.

```rust
use reqwest::blocking::Client;
use reqwest::Error;

fn main() -> Result<(), Error> {
    let client = Client::new();
    let response = client.head("https://webcode.me").send()?;

    println!("Status: {}", response.status());

    for (key, value) in response.headers() {
        println!("Header: {}: {}", key, value.to_str().unwrap_or("<invalid UTF-8>"));
    }

    Ok(())
}
```

HEAD requests return the same headers as GET requests but without the  
response body. This is efficient for checking file sizes, last-modified  
dates, or content types without downloading entire files. The response  
headers contain metadata about the resource including content length,  
content type, and caching information.


## Async fetch title

Extracting webpage titles using HTML parsing with async HTTP requests.  
This example combines HTTP fetching with HTML parsing to extract specific  
content from web pages efficiently.

```rust
use reqwest::get;
use scraper::{Html, Selector};

#[tokio::main]  // Add tokio runtime for async main
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // URL to fetch
    let url = "https://webcode.me";

    // Send GET request (await the async call)
    let response = get(url).await?.text().await?;

    // Parse the HTML
    let document = Html::parse_document(&response);
    let selector = Selector::parse("title").unwrap();

    // Extract and print the title
    if let Some(element) = document.select(&selector).next() {
        let title = element.text().collect::<Vec<_>>().join("");
        println!("Page title: {}", title);
    } else {
        println!("No title found");
    }

    Ok(())
}
```

The scraper crate provides CSS selector-based HTML parsing. The  
Html::parse_document() function creates a DOM tree from the response text.  
Selector::parse() compiles CSS selectors for element matching. The  
select() method returns an iterator of matching elements, enabling  
extraction of specific content from complex HTML documents.
