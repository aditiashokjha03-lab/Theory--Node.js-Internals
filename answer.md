# Q: 1 Theory (Node.js Internals)

## Explain the following in detail (in your own words):
### 1. Node.js Architecture
* JavaScript Engine (V8)
* Node.js Core APIs
* Native bindings
* Event Loop

## Answer.
Node.js is designed to build scalable and high-performance applications using a unique architecture that combines:

- Event-driven, non-blocking I/O model: Node.js uses callbacks and promises to handle asynchronous tasks.
- Single-threaded JavaScript execution: The main thread runs JavaScript code, while background tasks are delegated to libuv and thread pools.

Core Components are:
1. V8 Engine
Executes JavaScript
2. Node Core APIs
`fs`, `http`, `os`, `path`
3. Native Bindings
Bridge between JS and C/C++
4. Event Loop
Manages async execution

* **JavaScript Engine (V8)**
  - Developed by Google for Chrome.
  - Compiles JavaScript into machine code for fast execution.
  - Provides features like garbage collection, JIT compilation, and ECMAScript compliance.

* **Node.js Core APIs**
  - Modules like `fs`, `http`, `net`, `crypto`.
  - Abstract complex operations (file system, networking, encryption) into simple JavaScript functions.

* **Native Bindings**
  - Connect JavaScript APIs to C/C++ implementations.
  - Example: `fs.readFile()` in JavaScript internally calls C++ code via bindings.

* **Event Loop**
  - Central mechanism that schedules and executes tasks.
  - Continuously checks for pending callbacks, timers, and I/O events.
  - Ensures non-blocking execution by deferring heavy tasks to libuv.

<img src="https://d14lhgoyljo1xt.cloudfront.net/assets/b2c610ff35_img-nodejs-architecture0.webp">

### 2. libuv
* What is libuv?
* Why Node.js needs libuv
* Responsibilities of libuv

## Answer.
- **What is libuv?** 
A C library that provides cross-platform asynchronous I/O.
- **Why Node.js needs libuv?** 
JavaScript alone cannot handle low-level OS operations like networking or file I/O. Therefore, Node.js needs libuv.
- **Responsibilities:**
  - Event loop management.
  - Thread pool handling.
  - Asynchronous I/O (network, file system).
  - Timers and signals.

<img src="https://miro.medium.com/1%2Axm_WajiPlaOeJWcqgJb1xQ.png">

### 3. Thread Pool
* What is a thread pool?
* Why Node.js uses a thread pool
* Which operations are handled by the thread pool

## Answer.
- **What is a thread pool?**
A set of background threads managed by libuv.
- **Why Node.js uses it?**
To offload blocking operations (like file I/O, DNS lookups, crypto).
- **Operations handled:**
  - File system operations (`fs.readFile`, `fs.writeFile`).
  - DNS lookups.
  - Compression (`zlib`).
  - Crypto functions (`pbkdf2`, `scrypt`).
```
const crypto = require("crypto");

crypto.pbkdf2("pass", "salt", 100000, 64, "sha512", () => {
  console.log("Crypto done");
});
```

### 4. Worker Threads
* What are worker threads?
* Why are worker threads needed?
* Difference between thread pool and worker threads

## Answer.
- **What are worker threads?**
Separate threads that run JavaScript code in parallel.
```
const { Worker } = require("worker_threads");
```
- **Why needed?** 
For CPU-intensive tasks (e.g., image processing, large computations) that would block the event loop.
- **Difference from Thread Pool:**
  - Thread Pool: Handles C/C++ async tasks (I/O bound).
  - Worker Threads: Execute JavaScript code (CPU bound).


### 5. Event Loop Queues
* Macro Task Queue
* Micro Task Queue
* Execution priority between them
* Examples of tasks in each queue

## Answer.
Node.js uses two main types of task queues to manage asynchronous execution:

<img src="https://media.geeksforgeeks.org/wp-content/uploads/20200224050909/nodejs2.png">

1. **Macro Task Queue**
It contains tasks scheduled by timers, I/O, and other system-level operations.
* Examples:
  - `setTimeout()`
  - `setInterval()`
  - `setImmediate()`
  - I/O callbacks (e.g., `fs.readFile`)
  - UI rendering (in browsers)

2. **Micro Task Queue**
It contains tasks that are scheduled to run immediately after the current operation, before any macro tasks.
* Examples:
  - `process.nextTick()` (Node.js specific)
  - Promise callbacks (`.then`, `.catch`, `.finally`)
  - `queueMicrotask()`

**Execution Priority between them-**
- Microtasks have higher priority than macrotasks.
- After executing a macro task, the event loop empties the entire microtask queue before moving to the next macro task.
- This ensures that promise resolutions and critical updates happen as soon as possible.

<img src="https://blog.softbinator.com/wp-content/uploads/14_Microtask-vs.-Macrotask-in-JavaScript-What-are-The-Differences.png">

**Examples of tasks in each queue includes:**
  - Macro tasks: `setTimeout(() => console.log("timeout"))`
  - Micro tasks: `Promise.resolve().then(() => console.log("promise"))`

**Execution Order Example:**
```
console.log("Start");
setTimeout(() => console.log("Timeout"), 0);
Promise.resolve().then(() => console.log("Promise"));
console.log("End");
```

**Output would be**
```
Start
End
Promise
Timeout
```
