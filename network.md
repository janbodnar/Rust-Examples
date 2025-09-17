# Networking

## QOTD

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


## HEAD request

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


## HTTPS request

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

## UDP server

Basic UDP server that listens for incoming messages and sends responses  
back to clients. This example demonstrates how to create a simple echo  
server using UDP protocol.

```rust
// Import necessary modules for networking
use std::net::UdpSocket;

// Main function that runs a UDP echo server
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Bind the socket to listen on localhost port 8080
    let socket = UdpSocket::bind("127.0.0.1:8080")?;
    println!("UDP server listening on 127.0.0.1:8080");
    
    // Buffer to store incoming messages
    let mut buffer = [0; 1024];
    
    // Server loop - handle incoming messages
    loop {
        // Receive message from client
        let (bytes_received, client_addr) = socket.recv_from(&mut buffer)?;
        
        // Convert received bytes to string
        let message = String::from_utf8_lossy(&buffer[..bytes_received]);
        println!("Received from {}: {}", client_addr, message);
        
        // Echo the message back to the client
        let response = format!("Echo: {}", message);
        socket.send_to(response.as_bytes(), client_addr)?;
    }
}
```

The server binds to a specific address and port using UdpSocket::bind().  
The infinite loop continuously listens for incoming messages using  
recv_from(), which blocks until a message arrives. The server echoes  
each message back by sending a response to the client's address. This  
pattern is common for UDP servers that need to handle multiple clients.

## TCP server

Simple TCP server that accepts client connections and handles them  
sequentially. TCP provides reliable, ordered data delivery making it  
suitable for applications requiring guaranteed message delivery.

```rust
// Import necessary modules for networking and I/O
use std::io::{Read, Write};
use std::net::{TcpListener, TcpStream};

// Main function that runs a TCP echo server
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Bind the listener to localhost port 7878
    let listener = TcpListener::bind("127.0.0.1:7878")?;
    println!("TCP server listening on 127.0.0.1:7878");
    
    // Accept incoming connections
    for stream in listener.incoming() {
        match stream {
            Ok(stream) => {
                println!("New connection: {}", stream.peer_addr()?);
                handle_client(stream)?;
            }
            Err(e) => println!("Connection failed: {}", e),
        }
    }
    
    Ok(())
}

// Function to handle individual client connections
fn handle_client(mut stream: TcpStream) -> Result<(), Box<dyn std::error::Error>> {
    let mut buffer = [0; 1024];
    
    // Read data from the client
    let bytes_read = stream.read(&mut buffer)?;
    let message = String::from_utf8_lossy(&buffer[..bytes_read]);
    println!("Received: {}", message);
    
    // Send response back to client
    let response = format!("Echo: {}", message);
    stream.write_all(response.as_bytes())?;
    
    Ok(())
}
```

TcpListener::bind() creates a listener that waits for incoming  
connections. The incoming() method returns an iterator over connection  
attempts. Each successful connection yields a TcpStream that can be used  
for bidirectional communication. The handle_client() function processes  
each connection independently, reading data and sending responses.

## DNS lookup

Performing DNS resolution to convert domain names to IP addresses.  
This example demonstrates both forward and reverse DNS lookups using  
Rust's standard library networking functions.

```rust
// Import necessary modules for networking
use std::net::{IpAddr, ToSocketAddrs};

// Main function that performs DNS lookups
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Perform forward DNS lookup (domain to IP)
    let domain = "google.com";
    let addresses: Vec<_> = (domain, 80).to_socket_addrs()?.collect();
    
    println!("DNS lookup for {}:", domain);
    for addr in addresses {
        println!("  {}", addr.ip());
    }
    
    // Alternative method using specific DNS resolution
    let hostname = "rust-lang.org:443";
    match hostname.to_socket_addrs() {
        Ok(addrs) => {
            println!("\nDNS lookup for rust-lang.org:");
            for addr in addrs {
                println!("  {} (port {})", addr.ip(), addr.port());
            }
        }
        Err(e) => println!("DNS lookup failed: {}", e),
    }
    
    // Example of IP address parsing and validation
    let ip_str = "8.8.8.8";
    match ip_str.parse::<IpAddr>() {
        Ok(ip) => {
            println!("\nParsed IP address: {}", ip);
            println!("Is IPv4: {}", ip.is_ipv4());
            println!("Is IPv6: {}", ip.is_ipv6());
        }
        Err(e) => println!("Invalid IP address: {}", e),
    }
    
    Ok(())
}
```

The ToSocketAddrs trait provides DNS resolution functionality. Calling  
to_socket_addrs() on a hostname returns an iterator of socket addresses.  
The method handles both IPv4 and IPv6 addresses automatically. IP  
address parsing using the parse() method validates and converts string  
representations to IpAddr types, enabling programmatic IP manipulation.

## Port scanning

Simple port scanner that checks if specific ports are open on a target  
host. This example demonstrates TCP connection attempts with timeout  
handling for network reconnaissance purposes.

```rust
// Import necessary modules for networking and timing
use std::net::{SocketAddr, TcpStream};
use std::time::Duration;
use std::thread;

// Main function that scans ports on a target host
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let target_host = "127.0.0.1";
    let ports_to_scan = vec![22, 23, 53, 80, 110, 143, 443, 993, 995];
    
    println!("Scanning ports on {}...", target_host);
    
    for port in ports_to_scan {
        let address = format!("{}:{}", target_host, port);
        
        // Attempt to connect with a timeout
        match scan_port(&address) {
            Ok(true) => println!("Port {} is OPEN", port),
            Ok(false) => println!("Port {} is CLOSED", port),
            Err(e) => println!("Port {} scan failed: {}", port, e),
        }
        
        // Small delay between scans to be respectful
        thread::sleep(Duration::from_millis(100));
    }
    
    Ok(())
}

// Function to scan a single port
fn scan_port(address: &str) -> Result<bool, Box<dyn std::error::Error>> {
    let socket_addr: SocketAddr = address.parse()?;
    
    // Attempt connection with 3-second timeout
    match TcpStream::connect_timeout(&socket_addr, Duration::from_secs(3)) {
        Ok(_) => Ok(true),  // Connection successful - port is open
        Err(_) => Ok(false), // Connection failed - port is closed/filtered
    }
}
```

The TcpStream::connect_timeout() method attempts a connection with a  
specified timeout duration. This prevents the scanner from hanging on  
unresponsive ports. A successful connection indicates an open port,  
while connection failures typically mean the port is closed or  
filtered. The sleep between scans prevents overwhelming the target.

## IP address validation

Comprehensive IP address validation and manipulation functions.  
This example shows how to validate, parse, and work with both IPv4  
and IPv6 addresses in network applications.

```rust
// Import necessary modules for IP address handling
use std::net::{IpAddr, Ipv4Addr, Ipv6Addr};

// Main function demonstrating IP address validation
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let test_addresses = vec![
        "192.168.1.1",
        "10.0.0.1", 
        "172.16.0.1",
        "8.8.8.8",
        "2001:db8::1",
        "::1",
        "invalid.ip",
        "256.1.1.1",
        "192.168.1",
    ];
    
    println!("Validating IP addresses:\n");
    
    for addr_str in test_addresses {
        println!("Testing: {}", addr_str);
        validate_and_analyze_ip(addr_str);
        println!();
    }
    
    // Demonstrate IP address manipulation
    demonstrate_ip_operations();
    
    Ok(())
}

// Function to validate and analyze an IP address string
fn validate_and_analyze_ip(addr_str: &str) {
    match addr_str.parse::<IpAddr>() {
        Ok(ip) => {
            println!("  ✓ Valid IP address: {}", ip);
            
            match ip {
                IpAddr::V4(ipv4) => analyze_ipv4(ipv4),
                IpAddr::V6(ipv6) => analyze_ipv6(ipv6),
            }
        }
        Err(e) => println!("  ✗ Invalid IP address: {}", e),
    }
}

// Function to analyze IPv4 address properties
fn analyze_ipv4(ip: Ipv4Addr) {
    println!("    Type: IPv4");
    println!("    Is private: {}", ip.is_private());
    println!("    Is loopback: {}", ip.is_loopback());
    println!("    Is multicast: {}", ip.is_multicast());
    println!("    Is broadcast: {}", ip.is_broadcast());
    println!("    Octets: {:?}", ip.octets());
}

// Function to analyze IPv6 address properties  
fn analyze_ipv6(ip: Ipv6Addr) {
    println!("    Type: IPv6");
    println!("    Is loopback: {}", ip.is_loopback());
    println!("    Is multicast: {}", ip.is_multicast());
    println!("    Segments: {:?}", ip.segments());
}

// Function demonstrating IP address operations
fn demonstrate_ip_operations() {
    println!("IP Address Operations:");
    
    // Create specific IP addresses
    let localhost_v4 = Ipv4Addr::new(127, 0, 0, 1);
    let localhost_v6 = Ipv6Addr::new(0, 0, 0, 0, 0, 0, 0, 1);
    
    println!("  Localhost IPv4: {}", localhost_v4);
    println!("  Localhost IPv6: {}", localhost_v6);
    
    // Check if addresses are in same subnet (simplified example)
    let ip1 = Ipv4Addr::new(192, 168, 1, 10);
    let ip2 = Ipv4Addr::new(192, 168, 1, 20);
    println!("  {} and {} are in same /24 subnet: {}", 
             ip1, ip2, same_subnet_24(ip1, ip2));
}

// Helper function to check if two IPv4 addresses are in same /24 subnet
fn same_subnet_24(ip1: Ipv4Addr, ip2: Ipv4Addr) -> bool {
    let octets1 = ip1.octets();
    let octets2 = ip2.octets();
    octets1[0] == octets2[0] && octets1[1] == octets2[1] && octets1[2] == octets2[2]
}
```

The std::net module provides comprehensive IP address handling. The  
IpAddr enum encompasses both IPv4 and IPv6 addresses. Built-in methods  
like is_private(), is_loopback(), and is_multicast() help categorize  
addresses. The octets() and segments() methods provide access to the  
underlying binary representation for custom network calculations.

## Socket options

Configuring TCP socket options for fine-tuned network behavior.  
This example demonstrates setting various socket options like timeouts,  
keep-alive, and buffer sizes for optimized network performance.

```rust
// Import necessary modules for socket configuration
use std::net::{TcpStream, TcpListener};
use std::time::Duration;

// Main function demonstrating socket option configuration
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Configure a TCP client with custom socket options
    configure_client_socket()?;
    
    // Configure a TCP server with custom socket options
    configure_server_socket()?;
    
    Ok(())
}

// Function to configure client socket options
fn configure_client_socket() -> Result<(), Box<dyn std::error::Error>> {
    println!("Configuring client socket options:");
    
    let stream = TcpStream::connect("google.com:80")?;
    
    // Set read timeout
    stream.set_read_timeout(Some(Duration::from_secs(10)))?;
    println!("  ✓ Read timeout set to 10 seconds");
    
    // Set write timeout
    stream.set_write_timeout(Some(Duration::from_secs(10)))?;
    println!("  ✓ Write timeout set to 10 seconds");
    
    // Enable TCP keep-alive
    stream.set_ttl(64)?;
    println!("  ✓ TTL (Time To Live) set to 64");
    
    // Get and display current socket configuration
    if let Ok(timeout) = stream.read_timeout() {
        println!("  Current read timeout: {:?}", timeout);
    }
    
    if let Ok(timeout) = stream.write_timeout() {
        println!("  Current write timeout: {:?}", timeout);
    }
    
    if let Ok(ttl) = stream.ttl() {
        println!("  Current TTL: {}", ttl);
    }
    
    if let Ok(addr) = stream.local_addr() {
        println!("  Local address: {}", addr);
    }
    
    if let Ok(addr) = stream.peer_addr() {
        println!("  Peer address: {}", addr);
    }
    
    println!();
    Ok(())
}

// Function to configure server socket options
fn configure_server_socket() -> Result<(), Box<dyn std::error::Error>> {
    println!("Configuring server socket options:");
    
    let listener = TcpListener::bind("127.0.0.1:0")?;
    
    // Set TTL for the listener
    listener.set_ttl(64)?;
    println!("  ✓ Server TTL set to 64");
    
    // Get the actual bound address
    if let Ok(addr) = listener.local_addr() {
        println!("  Server bound to: {}", addr);
    }
    
    // Display current TTL setting
    if let Ok(ttl) = listener.ttl() {
        println!("  Current server TTL: {}", ttl);
    }
    
    println!("  ✓ Server socket configured successfully");
    
    Ok(())
}
```

Socket options provide fine-grained control over network behavior.  
set_read_timeout() and set_write_timeout() prevent operations from  
blocking indefinitely. The TTL (Time To Live) setting controls how  
many network hops a packet can make. These configurations are crucial  
for building robust network applications that handle various network  
conditions gracefully.

## Non-blocking I/O

Non-blocking network I/O operations that don't block the calling thread.  
This example demonstrates how to perform network operations without  
blocking execution, enabling responsive applications.

```rust
// Import necessary modules for non-blocking networking
use std::io::{Read, Write, ErrorKind};
use std::net::{TcpStream, TcpListener};
use std::time::Duration;
use std::thread;

// Main function demonstrating non-blocking I/O
fn main() -> Result<(), Box<dyn std::error::Error>> {
    // Start a simple server in a separate thread
    let server_handle = thread::spawn(|| {
        start_non_blocking_server().unwrap();
    });
    
    // Give server time to start
    thread::sleep(Duration::from_millis(100));
    
    // Connect client with non-blocking operations
    non_blocking_client()?;
    
    // Note: In a real application, you'd want to properly join the server thread
    // For this example, we'll let it run briefly
    thread::sleep(Duration::from_millis(500));
    
    Ok(())
}

// Function implementing a non-blocking client
fn non_blocking_client() -> Result<(), Box<dyn std::error::Error>> {
    println!("Starting non-blocking client...");
    
    let mut stream = TcpStream::connect("127.0.0.1:8081")?;
    
    // Set the stream to non-blocking mode
    stream.set_nonblocking(true)?;
    println!("  ✓ Client set to non-blocking mode");
    
    // Attempt to send data
    let message = "Hello from non-blocking client!";
    match stream.write_all(message.as_bytes()) {
        Ok(_) => println!("  ✓ Message sent successfully"),
        Err(e) if e.kind() == ErrorKind::WouldBlock => {
            println!("  ⚠ Write operation would block, trying again...");
            thread::sleep(Duration::from_millis(10));
        }
        Err(e) => return Err(e.into()),
    }
    
    // Attempt to read response with retry logic
    let mut buffer = [0; 1024];
    let mut attempts = 0;
    
    loop {
        match stream.read(&mut buffer) {
            Ok(0) => {
                println!("  ✓ Connection closed by server");
                break;
            }
            Ok(n) => {
                let response = String::from_utf8_lossy(&buffer[..n]);
                println!("  ✓ Received: {}", response);
                break;
            }
            Err(e) if e.kind() == ErrorKind::WouldBlock => {
                attempts += 1;
                if attempts > 100 {
                    println!("  ⚠ Read timeout after {} attempts", attempts);
                    break;
                }
                println!("  ⚠ Read would block, attempt {}", attempts);
                thread::sleep(Duration::from_millis(10));
            }
            Err(e) => return Err(e.into()),
        }
    }
    
    Ok(())
}

// Function implementing a non-blocking server
fn start_non_blocking_server() -> Result<(), Box<dyn std::error::Error>> {
    println!("Starting non-blocking server on 127.0.0.1:8081...");
    
    let listener = TcpListener::bind("127.0.0.1:8081")?;
    listener.set_nonblocking(true)?;
    
    println!("  ✓ Server listening in non-blocking mode");
    
    // Server loop with non-blocking accept
    for _ in 0..50 {  // Limited iterations for example
        match listener.accept() {
            Ok((mut stream, addr)) => {
                println!("  ✓ Accepted connection from: {}", addr);
                
                // Handle the connection in non-blocking mode
                stream.set_nonblocking(true)?;
                
                let mut buffer = [0; 1024];
                match stream.read(&mut buffer) {
                    Ok(n) if n > 0 => {
                        let message = String::from_utf8_lossy(&buffer[..n]);
                        println!("  Received: {}", message);
                        
                        // Echo response
                        let response = format!("Echo: {}", message);
                        let _ = stream.write_all(response.as_bytes());
                    }
                    Ok(0) => println!("  Connection closed"),
                    Err(e) if e.kind() == ErrorKind::WouldBlock => {
                        // No data available yet
                    }
                    Err(e) => println!("  Read error: {}", e),
                }
                break;
            }
            Err(e) if e.kind() == ErrorKind::WouldBlock => {
                // No pending connections, continue polling
                thread::sleep(Duration::from_millis(10));
            }
            Err(e) => return Err(e.into()),
        }
    }
    
    Ok(())
}
```

Non-blocking I/O prevents threads from blocking on network operations.  
set_nonblocking(true) configures sockets to return immediately with  
ErrorKind::WouldBlock when operations cannot complete. This enables  
building responsive applications that can handle multiple connections  
or perform other work while waiting for network operations to complete.

## Timeout handling

Implementing comprehensive timeout handling for network operations.  
This example demonstrates various timeout strategies for building  
resilient network applications that handle slow or unresponsive peers.

```rust
// Import necessary modules for timeout handling
use std::io::{Read, Write};
use std::net::{TcpStream, SocketAddr};
use std::time::{Duration, Instant};
use std::thread;

// Main function demonstrating various timeout strategies
fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("Demonstrating timeout handling strategies:\n");
    
    // 1. Connection timeout
    demonstrate_connection_timeout()?;
    
    // 2. Read/Write timeouts
    demonstrate_io_timeouts()?;
    
    // 3. Manual timeout implementation
    demonstrate_manual_timeout()?;
    
    Ok(())
}

// Function demonstrating connection timeouts
fn demonstrate_connection_timeout() -> Result<(), Box<dyn std::error::Error>> {
    println!("1. Connection Timeout Example:");
    
    let addresses = vec![
        "google.com:80",      // Should connect quickly
        "10.255.255.1:12345", // Likely to timeout
    ];
    
    for addr_str in addresses {
        let addr: SocketAddr = addr_str.parse()?;
        let timeout = Duration::from_secs(3);
        
        let start = Instant::now();
        
        match TcpStream::connect_timeout(&addr, timeout) {
            Ok(stream) => {
                let duration = start.elapsed();
                println!("  ✓ Connected to {} in {:?}", addr, duration);
                
                // Set additional timeouts for the connected stream
                stream.set_read_timeout(Some(Duration::from_secs(5)))?;
                stream.set_write_timeout(Some(Duration::from_secs(5)))?;
                println!("    Set read/write timeouts to 5 seconds");
            }
            Err(e) => {
                let duration = start.elapsed();
                println!("  ✗ Failed to connect to {} after {:?}: {}", addr, duration, e);
            }
        }
    }
    
    println!();
    Ok(())
}

// Function demonstrating I/O operation timeouts
fn demonstrate_io_timeouts() -> Result<(), Box<dyn std::error::Error>> {
    println!("2. I/O Operation Timeouts:");
    
    // Connect to a web server
    let mut stream = TcpStream::connect("httpbin.org:80")?;
    
    // Configure timeouts
    stream.set_read_timeout(Some(Duration::from_secs(10)))?;
    stream.set_write_timeout(Some(Duration::from_secs(5)))?;
    
    println!("  ✓ Connected to httpbin.org with timeouts configured");
    
    // Send HTTP request
    let request = "GET /delay/2 HTTP/1.1\r\nHost: httpbin.org\r\nConnection: close\r\n\r\n";
    
    let write_start = Instant::now();
    match stream.write_all(request.as_bytes()) {
        Ok(_) => {
            let write_duration = write_start.elapsed();
            println!("  ✓ Request sent in {:?}", write_duration);
        }
        Err(e) => {
            println!("  ✗ Write timeout or error: {}", e);
            return Ok(());
        }
    }
    
    // Read response with timeout
    let mut response = Vec::new();
    let read_start = Instant::now();
    
    match stream.read_to_end(&mut response) {
        Ok(bytes_read) => {
            let read_duration = read_start.elapsed();
            println!("  ✓ Received {} bytes in {:?}", bytes_read, read_duration);
            
            // Print first few lines of response
            let response_str = String::from_utf8_lossy(&response);
            for line in response_str.lines().take(3) {
                println!("    {}", line);
            }
        }
        Err(e) => {
            let read_duration = read_start.elapsed();
            println!("  ✗ Read timeout after {:?}: {}", read_duration, e);
        }
    }
    
    println!();
    Ok(())
}

// Function demonstrating manual timeout implementation
fn demonstrate_manual_timeout() -> Result<(), Box<dyn std::error::Error>> {
    println!("3. Manual Timeout Implementation:");
    
    let start = Instant::now();
    let timeout = Duration::from_secs(5);
    
    // Simulate a long-running operation with manual timeout checking
    loop {
        if start.elapsed() > timeout {
            println!("  ✗ Operation timed out after {:?}", timeout);
            break;
        }
        
        // Simulate some work
        thread::sleep(Duration::from_millis(100));
        
        // Simulate successful completion after some time
        if start.elapsed() > Duration::from_secs(2) {
            println!("  ✓ Operation completed successfully after {:?}", start.elapsed());
            break;
        }
    }
    
    // Demonstrate timeout with retry logic
    println!("\n  Timeout with retry logic:");
    let max_retries = 3;
    let retry_delay = Duration::from_millis(500);
    
    for attempt in 1..=max_retries {
        println!("    Attempt {} of {}", attempt, max_retries);
        
        // Simulate network operation that might fail
        let operation_start = Instant::now();
        let success = simulate_network_operation(Duration::from_millis(300));
        
        if success {
            println!("    ✓ Operation succeeded on attempt {} after {:?}", 
                     attempt, operation_start.elapsed());
            break;
        } else if attempt < max_retries {
            println!("    ✗ Attempt {} failed, retrying in {:?}", attempt, retry_delay);
            thread::sleep(retry_delay);
        } else {
            println!("    ✗ All {} attempts failed", max_retries);
        }
    }
    
    Ok(())
}

// Helper function to simulate a network operation
fn simulate_network_operation(duration: Duration) -> bool {
    thread::sleep(duration);
    // Randomly succeed or fail for demonstration
    use std::collections::hash_map::DefaultHasher;
    use std::hash::{Hash, Hasher};
    
    let mut hasher = DefaultHasher::new();
    duration.hash(&mut hasher);
    hasher.finish() % 3 == 0  // Succeed roughly 1/3 of the time
}
```

Timeout handling is crucial for network reliability. Connection timeouts  
using connect_timeout() prevent hanging on unreachable hosts. Socket  
timeouts with set_read_timeout() and set_write_timeout() limit operation  
duration. Manual timeout checking using Instant::now() and elapsed()  
provides custom timeout logic. Retry mechanisms with exponential  
backoff improve success rates for intermittent failures.

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
