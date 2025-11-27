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

**Core Concept**: JavaScript is single-threaded but handles asynchronous operations through the event loop, allowing non-blocking I/O operations.

**Key Mechanisms**:

- **Callbacks**: Functions passed as arguments, executed after async operation completes
- **Promises**: Objects representing eventual completion/failure of async operation
- **Async/Await**: Syntactic sugar over Promises for cleaner async code

**Critical for Senior Level**:

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

**Event Loop Architecture**:

1. **Call Stack**: Executes synchronous code
2. **Callback Queue (Macrotasks)**: `setTimeout`, `setInterval`, I/O operations
3. **Microtask Queue**: Promises, `queueMicrotask()`, `MutationObserver`
4. **Render Queue**: Browser repaints/reflows

**Execution Order**:
Synchronous code → Microtasks → Macrotask → Microtasks → Render → Repeat

**Microtasks** (higher priority):

- `Promise.then/catch/finally`
- `queueMicrotask()`
- `async/await` continuations

**Macrotasks** (lower priority):

- `setTimeout/setInterval`
- `setImmediate` (Node.js)
- I/O operations
- UI rendering

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

**Promise States**:

- **Pending**: Initial state
- **Fulfilled**: Operation completed successfully
- **Rejected**: Operation failed

**Promise Creation**:

```javascript
const promise = new Promise((resolve, reject) => {
  // async operation
  if (success) resolve(value);
  else reject(error);
});
```

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

**Callback Hell** (Pyramid of Doom):

```javascript
// ❌ Hard to read and maintain
getUserData(userId, (user) => {
  getOrders(user.id, (orders) => {
    getOrderDetails(orders[0].id, (details) => {
      processPayment(details.total, (payment) => {
        sendConfirmation(payment.id, (result) => {
          console.log('Done');
        });
      });
    });
  });
});
```

**Issues**:

- Deep nesting makes code unreadable
- Error handling must be repeated at each level
- Difficult to debug and test
- Hard to add parallel operations

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

```javascript
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];

// Object destructuring with rename and defaults
const { name: userName, age = 18 } = user;

// Nested destructuring
const { address: { city, country } } = user;

// Function parameters
function displayUser({ name, age = 18, role = 'user' }) {
  console.log(name, age, role);
}
```

### Spread/Rest Operators

```javascript
// Spread: Expands iterables
const arr1 = [1, 2];
const arr2 = [...arr1, 3, 4]; // [1, 2, 3, 4]

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 }; // { a: 1, b: 2, c: 3 }

// Rest: Collects multiple elements
function sum(...numbers) {
  return numbers.reduce((acc, n) => acc + n, 0);
}
```

### Arrow Functions

```javascript
// Lexical 'this' binding (doesn't create own 'this')
class Timer {
  constructor() {
    this.seconds = 0;
    setInterval(() => this.seconds++, 1000); // ✅ Works
    // setInterval(function() { this.seconds++; }, 1000); // ❌ 'this' is undefined
  }
}

// Implicit return
const double = x => x * 2;
const getUser = id => ({ id, name: 'John' }); // Must wrap object in ()
```

### Template Literals

```javascript
const name = 'John';
const age = 30;

// Multi-line strings and interpolation
const message = `
  Hello ${name},
  You are ${age} years old.
  Next year you'll be ${age + 1}.
`;

// Tagged templates (advanced)
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) => 
    `${acc}${str}<mark>${values[i] || ''}</mark>`, ''
  );
}
const html = highlight`Name: ${name}, Age: ${age}`;
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

**Critical Difference from Map/Set**: Keys in WeakMap and values in WeakSet are **weakly referenced**, allowing garbage collection when no other references exist.

### WeakMap

**Characteristics**:

- Keys **must** be objects (not primitives)
- No iteration methods (`.keys()`, `.values()`, `.entries()`, `.forEach()`)
- No `.size` property
- Keys can be garbage collected

**Use Cases**:

**1. Private Data Storage** (before private class fields):

```javascript
const privateData = new WeakMap();

class User {
  constructor(name, ssn) {
    this.name = name;
    privateData.set(this, { ssn }); // Private
  }
  
  getSSN() {
    return privateData.get(this).ssn;
  }
}

const user = new User('John', '123-45-6789');
// No way to access SSN except through getSSN()
```

**2. Caching/Memoization**:

```javascript
const cache = new WeakMap();

function processData(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }
  
  const result = expensiveOperation(obj);
  cache.set(obj, result);
  return result;
}
// Cache is automatically cleared when obj is garbage collected
```

**3. DOM Node Metadata** (prevents memory leaks):

```javascript
const metadata = new WeakMap();

function addMetadata(element, data) {
  metadata.set(element, data);
}
// When element is removed from DOM and no other references exist,
// metadata is automatically garbage collected
```

### WeakSet

**Characteristics**:

- Values **must** be objects
- No iteration or size checking
- Automatic garbage collection

**Use Cases**:

**1. Tracking Object Instances**:

```javascript
const processedItems = new WeakSet();

function processItem(item) {
  if (processedItems.has(item)) {
    return; // Already processed
  }
  
  // Process item
  processedItems.add(item);
}
```

**2. Marking Objects**:

```javascript
const disabledElements = new WeakSet();

function disableElement(element) {
  disabledElements.add(element);
  element.setAttribute('disabled', true);
}

function isDisabled(element) {
  return disabledElements.has(element);
}
```

**Interview Key Points**:

- WeakMap/WeakSet prevent memory leaks in scenarios involving DOM elements or large objects
- Cannot be iterated because entries can be garbage collected at any time
- Perfect for associating metadata with objects without preventing their cleanup

---

## Modules (ES6 vs. CommonJS)

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

**Definition**: A function that retains access to its outer scope's variables even after the outer function has returned.

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

**Definition**: Transform a function with multiple arguments into a sequence of functions each taking a single argument.

```javascript
// Regular function
function add(a, b, c) {
  return a + b + c;
}

// Curried version
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

curriedAdd(1)(2)(3); // 6

// ES6 arrow functions
const add = a => b => c => a + b + c;

// Practical use: Partial application
const add5 = add(5);
const add5and10 = add5(10);
console.log(add5and10(3)); // 18
```

**Real-World Example**:

```javascript
// Generic curry function
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return (...moreArgs) => curried(...args, ...moreArgs);
  };
}

const sum = (a, b, c) => a + b + c;
const curriedSum = curry(sum);

curriedSum(1)(2)(3); // 6
curriedSum(1, 2)(3); // 6
curriedSum(1)(2, 3); // 6
```

### `this` Context

**Rules** (in order of precedence):

**1. `new` binding**: `this` = newly created object

```javascript
function User(name) {
  this.name = name;
}
const user = new User('John'); // this = user object
```

**2. Explicit binding**: `.call()`, `.apply()`, `.bind()`

```javascript
function greet() {
  console.log(`Hello, ${this.name}`);
}
const user = { name: 'John' };
greet.call(user); // this = user
```

**3. Implicit binding**: Method invocation

```javascript
const obj = {
  name: 'John',
  greet() { console.log(this.name); }
};
obj.greet(); // this = obj
```

**4. Default binding**: Global object (or undefined in strict mode)

```javascript
function show() {
  console.log(this); // window (browser) or global (Node.js)
}
show();
```

**Arrow Functions**: No own `this`, inherits from enclosing scope

```javascript
const obj = {
  name: 'John',
  regularFunc: function() {
    console.log(this.name); // 'John'
  },
  arrowFunc: () => {
    console.log(this.name); // undefined (or global scope value)
  }
};
```

### Scopes

**Types**:

**1. Global Scope**: Accessible everywhere

```javascript
const global = 'I am global';
```

**2. Function Scope**: `var` is function-scoped

```javascript
function example() {
  var x = 10; // Only accessible within function
  if (true) {
    var y = 20; // Still function-scoped (hoisted)
  }
  console.log(y); // 20 (accessible)
}
```

**3. Block Scope**: `let`/`const` are block-scoped

```javascript
{
  let x = 10;
  const y = 20;
}
console.log(x); // ReferenceError
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

**Definition**: Function calls itself until base condition is met.

**Deep Clone Example**:

```javascript
function deepClone(obj) {
  // Base cases
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);
  if (obj instanceof Array) {
    return obj.map(item => deepClone(item));
  }
  
  // Recursive case for objects
  const cloned = {};
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key]);
    }
  }
  return cloned;
}
```

**Tail Recursion** (optimization):

```javascript
// Not tail-recursive (operation after recursive call)
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1); // Multiplication after call
}

// Tail-recursive (recursive call is last operation)
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc); // No operation after call
}
```

---

## OOP & Prototypes

### Prototypal Inheritance

**Core Concept**: JavaScript uses prototypal inheritance, not classical inheritance. Every object has a hidden `[[Prototype]]` property that links to another object.

**Prototype Chain**:

```javascript
const animal = {
  eats: true,
  walk() {
    console.log('Animal walks');
  }
};

const rabbit = {
  jumps: true,
  __proto__: animal // Sets prototype (use Object.create() in production)
};

console.log(rabbit.eats); // true (inherited from animal)
console.log(rabbit.jumps); // true (own property)

// Prototype chain: rabbit → animal → Object.prototype → null
```

**Constructor Functions** (ES5 Pattern):

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

// Methods on prototype (shared across instances)
Person.prototype.greet = function() {
  console.log(`Hello, I'm ${this.name}`);
};

const john = new Person('John', 30);
john.greet(); // "Hello, I'm John"

console.log(john.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true
```

### ES6 Classes

**Syntax Sugar** over prototypal inheritance:

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  
  // Methods automatically added to prototype
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
  
  // Static methods (on class itself, not instances)
  static species() {
    return 'Homo sapiens';
  }
  
  // Getters/Setters
  get info() {
    return `${this.name} (${this.age})`;
  }
  
  set info(value) {
    [this.name, this.age] = value.split(',');
  }
}

const john = new Person('John', 30);
console.log(Person.species()); // "Homo sapiens"
```

### Inheritance

```javascript
class Employee extends Person {
  constructor(name, age, salary) {
    super(name, age); // Call parent constructor
    this.salary = salary;
  }
  
  // Override parent method
  greet() {
    super.greet(); // Call parent method
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

**Definition**: Ensure a class has only one instance and provide global access to it.

#### Implementation 1: ES6 Class with Static Instance

```javascript
class Database {
  static #instance = null;
  #connection = null;
  
  constructor() {
    if (Database.#instance) {
      return Database.#instance;
    }
    
    this.#connection = this.#createConnection();
    Database.#instance = this;
  }
  
  #createConnection() {
    return { status: 'connected', host: 'localhost' };
  }
  
  query(sql) {
    console.log(`Executing: ${sql}`);
    return this.#connection;
  }
  
  static getInstance() {
    if (!Database.#instance) {
      Database.#instance = new Database();
    }
    return Database.#instance;
  }
}

// Usage
const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2); // true (same instance)
```

#### Implementation 2: Closure Pattern

```javascript
const DatabaseSingleton = (function() {
  let instance;
  
  function createInstance() {
    const connection = { status: 'connected' };
    
    return {
      query(sql) {
        console.log(`Executing: ${sql}`);
        return connection;
      },
      getConnection() {
        return connection;
      }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const db1 = DatabaseSingleton.getInstance();
const db2 = DatabaseSingleton.getInstance();
console.log(db1 === db2); // true
```

#### Implementation 3: Module Pattern (Modern)

```javascript
// database.js
class Database {
  #connection = null;
  
  constructor() {
    this.#connection = this.#createConnection();
  }
  
  #createConnection() {
    return { status: 'connected', host: 'localhost' };
  }
  
  query(sql) {
    console.log(`Executing: ${sql}`);
    return this.#connection;
  }
}

// Export single instance
export default new Database();

// Usage in other files
// import db from './database.js';
// db.query('SELECT * FROM users');
```

**When to Use Singleton**:

- Database connections
- Configuration managers
- Logging services
- Cache managers
- Thread pools

**Drawbacks**:

- Global state (can make testing harder)
- Tight coupling
- Violates Single Responsibility Principle
- Consider dependency injection instead for better testability

---

## Data Structures

### Array Methods

#### `map()` - Transform Elements

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
```

#### `filter()` - Select Elements

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

// Remove duplicates
const nums = [1, 2, 2, 3, 3, 4];
const unique = nums.filter((n, i, arr) => arr.indexOf(n) === i);
// [1, 2, 3, 4]
```

#### `reduce()` - Accumulate/Transform

**Most powerful array method**:

```javascript
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((acc, n) => acc + n, 0);
// 15

// Object from array
const users = ['John', 'Jane', 'Bob'];
const userObj = users.reduce((acc, name, i) => {
  acc[i] = name;
  return acc;
}, {});
// { 0: 'John', 1: 'Jane', 2: 'Bob' }

// Group by property
const products = [
  { name: 'laptop', category: 'electronics', price: 1000 },
  { name: 'phone', category: 'electronics', price: 500 },
  { name: 'shirt', category: 'clothing', price: 50 }
];

const grouped = products.reduce((acc, product) => {
  const { category } = product;
  if (!acc[category]) acc[category] = [];
  acc[category].push(product);
  return acc;
}, {});
// {
//   electronics: [{ laptop }, { phone }],
//   clothing: [{ shirt }]
// }

// Flatten array
const nested = [[1, 2], [3, 4], [5]];
const flattened = nested.reduce((acc, arr) => acc.concat(arr), []);
// [1, 2, 3, 4, 5]

// Count occurrences
const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});
// { apple: 3, banana: 2, orange: 1 }
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

**Problem**: Given an array of books with publication years, group them by decade.

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

**Step-by-Step Explanation**:

1. **`reduce()`** iterates through each book
2. **Calculate decade**: `Math.floor(1949 / 10) * 10 = 1940`
3. **Create key**: `"1940s"`
4. **Initialize array** if decade doesn't exist in accumulator
5. **Push book** into corresponding decade array
6. **Return accumulator** for next iteration
7. **Final result**: Object with decades as keys, arrays of books as values

**Key Takeaways**:

- `reduce()` is perfect for transforming arrays into objects
- Always initialize accumulator with correct type (`{}` for objects)
- Check if key exists before accessing/modifying
- Consider using `Map` for better performance with dynamic keys
