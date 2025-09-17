# Networking

## QOTD

Quote of the Day protocol client that connects to a QOTD server to  
retrieve a daily quote. This demonstrates basic TCP socket programming  
and text-based protocol communication.

```rust
// Import necessary modules for I/O and networking
use std::io::{Read, Write};
use std::net::TcpStream;

// Main function that connects to the quote of the day service
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Establish a TCP connection to the server
    let mut con = TcpStream::connect("djxmmx.net:17")?;
    
    // Define an empty message to send
    let msg = "";
    // Send the message to the server
    con.write_all(msg.as_bytes())?;
    
    // Prepare a buffer to store the reply
    let mut reply = [0; 1024];
    // Read the reply from the connection
    let n = con.read(&mut reply)?;
    
    // Print the reply as a string, handling invalid UTF-8
    println!("{}", String::from_utf8_lossy(&reply[..n]));
    
    // Return success (connection closes automatically)
    Ok(())
}
```

The QOTD protocol operates on port 17 and returns a quote when  
connected. TcpStream::connect() establishes a TCP connection to the  
server. Writing an empty message triggers the server response. The  
String::from_utf8_lossy() method safely converts potentially invalid  
UTF-8 bytes to a string, handling any encoding issues gracefully.


## HEAD request

HTTP HEAD request retrieves only response headers without the body.  
This is useful for checking resource metadata, content length, and  
last modification time without downloading the entire content.

```rust
// Import necessary modules for I/O and networking
use std::io::{Read, Write};
use std::net::TcpStream;

// Main function that performs the HTTP HEAD request
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Establish a TCP connection to the server
    let mut con = TcpStream::connect("webcode.me:80")?;
    
    // Define the HTTP HEAD request string
    let req = "HEAD / HTTP/1.0\r\n\r\n";
    // Send the request to the server
    con.write_all(req.as_bytes())?;
    
    // Prepare a vector to store the response
    let mut res = Vec::new();
    // Read the entire response from the connection
    con.read_to_end(&mut res)?;
    
    // Print the response as a string, handling invalid UTF-8
    println!("{}", String::from_utf8_lossy(&res));
    
    // Return success
    Ok(())
}
```

The HEAD method returns only HTTP headers without the response body.  
This is efficient for checking if a resource exists, getting its size,  
or verifying modification dates. The HTTP/1.0 protocol version ensures  
compatibility with older servers. The connection closes automatically  
after receiving the response headers.


## HTTPS request

Secure HTTP request using TLS encryption to protect data in transit.  
This example demonstrates establishing an encrypted connection using  
the rustls library for cryptographic operations.

```rust
// This program demonstrates how to make a simple HTTPS GET request using Rust's standard library
// and the rustls crate for TLS encryption. It connects to a web server, sends an HTTP request,
// and reads the response, all over a secure TLS connection.

use std::io::{Read, Write}; // For reading from and writing to streams
use std::net::TcpStream; // For establishing a TCP connection to the server
use std::sync::Arc; // For sharing the TLS configuration across threads (required by rustls)

use rustls::{ClientConfig, RootCertStore}; // Core types for TLS client configuration and certificate storage
use webpki_roots::TLS_SERVER_ROOTS; // A set of trusted root certificates for verifying server certificates

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Step 1: Set up the TLS configuration
    // We need to create a store of trusted root certificates that will be used to verify
    // the server's certificate during the TLS handshake.
    let mut root_store = RootCertStore::empty(); // Start with an empty certificate store
    // Add the default trusted root certificates (from webpki-roots) to the store
    root_store.extend(TLS_SERVER_ROOTS.iter().cloned());

    // Create a client configuration for TLS connections
    let config = ClientConfig::builder()
        .with_root_certificates(root_store) // Use our root store for certificate verification
        .with_no_client_auth(); // We don't need client certificates for this simple request

    // Step 2: Establish the HTTPS connection
    // For HTTPS, we need to specify the server's domain name for certificate validation
    let server_name = "webcode.me".try_into()?; // Convert the string to a ServerName type
                                                // Create a new TLS client connection with our config and the server name
    let mut conn = rustls::ClientConnection::new(Arc::new(config), server_name)?;
    // Establish the underlying TCP connection to the server on port 443 (standard HTTPS port)
    let mut sock = TcpStream::connect("webcode.me:443")?;
    // Wrap the TCP stream in a TLS stream to handle encryption/decryption
    let mut tls = rustls::Stream::new(&mut conn, &mut sock);

    // Step 3: Send the HTTP GET request
    // Craft a simple HTTP/1.1 GET request with necessary headers
    let request = "GET / HTTP/1.1\r\nHost: webcode.me\r\nConnection: close\r\n\r\n";
    // Write the request to the TLS stream (this will encrypt it before sending over TCP)
    tls.write_all(request.as_bytes())?;

    // Step 4: Read the server's response
    let mut response = String::new(); // Buffer to store the response
                                      // Read the entire response from the TLS stream (decrypting as we go)
    tls.read_to_string(&mut response)?;

    // Step 5: Output the response
    println!("{}", response); // Print the full HTTP response to the console

    Ok(()) // Indicate successful execution
}
```

HTTPS combines HTTP with TLS encryption for secure communication.  
The rustls crate provides modern TLS implementation with certificate  
verification using trusted root certificates. The ClientConnection  
handles the TLS handshake and encryption, while the underlying TCP  
stream carries the encrypted data. This pattern ensures data integrity  
and confidentiality over untrusted networks.

## HTTP POST request

Basic HTTP POST request using standard library TCP sockets to send  
data to a web server. This demonstrates manual HTTP protocol  
implementation without external HTTP client libraries.

```rust
// Import necessary modules for networking and I/O
use std::io::{Read, Write};
use std::net::TcpStream;

// Main function that sends a POST request with form data
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Connect to the target server
    let mut stream = TcpStream::connect("httpbin.org:80")?;
    
    // Prepare form data to send
    let form_data = "name=John&email=john@example.com&message=Hello";
    
    // Construct the HTTP POST request
    let request = format!(
        "POST /post HTTP/1.1\r\n\
         Host: httpbin.org\r\n\
         Content-Type: application/x-www-form-urlencoded\r\n\
         Content-Length: {}\r\n\
         Connection: close\r\n\
         \r\n\
         {}",
        form_data.len(),
        form_data
    );
    
    // Send the request to the server
    stream.write_all(request.as_bytes())?;
    println!("POST request sent");
    
    // Read the response from the server
    let mut response = String::new();
    stream.read_to_string(&mut response)?;
    
    // Print the HTTP response
    println!("Response received:");
    println!("{}", response);
    
    Ok(())
}
```

Manual HTTP POST implementation requires proper header formatting  
including Content-Type and Content-Length. The form data uses URL  
encoding for key-value pairs. The Connection: close header ensures  
the server closes the connection after sending the response. This  
approach provides full control over the HTTP protocol details.

## HTTP GET request

Simple HTTP GET request implementation using TCP sockets and manual  
HTTP protocol handling. This shows the fundamental structure of HTTP  
communication without high-level abstractions.

```rust
// Import necessary modules for networking and I/O
use std::io::{Read, Write};
use std::net::TcpStream;

// Main function that performs an HTTP GET request
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Establish TCP connection to the web server
    let mut stream = TcpStream::connect("httpbin.org:80")?;
    
    // Construct the HTTP GET request
    let request = "GET /get?param1=value1&param2=value2 HTTP/1.1\r\n\
                   Host: httpbin.org\r\n\
                   User-Agent: Rust-HTTP-Client/1.0\r\n\
                   Accept: application/json\r\n\
                   Connection: close\r\n\
                   \r\n";
    
    // Send the request to the server
    stream.write_all(request.as_bytes())?;
    println!("GET request sent to /get endpoint");
    
    // Read the complete response
    let mut response = String::new();
    stream.read_to_string(&mut response)?;
    
    // Parse and display response parts
    if let Some((headers, body)) = response.split_once("\r\n\r\n") {
        println!("Response headers:");
        for line in headers.lines().take(5) {
            println!("  {}", line);
        }
        
        println!("\nResponse body (first 200 chars):");
        let preview = if body.len() > 200 {
            &body[..200]
        } else {
            body
        };
        println!("{}", preview);
    } else {
        println!("Full response:\n{}", response);
    }
    
    Ok(())
}
```

HTTP GET requests follow a simple text-based protocol format. The  
request line specifies the method, path with query parameters, and  
HTTP version. Headers provide metadata like Host and User-Agent.  
The empty line separates headers from the body. Manual parsing  
splits the response into headers and body sections for analysis.

## WebSocket client

WebSocket client implementation for bidirectional real-time  
communication. This example establishes a WebSocket connection  
and demonstrates the handshake process.

```rust
// Import necessary modules for WebSocket communication
use std::io::{Read, Write};
use std::net::TcpStream;

// Main function that establishes a WebSocket connection
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Connect to WebSocket echo server
    let mut stream = TcpStream::connect("echo.websocket.org:80")?;
    
    // Generate WebSocket key for handshake
    let websocket_key = "dGhlIHNhbXBsZSBub25jZQ=="; // Base64 encoded key
    
    // Send WebSocket handshake request
    let handshake = format!(
        "GET / HTTP/1.1\r\n\
         Host: echo.websocket.org\r\n\
         Upgrade: websocket\r\n\
         Connection: Upgrade\r\n\
         Sec-WebSocket-Key: {}\r\n\
         Sec-WebSocket-Version: 13\r\n\
         \r\n",
        websocket_key
    );
    
    stream.write_all(handshake.as_bytes())?;
    println!("WebSocket handshake sent");
    
    // Read handshake response
    let mut response = [0; 1024];
    let bytes_read = stream.read(&mut response)?;
    let response_str = String::from_utf8_lossy(&response[..bytes_read]);
    
    // Check if handshake was successful
    if response_str.contains("101 Switching Protocols") {
        println!("✓ WebSocket handshake successful");
        
        // Send a simple text frame (minimal implementation)
        let message = "Hello WebSocket!";
        let frame = create_text_frame(message);
        stream.write_all(&frame)?;
        println!("✓ Sent message: {}", message);
        
        // Read echoed response (simplified frame parsing)
        let mut echo_response = [0; 1024];
        let echo_bytes = stream.read(&mut echo_response)?;
        if echo_bytes > 2 {
            // Skip frame header for echo display
            let echo_message = String::from_utf8_lossy(&echo_response[2..echo_bytes]);
            println!("✓ Received echo: {}", echo_message);
        }
    } else {
        println!("✗ WebSocket handshake failed");
        println!("Response: {}", response_str);
    }
    
    Ok(())
}

// Helper function to create a simple WebSocket text frame
fn create_text_frame(message: &str) -> Vec<u8> {
    let mut frame = Vec::new();
    let message_bytes = message.as_bytes();
    
    // Frame header: FIN=1, opcode=1 (text)
    frame.push(0x81);
    
    // Payload length and mask bit
    if message_bytes.len() < 126 {
        frame.push(0x80 | message_bytes.len() as u8); // Mask bit + length
    }
    
    // Masking key (simplified - normally should be random)
    let mask = [0x12, 0x34, 0x56, 0x78];
    frame.extend_from_slice(&mask);
    
    // Masked payload
    for (i, &byte) in message_bytes.iter().enumerate() {
        frame.push(byte ^ mask[i % 4]);
    }
    
    frame
}
```

WebSockets enable full-duplex communication over a single TCP  
connection. The protocol begins with an HTTP upgrade handshake  
using specific headers. The Sec-WebSocket-Key triggers server  
validation. Frame structure includes opcodes for different message  
types and masking for client-to-server data. This foundation  
supports real-time applications like chat and live updates.

## Multicast UDP

UDP multicast communication for one-to-many messaging. This example  
demonstrates joining a multicast group and broadcasting messages  
to multiple receivers simultaneously.

```rust
// Import necessary modules for multicast networking
use std::net::{UdpSocket, Ipv4Addr, SocketAddrV4};
use std::time::Duration;
use std::thread;

// Main function demonstrating multicast communication
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Start multicast receiver in a separate thread
    let receiver_handle = thread::spawn(|| {
        if let Err(e) = start_multicast_receiver() {
            eprintln!("Receiver error: {}", e);
        }
    });
    
    // Give receiver time to start and join multicast group
    thread::sleep(Duration::from_millis(500));
    
    // Start multicast sender
    start_multicast_sender()?;
    
    // Wait for receiver to finish
    receiver_handle.join().unwrap();
    
    Ok(())
}

// Function to start multicast message sender
fn start_multicast_sender() -> Result<(), Box<dyn std::error::Error>> {
    println!("Starting multicast sender...");
    
    // Create UDP socket for sending
    let socket = UdpSocket::bind("0.0.0.0:0")?;
    
    // Set TTL for multicast packets
    socket.set_multicast_ttl_v4(2)?;
    
    // Multicast group address (224.0.0.0 to 239.255.255.255)
    let multicast_addr = SocketAddrV4::new(Ipv4Addr::new(224, 0, 0, 251), 5000);
    
    // Send multicast messages
    for i in 1..=5 {
        let message = format!("Multicast message #{}", i);
        socket.send_to(message.as_bytes(), multicast_addr)?;
        println!("✓ Sent: {}", message);
        
        thread::sleep(Duration::from_secs(1));
    }
    
    // Send termination message
    socket.send_to(b"TERMINATE", multicast_addr)?;
    println!("✓ Sent termination signal");
    
    Ok(())
}

// Function to start multicast message receiver
fn start_multicast_receiver() -> Result<(), Box<dyn std::error::Error>> {
    println!("Starting multicast receiver...");
    
    // Bind to multicast port
    let socket = UdpSocket::bind("0.0.0.0:5000")?;
    
    // Join multicast group
    let multicast_addr = Ipv4Addr::new(224, 0, 0, 251);
    let interface_addr = Ipv4Addr::new(0, 0, 0, 0); // Any interface
    socket.join_multicast_v4(&multicast_addr, &interface_addr)?;
    
    println!("✓ Joined multicast group 224.0.0.251");
    
    // Set read timeout to avoid blocking forever
    socket.set_read_timeout(Some(Duration::from_secs(10)))?;
    
    // Listen for multicast messages
    let mut buffer = [0; 1024];
    loop {
        match socket.recv_from(&mut buffer) {
            Ok((bytes_received, sender_addr)) => {
                let message = String::from_utf8_lossy(&buffer[..bytes_received]);
                
                if message == "TERMINATE" {
                    println!("✓ Received termination signal from {}", sender_addr);
                    break;
                }
                
                println!("✓ Received from {}: {}", sender_addr, message);
            }
            Err(e) => {
                println!("✗ Receive error or timeout: {}", e);
                break;
            }
        }
    }
    
    // Leave multicast group
    socket.leave_multicast_v4(&multicast_addr, &interface_addr)?;
    println!("✓ Left multicast group");
    
    Ok(())
}
```

Multicast UDP enables efficient one-to-many communication using  
special IP addresses (224.0.0.0-239.255.255.255). Receivers join  
multicast groups using join_multicast_v4() and automatically  
receive messages sent to the group address. TTL controls how many  
network hops multicast packets can traverse. This is ideal for  
streaming, discovery protocols, and distributed notifications.

## HTTP proxy server

Basic HTTP proxy server that forwards client requests to target  
servers and relays responses back. This demonstrates request  
parsing, connection forwarding, and response handling.

```rust
// Import necessary modules for proxy server implementation
use std::io::{Read, Write, BufRead, BufReader};
use std::net::{TcpListener, TcpStream};
use std::thread;

// Main function that starts the HTTP proxy server
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let proxy_addr = "127.0.0.1:8888";
    let listener = TcpListener::bind(proxy_addr)?;
    
    println!("HTTP Proxy server listening on {}", proxy_addr);
    println!("Configure your browser to use 127.0.0.1:8888 as HTTP proxy");
    
    // Accept incoming client connections
    for stream in listener.incoming() {
        match stream {
            Ok(client_stream) => {
                // Handle each client in a separate thread
                thread::spawn(move || {
                    if let Err(e) = handle_client_connection(client_stream) {
                        eprintln!("Client handling error: {}", e);
                    }
                });
            }
            Err(e) => eprintln!("Connection error: {}", e),
        }
    }
    
    Ok(())
}

// Function to handle individual client connections
fn handle_client_connection(mut client_stream: TcpStream) -> Result<(), Box<dyn std::error::Error>> {
    let client_addr = client_stream.peer_addr()?;
    println!("New client connection from: {}", client_addr);
    
    // Read the HTTP request from client
    let mut reader = BufReader::new(&client_stream);
    let mut request_line = String::new();
    reader.read_line(&mut request_line)?;
    
    // Parse the request line (METHOD URL HTTP/VERSION)
    let parts: Vec<&str> = request_line.trim().split_whitespace().collect();
    if parts.len() != 3 {
        send_error_response(&mut client_stream, "400 Bad Request")?;
        return Ok(());
    }
    
    let method = parts[0];
    let url = parts[1];
    let _http_version = parts[2];
    
    println!("Request: {} {}", method, url);
    
    // Handle CONNECT method for HTTPS
    if method == "CONNECT" {
        return handle_connect_method(client_stream, url);
    }
    
    // Parse URL to extract host and path
    let (host, path) = if url.starts_with("http://") {
        parse_http_url(&url[7..])
    } else {
        // Relative URL - need Host header
        ("unknown", url)
    };
    
    if host == "unknown" {
        // Read headers to find Host header
        let mut host_header = String::new();
        for line_result in reader.lines() {
            let line = line_result?;
            if line.is_empty() {
                break;
            }
            if line.to_lowercase().starts_with("host:") {
                host_header = line[5..].trim().to_string();
                break;
            }
        }
        if host_header.is_empty() {
            send_error_response(&mut client_stream, "400 Bad Request - No Host")?;
            return Ok(());
        }
    }
    
    // Connect to target server
    let target_addr = if host.contains(':') {
        host.to_string()
    } else {
        format!("{}:80", host)
    };
    
    match TcpStream::connect(&target_addr) {
        Ok(mut target_stream) => {
            println!("✓ Connected to target server: {}", target_addr);
            
            // Forward the request to target server
            let forwarded_request = format!("{} {} HTTP/1.1\r\nHost: {}\r\nConnection: close\r\n\r\n", 
                                          method, path, host);
            target_stream.write_all(forwarded_request.as_bytes())?;
            
            // Forward response back to client
            let mut response_buffer = [0; 4096];
            loop {
                match target_stream.read(&mut response_buffer) {
                    Ok(0) => break, // Connection closed
                    Ok(n) => {
                        client_stream.write_all(&response_buffer[..n])?;
                    }
                    Err(_) => break,
                }
            }
            
            println!("✓ Request completed for {}", target_addr);
        }
        Err(e) => {
            eprintln!("✗ Failed to connect to {}: {}", target_addr, e);
            send_error_response(&mut client_stream, "502 Bad Gateway")?;
        }
    }
    
    Ok(())
}

// Function to handle HTTPS CONNECT method
fn handle_connect_method(mut client_stream: TcpStream, target: &str) -> Result<(), Box<dyn std::error::Error>> {
    // Connect to target HTTPS server
    match TcpStream::connect(target) {
        Ok(target_stream) => {
            // Send connection established response
            let response = "HTTP/1.1 200 Connection established\r\n\r\n";
            client_stream.write_all(response.as_bytes())?;
            
            println!("✓ HTTPS tunnel established to {}", target);
            
            // Start bidirectional data forwarding
            tunnel_data(client_stream, target_stream)?;
        }
        Err(e) => {
            eprintln!("✗ Failed to connect to HTTPS target {}: {}", target, e);
            let response = "HTTP/1.1 502 Bad Gateway\r\n\r\n";
            client_stream.write_all(response.as_bytes())?;
        }
    }
    
    Ok(())
}

// Function to tunnel data bidirectionally between client and server
fn tunnel_data(client_stream: TcpStream, target_stream: TcpStream) -> Result<(), Box<dyn std::error::Error>> {
    let client_clone = client_stream.try_clone()?;
    let target_clone = target_stream.try_clone()?;
    
    // Forward data from client to target server
    let client_to_target = thread::spawn(move || {
        copy_data(client_clone, target_clone)
    });
    
    // Forward data from target server to client
    let target_to_client = thread::spawn(move || {
        copy_data(target_stream, client_stream)
    });
    
    // Wait for either direction to complete
    let _ = client_to_target.join();
    let _ = target_to_client.join();
    
    Ok(())
}

// Helper function to copy data between streams
fn copy_data(mut source: TcpStream, mut destination: TcpStream) {
    let mut buffer = [0; 4096];
    while let Ok(n) = source.read(&mut buffer) {
        if n == 0 {
            break;
        }
        if destination.write_all(&buffer[..n]).is_err() {
            break;
        }
    }
}

// Helper function to parse HTTP URL
fn parse_http_url(url: &str) -> (&str, &str) {
    if let Some(slash_pos) = url.find('/') {
        (&url[..slash_pos], &url[slash_pos..])
    } else {
        (url, "/")
    }
}

// Helper function to send error responses
fn send_error_response(stream: &mut TcpStream, status: &str) -> Result<(), Box<dyn std::error::Error>> {
    let response = format!("HTTP/1.1 {}\r\nContent-Length: 0\r\nConnection: close\r\n\r\n", status);
    stream.write_all(response.as_bytes())?;
    Ok(())
}
```

HTTP proxy servers intercept and forward client requests to target  
servers. They parse HTTP request lines to extract methods and URLs,  
establish connections to destination servers, and relay responses.  
The CONNECT method enables HTTPS tunneling by creating bidirectional  
data pipes. This architecture supports web filtering, caching,  
and network monitoring applications.

## Network health checker

Network connectivity monitoring tool that checks the health of  
multiple network services and endpoints. This example performs  
various connection tests and reports service availability.

```rust
// Import necessary modules for network health checking
use std::net::{TcpStream, SocketAddr, ToSocketAddrs};
use std::time::{Duration, Instant};
use std::thread;
use std::sync::{Arc, Mutex};

// Structure to represent a network service endpoint
#[derive(Debug, Clone)]
struct ServiceEndpoint {
    name: String,
    address: String,
    port: u16,
    timeout: Duration,
}

// Structure to represent health check results
#[derive(Debug)]
struct HealthCheckResult {
    endpoint: ServiceEndpoint,
    is_healthy: bool,
    response_time: Duration,
    error_message: Option<String>,
}

impl ServiceEndpoint {
    fn new(name: &str, address: &str, port: u16, timeout_secs: u64) -> Self {
        ServiceEndpoint {
            name: name.to_string(),
            address: address.to_string(),
            port,
            timeout: Duration::from_secs(timeout_secs),
        }
    }
}

// Main function that performs network health checks
fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("Network Health Checker Starting...\n");
    
    // Define services to monitor
    let services = vec![
        ServiceEndpoint::new("Google DNS", "8.8.8.8", 53, 3),
        ServiceEndpoint::new("Cloudflare DNS", "1.1.1.1", 53, 3),
        ServiceEndpoint::new("Google HTTP", "google.com", 80, 5),
        ServiceEndpoint::new("GitHub HTTPS", "github.com", 443, 5),
        ServiceEndpoint::new("Local SSH", "127.0.0.1", 22, 2),
        ServiceEndpoint::new("Unreachable Service", "192.168.255.254", 12345, 2),
    ];
    
    // Perform health checks
    let results = perform_health_checks(&services);
    
    // Display results
    display_health_report(&results);
    
    // Monitor continuously (demo with 3 iterations)
    println!("\nStarting continuous monitoring (3 cycles)...");
    for cycle in 1..=3 {
        println!("\n--- Health Check Cycle {} ---", cycle);
        thread::sleep(Duration::from_secs(2));
        
        let current_results = perform_health_checks(&services);
        display_summary(&current_results);
    }
    
    Ok(())
}

// Function to perform health checks on all services
fn perform_health_checks(services: &[ServiceEndpoint]) -> Vec<HealthCheckResult> {
    let results = Arc::new(Mutex::new(Vec::new()));
    let mut handles = Vec::new();
    
    // Check each service in parallel
    for service in services {
        let service_clone = service.clone();
        let results_clone = Arc::clone(&results);
        
        let handle = thread::spawn(move || {
            let result = check_service_health(&service_clone);
            results_clone.lock().unwrap().push(result);
        });
        
        handles.push(handle);
    }
    
    // Wait for all checks to complete
    for handle in handles {
        handle.join().unwrap();
    }
    
    // Extract results from the Arc<Mutex<>>
    let results_guard = results.lock().unwrap();
    results_guard.clone()
}

// Function to check health of a single service
fn check_service_health(endpoint: &ServiceEndpoint) -> HealthCheckResult {
    let start_time = Instant::now();
    
    // Resolve hostname to socket address
    let socket_addrs: Result<Vec<SocketAddr>, _> = 
        (endpoint.address.as_str(), endpoint.port).to_socket_addrs()
            .map(|addrs| addrs.collect());
    
    match socket_addrs {
        Ok(addrs) if !addrs.is_empty() => {
            let addr = addrs[0];
            
            // Attempt connection with timeout
            match TcpStream::connect_timeout(&addr, endpoint.timeout) {
                Ok(_stream) => {
                    let response_time = start_time.elapsed();
                    HealthCheckResult {
                        endpoint: endpoint.clone(),
                        is_healthy: true,
                        response_time,
                        error_message: None,
                    }
                }
                Err(e) => {
                    let response_time = start_time.elapsed();
                    HealthCheckResult {
                        endpoint: endpoint.clone(),
                        is_healthy: false,
                        response_time,
                        error_message: Some(format!("Connection failed: {}", e)),
                    }
                }
            }
        }
        Ok(_) => {
            HealthCheckResult {
                endpoint: endpoint.clone(),
                is_healthy: false,
                response_time: start_time.elapsed(),
                error_message: Some("No IP addresses resolved".to_string()),
            }
        }
        Err(e) => {
            HealthCheckResult {
                endpoint: endpoint.clone(),
                is_healthy: false,
                response_time: start_time.elapsed(),
                error_message: Some(format!("DNS resolution failed: {}", e)),
            }
        }
    }
}

// Function to display detailed health report
fn display_health_report(results: &[HealthCheckResult]) {
    println!("=== Network Health Report ===");
    
    let mut healthy_count = 0;
    let total_count = results.len();
    
    for result in results {
        let status_icon = if result.is_healthy { "✓" } else { "✗" };
        let status_text = if result.is_healthy { "HEALTHY" } else { "UNHEALTHY" };
        
        println!(
            "{} {} ({}:{}) - {} - {:.2?}",
            status_icon,
            result.endpoint.name,
            result.endpoint.address,
            result.endpoint.port,
            status_text,
            result.response_time
        );
        
        if let Some(error) = &result.error_message {
            println!("    Error: {}", error);
        }
        
        if result.is_healthy {
            healthy_count += 1;
        }
    }
    
    let health_percentage = (healthy_count as f64 / total_count as f64) * 100.0;
    println!(
        "\nOverall Health: {}/{} services healthy ({:.1}%)",
        healthy_count, total_count, health_percentage
    );
}

// Function to display summary report
fn display_summary(results: &[HealthCheckResult]) {
    let healthy_count = results.iter().filter(|r| r.is_healthy).count();
    let total_count = results.len();
    
    let avg_response_time: Duration = {
        let total_time: Duration = results.iter()
            .filter(|r| r.is_healthy)
            .map(|r| r.response_time)
            .sum();
        
        if healthy_count > 0 {
            total_time / healthy_count as u32
        } else {
            Duration::from_millis(0)
        }
    };
    
    println!(
        "Summary: {}/{} healthy, Avg response: {:.2?}",
        healthy_count, total_count, avg_response_time
    );
    
    // List any unhealthy services
    let unhealthy: Vec<_> = results.iter()
        .filter(|r| !r.is_healthy)
        .collect();
    
    if !unhealthy.is_empty() {
        println!("Unhealthy services:");
        for result in unhealthy {
            println!("  ✗ {}", result.endpoint.name);
        }
    }
}
```

Network health checkers monitor service availability and performance  
by attempting TCP connections to specified endpoints. Parallel  
execution using threads enables concurrent checking of multiple  
services. DNS resolution validation ensures hostnames can be  
resolved before connection attempts. Response time measurement  
helps identify performance degradation. This tool is essential  
for monitoring distributed systems and network infrastructure.

## Connection pooling

Basic connection pooling implementation for efficient resource management.  
This example shows how to reuse network connections to reduce overhead  
and improve performance in applications making multiple requests.

```rust
// Import necessary modules for connection pooling
use std::collections::VecDeque;
use std::net::TcpStream;
use std::sync::{Arc, Mutex};
use std::time::{Duration, Instant};
use std::thread;

// Structure representing a connection pool
pub struct ConnectionPool {
    connections: Arc<Mutex<VecDeque<PooledConnection>>>,
    max_size: usize,
    host: String,
    port: u16,
}

// Structure representing a pooled connection with metadata
#[derive(Debug)]
struct PooledConnection {
    stream: TcpStream,
    created_at: Instant,
    last_used: Instant,
}

impl ConnectionPool {
    // Create a new connection pool
    pub fn new(host: String, port: u16, max_size: usize) -> Self {
        ConnectionPool {
            connections: Arc::new(Mutex::new(VecDeque::new())),
            max_size,
            host,
            port,
        }
    }
    
    // Get a connection from the pool or create a new one
    pub fn get_connection(&self) -> Result<TcpStream, Box<dyn std::error::Error>> {
        let mut pool = self.connections.lock().unwrap();
        
        // Try to reuse an existing connection
        while let Some(mut pooled_conn) = pool.pop_front() {
            // Check if connection is still valid (simple age check)
            if pooled_conn.created_at.elapsed() < Duration::from_secs(300) {
                pooled_conn.last_used = Instant::now();
                println!("  ✓ Reusing pooled connection (age: {:?})", 
                         pooled_conn.created_at.elapsed());
                return Ok(pooled_conn.stream);
            } else {
                println!("  ⚠ Discarding old connection (age: {:?})", 
                         pooled_conn.created_at.elapsed());
            }
        }
        
        // Create new connection if pool is empty
        println!("  ✓ Creating new connection to {}:{}", self.host, self.port);
        let stream = TcpStream::connect(format!("{}:{}", self.host, self.port))?;
        
        // Set connection options
        stream.set_read_timeout(Some(Duration::from_secs(30)))?;
        stream.set_write_timeout(Some(Duration::from_secs(30)))?;
        
        Ok(stream)
    }
    
    // Return a connection to the pool
    pub fn return_connection(&self, stream: TcpStream) {
        let mut pool = self.connections.lock().unwrap();
        
        // Only return to pool if under size limit
        if pool.len() < self.max_size {
            let pooled_conn = PooledConnection {
                stream,
                created_at: Instant::now(),
                last_used: Instant::now(),
            };
            pool.push_back(pooled_conn);
            println!("  ✓ Connection returned to pool (pool size: {})", pool.len());
        } else {
            println!("  ⚠ Pool full, closing connection (max size: {})", self.max_size);
        }
    }
    
    // Clean up old connections from the pool
    pub fn cleanup_old_connections(&self) {
        let mut pool = self.connections.lock().unwrap();
        let max_age = Duration::from_secs(300); // 5 minutes
        
        let initial_size = pool.len();
        pool.retain(|conn| conn.created_at.elapsed() < max_age);
        let cleaned = initial_size - pool.len();
        
        if cleaned > 0 {
            println!("  🧹 Cleaned up {} old connections", cleaned);
        }
    }
    
    // Get pool statistics
    pub fn stats(&self) -> (usize, usize) {
        let pool = self.connections.lock().unwrap();
        (pool.len(), self.max_size)
    }
}

// Main function demonstrating connection pooling
fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("Connection Pooling Example:\n");
    
    // Create a connection pool for Google's web server
    let pool = ConnectionPool::new("google.com".to_string(), 80, 3);
    
    // Simulate multiple requests using the pool
    for request_num in 1..=8 {
        println!("Request {}:", request_num);
        
        // Get connection from pool
        let stream = pool.get_connection()?;
        
        // Simulate using the connection (in real code, you'd send HTTP requests)
        thread::sleep(Duration::from_millis(100));
        println!("  ✓ Used connection for request {}", request_num);
        
        // Return connection to pool (50% of the time, to show both scenarios)
        if request_num % 2 == 0 {
            pool.return_connection(stream);
        } else {
            println!("  ⚠ Connection closed (not returned to pool)");
            // Connection automatically closed when dropped
        }
        
        // Show pool stats
        let (current, max) = pool.stats();
        println!("  📊 Pool stats: {}/{} connections\n", current, max);
        
        // Small delay between requests
        thread::sleep(Duration::from_millis(200));
    }
    
    // Demonstrate cleanup
    println!("Performing cleanup:");
    pool.cleanup_old_connections();
    
    let (final_size, max_size) = pool.stats();
    println!("Final pool state: {}/{} connections", final_size, max_size);
    
    Ok(())
}
```

Connection pooling reduces the overhead of establishing new connections  
for each request. The pool maintains a collection of reusable connections  
with metadata tracking creation time and usage. Key features include  
maximum pool size limits, connection age validation, and automatic  
cleanup of stale connections. This pattern significantly improves  
performance in applications making frequent network requests.

## UDP client

Simple UDP client that sends a message to a server and receives a  
response. UDP is connectionless and doesn't guarantee delivery, making  
it suitable for real-time applications where speed is more important  
than reliability.

```rust
// Import necessary modules for networking
use std::net::UdpSocket;

// Main function that sends and receives UDP messages
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Bind to any available local port for sending
    let socket = UdpSocket::bind("127.0.0.1:0")?;
    
    // Define the message to send
    let message = "Hello from UDP client";
    // Send the message to the server at localhost:8080
    socket.send_to(message.as_bytes(), "127.0.0.1:8080")?;
    
    // Prepare a buffer to receive the response
    let mut buffer = [0; 1024];
    // Receive response from server
    let (bytes_received, server_addr) = socket.recv_from(&mut buffer)?;
    
    // Convert received bytes to string and print
    let response = String::from_utf8_lossy(&buffer[..bytes_received]);
    println!("Received from {}: {}", server_addr, response);
    
    Ok(())
}
```

The UdpSocket::bind() creates a socket bound to a local address. Using  
"127.0.0.1:0" binds to localhost and lets the OS choose an available  
port. The send_to() method sends data to a specific address, while  
recv_from() receives data and returns both the data and sender's  
address. UDP is ideal for scenarios requiring low latency communication.
