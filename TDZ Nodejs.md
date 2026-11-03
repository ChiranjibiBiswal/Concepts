# Temporal Dead Zone (TDZ) in JavaScript

## 📌 Overview

The **Temporal Dead Zone (TDZ)** is the period during which a `let`, `const`, or `class` binding exists but is **not yet initialized**. It starts when the enclosing scope is entered (the binding is created, uninitialized) and ends the moment the declaration's initializer runs. Accessing the variable at any point in between throws a `ReferenceError`.

TDZ is **temporal, not spatial** — the variable already *exists* (it was hoisted). It simply cannot be touched yet. That is why the error says `Cannot access 'a' before initialization` and not `a is not defined`.

> All examples below verified on **Node.js v26.13.0** (V8). Error wording can differ slightly on other engines.

---

## Key Points

- Variables declared with `let`, `const`, and `class` are **hoisted** but remain **uninitialized**.
- Accessing them before their declaration causes a **ReferenceError**.
- The TDZ **starts at the beginning of the enclosing scope** and ends when the variable is initialized.
- `var` does **not** have a TDZ because it is initialized to `undefined` at scope entry, during declaration instantiation.
- `typeof` does **not** protect you — it only suppresses the error for variables that were *never declared*.
- The TDZ applies to the **whole scope**, even if the declaration line is never reached.

---

## Examples

### 1. TDZ with `let`

#### ✅ Input:

```javascript
console.log(a); // ❌ ReferenceError: Cannot access 'a' before initialization
let a = 10;     // ✅ TDZ ends here
console.log(a); // ✅ 10 (never reached — the script already threw)
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'a' before initialization
```

#### 🔎 Internal Execution:

```javascript
// ── TDZ for 'a' starts at scope entry ──
console.log(a); // ReferenceError — binding exists but is uninitialized
let a = 10;     // ── TDZ ends: binding is initialized ──
console.log(a); // 10
```

---

### 2. TDZ with `const`

#### ✅ Input:

```javascript
console.log(b); // ❌ ReferenceError: Cannot access 'b' before initialization
const b = 20;   // ✅ TDZ ends here
console.log(b); // ✅ 20
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'b' before initialization
```

#### 🔎 Internal Execution:

```javascript
// ── TDZ for 'b' starts at scope entry ──
console.log(b); // ReferenceError
const b = 20;   // ── TDZ ends ──
console.log(b); // 20
```

---

### 3. TDZ in Block Scope

#### ✅ Input:

```javascript
{
  console.log(x); // ❌ ReferenceError: Cannot access 'x' before initialization
  let x = 5;      // ✅ TDZ ends here
  console.log(x); // ✅ 5
}
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'x' before initialization
```

#### 🔎 Internal Execution:

```javascript
// ── TDZ for 'x' starts the instant the block is entered ──
{
  console.log(x); // ReferenceError — execution of the block aborts here
  let x = 5;
  console.log(x); // never reached
}
```

> Want to see both outcomes in one run? Wrap the block in `try/catch` — the `catch` runs, then the block can be re-entered fresh.

---

### 4. `var` Does NOT Have a TDZ

#### ✅ Input:

```javascript
console.log(y); // ✅ undefined (no TDZ)
var y = 30;
console.log(y); // ✅ 30
```

#### 🔍 Output:

```
undefined
30
```

#### 🔎 Internal Execution:

```javascript
var y;          // initialized to undefined at scope entry
console.log(y); // undefined
y = 30;         // assignment stays in place
console.log(y); // 30
```

---

### 5. TDZ with Function Parameters

#### ✅ Input:

```javascript
function test(x = y, y = 10) {
  console.log(x, y);
}

test(); // ❌ ReferenceError: Cannot access 'y' before initialization
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'y' before initialization
```

#### 🔎 Internal Execution:

```javascript
function test(x = y, y = 10) {
  // Parameters are initialized left to right.
  // While evaluating the default for 'x', 'y' already exists but is still in its TDZ.
}
```

> Works the other way too: `function test(x = 1, y = x)` is fine, because `x` is already initialized when `y`'s default is evaluated.

---

### 6. Multiple Declarations in One Statement

#### ✅ Input:

```javascript
let p = q, q = 1; // ❌ ReferenceError: Cannot access 'q' before initialization
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'q' before initialization
```

#### 🔎 Internal Execution:

```javascript
// Both 'p' and 'q' enter the TDZ at scope entry.
// 'p' is initialized by evaluating 'q' — but 'q' is still in its TDZ.
let p = q, q = 1;
```

---

### 7. The Shadowing Trap (most common real-world case)

#### ✅ Input:

```javascript
let x = 1;

{
  console.log(x); // ❌ ReferenceError: Cannot access 'x' before initialization
  let x = 2;      // inner 'x' shadows the outer one for the WHOLE block
}
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'x' before initialization
```

#### 🔎 Internal Execution:

```javascript
let x = 1;

{
  // A NEW 'x' is created here and starts its TDZ immediately.
  // It shadows the outer 'x' across the entire block, not just from line 5 onward.
  console.log(x); // ReferenceError — resolves to the inner, uninitialized 'x'
  let x = 2;
}
```

> This is the bug people actually hit in real code — the outer variable is invisible even *before* the inner declaration appears.

---

### 8. `typeof` Does NOT Protect You in the TDZ

#### ✅ Input:

```javascript
console.log(typeof neverDeclared); // ✅ "undefined" — no error
console.log(typeof inTdz);         // ❌ ReferenceError
let inTdz = 1;
```

#### 🔍 Output:

```
undefined
ReferenceError: Cannot access 'inTdz' before initialization
```

#### 🔎 Internal Execution:

```javascript
// typeof is safe only for names that were never declared at all.
console.log(typeof neverDeclared); // "undefined" (unresolvable reference)

// A hoisted-but-uninitialized binding is a real binding — so reading it throws.
console.log(typeof inTdz);         // ReferenceError
let inTdz = 1;
```

---

### 9. `switch` Cases Share One Block Scope

#### ✅ Input:

```javascript
switch (1) {
  case 1:
    console.log(s); // ❌ ReferenceError: Cannot access 's' before initialization
    break;
  case 2:
    let s = 1;
}
```

#### 🔍 Output:

```
ReferenceError: Cannot access 's' before initialization
```

#### 🔎 Internal Execution:

```javascript
// The whole switch body is ONE block. The `let s` in case 2 creates a single
// binding whose TDZ covers every case, including case 1 where it is read.
```

> Fix: wrap each case body in `{ }` braces, or declare before the `switch`.

---

### 10. Classes Live in the TDZ Too

#### ✅ Input:

```javascript
new MyClass(); // ❌ ReferenceError: Cannot access 'MyClass' before initialization
class MyClass {}
```

#### 🔍 Output:

```
ReferenceError: Cannot access 'MyClass' before initialization
```

#### 🔎 Internal Execution:

```javascript
// Class declarations are hoisted like let/const — the binding exists but is
// uninitialized until evaluation reaches the class body.
new MyClass(); // ReferenceError
class MyClass {}
```

> Same rule for `class Child extends Parent` when `Parent` is declared later.

---

### 11. `let` in Loops — Where the TDZ Helps You

#### ✅ Input:

```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 10);
}
```

#### 🔍 Output:

```
0
1
2
```

#### 🔎 Internal Execution:

```javascript
// Each iteration gets a FRESH 'i' binding, initialized before the body runs.
// Closures therefore capture three different variables.
// With `var` instead, all three callbacks share one 'i' and print 3 3 3.
```

> This is the flip side of example 8 in [Hoisting in JavaScript](./Hoisting%20Nodejs.md).

---

### 12. `const` Without an Initializer Is NOT a TDZ Error

#### ✅ Input:

```javascript
const c; // ❌ SyntaxError: Missing initializer in const declaration
```

#### 🔍 Output:

```
SyntaxError: Missing initializer in const declaration
```

#### 🔎 Internal Execution:

```javascript
// This never runs at all — it is rejected while the code is being parsed.
// A SyntaxError is thrown before any execution, so the TDZ never comes into play.
```

---

## Why TDZ Matters

- ✅ **Prevents access to uninitialized variables** — no silent `undefined`.
- ✅ **Turns a whole class of bugs into loud, immediate failures** instead of `NaN` or `undefined` leaking downstream.
- ✅ **Avoids unpredictable behavior from `var` hoisting** and accidental reuse across blocks.
- ✅ **Makes block scoping real** — the shadowing trap (example 7) is the compiler pointing out a genuine mistake.

---

## **Pro Tip:** Always declare and initialize `let` and `const` variables at the top of their scope, and never reuse a name across nested blocks without thinking about it.

---

## 📚 Reference

- [MDN — `let` and the Temporal Dead Zone](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#temporal_dead_zone_tdz)
- [GeeksforGeeks — What is the Temporal Dead Zone in ES6?](https://www.geeksforgeeks.org/what-is-the-temporal-dead-zone-in-es6/)
- Companion note: [Hoisting in JavaScript](./Hoisting%20Nodejs.md)
