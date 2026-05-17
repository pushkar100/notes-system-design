<!-- TOC --><a name="web-workers"></a>
# Web Workers

<!-- TOC start (generated with https://github.com/derlin/bitdowntoc) -->

- [Web Workers](#web-workers)
   * [1. Introduction: The Need, What, Why, and How](#1-introduction-the-need-what-why-and-how)
   * [2. Set Up & Best Practices](#2-set-up-best-practices)
   * [3. Pros and Cons](#3-pros-and-cons)
   * [4. Practical Use Cases & When NOT to use them](#4-practical-use-cases-when-not-to-use-them)
   * [5. Common Design Requirements (Scenarios)](#5-common-design-requirements-scenarios)
      + [Scenario A: High-Concurrency Data Streaming (e.g., Bank Ledger or Stock Ticker)](#scenario-a-high-concurrency-data-streaming-eg-bank-ledger-or-stock-ticker)
      + [Scenario B: Zero-Copy Image Processing (Transferable Objects)](#scenario-b-zero-copy-image-processing-transferable-objects)
   * [6. Modern Tools That Make It Easy](#6-modern-tools-that-make-it-easy)

<!-- TOC end -->

<!-- TOC --><a name="1-introduction-the-need-what-why-and-how"></a>
## 1. Introduction: The Need, What, Why, and How

**The Problem It Solves (The Need)**
JavaScript, by design, is strictly **single-threaded**. The Event Loop, the Call Stack, and the Critical Rendering Path (CRP) all share the exact same Main Thread. If you execute a mathematically heavy piece of code (like parsing a massive JSON payload or processing image pixels), the Call Stack becomes blocked. While the Call Stack is blocked, the browser absolutely cannot paint the screen or respond to user clicks. This results in UI freezing, jank, and a terrible user experience.

**What Web Workers Are**
Web Workers are the browser's API for true hardware multithreading. They allow you to spin up independent JavaScript execution threads running in the background, isolated from the Main Thread. 

**Why Use Them**
To protect the Main Thread. By offloading heavy, synchronous computational work to a Web Worker, the Main Thread remains entirely free to handle CSS animations, layout recalculations, and user interactions smoothly at 60fps.

**How They Work**
Workers run in a separate global context. They **cannot access the DOM** (`document.getElementById` will fail) or the `window` object. They communicate with the Main Thread exclusively through a message-passing system.

**ASCII Diagram: The Threading Model**

```text
=============================================================================
                     SINGLE THREADED (The Problem)
=============================================================================
Time -> 
Main Thread:  [ UI Render ] [ Heavy Math Loop (3 seconds) ] [ UI Render ]
                                     ^ 
                      (Browser is completely frozen here)
                      (User clicks do nothing)

=============================================================================
                     WEB WORKERS (The Solution)
=============================================================================
Time ->
Main Thread:  [ UI Render ] [ Send Data ] [ UI Render ] [ Recv Data & Render]
                                 |                               ^
                                 v                               |
Worker Thread:              [ Heavy Math Loop (3 seconds) ] -----+
                             (Running in parallel on a separate CPU core)
=============================================================================
```

<!-- TOC --><a name="2-set-up-best-practices"></a>
## 2. Set Up & Best Practices

Setting up a basic Web Worker requires two files: your main application file and a dedicated worker file.

**The Code Implementation:**

```javascript
// ==========================================
// 1. main.js (Running on the Main Thread)
// ==========================================

// 1. Instantiate the worker (points to a separate file)
const worker = new Worker('heavy-calculator.js');

// 2. Listen for messages coming FROM the worker
worker.onmessage = (event) => {
  console.log('Result from worker:', event.data);
  document.getElementById('result').textContent = event.data;
  
  // Best Practice: Terminate the worker when the job is done 
  // to free up memory if it won't be used again soon.
  worker.terminate(); 
};

// Listen for errors inside the worker
worker.onerror = (error) => {
  console.error('Worker failed:', error.message);
};

// 3. Send a message TO the worker to start the job
console.log('Sending workload to worker...');
worker.postMessage({ command: 'calculate', payload: 1000000000 });


// ==========================================
// 2. heavy-calculator.js (Running on the Worker Thread)
// ==========================================

// 1. Listen for messages coming FROM the main thread
self.onmessage = (event) => {
  const { command, payload } = event.data;
  
  if (command === 'calculate') {
    // 2. Do the heavy lifting
    let result = 0;
    for (let i = 0; i < payload; i++) {
      result += i;
    }
    
    // 3. Send the final result back to the main thread
    self.postMessage(result);
  }
};
```

**Best Practices for Production:**
1.  **Lifecycle Management:** Workers consume substantial memory (each one spins up its own V8 JavaScript engine instance). Always call `worker.terminate()` when they are no longer needed, or reuse a single "worker pool" rather than spinning up a new one for every click.
2.  **Avoid the Serialization Bottleneck:** When you use `postMessage(data)`, the browser uses the **Structured Clone Algorithm** to copy the data. If you pass a 50MB JSON object, the copying process itself will freeze the Main Thread, defeating the purpose of the worker. 
3.  **Use Transferable Objects:** For massive data (like image buffers), use `ArrayBuffer`. You can instantly *transfer* ownership of the buffer to the worker without copying it.

<!-- TOC --><a name="3-pros-and-cons"></a>
## 3. Pros and Cons

**Pros:**
*   **True Parallelism:** Actually utilizes multi-core processors.
*   **Jank-Free UI:** Keeps the Critical Rendering Path unblocked, guaranteeing a smooth Interaction to Next Paint (INP) metric.
*   **Network Capabilities:** Workers can use `fetch` and `XMLHttpRequest`, making them great for background data syncing.

**Cons:**
*   **No DOM Access:** You cannot manipulate HTML or read UI state directly. The worker must ask the Main Thread to do it.
*   **High Memory Footprint:** Spawning a worker is expensive. You shouldn't spawn 100 workers for tiny tasks.
*   **Message Overhead:** The data copying mechanism (`postMessage`) can be slow for deeply nested, massive JavaScript objects.

<!-- TOC --><a name="4-practical-use-cases-when-not-to-use-them"></a>
## 4. Practical Use Cases & When NOT to use them

**Excellent Use Cases:**
*   **Client-Side Search / Filtering:** Searching through a massive array of 50,000 products for a typeahead search bar.
*   **Data Parsing:** Parsing megabytes of CSV or JSON data returned from an analytics API before displaying it on a dashboard.
*   **Media Manipulation:** Processing audio streams, applying Instagram-style filters to an image canvas, or generating PDFs on the client.
*   **Heavy Math/Cryptography:** Hashing passwords, generating encryption keys, or client-side machine learning models.

**When NOT to use them:**
*   **Simple API Calls:** `fetch` is already asynchronous. Moving a simple `fetch('/api/users')` into a worker adds unnecessary complexity.
*   **DOM Manipulation:** If your task requires constantly reading the DOM (like tracking mouse movements), the overhead of messaging the Main Thread back and forth will be slower than just doing it on the Main Thread.
*   **Tiny Computations:** If a task takes 2 milliseconds, the time it takes to serialize the data, send it to the worker, and send it back will take longer than 2ms. Only use workers for tasks that take roughly 50ms or more (Long Tasks).

<!-- TOC --><a name="5-common-design-requirements-scenarios"></a>
## 5. Common Design Requirements (Scenarios)

When architecting high-performance web applications, you will encounter specific patterns where workers are mandatory.

<!-- TOC --><a name="scenario-a-high-concurrency-data-streaming-eg-bank-ledger-or-stock-ticker"></a>
### Scenario A: High-Concurrency Data Streaming (e.g., Bank Ledger or Stock Ticker)
**Requirement:** The frontend receives a firehose of WebSocket data (thousands of events per second). The data needs to be parsed, sorted, and aggregated before updating the UI chart. Doing this on the Main Thread will crash the browser.

```text
=============================================================================
                     SCENARIO: DATA AGGREGATION
=============================================================================
[ WebSocket Server ] 
        | (1000 msgs/sec)
        v
+-----------------------+      (Raw Data)     +--------------------------+
| MAIN THREAD           | ------------------> | WORKER THREAD            |
| (Only handles UI)     |                     | - Parses JSON            |
|                       |                     | - Aggregates totals      |
|                       | <------------------ | - Sorts arrays           |
+-----------------------+    (Cleaned Data)   +--------------------------+
        |                    every 500ms
        v
 [ Update React/Vue State ]
=============================================================================
```

<!-- TOC --><a name="scenario-b-zero-copy-image-processing-transferable-objects"></a>
### Scenario B: Zero-Copy Image Processing (Transferable Objects)
**Requirement:** Uploading a large image, applying a watermark/filter client-side, and then showing the preview. 

To avoid the structured clone bottleneck on a 20MB image, we use an `ArrayBuffer` and transfer ownership.

```javascript
// Main Thread
const buffer = new ArrayBuffer(20000000); // 20MB buffer

// Syntax: postMessage(message, [transferableList])
// The Main Thread instantly loses access to 'buffer'. It now belongs to the worker.
// This takes 0ms because no data is copied, only the memory pointer is moved.
worker.postMessage({ type: 'process_image', data: buffer }, [buffer]);
```

<!-- TOC --><a name="6-modern-tools-that-make-it-easy"></a>
## 6. Modern Tools That Make It Easy

Writing raw `postMessage` handlers gets very messy in complex applications due to the lack of strong typing and request/response matching. Modern tooling abstracts this away.

**1. Comlink (by Google Chrome Labs)**
This is the absolute gold standard for working with Web Workers. It turns the awkward message-passing API into a standard RPC (Remote Procedure Call) system. It makes worker functions look like normal asynchronous functions on the Main Thread.

```javascript
// Using Comlink: Notice there is no postMessage or onmessage!
import * as Comlink from 'comlink';

// You just await the worker as if it were a normal class/function
const heavyService = Comlink.wrap(new Worker('worker.js'));
const result = await heavyService.calculateMassiveData(payload);
```

**2. Vite & Webpack (Built-in Support)**
Modern bundlers have first-class support for workers. You no longer need to worry about hosting the worker file separately. You can import them directly in your JavaScript, and the bundler handles the file pathing and chunking.

```javascript
// Vite explicitly supports this syntax
import MyWorker from './compute.js?worker';
const worker = new MyWorker();
```

**3. React / Vue Hooks**
There are many libraries (like `react-use`) that provide a `useWebWorker` hook, allowing you to seamlessly integrate background processing into your component state lifecycle without writing boilerplate cleanup code.
