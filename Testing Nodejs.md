# Node.js Testing with Jest, Mocha, Chai, and Supertest

This README File demonstrates testing in Node.js using **Jest, Mocha, Chai, and Supertest**. Below, you will find test cases for a simple sum function and an API endpoint.

---

## 📌 **Project Setup**

### 1️⃣ Install Dependencies

Ensure you have Node.js installed, then install the required dependencies:

```sh
npm install --save-dev jest mocha chai supertest
```

### 2️⃣ Define Test Scripts in `package.json`

Modify your `package.json` to include:

```json
"scripts": {
  "test": "jest",
  "mocha-test": "mocha ./sum.test.js"
}
```

---

## ✅ **Testing a Simple Sum Function**

### 📌 `sum.js`

```js
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

### 📌 `sum.test.js` (Jest Test)

```js
const sum = require("./sum");

test("adds 1 + 2 to equal 3", () => {
  expect(sum(1, 2)).toBe(3);
});
```

#### ✅ **Expected Output (Jest)**

```sh
npm test

> example@1.0.0 test
> jest

 PASS  ./sum.test.js
  √ adds 1 + 2 to equal 3 (2 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
```

### 📌 `sum.test.js` (Mocha & Chai Test)

```js
const sum = require("./sum");
const { expect } = require("chai");

describe("Sum Function", () => {
  it("should return 5 when adding 2 + 3", () => {
    expect(sum(2, 3)).to.equal(5);
  });

  it("should return -1 when adding 2 + (-3)", () => {
    expect(sum(2, -3)).to.equal(-1);
  });
});
```

#### ✅ **Expected Output (Mocha)**

```sh
npm run mocha-test

> example@1.0.0 mocha-test
> mocha ./sum.test.js

  Sum Function
    ✔ should return 5 when adding 2 + 3
    ✔ should return -1 when adding 2 + (-3)

  2 passing (5ms)
```

---

## ✅ **Testing an Express API with Supertest**

### 📌 `server.js`

```js
const express = require("express");
const app = express();

app.get("/hello", (req, res) => {
  res.json({ message: "Hello, World!" });
});

module.exports = app;

if (require.main === module) {
  app.listen(3000, () => console.log("Server running on port 3000"));
}
```

### 📌 `server.test.js` (Jest & Supertest)

```js
const request = require("supertest");
const app = require("./server");

test("GET /hello should return Hello, World!", async () => {
  const res = await request(app).get("/hello");
  expect(res.statusCode).toBe(200);
  expect(res.body).toEqual({ message: "Hello, World!" });
});
```

#### ✅ **Expected Output (Jest)**

```sh
npx jest

 PASS  ./server.test.js
  √ GET /hello should return Hello, World! (51 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
```

---

## 🚀 **Running Tests**

### ✅ Run Jest Tests

```sh
npm test
```

### ✅ Run Mocha Tests

```sh
npm run mocha-test
```

### ✅ Run Jest for API Tests

```sh
npx jest
```

---

## 🎯 **Conclusion**

- **Jest** is great for unit and API testing. ( https://jestjs.io/docs/getting-started )
- **Mocha + Chai** provide flexibility with assertion styles. ( https://mochajs.org/#getting-started )
- **Supertest** makes API testing easy. ( https://www.npmjs.com/package/supertest )

Now you have a working test setup in Node.js! 🚀
