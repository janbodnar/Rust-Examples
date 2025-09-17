# Async Programming with Tokio

## Basic async function

Simple async function demonstrating the fundamental async/await pattern.  
Async functions return futures that must be awaited to execute.

```rust
async fn greet(name: &str) -> String {
    format!("Hello, {}!", name)
}

#[tokio::main]
async fn main() {
    let greeting = greet("Alice").await;
    println!("{}", greeting);
}
```

The async keyword transforms a function into a state machine that can be  
paused and resumed. The await keyword yields control back to the runtime  
until the future completes. The #[tokio::main] attribute sets up the  
async runtime and transforms the main function into an async context.


## Concurrent execution with join!

Running multiple async operations concurrently and waiting for all to  
complete. The join! macro executes futures in parallel.

```rust
use tokio::time::{sleep, Duration};

async fn task_one() -> u32 {
    sleep(Duration::from_millis(100)).await;
    42
}

async fn task_two() -> String {
    sleep(Duration::from_millis(200)).await;
    "completed".to_string()
}

#[tokio::main]
async fn main() {
    let start = std::time::Instant::now();
    
    let (result1, result2) = tokio::join!(task_one(), task_two());
    
    println!("Task 1: {}", result1);
    println!("Task 2: {}", result2);
    println!("Total time: {:?}", start.elapsed());
}
```

The join! macro runs all futures concurrently and waits for all to  
complete before returning. This is more efficient than sequential  
execution. If any future panics, the entire join! will panic. Use  
try_join! for error handling with Result types.


## Task spawning with handles

Spawning independent async tasks that run in the background. Tasks can  
be detached or joined for result collection.

```rust
use tokio::time::{sleep, Duration};

async fn background_work(id: u32) -> String {
    sleep(Duration::from_millis(100 * id as u64)).await;
    format!("Task {} completed", id)
}

#[tokio::main]
async fn main() {
    let mut handles = vec![];
    
    // Spawn multiple tasks
    for i in 1..=3 {
        let handle = tokio::spawn(background_work(i));
        handles.push(handle);
    }
    
    // Collect results
    for handle in handles {
        match handle.await {
            Ok(result) => println!("{}", result),
            Err(e) => println!("Task failed: {:?}", e),
        }
    }
}
```

The tokio::spawn function creates a new task that runs independently on  
the runtime. Each task returns a JoinHandle that can be awaited for the  
result. Tasks can be cancelled using the abort() method on the handle.  
Spawned tasks must have 'static lifetime or move their data.


## Racing futures with select!

Using select! to handle the first future that completes among multiple  
options. Useful for timeouts and event-driven programming.

```rust
use tokio::time::{sleep, Duration};

async fn slow_operation() -> String {
    sleep(Duration::from_millis(500)).await;
    "Slow operation completed".to_string()
}

async fn fast_operation() -> String {
    sleep(Duration::from_millis(100)).await;
    "Fast operation completed".to_string()
}

#[tokio::main]
async fn main() {
    tokio::select! {
        result = slow_operation() => {
            println!("Slow: {}", result);
        }
        result = fast_operation() => {
            println!("Fast: {}", result);
        }
        _ = sleep(Duration::from_millis(300)) => {
            println!("Timeout occurred");
        }
    }
}
```

The select! macro waits for the first future to complete and executes  
the corresponding branch. Other futures are dropped (cancelled). Each  
branch can have an optional condition guard. The biased attribute makes  
selection deterministic by checking branches in order.


## Timeout handling

Adding timeouts to async operations to prevent indefinite waiting.  
Essential for robust network and I/O operations.

```rust
use tokio::time::{timeout, sleep, Duration};

async fn potentially_slow_operation() -> Result<String, &'static str> {
    sleep(Duration::from_millis(300)).await;
    Ok("Operation completed".to_string())
}

#[tokio::main]
async fn main() {
    let timeout_duration = Duration::from_millis(200);
    
    match timeout(timeout_duration, potentially_slow_operation()).await {
        Ok(Ok(result)) => println!("Success: {}", result),
        Ok(Err(e)) => println!("Operation failed: {}", e),
        Err(_) => println!("Operation timed out"),
    }
    
    // Alternative using select!
    tokio::select! {
        result = potentially_slow_operation() => {
            match result {
                Ok(value) => println!("Completed: {}", value),
                Err(e) => println!("Error: {}", e),
            }
        }
        _ = sleep(Duration::from_millis(150)) => {
            println!("Timed out after 150ms");
        }
    }
}
```

The timeout function wraps any future with a time limit. If the future  
completes within the timeout, it returns Ok(result). If it times out,  
it returns Err(Elapsed). The select! approach provides more control over  
timeout behavior and allows custom timeout logic.


## Async channels for communication

Using channels to communicate between async tasks. Channels enable  
message passing and coordination between concurrent operations.

```rust
use tokio::sync::mpsc;
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let (tx, mut rx) = mpsc::channel::<String>(32);
    
    // Spawn producer task
    let producer = tokio::spawn(async move {
        for i in 1..=5 {
            let message = format!("Message {}", i);
            tx.send(message).await.unwrap();
            sleep(Duration::from_millis(100)).await;
        }
        // tx drops here, closing the channel
    });
    
    // Spawn consumer task
    let consumer = tokio::spawn(async move {
        while let Some(message) = rx.recv().await {
            println!("Received: {}", message);
        }
        println!("Channel closed");
    });
    
    // Wait for both tasks to complete
    let _ = tokio::join!(producer, consumer);
}
```

Multi-producer, single-consumer channels allow tasks to send messages  
asynchronously. The channel has a buffer size that determines how many  
messages can be queued. When the channel is full, senders will wait.  
The channel closes when all senders are dropped.


## Async streams and iteration

Working with async streams for processing sequences of async data.  
Streams are the async equivalent of iterators for handling data flows.

```rust
use tokio_stream::{self as stream, StreamExt};
use tokio::time::{interval, Duration};

async fn generate_numbers() -> impl stream::Stream<Item = u32> {
    stream::iter(1..=5)
}

#[tokio::main]
async fn main() {
    // Basic stream iteration
    let mut number_stream = generate_numbers().await;
    while let Some(number) = number_stream.next().await {
        println!("Number: {}", number);
    }
    
    // Stream transformations
    let doubled: Vec<u32> = stream::iter(1..=5)
        .map(|x| x * 2)
        .collect()
        .await;
    println!("Doubled: {:?}", doubled);
    
    // Interval stream (periodic events)
    let mut timer = interval(Duration::from_millis(100));
    for _ in 0..3 {
        timer.tick().await;
        println!("Timer tick");
    }
}
```

Streams provide lazy evaluation of async sequences. The StreamExt trait  
adds methods like map, filter, and collect for stream processing.  
Interval creates a stream that yields at regular time intervals. Streams  
can be infinite and are processed on-demand.


## Async file I/O operations

Performing file operations asynchronously to avoid blocking the runtime.  
Async file I/O prevents thread blocking during disk operations.

```rust
use tokio::fs::{File, OpenOptions};
use tokio::io::{AsyncReadExt, AsyncWriteExt, BufReader, AsyncBufReadExt};

#[tokio::main]
async fn main() -> tokio::io::Result<()> {
    // Write to file asynchronously
    let mut file = File::create("async_example.txt").await?;
    file.write_all(b"Hello, async world!\n").await?;
    file.write_all(b"Second line of text.\n").await?;
    
    // Read entire file
    let mut file = File::open("async_example.txt").await?;
    let mut contents = String::new();
    file.read_to_string(&mut contents).await?;
    println!("File contents:\n{}", contents);
    
    // Buffered line-by-line reading
    let file = File::open("async_example.txt").await?;
    let reader = BufReader::new(file);
    let mut lines = reader.lines();
    
    while let Some(line) = lines.next_line().await? {
        println!("Line: {}", line);
    }
    
    // Cleanup
    tokio::fs::remove_file("async_example.txt").await?;
    Ok(())
}
```

Async file operations use the tokio::fs module for non-blocking I/O.  
BufReader provides efficient buffered reading for large files. The  
AsyncReadExt and AsyncWriteExt traits add async methods to file handles.  
All operations return futures that can be awaited.


## Async TCP server

Creating a TCP server that handles multiple clients concurrently.  
Each client connection is handled in a separate async task.

```rust
use tokio::net::{TcpListener, TcpStream};
use tokio::io::{AsyncReadExt, AsyncWriteExt};

async fn handle_client(mut socket: TcpStream) -> tokio::io::Result<()> {
    let mut buffer = [0; 1024];
    
    loop {
        let bytes_read = socket.read(&mut buffer).await?;
        if bytes_read == 0 {
            break; // Client disconnected
        }
        
        let message = String::from_utf8_lossy(&buffer[..bytes_read]);
        println!("Received: {}", message.trim());
        
        // Echo back to client
        socket.write_all(&buffer[..bytes_read]).await?;
    }
    
    Ok(())
}

#[tokio::main]
async fn main() -> tokio::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Server listening on 127.0.0.1:8080");
    
    loop {
        let (socket, addr) = listener.accept().await?;
        println!("New client connected: {}", addr);
        
        // Spawn a task for each client
        tokio::spawn(async move {
            if let Err(e) = handle_client(socket).await {
                println!("Error handling client: {}", e);
            }
        });
    }
}
```

The TcpListener accepts incoming connections asynchronously. Each client  
is handled in a separate spawned task, allowing the server to handle  
multiple clients concurrently. The handle_client function processes  
messages in a loop until the client disconnects.


## Async TCP client

Creating a TCP client that connects to a server and exchanges messages  
asynchronously. Demonstrates client-side async networking.

```rust
use tokio::net::TcpStream;
use tokio::io::{AsyncReadExt, AsyncWriteExt};
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() -> tokio::io::Result<()> {
    // Connect to server
    let mut stream = TcpStream::connect("127.0.0.1:8080").await?;
    println!("Connected to server");
    
    // Send messages
    let messages = ["Hello", "How are you?", "Goodbye"];
    
    for message in messages {
        // Send message
        stream.write_all(message.as_bytes()).await?;
        println!("Sent: {}", message);
        
        // Read response
        let mut buffer = [0; 1024];
        let bytes_read = stream.read(&mut buffer).await?;
        let response = String::from_utf8_lossy(&buffer[..bytes_read]);
        println!("Received: {}", response);
        
        // Wait before next message
        sleep(Duration::from_millis(500)).await;
    }
    
    Ok(())
}
```

The TcpStream::connect method establishes an async connection to the  
server. Messages are sent using write_all and responses are read using  
read. The client demonstrates a typical request-response pattern with  
delays between messages.


## Broadcast channels

Broadcasting messages to multiple subscribers simultaneously. Useful for  
event notification and fan-out messaging patterns.

```rust
use tokio::sync::broadcast;
use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    let (tx, _rx) = broadcast::channel::<String>(16);
    
    // Create multiple subscribers
    let mut rx1 = tx.subscribe();
    let mut rx2 = tx.subscribe();
    let mut rx3 = tx.subscribe();
    
    // Spawn subscriber tasks
    let subscriber1 = tokio::spawn(async move {
        while let Ok(message) = rx1.recv().await {
            println!("Subscriber 1 received: {}", message);
        }
    });
    
    let subscriber2 = tokio::spawn(async move {
        while let Ok(message) = rx2.recv().await {
            println!("Subscriber 2 received: {}", message);
        }
    });
    
    let subscriber3 = tokio::spawn(async move {
        while let Ok(message) = rx3.recv().await {
            println!("Subscriber 3 received: {}", message);
        }
    });
    
    // Send broadcast messages
    let broadcaster = tokio::spawn(async move {
        for i in 1..=3 {
            let message = format!("Broadcast message {}", i);
            if let Err(e) = tx.send(message) {
                println!("Send error: {}", e);
            }
            sleep(Duration::from_millis(100)).await;
        }
    });
    
    // Wait for broadcaster to finish
    broadcaster.await.unwrap();
    
    // Give subscribers time to process
    sleep(Duration::from_millis(200)).await;
    
    // Cleanup (tasks will finish when channel closes)
    drop(tx);
    let _ = tokio::join!(subscriber1, subscriber2, subscriber3);
}
```

Broadcast channels allow one sender to transmit messages to multiple  
receivers. Each receiver gets a copy of every message. If a receiver  
is slow, it may miss messages when the buffer is full. The channel  
closes when the sender is dropped.


## Oneshot channels

One-time communication between tasks using oneshot channels. Perfect for  
returning results from spawned tasks or signaling completion.

```rust
use tokio::sync::oneshot;
use tokio::time::{sleep, Duration};

async fn compute_value(result_tx: oneshot::Sender<u64>) {
    // Simulate computation
    sleep(Duration::from_millis(200)).await;
    let result = 42 * 42;
    
    // Send result back
    if result_tx.send(result).is_err() {
        println!("Receiver was dropped");
    }
}

async fn maybe_compute(should_compute: bool) -> Result<String, &'static str> {
    if should_compute {
        sleep(Duration::from_millis(100)).await;
        Ok("Computation successful".to_string())
    } else {
        Err("Computation was skipped")
    }
}

#[tokio::main]
async fn main() {
    // Example 1: Task completion notification
    let (tx, rx) = oneshot::channel();
    
    tokio::spawn(compute_value(tx));
    
    match rx.await {
        Ok(result) => println!("Received result: {}", result),
        Err(_) => println!("Sender was dropped"),
    }
    
    // Example 2: Conditional computation
    let (tx2, rx2) = oneshot::channel();
    
    tokio::spawn(async move {
        let result = maybe_compute(true).await;
        let _ = tx2.send(result);
    });
    
    match rx2.await {
        Ok(Ok(value)) => println!("Success: {}", value),
        Ok(Err(e)) => println!("Error: {}", e),
        Err(_) => println!("Task panicked"),
    }
}
```

Oneshot channels provide single-use communication between tasks. The  
sender can only send one message, and the receiver can only receive  
once. They're ideal for returning results from async operations or  
signaling task completion.


## Async Mutex for synchronization

Using async-aware Mutex to protect shared data across async tasks.  
Unlike std::sync::Mutex, async Mutex doesn't block the thread.

```rust
use tokio::sync::Mutex;
use std::sync::Arc;
use tokio::time::{sleep, Duration};

#[derive(Debug)]
struct Counter {
    value: u64,
}

impl Counter {
    fn new() -> Self {
        Counter { value: 0 }
    }
    
    fn increment(&mut self) {
        self.value += 1;
    }
    
    fn get(&self) -> u64 {
        self.value
    }
}

#[tokio::main]
async fn main() {
    let counter = Arc::new(Mutex::new(Counter::new()));
    let mut handles = vec![];
    
    // Spawn multiple tasks that increment the counter
    for i in 1..=5 {
        let counter_clone = Arc::clone(&counter);
        let handle = tokio::spawn(async move {
            for _ in 0..3 {
                {
                    let mut guard = counter_clone.lock().await;
                    guard.increment();
                    println!("Task {} incremented counter to {}", i, guard.get());
                    // Lock is released when guard goes out of scope
                }
                sleep(Duration::from_millis(10)).await;
            }
        });
        handles.push(handle);
    }
    
    // Wait for all tasks to complete
    for handle in handles {
        handle.await.unwrap();
    }
    
    let final_count = counter.lock().await.get();
    println!("Final counter value: {}", final_count);
}
```

Async Mutex provides exclusive access to shared data across async tasks.  
Unlike regular Mutex, it yields to other tasks while waiting for the  
lock. The lock guard must be explicitly scoped to release the lock  
before awaiting other futures.


## RwLock for async read/write access

Using RwLock to allow multiple concurrent readers or exclusive writers.  
Efficient for data that is read frequently but written occasionally.

```rust
use tokio::sync::RwLock;
use std::sync::Arc;
use tokio::time::{sleep, Duration};
use std::collections::HashMap;

type Database = HashMap<String, String>;

#[tokio::main]
async fn main() {
    let db = Arc::new(RwLock::new(Database::new()));
    let mut handles = vec![];
    
    // Writer task
    let db_writer = Arc::clone(&db);
    let writer = tokio::spawn(async move {
        for i in 1..=3 {
            {
                let mut write_guard = db_writer.write().await;
                write_guard.insert(format!("key{}", i), format!("value{}", i));
                println!("Writer: Added key{} = value{}", i, i);
            }
            sleep(Duration::from_millis(100)).await;
        }
    });
    handles.push(writer);
    
    // Multiple reader tasks
    for reader_id in 1..=3 {
        let db_reader = Arc::clone(&db);
        let reader = tokio::spawn(async move {
            for _ in 0..4 {
                {
                    let read_guard = db_reader.read().await;
                    println!("Reader {}: Database has {} entries", 
                        reader_id, read_guard.len());
                    for (key, value) in read_guard.iter() {
                        println!("  Reader {}: {} = {}", reader_id, key, value);
                    }
                }
                sleep(Duration::from_millis(150)).await;
            }
        });
        handles.push(reader);
    }
    
    // Wait for all tasks
    for handle in handles {
        handle.await.unwrap();
    }
    
    println!("Final database state: {:?}", *db.read().await);
}
```

RwLock allows multiple concurrent readers when no writer is active, or  
one exclusive writer when no readers are active. This provides better  
performance for read-heavy workloads. Read and write guards automatically  
release locks when dropped.


## Semaphore for limiting concurrency

Using Semaphore to limit the number of concurrent operations. Essential  
for controlling resource usage and preventing system overload.

```rust
use tokio::sync::Semaphore;
use std::sync::Arc;
use tokio::time::{sleep, Duration};

async fn expensive_operation(id: u32, semaphore: Arc<Semaphore>) -> String {
    // Acquire permit before starting expensive work
    let _permit = semaphore.acquire().await.unwrap();
    
    println!("Starting expensive operation {}", id);
    
    // Simulate expensive work
    sleep(Duration::from_millis(200)).await;
    
    println!("Completed expensive operation {}", id);
    
    // Permit is automatically released when _permit is dropped
    format!("Result from operation {}", id)
}

#[tokio::main]
async fn main() {
    // Allow only 2 concurrent operations
    let semaphore = Arc::new(Semaphore::new(2));
    let mut handles = vec![];
    
    // Spawn 6 tasks, but only 2 can run concurrently
    for i in 1..=6 {
        let sem = Arc::clone(&semaphore);
        let handle = tokio::spawn(expensive_operation(i, sem));
        handles.push(handle);
    }
    
    // Wait for all operations to complete
    for handle in handles {
        match handle.await {
            Ok(result) => println!("Received: {}", result),
            Err(e) => println!("Task failed: {:?}", e),
        }
    }
    
    println!("All operations completed");
}
```

Semaphore controls access to a limited number of resources. It maintains  
a count of available permits. Tasks must acquire a permit before  
proceeding and release it when done. This prevents resource exhaustion  
and system overload.


## Async trait methods

Implementing async methods in traits using the async-trait crate.  
Async trait methods enable polymorphic async behavior.

```rust
use async_trait::async_trait;
use tokio::time::{sleep, Duration};

#[async_trait]
trait AsyncProcessor {
    async fn process(&self, data: &str) -> String;
    async fn validate(&self, data: &str) -> bool;
}

struct FastProcessor;
struct SlowProcessor;

#[async_trait]
impl AsyncProcessor for FastProcessor {
    async fn process(&self, data: &str) -> String {
        sleep(Duration::from_millis(50)).await;
        format!("Fast processed: {}", data)
    }
    
    async fn validate(&self, data: &str) -> bool {
        sleep(Duration::from_millis(10)).await;
        !data.is_empty()
    }
}

#[async_trait]
impl AsyncProcessor for SlowProcessor {
    async fn process(&self, data: &str) -> String {
        sleep(Duration::from_millis(200)).await;
        format!("Slow processed: {}", data.to_uppercase())
    }
    
    async fn validate(&self, data: &str) -> bool {
        sleep(Duration::from_millis(50)).await;
        data.len() > 3
    }
}

async fn process_with_any_processor(processor: &dyn AsyncProcessor, data: &str) {
    if processor.validate(data).await {
        let result = processor.process(data).await;
        println!("{}", result);
    } else {
        println!("Data validation failed for: {}", data);
    }
}

#[tokio::main]
async fn main() {
    let fast = FastProcessor;
    let slow = SlowProcessor;
    
    let data = "hello world";
    
    println!("Using fast processor:");
    process_with_any_processor(&fast, data).await;
    
    println!("Using slow processor:");
    process_with_any_processor(&slow, data).await;
    
    // Test validation
    println!("Validation tests:");
    process_with_any_processor(&fast, "").await;
    process_with_any_processor(&slow, "hi").await;
}
```

The async-trait crate enables async methods in traits by transforming  
them into methods that return boxed futures. This allows for dynamic  
dispatch with async methods. The #[async_trait] attribute must be  
applied to both trait definitions and implementations.


## Future and Pin concepts

Understanding Future trait and Pin for manual future implementation.  
Demonstrates low-level async concepts and custom future creation.

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll, Waker};
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::{Duration, Instant};

struct TimerFuture {
    shared_state: Arc<Mutex<SharedState>>,
}

struct SharedState {
    completed: bool,
    waker: Option<Waker>,
}

impl TimerFuture {
    pub fn new(duration: Duration) -> Self {
        let shared_state = Arc::new(Mutex::new(SharedState {
            completed: false,
            waker: None,
        }));
        
        let thread_shared_state = Arc::clone(&shared_state);
        thread::spawn(move || {
            thread::sleep(duration);
            let mut shared_state = thread_shared_state.lock().unwrap();
            shared_state.completed = true;
            if let Some(waker) = shared_state.waker.take() {
                waker.wake();
            }
        });
        
        TimerFuture { shared_state }
    }
}

impl Future for TimerFuture {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        let mut shared_state = self.shared_state.lock().unwrap();
        if shared_state.completed {
            Poll::Ready(())
        } else {
            shared_state.waker = Some(cx.waker().clone());
            Poll::Pending
        }
    }
}

#[tokio::main]
async fn main() {
    println!("Starting custom timer...");
    let start = Instant::now();
    
    TimerFuture::new(Duration::from_millis(500)).await;
    
    println!("Timer completed after {:?}", start.elapsed());
    
    // Using multiple custom timers
    let timer1 = TimerFuture::new(Duration::from_millis(100));
    let timer2 = TimerFuture::new(Duration::from_millis(200));
    let timer3 = TimerFuture::new(Duration::from_millis(150));
    
    println!("Racing three timers...");
    let start = Instant::now();
    
    tokio::select! {
        _ = timer1 => println!("Timer 1 finished first"),
        _ = timer2 => println!("Timer 2 finished first"),
        _ = timer3 => println!("Timer 3 finished first"),
    }
    
    println!("Race completed after {:?}", start.elapsed());
}
```

Future is the core trait for async operations in Rust. The poll method  
is called repeatedly until the future is ready. Pin ensures that futures  
don't move in memory, which is required for self-referential futures.  
Wakers notify the runtime when a future should be polled again.


## Async recursion

Implementing recursive async functions using Box for heap allocation.  
Async recursion requires explicit boxing due to unknown stack frame sizes.

```rust
use tokio::time::{sleep, Duration};

// Async recursive function with Box<dyn Future>
fn async_factorial(n: u64) -> Box<dyn std::future::Future<Output = u64> + Send + '_> {
    Box::new(async move {
        if n <= 1 {
            1
        } else {
            sleep(Duration::from_millis(10)).await; // Simulate async work
            n * async_factorial(n - 1).await
        }
    })
}

// Alternative using async-recursion crate (more ergonomic)
#[async_recursion::async_recursion]
async fn factorial_with_crate(n: u64) -> u64 {
    if n <= 1 {
        1
    } else {
        sleep(Duration::from_millis(10)).await;
        n * factorial_with_crate(n - 1).await
    }
}

// Tree traversal example
#[derive(Debug)]
struct Node {
    value: i32,
    children: Vec<Node>,
}

#[async_recursion::async_recursion]
async fn async_tree_sum(node: &Node) -> i32 {
    sleep(Duration::from_millis(5)).await; // Simulate async processing
    
    let mut sum = node.value;
    for child in &node.children {
        sum += async_tree_sum(child).await;
    }
    sum
}

#[tokio::main]
async fn main() {
    // Test factorial implementations
    println!("Computing 5! with manual boxing...");
    let result1 = async_factorial(5).await;
    println!("5! = {}", result1);
    
    println!("Computing 6! with async-recursion crate...");
    let result2 = factorial_with_crate(6).await;
    println!("6! = {}", result2);
    
    // Test tree traversal
    let tree = Node {
        value: 1,
        children: vec![
            Node {
                value: 2,
                children: vec![
                    Node { value: 4, children: vec![] },
                    Node { value: 5, children: vec![] },
                ],
            },
            Node {
                value: 3,
                children: vec![
                    Node { value: 6, children: vec![] },
                ],
            },
        ],
    };
    
    println!("Computing tree sum...");
    let tree_sum = async_tree_sum(&tree).await;
    println!("Tree sum: {}", tree_sum);
}
```

Async recursion requires heap allocation because the compiler cannot  
determine the size of recursive async call stacks at compile time.  
Manual boxing or the async-recursion crate solve this limitation.  
The crate provides a more ergonomic syntax for recursive async functions.


## Error handling in async contexts

Comprehensive error handling patterns for async operations including  
Result propagation, error conversion, and recovery strategies.

```rust
use tokio::fs::File;
use tokio::io::{AsyncReadExt, AsyncWriteExt};
use tokio::time::{timeout, Duration};
use std::error::Error;
use std::fmt;

#[derive(Debug)]
enum AsyncError {
    IoError(tokio::io::Error),
    TimeoutError,
    ValidationError(String),
}

impl fmt::Display for AsyncError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AsyncError::IoError(e) => write!(f, "IO error: {}", e),
            AsyncError::TimeoutError => write!(f, "Operation timed out"),
            AsyncError::ValidationError(msg) => write!(f, "Validation error: {}", msg),
        }
    }
}

impl Error for AsyncError {}

impl From<tokio::io::Error> for AsyncError {
    fn from(error: tokio::io::Error) -> Self {
        AsyncError::IoError(error)
    }
}

async fn validate_data(data: &str) -> Result<(), AsyncError> {
    if data.is_empty() {
        return Err(AsyncError::ValidationError("Data cannot be empty".to_string()));
    }
    if data.len() > 100 {
        return Err(AsyncError::ValidationError("Data too long".to_string()));
    }
    Ok(())
}

async fn write_file_with_timeout(filename: &str, data: &str) -> Result<(), AsyncError> {
    validate_data(data).await?;
    
    let write_operation = async {
        let mut file = File::create(filename).await?;
        file.write_all(data.as_bytes()).await?;
        Ok::<(), tokio::io::Error>(())
    };
    
    timeout(Duration::from_millis(1000), write_operation)
        .await
        .map_err(|_| AsyncError::TimeoutError)?
        .map_err(AsyncError::from)
}

async fn read_file_with_retry(filename: &str, max_retries: u32) -> Result<String, AsyncError> {
    let mut last_error = None;
    
    for attempt in 1..=max_retries {
        match File::open(filename).await {
            Ok(mut file) => {
                let mut contents = String::new();
                file.read_to_string(&mut contents).await?;
                return Ok(contents);
            }
            Err(e) => {
                println!("Attempt {} failed: {}", attempt, e);
                last_error = Some(e);
                if attempt < max_retries {
                    tokio::time::sleep(Duration::from_millis(100 * attempt as u64)).await;
                }
            }
        }
    }
    
    Err(AsyncError::IoError(last_error.unwrap()))
}

#[tokio::main]
async fn main() {
    let test_data = "Hello, async error handling!";
    let filename = "test_async_errors.txt";
    
    // Test successful operation
    match write_file_with_timeout(filename, test_data).await {
        Ok(()) => println!("File written successfully"),
        Err(e) => println!("Write failed: {}", e),
    }
    
    // Test reading with retry
    match read_file_with_retry(filename, 3).await {
        Ok(contents) => println!("File contents: {}", contents.trim()),
        Err(e) => println!("Read failed: {}", e),
    }
    
    // Test validation error
    match write_file_with_timeout("test2.txt", "").await {
        Ok(()) => println!("Empty file written"),
        Err(e) => println!("Expected validation error: {}", e),
    }
    
    // Test timeout (simulate slow operation)
    let slow_operation = async {
        tokio::time::sleep(Duration::from_millis(2000)).await;
        Ok(())
    };
    
    match timeout(Duration::from_millis(500), slow_operation).await {
        Ok(_) => println!("Operation completed"),
        Err(_) => println!("Operation timed out as expected"),
    }
    
    // Cleanup
    let _ = tokio::fs::remove_file(filename).await;
}
```

Async error handling uses the same Result patterns as sync code but with  
additional considerations for timeouts and task cancellation. Custom  
error types can implement From traits for automatic conversion. Retry  
logic and timeout patterns are common in async error handling strategies.


## Async HTTP client with retry

Making HTTP requests with automatic retry logic and circuit breaker  
pattern. Essential for resilient network communication in distributed  
systems.

```rust
use tokio::time::{sleep, Duration, Instant};
use std::collections::HashMap;

#[derive(Debug)]
struct HttpClient {
    base_url: String,
    max_retries: u32,
    retry_delay: Duration,
}

#[derive(Debug)]
enum HttpError {
    NetworkError(String),
    TimeoutError,
    ServerError(u16),
    TooManyRetries,
}

impl std::fmt::Display for HttpError {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        match self {
            HttpError::NetworkError(msg) => write!(f, "Network error: {}", msg),
            HttpError::TimeoutError => write!(f, "Request timed out"),
            HttpError::ServerError(code) => write!(f, "Server error: {}", code),
            HttpError::TooManyRetries => write!(f, "Too many retry attempts"),
        }
    }
}

impl std::error::Error for HttpError {}

impl HttpClient {
    fn new(base_url: String) -> Self {
        HttpClient {
            base_url,
            max_retries: 3,
            retry_delay: Duration::from_millis(1000),
        }
    }
    
    async fn get_with_retry(&self, path: &str) -> Result<String, HttpError> {
        let url = format!("{}{}", self.base_url, path);
        let mut last_error = None;
        
        for attempt in 1..=self.max_retries {
            println!("Attempt {} for {}", attempt, url);
            
            match self.make_request(&url).await {
                Ok(response) => return Ok(response),
                Err(e) => {
                    println!("Attempt {} failed: {:?}", attempt, e);
                    last_error = Some(e);
                    
                    if attempt < self.max_retries {
                        let delay = self.retry_delay * attempt;
                        println!("Retrying in {:?}...", delay);
                        sleep(delay).await;
                    }
                }
            }
        }
        
        Err(last_error.unwrap_or(HttpError::TooManyRetries))
    }
    
    async fn make_request(&self, url: &str) -> Result<String, HttpError> {
        // Simulate HTTP request with random failures
        let start = Instant::now();
        
        // Simulate network delay
        sleep(Duration::from_millis(100)).await;
        
        // Simulate different failure scenarios
        let random_outcome = (start.elapsed().as_millis() % 4) as u32;
        
        match random_outcome {
            0 => Ok(format!("Success response from {}", url)),
            1 => Err(HttpError::NetworkError("Connection refused".to_string())),
            2 => Err(HttpError::TimeoutError),
            3 => Err(HttpError::ServerError(500)),
            _ => Ok("Default response".to_string()),
        }
    }
    
    async fn batch_requests(&self, paths: Vec<&str>) -> Vec<Result<String, HttpError>> {
        let mut handles = vec![];
        
        // Spawn concurrent requests
        for path in paths {
            let client = HttpClient::new(self.base_url.clone());
            let path = path.to_string();
            
            let handle = tokio::spawn(async move {
                client.get_with_retry(&path).await
            });
            
            handles.push(handle);
        }
        
        // Collect results
        let mut results = vec![];
        for handle in handles {
            match handle.await {
                Ok(result) => results.push(result),
                Err(_) => results.push(Err(HttpError::NetworkError("Task panicked".to_string()))),
            }
        }
        
        results
    }
}

#[tokio::main]
async fn main() {
    let client = HttpClient::new("https://api.example.com".to_string());
    
    // Single request with retry
    println!("=== Single Request with Retry ===");
    match client.get_with_retry("/users/1").await {
        Ok(response) => println!("Success: {}", response),
        Err(e) => println!("Failed after retries: {}", e),
    }
    
    println!("\n=== Batch Requests ===");
    let paths = vec!["/users/1", "/users/2", "/users/3", "/posts/1"];
    let results = client.batch_requests(paths).await;
    
    for (i, result) in results.iter().enumerate() {
        match result {
            Ok(response) => println!("Request {}: Success - {}", i + 1, response),
            Err(e) => println!("Request {}: Failed - {}", i + 1, e),
        }
    }
    
    // Demonstrate circuit breaker pattern
    println!("\n=== Circuit Breaker Pattern ===");
    let mut consecutive_failures = 0;
    let max_failures = 2;
    
    for i in 1..=5 {
        match client.get_with_retry(&format!("/health/{}", i)).await {
            Ok(response) => {
                println!("Health check {}: OK - {}", i, response);
                consecutive_failures = 0; // Reset on success
            }
            Err(e) => {
                consecutive_failures += 1;
                println!("Health check {}: Failed - {}", i, e);
                
                if consecutive_failures >= max_failures {
                    println!("Circuit breaker opened! Stopping requests.");
                    break;
                }
            }
        }
        
        sleep(Duration::from_millis(500)).await;
    }
}
```

This example demonstrates practical HTTP client patterns including retry  
logic with exponential backoff, concurrent batch requests, and circuit  
breaker pattern for fault tolerance. The retry mechanism helps handle  
transient network failures while the circuit breaker prevents cascade  
failures in distributed systems.
