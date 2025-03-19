# Load Testing, Unit Testing, and API Testing for Node.js and Go

## Introduction

This document provides a detailed explanation of how to perform **Load Testing, Unit Testing, and API Testing** in both **Node.js** and **Go**. We will cover the tools required, step-by-step setup, and execution for each testing type.

---

# 1️⃣ Load Testing

## What is Load Testing?

Load testing checks how well an application performs under **normal and peak loads**. It ensures that the system can handle expected traffic without slowing down or crashing.

### 📌 Tools for Load Testing:

- **k6** (Recommended for API Load Testing)
- **Apache JMeter** (GUI-based performance testing tool)
- **Gatling** (Highly scalable and script-based)
- **Vegeta** (Go-based HTTP load testing tool)

## 🔹 Load Testing in Node.js using k6

### **Step 1: Install k6**

```sh
npm install -g k6
```

### **Step 2: Create `server.js`**

```javascript
import express from "express";
const app = express();
const port = 3000;

app.get("/", function (req, res) {
  res.json({ message: "Load testing!" });
});

app.listen(port, (req, res) => console.log(`listening on port ${port}`));
```

### **Step 3: Create `load-test.js`**

```javascript
import http from "k6/http";
import { check } from "k6";

export let options = {
  vus: 10, // 10 virtual users
  duration: "30s", // Test duration
};

export default function () {
  let res = http.get("http://localhost:3000/");

  check(res, {
    "Status is 200": (r) => r.status === 200,
  });
}
```

### **Step 4: Run the Test**

```sh
k6 run load-test.js
```

**or**

### **Run the Test using Docker**

```sh
docker run --rm -i grafana/k6 run - < load-test.js
```

---

## 🔹 Load Testing in Go using k6

### **Step 1: Create a Go API `main.go`**

```go
package main

import (
	"net/http"
	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/users", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "Users fetched successfully"})
	})
	r.Run(":8080")
}
```

### **Step 2: Create `load-test.js`**

```javascript
import http from "k6/http";
import { check } from "k6";

export let options = {
  vus: 10, // 10 virtual users
  duration: "10s", // Test duration
};

export default function () {
  let res = http.get("http://host.docker.internal:8080/users");

  check(res, {
    "Status is 200": (r) => r.status === 200,
  });
}
```

### **Step 3: Run Load Test using k6**

```sh
k6 run load-test.js
```

**or**

### **Run the Test using Docker**

```sh
docker run --rm -i grafana/k6 run - < load-test.js
```

---

# 2️⃣ Unit Testing

## What is Unit Testing?

Unit testing involves testing **individual functions** to verify that they work correctly in isolation.

### 📌 Tools for Unit Testing:

- **Node.js**: Jest, Mocha + Chai
- **Go**: Built-in `testing` package

---

## 🔹 Unit Testing in Node.js using Jest

### **Step 1: Install Jest**

```sh
npm install --save-dev jest
```

### **Step 2: Create `math.js`**

```javascript
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

### **Step 3: Create `math.test.js`**

```javascript
const sum = require("./math");

test("adds 2 + 3 to equal 5", () => {
  expect(sum(2, 3)).toBe(5);
});
```

### **Step 4: Run the Test**

```sh
npx jest
```

---

## 🔹 Unit Testing in Go

### **Step 1: Create `math.go`**

```go
package math

func Sum(a, b int) int {
	return a + b
}
```

### **Step 2: Create `math_test.go`**

```go
package math

import "testing"

func TestSum(t *testing.T) {
	result := Sum(2, 3)
	expected := 5

	if result != expected {
		t.Errorf("Expected %d, but got %d", expected, result)
	}
}
```

### **Step 3: Run the Test**

```sh
go test
```

---

# 3️⃣ API Testing

## What is API Testing?

API testing verifies that API endpoints return **correct responses** and handle **errors** correctly.

### 📌 Tools for API Testing:

- **Postman** (Manual testing)
- **Supertest** (Node.js API automation)
- **`httptest`** (Go built-in package)

---

## 🔹 API Testing in Node.js using Supertest

### **Step 1: Install Dependencies**

```sh
npm install --save-dev supertest jest
```

### **Step 2: Create `server.js`**

```javascript
const express = require("express");
const app = express();

app.get("/users", (req, res) => {
  res.json([
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
  ]);
});

module.exports = app;
```

### **Step 3: Create `server.test.js`**

```javascript
const request = require("supertest");
const app = require("./server");

test("GET /users should return user data", async () => {
  const res = await request(app).get("/users");

  expect(res.status).toBe(200);
  expect(res.body.length).toBeGreaterThan(0);
  expect(res.body[0]).toHaveProperty("name");
});
```

### **Step 4: Run the Test**

```sh
npx jest
```

---

## 🔹 API Testing in Go using `httptest`

### **Step 1: Create `server.go`**

```go
package main

import (
	"net/http"
	"github.com/gin-gonic/gin"
)

func SetupRouter() *gin.Engine {
	r := gin.Default()
	r.GET("/users", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "Users fetched successfully"})
	})
	return r
}

func main() {
	r := SetupRouter()
	r.Run(":8080")
}
```

### **Step 2: Create `server_test.go`**

```go
package main

import (
	"net/http"
	"net/http/httptest"
	"testing"
	"github.com/stretchr/testify/assert"
)

func TestUsersEndpoint(t *testing.T) {
	router := SetupRouter()

	req, _ := http.NewRequest("GET", "/users", nil)
	w := httptest.NewRecorder()
	router.ServeHTTP(w, req)

	assert.Equal(t, 200, w.Code)
	assert.Contains(t, w.Body.String(), "Users fetched successfully")
}
```

### **Step 3: Run the Test**

```sh
go test
```

---

# 🎯 Conclusion

| **Testing Type** | **Node.js Tool**   | **Go Tool**   |
| ---------------- | ------------------ | ------------- |
| Load Testing     | k6                 | k6, Vegeta    |
| Unit Testing     | Jest, Mocha        | `testing` pkg |
| API Testing      | Supertest, Postman | `httptest`    |

This guide provides a detailed walkthrough for performing testing in both **Node.js** and **Go** effectively. 🚀
