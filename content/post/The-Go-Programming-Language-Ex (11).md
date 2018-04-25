---
title: The Go Programming Language Ex（11）
date: 2018-01-04 00:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go goroutine
- Go cache store
---

### Ex 9.1

Add a function Withdraw(amount int) bool to the gopl.io/ch9/bank1 program. The result should indicate whether the transaction succeeded or failed due to insufﬁcient funds. The message sent to the monitor goroutine must contain both the amount to withdraw and a new channel over which the monitor goroutine can send the boolean result back to Withdraw.

<!--more-->

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

package bank_test

import (
	"fmt"
	"testing"

	"The_Go_Programming_Language_Exercises/CH9/ex9.1"
)

func TestBank(t *testing.T) {
	done := make(chan struct{})

	// Alice
	go func() {
		bank.Deposit(200)
		fmt.Println("=", bank.Balance())
		done <- struct{}{}
	}()

	// Bob
	go func() {
		bank.Deposit(100)
		done <- struct{}{}
	}()

	go func() {
		fmt.Println(bank.Withdraw(500))
		done <- struct{}{}
	}()

	// Wait for both transactions.
	<-done
	<-done
	<-done

	if got, want := bank.Balance(), 300; got != want {
		t.Errorf("Balance = %d, want %d", got, want)
	}
}
```

### Ex 9.2

Rewrite the PopCount example from Section 2.6.2 so that it initializes the l table using sync.Once the ﬁrst time it is needed. (Realistically, the cost of synchronization would be prohibitive for a small and highly optimized function like PopCount.)

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// See page 45.

// (Package doc comment intentionally malformed to demonstrate golint.)
//!+
package popcount

import "sync"

// pc[i] is the population count of i.
var pc [256]byte
var loadpcOnce sync.Once

//func init() {
//	for i := range pc {
//		pc[i] = pc[i/2] + byte(i&1)
//	}
//}

// PopCount returns the population count (number of set bits) of x.
func PopCount(x uint64) int {
	loadpcOnce.Do(loadpc)
	return int(pc[byte(x>>(0*8))] +
		pc[byte(x>>(1*8))] +
		pc[byte(x>>(2*8))] +
		pc[byte(x>>(3*8))] +
		pc[byte(x>>(4*8))] +
		pc[byte(x>>(5*8))] +
		pc[byte(x>>(6*8))] +
		pc[byte(x>>(7*8))])
}

func loadpc() {
	for i := range pc {
		pc[i] = pc[i/2] + byte(i&1)
	}
}

//!-

```



### Ex 9.3

Extend the Func type and the (*Memo).Get method so that callers may provide an optional done channel through which they can cancel the operation (§8.9). The results of a cancelled Func call should not be cached.

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// See page 278.

// Package memo provides a concurrency-safe non-blocking memoization
// of a function.  Requests for different keys proceed in parallel.
// Concurrent requests for the same key block until the first completes.
// This implementation uses a monitor goroutine.
package memo

import "errors"
import "time"

//!+Func

// Func is the type of the function to memoize.
type Func func(key string) (interface{}, error)

// A result is the result of calling a Func.
type result struct {
	value interface{}
	err   error
}

type entry struct {
	res   result
	ready chan struct{} // closed when res is ready
}

//!-Func

//!+get

// A request is a message requesting that the Func be applied to key.
type request struct {
	key      string
	response chan<- result // the client wants a single result
}

type Memo struct{ requests chan request }

// New returns a memoization of f.  Clients must subsequently call Close.
func New(f Func) *Memo {
	memo := &Memo{requests: make(chan request)}
	go memo.server(f)
	return memo
}

func (memo *Memo) Get(key string) (interface{}, error) {
	response := make(chan result)
	memo.requests <- request{key, response}
	res := <-response
	return res.value, res.err
}

func (memo *Memo) Close() { close(memo.requests) }

//!-get

//!+monitor

func (memo *Memo) server(f Func) {
	var done = make(chan struct{})
	// Cancel traversal when input is detected.
	go func() {
		//os.Stdin.Read(make([]byte, 1)) // read a single byte
		time.Sleep(300 * time.Millisecond)
		close(done)
	}()
	cancelled := func() bool {
		select {
		case <-done:
			return true
		default:
			return false
		}
	}
	cache := make(map[string]*entry)
	for req := range memo.requests {
		if cancelled() {
			//close(memo.requests)
			break
		}
		e := cache[req.key]
		if e == nil {
			// This is the first request for this key.
			e = &entry{ready: make(chan struct{})}
			cache[req.key] = e
			go e.call(f, req.key) // call f(key)
			if cancelled() {
				delete(cache, req.key)
			}
		}
		go e.deliver(req.response, done)
	}
}

func (e *entry) call(f Func, key string) {
	// Evaluate the function.
	e.res.value, e.res.err = f(key)
	// Broadcast the ready condition.
	close(e.ready)
}

func (e *entry) deliver(response chan<- result, done <-chan struct{}) {
	// Wait for the ready condition.
	select {
	case <-e.ready:
		response <- e.res
	case <-done:
		response <- result{nil, errors.New("request cancelled")}
	}
}
```

