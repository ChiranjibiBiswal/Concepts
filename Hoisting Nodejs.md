# JavaScript Hoisting

## 📌 Overview

Hoisting refers to JavaScript's behavior of moving variable, function, and class declarations to the top of their scope during the compilation phase. This affects how and when variables and functions can be accessed in a program.

---

## 🚀 Hoisting Behavior and Internal Execution

### 🔹 1. Variable Hoisting with `var`

#### ✅ Input:

```javascript
console.log(a); // undefined
var a = 5;
```

#### 🔍 Output:

```
undefined
```

#### 🔎 Internal Execution:

```javascript
var a; // Declaration is hoisted to the top
console.log(a); // undefined
a = 5; // Assignment remains in place
```

---

### 🔹 2. Variable Hoisting with `let` and `const`

#### ✅ Input:

```javascript
console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 10;
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'b' before initialization
```

#### 🔎 Internal Execution:

```javascript
// 'b' is hoisted but remains in the Temporal Dead Zone (TDZ)
console.log(b); // ReferenceError
let b = 10; // Initialization happens here
```

---

### 🔹 3. Function Declaration Hoisting

#### ✅ Input:

```javascript
greet(); // "Hello, world!"
function greet() {
  console.log("Hello, world!");
}
```

#### 🔍 Output:

```
Hello, world!
```

#### 🔎 Internal Execution:

```javascript
// Function declarations are fully hoisted
function greet() {
  console.log("Hello, world!");
}
greet(); // Can be called before declaration
```

---

### 🔹 4. Function Expression Hoisting

#### ✅ Input:

```javascript
hello(); // TypeError: hello is not a function
var hello = function () {
  console.log("Hi!");
};
```

#### 🔍 Output:

```
TypeError: hello is not a function
```

#### 🔎 Internal Execution:

```javascript
var hello; // Declaration is hoisted
hello(); // undefined() -> TypeError because hello is still undefined
hello = function () {
  console.log("Hi!");
};
```

---

### 🔹 5. Hoisting with `let` and `const` in Functions

#### ✅ Input:

```javascript
function test() {
  console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = 50;
}
test();
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'x' before initialization
```

#### 🔎 Internal Execution:

```javascript
function test() {
  // 'x' is in the Temporal Dead Zone
  console.log(x); // ReferenceError
  let x = 50; // Initialization occurs here
}
```

---

### 🔹 6. Hoisting with Classes

#### ✅ Input:

```javascript
const obj = new MyClass(); // ReferenceError
class MyClass {
  constructor() {
    this.name = "Example";
  }
}
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'MyClass' before initialization
```

#### 🔎 Internal Execution:

```javascript
// 'MyClass' is hoisted but in Temporal Dead Zone
const obj = new MyClass(); // ReferenceError
class MyClass {
  constructor() {
    this.name = "Example";
  }
}
```

---

### 🔹 7. Re-declaring Variables with `var`

#### ✅ Input:

```javascript
var a = 10;
var a = 20; // No error
console.log(a);
```

#### 🔍 Output:

```
20
```

#### 🔎 Internal Execution:

```javascript
var a = 10;
var a; // Re-declaration does nothing
a = 20; // Overwrites previous value
console.log(a); // 20
```

---

### 🔹 8. Accessing Variables Declared Later in Loops

#### ✅ Input:

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100);
}
```

#### 🔍 Output:

```
3
3
3
```

#### 🔎 Internal Execution:

```javascript
var i;
for (i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // Refers to the same i, which is now 3
  }, 100);
}
```

---

### 🔹 9. Using Hoisted Functions with Parameters

#### ✅ Input:

```javascript
test(10);
function test(num) {
  console.log(num);
}
```

#### 🔍 Output:

```
10
```

#### 🔎 Internal Execution:

```javascript
function test(num) {
  console.log(num);
}
test(10);
```

---

### 🔹 10. Hoisting in Nested Functions

#### ✅ Input:

```javascript
function outer() {
  console.log(a); // undefined
  var a = 5;
  function inner() {
    console.log(b); // undefined
    var b = 10;
  }
  inner();
}
outer();
```

#### 🔍 Output:

```
undefined
undefined
```

#### 🔎 Internal Execution:

```javascript
function outer() {
  var a; // Hoisted declaration
  console.log(a); // undefined
  a = 5; // Assignment
  function inner() {
    var b; // Hoisted declaration
    console.log(b); // undefined
    b = 10; // Assignment
  }
  inner();
}
outer();
```

---

## 📚 Reference

[GeeksforGeeks - JavaScript Hoisting](https://www.geeksforgeeks.org/javascript-hoisting/)

[Medium - Event Loop in NodeJs](https://medium.com/@mmoshikoo/event-loop-in-nodejs-visualized-235867255e81)

[Medium - NodeJs architecture](https://medium.com/@ibrahimlanre1890/node-js-architecture-understanding-node-js-architecture-5fb32879b994)
