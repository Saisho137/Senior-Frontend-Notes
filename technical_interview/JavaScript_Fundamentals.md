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

**Asynchronous programming** allows operations to run independently without blocking the main thread. When long-running operations (network requests, file I/O, database queries) complete, they notify the program via callbacks, promises, or events.

**Key Concepts**:

- **Synchronous**: Operations execute sequentially; each must complete before the next starts
- **Asynchronous**: Operations start and code continues executing; completion notifies via callback/promise

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

JavaScript delegates I/O operations (file system, network, database) to system APIs that handle them in the background. When complete, callbacks are queued in the Event Loop.

```javascript
// Blocking (sync)
const data = readFileSync('file.txt'); // Waits until complete

// Non-blocking (async)
readFile('file.txt', (data) => console.log(data));
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

### Async Mechanisms

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

**Browser**: `setTimeout`, `fetch`, DOM events, `requestAnimationFrame`

**Node.js**: File system, network, timers, `process.nextTick`

#### 3. Macrotask Queue

FIFO queue for async callbacks: `setTimeout`, I/O, `setImmediate`, UI rendering. Processes **one per cycle**.

#### 4. Microtask Queue

**Higher priority** than macrotasks. Contains: `Promise` callbacks, `queueMicrotask()`, `async/await`. Processes **ALL** before next macrotask.

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
function bad() {
  queueMicrotask(() => { console.log('Loop'); bad(); });
}
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

#### Node.js

6 phases (timers → poll → check). Has `process.nextTick()` (highest priority) and `setImmediate()`.

```javascript
process.nextTick(() => console.log('nextTick')); // Executes first
Promise.resolve().then(() => console.log('promise')); // Then microtasks
setTimeout(() => console.log('timeout'), 0); // Then timers phase
setImmediate(() => console.log('immediate')); // Then check phase
```

**Execution Order**: Sync → ALL Microtasks → ONE Macrotask → Repeat

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

Promises are **immutable** once settled (first `resolve`/`reject` wins).

### Creation

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

// allSettled: Returns all outcomes
const results = await Promise.allSettled([fetch('/api/1'), fetch('/api/2')]);
// [{status: 'fulfilled', value: ...}, {status: 'rejected', reason: ...}]

// race: First to finish
await Promise.race([fetch('/api'), timeout(5000)]);

// any: First success (ignores failures until all fail)
await Promise.any([fetch('/mirror1'), fetch('/mirror2')]);
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

### Callback Hell (Pyramid of Doom)

```javascript
// ❌ Nested callbacks
getData(id, (err, user) => {
  if (err) return handle(err);
  getOrders(user.id, (err, orders) => {
    if (err) return handle(err);
    process(orders[0], (err, result) => {
      if (err) return handle(err);
      console.log(result); // Deep nesting →
    });
  });
});
```

**Problems**: Deep nesting, repetitive error handling, inversion of control (trust third-party to call callback correctly), no composition, hard debugging.

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

Extract values from arrays/objects into variables:

```javascript
// Arrays
const [first, second, ...rest] = [1, 2, 3, 4, 5];
const [a, , c] = [1, 2, 3]; // Skip elements
[x, y] = [y, x]; // Swap

// Objects
const { name, age } = user;
const { name: userName } = user; // Rename
const { age = 18, role = 'user' } = user; // Defaults
const { address: { city } } = user; // Nested

// Function params (named params pattern)
function create({ name, age, role = 'user' }) { }
create({ name: 'John', age: 30 }); // Order doesn't matter
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

```javascript
// Arrays
const combined = [...arr1, ...arr2];
const copy = [...arr];

// Objects (order matters)
const config = { ...defaults, ...user }; // user overrides defaults

// Functions
Math.max(...numbers); // spread into arguments

// Strings (handles Unicode)
[...'👨‍👩‍👧‍👦'].length; // 1 (correct)
```

#### Rest (Collecting)

```javascript
// Functions (must be last parameter)
function sum(...nums) { return nums.reduce((a, b) => a + b, 0); }
function greet(msg, ...names) { return `${msg} ${names.join(', ')}`; }

// Destructuring
const [first, ...rest] = [1, 2, 3, 4]; // rest = [2, 3, 4]
const { name, ...address } = user; // Extract name, rest in address

// Remove properties
const { password, ...safe } = user; // Remove password
```

#### ⚠️ Shallow Copy

```javascript
const copy = {...obj}; // Nested objects still reference originals!
copy.nested.x = 5; // Mutates original.nested

// Deep copy: JSON.parse(JSON.stringify(obj)) or structuredClone(obj)
```

### Arrow Functions

**Syntax**:

```javascript
// Basic
const add = (a, b) => a + b;
x => x * 2; // Single param (no parens)
() => 'Hello'; // No params

// Implicit return (wrap objects in parens)
id => ({ id, name: 'John' });
```

#### Lexical `this` Binding

**The MAIN difference from regular functions**: Arrow functions don't create their own `this` - they inherit it from the enclosing scope at **definition time** (not call time).

```javascript
const obj = {
  name: 'Object',
  regular: function() { console.log(this.name); }, // 'this' = obj
  arrow: () => { console.log(this.name); }          // 'this' = global/undefined
};

obj.regular(); // "Object"
obj.arrow();   // undefined

// Why it's useful: Callbacks preserve 'this'
class Timer {
  constructor() {
    this.seconds = 0;
    setInterval(() => this.seconds++, 1000); // ✅ 'this' preserved
    // vs: setInterval(function() { this.seconds++; }, 1000); // ❌ 'this' lost
  }
}
```

> **See also**: [`this` Context](#this-context) for complete binding rules and priority.

#### What Arrow Functions DON'T Have

| Feature | Arrow Functions | Regular Functions |
|---------|-----------------|-------------------|
| `this` binding | ❌ Inherits from scope | ✅ Own binding |
| `arguments` object | ❌ Use `...rest` instead | ✅ Available |
| `new` constructor | ❌ TypeError | ✅ Works |
| `prototype` property | ❌ undefined | ✅ Exists |
| `call/apply/bind` for `this` | ❌ Can't change | ✅ Works |

#### When to Use (and When NOT to)

| Use Case | Arrow ✅ | Regular ✅ |
|----------|---------|------------|
| Callbacks (`map`, `filter`, etc.) | ✅ Preferred | ✓ Works |
| Preserving `this` in classes | ✅ Ideal | ❌ Needs `.bind()` |
| Object methods | ❌ No own `this` | ✅ Required |
| Constructors (`new`) | ❌ TypeError | ✅ Required |
| Event handlers needing element | ❌ `this` is outer | ✅ `this` is element |
| Dynamic `this` with `call/apply` | ❌ Can't change | ✅ Works |

```javascript
// ✅ Callbacks: Arrow is cleaner
numbers.map(n => n * 2);
fetch(url).then(res => res.json()).then(data => this.process(data));

// ✅ Object methods: Use regular
const user = {
  name: 'John',
  greet() { console.log(this.name); } // ✅ 'this' = user
  // greet: () => { console.log(this.name); } // ❌ undefined
};

// ✅ Event handlers needing element: Use regular
button.addEventListener('click', function() {
  this.classList.toggle('active'); // 'this' = button element
});
```

### Template Literals

**Interpolation, expressions, multi-line**:

```javascript
// Basic interpolation
const msg = `Hello ${name}, you are ${age} years old.`;

// Expressions & function calls
`Sum: ${a + b}`, `Result: ${fn()}`, `Name: ${getFullName('John', 'Doe')}`

// Multi-line (preserves whitespace)
const html = `
  <div class="user-card">
    <h1>${title}</h1>
    <p class="${isActive ? 'active' : 'inactive'}">${user.name}</p>
  </div>
`;
```

#### Tagged Templates (Advanced)

**Tag functions** process template literals with full control over string construction:

```javascript
function myTag(strings, ...values) {
  // strings = ['Hello ', ' and ', '!']
  // values = ['John', 'Jane']
  return strings.reduce((acc, s, i) => acc + s + (values[i] || ''), '');
}

myTag`Hello ${'John'} and ${'Jane'}!`;

// Common uses:
// HTML escaping
function safeHTML(strings, ...values) {
  const escape = v => String(v).replace(/</g, '&lt;').replace(/>/g, '&gt;');
  return strings.reduce((a, s, i) => a + s + (values[i] ? escape(values[i]) : ''), '');
}
const safe = safeHTML`<div>${userInput}</div>`;

// Popular libraries: styled-components, GraphQL
const Button = styled.button`background: ${props => props.primary ? 'blue' : 'gray'};`;
const query = gql`query GetUser($id: ID!) { user(id: $id) { name } }`;

// String.raw: Literal backslashes
String.raw`C:\Users\file.txt`; // No escaping needed
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

### Core Concept: Weak References

**WeakMap** and **WeakSet** hold **weak references** to keys/values - they don't prevent garbage collection.

| Aspect | Strong Reference (Map/Set) | Weak Reference (WeakMap/WeakSet) |
|--------|---------------------------|----------------------------------|
| **Keys** | Any type | Objects only |
| **GC** | Prevents collection | Allows collection |
| **Iteration** | ✅ `.keys()`, `.values()`, `.forEach()` | ❌ Not possible |
| **`.size`** | ✅ Available | ❌ Not available |
| **Memory Leaks** | Possible if not cleaned | Auto-prevented |

### Why They Exist

```javascript
// Problem with Map: Memory leak
const cache = new Map();
let obj = { data: 'huge' };
cache.set(obj, 'metadata');
obj = null; // Object NOT garbage collected (Map holds strong reference)

// Solution with WeakMap: Auto-cleanup
const cache = new WeakMap();
let obj = { data: 'huge' };
cache.set(obj, 'metadata');
obj = null; // Object CAN be garbage collected (WeakMap allows it)
```

### WeakMap: Key Patterns

**Available methods**: `.set(obj, value)`, `.get(obj)`, `.has(obj)`, `.delete(obj)`

```javascript
const wm = new WeakMap();
const obj = {};

wm.set(obj, 'value');    // ✅ Works (object key)
wm.set('string', 'v');   // ❌ TypeError (primitives not allowed)
```

**Use Cases**:

```javascript
// 1. Private Data (before #private fields)
const privateData = new WeakMap();
class User {
  constructor(name, ssn) {
    this.name = name;
    privateData.set(this, { ssn }); // Private, auto-cleaned on GC
  }
  getSSN() { return privateData.get(this).ssn; }
}

// 2. Caching (auto-eviction)
const cache = new WeakMap();
function expensive(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = /* compute */;
  cache.set(obj, result);
  return result;
}

// 3. DOM Metadata (no manual cleanup needed)
const domMeta = new WeakMap();
function attachData(el, data) { domMeta.set(el, data); }
button.remove(); // Metadata auto-cleaned when element is GC'd
```

### WeakSet: Key Patterns

**Available methods**: `.add(obj)`, `.has(obj)`, `.delete(obj)`

```javascript
// 1. Track Processed Objects
const processed = new WeakSet();
function process(item) {
  if (processed.has(item)) return; // Already processed
  /* ... */
  processed.add(item);
}

// 2. Prevent Circular Reference Loops
const visited = new WeakSet();
function traverse(node) {
  if (visited.has(node)) return; // Circular check
  visited.add(node);
  node.children?.forEach(traverse);
}
```

### Why No Iteration?

Entries can be garbage collected **at any time** (non-deterministic). Iteration would be inconsistent - an entry might disappear mid-loop.

---

## Modules (ES6 vs. CommonJS)

### The Problem (Pre-Modules)

```javascript
// Global scope pollution + no encapsulation
var apiKey = 'secret'; // Exposed globally!
var userName = 'John'; // Overwritten by other files!

// No dependency management (order-dependent scripts)
<script src="jquery.js"></script>           // Must load first
<script src="jquery-plugin.js"></script>   // Depends on jQuery
```

**Early solution**: IIFE pattern (creates private scope but manual, no tree-shaking)

### ES6 Modules (ESM) — The Standard

```javascript
// Named exports/imports
export const PI = 3.14;
export function add(a, b) { return a + b; }

import { PI, add } from './math.js';
import { add as sum } from './math.js'; // Rename
import * as math from './math.js';       // Namespace

// Default export/import
export default class Calculator {}
import Calculator from './calc.js';

// Dynamic import (runtime, async)
const { feature } = await import('./module.js');
```

### CommonJS (CJS) — Node.js Legacy

```javascript
// Export
const PI = 3.14;
function add(a, b) { return a + b; }
module.exports = { PI, add };

// Import
const { PI, add } = require('./math.js');
```

### Key Differences

| Feature | ES6 Modules (ESM) | CommonJS (CJS) |
|---------|-------------------|----------------|
| **Structure** | Static (compile-time) | Dynamic (runtime) |
| **Loading** | Async | Sync (blocking) |
| **Binding** | Live references | Value copies |
| **Tree-shaking** | ✅ Fully supported | ❌ Limited |
| **Browser** | ✅ Native | ❌ Bundler required |
| **Top-level await** | ✅ Supported | ❌ No |
| **Conditional imports** | `await import()` | `require()` anywhere |

### Live Bindings vs Value Copies

```javascript
// ESM: Live bindings (changes reflected)
export let count = 0;
export function inc() { count++; }

import { count, inc } from './counter.js';
console.log(count); // 0
inc();
console.log(count); // 1 ✅ Updated!

// CJS: Value copies (changes NOT reflected)
let count = 0;
module.exports = { count, inc() { count++; } };

const c = require('./counter');
console.log(c.count); // 0
c.inc();
console.log(c.count); // 0 ❌ Still 0!
```

### When to Use What

| Use ESM When | Use CJS When |
|--------------|---------------|
| Modern frontend projects | Legacy Node.js projects |
| New Node.js (`"type": "module"`) | Sync loading required |
| Tree-shaking matters | Older tooling |
| Universal browser + Node code | |

**Interoperability**:

```javascript
// ESM importing CJS
import pkg from 'cjs-package'; // Default import

// CJS importing ESM (must use dynamic import)
const mod = await import('esm-package');
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

#### ⚠️ Memory Leaks

Closures keep referenced variables alive. Only close over what's needed:

```javascript
function attach() {
  const huge = new Array(1e6);
  const result = process(huge); // Extract what you need
  btn.onclick = () => console.log(result); // Close over result, not huge
}
```

#### Closure Patterns

```javascript
// Private data
function counter() {
  let count = 0;
  return { inc: () => ++count, get: () => count };
}

// Loop closure (interview pitfall)
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // ❌ 3, 3, 3
}
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // ✅ 0, 1, 2
}
```

### Currying

**Transform multi-arg function into sequence of single-arg functions** (enables partial application):

```javascript
// Regular vs Curried
const add = (a, b, c) => a + b + c;
const curried = a => b => c => a + b + c;

curried(1)(2)(3);        // 6
const add5 = curried(5); // Partial application
add5(2)(3);              // 10
```

#### Generic Curry Helper

```javascript
const curry = fn => function curried(...args) {
  return args.length >= fn.length
    ? fn(...args)
    : (...more) => curried(...args, ...more);
};

const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);     // 6
add(1, 2)(3);     // 6
add(1)(2, 3);     // 6
add(1, 2, 3);     // 6
```

#### Practical Use Cases

```javascript
// 1. Config/Logging
const log = level => msg => console.log(`[${level}] ${msg}`);
const error = log('ERROR');
error('Connection failed'); // [ERROR] Connection failed

// 2. Data Pipelines
const map = fn => arr => arr.map(fn);
const filter = pred => arr => arr.filter(pred);
const double = map(x => x * 2);
const evens = filter(x => x % 2 === 0);

double([1, 2, 3]);  // [2, 4, 6]
evens([1, 2, 3]);   // [2]

// 3. Validators
const minLen = min => val => val.length >= min;
const matches = regex => val => regex.test(val);
const isEmail = matches(/^[^\s@]+@[^\s@]+\.[^\s@]+$/);
```

#### Function Composition

```javascript
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

const process = pipe(
  x => x + 1,
  x => x * 2,
  x => `Result: ${x}`
);
process(5); // "Result: 12"
```

#### When NOT to Use

- **Performance-critical code**: Extra function calls add overhead
- **Simple one-time operations**: `add(1, 2, 3)` is clearer than `add(1)(2)(3)`
- **Dynamic argument counts**: `sum(...args)` can't be curried meaningfully

### `this` Context

#### What is `this`?

**`this`** is a special keyword in JavaScript that refers to the context in which a function is executed. Its value is determined at **runtime** based on **how** the function is called, not where it's defined.

**`this`** is determined by **call-site** (where function is called), not where it's written.

#### Binding Rules (Priority)

**1. `new` Binding** (highest): `this` = new object

```javascript
function User(name) { this.name = name; }
const u = new User('John'); // 'this' = u
```

**2. Explicit Binding**: `.call(thisArg, ...args)`, `.apply(thisArg, [args])`, `.bind(thisArg)`

```javascript
fn.call(obj, arg1, arg2); // Call with 'this' = obj
fn.apply(obj, [arg1, arg2]); // Same, array args
const bound = fn.bind(obj); // Permanent binding (even .call can't override)
```

**3. Implicit Binding**: Method call → `this` = object before the dot

```javascript
obj.method(); // 'this' = obj

// ⚠️ Lost 'this' (common bug)
const fn = obj.method;
fn(); // 'this' lost!
setTimeout(obj.method, 100); // 'this' lost!

// Fix: Arrow function or .bind()
setTimeout(() => obj.method(), 100); // ✅
setTimeout(obj.method.bind(obj), 100); // ✅
```

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

**Global**: Everywhere accessible. **Function** (`var`): Function-scoped, ignores blocks. **Block** (`let`/`const`): Block-scoped. **Lexical**: Inner functions access outer variables (scope chain).

```javascript
// Global
var global = 'global';

// Function scope (var)
function fn() {
  var x = 1;
  if (true) { var y = 2; } // y leaks to function scope
  console.log(y); // 2
}

// Block scope (let/const)
{ let x = 1; } // x dies here
for (let i = 0; i < 3; i++) {} // i dies here

// Lexical (scope chain)
function outer() {
  const x = 1;
  function inner() {
    console.log(x); // Accessible via scope chain
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

Function calling itself. Needs **base case** to stop.

```javascript
function factorial(n) {
  if (n <= 1) return 1; // Base case
  return n * factorial(n - 1); // Recursive case
}

// Space: O(n) stack frames, Time: O(n)
// Stack overflow if n > ~10k-15k
```

**vs Iteration**: Recursion = readable for trees/graphs but slower + stack risk. Iteration = faster + O(1) space but verbose.

```javascript
// Use recursion for trees
function traverse(node) {
  console.log(node.name);
  node.children.forEach(traverse);
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

Objects inherit from other objects via **prototype chain**. Property lookup: object → prototype → prototype's prototype → `null`.

```javascript
const animal = { eats: true };
const rabbit = Object.create(animal); // rabbit.__proto__ = animal
rabbit.jumps = true;

console.log(rabbit.jumps); // Own property
console.log(rabbit.eats); // Inherited from animal
console.log(rabbit.fly); // undefined (not in chain)
```

**`[[Prototype]]` vs `__proto__` vs `prototype`**:

- `[[Prototype]]`: Internal slot (spec)
- `__proto__`: Accessor (legacy, avoid)
- `prototype`: Property on constructor functions

```javascript
function Person(name) { this.name = name; }
Person.prototype.greet = function() { console.log(this.name); };
const p = new Person('John');

p.__proto__ === Person.prototype; // true
Object.getPrototypeOf(p) === Person.prototype; // true (modern)
```

#### Constructor Functions

```javascript
function Person(name, age) {
  this.name = name; // Instance property
  this.age = age;
}

Person.prototype.greet = function() { // Shared method
  console.log(`Hello, ${this.name}`);
};

const john = new Person('John', 30);
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

**Singleton** ensures a class has **only one instance** with a **global access point**.

**Use cases**: Database connections, configuration, logging, caching, event bus.

#### Implementation (Modern ES6)

```javascript
class Database {
  static #instance = null;
  #connection = null;

  constructor() {
    if (Database.#instance) return Database.#instance;
    this.#connection = this.#createConnection();
    Database.#instance = this;
  }

  #createConnection() {
    console.log('Connecting...');
    return { status: 'connected' };
  }

  query(sql) { return this.#connection; }

  static getInstance() {
    if (!Database.#instance) Database.#instance = new Database();
    return Database.#instance;
  }
}

const db1 = new Database();       // "Connecting..."
const db2 = new Database();       // (nothing - returns same instance)
console.log(db1 === db2);         // true
```

#### ES6 Module Pattern (Simplest)

```javascript
// database.js - ES6 modules are singletons by design
class Database { /* ... */ }
export default new Database();

// Or lazy:
let instance = null;
export function getDatabase() {
  if (!instance) instance = new Database();
  return instance;
}
```

#### Advantages vs Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|------------------|
| Single instance (resource efficient) | Global state (hard to track changes) |
| Lazy initialization | Testing difficulties (shared state) |
| Global access | Tight coupling |
| | Violates Single Responsibility |

#### When to Use / Avoid

| ✅ Use For | ❌ Avoid When |
|-----------|---------------|
| Configuration managers | Need multiple instances |
| Logging services | Testing is critical |
| Caching | State varies by context |
| Connection pools | Concurrency matters |

#### Better Alternative: Dependency Injection

```javascript
// Singleton (tightly coupled, hard to test)
class UserService {
  getUser(id) {
    const db = Database.getInstance(); // ❌
    return db.query(`SELECT...`);
  }
}

// DI (loosely coupled, testable)
class UserService {
  constructor(database) { this.database = database; }
  getUser(id) { return this.database.query(`SELECT...`); }
}

// Testing
const mockDb = { query: jest.fn() };
const service = new UserService(mockDb); // ✅ Easy to test
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

// All have O(n) time complexity
// Map is slightly faster for large datasets
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

---

## Key Takeaways & Study Tips

### Core Concepts Summary

**Asynchronous Programming**:

- JavaScript is single-threaded but achieves concurrency through the Event Loop
- Web APIs/Node.js APIs handle async operations outside the JS thread
- Understanding the Event Loop is crucial for debugging timing issues
- Microtasks (Promises) always execute before macrotasks (setTimeout)

**Modern JavaScript Features**:

- **Destructuring** simplifies data extraction from arrays and objects
- **Spread/Rest** operators enable flexible function parameters and array/object manipulation
- **Arrow functions** provide lexical `this` binding (critical for callbacks)
- **Template literals** make string interpolation and multi-line strings easier
- **Modules (ESM)** are the standard for modern JavaScript (tree-shaking, static analysis)

**Functions & Scope**:

- **Closures** are fundamental to JavaScript - inner functions remember outer scope
- **Currying** enables partial application and function specialization
- **`this` context** is determined by call-site, not definition site (except arrow functions)
- **Block scope** (`let`/`const`) prevents common bugs associated with `var`
- **Recursion** is powerful but watch for stack overflow

**OOP & Patterns**:

- JavaScript uses **prototypal inheritance**, not classical inheritance
- **ES6 classes** are syntactic sugar over prototypes
- **Singleton pattern** ensures single instance (use with caution - global state issues)
- Always consider memory implications of closures and prototypes

**Data Structures**:

- **Higher-order functions** (`map`, `filter`, `reduce`) enable declarative programming
- **`reduce()`** is the most powerful - can implement map, filter, and more
- **Immutability** is crucial for modern frameworks (React, Redux)
- **Map/Set** are better than Objects for dynamic key-value pairs

### Interview Preparation Checklist

**Must Know**:

- [ ] Explain the Event Loop and demonstrate with examples
- [ ] Explain Promises vs callbacks and handle errors properly
- [ ] Implement common array methods from scratch (map, filter, reduce)
- [ ] Explain closures and provide practical examples
- [ ] Demonstrate understanding of `this` binding in different contexts
- [ ] Explain prototypal inheritance and how ES6 classes work under the hood
- [ ] Solve the "group by" problem efficiently using `reduce()`

**Should Know**:

- [ ] Difference between microtasks and macrotasks
- [ ] When to use Map/Set vs Objects
- [ ] How WeakMap/WeakSet prevent memory leaks
- [ ] Tail recursion and its limitations in JavaScript
- [ ] Module systems (ESM vs CommonJS) and their trade-offs
- [ ] Singleton pattern and when to avoid it

**Advanced (Senior Level)**:

- [ ] Performance implications of prototype chain lookups
- [ ] Memory management in closures
- [ ] Tree-shaking and static analysis in bundlers
- [ ] Async iteration and generators
- [ ] Concurrency patterns (Promise.all, Promise.race, etc.)
- [ ] Design patterns in JavaScript (Factory, Observer, Decorator)

### Common Interview Pitfalls

**1. Event Loop Misconceptions**:

```javascript
// ❌ Wrong: Thinking this runs after 1 second exactly
setTimeout(() => console.log('Done'), 1000);
heavyComputation(); // Blocks for 5 seconds

// Reality: Timer queued after 1s, but must wait for call stack to clear
// Actual execution: ~6 seconds
```

**2. `this` Binding Mistakes**:

```javascript
// ❌ Wrong: Losing `this` in callbacks
class Counter {
  count = 0;
  increment() { this.count++; }
  
  start() {
    setInterval(this.increment, 1000); // `this` is undefined/window!
  }
}

// ✅ Correct: Arrow function or .bind()
start() {
  setInterval(() => this.increment(), 1000);
  // or
  setInterval(this.increment.bind(this), 1000);
}
```

**3. Closure Memory Leaks**:

```javascript
// ❌ Wrong: Keeping unnecessary data in closure
function attachHandler() {
  const hugeData = new Array(1000000).fill('data');
  
  button.addEventListener('click', function() {
    console.log('clicked'); // Closure keeps hugeData alive!
  });
}

// ✅ Correct: Only close over what you need
function attachHandler() {
  const hugeData = new Array(1000000).fill('data');
  const summary = processData(hugeData); // Process and discard
  
  button.addEventListener('click', function() {
    console.log('clicked', summary); // Only summary kept in memory
  });
}
```

**4. Array Method Confusion**:

```javascript
// ❌ Wrong: Using map when you need filter
const evens = numbers.map(n => n % 2 === 0); // [false, true, false, true]

// ✅ Correct: Use filter for conditional inclusion
const evens = numbers.filter(n => n % 2 === 0); // [2, 4, 6]
```

### Practice Problems

**1. Implement debounce**:

```javascript
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

**2. Implement deep clone**:

```javascript
function deepClone(obj, seen = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (seen.has(obj)) return seen.get(obj);
  
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (Array.isArray(obj)) {
    const cloned = [];
    seen.set(obj, cloned);
    obj.forEach((item, i) => cloned[i] = deepClone(item, seen));
    return cloned;
  }
  
  const cloned = {};
  seen.set(obj, cloned);
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key], seen);
    }
  }
  return cloned;
}
```

**3. Implement Promise.all**:

```javascript
function promiseAll(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;
    
    if (promises.length === 0) {
      resolve(results);
      return;
    }
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(result => {
          results[index] = result;
          completed++;
          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}
```

### Additional Resources

**Official Documentation**:

- [MDN Web Docs - JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- [ECMAScript Specification](https://tc39.es/ecma262/)
- [Node.js Documentation](https://nodejs.org/docs/)

**Recommended Reading**:

- "You Don't Know JS" series by Kyle Simpson
- "JavaScript: The Good Parts" by Douglas Crockford
- "Eloquent JavaScript" by Marijn Haverbeke
- "JavaScript Patterns" by Stoyan Stefanov

**Online Resources**:

- [JavaScript.info](https://javascript.info/) - Comprehensive modern JavaScript tutorial
- [2ality](https://2ality.com/) - Dr. Axel Rauschmayer's blog on JavaScript
- [V8 Blog](https://v8.dev/blog) - JavaScript engine insights

**Practice Platforms**:

- [LeetCode](https://leetcode.com/) - Algorithm problems
- [HackerRank](https://www.hackerrank.com/) - JavaScript challenges
- [Exercism](https://exercism.org/) - Mentored code practice
