# JavaScript Fundamentals

## Table of Contents

- [Asynchronous Programming](#asynchronous-programming)
- [Event Loop, Micro/Macro Tasks](#event-loop-micromacro-tasks)
- [Promises, async/await, Error Handling](#promises-asyncawait-error-handling)
- [Callback Hell vs. Promise Chaining](#callback-hell-vs-promise-chaining)
  - Example: Predict output of mixed setTimeout/Promise code
- [ES6+ Features](#es6-features)
  - [Destructuring](#destructuring)
  - [Spread/Rest Operators](#spreadrest-operators)
  - [Arrow Functions](#arrow-functions)
  - [Template Literals](#template-literals)
- [WeakMap/WeakSet](#weakmapweakset) *(common gap)*
- [Modules (ES6 vs. CommonJS)](#modules-es6-vs-commonjs)
- [Functions & Scopes](#functions--scopes)
  - [Closures](#closures)
  - [Currying](#currying)
  - [`this` Context](#this-context)
  - [Scopes](#scopes)
  - [Recursion](#recursion) *(e.g., deep object cloning)*
- [OOP & Prototypes](#oop--prototypes)
  - [Prototypal Inheritance](#prototypal-inheritance)
  - [ES6 Classes](#es6-classes)
  - [Inheritance](#inheritance)
  - [Singleton Pattern](#singleton-pattern) *(often missed)*
- [Data Structures](#data-structures)
  - [Array Methods](#array-methods) *(map, reduce, filter, sort)*
  - [Objects vs. Maps/Sets](#objects-vs-mapssets)
- [Coding Task: Group Books by Decade](#coding-task-group-books-by-decade)

---

## Asynchronous Programming

### What is Asynchronous Programming?

**Asynchronous programming** is a programming paradigm that allows operations to run independently of the main program flow. Instead of waiting (blocking) for long-running operations to complete, the program can continue executing other code, and be notified when the operation finishes.

**Key Concepts**:

- **Synchronous (Blocking)**: Operations execute sequentially, one after another. Each operation must complete before the next one starts.
- **Asynchronous (Non-blocking)**: Operations can start and the program continues executing without waiting for completion. When the operation finishes, a callback, promise, or event notifies the program.

**Why Asynchronous Programming?**

In JavaScript, operations like:

- Network requests (fetch API calls)
- File system operations (Node.js)
- Database queries
- Timers (setTimeout, setInterval)
- User interactions (click events)

...can take significant time. Without asynchronicity, the entire application would freeze while waiting.

### JavaScript: Single-Threaded Nature

**What does Single-Threaded mean?**

JavaScript has **one call stack** and executes **one piece of code at a time** in a single thread. This means:

- Only one operation is being executed at any given moment
- Code cannot truly run in parallel within the same JavaScript context
- If an operation takes a long time, it blocks all other operations

**Example of Blocking Code**:

```javascript
console.log('Start');
// Simulate heavy computation
for (let i = 0; i < 5000000000; i++) {} // Blocks for several seconds
console.log('End'); // Won't execute until loop completes
// UI freezes, no clicks or inputs work
```

**Single-Threaded vs Multi-Threaded**:

| Single-Threaded (JavaScript) | Multi-Threaded (Java, C++) |
|------------------------------|----------------------------|
| One execution context | Multiple execution contexts |
| Simpler, no race conditions | Complex synchronization needed |
| No parallel computation | True parallel execution |
| Uses async patterns for concurrency | Uses threads for parallelism |

### How JavaScript Achieves Concurrency

Despite being single-threaded, JavaScript achieves **concurrency** (not parallelism) through:

**1. Event Loop**: Manages execution of asynchronous operations
**2. Web APIs / C++ APIs**: Browser/Node.js provides APIs that run outside JavaScript's single thread
**3. Callback Queue**: Stores callbacks waiting to be executed

**Concurrency vs Parallelism**:

- **Concurrency**: Multiple operations making progress in overlapping time periods (JavaScript's approach)
- **Parallelism**: Multiple operations executing simultaneously on different CPU cores (not possible in standard JavaScript)

**Visualization**:

```javascript
console.log('1'); // Executes in JavaScript thread

setTimeout(() => {
  console.log('2'); // Executes in JavaScript thread (after Web API timer completes)
}, 1000);

console.log('3'); // Executes in JavaScript thread

// Output: 1, 3, 2
// The timer runs in a Web API (outside JS thread) while JS continues execution
```

### Web Workers: True Parallelism in JavaScript

**Web Workers** are the only way to achieve true parallel execution in JavaScript. They run JavaScript code in **separate threads**, enabling CPU-intensive tasks without blocking the main thread.

**How Web Workers Work**:

1. Main thread creates a worker thread
2. Worker runs in complete isolation (separate global scope)
3. Communication happens via message passing (no shared memory)
4. Worker cannot access DOM (security/safety)

**Creating a Web Worker**:

```javascript
// main.js (Main Thread)
const worker = new Worker('worker.js');

// Send data to worker
worker.postMessage({ numbers: [1, 2, 3, 4, 5] });

// Receive result from worker
worker.addEventListener('message', (event) => {
  console.log('Result from worker:', event.data); // 15
});

// Terminate worker when done
worker.terminate();
```

```javascript
// worker.js (Worker Thread)
self.addEventListener('message', (event) => {
  const numbers = event.data.numbers;
  
  // Perform CPU-intensive calculation
  const sum = numbers.reduce((acc, n) => acc + n, 0);
  
  // Send result back to main thread
  self.postMessage(sum);
});
```

**When to Use Web Workers**:

- Heavy computational tasks (image processing, data parsing, encryption)
- Operations that would freeze the UI
- Background data processing
- Complex algorithms (sorting large datasets, scientific calculations)

**Limitations**:

- No DOM access
- No access to window object
- Communication overhead (message serialization/deserialization)
- More complex debugging

### Non-Blocking I/O

**What is I/O (Input/Output)?**

Operations that interact with external systems:

- Reading/writing files
- Network requests
- Database queries
- Reading sensors, etc.

**Blocking I/O** (Traditional approach):

```javascript
// Pseudocode - This blocks
const data = readFileSync('large-file.txt'); // Waits here until complete
console.log(data);
console.log('Continue...'); // Can't execute until readFileSync finishes
```

**Non-Blocking I/O** (JavaScript approach):

```javascript
// This doesn't block
readFile('large-file.txt', (data) => {
  console.log(data);
});
console.log('Continue...'); // Executes immediately
// Output: "Continue..." then file data
```

**How Non-Blocking I/O Works**:

1. JavaScript delegates I/O operation to system APIs (libuv in Node.js, browser APIs)
2. These APIs handle the operation in the background
3. JavaScript thread continues executing other code
4. When I/O completes, a callback is queued
5. Event loop picks up the callback and executes it

**Benefits**:

- Better resource utilization
- Responsive applications
- Handle many operations concurrently
- Scalability (servers can handle thousands of requests)

### Key Mechanisms

- **Callbacks**: Functions passed as arguments, executed after async operation completes
- **Promises**: Objects representing eventual completion/failure of async operation
- **Async/Await**: Syntactic sugar over Promises for cleaner async code

### Critical for Senior Level

- Understanding that async code doesn't run in parallel (unless using Web Workers)
- Properly handling race conditions and concurrency
- Knowing when to use `Promise.all()`, `Promise.race()`, `Promise.allSettled()`, `Promise.any()`

```javascript
// Promise.all vs Promise.allSettled
const promises = [fetch('/api/1'), fetch('/api/2'), fetch('/api/3')];

// Fails if ANY promise rejects
await Promise.all(promises);

// Returns all results regardless of success/failure
await Promise.allSettled(promises);
// [{status: 'fulfilled', value: ...}, {status: 'rejected', reason: ...}]
```

---

## Event Loop, Micro/Macro Tasks

### What is the Event Loop?

The **Event Loop** is the mechanism that enables JavaScript to perform non-blocking operations despite being single-threaded. It's the coordinator that manages the execution of code, handles events, and processes queued tasks.

**Why does the Event Loop exist?**

Without the Event Loop:

- Asynchronous operations wouldn't be possible
- JavaScript would freeze during I/O operations
- No way to handle multiple operations concurrently
- User interfaces would be completely unresponsive

The Event Loop bridges the gap between JavaScript's single-threaded nature and the need for non-blocking, concurrent operations.

### Event Loop Architecture

The Event Loop consists of several key components that work together:

#### 1. Call Stack

**What it is**: A LIFO (Last In, First Out) data structure that tracks function execution.

**How it works**:

- When a function is called, it's pushed onto the stack
- When a function returns, it's popped from the stack
- JavaScript can only execute the function at the top of the stack
- If the stack is empty, JavaScript is idle

**Example**:

```javascript
function first() {
  console.log('First');
  second();
  console.log('First again');
}

function second() {
  console.log('Second');
}

first();

// Call Stack visualization:
// 1. first() pushed
// 2. console.log() pushed and executed (prints "First")
// 3. second() pushed
// 4. console.log() pushed and executed (prints "Second")
// 5. second() popped
// 6. console.log() pushed and executed (prints "First again")
// 7. first() popped
```

**Stack Overflow**: When too many functions are pushed without being popped (usually from infinite recursion):

```javascript
function recursiveFunction() {
  recursiveFunction(); // No base case - stack overflow!
}
```

#### 2. Web APIs / Node.js APIs

**What they are**: Separate APIs provided by the runtime environment (browser or Node.js) that handle asynchronous operations **outside** the JavaScript thread.

**Browser Web APIs**:

- `setTimeout` / `setInterval`
- `fetch` / `XMLHttpRequest`
- DOM events (`addEventListener`)
- `requestAnimationFrame`
- `IntersectionObserver`, `MutationObserver`

**Node.js APIs**:

- File system operations (`fs.readFile`)
- Network operations (`http.request`)
- Timers (`setTimeout`, `setImmediate`)
- `process.nextTick`

**Key Point**: These APIs run in **separate threads** managed by the browser or Node.js C++ layer, not in JavaScript's single thread.

#### 3. Task Queues (Macrotask Queue)

**What it is**: A FIFO (First In, First Out) queue that stores callbacks from completed asynchronous operations.

**Contents**:

- `setTimeout` / `setInterval` callbacks
- I/O operation callbacks
- `setImmediate` (Node.js)
- UI rendering tasks (browser)

**How it works**: When a Web API completes, it pushes the callback to this queue. The Event Loop processes these one at a time.

#### 4. Microtask Queue

**What it is**: A special queue with **higher priority** than the Macrotask Queue.

**Contents**:

- `Promise.then/catch/finally` callbacks
- `queueMicrotask()` callbacks
- `MutationObserver` callbacks
- `async/await` continuations

**Critical Difference**: ALL microtasks are processed before moving to the next macrotask.

#### 5. Render Queue (Browser Only)

**What it is**: The browser's rendering pipeline for updating the UI.

**When it happens**:

- After microtasks are processed
- Before next macrotask (typically)
- Approximately 60 times per second (60 FPS)

**Includes**:

- Style calculations
- Layout (reflow)
- Paint
- Composite layers

### Event Loop Execution Flow

**The complete cycle**:

1. Execute all code in the Call Stack (synchronous code)
2. Check Microtask Queue - execute ALL microtasks
3. Check if rendering is needed (browser only)
4. Take ONE task from Macrotask Queue
5. Execute that task (may add more tasks to queues)
6. Go back to step 2

**Visual Flow**:

```text
┌───────────────────────────┐
│   Execute Synchronous     │
│   Code (Call Stack)       │
└─────────────┬─────────────┘
              │
              ▼
┌───────────────────────────┐
│   Process ALL             │◄────┐
│   Microtasks              │     │
└─────────────┬─────────────┘     │
              │                   │
              ▼                   │
┌───────────────────────────┐     │
│   Render (if needed)      │     │
│   (Browser only)          │     │
└─────────────┬─────────────┘     │
              │                   │
              ▼                   │
┌───────────────────────────┐     │
│   Process ONE             │     │
│   Macrotask               │─────┘
└───────────────────────────┘
     (Loop continues)
```

**Execution Order**:
Synchronous code → **ALL Microtasks** → Render → **ONE Macrotask** → **ALL Microtasks** → Render → Repeat

### Task Priority: Microtasks vs Macrotasks

#### Microtasks (Higher Priority)

**Definition**: Tasks that should be executed immediately after the currently executing script, before any macrotasks or rendering.

**Sources**:

- `Promise.then/catch/finally`
- `queueMicrotask()`
- `async/await` continuations
- `MutationObserver` callbacks

**Key Behavior**: ALL queued microtasks are executed before moving to the next phase. If a microtask adds more microtasks, they are also executed in the same cycle.

**Warning - Microtask Infinite Loop**:

```javascript
function recursiveMicrotask() {
  queueMicrotask(() => {
    console.log('Microtask');
    recursiveMicrotask(); // Creates infinite loop - blocks everything!
  });
}

recursiveMicrotask(); // Never proceeds to macrotasks or rendering
```

#### Macrotasks (Lower Priority)

**Definition**: Tasks that are executed one per Event Loop cycle.

**Sources**:

- `setTimeout/setInterval`
- `setImmediate` (Node.js only)
- I/O operations (file system, network)
- UI rendering (browser only)
- Script loading

**Key Behavior**: Only ONE macrotask is executed per Event Loop cycle. After each macrotask, ALL microtasks are processed before the next macrotask.

### Browser vs Node.js Event Loop

While the fundamental concept is the same, there are important differences:

#### Browser Event Loop

**Characteristics**:

- Integrated with rendering pipeline
- Frame-based (targeting 60 FPS)
- Handles user interactions
- `requestAnimationFrame` runs before rendering

**Unique Features**:

- **Rendering** happens between Event Loop cycles (when needed)
- **`requestAnimationFrame`**: Runs before paint, ideal for animations
- **`requestIdleCallback`**: Runs during idle periods

**Example**:

```javascript
console.log('Script start');

setTimeout(() => console.log('setTimeout'), 0);

requestAnimationFrame(() => console.log('rAF'));

Promise.resolve().then(() => console.log('Promise'));

console.log('Script end');

// Output: 
// Script start
// Script end
// Promise
// rAF (before next paint)
// setTimeout
```

#### Node.js Event Loop

**Characteristics**:

- No rendering pipeline
- More complex with multiple phases
- Built on **libuv** (C library)
- Optimized for I/O operations

**Event Loop Phases** (Node.js specific):

```text
   ┌───────────────────────────┐
┌─>│           timers          │ - setTimeout, setInterval
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │ - I/O callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │ - Internal use only
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │ - Retrieve new I/O events
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │ - setImmediate callbacks
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──│      close callbacks      │ - socket.on('close', ...)
   └───────────────────────────┘
```

**Unique APIs**:

1. **`process.nextTick()`**: Executes immediately after current operation, before any I/O events or timers. Higher priority than regular microtasks.

```javascript
console.log('start');

setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));

Promise.resolve().then(() => console.log('Promise'));

process.nextTick(() => console.log('nextTick'));

console.log('end');

// Output:
// start
// end
// nextTick (highest priority)
// Promise (microtask)
// setTimeout (timer phase)
// setImmediate (check phase)
```

**`setImmediate()`**: Executes in the "check" phase, after I/O events. More predictable than `setTimeout(fn, 0)`.

**Key Differences Summary**:

| Feature | Browser | Node.js |
|---------|---------|---------|
| **Rendering** | Yes (integrated) | No |
| **Phases** | Simple (Macro/Micro) | Complex (6 phases) |
| **`setImmediate`** | ❌ Not available | ✅ Available |
| **`process.nextTick`** | ❌ Not available | ✅ Available |
| **`requestAnimationFrame`** | ✅ Available | ❌ Not available |
| **Primary use case** | UI interactions | I/O operations |
| **Underlying lib** | Browser engine | libuv (C) |

**Classic Interview Question**:

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

// Output: 1, 4, 3, 2
// Explanation: Sync code first (1, 4), then microtasks (3), then macrotasks (2)
```

**Complex Example**:

```javascript
console.log('start');

setTimeout(() => {
  console.log('setTimeout 1');
  Promise.resolve().then(() => console.log('promise 1'));
}, 0);

Promise.resolve()
  .then(() => {
    console.log('promise 2');
    setTimeout(() => console.log('setTimeout 2'), 0);
  });

console.log('end');

// Output: start, end, promise 2, setTimeout 1, promise 1, setTimeout 2
```

---

## Promises, async/await, Error Handling

### What are Promises?

A **Promise** is an object representing the eventual completion or failure of an asynchronous operation. It's a placeholder for a value that will be available in the future.

**Why were Promises created?**

Before Promises, JavaScript only had callbacks for asynchronous operations, which led to:

- **Callback Hell**: Deeply nested code that's hard to read and maintain
- **Inversion of Control**: You give your callback to a third-party function, trusting it will execute correctly
- **Limited Error Handling**: No unified way to catch errors across async operations
- **No Composition**: Difficult to combine multiple async operations

Promises solve these problems by:

- Providing a standard interface for async operations
- Enabling chainable operations (`.then()`)
- Offering unified error handling (`.catch()`)
- Supporting composition (`Promise.all()`, `Promise.race()`, etc.)
- Maintaining control flow in your code

### Promise States

A Promise exists in one of three mutually exclusive states:

**1. Pending**: Initial state, neither fulfilled nor rejected

```javascript
const promise = new Promise((resolve, reject) => {
  // Still executing, hasn't called resolve() or reject()
});
console.log(promise); // Promise { <pending> }
```

**2. Fulfilled**: Operation completed successfully, has a resulting value

```javascript
const promise = Promise.resolve(42);
console.log(promise); // Promise { <fulfilled>: 42 }
```

**3. Rejected**: Operation failed, has a reason for failure

```javascript
const promise = Promise.reject(new Error('Failed'));
console.log(promise); // Promise { <rejected>: Error: Failed }
```

**State Transitions**:

```text
         ┌─────────┐
         │ Pending │
         └────┬────┘
              │
      ┌───────┴────────┐
      │                │
      ▼                ▼
┌──────────┐    ┌──────────┐
│Fulfilled │    │ Rejected │
│(Success) │    │ (Failure)│
└──────────┘    └──────────┘
```

**Important**: Once a Promise is fulfilled or rejected (**settled**), its state cannot change. It's immutable.

```javascript
const promise = new Promise((resolve, reject) => {
  resolve('Success');
  reject('Failure'); // Ignored - Promise already fulfilled
  resolve('Another success'); // Also ignored
});
```

### Promise Creation

**Using the Promise Constructor**:

```javascript
const promise = new Promise((resolve, reject) => {
  // Perform async operation
  const success = Math.random() > 0.5;
  
  if (success) {
    resolve('Operation successful!'); // Fulfill the promise
  } else {
    reject(new Error('Operation failed')); // Reject the promise
  }
});
```

**Executor Function**: The function passed to `new Promise()` is called the **executor**. It:

- Runs immediately and synchronously
- Receives two functions: `resolve` and `reject`
- Should call one of them when the async operation completes

**Creating Immediately Resolved/Rejected Promises**:

```javascript
// Immediately fulfilled
const fulfilled = Promise.resolve(42);

// Immediately rejected
const rejected = Promise.reject(new Error('Failed'));

// From another promise
const fromPromise = Promise.resolve(existingPromise); // Returns same promise
```

### Promise Methods

#### Instance Methods

**`.then(onFulfilled, onRejected)`**: Handles fulfillment and rejection

```javascript
promise
  .then(
    value => console.log('Success:', value),
    error => console.error('Error:', error)
  );

// More commonly, only handle success in .then()
promise.then(value => console.log('Success:', value));
```

**`.catch(onRejected)`**: Handles rejection (syntactic sugar for `.then(null, onRejected)`)

```javascript
promise.catch(error => console.error('Error:', error));
```

**`.finally(onFinally)`**: Runs regardless of outcome (for cleanup)

```javascript
promise
  .then(value => processValue(value))
  .catch(error => handleError(error))
  .finally(() => {
    // Always runs - perfect for cleanup
    hideLoadingSpinner();
    closeConnection();
  });
```

#### Static Methods (on Promise class)

**`Promise.all(iterable)`**: Waits for ALL promises to fulfill (or ANY to reject)

```javascript
const [user, posts, comments] = await Promise.all([
  fetchUser(id),
  fetchPosts(id),
  fetchComments(id)
]);

// Rejects immediately if ANY promise rejects
// Returns array of results in same order as input
```

**`Promise.allSettled(iterable)`**: Waits for ALL promises to settle (ES2020)

```javascript
const results = await Promise.allSettled([
  fetch('/api/1'),
  fetch('/api/2'),
  fetch('/api/3')
]);

// Always fulfills with array of outcome objects:
// { status: 'fulfilled', value: ... } or
// { status: 'rejected', reason: ... }
```

**`Promise.race(iterable)`**: Resolves/rejects with first promise to settle

```javascript
const result = await Promise.race([
  fetch('/api/fast'),
  fetch('/api/slow')
]);
// Returns result from whichever completes first
```

**`Promise.any(iterable)`**: Resolves with first fulfilled promise (ES2021)

```javascript
const first = await Promise.any([
  fetch('/mirror1/data'),
  fetch('/mirror2/data'),
  fetch('/mirror3/data')
]);
// Returns first successful result, ignores rejections
// Only rejects if ALL promises reject (AggregateError)
```

### Async/Await: Syntactic Sugar

**`async/await`** is syntactic sugar over Promises, making asynchronous code look and behave like synchronous code.

**What `async` does**:

1. Makes a function always return a Promise
2. Allows use of `await` inside the function

```javascript
// These are equivalent:
async function getUser() {
  return 'John';
}

function getUser() {
  return Promise.resolve('John');
}

// Both return Promise<string>
```

**What `await` does**:

1. Pauses function execution until Promise settles
2. Returns the fulfilled value
3. Throws if Promise rejects
4. Only works inside `async` functions (or top-level in modules)

```javascript
// With Promises
function fetchUserData() {
  return fetch('/user')
    .then(res => res.json())
    .then(user => {
      console.log(user);
      return user;
    });
}

// With async/await (cleaner!)
async function fetchUserData() {
  const res = await fetch('/user');
  const user = await res.json();
  console.log(user);
  return user;
}
```

**How it works under the hood**:

```javascript
async function example() {
  const result = await promise;
  console.log(result);
}

// Is transformed to (roughly):
function example() {
  return promise.then(result => {
    console.log(result);
  });
}
```

**Top-Level Await** (ES2022):

```javascript
// In modules, can use await at top level
const data = await fetch('/api/config').then(r => r.json());
console.log(data);
// No need to wrap in async function
```

### Error Handling in JavaScript

**Types of Errors**:

1. **Syntax Errors**: Code violates JavaScript grammar (caught at parse time)
2. **Runtime Errors**: Occur during execution (e.g., `TypeError`, `ReferenceError`)
3. **Logical Errors**: Code runs but produces wrong results
4. **Async Errors**: Errors in asynchronous operations (Promise rejections)

**Error Objects**:

```javascript
// Built-in error types
new Error('Generic error');
new TypeError('Value is wrong type');
new ReferenceError('Variable not defined');
new RangeError('Value out of range');
new SyntaxError('Invalid syntax');

// Custom errors
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}
```

### Error Handling Patterns

**Error Handling Patterns**:

```javascript
// 1. .catch() method
fetch('/api/data')
  .then(res => res.json())
  .catch(err => console.error('Failed:', err));

// 2. try/catch with async/await
async function fetchData() {
  try {
    const res = await fetch('/api/data');
    const data = await res.json();
    return data;
  } catch (err) {
    console.error('Failed:', err);
    throw err; // Re-throw if needed
  }
}

// 3. Error handling in Promise.all
const results = await Promise.allSettled([
  fetch('/api/1'),
  fetch('/api/2')
]);

results.forEach((result, i) => {
  if (result.status === 'fulfilled') {
    console.log(`API ${i + 1}:`, result.value);
  } else {
    console.error(`API ${i + 1} failed:`, result.reason);
  }
});
```

**Senior-Level Best Practices**:

- Always handle rejections to avoid unhandled promise rejection warnings
- Use `finally()` for cleanup operations (close connections, remove loaders)
- Consider using `AbortController` for cancellable requests
- Be aware of error swallowing in async functions

```javascript
// AbortController pattern
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch('/api/data', { 
    signal: controller.signal 
  });
  clearTimeout(timeoutId);
} catch (err) {
  if (err.name === 'AbortError') {
    console.log('Request timed out');
  }
}
```

---

## Callback Hell vs. Promise Chaining

### What are Callbacks?

A **callback** is a function passed as an argument to another function, to be executed later (usually after an asynchronous operation completes).

**Why callbacks exist**:

```javascript
// Synchronous world (impossible for async operations)
const data = readFile('file.txt'); // Would block entire app
console.log(data);

// Asynchronous world (using callbacks)
readFile('file.txt', (data) => {
  console.log(data); // Executes when file is ready
});
console.log('Continues immediately'); // Doesn't wait
```

Callbacks enable non-blocking operations by allowing code to continue executing while waiting for operations to complete.

### The Callback Hell Problem (Pyramid of Doom)

**Example**:

```javascript
// ❌ Hard to read and maintain
getUserData(userId, (error, user) => {
  if (error) {
    handleError(error);
    return;
  }
  
  getOrders(user.id, (error, orders) => {
    if (error) {
      handleError(error);
      return;
    }
    
    getOrderDetails(orders[0].id, (error, details) => {
      if (error) {
        handleError(error);
        return;
      }
      
      processPayment(details.total, (error, payment) => {
        if (error) {
          handleError(error);
          return;
        }
        
        sendConfirmation(payment.id, (error, result) => {
          if (error) {
            handleError(error);
            return;
          }
          
          console.log('Done:', result);
        });
      });
    });
  });
});
```

### Problems with Callback-Based Code

#### 1. Deep Nesting (Pyramid of Doom)

- Code grows horizontally instead of vertically
- Difficult to follow control flow
- Hard to refactor

#### 2. Error Handling Complexity

```javascript
// Must check for errors at EVERY level
function process(callback) {
  step1((err1, result1) => {
    if (err1) return callback(err1);
    
    step2(result1, (err2, result2) => {
      if (err2) return callback(err2);
      
      step3(result2, (err3, result3) => {
        if (err3) return callback(err3);
        
        callback(null, result3);
      });
    });
  });
}
```

#### 3. Inversion of Control

**Definition**: You give your callback to a third-party function, trusting it to:

- Call it (not forget)
- Call it only once (not multiple times)
- Call it with correct arguments
- Handle errors properly

**Problem Example**:

```javascript
// You're trusting this third-party library
thirdPartyLibrary.doAsyncOperation(data, (result) => {
  chargeCustomer(result.amount); // What if called multiple times?!
});

// Potential issues:
// - Library calls callback twice → customer charged twice
// - Library never calls callback → operation hangs forever
// - Library calls with wrong data → errors
```

**Trust Issues**: You lose control over when, how, and if your code executes.

#### 4. No Composition

```javascript
// Can't easily combine multiple callbacks
const result1 = operation1(callback1);
const result2 = operation2(callback2);
// How to wait for both? Complex custom logic needed
```

#### 5. Difficult Debugging

- Stack traces become hard to follow
- Breakpoints less effective
- Hard to trace execution flow

**Promise Chaining**:

```javascript
// ✅ Flat, readable structure
getUserData(userId)
  .then(user => getOrders(user.id))
  .then(orders => getOrderDetails(orders[0].id))
  .then(details => processPayment(details.total))
  .then(payment => sendConfirmation(payment.id))
  .then(result => console.log('Done'))
  .catch(err => console.error('Error:', err)); // Single error handler
```

**Async/Await** (Best for Senior Level):

```javascript
// ✅ Most readable, synchronous-looking code
async function processOrder(userId) {
  try {
    const user = await getUserData(userId);
    const orders = await getOrders(user.id);
    const details = await getOrderDetails(orders[0].id);
    const payment = await processPayment(details.total);
    const result = await sendConfirmation(payment.id);
    console.log('Done');
  } catch (err) {
    console.error('Error:', err);
  }
}
```

**Parallel Execution** (Critical Optimization):

```javascript
// ❌ Sequential (slower)
const user = await getUser(id);
const posts = await getPosts(id);
const comments = await getComments(id);

// ✅ Parallel (faster)
const [user, posts, comments] = await Promise.all([
  getUser(id),
  getPosts(id),
  getComments(id)
]);
```

---

## ES6+ Features

### Destructuring

#### What is Destructuring?

**Destructuring** is a convenient syntax for extracting values from arrays or properties from objects into distinct variables.

**Why it was added to JavaScript**:

- **Reduces boilerplate code**: Extract multiple values in one line
- **Improves readability**: Clear intent of what's being extracted
- **Enables elegant patterns**: Swapping variables, function parameters, default values
- **Prevents errors**: Less chance of typos when accessing nested properties

**Before Destructuring (ES5)**:

```javascript
// Array access
var arr = [1, 2, 3];
var first = arr[0];
var second = arr[1];
var third = arr[2];

// Object access
var user = { name: 'John', age: 30 };
var name = user.name;
var age = user.age;
```

**With Destructuring (ES6)**:

```javascript
// Array destructuring
const [first, second, third] = [1, 2, 3];

// Object destructuring
const { name, age } = user;
```

#### Array Destructuring

**Basic Pattern**:

```javascript
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1, second = 2, rest = [3, 4, 5]
```

**Skipping Elements**:

```javascript
const [first, , third] = [1, 2, 3];
// first = 1, third = 3 (skipped 2)
```

**Default Values**:

```javascript
const [a = 10, b = 20] = [5];
// a = 5, b = 20 (default used)
```

**Swapping Variables** (elegant solution):

```javascript
let x = 1, y = 2;
[x, y] = [y, x]; // Swap without temp variable
// x = 2, y = 1
```

**Nested Arrays**:

```javascript
const [a, [b, c]] = [1, [2, 3]];
// a = 1, b = 2, c = 3
```

#### Object Destructuring

**Basic Pattern**:

```javascript
const user = { name: 'John', age: 30, city: 'NYC' };
const { name, age } = user;
// name = 'John', age = 30
```

**Renaming Variables**:

```javascript
const { name: userName, age: userAge } = user;
// userName = 'John', userAge = 30
// (name and age don't exist as variables)
```

**Default Values**:

```javascript
const { name, age = 18, role = 'user' } = user;
// If age doesn't exist in user, use 18
// If role doesn't exist, use 'user'
```

**Nested Objects**:

```javascript
const user = {
  name: 'John',
  address: {
    city: 'NYC',
    country: 'USA'
  }
};

const { address: { city, country } } = user;
// city = 'NYC', country = 'USA'
// Note: 'address' is NOT created as a variable
```

**Computed Property Names**:

```javascript
const key = 'name';
const { [key]: value } = { name: 'John' };
// value = 'John'
```

#### Function Parameter Destructuring

**Why it's powerful**:

- Named parameters (like Python)
- Clear function signature
- Easy to add/remove parameters
- Built-in default values

**Object Parameters**:

```javascript
// ❌ Old way: positional parameters
function createUser(name, age, role, active) {
  // Hard to remember order
  // Must pass undefined for defaults
}
createUser('John', 30, undefined, true);

// ✅ New way: destructured object
function createUser({ name, age, role = 'user', active = true }) {
  console.log(name, age, role, active);
}
createUser({ name: 'John', age: 30, active: false });
// Order doesn't matter, clear intent, easy defaults
```

**Array Parameters**:

```javascript
function displayCoordinates([x, y, z = 0]) {
  console.log(`X: ${x}, Y: ${y}, Z: ${z}`);
}

displayCoordinates([10, 20]); // Z uses default 0
```

#### Destructuring: Practical Use Cases

**1. Function Return Values**:

```javascript
function getUser() {
  return { name: 'John', age: 30, email: 'john@example.com' };
}

const { name, email } = getUser();
// Extract only what you need
```

**2. React Props** (very common):

```javascript
// Instead of:
function Component(props) {
  return <div>{props.title} - {props.content}</div>;
}

// Use:
function Component({ title, content }) {
  return <div>{title} - {content}</div>;
}
```

**3. Importing Modules**:

```javascript
import { useState, useEffect } from 'react';
// Destructure only what you need
```

**4. For-of Loops with Objects**:

```javascript
const users = [
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 }
];

for (const { name, age } of users) {
  console.log(`${name} is ${age} years old`);
}
```

### Spread/Rest Operators

#### The `...` Syntax: Two Different Meanings

The three dots (`...`) have **completely different behaviors** depending on context:

- **Spread Operator**: Expands/spreads an iterable into individual elements
- **Rest Parameter**: Collects multiple elements into an array

**Why same syntax for different things?**

- Same concept, opposite directions
- Spread = "unpack"
- Rest = "pack"

#### Spread Operator (Expanding)

**Definition**: Takes an iterable (array, string, set, etc.) and expands it into individual elements.

**Array Spreading**:

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// Combining arrays
const combined = [...arr1, ...arr2];
// [1, 2, 3, 4, 5, 6]

// Adding elements
const extended = [0, ...arr1, 4];
// [0, 1, 2, 3, 4]

// Copying arrays (shallow copy)
const copy = [...arr1];
// Changes to copy don't affect arr1
```

**Why spreading is useful for arrays**:

```javascript
// ❌ Old way
const combined = arr1.concat(arr2);
const copy = arr1.slice();

// ✅ New way (more intuitive)
const combined = [...arr1, ...arr2];
const copy = [...arr1];
```

**Object Spreading** (ES2018):

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

// Combining objects
const combined = { ...obj1, ...obj2 };
// { a: 1, b: 2, c: 3, d: 4 }

// Overriding properties
const updated = { ...obj1, b: 999 };
// { a: 1, b: 999 } (b was overridden)

// Shallow copy
const copy = { ...obj1 };
```

**Property Order Matters**:

```javascript
const defaults = { theme: 'dark', lang: 'en' };
const user = { lang: 'es' };

// User preferences override defaults
const config = { ...defaults, ...user };
// { theme: 'dark', lang: 'es' }

// Defaults fill in missing values
const config2 = { ...user, ...defaults };
// { lang: 'en', theme: 'dark' } (defaults override!)
```

**Function Arguments**:

```javascript
const numbers = [1, 2, 3];

// Spread array into function arguments
Math.max(...numbers); // 3
// Equivalent to: Math.max(1, 2, 3)

console.log(...numbers); // 1 2 3
// Equivalent to: console.log(1, 2, 3)
```

**String Spreading**:

```javascript
const str = 'hello';
const chars = [...str];
// ['h', 'e', 'l', 'l', 'o']

// Useful for counting characters (handles Unicode properly)
const emoji = '👨‍👩‍👧‍👦';
[...emoji].length; // 1 (correct!)
emoji.length; // 11 (wrong - counts code points)
```

#### Rest Parameters (Collecting)

**Definition**: Collects multiple function arguments into an array.

**Basic Usage**:

```javascript
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}

sum(1, 2, 3); // 6
sum(1, 2, 3, 4, 5); // 15
// All arguments collected into 'numbers' array
```

**Before Rest Parameters**:

```javascript
// ❌ Old way: using 'arguments' object
function oldSum() {
  // arguments is array-like, not a real array
  var args = Array.prototype.slice.call(arguments);
  return args.reduce(function(acc, n) { return acc + n; }, 0);
}

// ✅ New way: rest parameters
function newSum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
```

**Combining with Regular Parameters**:

```javascript
function greet(greeting, ...names) {
  return `${greeting} ${names.join(', ')}`;
}

greet('Hello', 'John', 'Jane', 'Bob');
// "Hello John, Jane, Bob"

// greeting = 'Hello'
// names = ['John', 'Jane', 'Bob']
```

**⚠️ Important Rule**: Rest parameter must be LAST

```javascript
// ✅ Valid
function fn(a, b, ...rest) { }

// ❌ Invalid
function fn(...rest, a, b) { } // SyntaxError
function fn(a, ...rest, b) { } // SyntaxError
```

#### Rest in Destructuring

**Array Rest**:

```javascript
const [first, second, ...rest] = [1, 2, 3, 4, 5];
// first = 1
// second = 2
// rest = [3, 4, 5]
```

**Object Rest** (ES2018):

```javascript
const user = { name: 'John', age: 30, city: 'NYC', country: 'USA' };
const { name, age, ...address } = user;
// name = 'John'
// age = 30
// address = { city: 'NYC', country: 'USA' }
```

**Common Pattern: Removing Properties**:

```javascript
const user = { id: 1, password: 'secret', name: 'John', email: 'john@example.com' };

// Remove sensitive data
const { password, ...safeUser } = user;
// safeUser = { id: 1, name: 'John', email: 'john@example.com' }
```

#### Shallow Copy Warning

**Critical Concept**: Spread creates **shallow copies**, not deep copies.

```javascript
const original = {
  name: 'John',
  address: { city: 'NYC' } // Nested object
};

const copy = { ...original };

// Changing nested object affects both!
copy.address.city = 'LA';
console.log(original.address.city); // 'LA' (changed!)

// Top-level changes are isolated
copy.name = 'Jane';
console.log(original.name); // 'John' (unchanged)
```

**Why this happens**:

```text
original.address ───┐
                    ├──> { city: 'NYC' } (same object in memory)
copy.address ───────┘
```

**For deep copies**, use:

```javascript
// Option 1: JSON (limited - loses functions, dates, etc.)
const deepCopy = JSON.parse(JSON.stringify(original));

// Option 2: Lodash
const deepCopy = _.cloneDeep(original);

// Option 3: structuredClone (modern browsers)
const deepCopy = structuredClone(original);
```

### Arrow Functions

#### What are Arrow Functions?

**Arrow functions** (`=>`) are a shorter syntax for writing function expressions, with special behavior regarding `this` binding.

**Syntax Evolution**:

```javascript
// Traditional function expression
const add = function(a, b) {
  return a + b;
};

// Arrow function
const add = (a, b) => {
  return a + b;
};

// Arrow function with implicit return
const add = (a, b) => a + b;
```

#### Syntax Variations

**No parameters**:

```javascript
const greet = () => 'Hello';
```

**One parameter** (parentheses optional):

```javascript
const double = x => x * 2;
// or
const double = (x) => x * 2;
```

**Multiple parameters** (parentheses required):

```javascript
const add = (a, b) => a + b;
```

**Implicit return** (no braces, single expression):

```javascript
const square = x => x * x;
const getUser = id => ({ id, name: 'John' });
// Note: wrap objects in () to distinguish from function body
```

**Explicit return** (with braces):

```javascript
const add = (a, b) => {
  const result = a + b;
  return result;
};
```

#### Lexical `this` Binding

**This is the MAIN difference from regular functions.**

**What "Lexical" Means**:

- Arrow functions don't create their own `this`
- They inherit `this` from the enclosing scope
- `this` is determined at **author-time** (where function is defined), not **call-time** (where function is called)

**Regular Function vs Arrow Function**:

```javascript
const obj = {
  name: 'Object',
  
  regularFunction: function() {
    console.log(this.name); // 'this' = obj (where it's called)
  },
  
  arrowFunction: () => {
    console.log(this.name); // 'this' = global/window (where it's defined)
  }
};

obj.regularFunction(); // "Object"
obj.arrowFunction();   // undefined (or global name)
```

**Why Lexical `this` is Useful**:

**Problem with Regular Functions**:

```javascript
class Timer {
  constructor() {
    this.seconds = 0;
    
    // ❌ Doesn't work - 'this' becomes undefined
    setInterval(function() {
      this.seconds++; // Error: Cannot read property 'seconds' of undefined
    }, 1000);
  }
}

// Old solutions:
// 1. Store 'this'
var self = this;
setInterval(function() { self.seconds++; }, 1000);

// 2. .bind(this)
setInterval(function() { this.seconds++; }.bind(this), 1000);
```

**Solution with Arrow Functions**:

```javascript
class Timer {
  constructor() {
    this.seconds = 0;
    
    // ✅ Works! Arrow function inherits 'this' from constructor
    setInterval(() => {
      this.seconds++;
    }, 1000);
  }
}
```

**Event Handlers Example**:

```javascript
class Button {
  constructor() {
    this.clicks = 0;
    
    // ❌ Regular function - loses 'this'
    document.getElementById('btn').addEventListener('click', function() {
      this.clicks++; // Error: 'this' is the button element, not Button instance
    });
    
    // ✅ Arrow function - preserves 'this'
    document.getElementById('btn').addEventListener('click', () => {
      this.clicks++; // Works! 'this' is Button instance
    });
  }
}
```

#### What Arrow Functions DON'T Have

**1. No `this` binding**:

```javascript
const obj = {
  method: () => {
    console.log(this); // 'this' is NOT obj
  }
};
```

**2. No `arguments` object**:

```javascript
// Regular function
function regular() {
  console.log(arguments); // Works
}

// Arrow function
const arrow = () => {
  console.log(arguments); // ReferenceError
};

// Use rest parameters instead
const arrow = (...args) => {
  console.log(args); // ✅ Works
};
```

**3. Cannot be used as constructors**:

```javascript
const Person = (name) => {
  this.name = name;
};

const p = new Person('John'); // TypeError: Person is not a constructor
```

**4. No `prototype` property**:

```javascript
const regular = function() {};
console.log(regular.prototype); // {} (exists)

const arrow = () => {};
console.log(arrow.prototype); // undefined
```

**5. Cannot be used with `call()`, `apply()`, or `bind()` to change `this`**:

```javascript
const arrow = () => console.log(this);
const obj = { name: 'Object' };

arrow.call(obj);  // 'this' is NOT obj
arrow.apply(obj); // 'this' is NOT obj
const bound = arrow.bind(obj);
bound(); // 'this' is still NOT obj

// 'this' remains lexically bound, cannot be changed
```

#### When NOT to Use Arrow Functions

**1. Object Methods** (need dynamic `this`):

```javascript
const user = {
  name: 'John',
  
  // ❌ Arrow function - 'this' is NOT user
  greet: () => {
    console.log(`Hello, ${this.name}`); // undefined
  },
  
  // ✅ Regular method
  greet() {
    console.log(`Hello, ${this.name}`); // "Hello, John"
  }
};
```

**2. Prototype Methods**:

```javascript
function Person(name) {
  this.name = name;
}

// ❌ Arrow function
Person.prototype.greet = () => {
  console.log(this.name); // undefined
};

// ✅ Regular function
Person.prototype.greet = function() {
  console.log(this.name); // Works
};
```

**3. Event Handlers (when you need the element)**:

```javascript
// If you need 'this' to be the clicked element:
button.addEventListener('click', function() {
  this.classList.toggle('active'); // ✅ 'this' is button
});

button.addEventListener('click', () => {
  this.classList.toggle('active'); // ❌ 'this' is NOT button
});
```

**4. Functions with `arguments` object**:

```javascript
// If you need the arguments object:
function sum() {
  return Array.from(arguments).reduce((a, b) => a + b);
}

// Arrow version needs rest parameters:
const sum = (...args) => args.reduce((a, b) => a + b);
```

#### When TO Use Arrow Functions

**1. Callbacks** (most common use):

```javascript
const numbers = [1, 2, 3, 4, 5];

// ✅ Perfect for map, filter, reduce
const doubled = numbers.map(n => n * 2);
const evens = numbers.filter(n => n % 2 === 0);
```

**2. Preserving `this` context**:

```javascript
class API {
  constructor() {
    this.baseURL = 'https://api.example.com';
  }
  
  fetchData() {
    // ✅ Arrow function preserves 'this'
    fetch(this.baseURL)
      .then(res => res.json())
      .then(data => this.processData(data));
  }
}
```

**3. Short, functional code**:

```javascript
// ✅ Clean and readable
const add = (a, b) => a + b;
const square = x => x * x;
const getUser = id => ({ id, name: 'John' });
```

### Template Literals

#### What are Template Literals?

**Template literals** are string literals that allow embedded expressions, multi-line strings, and string interpolation using backticks (`` ` ``) instead of quotes.

**Why they were added**:

- **String interpolation**: Embed variables directly without concatenation
- **Multi-line strings**: Write strings across multiple lines naturally
- **Tagged templates**: Process template literals with custom functions
- **Better readability**: Cleaner than concatenation

#### Basic Syntax

**Before Template Literals (ES5)**:

```javascript
// String concatenation (messy)
var name = 'John';
var age = 30;
var message = 'Hello ' + name + ', you are ' + age + ' years old.';

// Multi-line strings (awkward)
var html = '<div>\n' +
           '  <h1>' + title + '</h1>\n' +
           '  <p>' + content + '</p>\n' +
           '</div>';
```

**With Template Literals (ES6)**:

```javascript
// Clean interpolation
const name = 'John';
const age = 30;
const message = `Hello ${name}, you are ${age} years old.`;

// Natural multi-line strings
const html = `
  <div>
    <h1>${title}</h1>
    <p>${content}</p>
  </div>
`;
```

#### String Interpolation

**Basic Interpolation**:

```javascript
const name = 'John';
const age = 30;

const greeting = `Hello, ${name}!`;
// "Hello, John!"
```

**Expressions** (not just variables):

```javascript
const a = 5;
const b = 10;

console.log(`Sum: ${a + b}`);        // "Sum: 15"
console.log(`Product: ${a * b}`);    // "Product: 50"
console.log(`Comparison: ${a > b}`); // "Comparison: false"
```

**Function Calls**:

```javascript
function getFullName(first, last) {
  return `${first} ${last}`;
}

const message = `Welcome, ${getFullName('John', 'Doe')}!`;
// "Welcome, John Doe!"
```

**Nested Template Literals**:

```javascript
const isActive = true;
const user = { name: 'John', role: 'admin' };

const badge = `
  <span class="${isActive ? 'active' : 'inactive'}">
    ${user.name} (${user.role})
  </span>
`;
```

#### Multi-line Strings

**Preserves formatting**:

```javascript
const poem = `
  Roses are red,
  Violets are blue,
  Template literals
  Are great for you!
`;

// All whitespace and newlines are preserved
```

**SQL Queries**:

```javascript
const userId = 123;
const query = `
  SELECT u.name, u.email, o.total
  FROM users u
  JOIN orders o ON u.id = o.user_id
  WHERE u.id = ${userId}
  ORDER BY o.created_at DESC
`;
```

**HTML Templates**:

```javascript
const user = { name: 'John', avatar: '/img/john.jpg' };

const html = `
  <div class="user-card">
    <img src="${user.avatar}" alt="${user.name}">
    <h2>${user.name}</h2>
  </div>
`;
```

#### Tagged Templates (Advanced)

**What are Tagged Templates?**

A **tag function** processes a template literal, giving you control over how the string is constructed.

**Syntax**:

```javascript
tagFunction`template ${expression} literal`;
//          ^ No parentheses!
```

**How it works**:

```javascript
function myTag(strings, ...values) {
  // strings: array of string literals
  // values: array of evaluated expressions
  console.log(strings); // ['Hello ', ' and ', '!']
  console.log(values);  // ['John', 'Jane']
  
  return 'processed string';
}

const result = myTag`Hello ${'John'} and ${'Jane'}!`;
```

**Structure passed to tag function**:

```javascript
myTag`a${1}b${2}c`
// strings = ['a', 'b', 'c']
// values = [1, 2]

// Note: strings.length === values.length + 1 (always)
```

**Real-World Example: HTML Escaping**:

```javascript
function safeHTML(strings, ...values) {
  const escape = str => String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
  
  return strings.reduce((result, str, i) => {
    const value = values[i] ? escape(values[i]) : '';
    return result + str + value;
  }, '');
}

const userInput = '<script>alert("XSS")</script>';
const safe = safeHTML`<div>${userInput}</div>`;
// "<div>&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;</div>"
```

**Syntax Highlighting Example**:

```javascript
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) => {
    const value = values[i] || '';
    return `${acc}${str}<mark>${value}</mark>`;
  }, '');
}

const name = 'John';
const age = 30;
const html = highlight`Name: ${name}, Age: ${age}`;
// "Name: <mark>John</mark>, Age: <mark>30</mark>"
```

**i18n (Internationalization) Example**:

```javascript
const translations = {
  en: { greeting: 'Hello {0}, you have {1} messages' },
  es: { greeting: 'Hola {0}, tienes {1} mensajes' }
};

function i18n(lang) {
  return function(strings, ...values) {
    let key = strings.join('{n}').trim();
    let template = translations[lang][key];
    
    values.forEach((val, i) => {
      template = template.replace(`{${i}}`, val);
    });
    
    return template;
  };
}

const t = i18n('es');
const userName = 'Juan';
const count = 5;
const message = t`greeting${userName}${count}`;
// "Hola Juan, tienes 5 mensajes"
```

**Styled Components** (React library uses this):

```javascript
const Button = styled.button`
  background: ${props => props.primary ? 'blue' : 'gray'};
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  
  &:hover {
    background: ${props => props.primary ? 'darkblue' : 'darkgray'};
  }
`;

// Usage: <Button primary>Click me</Button>
```

**GraphQL Queries** (uses tagged templates):

```javascript
const query = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      name
      email
      posts {
        title
      }
    }
  }
`;
```

#### Raw Strings

**`String.raw` tag function**:

Interprets backslashes literally (doesn't process escape sequences):

```javascript
// Normal template literal
const normal = `Line 1\nLine 2`;
console.log(normal);
// Line 1
// Line 2

// Raw template literal
const raw = String.raw`Line 1\nLine 2`;
console.log(raw); // "Line 1\nLine 2" (literal \n)
```

**Use case: File paths on Windows**:

```javascript
const path = String.raw`C:\Users\John\Documents\file.txt`;
// No need to escape backslashes!
```

**Use case: Regular expressions**:

```javascript
const regex = new RegExp(String.raw`\d+\.\d+`);
// Easier to read than: new RegExp('\\d+\\.\\d+')
```

#### Template Literals: Practical Use Cases

**1. Dynamic SQL (be careful with injection!)**:

```javascript
// ⚠️ Vulnerable to SQL injection
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ Better: use parameterized queries
const query = prepared`SELECT * FROM users WHERE id = ${userId}`;
```

**2. Configuration Files**:

```javascript
const config = `
  server {
    listen ${port};
    server_name ${domain};
    root ${rootPath};
  }
`;
```

**3. Email Templates**:

```javascript
function sendEmail(user, orderNumber) {
  const email = `
    Dear ${user.name},
    
    Your order #${orderNumber} has been confirmed.
    Total: $${order.total.toFixed(2)}
    
    Thank you for your purchase!
  `;
  
  return sendMail(user.email, 'Order Confirmation', email);
}
```

**4. Debug Logging**:

```javascript
function debugLog(variable) {
  console.log(`[DEBUG] ${variable.name} = ${JSON.stringify(variable.value)}`);
}
```

### Other Critical ES6+ Features

- **`let`/`const`**: Block-scoped variables (no hoisting issues)
- **Default parameters**: `function greet(name = 'Guest') {}`
- **Optional chaining**: `user?.address?.city`
- **Nullish coalescing**: `value ?? defaultValue` (only null/undefined, not 0/'')
- **Object shorthand**: `{ name, age }` instead of `{ name: name, age: age }`
- **Computed property names**: `{ [key]: value }`

---

## WeakMap/WeakSet

### What are WeakMap and WeakSet?

**WeakMap** and **WeakSet** are collections similar to Map and Set, but with a critical difference: they hold **weak references** to their keys/values.

**Why they exist**:

- **Prevent memory leaks**: Associate data with objects without preventing garbage collection
- **Automatic cleanup**: No need to manually remove entries when objects are destroyed
- **DOM metadata**: Safely store data about DOM elements that may be removed
- **Private data**: Store private object data without exposing it

#### Understanding Garbage Collection

**What is Garbage Collection?**

Automatic memory management that frees memory occupied by objects that are no longer **reachable**.

**Reachability**:

An object is **reachable** if it can be accessed from:

1. **Global scope** variables
2. **Currently executing function** (local variables, parameters)
3. **Call stack** (variables in functions that are waiting)
4. **Any reference** from another reachable object

**Mark-and-Sweep Algorithm** (JavaScript GC):

```text
Phase 1: Mark
- Start from roots (global variables, call stack)
- Traverse all references, mark reachable objects

Phase 2: Sweep
- Go through all objects in memory
- Delete unmarked (unreachable) objects
- Free memory
```

**Example**:

```javascript
let user = { name: 'John' };

// user is reachable (referenced by global variable)

user = null; // No more references

// { name: 'John' } is now unreachable
// Garbage collector will free this memory
```

#### Strong vs Weak References

**Strong Reference** (regular variables):

```javascript
const obj = { data: 'important' };
const references = [obj]; // Strong reference

// Even if obj is set to null:
obj = null;

// Object is still reachable through 'references' array
console.log(references[0].data); // "important"

// Object is NOT garbage collected
```

**Weak Reference** (WeakMap/WeakSet):

```javascript
let obj = { data: 'important' };
const weakMap = new WeakMap();
weakMap.set(obj, 'metadata');

// WeakMap holds a WEAK reference to obj

obj = null; // Remove strong reference

// obj is now unreachable (no strong references)
// Garbage collector CAN collect obj
// WeakMap entry is automatically removed
```

**Critical Difference**:

- **Strong reference**: Keeps object alive (prevents GC)
- **Weak reference**: Doesn't keep object alive (allows GC)

#### WeakMap Deep Dive

**Structure**:

```javascript
const weakMap = new WeakMap();
```

**Characteristics**:

1. **Keys MUST be objects** (not primitives):

```javascript
const wm = new WeakMap();

const obj = {};
wm.set(obj, 'value'); // ✅ Works

wm.set('string', 'value'); // ❌ TypeError: Invalid value used as weak map key
wm.set(42, 'value');       // ❌ TypeError
wm.set(null, 'value');     // ❌ TypeError
```

**Why?** Primitives are never garbage collected (they're values, not references).

**No iteration methods**:

```javascript
const wm = new WeakMap();

// These don't exist:
wm.keys();    // undefined
wm.values();  // undefined
wm.entries(); // undefined
wm.forEach(); // undefined
wm.size;      // undefined
```

**Why?** Entries can be garbage collected at any time, making iteration non-deterministic.

**Available methods**:

```javascript
const wm = new WeakMap();
const obj = {};

wm.set(obj, 'value');    // Add entry
wm.get(obj);             // 'value'
wm.has(obj);             // true
wm.delete(obj);          // true (manually remove)
```

#### WeakMap Use Cases

**1. Private Data Storage** (before `#private` fields):

```javascript
const privateData = new WeakMap();

class User {
  constructor(name, ssn) {
    this.name = name; // Public
    privateData.set(this, { ssn }); // Private
  }
  
  getSSN() {
    return privateData.get(this).ssn;
  }
}

const user = new User('John', '123-45-6789');
console.log(user.name);    // "John" (accessible)
console.log(user.ssn);     // undefined (not accessible)
console.log(user.getSSN()); // "123-45-6789" (via method)

// When user is garbage collected, private data is too
```

**Why this prevents memory leaks**:

```javascript
// Bad (with regular Map):
const privateData = new Map();

class User {
  constructor(name, ssn) {
    this.name = name;
    privateData.set(this, { ssn }); // Strong reference!
  }
}

let user = new User('John', '123');
user = null;

// User instance is NOT garbage collected!
// Map still holds strong reference
// Memory leak!
```

**2. Caching/Memoization**:

```javascript
const cache = new WeakMap();

function expensiveOperation(obj) {
  // Check cache
  if (cache.has(obj)) {
    console.log('Cache hit!');
    return cache.get(obj);
  }
  
  // Expensive computation
  console.log('Computing...');
  const result = JSON.stringify(obj).length * 1000;
  
  // Store in cache
  cache.set(obj, result);
  return result;
}

const data = { huge: 'object' };
expensiveOperation(data); // "Computing..."
expensiveOperation(data); // "Cache hit!"

// If 'data' is no longer used:
data = null;
// Cache entry is automatically removed (no memory leak)
```

**3. DOM Node Metadata**:

```javascript
const domMetadata = new WeakMap();

function attachMetadata(element, data) {
  domMetadata.set(element, data);
}

function getMetadata(element) {
  return domMetadata.get(element);
}

// Usage:
const button = document.querySelector('#myButton');
attachMetadata(button, { clicks: 0, lastClicked: null });

button.addEventListener('click', () => {
  const meta = getMetadata(button);
  meta.clicks++;
  meta.lastClicked = new Date();
});

// If button is removed from DOM:
button.remove();

// And no other references exist:
// button and its metadata are garbage collected
// No memory leak!
```

**Memory leak example (without WeakMap)**:

```javascript
// Bad:
const domMetadata = new Map(); // Strong references

function attachMetadata(element, data) {
  domMetadata.set(element, data);
}

const button = document.querySelector('#myButton');
attachMetadata(button, { data: 'example' });

// Remove button:
button.remove();

// Button is removed from DOM, but:
// - Map still holds strong reference to button
// - Button is NOT garbage collected
// - Memory leak!

// Solution: Manually clean up
domMetadata.delete(button); // Must remember to do this!
```

#### WeakSet Deep Dive

**Structure**:

```javascript
const weakSet = new WeakSet();
```

**Characteristics**:

1. **Values MUST be objects**:

```javascript
const ws = new WeakSet();

const obj = {};
ws.add(obj); // ✅ Works

ws.add('string'); // ❌ TypeError
ws.add(42);       // ❌ TypeError
```

**No iteration methods** (same reason as WeakMap)

**Available methods**:

```javascript
const ws = new WeakSet();
const obj = {};

ws.add(obj);     // Add object
ws.has(obj);     // true
ws.delete(obj);  // true (manually remove)
```

#### WeakSet Use Cases

**1. Tracking Processed Objects**:

```javascript
const processedItems = new WeakSet();

function processItem(item) {
  if (processedItems.has(item)) {
    console.log('Already processed');
    return;
  }
  
  console.log('Processing...');
  // ... expensive operation ...
  
  processedItems.add(item);
}

const obj1 = { id: 1 };
const obj2 = { id: 2 };

processItem(obj1); // "Processing..."
processItem(obj1); // "Already processed"
processItem(obj2); // "Processing..."

// When obj1 is no longer referenced:
obj1 = null;
// It's automatically removed from processedItems
```

**2. Marking Objects (Flags)**:

```javascript
const disabledElements = new WeakSet();

function disableElement(element) {
  disabledElements.add(element);
  element.disabled = true;
}

function isDisabled(element) {
  return disabledElements.has(element);
}

const input = document.querySelector('input');
disableElement(input);

console.log(isDisabled(input)); // true

// When input is removed and garbage collected,
// it's automatically removed from disabledElements
```

**3. Preventing Circular Reference Issues**:

```javascript
const visited = new WeakSet();

function traverse(node) {
  if (visited.has(node)) {
    return; // Prevent infinite loop
  }
  
  visited.add(node);
  console.log(node.name);
  
  // Traverse children
  node.children?.forEach(traverse);
}

// Circular structure:
const parent = { name: 'Parent', children: [] };
const child = { name: 'Child', children: [parent] };
parent.children.push(child);

traverse(parent); // Safely handles circular references
```

#### Why WeakMap/WeakSet Can't Be Iterated

**The Problem**:

```javascript
const wm = new WeakMap();
let obj = { data: 'example' };
wm.set(obj, 'value');

// If we could iterate:
for (let [key, value] of wm) { // ❌ Doesn't exist
  console.log(key, value);
}

// During iteration:
obj = null; // Object becomes unreachable

// Garbage collector runs (non-deterministic)
// Entry might be removed MID-ITERATION
// Iterator becomes invalid!
```

**The Solution**: No iteration allowed

- Can't observe when GC runs
- Can't provide consistent iteration
- Entries may disappear at any time

#### Comparison: Map/Set vs WeakMap/WeakSet

| Feature | Map/Set | WeakMap/WeakSet |
|---------|---------|------------------|
| **Keys/Values** | Any type | Objects only |
| **References** | Strong | Weak |
| **Iteration** | Yes (`.keys()`, `.values()`, `.entries()`, `.forEach()`) | No |
| **`.size`** | Yes | No |
| **GC** | Prevents GC | Allows GC |
| **Use Case** | General collections | Metadata, caching, private data |
| **Memory Leaks** | Possible if not cleaned | Prevented automatically |

**When to use each**:

```javascript
// Use Map when:
// - Need to iterate over entries
// - Need to know collection size
// - Keys are primitives
// - Want to prevent GC (keep references alive)

const userRoles = new Map();
userRoles.set('user1', 'admin');
userRoles.set('user2', 'editor');

// Use WeakMap when:
// - Associating metadata with objects
// - Don't want to prevent GC
// - Keys are objects that may be removed
// - Preventing memory leaks is critical

const objectMetadata = new WeakMap();
objectMetadata.set(domElement, { clicks: 0 });
```

---

## Modules (ES6 vs. CommonJS)

### Why Module Systems Exist

**JavaScript Without Modules (Pre-2009)**:

Before module systems, JavaScript applications faced critical problems:

#### 1. Global Scope Pollution

```javascript
// file1.js
var userName = 'John';
function getUserData() { /* ... */ }

// file2.js
var userName = 'Jane'; // Accidentally overwrites file1's userName!
function getUserData() { /* ... */ } // Overwrites function!

// All <script> tags share the same global scope
```

#### 2. No Dependency Management

```html
<!-- Order matters! Must load in correct sequence -->
<script src="jquery.js"></script>
<script src="jquery-plugin.js"></script> <!-- Depends on jQuery -->
<script src="app.js"></script> <!-- Depends on both -->
```

If order is wrong, application breaks. No way to express dependencies explicitly.

#### 3. No Encapsulation

```javascript
// Everything is globally accessible
var apiKey = 'secret-key-123'; // Exposed to entire app!
// No way to hide implementation details
```

**Early Solutions**:

**IIFE Pattern** (Immediately Invoked Function Expression):

```javascript
// Creates private scope
var myModule = (function() {
  var privateVar = 'secret';
  
  return {
    publicMethod: function() {
      return privateVar;
    }
  };
})();

myModule.publicMethod(); // Works
myModule.privateVar; // undefined (private!)
```

**Revealing Module Pattern**:

```javascript
var calculator = (function() {
  var result = 0;
  
  function add(x) { result += x; }
  function getResult() { return result; }
  
  // Reveal only what you want public
  return {
    add: add,
    getResult: getResult
  };
})();
```

**Problems with these patterns**:

- Still manually managing `<script>` order
- No standard way to declare dependencies
- No built-in way to load modules dynamically
- Build tools couldn't optimize effectively

### CommonJS: Node.js Solution (2009)

**Why it was created**:

- Node.js needed server-side modules
- Couldn't use `<script>` tags (no browser)
- Needed synchronous loading (file system is fast)
- Inspired by ServerJS (server-side JavaScript standards)

**How CommonJS Works**:

```javascript
// Each file is a module with own scope
// module.js
const privateVar = 'secret';

function privateFunction() {
  return privateVar;
}

// Explicitly export what's public
module.exports = {
  publicFunction: privateFunction
};

// app.js
const myModule = require('./module'); // Synchronous load
myModule.publicFunction(); // Works
myModule.privateVar; // undefined (not exported)
```

**Module Wrapping**:

Node.js wraps each module in a function:

```javascript
// Your code:
const x = 10;
module.exports = x;

// Actually executed as:
(function(exports, require, module, __filename, __dirname) {
  const x = 10;
  module.exports = x;
});
```

This creates module scope and provides `module`, `exports`, `require`.

### ES6 Modules (ESM): Standard Solution (2015)

**Why a new standard was needed**:

- CommonJS was Node.js-specific (not browser standard)
- Synchronous loading doesn't work for browsers (network is slow)
- **Static analysis impossible** - can't optimize without running code
- JavaScript needed a universal, standardized module system

#### Key Innovation: Static Structure

**CommonJS (Dynamic)**:

```javascript
// Can require() anywhere, even conditionally
if (condition) {
  const module = require('./module'); // Runtime decision
}

// Can't analyze dependencies without running code
const moduleName = 'module-' + version;
require(moduleName); // Unknown until runtime
```

**ES6 Modules (Static)**:

```javascript
// Imports must be at top level
import { feature } from './module'; // Analyzed at parse time

// This is NOT allowed:
if (condition) {
  import { feature } from './module'; // ❌ Syntax Error
}

// For dynamic imports, use import():
if (condition) {
  const { feature } = await import('./module'); // ✅ Runtime loading
}
```

**Benefits of Static Structure**:

#### Tree Shaking (Dead Code Elimination)

```javascript
// utils.js
export function usedFunction() { /* ... */ }
export function unusedFunction() { /* ... */ }

// app.js
import { usedFunction } from './utils';

// Bundler can detect unusedFunction is never imported
// → Removes it from final bundle (smaller file size)
```

With CommonJS, bundlers can't be sure what's used:

```javascript
const utils = require('./utils');
// Bundler doesn't know which properties are accessed
// Must include entire module
```

#### Early Error Detection

```javascript
// ESM: Error at parse time (before execution)
import { typo } from './module'; // SyntaxError immediately

// CJS: Error at runtime (when line executes)
const { typo } = require('./module'); // Error only when this runs
```

#### Better IDE Support

Static imports enable autocomplete and refactoring:

```javascript
import { feature } from './module';
//      ^-- IDE knows what's exported, provides autocomplete
```

### Live Bindings vs Value Copies

**Critical Difference**:

**CommonJS** exports copies (primitives) or references (objects):

```javascript
// counter.js (CommonJS)
let count = 0;
module.exports = {
  count: count, // Exports COPY of primitive
  increment() { count++; }
};

// app.js
const counter = require('./counter');
console.log(counter.count); // 0
counter.increment();
console.log(counter.count); // Still 0 (copy didn't update!)
```

**ES6 Modules** export live bindings:

```javascript
// counter.js (ESM)
export let count = 0;
export function increment() { count++; }

// app.js
import { count, increment } from './counter';
console.log(count); // 0
increment();
console.log(count); // 1 (live binding updated!)
```

**Why it matters**:

- ESM reflects changes automatically
- CJS requires re-exporting or using objects
- ESM more predictable for mutable state

### ES6 Modules (ESM)

**Syntax**:

```javascript
// export.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export default class Calculator {}

// import.js
import Calculator, { PI, add } from './export.js';
import * as math from './export.js';
import { add as sum } from './export.js'; // Rename
```

**Characteristics**:

- **Static structure**: Imports/exports determined at compile time
- **Live bindings**: Imported values are references, not copies
- **Asynchronous loading**: Can be loaded dynamically with `import()`
- **Tree-shaking**: Unused exports can be eliminated by bundlers
- **Strict mode**: Always runs in strict mode
- **Top-level await**: Supported in modern environments

```javascript
// Dynamic imports
const module = await import('./module.js');

// Conditional loading
if (condition) {
  const { feature } = await import('./feature.js');
}
```

### CommonJS (CJS)

**Syntax**:

```javascript
// export.js
const PI = 3.14159;
function add(a, b) { return a + b; }
module.exports = { PI, add };
// or
exports.PI = PI;
exports.add = add;

// import.js
const { PI, add } = require('./export.js');
const math = require('./export.js');
```

**Characteristics**:

- **Dynamic structure**: Requires resolved at runtime
- **Synchronous loading**: Blocks execution until module loads
- **Value copies**: Imported values are copies (primitives) or shallow copies (objects)
- **No tree-shaking**: Harder for bundlers to optimize
- **Default in Node.js**: Until `"type": "module"` in package.json

### Key Differences

| Feature | ES6 Modules | CommonJS |
|---------|-------------|----------|
| **Loading** | Asynchronous | Synchronous |
| **Binding** | Live references | Value copies |
| **Structure** | Static (compile-time) | Dynamic (runtime) |
| **Tree-shaking** | Yes | Limited |
| **Top-level await** | Yes | No |
| **Browser support** | Native | Requires bundler |
| **Conditional imports** | Dynamic `import()` | Direct `require()` |

### When to Use What?

**ES6 Modules**:

- Modern frontend projects
- New Node.js projects (with `"type": "module"`)
- When tree-shaking is important
- Universal code (browser + Node.js)

**CommonJS**:

- Legacy Node.js projects
- When synchronous loading is needed
- Some tooling still requires it

**Interoperability**:

```javascript
// ESM importing CJS
import pkg from 'commonjs-package'; // Default import
import { named } from 'commonjs-package'; // May not work

// CJS importing ESM (Node.js)
const module = await import('esm-module'); // Must use dynamic import
```

---

## Functions & Scopes

### Closures

#### What is a Closure?

A **closure** is a function that has access to variables from its outer (enclosing) lexical scope, even after the outer function has finished executing.

**Why Closures Exist in JavaScript**:

Closures are a natural consequence of:

1. **First-class functions** - functions are values that can be returned
2. **Lexical scoping** - functions remember where they were defined
3. **Persistent scope chain** - inner functions maintain reference to outer scope

#### Lexical Environment

**What is it?**

A **Lexical Environment** is a data structure that holds identifier-variable mapping. Every time code runs, a Lexical Environment is created.

**Structure**:

```text
Lexical Environment {
  Environment Record: {
    // Variables declared in this scope
    variableName: value,
    functionName: function reference
  },
  Outer Environment Reference: <parent Lexical Environment>
}
```

**Example**:

```javascript
function outer() {
  const x = 10; // Stored in outer's Environment Record
  
  function inner() {
    console.log(x); // Accesses outer's Environment Record
  }
  
  return inner;
}

const fn = outer();
fn(); // 10 - still has access to outer's environment!
```

**What happened**:

1. `outer()` executes → creates Lexical Environment A
2. `inner` is defined → stores reference to Environment A (its outer environment)
3. `outer()` returns `inner` → `outer` execution context destroyed
4. BUT: Environment A persists because `inner` still references it
5. `fn()` (which is `inner`) → can still access `x` through Environment A

#### Execution Context

**What is it?**

An **Execution Context** is the environment in which JavaScript code is executed. It contains:

1. **Variable Environment** - where `var` declarations are stored
2. **Lexical Environment** - where `let`/`const` declarations are stored  
3. **This Binding** - value of `this`
4. **Outer Environment Reference** - reference to parent scope

**Types**:

- **Global Execution Context** (GEC) - created when script first runs
- **Function Execution Context** (FEC) - created each time a function is called
- **Eval Execution Context** - created for `eval()` code (rarely used)

**Lifecycle**:

```text
1. Creation Phase:
   - Create Variable/Lexical Environment
   - Create scope chain
   - Determine 'this' value

2. Execution Phase:
   - Assign values to variables
   - Execute code line by line

3. Destruction Phase (when function returns):
   - Execution Context popped from Call Stack
   - BUT: Lexical Environment may persist if closures reference it
```

#### How Closures are Created (Step-by-Step)

```javascript
function createMultiplier(multiplier) {  // Step 1
  return function(num) {                  // Step 2
    return num * multiplier;              // Step 3
  };
}

const double = createMultiplier(2);      // Step 4
console.log(double(5));                  // Step 5: 10
```

**Detailed Steps**:

**Step 1**: `createMultiplier(2)` called

- New Function Execution Context created
- Lexical Environment A created:

```text
Environment A {
  Record: { multiplier: 2 },
  Outer: Global Environment
}
```

**Step 2**: Anonymous function defined inside

- Function object created
- **Critical**: Function stores reference to Environment A in internal `[[Environment]]` property

**Step 3**: Function returned

- `createMultiplier` Execution Context destroyed
- BUT: Environment A **persists** because the returned function references it

**Step 4**: Returned function assigned to `double`

- `double` is now a closure - it "closes over" `multiplier`

**Step 5**: `double(5)` called

- New Execution Context created
- Through `[[Environment]]`, function can access `multiplier` from Environment A
- Returns `5 * 2 = 10`

#### The `[[Environment]]` Hidden Property

Every function has a hidden `[[Environment]]` property that references the Lexical Environment where the function was created.

```javascript
function outer() {
  const secret = 'hidden';
  
  function inner() {
    return secret;
  }
  
  // inner.[[Environment]] → points to outer's Lexical Environment
  return inner;
}
```

You can't access `[[Environment]]` directly, but it's what makes closures work.

#### Scope Chain Walking

When a variable is accessed, JavaScript walks up the scope chain:

```javascript
const global = 'global';

function level1() {
  const level1Var = 'level 1';
  
  function level2() {
    const level2Var = 'level 2';
    
    function level3() {
      console.log(level2Var); // Step 1: Check level3's scope
                              // Step 2: Not found, check level2's scope ✓
      console.log(level1Var); // Check level3 → level2 → level1 ✓
      console.log(global);    // Check level3 → level2 → level1 → global ✓
      console.log(notExist);  // Check all scopes → ReferenceError
    }
    
    return level3;
  }
  
  return level2();
}

const fn = level1();
fn();
```

#### Memory Implications

**Closures keep outer variables alive**:

```javascript
function heavy() {
  const hugeData = new Array(1000000).fill('data'); // ~8MB
  
  return function() {
    console.log('Hello');
    // Doesn't use hugeData, but hugeData can't be garbage collected!
  };
}

const fn = heavy(); // hugeData remains in memory as long as fn exists
```

**Potential Memory Leak**:

```javascript
function attachHandlers() {
  const largeArray = new Array(1000000);
  
  document.getElementById('btn').addEventListener('click', function() {
    // This closure keeps largeArray in memory
    console.log('Clicked');
  });
}

attachHandlers(); // largeArray won't be GC'd until event listener removed
```

**Solution**:

```javascript
function attachHandlers() {
  const largeArray = new Array(1000000);
  // Process largeArray
  const result = process(largeArray);
  
  // Only close over what's needed
  document.getElementById('btn').addEventListener('click', function() {
    console.log(result); // Only 'result' kept in memory, not largeArray
  });
}
```

#### Practical Example: Private Data

```javascript
function createCounter() {
  let count = 0; // Private variable
  
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; }
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
// count is not directly accessible - true encapsulation

// Even this doesn't work:
counter.count = 999; // Creates NEW property, doesn't affect closure
console.log(counter.getCount()); // Still 2
```

**Common Interview Pitfall** (Loop closure):

```javascript
// ❌ Common mistake
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 3, 3, 3 (var has function scope, not block scope)

// ✅ Solutions
// 1. Use let (block scope)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 0, 1, 2

// 2. IIFE closure
for (var i = 0; i < 3; i++) {
  ((j) => {
    setTimeout(() => console.log(j), 100);
  })(i);
}
// Output: 0, 1, 2
```

### Currying

#### What is Currying?

**Currying** is transforming a function that takes multiple arguments into a sequence of functions, each taking a single argument.

**Named after**: Haskell Curry (mathematician/logician)

**Why currying exists**:

- **Partial application**: Create specialized functions from general ones
- **Function composition**: Build complex operations from simple ones
- **Reusability**: Configure functions once, use many times
- **Functional programming**: Core technique in FP paradigms

#### Basic Concept

```javascript
// Regular function (takes all args at once)
function add(a, b, c) {
  return a + b + c;
}

add(1, 2, 3); // 6

// Curried version (takes one arg at a time)
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

curriedAdd(1)(2)(3); // 6

// ES6 arrow syntax (much cleaner)
const add = a => b => c => a + b + c;
```

**How it works**:

```javascript
// Step by step:
const add = a => b => c => a + b + c;

const step1 = add(1);      // Returns: b => c => 1 + b + c
const step2 = step1(2);    // Returns: c => 1 + 2 + c
const result = step2(3);   // Returns: 1 + 2 + 3 = 6

// Or all at once:
add(1)(2)(3); // 6
```

#### Currying vs Partial Application

**Currying**: ALWAYS returns unary functions (one argument at a time)

```javascript
const curriedAdd = a => b => c => a + b + c;

// Must call with one arg at a time:
curriedAdd(1)(2)(3); // ✅
curriedAdd(1, 2, 3); // ❌ Only uses first arg
```

**Partial Application**: Can accept multiple arguments at once

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return (...moreArgs) => curried(...args, ...moreArgs);
  };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

// All these work:
curriedAdd(1)(2)(3);    // ✅ 6
curriedAdd(1, 2)(3);    // ✅ 6
curriedAdd(1)(2, 3);    // ✅ 6
curriedAdd(1, 2, 3);    // ✅ 6
```

#### Currying: Practical Use Cases

**1. Configuration Functions**:

```javascript
// Generic logger
const log = level => message => timestamp => {
  console.log(`[${timestamp}] [${level}] ${message}`);
};

// Create specialized loggers
const errorLog = log('ERROR');
const infoLog = log('INFO');

// Use them
errorLog('Database connection failed')(new Date());
infoLog('User logged in')(new Date());

// Even more specialized
const errorLogNow = errorLog('Something went wrong');
errorLogNow(Date.now());
errorLogNow(Date.now()); // Reuse with different timestamps
```

**2. Event Handlers**:

```javascript
const handleEvent = eventType => element => callback => {
  element.addEventListener(eventType, callback);
};

// Create specialized handlers
const onClick = handleEvent('click');
const onHover = handleEvent('mouseenter');

// Use them
const button = document.querySelector('#myButton');
conClick(button)(() => console.log('Clicked!'));

// Or create even more specialized
const onButtonClick = onClick(button);
onButtonClick(() => console.log('Button clicked'));
onButtonClick(() => console.log('Another handler'));
```

**3. Data Transformation Pipelines**:

```javascript
// Generic transformers
const map = fn => array => array.map(fn);
const filter = predicate => array => array.filter(predicate);
const reduce = reducer => initial => array => array.reduce(reducer, initial);

// Create specialized functions
const double = map(x => x * 2);
const evensOnly = filter(x => x % 2 === 0);
const sum = reduce((acc, n) => acc + n)(0);

// Use them
const numbers = [1, 2, 3, 4, 5];

double(numbers);        // [2, 4, 6, 8, 10]
evensOnly(numbers);     // [2, 4]
sum(numbers);           // 15

// Compose transformations
const result = sum(double(evensOnly(numbers)));
// evensOnly: [2, 4] → double: [4, 8] → sum: 12
```

**4. API Request Builders**:

```javascript
const request = method => url => headers => body => {
  return fetch(url, {
    method,
    headers,
    body: JSON.stringify(body)
  });
};

// Create specialized requests
const get = request('GET');
const post = request('POST');
const delete = request('DELETE');

// Further specialize
const apiGet = get('https://api.example.com');
const apiPost = post('https://api.example.com');

// Use with different endpoints
const authHeaders = { 'Authorization': 'Bearer token' };

apiGet('/users')(authHeaders)(null);
apiPost('/users')(authHeaders)({ name: 'John' });
```

#### Implementing a Generic Curry Function

**Simple curry** (strict: one arg at a time):

```javascript
function curry(fn) {
  return function curried(arg) {
    if (fn.length <= 1) {
      return fn(arg);
    }
    return curry(fn.bind(null, arg));
  };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

curriedAdd(1)(2)(3); // 6
```

**Flexible curry** (partial application):

```javascript
function curry(fn) {
  return function curried(...args) {
    // If all args provided, call function
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    
    // Otherwise, return function waiting for more args
    return function(...moreArgs) {
      return curried.apply(this, [...args, ...moreArgs]);
    };
  };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

// All variations work:
curriedAdd(1)(2)(3);        // 6
curriedAdd(1, 2)(3);        // 6
curriedAdd(1)(2, 3);        // 6
curriedAdd(1, 2, 3);        // 6

// Partial application:
const add5 = curriedAdd(5);
add5(10, 15);               // 30
add5(10)(15);               // 30
```

#### Function Composition with Currying

**Compose**: Apply functions right to left

```javascript
const compose = (...fns) => x => 
  fns.reduceRight((acc, fn) => fn(acc), x);

// With currying:
const add = x => y => x + y;
const multiply = x => y => x * y;
const subtract = x => y => y - x;

const calculate = compose(
  add(1),        // 3. Add 1
  multiply(2),   // 2. Multiply by 2
  subtract(5)    // 1. Subtract from 5
);

calculate(3);    // (3 - 5) * 2 + 1 = -3

// Step by step:
// 1. subtract(5)(3) = 5 - 3 = 2
// 2. multiply(2)(2) = 2 * 2 = 4
// 3. add(1)(4) = 1 + 4 = 5
```

**Pipe**: Apply functions left to right

```javascript
const pipe = (...fns) => x => 
  fns.reduce((acc, fn) => fn(acc), x);

const processUser = pipe(
  user => ({ ...user, name: user.name.toUpperCase() }),
  user => ({ ...user, age: user.age + 1 }),
  user => ({ ...user, active: true })
);

const user = { name: 'john', age: 30 };
processUser(user);
// { name: 'JOHN', age: 31, active: true }
```

#### Real-World Example: Validation

```javascript
// Generic validators (curried)
const minLength = min => value => 
  value.length >= min;

const maxLength = max => value => 
  value.length <= max;

const matches = regex => value => 
  regex.test(value);

const isEmail = matches(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
const isStrongPassword = value => 
  minLength(8)(value) && 
  matches(/[A-Z]/)(value) && 
  matches(/[0-9]/)(value);

// Create specific validators
const isValidUsername = value =>
  minLength(3)(value) && 
  maxLength(20)(value) && 
  matches(/^[a-zA-Z0-9_]+$/)(value);

// Use them
isValidUsername('john_doe');     // true
isValidUsername('ab');           // false (too short)
isEmail('test@example.com');     // true
isStrongPassword('Weak');        // false
isStrongPassword('Strong123');   // true
```

#### Benefits of Currying

**1. Reusability**:

```javascript
const multiply = x => y => x * y;

const double = multiply(2);
const triple = multiply(3);
const quadruple = multiply(4);

double(5);      // 10
triple(5);      // 15
quadruple(5);   // 20
```

**2. Testability**:

```javascript
// Instead of testing with all params:
function sendEmail(to, subject, body, smtp) {
  // ...
}

// Curry it:
const sendEmail = smtp => to => subject => body => {
  // ...
}

// Configure once for tests:
const testEmail = sendEmail(mockSmtp);

// Test different scenarios easily:
testEmail('user@example.com')('Test')('Body');
testEmail('admin@example.com')('Alert')('Message');
```

**3. Delayed Execution**:

```javascript
const calculate = operation => a => b => {
  console.log(`Calculating ${operation}`);
  switch(operation) {
    case 'add': return a + b;
    case 'multiply': return a * b;
  }
};

// Setup operations without executing
const add = calculate('add');
const multiply = calculate('multiply');

// Execute later with specific values
const result1 = add(5)(10);      // "Calculating add" → 15
const result2 = multiply(5)(10); // "Calculating multiply" → 50
```

#### When NOT to Use Currying

**1. Performance-critical code** (function calls have overhead)

```javascript
// Curried (more function calls)
const add = a => b => c => a + b + c;
add(1)(2)(3);

// Regular (single function call)
const add = (a, b, c) => a + b + c;
add(1, 2, 3); // Faster
```

#### 2. Simple one-time operations

```javascript
// Overkill for simple, non-reusable operations
const calculateTotal = tax => shipping => items => 
  items + tax + shipping;

calculateTotal(10)(5)(100); // Unnecessarily complex

// Better:
const calculateTotal = (items, tax, shipping) => 
  items + tax + shipping;

calculateTotal(100, 10, 5); // Clearer
```

#### 3. Dynamic argument counts

```javascript
// When you don't know how many args you'll need
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}

sum(1, 2, 3, 4, 5); // Works

// Currying doesn't make sense here
```

### `this` Context

#### What is `this`?

**`this`** is a special keyword in JavaScript that refers to the context in which a function is executed. Its value is determined at **runtime** based on **how** the function is called, not where it's defined.

**Why `this` exists**:

- Enables code reuse (same function, different contexts)
- Implements object-oriented patterns
- Provides dynamic context binding
- Allows method sharing across objects

**Critical Concept**: `this` is determined by the **call-site** (where the function is invoked), not the **author-site** (where it's written).

#### The Call-Site

**Call-site** = the location in code where a function is called (not where it's declared).

```javascript
function identify() {
  console.log(this.name);
}

const person = { name: 'John' };

// Call-site determines 'this'
identify();          // Call-site: global scope
person.identify = identify;
person.identify();   // Call-site: person object
identify.call(person); // Call-site: explicitly set to person
```

**Finding the call-site**:

```javascript
function outer() {
  // Call-site for inner()
  inner(); // ← THIS is the call-site
}

function inner() {
  console.log(this);
}

outer(); // Call-site for outer()
```

#### Execution Context and ThisBinding

**Every Execution Context has a `ThisBinding` component**:

```text
Execution Context {
  LexicalEnvironment: {...},
  VariableEnvironment: {...},
  ThisBinding: <value of 'this'> ← Determined at creation time
}
```

**When is `this` determined?**

1. Function is called
2. New Execution Context is created
3. `this` is bound based on call-site rules
4. Function executes with that `this` value

#### The Four Binding Rules (Priority Order)

When a function is called, JavaScript applies these rules in order:

#### 1. `new` Binding (Highest Priority)

**When**: Function is called with `new` keyword

**Result**: `this` = newly created object

```javascript
function User(name, age) {
  this.name = name;
  this.age = age;
  // Implicit: return this;
}

const user = new User('John', 30);
// 'this' inside User = the new object
console.log(user.name); // "John"
```

**What `new` does (step-by-step)**:

1. Creates a new empty object: `{}`
2. Links object to constructor's prototype: `obj.__proto__ = User.prototype`
3. Binds `this` to the new object
4. Executes constructor function
5. Returns the object (unless constructor explicitly returns an object)

```javascript
// What happens internally:
const user = (function() {
  const obj = Object.create(User.prototype); // Steps 1 & 2
  User.call(obj, 'John', 30);                 // Steps 3 & 4
  return obj;                                 // Step 5
})();
```

#### 2. Explicit Binding

**When**: Using `.call()`, `.apply()`, or `.bind()`

**Result**: `this` = the object you explicitly specify

**`.call(thisArg, ...args)`**: Calls function with specific `this` and arguments

```javascript
function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const user = { name: 'John' };
greet.call(user, 'Hello', '!'); // "Hello, John!"
// 'this' = user
```

**`.apply(thisArg, [args])`**: Same as `.call()` but takes array of arguments

```javascript
greet.apply(user, ['Hello', '!']); // "Hello, John!"

// Useful with Math functions:
const numbers = [5, 6, 2, 3, 7];
const max = Math.max.apply(null, numbers);
// Modern: Math.max(...numbers)
```

**`.bind(thisArg, ...args)`**: Creates **new function** with `this` permanently bound

```javascript
const boundGreet = greet.bind(user, 'Hello');
boundGreet('!'); // "Hello, John!"

// 'this' is permanently bound, cannot be changed
const other = { name: 'Jane' };
boundGreet.call(other, '!'); // Still "Hello, John!" (not Jane)
```

**Hard Binding**: Once bound with `.bind()`, `this` cannot be overridden

```javascript
function fn() { console.log(this.name); }

const obj1 = { name: 'Object 1' };
const obj2 = { name: 'Object 2' };

const bound = fn.bind(obj1);
bound();              // "Object 1"
bound.call(obj2);     // Still "Object 1" (hard bound)
bound.apply(obj2);    // Still "Object 1"
new bound();          // Only 'new' can override (creates new object)
```

#### 3. Implicit Binding

**When**: Function is called as a method of an object

**Result**: `this` = the object that owns the method

```javascript
const user = {
  name: 'John',
  greet() {
    console.log(`Hello, ${this.name}`);
  }
};

user.greet(); // "Hello, John"
// 'this' = user (the object before the dot)
```

**Call-site Analysis**:

```javascript
obj.method(); // 'this' = obj
obj1.obj2.method(); // 'this' = obj2 (immediate parent)
```

**Lost `this` Problem** (very common bug):

```javascript
const user = {
  name: 'John',
  greet() {
    console.log(`Hello, ${this.name}`);
  }
};

const greet = user.greet; // Reference to function
greet(); // "Hello, undefined"
// 'this' is lost! No implicit binding

// Common scenarios:
setTimeout(user.greet, 1000);        // Lost 'this'
array.forEach(user.greet);           // Lost 'this'
button.addEventListener('click', user.greet); // Lost 'this'
```

**Solutions to Lost `this`**:

```javascript
// Solution 1: Arrow function wrapper
setTimeout(() => user.greet(), 1000);

// Solution 2: .bind()
setTimeout(user.greet.bind(user), 1000);

// Solution 3: Store reference to 'this' (old way)
const self = user;
setTimeout(function() { self.greet(); }, 1000);
```

#### 4. Default Binding (Lowest Priority)

**When**: None of the above rules apply (standalone function call)

**Result**:

- **Non-strict mode**: `this` = global object (window/global)
- **Strict mode**: `this` = undefined

```javascript
function showThis() {
  console.log(this);
}

showThis(); // window (browser) or global (Node.js)

// In strict mode:
'use strict';
function showThis() {
  console.log(this);
}

showThis(); // undefined
```

**Why strict mode changes this**:

- Prevents accidental global variable creation
- Catches errors early
- Global object access should be explicit

```javascript
function createProperty() {
  this.accidental = 'oops'; // Creates global variable (bad!)
}

createProperty();
console.log(window.accidental); // "oops"

// In strict mode:
'use strict';
function createProperty() {
  this.accidental = 'oops'; // TypeError: Cannot set property of undefined
}
```

#### Arrow Functions: No `this` Binding

**Arrow functions DON'T have their own `this`** - they inherit from enclosing lexical scope.

```javascript
const obj = {
  name: 'Object',
  
  regular: function() {
    console.log(this.name); // 'this' determined by call-site
  },
  
  arrow: () => {
    console.log(this.name); // 'this' inherited from outer scope
  }
};

obj.regular(); // "Object" (implicit binding works)
obj.arrow();   // undefined (no implicit binding, inherits from outer)
```

**Arrow functions determine `this` at author-time, not call-time**:

```javascript
function Timer() {
  this.seconds = 0;
  
  setInterval(function() {
    this.seconds++; // ❌ 'this' is NOT Timer instance
  }, 1000);
}

function Timer() {
  this.seconds = 0;
  
  setInterval(() => {
    this.seconds++; // ✅ 'this' IS Timer instance (lexically inherited)
  }, 1000);
}
```

**Arrow functions CANNOT have `this` changed**:

```javascript
const arrow = () => console.log(this);
const obj = { name: 'Object' };

arrow.call(obj);  // 'this' is NOT obj
arrow.apply(obj); // 'this' is NOT obj  
arrow.bind(obj)(); // 'this' is NOT obj

// 'this' is lexically bound, immutable
```

#### Priority Summary

When multiple rules could apply:

```javascript
function fn() { console.log(this.name); }

const obj = { name: 'Object' };

// 1. new binding (highest)
const instance = new fn(); // 'this' = new object (ignores everything else)

// 2. Explicit binding
fn.call(obj); // 'this' = obj (overrides implicit & default)

// 3. Implicit binding
obj.fn = fn;
obj.fn(); // 'this' = obj (overrides default)

// 4. Default binding (lowest)
fn(); // 'this' = window/global or undefined
```

#### Common Pitfalls

**1. Nested Functions Lose `this`**:

```javascript
const obj = {
  name: 'Object',
  method() {
    console.log(this.name); // "Object" ✅
    
    function inner() {
      console.log(this.name); // undefined ❌ (default binding)
    }
    inner();
  }
};

// Solution: Arrow function
const obj = {
  name: 'Object',
  method() {
    const inner = () => {
      console.log(this.name); // "Object" ✅ (inherits from method)
    };
    inner();
  }
};
```

**2. Array Methods**:

```javascript
const obj = {
  name: 'Object',
  items: [1, 2, 3],
  process() {
    this.items.forEach(function(item) {
      console.log(this.name, item); // undefined, 1,2,3 ❌
    });
  }
};

// Solution 1: Arrow function
obj.process = function() {
  this.items.forEach(item => {
    console.log(this.name, item); // ✅
  });
};

// Solution 2: thisArg parameter
obj.process = function() {
  this.items.forEach(function(item) {
    console.log(this.name, item); // ✅
  }, this); // Pass 'this' as second argument
};
```

### Scopes

#### What is Scope?

**Scope** is the accessibility/visibility of variables, functions, and objects in a particular part of your code during runtime.

**Why scope exists**:

- **Namespace management**: Prevent variable name collisions
- **Encapsulation**: Hide implementation details
- **Memory management**: Variables can be garbage collected when out of scope
- **Security**: Limit access to sensitive data

#### Types of Scope

#### 1. Global Scope

**Definition**: Variables accessible from anywhere in the code.

```javascript
var globalVar = 'I am global';
const globalConst = 'Also global';

function anyFunction() {
  console.log(globalVar); // Accessible
}
```

**In browsers**:

```javascript
var x = 10;
console.log(window.x); // 10 (attached to global object)

let y = 20;
console.log(window.y); // undefined (let/const not attached to window)
```

**Global scope pollution** (why it's bad):

```javascript
// library1.js
var config = { theme: 'dark' };

// library2.js
var config = { language: 'en' }; // Overwrites library1's config!
```

#### 2. Function Scope

**Definition**: Variables declared with `var` are scoped to the enclosing function.

```javascript
function example() {
  var x = 10; // Function scoped
  
  if (true) {
    var y = 20; // Still function scoped (NOT block scoped)
    console.log(x); // 10 (accessible)
  }
  
  console.log(y); // 20 (accessible, hoisted to function scope)
}

console.log(x); // ReferenceError (not in scope)
```

**Why `var` ignores blocks**:

```javascript
if (true) {
  var x = 10;
}
console.log(x); // 10 (leaked out of if block!)

for (var i = 0; i < 3; i++) {}
console.log(i); // 3 (leaked out of loop!)
```

#### 3. Block Scope

**Definition**: Variables declared with `let`/`const` are scoped to the enclosing block `{}`.

```javascript
{
  let x = 10;
  const y = 20;
  var z = 30;
}

console.log(x); // ReferenceError (block scoped)
console.log(y); // ReferenceError (block scoped)
console.log(z); // 30 (function scoped, leaked out)
```

**Block scope in `if` statements**:

```javascript
if (true) {
  let x = 10;
  const y = 20;
}
console.log(x); // ReferenceError
```

**Block scope in loops**:

```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 0, 1, 2 (each iteration has its own 'i')

for (var j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100);
}
// Output: 3, 3, 3 (same 'j' shared across iterations)
```

#### 4. Lexical Scope (Static Scope)

**Definition**: Inner functions have access to variables from outer functions (scope chain).

```javascript
function outer() {
  const outerVar = 'I am outer';
  
  function inner() {
    const innerVar = 'I am inner';
    console.log(outerVar); // Accessible (lexical scope)
    console.log(innerVar); // Accessible (own scope)
  }
  
  inner();
  console.log(innerVar); // ReferenceError (not in scope)
}
```

**Scope determined at author-time** (where code is written):

```javascript
const global = 'global';

function foo() {
  const fooVar = 'foo';
  bar(); // What can bar() access?
}

function bar() {
  console.log(fooVar); // ReferenceError
  // bar() was DEFINED in global scope, not inside foo()
}

foo();
```

#### Scope Chain

**How variable lookup works**:

```javascript
const level1 = 'Level 1';

function outer() {
  const level2 = 'Level 2';
  
  function middle() {
    const level3 = 'Level 3';
    
    function inner() {
      const level4 = 'Level 4';
      
      console.log(level4); // 1. Check current scope ✅
      console.log(level3); // 2. Check parent scope ✅
      console.log(level2); // 3. Check grandparent scope ✅
      console.log(level1); // 4. Check global scope ✅
      console.log(notExist); // 5. Not found anywhere ❌ ReferenceError
    }
    
    inner();
  }
  
  middle();
}

outer();
```

**Visualization**:

```text
inner() scope
  │
  └─→ middle() scope
       │
       └─→ outer() scope
            │
            └─→ global scope
                 │
                 └─→ null (end of chain)
```

**Scope chain is one-directional** (inner → outer, not reverse):

```javascript
function outer() {
  const outerVar = 'outer';
  
  function inner() {
    const innerVar = 'inner';
  }
  
  console.log(innerVar); // ReferenceError
  // Cannot access child scope from parent
}
```

#### Hoisting

**What is Hoisting?**

"Hoisting" is often misunderstood as "moving code to the top". Actually, it's how JavaScript's **Execution Context creation** works.

**The Real Process**:

Execution Context creation happens in **two phases**:

**Phase 1: Creation Phase** (Memory Creation)

- Scan through code
- Create Lexical Environment
- Register variable and function declarations
- Allocate memory

**Phase 2: Execution Phase** (Code Execution)

- Execute code line by line
- Assign values to variables

**Variable Hoisting**:

```javascript
console.log(x); // undefined (not ReferenceError!)
var x = 10;
console.log(x); // 10

// What actually happens:
// Phase 1 (Creation):
var x = undefined; // Memory allocated, initialized to undefined

// Phase 2 (Execution):
console.log(x); // undefined
x = 10; // Assignment happens here
console.log(x); // 10
```

**Function Hoisting**:

```javascript
// Can call before declaration!
greet(); // "Hello"

function greet() {
  console.log('Hello');
}

// Function declarations are FULLY hoisted
// (both declaration and body)
```

**Function Expressions are NOT fully hoisted**:

```javascript
greet(); // TypeError: greet is not a function

var greet = function() {
  console.log('Hello');
};

// What happens:
// Phase 1: var greet = undefined;
// Phase 2: greet(); // undefined is not a function
//          greet = function() {...};
```

#### Temporal Dead Zone (TDZ)

**What is TDZ?**

The **Temporal Dead Zone** is the period between entering scope and variable declaration for `let`/`const`.

**`let`/`const` are hoisted BUT NOT initialized**:

```javascript
console.log(x); // ReferenceError: Cannot access 'x' before initialization
let x = 10;

// Execution:
// 1. Enter scope: 'x' exists but is in TDZ
// 2. Try to access: ReferenceError
// 3. Declaration reached: 'x' exits TDZ, becomes usable
```

**TDZ visualization**:

```javascript
// TDZ starts
const func = () => console.log(x); // OK, doesn't access x yet

// Still in TDZ
func(); // ReferenceError: x is in TDZ

let x = 10; // TDZ ends, x is initialized

func(); // 10 (now works)
```

**Why TDZ exists**:

1. **Catch errors early**: Using variables before declaration is usually a bug
2. **`const` semantics**: Can't initialize `const` multiple times
3. **Better code quality**: Encourages declaring variables at the top

```javascript
// Bug caught by TDZ:
function example() {
  console.log(data); // ReferenceError (catches typo)
  
  let date = new Date(); // Probably meant 'date', not 'data'
}
```

#### Variable Lifecycle

**Complete lifecycle** of a variable:

```javascript
// 1. Declaration phase: variable enters scope
let x;

// 2. Initialization phase: variable gets a value
x = 10;

// 3. Assignment phase: variable value changes
x = 20;
```

**Comparison**:

| Declaration | Declaration | Initialization | TDZ? |
|-------------|-------------|----------------|------|
| `var x;` | Hoisted | `undefined` | No |
| `let x;` | Hoisted | Uninitialized | Yes |
| `const x;` | Hoisted | Must initialize at declaration | Yes |
| `function` | Hoisted | Fully hoisted | No |

```javascript
// var: Declaration + Initialization
console.log(x); // undefined (no TDZ)
var x = 10;

// let: Declaration, but NOT Initialization
console.log(y); // ReferenceError (TDZ!)
let y = 10;

// const: Must declare and initialize together
const z; // SyntaxError: Missing initializer
const z = 10; // ✅
```

#### Why `var` is Problematic

**1. Function scope (ignores blocks)**:

```javascript
if (true) {
  var x = 10;
}
console.log(x); // 10 (leaked!)
```

**2. Re-declaration allowed**:

```javascript
var x = 10;
var x = 20; // No error (silently overrides)
```

**3. Hoisting confusion**:

```javascript
function example() {
  console.log(x); // undefined (not ReferenceError)
  var x = 10;
}
```

**4. No TDZ**:

```javascript
console.log(x); // undefined (should be error)
var x = 10;
```

**Modern best practice**: Always use `const`, use `let` when reassignment needed, never use `var`.

#### Block Scope Benefits

**1. Prevents variable leaking**:

```javascript
for (let i = 0; i < 3; i++) {
  // i is scoped to loop
}
console.log(i); // ReferenceError (good!)
```

**2. Memory efficiency**:

```javascript
{
  let hugeArray = new Array(1000000);
  // Use hugeArray
} // hugeArray can be garbage collected here

console.log(hugeArray); // ReferenceError
```

**3. Clearer intent**:

```javascript
if (condition) {
  const result = calculate();
  // result only exists where it's needed
}
// result not polluting outer scope
```

**4. Lexical Scope**: Inner functions access outer variables

```javascript
function outer() {
  const x = 10;
  function inner() {
    console.log(x); // 10 (lexical scope)
  }
  inner();
}
```

### Recursion

#### What is Recursion?

**Recursion** is when a function calls itself until a base condition (terminating case) is met.

**Why recursion exists**:

- **Natural for tree/graph structures**: DOM traversal, file systems
- **Divide and conquer algorithms**: Quick sort, merge sort
- **Mathematical elegance**: Factorial, Fibonacci
- **Backtracking problems**: Maze solving, permutations

**Anatomy of Recursion**:

```javascript
function recursiveFunction(input) {
  // 1. Base Case (terminating condition)
  if (baseConditionMet) {
    return baseValue;
  }
  
  // 2. Recursive Case (moves toward base case)
  return recursiveFunction(smallerInput);
}
```

#### The Call Stack in Recursion

**Each function call creates a new Stack Frame** containing:

- Local variables
- Parameters
- Return address

```javascript
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

factorial(4);
```

**Call Stack visualization**:

```text
factorial(4) called:
  Stack Frame 4: n=4, waiting for factorial(3)
  Stack Frame 3: n=3, waiting for factorial(2)
  Stack Frame 2: n=2, waiting for factorial(1)
  Stack Frame 1: n=1, returns 1
  
Unwinding:
  Stack Frame 1: returns 1
  Stack Frame 2: returns 2 * 1 = 2
  Stack Frame 3: returns 3 * 2 = 6
  Stack Frame 4: returns 4 * 6 = 24
```

**Memory consumption**:

```text
Time Complexity: O(n) - n function calls
Space Complexity: O(n) - n stack frames in memory
```

#### Stack Overflow

**What is Stack Overflow?**

When recursion depth exceeds the **Call Stack limit** (usually 10,000-15,000 calls in browsers).

```javascript
function infiniteRecursion() {
  return infiniteRecursion(); // No base case!
}

infiniteRecursion(); // RangeError: Maximum call stack size exceeded
```

**Common causes**:

1. **Missing base case**:

```javascript
function countdown(n) {
  console.log(n);
  countdown(n - 1); // Never stops!
}
```

**Base case never reached**:

```javascript
function countdown(n) {
  if (n === 0) return;
  console.log(n);
  countdown(n - 2); // If n is odd, skips 0
}

countdown(5); // 5, 3, 1, -1, -3... Stack Overflow!
```

**Input too large**:

```javascript
factorial(100000); // Stack Overflow even with correct base case
```

#### Recursion vs Iteration

**Same problem, different approaches**:

```javascript
// Recursive
function sumRecursive(n) {
  if (n <= 0) return 0;
  return n + sumRecursive(n - 1);
}

// Iterative
function sumIterative(n) {
  let sum = 0;
  for (let i = 1; i <= n; i++) {
    sum += i;
  }
  return sum;
}
```

**Trade-offs**:

| Aspect | Recursion | Iteration |
|--------|-----------|------------|
| **Readability** | Often clearer for tree/graph problems | More verbose |
| **Memory** | O(n) stack space | O(1) constant space |
| **Performance** | Slower (function call overhead) | Faster |
| **Stack Overflow Risk** | Yes | No |
| **Best For** | Trees, graphs, divide-and-conquer | Linear problems, large datasets |

**When to use recursion**:

```javascript
// Tree traversal (recursion is natural)
function traverseDOM(node) {
  console.log(node.nodeName);
  
  for (let child of node.children) {
    traverseDOM(child); // Recursive descent
  }
}

// Iterative version is much more complex:
function traverseDOMIterative(root) {
  const stack = [root];
  
  while (stack.length > 0) {
    const node = stack.pop();
    console.log(node.nodeName);
    
    for (let i = node.children.length - 1; i >= 0; i--) {
      stack.push(node.children[i]);
    }
  }
}
```

#### Tail Recursion

**What is Tail Recursion?**

A recursive call is **tail recursive** if it's the **last operation** in the function (no operations after the call).

**Non-tail recursive** (NOT optimizable):

```javascript
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // ❌ Multiplication AFTER recursive call
}

// Stack must be maintained to perform multiplication
```

**Tail recursive** (optimizable):

```javascript
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc); // ✅ Recursive call is LAST operation
}

// No operations after call, stack frame can be reused
```

#### Tail Call Optimization (TCO)

**What is TCO?**

Compiler optimization that **reuses the same stack frame** for tail-recursive calls.

**Without TCO** (current stack frame kept):

```text
factorialTail(4, 1):
  Stack Frame 4: n=4, acc=1
  Stack Frame 3: n=3, acc=4
  Stack Frame 2: n=2, acc=12
  Stack Frame 1: n=1, acc=24, returns 24
```

**With TCO** (stack frame reused):

```text
factorialTail(4, 1):
  Stack Frame: n=4, acc=1
  Stack Frame: n=3, acc=4   (reused!)
  Stack Frame: n=2, acc=12  (reused!)
  Stack Frame: n=1, acc=24, returns 24
```

**Memory**: O(n) → O(1)

**JavaScript and TCO**:

⚠️ **TCO is NOT widely supported in JavaScript** (only Safari implements it in strict mode).

```javascript
'use strict';

function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc);
}

// In Safari (with TCO): Can handle large n
// In Chrome/Firefox: Stack Overflow at ~10,000
```

**Why TCO isn't implemented**:

- Breaks debugger (stack traces)
- Complicates error handling
- Performance benefits unclear in JavaScript

**Solution**: Convert to iteration for large inputs

```javascript
function factorialIterative(n) {
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

// Safe for any n (no stack overflow)
```

#### Common Recursion Patterns

**1. Linear Recursion** (single recursive call):

```javascript
function sum(n) {
  if (n <= 0) return 0;
  return n + sum(n - 1);
}
```

**2. Binary Recursion** (two recursive calls):

```javascript
function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

// Time: O(2^n) - exponential!
```

**3. Tree Recursion** (multiple recursive calls):

```javascript
function traverseTree(node) {
  if (!node) return;
  
  console.log(node.value);
  
  for (let child of node.children) {
    traverseTree(child); // Multiple recursive calls
  }
}
```

**4. Indirect Recursion** (mutual recursion):

```javascript
function isEven(n) {
  if (n === 0) return true;
  return isOdd(n - 1);
}

function isOdd(n) {
  if (n === 0) return false;
  return isEven(n - 1);
}
```

#### Deep Clone with Recursion

**Practical example**:

```javascript
function deepClone(obj, seen = new WeakMap()) {
  // Base cases
  if (obj === null || typeof obj !== 'object') {
    return obj; // Primitives
  }
  
  // Handle circular references
  if (seen.has(obj)) {
    return seen.get(obj);
  }
  
  // Handle special objects
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  
  // Arrays
  if (Array.isArray(obj)) {
    const cloned = [];
    seen.set(obj, cloned);
    obj.forEach((item, i) => {
      cloned[i] = deepClone(item, seen); // Recursive
    });
    return cloned;
  }
  
  // Objects
  const cloned = {};
  seen.set(obj, cloned);
  
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key], seen); // Recursive
    }
  }
  
  return cloned;
}

// Usage:
const obj = { a: 1, b: { c: 2, d: [3, 4] } };
const copy = deepClone(obj);
```

#### Memoization (Optimization Technique)

**Problem**: Fibonacci has exponential time complexity

```javascript
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}

// fib(5) calls fib(3) twice, fib(2) three times, etc.
// Time: O(2^n)
```

**Solution**: Cache results

```javascript
function fibMemo(n, cache = {}) {
  if (n <= 1) return n;
  
  if (cache[n]) {
    return cache[n]; // Return cached result
  }
  
  cache[n] = fibMemo(n - 1, cache) + fibMemo(n - 2, cache);
  return cache[n];
}

// Time: O(n) - each number calculated once
// Space: O(n) - cache + stack
```

---

## OOP & Prototypes

### Prototypal Inheritance

#### What is Prototypal Inheritance?

**JavaScript uses prototypal inheritance** instead of classical inheritance (like Java/C++).

**Core Concept**: Every object has an internal link to another object called its **prototype**. When a property is accessed, JavaScript searches:

1. The object itself
2. Its prototype
3. The prototype's prototype
4. Until reaching `null`

**Why JavaScript chose prototypes**:

**Historical Context** (1995 - Brendan Eich created JavaScript):

- Needed to compete with Java's "class-based" approach
- But wanted a simpler, more flexible system
- **Prototype-based** was easier to implement in 10 days
- Inspired by **Self** language (prototype-based)

**Benefits of Prototypal Inheritance**:

- **Memory efficient**: Methods shared, not duplicated per instance
- **Dynamic**: Can modify prototypes at runtime
- **Simple**: No complex class hierarchies
- **Flexible**: Objects can inherit from any object

#### The [[Prototype]] Internal Slot

**Every object has a hidden `[[Prototype]]` property** (internal slot).

```javascript
const obj = {};

// Internal structure:
{
  [[Prototype]]: Object.prototype // ← Hidden internal property
}
```

**Accessing [[Prototype]]**:

```javascript
const obj = {};

// Modern way (ES6):
Object.getPrototypeOf(obj); // Object.prototype

// Legacy way (don't use in production):
obj.__proto__; // Object.prototype (property accessor)

// Setting prototype:
const parent = { a: 1 };
const child = Object.create(parent); // Sets [[Prototype]] to parent
```

**`__proto__` vs `prototype` vs `[[Prototype]]`**:

This is **extremely confusing** for beginners:

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

const john = new Person('John');

// [[Prototype]] - Internal slot (not directly accessible)
// john.[[Prototype]] - doesn't exist in code

// __proto__ - Property accessor for [[Prototype]]
john.__proto__ === Person.prototype; // true

// prototype - Property on constructor functions
Person.prototype; // { greet: [Function], constructor: Person }

// Relationship:
john.[[Prototype]] === john.__proto__ === Person.prototype
```

**Terminology**:

- **`[[Prototype]]`**: Internal slot (specification)
- **`__proto__`**: Accessor property (legacy, avoid)
- **`prototype`**: Property on functions (used by `new`)

#### The Prototype Chain

**Property Lookup Algorithm**:

When accessing `obj.property`:

```text
1. Check if 'property' exists on obj itself
   → If yes, return it
   → If no, go to step 2

2. Check if 'property' exists on obj.[[Prototype]]
   → If yes, return it
   → If no, go to step 3

3. Check if 'property' exists on obj.[[Prototype]].[[Prototype]]
   → Continue until reaching null

4. If reaches null (end of chain)
   → Return undefined
```

**Example**:

```javascript
const animal = {
  eats: true,
  walk() {
    console.log('Animal walks');
  }
};

const rabbit = Object.create(animal);
rabbit.jumps = true;

// Prototype chain: rabbit → animal → Object.prototype → null

console.log(rabbit.jumps);    // 1. Found on rabbit (own property)
console.log(rabbit.eats);     // 2. Found on animal (inherited)
console.log(rabbit.toString); // 3. Found on Object.prototype (inherited)
console.log(rabbit.fly);      // 4. Reached null, undefined
```

**Visualization**:

```text
rabbit
  │
  │ [[Prototype]]
  ↓
animal
  │
  │ [[Prototype]]
  ↓
Object.prototype
  │
  │ [[Prototype]]
  ↓
null (end of chain)
```

#### Constructor Functions (Pre-ES6)

**Pattern**:

```javascript
function Person(name, age) {
  // Instance properties
  this.name = name;
  this.age = age;
}

// Shared methods (on prototype)
Person.prototype.greet = function() {
  console.log(`Hello, I'm ${this.name}`);
};

const john = new Person('John', 30);
```

**What `new` does (4 steps)**:

```javascript
function Person(name, age) {
  // When called with 'new':
  
  // 1. Create new empty object
  // const this = {};
  
  // 2. Set [[Prototype]] of object
  // this.[[Prototype]] = Person.prototype;
  
  // 3. Execute constructor, bind 'this'
  this.name = name;
  this.age = age;
  
  // 4. Return 'this' (unless you explicitly return object)
  // return this;
}

const john = new Person('John', 30);
```

**Why use `prototype` for methods?**

**Bad** (methods duplicated per instance):

```javascript
function Person(name) {
  this.name = name;
  
  this.greet = function() { // ❌ New function per instance
    console.log(`Hello, ${this.name}`);
  };
}

const john = new Person('John');
const jane = new Person('Jane');

john.greet === jane.greet; // false (different functions!)

// Memory:
// john: { name: 'John', greet: [Function] }
// jane: { name: 'Jane', greet: [Function] }  ← Wasted memory
```

**Good** (methods shared via prototype):

```javascript
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() { // ✅ Shared function
  console.log(`Hello, ${this.name}`);
};

const john = new Person('John');
const jane = new Person('Jane');

john.greet === jane.greet; // true (same function!)

// Memory:
// john: { name: 'John' } → Person.prototype: { greet: [Function] }
// jane: { name: 'Jane' } → Person.prototype: { greet: [Function] }
```

#### The `prototype` Property

**Only functions have `.prototype` property** (used by `new`):

```javascript
function Person() {}

Person.prototype; // { constructor: Person }

// Regular objects don't have .prototype:
const obj = {};
obj.prototype; // undefined
```

**`prototype.constructor` points back to function**:

```javascript
function Person() {}

Person.prototype.constructor === Person; // true

const john = new Person();
john.constructor === Person; // true (inherited from prototype)
```

**Relationship diagram**:

```text
Person (Function)
  │
  └── .prototype property
      │
      ↓
Person.prototype (Object)
  │
  └── .constructor → Person
  │
  └── methods (greet, etc.)


john (Instance)
  │
  └── [[Prototype]] → Person.prototype
```

#### Property Shadowing

**Own property shadows inherited property**:

```javascript
const animal = {
  eats: true
};

const rabbit = Object.create(animal);

console.log(rabbit.eats); // true (inherited)

// Add own property
rabbit.eats = false;

console.log(rabbit.eats); // false (own property shadows inherited)
console.log(animal.eats); // true (parent unchanged)

// Delete own property
delete rabbit.eats;
console.log(rabbit.eats); // true (inherited again)
```

#### Checking Own vs Inherited Properties

```javascript
const animal = { eats: true };
const rabbit = Object.create(animal);
rabbit.jumps = true;

// hasOwnProperty (checks own properties only)
rabbit.hasOwnProperty('jumps'); // true
rabbit.hasOwnProperty('eats');  // false (inherited)

// in operator (checks entire chain)
'jumps' in rabbit; // true (own)
'eats' in rabbit;  // true (inherited)

// Object.keys (own enumerable only)
Object.keys(rabbit); // ['jumps']

// for...in (own + inherited enumerable)
for (let key in rabbit) {
  console.log(key); // 'jumps', 'eats'
}
```

#### Performance Considerations

**Prototype chain lookup has cost**:

```javascript
const obj = Object.create(null); // No prototype (fastest)

const obj2 = {}; // Prototype: Object.prototype

const obj3 = Object.create(Object.create(Object.create({})));
// Long chain (slower)

// Property access:
obj.property;  // Fastest (checks only obj)
obj2.property; // Slower (checks obj2 → Object.prototype)
obj3.property; // Slowest (checks obj3 → parent → grandparent → Object.prototype)
```

**Best practices**:

- Keep prototype chains short
- Cache frequently accessed inherited properties
- Use own properties for hot paths

```javascript
// Bad (repeated prototype lookups in loop)
for (let i = 0; i < 1000000; i++) {
  obj.inheritedMethod(); // Lookup every iteration
}

// Good (cache the method)
const method = obj.inheritedMethod;
for (let i = 0; i < 1000000; i++) {
  method.call(obj);
}
```

### ES6 Classes

#### What are ES6 Classes?

**ES6 Classes are syntactic sugar over JavaScript's prototypal inheritance**. They provide a cleaner, more familiar syntax similar to class-based languages, but underneath they still use prototypes.

**Why classes were added**:

- **Familiarity**: Developers from Java/C++/Python backgrounds
- **Readability**: Cleaner than constructor functions
- **Standardization**: One clear way to create objects
- **Tooling**: Better IDE support and refactoring

**Critical Understanding**: Classes don't introduce a new inheritance model. They're just a nicer syntax for the same prototype-based system.

#### Class Syntax

```javascript
class Person {
  // Constructor (called when 'new Person()' is invoked)
  constructor(name, age) {
    this.name = name; // Instance properties
    this.age = age;
  }
  
  // Instance method (added to Person.prototype)
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
  
  // Static method (added to Person class itself)
  static species() {
    return 'Homo sapiens';
  }
  
  // Getter
  get info() {
    return `${this.name} (${this.age})`;
  }
  
  // Setter
  set info(value) {
    [this.name, this.age] = value.split(',');
  }
}

const john = new Person('John', 30);
john.greet(); // "Hello, I'm John"
console.log(Person.species()); // "Homo sapiens"
```

#### What Happens Under the Hood

**Class is converted to constructor function + prototype**:

```javascript
// This class:
class Person {
  constructor(name) {
    this.name = name;
  }
  
  greet() {
    console.log(`Hello, ${this.name}`);
  }
}

// Is equivalent to:
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

// Proof:
console.log(typeof Person); // "function" (not a special "class" type)
console.log(Person.prototype.greet); // [Function: greet]
```

**Key Differences from Constructor Functions**:

```javascript
// 1. Class methods are non-enumerable
class Person {
  greet() {}
}

for (let key in Person.prototype) {
  console.log(key); // Nothing logged (non-enumerable)
}

// Constructor function methods ARE enumerable:
function PersonFunc() {}
PersonFunc.prototype.greet = function() {};

for (let key in PersonFunc.prototype) {
  console.log(key); // "greet"
}

// 2. Classes must be called with 'new'
class Person {}
Person(); // TypeError: Class constructor Person cannot be invoked without 'new'

function PersonFunc() {}
PersonFunc(); // Works (but 'this' = window/global)

// 3. Classes are in strict mode by default
class Person {
  test() {
    console.log(this); // undefined if called without context
  }
}

const p = new Person();
const test = p.test;
test(); // undefined (strict mode)

// 4. Classes are not hoisted (TDZ)
const p = new Person(); // ReferenceError
class Person {}

const p2 = new PersonFunc(); // Works (hoisted)
function PersonFunc() {}
```

#### Instance vs Prototype vs Static Members

```javascript
class Person {
  // Instance property (unique per instance)
  constructor(name) {
    this.name = name; // Created on each instance
  }
  
  // Prototype method (shared across instances)
  greet() {
    console.log(`Hello, ${this.name}`);
  }
  
  // Static method (on class itself, not instances)
  static create(name) {
    return new Person(name);
  }
  
  // Static property
  static species = 'Homo sapiens';
}

const john = new Person('John');
const jane = new Person('Jane');

// Instance properties (different for each):
john.name; // "John"
jane.name; // "Jane"
john.name === jane.name; // false

// Prototype methods (same function):
john.greet === jane.greet; // true (shared!)
john.greet === Person.prototype.greet; // true

// Static methods (on constructor):
Person.create('Bob'); // Works
john.create('Bob'); // TypeError: john.create is not a function

// Where things live:
john.hasOwnProperty('name'); // true (instance)
john.hasOwnProperty('greet'); // false (prototype)
Person.hasOwnProperty('create'); // true (constructor)
```

**Memory implications**:

```javascript
// Bad (instance methods - wasted memory):
class Person {
  constructor(name) {
    this.name = name;
    this.greet = function() { // ❌ New function per instance
      console.log(`Hello, ${this.name}`);
    };
  }
}

const john = new Person('John');
const jane = new Person('Jane');
john.greet === jane.greet; // false (two separate functions!)

// Good (prototype methods - shared):
class Person {
  constructor(name) {
    this.name = name;
  }
  
  greet() { // ✅ Shared via prototype
    console.log(`Hello, ${this.name}`);
  }
}

const john2 = new Person('John');
const jane2 = new Person('Jane');
john2.greet === jane2.greet; // true (same function)
```

#### Getters and Setters

**Computed properties** that look like regular properties:

```javascript
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
  
  // Getter (no parentheses when accessing)
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
  
  // Setter (assignment syntax)
  set fullName(value) {
    [this.firstName, this.lastName] = value.split(' ');
  }
  
  get initials() {
    return `${this.firstName[0]}${this.lastName[0]}`;
  }
}

const person = new Person('John', 'Doe');

// Getter (looks like property access):
console.log(person.fullName); // "John Doe" (no parentheses!)
console.log(person.initials); // "JD"

// Setter (looks like assignment):
person.fullName = 'Jane Smith'; // Calls setter
console.log(person.firstName); // "Jane"
console.log(person.lastName); // "Smith"
```

**Use cases**:

- **Computed values**: Calculate on-the-fly
- **Validation**: Control what values are set
- **Legacy compatibility**: Maintain API while changing implementation

```javascript
class Temperature {
  constructor(celsius) {
    this._celsius = celsius; // Backing property
  }
  
  get celsius() {
    return this._celsius;
  }
  
  set celsius(value) {
    if (value < -273.15) {
      throw new Error('Temperature below absolute zero!');
    }
    this._celsius = value;
  }
  
  get fahrenheit() {
    return this._celsius * 9/5 + 32;
  }
  
  set fahrenheit(value) {
    this._celsius = (value - 32) * 5/9;
  }
}

const temp = new Temperature(0);
console.log(temp.celsius);    // 0
console.log(temp.fahrenheit); // 32

temp.fahrenheit = 212;
console.log(temp.celsius);    // 100
```

### Inheritance

#### `extends` Keyword

**Establishes prototype chain** between classes:

```javascript
class Employee extends Person {
  constructor(name, age, salary) {
    super(name, age); // MUST call parent constructor first
    this.salary = salary;
  }
  
  // Override parent method
  greet() {
    super.greet(); // Call parent implementation
    console.log(`I work here`);
  }
  
  getSalary() {
    return this.salary;
  }
}

const emp = new Employee('Jane', 28, 50000);
emp.greet();
// "Hello, I'm Jane"
// "I work here"
```

**What `extends` does**:

```javascript
// Sets up prototype chain:
Employee.prototype.[[Prototype]] = Person.prototype;
Employee.[[Prototype]] = Person;

// Result:
emp → Employee.prototype → Person.prototype → Object.prototype → null
```

#### The `super` Keyword

**Two uses**:

**1. `super()` in constructor** - calls parent constructor:

```javascript
class Employee extends Person {
  constructor(name, age, salary) {
    // Must call super() before accessing 'this'
    super(name, age); // Calls Person constructor
    
    this.salary = salary; // Now can use 'this'
  }
}

// If you forget super():
class Employee extends Person {
  constructor(name, age, salary) {
    this.salary = salary; // ReferenceError: Must call super constructor
  }
}
```

**Why `super()` is required**:

- Parent constructor sets up instance properties
- Child class doesn't create `this` object, parent does
- Ensures proper initialization order

**2. `super.method()` - calls parent method**:

```javascript
class Employee extends Person {
  greet() {
    super.greet(); // Calls Person.prototype.greet()
    console.log('I am an employee');
  }
  
  static create(name, age, salary) {
    super.create(name, age); // Calls Person.create() (static)
    // ...
  }
}
```

**How `super.method()` works**:

```javascript
// In instance methods, super = parent prototype
Employee.prototype.greet = function() {
  Person.prototype.greet.call(this); // This is what super.greet() does
};

// In static methods, super = parent class
Employee.create = function() {
  Person.create(); // This is what super.create() does
};
```

### Object.create() Pattern

**Most explicit prototypal inheritance**:

```javascript
const personMethods = {
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  },
  getAge() {
    return this.age;
  }
};

function createPerson(name, age) {
  const person = Object.create(personMethods);
  person.name = name;
  person.age = age;
  return person;
}

const john = createPerson('John', 30);
john.greet(); // "Hello, I'm John"

// Prototype chain: john → personMethods → Object.prototype → null
```

### Key Differences: Prototypal vs Classical

| Aspect | Prototypal (JS) | Classical (Java/C++) |
|--------|-----------------|----------------------|
| **Inheritance** | Object from object | Class from class |
| **Flexibility** | Dynamic, can modify at runtime | Static, fixed at compile time |
| **Syntax** | Multiple patterns | Single class syntax |
| **Memory** | Shared methods via prototype | Methods copied to instances |
| **Multiple inheritance** | Possible via mixins | Not supported (interfaces instead) |

### Private Fields (Modern JS)

```javascript
class BankAccount {
  #balance = 0; // Private field
  
  constructor(initialBalance) {
    this.#balance = initialBalance;
  }
  
  deposit(amount) {
    this.#balance += amount;
  }
  
  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount(1000);
console.log(account.getBalance()); // 1000
console.log(account.#balance); // SyntaxError: Private field
```

### Singleton Pattern

#### What is the Singleton Pattern?

**Singleton** is a creational design pattern that ensures a class has **only one instance** and provides a **global access point** to that instance.

**Why Singleton exists**:

- **Resource management**: Database connections, file systems (expensive to create)
- **Shared state**: Configuration, logging, caching (needs to be consistent)
- **Control access**: Ensure only one instance manages a resource
- **Lazy initialization**: Create instance only when needed

**Origin**: Part of "Gang of Four" (GoF) Design Patterns book (1994)

#### The Problem Singleton Solves

**Without Singleton**:

```javascript
class Database {
  constructor() {
    this.connection = this.connect();
    console.log('Creating new database connection');
  }
  
  connect() {
    return { status: 'connected', host: 'localhost' };
  }
}

// Problem: Multiple instances = multiple connections
const db1 = new Database(); // "Creating new database connection"
const db2 = new Database(); // "Creating new database connection"
const db3 = new Database(); // "Creating new database connection"

console.log(db1 === db2); // false (different instances!)

// Wasted resources, inconsistent state
```

**With Singleton**:

```javascript
class Database {
  static #instance = null;
  
  constructor() {
    if (Database.#instance) {
      return Database.#instance;
    }
    
    this.connection = this.connect();
    console.log('Creating new database connection');
    Database.#instance = this;
  }
  
  connect() {
    return { status: 'connected', host: 'localhost' };
  }
}

const db1 = new Database(); // "Creating new database connection"
const db2 = new Database(); // (nothing logged)
const db3 = new Database(); // (nothing logged)

console.log(db1 === db2); // true (same instance!)
console.log(db1 === db3); // true
```

#### Implementation Patterns

#### 1. ES6 Class with Private Static Instance

**Modern, recommended approach**:

```javascript
class Database {
  static #instance = null;
  #connection = null;
  
  constructor() {
    // Return existing instance if it exists
    if (Database.#instance) {
      return Database.#instance;
    }
    
    // Initialize connection
    this.#connection = this.#createConnection();
    
    // Store instance
    Database.#instance = this;
  }
  
  #createConnection() {
    console.log('Establishing database connection...');
    return { 
      status: 'connected', 
      host: 'localhost',
      port: 5432
    };
  }
  
  query(sql) {
    console.log(`Executing: ${sql}`);
    return this.#connection;
  }
  
  disconnect() {
    this.#connection = null;
    console.log('Disconnected');
  }
  
  // Alternative: static factory method
  static getInstance() {
    if (!Database.#instance) {
      Database.#instance = new Database();
    }
    return Database.#instance;
  }
}

// Usage
const db1 = new Database();        // Establishes connection
const db2 = new Database();        // Returns existing
const db3 = Database.getInstance(); // Also returns existing

console.log(db1 === db2 === db3); // true
```

#### 2. IIFE with Closure (Pre-ES6)

**Closure-based approach** (before private fields):

```javascript
const DatabaseSingleton = (function() {
  // Private instance variable (closure)
  let instance;
  
  // Private initialization function
  function createInstance() {
    const connection = { 
      status: 'connected',
      host: 'localhost' 
    };
    
    // Public interface
    return {
      query(sql) {
        console.log(`Executing: ${sql}`);
        return connection;
      },
      
      getConnection() {
        return { ...connection }; // Return copy, not reference
      },
      
      disconnect() {
        connection.status = 'disconnected';
        console.log('Disconnected');
      }
    };
  }
  
  // Public API
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

// Usage
const db1 = DatabaseSingleton.getInstance();
const db2 = DatabaseSingleton.getInstance();

console.log(db1 === db2); // true

// Cannot create new instances directly
const db3 = new DatabaseSingleton(); // TypeError: DatabaseSingleton is not a constructor
```

#### 3. ES6 Module Pattern

**Simplest approach using modules**:

```javascript
// database.js
class Database {
  #connection = null;
  
  constructor() {
    this.#connection = this.#createConnection();
  }
  
  #createConnection() {
    console.log('Creating connection');
    return { status: 'connected', host: 'localhost' };
  }
  
  query(sql) {
    console.log(`Executing: ${sql}`);
    return this.#connection;
  }
}

// Create single instance and export it
export default new Database();

// Or with lazy initialization:
let instance = null;

export function getDatabase() {
  if (!instance) {
    instance = new Database();
  }
  return instance;
}
```

```javascript
// Usage in other files
import db from './database.js';
// or
import { getDatabase } from './database.js';
const db = getDatabase();

db.query('SELECT * FROM users');
```

**Why this works**: ES6 modules are **singletons by design** - they're evaluated once and cached.

#### 4. Lazy Initialization

**Only create instance when first accessed**:

```javascript
class Logger {
  static #instance = null;
  #logs = [];
  
  constructor() {
    if (Logger.#instance) {
      throw new Error('Use Logger.getInstance() instead of new');
    }
    Logger.#instance = this;
  }
  
  static getInstance() {
    // Lazy initialization: create only when needed
    if (!Logger.#instance) {
      Logger.#instance = new Logger();
      console.log('Logger instance created');
    }
    return Logger.#instance;
  }
  
  log(message) {
    const entry = `[${new Date().toISOString()}] ${message}`;
    this.#logs.push(entry);
    console.log(entry);
  }
  
  getLogs() {
    return [...this.#logs]; // Return copy
  }
}

// No instance created yet
console.log('App starting...');

// Instance created on first access
const logger = Logger.getInstance(); // "Logger instance created"
logger.log('First log');

// Same instance returned
const logger2 = Logger.getInstance(); // (nothing logged)
console.log(logger === logger2); // true
```

#### Real-World Use Cases

**1. Configuration Manager**:

```javascript
class Config {
  static #instance = null;
  #settings = {};
  
  constructor() {
    if (Config.#instance) {
      return Config.#instance;
    }
    
    // Load configuration from environment
    this.#settings = {
      apiUrl: process.env.API_URL || 'http://localhost:3000',
      maxRetries: parseInt(process.env.MAX_RETRIES) || 3,
      timeout: parseInt(process.env.TIMEOUT) || 5000
    };
    
    Config.#instance = this;
  }
  
  get(key) {
    return this.#settings[key];
  }
  
  set(key, value) {
    this.#settings[key] = value;
  }
}

// Usage across application
const config1 = new Config();
const config2 = new Config();

config1.set('apiUrl', 'https://api.production.com');
console.log(config2.get('apiUrl')); // "https://api.production.com"
// Same configuration everywhere!
```

**2. Cache Manager**:

```javascript
class Cache {
  static #instance = null;
  #cache = new Map();
  
  static getInstance() {
    if (!Cache.#instance) {
      Cache.#instance = new Cache();
    }
    return Cache.#instance;
  }
  
  set(key, value, ttl = 60000) {
    this.#cache.set(key, {
      value,
      expiry: Date.now() + ttl
    });
  }
  
  get(key) {
    const item = this.#cache.get(key);
    
    if (!item) return null;
    
    if (Date.now() > item.expiry) {
      this.#cache.delete(key);
      return null;
    }
    
    return item.value;
  }
  
  clear() {
    this.#cache.clear();
  }
}

// Usage
const cache = Cache.getInstance();
cache.set('user:123', { name: 'John' }, 5000);

// Elsewhere in app
const cache2 = Cache.getInstance();
const user = cache2.get('user:123'); // Same cache!
```

**3. Event Bus**:

```javascript
class EventBus {
  static #instance = null;
  #listeners = new Map();
  
  static getInstance() {
    if (!EventBus.#instance) {
      EventBus.#instance = new EventBus();
    }
    return EventBus.#instance;
  }
  
  on(event, callback) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, []);
    }
    this.#listeners.get(event).push(callback);
  }
  
  emit(event, data) {
    const callbacks = this.#listeners.get(event) || [];
    callbacks.forEach(cb => cb(data));
  }
  
  off(event, callback) {
    const callbacks = this.#listeners.get(event) || [];
    const index = callbacks.indexOf(callback);
    if (index > -1) {
      callbacks.splice(index, 1);
    }
  }
}

// Usage
const bus = EventBus.getInstance();
bus.on('user:login', (user) => console.log(`${user.name} logged in`));

// Elsewhere
const bus2 = EventBus.getInstance();
bus2.emit('user:login', { name: 'John' });
// Same event bus!
```

#### Advantages of Singleton

**1. Controlled access to sole instance**:

- Only one instance exists
- Can't accidentally create multiple

**2. Reduced memory footprint**:

```javascript
// Without Singleton: 3 instances = 3x memory
const db1 = new Database();
const db2 = new Database();
const db3 = new Database();

// With Singleton: 1 instance = 1x memory
const db1 = Database.getInstance();
const db2 = Database.getInstance();
const db3 = Database.getInstance();
// All point to same instance
```

**3. Lazy initialization**:

```javascript
// Resource created only when needed
const logger = Logger.getInstance(); // Created here
```

**4. Global access point**:

```javascript
// Can access from anywhere
function someFunction() {
  const config = Config.getInstance();
  const apiUrl = config.get('apiUrl');
}
```

#### Disadvantages of Singleton

**1. Global State** (biggest problem):

```javascript
// Hard to track who modified state
const config = Config.getInstance();
config.set('mode', 'production');

// Later, somewhere else:
const config2 = Config.getInstance();
console.log(config2.get('mode')); // 'production' (who set this?)
```

**2. Testing Difficulties**:

```javascript
// Tests share same instance
test('test 1', () => {
  const cache = Cache.getInstance();
  cache.set('key', 'value1');
  // ...
});

test('test 2', () => {
  const cache = Cache.getInstance();
  console.log(cache.get('key')); // 'value1' from previous test!
  // Tests are not isolated!
});

// Solution: Add reset method
class Cache {
  static reset() {
    Cache.#instance = null;
  }
}

afterEach(() => {
  Cache.reset();
});
```

**3. Tight Coupling**:

```javascript
// Directly depends on concrete class
class UserService {
  saveUser(user) {
    const db = Database.getInstance(); // Tightly coupled!
    db.query(`INSERT INTO users VALUES (...)`);
  }
}

// Better: Dependency Injection
class UserService {
  constructor(database) {
    this.database = database; // Loosely coupled
  }
  
  saveUser(user) {
    this.database.query(`INSERT INTO users VALUES (...)`);
  }
}
```

**4. Violates Single Responsibility Principle**:

```javascript
// Singleton manages:
// 1. Its own creation (getInstance)
// 2. Its business logic (query, etc.)
// Two responsibilities!
```

#### When to Use Singleton

✅ **Good use cases**:

- **Configuration**: App-wide settings
- **Logging**: Centralized logging service  
- **Caching**: Shared cache
- **Resource pools**: Database connection pools
- **Hardware interfaces**: Printer, GPU access

❌ **Avoid when**:

- You need multiple instances (use factory pattern)
- Testing is important (prefer dependency injection)
- State can vary (use parameter passing)
- Concurrency matters (singletons + threading = problems)

#### Modern Alternative: Dependency Injection

**Instead of Singleton**:

```javascript
// Singleton (global state)
class Database {
  static getInstance() { /*...*/ }
}

class UserService {
  getUser(id) {
    const db = Database.getInstance(); // Hard to test
    return db.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}
```

**Better: Dependency Injection**:

```javascript
// Inject dependency
class UserService {
  constructor(database) {
    this.database = database; // Easy to mock in tests
  }
  
  getUser(id) {
    return this.database.query(`SELECT * FROM users WHERE id = ${id}`);
  }
}

// Usage
const db = new Database();
const userService = new UserService(db);

// Testing
const mockDb = { query: jest.fn() };
const userService = new UserService(mockDb); // Easy to test!
```

---

## Data Structures

### Array Methods

#### What are Higher-Order Functions?

**Higher-order functions** are functions that:

1. **Take functions as arguments**, OR
2. **Return functions as results**

Array methods (`map`, `filter`, `reduce`, etc.) are higher-order functions - they take a function (callback) as an argument.

**Why higher-order functions exist**:

- **Code reuse**: Generic operations that work with any logic
- **Abstraction**: Hide iteration complexity
- **Composition**: Combine simple operations into complex ones
- **Declarative style**: Say "what" not "how"

```javascript
// Imperative (HOW):
const doubled = [];
for (let i = 0; i < numbers.length; i++) {
  doubled.push(numbers[i] * 2);
}

// Declarative (WHAT):
const doubled = numbers.map(n => n * 2);
```

#### Pure Functions

**Pure functions**:

1. **Same input → same output** (deterministic)
2. **No side effects** (don't modify external state)

```javascript
// Pure function ✅
function add(a, b) {
  return a + b; // Always same result for same inputs
}

// Impure function ❌
let total = 0;
function addToTotal(n) {
  total += n; // Modifies external state (side effect)
  return total; // Different result each call
}

addToTotal(5); // 5
addToTotal(5); // 10 (different result!)

// Pure version ✅
function addToTotal(total, n) {
  return total + n; // No external state
}
```

**Why pure functions matter for arrays**:

- **Predictable**: Easy to test and reason about
- **Composable**: Can chain operations
- **Parallelizable**: Safe to run concurrently
- **Cacheable**: Can memoize results

#### Immutability

**Immutability** = not modifying original data, creating new data instead.

**Mutating methods** (modify original array):

```javascript
const arr = [1, 2, 3];

arr.push(4);      // [1, 2, 3, 4] - mutates
arr.pop();        // [1, 2, 3] - mutates
arr.shift();      // [2, 3] - mutates
arr.unshift(0);   // [0, 2, 3] - mutates
arr.splice(1, 1); // [0, 3] - mutates
arr.sort();       // mutates
arr.reverse();    // mutates

// Original array is changed!
```

**Non-mutating methods** (return new array):

```javascript
const arr = [1, 2, 3];

const mapped = arr.map(n => n * 2);     // New array
const filtered = arr.filter(n => n > 1); // New array
const sliced = arr.slice(1);            // New array
const concatenated = arr.concat([4]);   // New array
const spread = [...arr, 4];             // New array

// Original array unchanged!
console.log(arr); // [1, 2, 3]
```

**Why immutability matters**:

- **React/Redux**: State updates must be immutable
- **Debugging**: Track changes over time
- **Undo/Redo**: Keep history of states
- **Concurrent safety**: Avoid race conditions

```javascript
// Bad (mutating state):
const state = { items: [1, 2, 3] };
state.items.push(4); // Mutated!

// Good (immutable update):
const state = { items: [1, 2, 3] };
const newState = { items: [...state.items, 4] }; // New state
```

#### `map()` - Transform Elements

**Purpose**: Create a new array by transforming each element.

**Signature**: `array.map(callback(element, index, array), thisArg)`

**Returns**: New array (same length as original)

```javascript
const numbers = [1, 2, 3, 4, 5];

// Basic transformation
const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8, 10]

// Object transformation
const users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' }
];

const usernames = users.map(u => u.name);
// ['John', 'Jane']

// With index and array parameters
const indexed = numbers.map((num, i, arr) => ({
  value: num,
  index: i,
  isLast: i === arr.length - 1
}));

// Using thisArg
const multiplier = {
  factor: 10,
  multiply(n) {
    return n * this.factor;
  }
};

const result = numbers.map(function(n) {
  return this.multiply(n);
}, multiplier); // Pass 'this' context
// [10, 20, 30, 40, 50]
```

**Key characteristics**:

- **Always returns same length**: 5 elements in → 5 elements out
- **Pure**: Doesn't modify original array
- **Chainable**: Can chain with other methods

```javascript
// map ALWAYS returns same length (even with conditional logic):
const numbers = [1, 2, 3, 4, 5];

const result = numbers.map(n => {
  if (n % 2 === 0) {
    return n * 2;
  }
  // What about odd numbers?
});

console.log(result); // [undefined, 4, undefined, 8, undefined]
// Still 5 elements! Use filter for conditional inclusion
```

#### `filter()` - Select Elements

**Purpose**: Create a new array with elements that pass a test.

**Signature**: `array.filter(callback(element, index, array), thisArg)`

**Returns**: New array (0 to N elements, where N ≤ original length)

```javascript
const numbers = [1, 2, 3, 4, 5, 6];

// Basic filtering
const evens = numbers.filter(n => n % 2 === 0);
// [2, 4, 6]

// Complex filtering
const users = [
  { name: 'John', age: 30, active: true },
  { name: 'Jane', age: 25, active: false },
  { name: 'Bob', age: 35, active: true }
];

const activeAdults = users.filter(u => u.active && u.age >= 30);
// [{ name: 'John', age: 30, active: true }]

// Remove falsy values
const mixed = [0, 1, false, 2, '', 3, null, undefined, 4, NaN];
const truthy = mixed.filter(Boolean);
// [1, 2, 3, 4]

// Remove duplicates (using index trick)
const nums = [1, 2, 2, 3, 3, 4];
const unique = nums.filter((n, i, arr) => arr.indexOf(n) === i);
// [1, 2, 3, 4]
// How it works: indexOf returns FIRST occurrence index
// If current index !== first index, it's a duplicate
```

**Key characteristics**:

- **Variable length**: Returns only elements that pass test
- **Pure**: Doesn't modify original
- **Truthy test**: Callback must return truthy/falsy value

```javascript
// Common mistake: Trying to transform while filtering
const numbers = [1, 2, 3, 4, 5];

// Wrong: filter expects boolean, not transformed value
const doubled = numbers.filter(n => n * 2); // [1, 2, 3, 4, 5] (all truthy!)

// Right: Use filter THEN map
const doubledEvens = numbers
  .filter(n => n % 2 === 0)
  .map(n => n * 2);
// [4, 8]
```

#### `reduce()` - Accumulate/Transform

**Purpose**: Reduce an array to a single value (or object/array).

**Signature**: `array.reduce(callback(accumulator, element, index, array), initialValue)`

**Returns**: Single value (can be any type: number, object, array, etc.)

**This is the most powerful array method** - it can implement map, filter, and more.

**How reduce works**:

```javascript
const numbers = [1, 2, 3, 4, 5];

// Step-by-step execution:
const sum = numbers.reduce((acc, n) => {
  console.log(`acc: ${acc}, n: ${n}`);
  return acc + n;
}, 0);

// Output:
// acc: 0, n: 1   → returns 1
// acc: 1, n: 2   → returns 3
// acc: 3, n: 3   → returns 6
// acc: 6, n: 4   → returns 10
// acc: 10, n: 5  → returns 15
// Final: 15
```

**Without initialValue** (uses first element as initial acc):

```javascript
const numbers = [1, 2, 3, 4, 5];

const sum = numbers.reduce((acc, n) => acc + n);
// Starts: acc = 1, n = 2 (skips first element in iteration)

// ⚠️ Dangerous with empty arrays:
const empty = [];
empty.reduce((acc, n) => acc + n); // TypeError: Reduce of empty array with no initial value

// Safe version:
empty.reduce((acc, n) => acc + n, 0); // 0
```

**Common patterns**:

**1. Sum/Product**:

```javascript
const numbers = [1, 2, 3, 4, 5];

const sum = numbers.reduce((acc, n) => acc + n, 0); // 15
const product = numbers.reduce((acc, n) => acc * n, 1); // 120
```

**2. Object from array**:

```javascript
const users = ['John', 'Jane', 'Bob'];

const userObj = users.reduce((acc, name, i) => {
  acc[i] = name;
  return acc;
}, {});
// { 0: 'John', 1: 'Jane', 2: 'Bob' }

// Or use array keys as object keys:
const fruits = ['apple', 'banana', 'orange'];
const inventory = fruits.reduce((acc, fruit, i) => {
  acc[fruit] = i + 1; // Quantity
  return acc;
}, {});
// { apple: 1, banana: 2, orange: 3 }
```

**3. Group by property**:

```javascript
const products = [
  { name: 'laptop', category: 'electronics', price: 1000 },
  { name: 'phone', category: 'electronics', price: 500 },
  { name: 'shirt', category: 'clothing', price: 50 }
];

const grouped = products.reduce((acc, product) => {
  const { category } = product;
  
  // Initialize array if category doesn't exist
  if (!acc[category]) {
    acc[category] = [];
  }
  
  acc[category].push(product);
  return acc;
}, {});

// Result:
// {
//   electronics: [
//     { name: 'laptop', category: 'electronics', price: 1000 },
//     { name: 'phone', category: 'electronics', price: 500 }
//   ],
//   clothing: [
//     { name: 'shirt', category: 'clothing', price: 50 }
//   ]
// }
```

**4. Flatten array**:

```javascript
const nested = [[1, 2], [3, 4], [5]];

const flattened = nested.reduce((acc, arr) => acc.concat(arr), []);
// [1, 2, 3, 4, 5]

// Modern alternative: flat()
nested.flat(); // [1, 2, 3, 4, 5]
```

**5. Count occurrences**:

```javascript
const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];

const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
// { apple: 3, banana: 2, orange: 1 }
```

**6. Implement map with reduce**:

```javascript
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.reduce((acc, n) => {
  acc.push(n * 2);
  return acc;
}, []);
// [2, 4, 6, 8, 10]

// Same as: numbers.map(n => n * 2)
```

**7. Implement filter with reduce**:

```javascript
const numbers = [1, 2, 3, 4, 5, 6];

const evens = numbers.reduce((acc, n) => {
  if (n % 2 === 0) {
    acc.push(n);
  }
  return acc;
}, []);
// [2, 4, 6]

// Same as: numbers.filter(n => n % 2 === 0)
```

**8. Async reduce** (sequential promises):

```javascript
const urls = ['url1', 'url2', 'url3'];

const results = await urls.reduce(async (accPromise, url) => {
  const acc = await accPromise; // Wait for previous result
  const result = await fetch(url);
  acc.push(result);
  return acc;
}, Promise.resolve([]));

// Fetches sequentially: url1 → url2 → url3
```

#### `sort()` - Order Elements

**⚠️ Mutates original array!**

```javascript
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];

// Default (converts to strings, lexicographic order)
numbers.sort(); // [1, 1, 2, 3, 4, 5, 6, 9] ✅
// BUT: [10, 5, 40, 25].sort() → [10, 25, 40, 5] ❌

// Numeric sort
numbers.sort((a, b) => a - b); // Ascending
numbers.sort((a, b) => b - a); // Descending

// Object sort
const users = [
  { name: 'John', age: 30 },
  { name: 'Jane', age: 25 },
  { name: 'Bob', age: 35 }
];

// Sort by age
users.sort((a, b) => a.age - b.age);

// Sort by name (strings)
users.sort((a, b) => a.name.localeCompare(b.name));

// Multi-level sort (by age, then name)
users.sort((a, b) => {
  if (a.age !== b.age) return a.age - b.age;
  return a.name.localeCompare(b.name);
});

// Non-mutating sort
const sorted = [...numbers].sort((a, b) => a - b);
```

#### Method Chaining

```javascript
const users = [
  { name: 'John', age: 30, active: true },
  { name: 'Jane', age: 25, active: false },
  { name: 'Bob', age: 35, active: true },
  { name: 'Alice', age: 28, active: true }
];

// Get names of active users over 25, sorted
const result = users
  .filter(u => u.active)
  .filter(u => u.age > 25)
  .map(u => u.name)
  .sort();
// ['Bob', 'John']

// Calculate average age of active users
const avgAge = users
  .filter(u => u.active)
  .reduce((sum, u, i, arr) => {
    sum += u.age;
    return i === arr.length - 1 ? sum / arr.length : sum;
  }, 0);
// 31
```

**Performance Tip**: Each method iterates the entire array. For large datasets, consider combining operations:

```javascript
// ❌ Multiple iterations
const result = arr
  .filter(x => x > 0)
  .map(x => x * 2)
  .filter(x => x < 100);

// ✅ Single iteration with reduce
const result = arr.reduce((acc, x) => {
  if (x > 0) {
    const doubled = x * 2;
    if (doubled < 100) acc.push(doubled);
  }
  return acc;
}, []);
```

### Objects vs. Maps/Sets

#### Objects

**Best for**: Structured data with known keys

```javascript
const user = {
  name: 'John',
  age: 30,
  'complex-key': 'value',
  [Symbol('id')]: 123
};

// Access
user.name; // 'John'
user['complex-key']; // 'value'

// Iteration (keys are strings/symbols)
Object.keys(user); // ['name', 'age', 'complex-key']
Object.values(user); // ['John', 30, 'value']
Object.entries(user); // [['name', 'John'], ...]

// Check property
'name' in user; // true
user.hasOwnProperty('name'); // true
```

**Limitations**:

- Keys must be strings or symbols
- No built-in size property
- Prototype chain can cause issues
- Order not guaranteed (before ES2015)

#### Map

**Best for**: Dynamic key-value pairs, especially with non-string keys

```javascript
const map = new Map();

// Any type as key
map.set('string', 'value');
map.set(42, 'number key');
map.set({ id: 1 }, 'object key');
map.set(true, 'boolean key');

// Access
map.get('string'); // 'value'
map.has(42); // true
map.delete(42);
map.size; // 3

// Iteration (maintains insertion order)
for (const [key, value] of map) {
  console.log(key, value);
}

map.forEach((value, key) => {
  console.log(key, value);
});

// Convert to/from objects
const obj = Object.fromEntries(map);
const map2 = new Map(Object.entries(obj));
```

**Advantages over Objects**:

- Any value as key (objects, functions, primitives)
- `.size` property
- Direct iteration
- Better performance for frequent additions/deletions
- No prototype pollution

#### Set

**Best for**: Unique value collections

```javascript
const set = new Set([1, 2, 3, 3, 4, 4]);
console.log(set); // Set {1, 2, 3, 4}

// Add/remove
set.add(5);
set.delete(2);
set.has(3); // true
set.size; // 4

// Iteration
for (const value of set) {
  console.log(value);
}

// Convert to array
const arr = [...set];
const arr2 = Array.from(set);

// Practical: Remove duplicates
const numbers = [1, 2, 2, 3, 3, 4];
const unique = [...new Set(numbers)];
// [1, 2, 3, 4]

// Set operations
const setA = new Set([1, 2, 3]);
const setB = new Set([3, 4, 5]);

// Union
const union = new Set([...setA, ...setB]);
// Set {1, 2, 3, 4, 5}

// Intersection
const intersection = new Set([...setA].filter(x => setB.has(x)));
// Set {3}

// Difference
const difference = new Set([...setA].filter(x => !setB.has(x)));
// Set {1, 2}
```

#### Comparison Table

| Feature | Object | Map | Set |
|---------|--------|-----|-----|
| **Key types** | String, Symbol | Any | N/A (values only) |
| **Size** | Manual (`Object.keys().length`) | `.size` | `.size` |
| **Iteration** | `Object.keys/values/entries` | Direct iteration | Direct iteration |
| **Insertion order** | Maintained (ES2015+) | Maintained | Maintained |
| **Performance** | Good for static structure | Better for dynamic operations | Unique values |
| **Use case** | Structured data | Dynamic key-value | Unique collections |
| **JSON** | Native support | Manual conversion | Manual conversion |

**When to Use What**:

- **Object**: Configuration, POJOs, static structure, JSON data
- **Map**: Cache, metadata, non-string keys, frequent add/delete
- **Set**: Unique values, deduplication, mathematical set operations

---

## Coding Task: Group Books by Decade

### Problem Statement

**Task**: Given an array of books with publication years, group them by decade.

**Input**:

```javascript
const books = [
  { title: '1984', year: 1949 },
  { title: 'To Kill a Mockingbird', year: 1960 },
  { title: 'The Great Gatsby', year: 1925 },
  { title: 'Harry Potter', year: 1997 },
  { title: 'The Hobbit', year: 1937 },
  { title: 'The Catcher in the Rye', year: 1951 },
  { title: 'Pride and Prejudice', year: 1813 },
  { title: 'The Hunger Games', year: 2008 }
];
```

**Expected Output**:

```javascript
{
  '1810s': [{ title: 'Pride and Prejudice', year: 1813 }],
  '1920s': [{ title: 'The Great Gatsby', year: 1925 }],
  '1930s': [{ title: 'The Hobbit', year: 1937 }],
  '1940s': [{ title: '1984', year: 1949 }],
  '1950s': [{ title: 'The Catcher in the Rye', year: 1951 }],
  '1960s': [{ title: 'To Kill a Mockingbird', year: 1960 }],
  '1990s': [{ title: 'Harry Potter', year: 1997 }],
  '2000s': [{ title: 'The Hunger Games', year: 2008 }]
}
```

### Understanding the Problem

**What we need to do**:

1. **Group data** by a computed property (decade)
2. **Transform** array into object structure
3. **Calculate** decade from year
4. **Accumulate** books into their respective decade arrays

**Key insight**: This is a **data transformation** problem - converting a flat array into a grouped object structure.

### Solution 1: Using `reduce()`

```javascript
const groupByDecade = books.reduce((acc, book) => {
  // Calculate decade (e.g., 1949 → 1940s)
  const decade = Math.floor(book.year / 10) * 10;
  const decadeKey = `${decade}s`;
  
  // Initialize array if decade doesn't exist
  if (!acc[decadeKey]) {
    acc[decadeKey] = [];
  }
  
  // Add book to decade group
  acc[decadeKey].push(book);
  
  return acc;
}, {});

console.log(groupByDecade);
/*
{
  '1810s': [{ title: 'Pride and Prejudice', year: 1813 }],
  '1920s': [{ title: 'The Great Gatsby', year: 1925 }],
  '1930s': [{ title: 'The Hobbit', year: 1937 }],
  '1940s': [{ title: '1984', year: 1949 }],
  '1950s': [{ title: 'The Catcher in the Rye', year: 1951 }],
  '1960s': [{ title: 'To Kill a Mockingbird', year: 1960 }],
  '1990s': [{ title: 'Harry Potter', year: 1997 }],
  '2000s': [{ title: 'The Hunger Games', year: 2008 }]
}
*/
```

### Solution 2: Using Map (Better for Performance)

```javascript
const groupByDecadeMap = books.reduce((map, book) => {
  const decade = Math.floor(book.year / 10) * 10;
  const decadeKey = `${decade}s`;
  
  // Get existing array or create new one
  const group = map.get(decadeKey) || [];
  group.push(book);
  map.set(decadeKey, group);
  
  return map;
}, new Map());

// Convert to object if needed
const result = Object.fromEntries(groupByDecadeMap);
```

### Solution 3: Generic Grouping Function

```javascript
function groupBy(array, keyFn) {
  return array.reduce((acc, item) => {
    const key = keyFn(item);
    
    if (!acc[key]) {
      acc[key] = [];
    }
    
    acc[key].push(item);
    return acc;
  }, {});
}

// Usage
const byDecade = groupBy(books, book => {
  const decade = Math.floor(book.year / 10) * 10;
  return `${decade}s`;
});

// Other use cases
const byAuthor = groupBy(books, book => book.author);
const byCentury = groupBy(books, book => {
  const century = Math.ceil(book.year / 100);
  return `${century}th century`;
});
```

### Solution 4: Using Object.groupBy() (ES2024)

```javascript
// Modern JS (Node 21+, Chrome 117+)
const grouped = Object.groupBy(books, book => {
  const decade = Math.floor(book.year / 10) * 10;
  return `${decade}s`;
});

// For Map instead of Object
const groupedMap = Map.groupBy(books, book => {
  const decade = Math.floor(book.year / 10) * 10;
  return `${decade}s`;
});
```

### Additional Operations

```javascript
// Get decade with most books
const decades = Object.entries(groupByDecade);
const mostPopular = decades.reduce((max, [decade, books]) => 
  books.length > max.count ? { decade, count: books.length } : max,
  { decade: null, count: 0 }
);

// Sort decades chronologically
const sortedDecades = Object.entries(groupByDecade)
  .sort(([a], [b]) => parseInt(a) - parseInt(b));

// Get books from specific decade range
const modernBooks = Object.entries(groupByDecade)
  .filter(([decade]) => parseInt(decade) >= 1900)
  .flatMap(([_, books]) => books);
```

### Detailed Step-by-Step Walkthrough

**Let's trace through the first few iterations**:

```javascript
// Initial state
const acc = {};
const books = [
  { title: '1984', year: 1949 },
  { title: 'To Kill a Mockingbird', year: 1960 },
  // ...
];

// Iteration 1: { title: '1984', year: 1949 }
const decade = Math.floor(1949 / 10) * 10; // 194 * 10 = 1940
const decadeKey = '1940s';
// acc['1940s'] doesn't exist, create it
acc['1940s'] = [];
acc['1940s'].push({ title: '1984', year: 1949 });
// acc = { '1940s': [{ title: '1984', year: 1949 }] }

// Iteration 2: { title: 'To Kill a Mockingbird', year: 1960 }
const decade = Math.floor(1960 / 10) * 10; // 196 * 10 = 1960
const decadeKey = '1960s';
// acc['1960s'] doesn't exist, create it
acc['1960s'] = [];
acc['1960s'].push({ title: 'To Kill a Mockingbird', year: 1960 });
// acc = {
//   '1940s': [{ title: '1984', year: 1949 }],
//   '1960s': [{ title: 'To Kill a Mockingbird', year: 1960 }]
// }

// ... continues for all books
```

**Why `Math.floor(year / 10) * 10` works**:

```javascript
// Examples:
1949 / 10 = 194.9
Math.floor(194.9) = 194
194 * 10 = 1940 ✅

1960 / 10 = 196.0
Math.floor(196.0) = 196
196 * 10 = 1960 ✅

2008 / 10 = 200.8
Math.floor(200.8) = 200
200 * 10 = 2000 ✅

// This works for any year!
```

### Performance Comparison

```javascript
// Solution 1: Object (O(n))
const objectResult = books.reduce((acc, book) => {
  const decade = Math.floor(book.year / 10) * 10;
  const key = `${decade}s`;
  if (!acc[key]) acc[key] = [];
  acc[key].push(book);
  return acc;
}, {});

// Solution 2: Map (O(n), but faster)
const mapResult = books.reduce((map, book) => {
  const decade = Math.floor(book.year / 10) * 10;
  const key = `${decade}s`;
  const group = map.get(key) || [];
  group.push(book);
  map.set(key, group);
  return map;
}, new Map());

// Solution 3: for...of loop (most explicit)
const result = {};
for (const book of books) {
  const decade = Math.floor(book.year / 10) * 10;
  const key = `${decade}s`;
  
  if (!result[key]) {
    result[key] = [];
  }
  
  result[key].push(book);
}

// All have O(n) time complexity
// Map is slightly faster for large datasets
// Object is more idiomatic for small datasets
```

### Common Mistakes to Avoid

**1. Forgetting to initialize arrays**:

```javascript
// ❌ Wrong
const result = books.reduce((acc, book) => {
  const decade = Math.floor(book.year / 10) * 10;
  const key = `${decade}s`;
  
  acc[key].push(book); // TypeError: Cannot read property 'push' of undefined
  return acc;
}, {});

// ✅ Correct
const result = books.reduce((acc, book) => {
  const decade = Math.floor(book.year / 10) * 10;
  const key = `${decade}s`;
  
  if (!acc[key]) acc[key] = []; // Initialize first
  acc[key].push(book);
  return acc;
}, {});
```

**2. Not returning accumulator**:

```javascript
// ❌ Wrong
const result = books.reduce((acc, book) => {
  const decade = Math.floor(book.year / 10) * 10;
  const key = `${decade}s`;
  
  if (!acc[key]) acc[key] = [];
  acc[key].push(book);
  // Missing return! acc becomes undefined in next iteration
}, {});

// ✅ Correct
const result = books.reduce((acc, book) => {
  const decade = Math.floor(book.year / 10) * 10;
  const key = `${decade}s`;
  
  if (!acc[key]) acc[key] = [];
  acc[key].push(book);
  return acc; // Always return accumulator
}, {});
```

**3. Wrong decade calculation**:

```javascript
// ❌ Wrong (doesn't round down)
const decade = (book.year / 10) * 10; // 1949 → 1949 (not rounded!)

// ❌ Wrong (string concatenation issue)
const decade = book.year.toString().slice(0, 3) + '0'; // "194" + "0" = "1940" (string!)

// ✅ Correct
const decade = Math.floor(book.year / 10) * 10; // 1949 → 1940 (number)
```

### Key Takeaways

**1. `reduce()` is perfect for array → object transformations**:

```javascript
// Pattern:
array.reduce((accumulator, item) => {
  // Calculate key
  const key = computeKey(item);
  
  // Initialize if needed
  if (!accumulator[key]) {
    accumulator[key] = initialValue;
  }
  
  // Update accumulator
  accumulator[key] = updateValue(accumulator[key], item);
  
  // Return accumulator
  return accumulator;
}, {});
```

**2. Always initialize accumulator with correct type**:

- `{}` for objects
- `[]` for arrays
- `0` for numbers
- `''` for strings
- `new Map()` for maps
- `new Set()` for sets

**3. Check if key exists before accessing**:

```javascript
// Safe patterns:
if (!acc[key]) acc[key] = [];
acc[key] = acc[key] || [];
const value = acc[key] || defaultValue;
```

**4. Consider using `Map` for dynamic keys**:

```javascript
// Object keys are always strings
const obj = {};
obj[1] = 'one';
obj['1'] = 'ONE';
console.log(obj); // { '1': 'ONE' } (only one key!)

// Map keys can be any type
const map = new Map();
map.set(1, 'one');
map.set('1', 'ONE');
console.log(map); // Map { 1 => 'one', '1' => 'ONE' } (two keys!)
```

**5. This pattern works for any grouping operation**:

```javascript
// Group by any property
const groupBy = (array, keyFn) => 
  array.reduce((acc, item) => {
    const key = keyFn(item);
    if (!acc[key]) acc[key] = [];
    acc[key].push(item);
    return acc;
  }, {});

// Examples:
groupBy(books, b => b.author);              // By author
groupBy(books, b => b.genre);               // By genre
groupBy(books, b => b.year > 2000);         // By condition (true/false)
groupBy(users, u => u.age >= 18 ? 'adult' : 'minor'); // By computed value
```

### Interview Follow-up Questions

**Q: How would you handle missing years?**

```javascript
const groupByDecade = books.reduce((acc, book) => {
  if (!book.year) return acc; // Skip books without year
  
  const decade = Math.floor(book.year / 10) * 10;
  const key = `${decade}s`;
  
  if (!acc[key]) acc[key] = [];
  acc[key].push(book);
  return acc;
}, {});
```

**Q: How would you sort the result?**

```javascript
// Object.entries() → sort → Object.fromEntries()
const sorted = Object.fromEntries(
  Object.entries(grouped).sort(([a], [b]) => 
    parseInt(a) - parseInt(b)
  )
);
```

**Q: What's the time/space complexity?**

- **Time**: O(n) - single pass through array
- **Space**: O(n) - all books stored in result
- **Optimal**: Can't do better than O(n) since we need to process all items

**Q: How would you make it work with ES2024's `Object.groupBy()`?**

```javascript
// Modern approach (Node 21+, Chrome 117+)
const grouped = Object.groupBy(books, book => {
  const decade = Math.floor(book.year / 10) * 10;
  return `${decade}s`;
});

// Same result, cleaner syntax, built-in!
```
