# Concurrency

Rust provides powerful concurrency primitives that enable safe, efficient  
parallel programming. The ownership system prevents data races at compile  
time, while providing zero-cost abstractions for high-performance  
concurrent applications.

Key concurrency concepts in Rust:  
- **Thread safety**: Rust's type system prevents data races  
- **Message passing**: Safe communication between threads via channels  
- **Shared state**: Controlled access to mutable data with Arc and Mutex  
- **Atomic operations**: Lock-free programming for high performance  
- **Async programming**: Non-blocking I/O with futures and async/await  

These examples progress from basic thread creation to advanced concurrent  
patterns, demonstrating practical solutions for real-world parallel  
programming challenges.

## Basic thread spawning

Creating and managing threads is the foundation of concurrent programming.  
Threads run independently and can execute code in parallel.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    // Spawn a new thread
    let handle = thread::spawn(|| {
        for i in 1..=5 {
            println!("Thread: count {}", i);
            thread::sleep(Duration::from_millis(100));
        }
    });
    
    // Main thread continues
    for i in 1..=3 {
        println!("Main: count {}", i);
        thread::sleep(Duration::from_millis(150));
    }
    
    // Wait for spawned thread to complete
    handle.join().unwrap();
    println!("Both threads completed");
}
```

Thread spawning creates a new operating system thread. The join method  
blocks until the thread completes, ensuring proper cleanup. Without join,  
the main thread might exit before spawned threads finish.

## Thread communication with channels

Channels provide safe message passing between threads, enabling  
communication without shared memory access.

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    
    // Spawn producer thread
    thread::spawn(move || {
        for i in 1..=5 {
            let message = format!("Message {}", i);
            tx.send(message).unwrap();
            thread::sleep(Duration::from_millis(100));
        }
    });
    
    // Consumer in main thread
    for received in rx {
        println!("Received: {}", received);
    }
    
    println!("Channel closed");
}
```

Channels follow the "do not communicate by sharing memory; instead,  
share memory by communicating" principle. The sender (tx) moves into  
the thread closure, while the receiver (rx) remains in the main thread.

## Multiple producers with channels

Multiple threads can send messages to a single receiver using  
cloned channel senders for scalable producer-consumer patterns.

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();
    let mut handles = vec![];
    
    // Spawn multiple producer threads
    for id in 1..=3 {
        let tx_clone = tx.clone();
        let handle = thread::spawn(move || {
            for i in 1..=3 {
                let message = format!("Producer {}: Message {}", id, i);
                tx_clone.send(message).unwrap();
                thread::sleep(Duration::from_millis(200));
            }
        });
        handles.push(handle);
    }
    
    // Drop original sender to signal completion
    drop(tx);
    
    // Wait for all producers to finish
    for handle in handles {
        handle.join().unwrap();
    }
    
    // Collect all messages
    for received in rx {
        println!("Received: {}", received);
    }
}
```

Cloning the sender allows multiple threads to send messages. Dropping  
the original sender ensures the receiver knows when all producers are  
done, preventing indefinite blocking.

## Shared state with Arc and Mutex

Arc (Atomically Reference Counted) and Mutex enable safe sharing of  
mutable data across multiple threads.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    // Spawn multiple threads that increment the counter
    for i in 1..=5 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 1..=100 {
                let mut num = counter_clone.lock().unwrap();
                *num += 1;
            }
            println!("Thread {} completed", i);
        });
        handles.push(handle);
    }
    
    // Wait for all threads to complete
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final counter value: {}", *counter.lock().unwrap());
}
```

Arc provides thread-safe reference counting for shared ownership.  
Mutex ensures only one thread can access the data at a time,  
preventing data races and ensuring consistent state.

## Producer-consumer with bounded channel

Bounded channels limit queue size, providing backpressure when  
consumers can't keep up with producers.

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::sync_channel(3); // Buffer size of 3
    
    // Producer thread
    let producer = thread::spawn(move || {
        for i in 1..=10 {
            println!("Producing item {}", i);
            tx.send(i).unwrap(); // Blocks when buffer is full
            thread::sleep(Duration::from_millis(50));
        }
    });
    
    // Consumer thread (slower than producer)
    let consumer = thread::spawn(move || {
        for item in rx {
            println!("Consuming item {}", item);
            thread::sleep(Duration::from_millis(200)); // Slower processing
        }
    });
    
    producer.join().unwrap();
    consumer.join().unwrap();
    println!("Producer-consumer completed");
}
```

Sync channels block producers when the buffer is full, preventing  
memory issues with fast producers and slow consumers. This provides  
natural flow control in concurrent systems.

## Atomic operations for simple shared state

Atomic types provide lock-free operations for simple shared state,  
offering better performance than Mutex for basic counters.

```rust
use std::sync::atomic::{AtomicUsize, Ordering};
use std::sync::Arc;
use std::thread;

fn main() {
    let counter = Arc::new(AtomicUsize::new(0));
    let mut handles = vec![];
    
    // Spawn threads that increment atomically
    for i in 1..=5 {
        let counter_clone = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            for _ in 1..=1000 {
                counter_clone.fetch_add(1, Ordering::SeqCst);
            }
            println!("Thread {} completed", i);
        });
        handles.push(handle);
    }
    
    // Wait for all threads
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final atomic counter: {}", counter.load(Ordering::SeqCst));
}
```

Atomic operations are faster than Mutex for simple operations like  
counters. SeqCst ordering provides the strongest memory ordering  
guarantees, ensuring operations appear atomic across all threads.

## Read-write locks for shared data

RwLock allows multiple concurrent readers or exclusive writers,  
optimizing for read-heavy workloads.

```rust
use std::sync::{Arc, RwLock};
use std::thread;
use std::time::Duration;
use std::collections::HashMap;

fn main() {
    let data = Arc::new(RwLock::new(HashMap::new()));
    let mut handles = vec![];
    
    // Writer thread
    let data_writer = Arc::clone(&data);
    let writer = thread::spawn(move || {
        for i in 1..=5 {
            {
                let mut map = data_writer.write().unwrap();
                map.insert(format!("key{}", i), i * 10);
                println!("Writer: Added key{} = {}", i, i * 10);
            }
            thread::sleep(Duration::from_millis(100));
        }
    });
    handles.push(writer);
    
    // Multiple reader threads
    for reader_id in 1..=3 {
        let data_reader = Arc::clone(&data);
        let reader = thread::spawn(move || {
            for _ in 1..=10 {
                {
                    let map = data_reader.read().unwrap();
                    let count = map.len();
                    if count > 0 {
                        println!("Reader {}: {} items in map", reader_id, count);
                    }
                }
                thread::sleep(Duration::from_millis(50));
            }
        });
        handles.push(reader);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

RwLock enables concurrent reads while ensuring exclusive writes.  
This pattern is ideal for data structures that are read frequently  
but updated rarely, providing better performance than Mutex.

## Thread barriers for synchronization

Barriers coordinate multiple threads to wait for all threads to  
reach a synchronization point before continuing.

```rust
use std::sync::{Arc, Barrier};
use std::thread;
use std::time::Duration;

fn main() {
    let barrier = Arc::new(Barrier::new(4));
    let mut handles = vec![];
    
    for i in 1..=4 {
        let barrier_clone = Arc::clone(&barrier);
        let handle = thread::spawn(move || {
            // Phase 1: Initialization
            println!("Thread {} starting initialization", i);
            thread::sleep(Duration::from_millis(i * 100)); // Different speeds
            println!("Thread {} finished initialization", i);
            
            // Wait for all threads to complete phase 1
            barrier_clone.wait();
            
            // Phase 2: Processing (all threads start together)
            println!("Thread {} starting processing", i);
            thread::sleep(Duration::from_millis(200));
            println!("Thread {} finished processing", i);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("All threads completed both phases");
}
```

Barriers ensure all threads reach the same execution point before  
any thread proceeds. This is useful for phased computations where  
each phase depends on the previous phase completing in all threads.

## Scoped threads for borrowing

Scoped threads allow borrowing local data safely, avoiding the need  
for Arc when data lifetime is guaranteed.

```rust
use std::thread;

fn main() {
    let data = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let chunk_size = 3;
    
    thread::scope(|s| {
        let mut handles = vec![];
        
        // Process data in parallel chunks
        for chunk in data.chunks(chunk_size) {
            let handle = s.spawn(|| {
                let sum: i32 = chunk.iter().sum();
                println!("Chunk {:?} sum: {}", chunk, sum);
                sum
            });
            handles.push(handle);
        }
        
        // Collect results
        let results: Vec<i32> = handles
            .into_iter()
            .map(|handle| handle.join().unwrap())
            .collect();
        
        let total: i32 = results.iter().sum();
        println!("Total sum: {}", total);
    });
}
```

Scoped threads automatically join when the scope ends, and can borrow  
local data safely. This eliminates the need for Arc when the data  
lifetime extends beyond thread execution.

## Thread-local storage

Thread-local storage provides each thread with its own copy of data,  
avoiding synchronization overhead when sharing isn't needed.

```rust
use std::cell::RefCell;
use std::thread;

thread_local! {
    static COUNTER: RefCell<u32> = RefCell::new(0);
}

fn increment_counter() {
    COUNTER.with(|c| {
        let mut counter = c.borrow_mut();
        *counter += 1;
        println!("Thread {:?}: Counter = {}", thread::current().id(), *counter);
    });
}

fn main() {
    let mut handles = vec![];
    
    for i in 1..=3 {
        let handle = thread::spawn(move || {
            for _ in 1..=3 {
                increment_counter();
                thread::sleep(std::time::Duration::from_millis(100));
            }
            println!("Thread {} completed", i);
        });
        handles.push(handle);
    }
    
    // Main thread also has its own counter
    for _ in 1..=2 {
        increment_counter();
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

Thread-local storage gives each thread its own instance of data,  
eliminating the need for synchronization. This is ideal for per-thread  
state like counters, caches, or configuration.

## Condition variables for thread coordination

Condition variables allow threads to wait for specific conditions  
and be notified when those conditions change.

```rust
use std::sync::{Arc, Mutex, Condvar};
use std::thread;
use std::time::Duration;

fn main() {
    let pair = Arc::new((Mutex::new(false), Condvar::new()));
    let pair2 = Arc::clone(&pair);
    
    // Spawned thread waits for condition
    let waiter = thread::spawn(move || {
        let (lock, cvar) = &*pair2;
        let mut ready = lock.lock().unwrap();
        
        while !*ready {
            println!("Waiter: Waiting for condition...");
            ready = cvar.wait(ready).unwrap();
        }
        
        println!("Waiter: Condition met, proceeding!");
    });
    
    // Main thread sets condition after delay
    thread::sleep(Duration::from_millis(500));
    
    let (lock, cvar) = &*pair;
    {
        let mut ready = lock.lock().unwrap();
        *ready = true;
        println!("Main: Setting condition to true");
    }
    cvar.notify_one();
    
    waiter.join().unwrap();
    println!("Program completed");
}
```

Condition variables efficiently coordinate threads by allowing them  
to sleep until a condition is met. This avoids busy waiting and  
provides precise thread coordination.

## Work-stealing queue pattern

Work-stealing queues distribute work dynamically among threads,  
balancing load automatically as threads complete tasks.

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;
use std::collections::VecDeque;

struct WorkQueue {
    tasks: Arc<Mutex<VecDeque<i32>>>,
}

impl WorkQueue {
    fn new() -> Self {
        Self {
            tasks: Arc::new(Mutex::new(VecDeque::new())),
        }
    }
    
    fn add_task(&self, task: i32) {
        let mut queue = self.tasks.lock().unwrap();
        queue.push_back(task);
    }
    
    fn steal_task(&self) -> Option<i32> {
        let mut queue = self.tasks.lock().unwrap();
        queue.pop_front()
    }
    
    fn clone_handle(&self) -> Self {
        Self {
            tasks: Arc::clone(&self.tasks),
        }
    }
}

fn worker(id: usize, queue: WorkQueue) {
    while let Some(task) = queue.steal_task() {
        println!("Worker {} processing task {}", id, task);
        thread::sleep(Duration::from_millis(100 + task as u64 * 10));
        println!("Worker {} completed task {}", id, task);
    }
    println!("Worker {} finished", id);
}

fn main() {
    let queue = WorkQueue::new();
    
    // Add tasks to queue
    for i in 1..=20 {
        queue.add_task(i);
    }
    
    let mut handles = vec![];
    
    // Spawn worker threads
    for id in 1..=4 {
        let worker_queue = queue.clone_handle();
        let handle = thread::spawn(move || {
            worker(id, worker_queue);
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

Work-stealing queues allow idle threads to steal work from busy  
threads, providing automatic load balancing. This pattern is used  
in many parallel frameworks for optimal work distribution.

## Parallel iteration with Rayon

Rayon provides data parallelism through parallel iterators,  
automatically distributing work across available CPU cores.

```rust
use rayon::prelude::*;

fn expensive_computation(n: u64) -> u64 {
    // Simulate expensive work
    (0..n).map(|i| i * i).sum::<u64>() % 1000
}

fn main() {
    let data: Vec<u64> = (1..=1000).collect();
    
    // Sequential processing
    let start = std::time::Instant::now();
    let sequential_result: Vec<u64> = data
        .iter()
        .map(|&x| expensive_computation(x))
        .collect();
    let sequential_time = start.elapsed();
    
    // Parallel processing with Rayon
    let start = std::time::Instant::now();
    let parallel_result: Vec<u64> = data
        .par_iter()
        .map(|&x| expensive_computation(x))
        .collect();
    let parallel_time = start.elapsed();
    
    println!("Sequential time: {:?}", sequential_time);
    println!("Parallel time: {:?}", parallel_time);
    println!("Speedup: {:.2}x", 
             sequential_time.as_millis() as f64 / parallel_time.as_millis() as f64);
    
    // Verify results are identical
    assert_eq!(sequential_result, parallel_result);
    println!("Results verified identical");
}
```

Rayon automatically parallelizes iterators by replacing iter() with  
par_iter(). It handles thread management, work distribution, and  
result collection, making parallel processing simple and efficient.

## Async basic futures

Futures represent values that will be available in the future,  
enabling non-blocking asynchronous programming.

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll};
use std::time::{Duration, Instant};

struct Timer {
    deadline: Instant,
}

impl Timer {
    fn new(duration: Duration) -> Self {
        Timer {
            deadline: Instant::now() + duration,
        }
    }
}

impl Future for Timer {
    type Output = ();
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        if Instant::now() >= self.deadline {
            Poll::Ready(())
        } else {
            // In a real async runtime, this would register the waker
            cx.waker().wake_by_ref();
            Poll::Pending
        }
    }
}

async fn delayed_greeting(name: &str, delay_ms: u64) -> String {
    Timer::new(Duration::from_millis(delay_ms)).await;
    format!("Hello, {}!", name)
}

// Simple executor for demonstration
fn block_on<F: Future>(mut future: F) -> F::Output {
    use std::task::{RawWaker, RawWakerVTable, Waker};
    
    let mut future = Box::pin(future);
    let waker = unsafe {
        Waker::from_raw(RawWaker::new(
            std::ptr::null(),
            &RawWakerVTable::new(
                |_| RawWaker::new(std::ptr::null(), &VTABLE),
                |_| {}, |_| {}, |_| {},
            ),
        ))
    };
    let mut context = Context::from_waker(&waker);
    
    loop {
        match future.as_mut().poll(&mut context) {
            Poll::Ready(result) => return result,
            Poll::Pending => std::thread::sleep(Duration::from_millis(1)),
        }
    }
}

const VTABLE: RawWakerVTable = RawWakerVTable::new(
    |_| RawWaker::new(std::ptr::null(), &VTABLE),
    |_| {}, |_| {}, |_| {},
);

fn main() {
    let greeting = block_on(delayed_greeting("Alice", 100));
    println!("{}", greeting);
}
```

Futures are lazy and don't execute until polled by an executor.  
The poll method returns Ready when complete or Pending when more  
time is needed. This enables efficient cooperative multitasking.

## Async task spawning

Async runtimes enable spawning concurrent tasks that run cooperatively  
on the same thread or across multiple threads.

```rust
// Note: This example requires tokio = "1.0" in Cargo.toml
use tokio::time::{sleep, Duration};

async fn task_with_id(id: u32, duration_ms: u64) -> u32 {
    println!("Task {} starting ({}ms)", id, duration_ms);
    sleep(Duration::from_millis(duration_ms)).await;
    println!("Task {} completed", id);
    id * 2
}

#[tokio::main]
async fn main() {
    let mut handles = vec![];
    
    // Spawn multiple async tasks
    for i in 1..=5 {
        let handle = tokio::spawn(task_with_id(i, i * 100));
        handles.push(handle);
    }
    
    // Collect results as they complete
    for handle in handles {
        match handle.await {
            Ok(result) => println!("Task result: {}", result),
            Err(e) => eprintln!("Task failed: {}", e),
        }
    }
    
    println!("All async tasks completed");
}
```

Tokio::spawn creates new async tasks that run concurrently.  
Unlike threads, tasks are lightweight and cooperatively scheduled,  
making it efficient to spawn thousands of concurrent tasks.

## Async channels for communication

Async channels enable communication between async tasks without  
blocking the async runtime.

```rust
use tokio::sync::mpsc;
use tokio::time::{sleep, Duration};

async fn producer(mut tx: mpsc::Sender<String>, id: u32) {
    for i in 1..=5 {
        let message = format!("Producer {}: Message {}", id, i);
        
        if let Err(e) = tx.send(message).await {
            eprintln!("Producer {} send error: {}", id, e);
            break;
        }
        
        sleep(Duration::from_millis(100)).await;
    }
    println!("Producer {} finished", id);
}

async fn consumer(mut rx: mpsc::Receiver<String>) {
    while let Some(message) = rx.recv().await {
        println!("Received: {}", message);
        sleep(Duration::from_millis(50)).await;
    }
    println!("Consumer finished");
}

#[tokio::main]
async fn main() {
    let (tx, rx) = mpsc::channel(10);
    
    // Spawn consumer
    let consumer_handle = tokio::spawn(consumer(rx));
    
    // Spawn multiple producers
    let mut producer_handles = vec![];
    for id in 1..=3 {
        let tx_clone = tx.clone();
        let handle = tokio::spawn(producer(tx_clone, id));
        producer_handles.push(handle);
    }
    
    // Drop original sender
    drop(tx);
    
    // Wait for all producers
    for handle in producer_handles {
        handle.await.unwrap();
    }
    
    // Wait for consumer
    consumer_handle.await.unwrap();
}
```

Async channels work like regular channels but don't block the  
async runtime. They integrate seamlessly with async/await syntax  
for efficient message passing between async tasks.

## Select for async operations

Select allows racing multiple async operations and handling  
whichever completes first, enabling timeouts and cancellation.

```rust
use tokio::time::{sleep, Duration, timeout};
use tokio::select;

async fn slow_operation() -> &'static str {
    sleep(Duration::from_millis(1000)).await;
    "Slow operation completed"
}

async fn fast_operation() -> &'static str {
    sleep(Duration::from_millis(100)).await;
    "Fast operation completed"
}

async fn user_input() -> &'static str {
    sleep(Duration::from_millis(500)).await;
    "User input received"
}

#[tokio::main]
async fn main() {
    println!("=== Select first to complete ===");
    
    select! {
        result = fast_operation() => {
            println!("Fast won: {}", result);
        }
        result = slow_operation() => {
            println!("Slow won: {}", result);
        }
        result = user_input() => {
            println!("User won: {}", result);
        }
    }
    
    println!("\n=== Timeout example ===");
    
    match timeout(Duration::from_millis(200), slow_operation()).await {
        Ok(result) => println!("Completed: {}", result),
        Err(_) => println!("Operation timed out"),
    }
    
    match timeout(Duration::from_millis(200), fast_operation()).await {
        Ok(result) => println!("Completed: {}", result),
        Err(_) => println!("Operation timed out"),
    }
}
```

Select provides non-deterministic choice between async operations.  
This enables implementing timeouts, cancellation, and event-driven  
programming patterns efficiently.

## Async mutex for shared state

Async mutexes provide exclusive access to shared state without  
blocking the async runtime, maintaining async task efficiency.

```rust
use tokio::sync::Mutex;
use tokio::time::{sleep, Duration};
use std::sync::Arc;

#[derive(Debug)]
struct SharedCounter {
    value: i32,
}

impl SharedCounter {
    fn new() -> Self {
        Self { value: 0 }
    }
    
    async fn increment(&mut self) {
        self.value += 1;
        // Simulate some async work
        sleep(Duration::from_millis(10)).await;
    }
    
    fn get(&self) -> i32 {
        self.value
    }
}

async fn worker(id: u32, counter: Arc<Mutex<SharedCounter>>) {
    for i in 1..=5 {
        {
            let mut guard = counter.lock().await;
            guard.increment().await;
            println!("Worker {}: incremented to {} (iteration {})", 
                     id, guard.get(), i);
        } // Lock released here
        
        // Do some other async work without holding the lock
        sleep(Duration::from_millis(50)).await;
    }
    println!("Worker {} completed", id);
}

#[tokio::main]
async fn main() {
    let counter = Arc::new(Mutex::new(SharedCounter::new()));
    let mut handles = vec![];
    
    // Spawn multiple async workers
    for id in 1..=3 {
        let counter_clone = Arc::clone(&counter);
        let handle = tokio::spawn(worker(id, counter_clone));
        handles.push(handle);
    }
    
    // Wait for all workers to complete
    for handle in handles {
        handle.await.unwrap();
    }
    
    let final_value = counter.lock().await.get();
    println!("Final counter value: {}", final_value);
}
```

Async mutexes yield control when waiting for locks instead of  
blocking threads. This maintains async runtime efficiency and  
prevents deadlocks in async contexts.

## Lock-free data structures

Lock-free data structures use atomic operations to achieve  
thread safety without locks, providing better performance  
under high contention.

```rust
use std::sync::atomic::{AtomicPtr, Ordering};
use std::sync::Arc;
use std::thread;
use std::ptr;

struct Node<T> {
    data: T,
    next: *mut Node<T>,
}

struct LockFreeStack<T> {
    head: AtomicPtr<Node<T>>,
}

impl<T> LockFreeStack<T> {
    fn new() -> Self {
        Self {
            head: AtomicPtr::new(ptr::null_mut()),
        }
    }
    
    fn push(&self, data: T) {
        let new_node = Box::into_raw(Box::new(Node {
            data,
            next: ptr::null_mut(),
        }));
        
        loop {
            let current_head = self.head.load(Ordering::Acquire);
            unsafe {
                (*new_node).next = current_head;
            }
            
            // Try to update head to point to new node
            match self.head.compare_exchange_weak(
                current_head,
                new_node,
                Ordering::Release,
                Ordering::Relaxed,
            ) {
                Ok(_) => break, // Success
                Err(_) => continue, // Retry
            }
        }
    }
    
    fn pop(&self) -> Option<T> {
        loop {
            let current_head = self.head.load(Ordering::Acquire);
            if current_head.is_null() {
                return None;
            }
            
            let next = unsafe { (*current_head).next };
            
            match self.head.compare_exchange_weak(
                current_head,
                next,
                Ordering::Release,
                Ordering::Relaxed,
            ) {
                Ok(_) => {
                    let data = unsafe { Box::from_raw(current_head).data };
                    return Some(data);
                }
                Err(_) => continue, // Retry
            }
        }
    }
}

unsafe impl<T: Send> Send for LockFreeStack<T> {}
unsafe impl<T: Send> Sync for LockFreeStack<T> {}

fn main() {
    let stack = Arc::new(LockFreeStack::new());
    let mut handles = vec![];
    
    // Spawn producer threads
    for i in 1..=3 {
        let stack_clone = Arc::clone(&stack);
        let handle = thread::spawn(move || {
            for j in 1..=5 {
                let value = i * 10 + j;
                stack_clone.push(value);
                println!("Thread {} pushed {}", i, value);
            }
        });
        handles.push(handle);
    }
    
    // Wait for producers
    for handle in handles {
        handle.join().unwrap();
    }
    
    // Pop all values
    let mut values = vec![];
    while let Some(value) = stack.pop() {
        values.push(value);
    }
    
    values.sort();
    println!("Popped values: {:?}", values);
}
```

Lock-free data structures use compare-and-swap operations to  
achieve thread safety without blocking. They provide better  
performance under high contention but are more complex to implement.

## Deadlock prevention patterns

Proper lock ordering and timeout mechanisms prevent deadlocks  
in complex multi-threaded applications.

```rust
use std::sync::{Arc, Mutex, TryLockError};
use std::thread;
use std::time::Duration;

struct Account {
    id: u32,
    balance: Mutex<u32>,
}

impl Account {
    fn new(id: u32, balance: u32) -> Self {
        Self {
            id,
            balance: Mutex::new(balance),
        }
    }
}

// Good: Consistent lock ordering prevents deadlocks
fn transfer_with_ordering(from: &Account, to: &Account, amount: u32) -> Result<(), String> {
    // Always acquire locks in order of account ID
    let (first, second) = if from.id < to.id {
        (&from.balance, &to.balance)
    } else {
        (&to.balance, &from.balance)
    };
    
    let mut first_guard = first.lock().unwrap();
    let mut second_guard = second.lock().unwrap();
    
    // Determine which is from and which is to
    let (from_balance, to_balance) = if from.id < to.id {
        (&mut *first_guard, &mut *second_guard)
    } else {
        (&mut *second_guard, &mut *first_guard)
    };
    
    if *from_balance >= amount {
        *from_balance -= amount;
        *to_balance += amount;
        println!("Transferred {} from account {} to account {}", 
                 amount, from.id, to.id);
        Ok(())
    } else {
        Err("Insufficient funds".to_string())
    }
}

// Alternative: Use timeouts to avoid indefinite blocking
fn transfer_with_timeout(from: &Account, to: &Account, amount: u32) -> Result<(), String> {
    let timeout = Duration::from_millis(100);
    
    // Try to acquire both locks with timeout
    let from_guard = match from.balance.try_lock() {
        Ok(guard) => guard,
        Err(TryLockError::WouldBlock) => {
            thread::sleep(timeout);
            return Err("Could not acquire from lock".to_string());
        }
        Err(TryLockError::Poisoned(err)) => return Err(format!("Lock poisoned: {}", err)),
    };
    
    let mut to_guard = match to.balance.try_lock() {
        Ok(guard) => guard,
        Err(TryLockError::WouldBlock) => {
            return Err("Could not acquire to lock".to_string());
        }
        Err(TryLockError::Poisoned(err)) => return Err(format!("Lock poisoned: {}", err)),
    };
    
    let mut from_balance = from_guard;
    if *from_balance >= amount {
        *from_balance -= amount;
        *to_guard += amount;
        println!("Transferred {} from account {} to account {} (with timeout)", 
                 amount, from.id, to.id);
        Ok(())
    } else {
        Err("Insufficient funds".to_string())
    }
}

fn main() {
    let account1 = Arc::new(Account::new(1, 1000));
    let account2 = Arc::new(Account::new(2, 1000));
    let mut handles = vec![];
    
    // Spawn threads doing transfers in both directions
    for i in 1..=5 {
        let acc1 = Arc::clone(&account1);
        let acc2 = Arc::clone(&account2);
        
        let handle = thread::spawn(move || {
            if i % 2 == 0 {
                // Transfer from account1 to account2
                match transfer_with_ordering(&acc1, &acc2, 50) {
                    Ok(_) => println!("Thread {}: Transfer 1->2 successful", i),
                    Err(e) => println!("Thread {}: Transfer 1->2 failed: {}", i, e),
                }
            } else {
                // Transfer from account2 to account1
                match transfer_with_ordering(&acc2, &acc1, 30) {
                    Ok(_) => println!("Thread {}: Transfer 2->1 successful", i),
                    Err(e) => println!("Thread {}: Transfer 2->1 failed: {}", i, e),
                }
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Final balances:");
    println!("Account 1: {}", *account1.balance.lock().unwrap());
    println!("Account 2: {}", *account2.balance.lock().unwrap());
}
```

Deadlock prevention requires consistent lock ordering or timeout  
mechanisms. Always acquire locks in the same order across all  
threads, or use timeouts to avoid indefinite blocking.

## Parallel computation with divide-and-conquer

Divide-and-conquer algorithms naturally parallelize by processing  
subproblems independently and combining results.

```rust
use std::thread;
use std::sync::Arc;

fn sequential_sum(data: &[i32]) -> i64 {
    data.iter().map(|&x| x as i64).sum()
}

fn parallel_sum(data: Arc<Vec<i32>>, threshold: usize) -> i64 {
    if data.len() <= threshold {
        return sequential_sum(&data);
    }
    
    let mid = data.len() / 2;
    let left_data = Arc::new(data[..mid].to_vec());
    let right_data = Arc::new(data[mid..].to_vec());
    
    let left_handle = thread::spawn(move || {
        parallel_sum(left_data, threshold)
    });
    
    let right_handle = thread::spawn(move || {
        parallel_sum(right_data, threshold)
    });
    
    let left_sum = left_handle.join().unwrap();
    let right_sum = right_handle.join().unwrap();
    
    left_sum + right_sum
}

fn parallel_quicksort(mut data: Vec<i32>) -> Vec<i32> {
    if data.len() <= 1 {
        return data;
    }
    
    let pivot = data[data.len() / 2];
    let mut left = Vec::new();
    let mut middle = Vec::new();
    let mut right = Vec::new();
    
    for value in data {
        if value < pivot {
            left.push(value);
        } else if value == pivot {
            middle.push(value);
        } else {
            right.push(value);
        }
    }
    
    if left.len() > 100 && right.len() > 100 {
        // Parallel recursive calls for large partitions
        let left_handle = thread::spawn(move || parallel_quicksort(left));
        let right_handle = thread::spawn(move || parallel_quicksort(right));
        
        let mut sorted_left = left_handle.join().unwrap();
        let sorted_right = right_handle.join().unwrap();
        
        sorted_left.extend(middle);
        sorted_left.extend(sorted_right);
        sorted_left
    } else {
        // Sequential for small partitions
        let mut result = parallel_quicksort(left);
        result.extend(middle);
        result.extend(parallel_quicksort(right));
        result
    }
}

fn main() {
    // Test parallel sum
    let data: Vec<i32> = (1..=10000).collect();
    let data_arc = Arc::new(data.clone());
    
    let start = std::time::Instant::now();
    let sequential_result = sequential_sum(&data);
    let sequential_time = start.elapsed();
    
    let start = std::time::Instant::now();
    let parallel_result = parallel_sum(data_arc, 1000);
    let parallel_time = start.elapsed();
    
    println!("Sum results - Sequential: {}, Parallel: {}", 
             sequential_result, parallel_result);
    println!("Sum times - Sequential: {:?}, Parallel: {:?}", 
             sequential_time, parallel_time);
    
    // Test parallel quicksort
    let mut unsorted: Vec<i32> = (1..=1000).rev().collect();
    let original = unsorted.clone();
    
    let start = std::time::Instant::now();
    unsorted.sort();
    let sequential_sort_time = start.elapsed();
    
    let start = std::time::Instant::now();
    let parallel_sorted = parallel_quicksort(original);
    let parallel_sort_time = start.elapsed();
    
    println!("Sort verification: {}", unsorted == parallel_sorted);
    println!("Sort times - Sequential: {:?}, Parallel: {:?}", 
             sequential_sort_time, parallel_sort_time);
}
```

Divide-and-conquer algorithms split problems into independent  
subproblems that can be solved in parallel. Proper threshold  
values prevent overhead from creating too many small tasks.

## Actor pattern for message passing

The actor pattern encapsulates state and processes messages  
sequentially, providing safe concurrency without shared state.

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

#[derive(Debug)]
enum ActorMessage {
    GetValue(mpsc::Sender<i32>),
    Increment,
    Add(i32),
    Stop,
}

struct CounterActor {
    value: i32,
    receiver: mpsc::Receiver<ActorMessage>,
}

impl CounterActor {
    fn new() -> (Self, mpsc::Sender<ActorMessage>) {
        let (sender, receiver) = mpsc::channel();
        let actor = Self {
            value: 0,
            receiver,
        };
        (actor, sender)
    }
    
    fn run(mut self) {
        println!("Counter actor started");
        
        while let Ok(message) = self.receiver.recv() {
            match message {
                ActorMessage::GetValue(reply_to) => {
                    reply_to.send(self.value).unwrap();
                }
                ActorMessage::Increment => {
                    self.value += 1;
                    println!("Counter incremented to {}", self.value);
                }
                ActorMessage::Add(amount) => {
                    self.value += amount;
                    println!("Counter increased by {} to {}", amount, self.value);
                }
                ActorMessage::Stop => {
                    println!("Counter actor stopping with final value: {}", self.value);
                    break;
                }
            }
        }
    }
}

struct ActorHandle {
    sender: mpsc::Sender<ActorMessage>,
}

impl ActorHandle {
    fn new(sender: mpsc::Sender<ActorMessage>) -> Self {
        Self { sender }
    }
    
    fn get_value(&self) -> i32 {
        let (reply_sender, reply_receiver) = mpsc::channel();
        self.sender.send(ActorMessage::GetValue(reply_sender)).unwrap();
        reply_receiver.recv().unwrap()
    }
    
    fn increment(&self) {
        self.sender.send(ActorMessage::Increment).unwrap();
    }
    
    fn add(&self, amount: i32) {
        self.sender.send(ActorMessage::Add(amount)).unwrap();
    }
    
    fn stop(&self) {
        self.sender.send(ActorMessage::Stop).unwrap();
    }
}

fn main() {
    let (actor, sender) = CounterActor::new();
    let handle = ActorHandle::new(sender);
    
    // Spawn actor thread
    let actor_handle = thread::spawn(move || {
        actor.run();
    });
    
    // Spawn client threads
    let mut client_handles = vec![];
    
    for i in 1..=3 {
        let handle_clone = ActorHandle::new(handle.sender.clone());
        let client = thread::spawn(move || {
            for j in 1..=3 {
                handle_clone.increment();
                thread::sleep(Duration::from_millis(50));
                
                if j == 2 {
                    handle_clone.add(i * 10);
                }
                
                let current_value = handle_clone.get_value();
                println!("Client {}: Current value is {}", i, current_value);
            }
        });
        client_handles.push(client);
    }
    
    // Wait for clients to finish
    for client in client_handles {
        client.join().unwrap();
    }
    
    // Get final value and stop actor
    let final_value = handle.get_value();
    println!("Final value before stopping: {}", final_value);
    handle.stop();
    
    // Wait for actor to finish
    actor_handle.join().unwrap();
}
```

The actor pattern processes messages sequentially in isolation,  
eliminating race conditions. Each actor owns its state and  
communicates only through message passing, ensuring thread safety.

## Performance monitoring and profiling

Measuring performance helps identify bottlenecks and optimize  
concurrent programs for better throughput and latency.

```rust
use std::sync::{Arc, Mutex, atomic::{AtomicUsize, Ordering}};
use std::thread;
use std::time::{Duration, Instant};

struct PerformanceMonitor {
    operations_completed: AtomicUsize,
    start_time: Instant,
}

impl PerformanceMonitor {
    fn new() -> Self {
        Self {
            operations_completed: AtomicUsize::new(0),
            start_time: Instant::now(),
        }
    }
    
    fn record_operation(&self) {
        self.operations_completed.fetch_add(1, Ordering::Relaxed);
    }
    
    fn get_stats(&self) -> (usize, f64) {
        let operations = self.operations_completed.load(Ordering::Relaxed);
        let elapsed = self.start_time.elapsed().as_secs_f64();
        let ops_per_sec = operations as f64 / elapsed;
        (operations, ops_per_sec)
    }
}

fn benchmark_mutex_contention(threads: usize, operations_per_thread: usize) -> Duration {
    let counter = Arc::new(Mutex::new(0));
    let monitor = Arc::new(PerformanceMonitor::new());
    let mut handles = vec![];
    
    let start = Instant::now();
    
    for _ in 0..threads {
        let counter_clone = Arc::clone(&counter);
        let monitor_clone = Arc::clone(&monitor);
        
        let handle = thread::spawn(move || {
            for _ in 0..operations_per_thread {
                {
                    let mut val = counter_clone.lock().unwrap();
                    *val += 1;
                }
                monitor_clone.record_operation();
                
                // Simulate small amount of work
                thread::sleep(Duration::from_nanos(100));
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    let duration = start.elapsed();
    let (ops, ops_per_sec) = monitor.get_stats();
    
    println!("Mutex benchmark: {} threads, {} total ops, {:.2} ops/sec, {:?}",
             threads, ops, ops_per_sec, duration);
    
    duration
}

fn benchmark_atomic_operations(threads: usize, operations_per_thread: usize) -> Duration {
    let counter = Arc::new(AtomicUsize::new(0));
    let monitor = Arc::new(PerformanceMonitor::new());
    let mut handles = vec![];
    
    let start = Instant::now();
    
    for _ in 0..threads {
        let counter_clone = Arc::clone(&counter);
        let monitor_clone = Arc::clone(&monitor);
        
        let handle = thread::spawn(move || {
            for _ in 0..operations_per_thread {
                counter_clone.fetch_add(1, Ordering::Relaxed);
                monitor_clone.record_operation();
                
                // Simulate small amount of work
                thread::sleep(Duration::from_nanos(100));
            }
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    let duration = start.elapsed();
    let (ops, ops_per_sec) = monitor.get_stats();
    
    println!("Atomic benchmark: {} threads, {} total ops, {:.2} ops/sec, {:?}",
             threads, ops, ops_per_sec, duration);
    
    duration
}

fn main() {
    const THREADS: usize = 4;
    const OPS_PER_THREAD: usize = 10000;
    
    println!("=== Performance Comparison ===");
    
    // Benchmark mutex performance
    println!("\n--- Mutex Performance ---");
    let mutex_time = benchmark_mutex_contention(THREADS, OPS_PER_THREAD);
    
    // Benchmark atomic performance
    println!("\n--- Atomic Performance ---");
    let atomic_time = benchmark_atomic_operations(THREADS, OPS_PER_THREAD);
    
    // Compare results
    println!("\n--- Comparison ---");
    let speedup = mutex_time.as_nanos() as f64 / atomic_time.as_nanos() as f64;
    println!("Atomic operations are {:.2}x faster than mutex for this workload", speedup);
    
    // Thread scaling analysis
    println!("\n--- Thread Scaling Analysis ---");
    for thread_count in [1, 2, 4, 8] {
        let time = benchmark_atomic_operations(thread_count, OPS_PER_THREAD);
        let total_ops = thread_count * OPS_PER_THREAD;
        let ops_per_sec = total_ops as f64 / time.as_secs_f64();
        println!("Threads: {}, Throughput: {:.0} ops/sec", thread_count, ops_per_sec);
    }
}
```

Performance monitoring tracks key metrics like throughput and  
latency. Comparing different synchronization primitives helps  
choose the optimal approach for specific workloads and contention  
patterns.

## Thread pools for task management

Thread pools manage a fixed set of worker threads to process  
tasks efficiently without creating new threads for each task.

```rust
use std::sync::{mpsc, Arc, Mutex};
use std::thread;
use std::time::Duration;

type Job = Box<dyn FnOnce() + Send + 'static>;

struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

impl ThreadPool {
    fn new(size: usize) -> Self {
        assert!(size > 0);
        
        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));
        let mut workers = Vec::with_capacity(size);
        
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }
        
        ThreadPool { workers, sender }
    }
    
    fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());
        
        for worker in &mut self.workers {
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Self {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv();
            
            match job {
                Ok(job) => {
                    println!("Worker {} executing job", id);
                    job();
                }
                Err(_) => {
                    println!("Worker {} shutting down", id);
                    break;
                }
            }
        });
        
        Worker {
            id,
            thread: Some(thread),
        }
    }
}

fn main() {
    let pool = ThreadPool::new(4);
    
    // Submit tasks to the thread pool
    for i in 1..=10 {
        pool.execute(move || {
            println!("Task {} starting", i);
            thread::sleep(Duration::from_millis(500));
            println!("Task {} completed", i);
        });
    }
    
    // Give tasks time to complete
    thread::sleep(Duration::from_secs(3));
    
    println!("All tasks submitted, shutting down");
}
```

Thread pools provide controlled parallelism by reusing a fixed  
number of threads. This eliminates thread creation overhead and  
prevents resource exhaustion from creating too many threads.

## Fan-out/fan-in pattern

Fan-out distributes work to multiple workers, while fan-in  
collects results, enabling parallel processing pipelines.

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn worker(id: usize, input: mpsc::Receiver<i32>, output: mpsc::Sender<i32>) {
    while let Ok(value) = input.recv() {
        println!("Worker {} processing {}", id, value);
        
        // Simulate processing time
        thread::sleep(Duration::from_millis(100));
        
        let result = value * value;
        output.send(result).unwrap();
        
        println!("Worker {} completed {}, result: {}", id, value, result);
    }
    println!("Worker {} finished", id);
}

fn main() {
    let num_workers = 3;
    let mut input_senders = vec![];
    let (result_tx, result_rx) = mpsc::channel();
    
    // Create workers (fan-out)
    for id in 0..num_workers {
        let (input_tx, input_rx) = mpsc::channel();
        let result_tx_clone = result_tx.clone();
        
        // Spawn worker thread
        thread::spawn(move || {
            worker(id, input_rx, result_tx_clone);
        });
        
        input_senders.push(input_tx);
    }
    
    // Distribute work to workers
    let work_items = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    for (i, item) in work_items.into_iter().enumerate() {
        let worker_id = i % num_workers;
        input_senders[worker_id].send(item).unwrap();
    }
    
    // Close input channels
    drop(input_senders);
    drop(result_tx);
    
    // Collect results (fan-in)
    let mut results = vec![];
    while let Ok(result) = result_rx.recv() {
        results.push(result);
    }
    
    results.sort();
    println!("Final results: {:?}", results);
}
```

Fan-out/fan-in patterns enable parallel processing by distributing  
work across multiple workers and collecting results. This pattern  
is common in data processing pipelines and parallel algorithms.