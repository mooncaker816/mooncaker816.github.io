---
title: The Go Programming Language Ex（8）
date: 2017-12-31 00:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go Interface
- Go Scanner
---

### Ex 7.1

Using the ideas from ByteCounter, implement counters for words and for lines. You will ﬁnd bufio.ScanWords useful.

<!--more-->

```go
package main

import (
	"bufio"
	"bytes"
	"fmt"
)

func main() {
	var c ByteCounter
	c.Write([]byte("hello"))
	fmt.Println(c) // "5", = len("hello")
	c = 0          // reset the counter
	var name = "Dolly"
	fmt.Fprintf(&c, "hello, %s", name)
	fmt.Println(c) // "12", = len("hello, Dolly")

	var w WordCounter
	w.Write([]byte("hello world, 世界"))
	fmt.Println(w)
	w = 0
	var str = "你 好"
	fmt.Fprintf(&w, "hello, %s", str)
	fmt.Println(w)

	var l LineCounter
	l.Write([]byte("hello world, 世界\nabc\nosjoij\nwefw"))
	fmt.Println(l)
}

type ByteCounter int

func (c *ByteCounter) Write(p []byte) (int, error) {
	*c += ByteCounter(len(p)) // convert int to ByteCounter
	return len(p), nil
}

type WordCounter int

func (w *WordCounter) Write(p []byte) (int, error) {
	scanner := bufio.NewScanner(bytes.NewReader(p))
	scanner.Split(bufio.ScanWords)
	var count int
	for scanner.Scan() {
		count++
	}
	*w += WordCounter(count)
	return count, nil
}

type LineCounter int

func (l *LineCounter) Write(p []byte) (int, error) {
	scanner := bufio.NewScanner(bytes.NewReader(p))
	scanner.Split(bufio.ScanLines)
	var count int
	for scanner.Scan() {
		count++
	}
	*l += LineCounter(count)
	return count, nil
}
```

```bash
$ go run counter.go
5
12
3
3
4
```

### Ex 7.2

Write a function CountingWriter with the signature below that, given an io.Writer, returns a new Writer that wraps the original, and a pointer to an int64 variable that at any moment contains the number of bytes written to the new Writer.

`func CountingWriter(w io.Writer) (io.Writer, *int64)`

```go
package main

import (
	"fmt"
	"io"
	"os"
)

type byteCounter struct {
	n int64
	w io.Writer
}

func (c *byteCounter) Write(p []byte) (int, error) {
	c.n += int64(len(p))
	var err error
	if c.w != nil {
		_, err = c.w.Write(p)
	}
	return len(p), err
}

func CountingWriter(w io.Writer) (io.Writer, *int64) {
	var b byteCounter
	b.w = w
	return &b, &b.n
}

func main() {
	w, n := CountingWriter(os.Stdout)
	fmt.Fprintf(w, "hello, word!\n")
	fmt.Printf("writer count [%d]\n", *n)
	fmt.Fprintf(w, "1234567890\n")
	fmt.Printf("writer count [%d]\n", *n)
}
```

```bash
$ go run counting.go
hello, word!
writer count [13]
1234567890
writer count [24]
```

### Ex 7.11

Add additional handlers so that clients can create, read, update, and delete database entries. For example, a request of the form /update?item=socks&price=6 will update the price of an item in the inventory and report an error if the item does not exist or if the price is invalid. (Warning: this change introduces concurrent variable updates.)

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"strconv"
	"sync"
)

var mu sync.Mutex

func main() {
	db := database{"shoes": 50, "socks": 5}
	http.HandleFunc("/list", db.list)
	http.HandleFunc("/price", db.price)
	http.HandleFunc("/update", db.update)
	http.HandleFunc("/add", db.add)
	http.HandleFunc("/delete", db.delete)
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars

func (db database) list(w http.ResponseWriter, req *http.Request) {
	for item, price := range db {
		fmt.Fprintf(w, "%s: %s\n", item, price)
	}
}

func (db database) price(w http.ResponseWriter, req *http.Request) {
	item := req.URL.Query().Get("item")
	if price, ok := db[item]; ok {
		fmt.Fprintf(w, "%s\n", price)
	} else {
		w.WriteHeader(http.StatusNotFound) // 404
		fmt.Fprintf(w, "no such item: %q\n", item)
	}
}

func (db database) update(w http.ResponseWriter, req *http.Request) {
	mu.Lock()
	defer mu.Unlock()
	item := req.URL.Query().Get("item")
	if _, ok := db[item]; ok {
		newpricestr := req.URL.Query().Get("price")
		newprice, err := strconv.ParseFloat(newpricestr, 32)
		if err != nil {
			w.Write([]byte("price invalid!"))
		} else {
			db[item] = dollars(float32(newprice))
			w.Write([]byte("new price updated!\n\n"))
			db.list(w, req)
		}
	} else {
		fmt.Fprintf(w, "no such item: %q\n", item)
	}
}

func (db database) add(w http.ResponseWriter, req *http.Request) {
	mu.Lock()
	defer mu.Unlock()
	item := req.URL.Query().Get("item")
	if _, ok := db[item]; ok {
		fmt.Fprintf(w, "%s already exist!", item)
	} else {
		newpricestr := req.URL.Query().Get("price")
		if len(newpricestr) <= 0 {
			newpricestr = "0"
		}
		newprice, err := strconv.ParseFloat(newpricestr, 32)
		if err != nil {
			w.Write([]byte("price invalid!"))
		} else {
			db[item] = dollars(float32(newprice))
			fmt.Fprintf(w, "%s added!\n\n", item)
			db.list(w, req)
		}
	}
}

func (db database) delete(w http.ResponseWriter, req *http.Request) {
	mu.Lock()
	defer mu.Unlock()
	item := req.URL.Query().Get("item")
	if _, ok := db[item]; ok {
		delete(db, item)
		fmt.Fprintf(w, "%s deleted!\n\n", item)
		db.list(w, req)
	} else {
		fmt.Fprintf(w, "%s not exist!", item)
	}
}
```

![list](http://oumnldfwl.bkt.clouddn.com/list item.png?imageView2/0/h/200)

![](http://oumnldfwl.bkt.clouddn.com/add item.png?imageView2/0/h/200)

![](http://oumnldfwl.bkt.clouddn.com/update price.png?imageView2/0/h/200)

![](http://oumnldfwl.bkt.clouddn.com/delete item.png?imageView2/0/h/200)

### Ex 7.12

Change the handler for /list to print its output as an HTML table, not text. You may ﬁnd the html/template package (§4.6) useful.

```go
package main

import (
	"fmt"
	"html/template"
	"log"
	"net/http"
	"strconv"
	"sync"
)

var mu sync.Mutex

const templ = `
<h1>current prices:</h1>
<table>
<tr style='text-align: left'>
  <th>Item</th>
  <th>Price</th>
</tr>
{{range $key, $value := . }}
<tr>
  <td>{{$key}}</td>
  <td>{{$value}}</td>
</tr>
{{end}}
</table>
`

var report = template.Must(template.New("result").
	Parse(templ))

func main() {
	db := database{"shoes": 50, "socks": 5}
	http.HandleFunc("/list", db.list)
	http.HandleFunc("/price", db.price)
	http.HandleFunc("/update", db.update)
	http.HandleFunc("/add", db.add)
	http.HandleFunc("/delete", db.delete)
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

type dollars float32

func (d dollars) String() string { return fmt.Sprintf("$%.2f", d) }

type database map[string]dollars

func (db database) list(w http.ResponseWriter, req *http.Request) {
	if err := report.Execute(w, db); err != nil {
		log.Fatal(err)
	}
}

func (db database) price(w http.ResponseWriter, req *http.Request) {
	item := req.URL.Query().Get("item")
	if price, ok := db[item]; ok {
		fmt.Fprintf(w, "%s\n", price)
	} else {
		w.WriteHeader(http.StatusNotFound) // 404
		fmt.Fprintf(w, "no such item: %q\n", item)
	}
}

func (db database) update(w http.ResponseWriter, req *http.Request) {
	mu.Lock()
	defer mu.Unlock()
	item := req.URL.Query().Get("item")
	if _, ok := db[item]; ok {
		newpricestr := req.URL.Query().Get("price")
		newprice, err := strconv.ParseFloat(newpricestr, 32)
		if err != nil {
			w.Write([]byte("price invalid!"))
		} else {
			db[item] = dollars(float32(newprice))
			w.Write([]byte("new price updated!\n\n"))
			db.list(w, req)
		}
	} else {
		fmt.Fprintf(w, "no such item: %q\n", item)
	}
}

func (db database) add(w http.ResponseWriter, req *http.Request) {
	mu.Lock()
	defer mu.Unlock()
	item := req.URL.Query().Get("item")
	if _, ok := db[item]; ok {
		fmt.Fprintf(w, "%s already exist!", item)
	} else {
		newpricestr := req.URL.Query().Get("price")
		if len(newpricestr) <= 0 {
			newpricestr = "0"
		}
		newprice, err := strconv.ParseFloat(newpricestr, 32)
		if err != nil {
			w.Write([]byte("price invalid!"))
		} else {
			db[item] = dollars(float32(newprice))
			fmt.Fprintf(w, "%s added!\n\n", item)
			db.list(w, req)
		}
	}
}

func (db database) delete(w http.ResponseWriter, req *http.Request) {
	mu.Lock()
	defer mu.Unlock()
	item := req.URL.Query().Get("item")
	if _, ok := db[item]; ok {
		delete(db, item)
		fmt.Fprintf(w, "%s deleted!\n\n", item)
		db.list(w, req)
	} else {
		fmt.Fprintf(w, "%s not exist!", item)
	}
}
```

