# Testing in Go

## 📌 Overview

Go provides a built-in testing framework that allows developers to write unit tests, benchmark tests, and API tests efficiently. In this guide, we will cover:

- Unit Testing
- Table-Driven Testing
- Benchmarking
- API Testing
- Mocking with `gomock`

## ✅ Running Tests in Go

To execute tests in Go, use the `go test` command:

```sh
go test
```

Example Output:

```
$ go test
PASS
ok      test    0.191s
```

---

## 📌 1. Unit Testing

Unit testing in Go is done using the `testing` package. Test files should be named with `_test.go`, and test functions should start with `Test`.

### 📝 Example: Sum Function

```go
package main

func Sum(a, b int) int {
    return a + b
}
```

### 📝 Unit Test (`sum_test.go`)

```go
package main

import "testing"

func TestSum(t *testing.T) {
    result := Sum(2, 3)
    expected := 5

    if result != expected {
        t.Errorf("Expected %d, got %d", expected, result)
    }
}
```

✅ **Run the test:**

```sh
go test
```

Example Output:

```
$ go test
PASS
ok      test    0.191s
```

---

## 📌 2. Table-Driven Testing

Table-driven tests allow you to test multiple cases in a single function.

### 📝 Example:

```go
func TestSumMultipleCases(t *testing.T) {
    testCases := []struct {
        a, b, expected int
    }{
        {1, 2, 3},
        {2, 3, 6}, // Wrong expected value to show failure
        {10, -5, 5},
    }

    for _, tc := range testCases {
        result := Sum(tc.a, tc.b)
        if result != tc.expected {
            t.Errorf("Sum(%d, %d) = %d; want %d", tc.a, tc.b, result, tc.expected)
        }
    }
}
```

❌ **Failing Test Output:**

```
go test
{1 2 3}
{2 3 6}
{10 -5 5}
--- FAIL: TestSumMultipleCases (0.00s)
    sum_test.go:39: Sum(2, 3) = 5; want 6
FAIL
exit status 1
FAIL    test    0.164s
```

---

## 📌 3. Benchmarking

Benchmark tests measure the performance of functions.

### 📝 Benchmark Example:

```go
import "testing"

func BenchmarkSum(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Sum(2, 3)
    }
}
```

✅ **Run Benchmark Tests:**

```sh
go test -bench=.
```

Example Output:

```
$ go test -bench=.
goos: windows
goarch: amd64
pkg: test
cpu: AMD Ryzen 5 4600H with Radeon Graphics
BenchmarkSum-12         1000000000               0.2572 ns/op
PASS
ok      test    0.476s
```

---

## 📌 4. API Testing

The `httptest` package allows you to test HTTP handlers.

### 📝 Example: Simple HTTP Handler (`server.go`)

```go
package main

import (
    "fmt"
    "net/http"
)

func HelloHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Hello, World!")
}
```

### 📝 API Test (`server_test.go`)

```go
package main

import (
    "net/http"
    "net/http/httptest"
    "testing"
)

func TestHelloHandler(t *testing.T) {
    req, err := http.NewRequest("GET", "/", nil)
    if err != nil {
        t.Fatal(err)
    }

    recorder := httptest.NewRecorder()
    handler := http.HandlerFunc(HelloHandler)

    handler.ServeHTTP(recorder, req)

    expected := "Hello!\n"
    if recorder.Body.String() != expected {
        t.Errorf("Expected %q, got %q", expected, recorder.Body.String())
    }
}
```

❌ **Failing Test Output:**

```
$ go test
--- FAIL: TestHelloHandler (0.00s)
    server_test.go:22: Expected "Hello!\n", got "Hello, World!\n"
FAIL
exit status 1
FAIL    test    0.206s
```

✅ **Passing Test Output:**

```
$ go test
PASS
ok      test    0.222s
```

---

## 📌 5. Mocking with `gomock`

`gomock` is used to create mock dependencies for interfaces.

### ✅ Install `gomock`

```sh
go install github.com/golang/mock/mockgen@latest
go get github.com/golang/mock/gomock
```

### 📝 Example: Mocking a Service (`service.go`)

```go
package main

type UserService interface {
    GetUser(id int) (string, error)
}
```

✅ **Generate Mocks**

```sh
mockgen -source=service.go -destination=mocks/service_mock.go -package=mocks
```

### 📝 Writing a Test with `gomock` (`user_handler_test.go`)

```go
package main

import (
    "testing"
    "github.com/golang/mock/gomock"
    "github.com/your_project/mocks"
)

func TestGetUserName(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()

    mockUserService := mocks.NewMockUserService(ctrl)

    mockUserService.EXPECT().GetUser(1).Return("Alice", nil)
    mockUserService.EXPECT().GetUser(2).Return("", fmt.Errorf("not found"))

    result := GetUserName(mockUserService, 1)
    if result != "Alice" {
        t.Errorf("Expected Alice, got %s", result)
    }
}
```

✅ **Run Tests:**

```sh
go test ./...
```

Example Output:

```
$ go test ./...
ok      testing_go      0.172s
```

---

## 📌 Conclusion

- **Unit Tests:** Use `go test` to test functions.
- **Table-Driven Tests:** Run multiple test cases in a single test function.
- **Benchmarking:** Use `go test -bench=.` to test performance.
- **API Testing:** Use `httptest` for HTTP handler tests.
- **Mocking:** Use `gomock` to mock interfaces.

🚀 Now you are ready to test Go applications efficiently!
