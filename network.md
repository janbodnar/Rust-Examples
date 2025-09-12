# Networking


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
