```go
package main

import (
	"fmt"
)

```

## Variadic

```go
func printTotal(n ...int) {
	total := 0

	for _, v := range n {
		total += v
	}

	fmt.Println("Total: ", total)
}

func main() {

	nums := []int{1, 2, 3, 4}
	printTotal(nums...)

}
```

## Closer function

```go
func checkData() func() string {
	return func() string {
		data := "Client data received at the Server"
		return data
	}
}

func main() {
	d := checkData()
	fmt.Println(d())
}
```

## Pointer

```go
type Client struct {
	Name    string
	Pan     string
	Aadhaar string
	Email   string
}

func main() {
	c := Client{
		Name:    "ICICI Bank Client",
		Pan:     "ABCDE1234F",
		Aadhaar: "1234 5678 1234",
		Email:   "icicibankuser123@gmail.com",
	}

	c.checkUsers()

	fmt.Println(c)

}

func (u *Client) checkUsers() {
	if u.Aadhaar == "1234 5678 1235" {
		u.Name = "ICICI Bank Client"
	} else {
		u.Name = "Not ICICI Bank Client"
	}
}
```

## Enum

```go
type Payment int

const (
	Credit Payment = iota
	Debit
	Deduct
	Pending
)

func checkPayment(payment Payment) {
	fmt.Println("Payment Status: ", payment)
}

func main() {
	checkPayment(Debit)
}
```

## Interface

```go

// Interface is a contract, it uses implicitly declaration as Go don't support explicitly declaration

package main

import "fmt"

type paymenter interface {
	pay(amount float32)
}

type payment struct {
	getway paymenter
}

func (p payment) makePayment(amount float32) {
	// razorpayGw := razorpay{}
	// razorpayGw.pay(amount)

	// paypalGw := paypal{}
	// paypalGw.pay(amount)

	p.getway.pay(amount)
}

type razorpay struct{}

func (r razorpay) pay(amount float32) {
	fmt.Println("Making payment using razorpay: ", amount)
}

type paypal struct{}

func (p paypal) pay(amount float32) {
	fmt.Println("Making payment using paypal: ", amount)
}

func main() {
	// newPayment := payment{}
	// newPayment.makePayment(1000)

	// newPaymentGw := razorpay{}
	// newPayment := payment{
	// 	getway: newPaymentGw,
	// }
	// newPayment.makePayment(1000)

	newPaymentGw := razorpay{}
	newPayment := payment{
		getway: newPaymentGw,
	}
	newPayment.makePayment(1000)
}

```

## Interface concept in Javascript

```javascript
class Payment {
  constructor(gateway) {
    this.gateway = gateway;
  }

  makePayment(amount) {
    this.gateway.pay(amount);
  }
}

class Razorpay {
  pay(amount) {
    console.log("Making payment using Razorpay:", amount);
  }
}

class Paypal {
  pay(amount) {
    console.log("Making payment using PayPal:", amount);
  }
}

// Example usage
const razorpayGateway = new Razorpay();
razorpayGateway.pay(1000);
const newPayment = new Payment(razorpayGateway);
newPayment.makePayment(1000);

// Using PayPal
const paypalGateway = new Paypal();
const anotherPayment = new Payment(paypalGateway);
anotherPayment.makePayment(500);
```

## Generics

```go
func generateData[T any](d []T) {
	for _, v := range d {
		fmt.Println(v)
	}
}

func main() {
	generateData([]string{"golang", "go"})
}
```

## Goroutine

```go
func sayHello(s string) {
	time.Sleep(1000 * time.Millisecond)
	fmt.Println(s)
}

func main() {
	go sayHello("Hello")
	sayHello("World")
}
```

## Channel

```go
func main() {
	ch := make(chan int)

	go func() {
		for value := range ch {
			fmt.Println(value)
		}
		fmt.Println("Channel closed successfully!")
	}()

	ch <- 100
	ch <- 200
	close(ch)
}
```

## Unbuffered Channel

```go

package main

import (
	"fmt"
	"time"
)

func chef(orderChannel chan string) {
	// Receive an order from the channel
	order := <-orderChannel
	fmt.Println("Chef received order:", order)

	// Simulate cooking time
	time.Sleep(2 * time.Second)
	fmt.Println("Chef finished preparing:", order)
}

func main() {
	orderChannel := make(chan string) // Create a channel

	go chef(orderChannel) // Start chef as a goroutine

	fmt.Println("Waiter taking order: Burger")
	orderChannel <- "Burger" // Send order to chef

	// Wait before main() exits to see the output
	time.Sleep(3 * time.Second)
}

```

## Buffered Channel

```go

package main

import (
	"fmt"
	"time"
)

func chef(orderChannel chan string) {
	for order := range orderChannel { // Continuously receive orders
		fmt.Println("Chef received order:", order)
		time.Sleep(2 * time.Second) // Simulate cooking time
		fmt.Println("Chef finished preparing:", order)
	}
}

func main() {
	orderChannel := make(chan string, 3) // Buffered channel with capacity 3

	go chef(orderChannel) // Start chef as a goroutine

	// Waiter taking multiple orders
	fmt.Println("Waiter taking orders...")
	orderChannel <- "Burger"
	orderChannel <- "Pizza"
	orderChannel <- "Pasta"

	fmt.Println("Waiter finished taking orders!")

	// Allow time for the chef to process the orders
	time.Sleep(7 * time.Second)
}

```
