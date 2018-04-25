---
title: The Go Programming Language Ex（9）
date: 2018-01-01 00:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go goroutine
- Go TCP Conn
---

### Ex 8.1 

Modify clock2 to accept a port number, and write a program, clockwall, that acts as a client of several clock servers at once, reading the times from each one and displaying the results in a table, akin to the wall of clocks seen in some business ofﬁces. If you have access to geographically distributed computers, run instances remotely ; otherwise run local instances on different ports with fake time zones.

<!--more-->

$ TZ=US/Eastern ./clock2 -port 8010 & $ TZ=Asia/Tokyo ./clock2 -port 8020 & $ TZ=Europe/London ./clock2 -port 8030 & $ clockwall NewYork=localhost:8010 London=localhost:8020 Tokyo=localhost:8030

```go
//clockwall
package main

import (
	"html/template"
	"log"
	"net"
	"net/http"
)

type City struct {
	Name string
	Addr string
	Time string
}

type cities []City

const templ = `
<h1>current times:</h1>
<table>
<tr style='text-align: left'>
  <th>City</th>
  <th>Time</th>
</tr>
{{range . }}
<tr>
  <td>{{.Name}}</td>
  <td>{{.Time}}</td>
</tr>
{{end}}
</table>
`

var report = template.Must(template.New("result").
	Parse(templ))
var cs = cities{
	{"NewYork", "localhost:8010", ""},
	{"Tokyo", "localhost:8020", ""},
	{"London", "localhost:8030", ""},
}

func main() {
	for i := range cs {
		go func(c *City) {
			conn, err := net.Dial("tcp", c.Addr)
			if err != nil {
				log.Fatal(err)
			}
			defer conn.Close()
			buf := make([]byte, 20)
			for {
				n, _ := conn.Read(buf)
				c.Time = string(buf[:n])
			}
		}(&cs[i])
	}
	http.HandleFunc("/gettime", cs.gettime)
	log.Fatal(http.ListenAndServe("localhost:8000", nil))
}

func (cs cities) gettime(w http.ResponseWriter, req *http.Request) {
	if err := report.Execute(w, cs); err != nil {
		log.Fatal(err)
	}
}
```

```go
//clock
package main

import (
	"flag"
	"io"
	"log"
	"net"
	"time"
)

func handleConn(c net.Conn) {
	defer c.Close()
	for {
		_, err := io.WriteString(c, time.Now().Format("15:04:05\n"))
		if err != nil {
			return // e.g., client disconnected
		}
		time.Sleep(1 * time.Second)
	}
}

func main() {
	port := flag.String("port", "8000", "port num")
	flag.Parse()
	address := "localhost:" + *port
	listener, err := net.Listen("tcp", address)
	if err != nil {
		log.Fatal(err)
	}
	//!+
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err) // e.g., connection aborted
			continue
		}
		go handleConn(conn) // handle connections concurrently
	}
	//!-
}
```

```bash
//启动服务器
TZ=US/Eastern    ./clock -port 8010 &
TZ=Asia/Tokyo    ./clock -port 8020 &
TZ=Europe/London ./clock -port 8030
```

![](http://oumnldfwl.bkt.clouddn.com/clockwall.png?imageView2/0/h/200)

### Ex 8.3

In netcat3, the interface value conn has the concrete type *net.TCPConn, which represents a TCP connection. A TCP connection consists of two halves that may be closed independently using its CloseRead and CloseWrite methods. Modify the main goroutine of netcat3 to close only the write half of the connection so that the program will continue to print the ﬁnal echoes from the reverb1 server even after the standard input has been closed. (Doing this for the reverb2 server is harder; see Exercise 8.4.)

```go
package main

import (
	"io"
	"log"
	"net"
	"os"
)

//!+
func main() {
	conn, err := net.Dial("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	done := make(chan struct{})
	go func() {
		io.Copy(os.Stdout, conn) // NOTE: ignoring errors
		log.Println("done")
		done <- struct{}{} // signal the main goroutine
	}()
	mustCopy(conn, os.Stdin)
	//conn.Close()
	if conn, ok := conn.(*net.TCPConn); ok {
		conn.CloseWrite()
	}
	<-done // wait for background goroutine to finish
}

//!-

func mustCopy(dst io.Writer, src io.Reader) {
	if _, err := io.Copy(dst, src); err != nil {
		log.Fatal(err)
	}
}
```

### Ex 8.4

Modify the reverb2 server to use a sync.WaitGroup per connection to count the number of active echo goroutines. When it falls to zero, close the write half of the TCP connection as described in Exercise 8.3. Verify that your modiﬁed netcat3 client from that exercise waits for the ﬁnal echoes of multiple concurrent shouts, even after the standard input has been closed.

```go
//client
package main

import (
	"io"
	"log"
	"net"
	"os"
)

func init() {
	log.SetFlags(log.Ldate | log.Lmicroseconds)
}

//!+
func main() {
	f, err := os.Open("a")
	defer f.Close()
	conn, err := net.Dial("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	done := make(chan struct{})
	go func() {
		io.Copy(os.Stdout, conn) // NOTE: ignoring errors
		log.Println("done")
		done <- struct{}{} // signal the main goroutine
	}()
	mustCopy(conn, f)
	//conn.Close()
	log.Printf("local in server: %#v\n", conn.LocalAddr())
	log.Printf("remote in server: %#v\n", conn.RemoteAddr())
	err = conn.(*net.TCPConn).CloseWrite() //经实验：必须要关，否则无法通知服务端输入已结束（但不应该是服务端根据EOF来判断吗？）
	log.Println("write finished in client: ", err)
	<-done // wait for background goroutine to finish
	log.Println("finish print")
	//err = conn.(*net.TCPConn).CloseRead()
	//log.Println("read finished in client: ", err)
	err = conn.Close()
	log.Println(err)
}

//!-

func mustCopy(dst io.Writer, src io.Reader) {
	if _, err := io.Copy(dst, src); err != nil {
		log.Fatal(err)
	}
}
```

```go
//server
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// See page 224.

// Reverb2 is a TCP server that simulates an echo.
package main

import (
	"bufio"
	"fmt"
	"log"
	"net"
	"strings"
	"sync"
	"time"
)

func init() {
	log.SetFlags(log.Ldate | log.Lmicroseconds)
}
func echo(c net.Conn, shout string, delay time.Duration, wg *sync.WaitGroup) {
	defer wg.Done()
	fmt.Fprintln(c, "\t", strings.ToUpper(shout))
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", shout)
	time.Sleep(delay)
	fmt.Fprintln(c, "\t", strings.ToLower(shout))
}

//!+
func handleConn(c net.Conn) {
	input := bufio.NewScanner(c)
	var wg sync.WaitGroup
	for input.Scan() {
		wg.Add(1)
		go echo(c, input.Text(), 1*time.Second, &wg)
		fmt.Println(input.Text())
	}
	// NOTE: ignoring potential errors from input.Err()
	//err := c.(*net.TCPConn).CloseRead()
	//log.Println("read finished in server: ", err)
	log.Println("finish scan")
	log.Printf("local in server: %#v\n", c.LocalAddr())
	log.Printf("remote in server: %#v\n", c.RemoteAddr())
	wg.Wait()
	err := c.(*net.TCPConn).CloseWrite()
	//err := c.Close()
	log.Println("finished in server: ", err)
}

//!-

func main() {
	l, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}
	for {
		conn, err := l.Accept()
		if err != nil {
			log.Print(err) // e.g., connection aborted
			continue
		}
		go handleConn(conn)
	}
}
```

```bash
//client
$ ./ex8.3
2018/01/02 08:56:18.122881 local in server: &net.TCPAddr{IP:net.IP{0x7f, 0x0, 0x0, 0x1}, Port:59580, Zone:""}
2018/01/02 08:56:18.123015 remote in server: &net.TCPAddr{IP:net.IP{0x7f, 0x0, 0x0, 0x1}, Port:8000, Zone:""}
2018/01/02 08:56:18.123035 write finished in client:  <nil>
	 A
	 F
	 B
	 C
	 D
	 E
	 G
	 e
	 b
	 c
	 f
	 d
	 g
	 a
	 e
	 a
	 d
	 b
	 f
	 c
	 g
2018/01/02 08:56:20.125582 done
2018/01/02 08:56:20.125625 finish print
2018/01/02 08:56:20.125699 <nil>
```

```bash
//server
$ ./ex8.4
a
b
c
d
e
f
g
2018/01/02 08:56:18.123125 finish scan
2018/01/02 08:56:18.123409 local in server: &net.TCPAddr{IP:net.IP{0x7f, 0x0, 0x0, 0x1}, Port:8000, Zone:""}
2018/01/02 08:56:18.123427 remote in server: &net.TCPAddr{IP:net.IP{0x7f, 0x0, 0x0, 0x1}, Port:59580, Zone:""}
2018/01/02 08:56:20.125383 finished in server:  <nil>
```

- conn是双向的（从各自local，remote属性相反可以看出），客户端本身对连接的关闭，只会影响客户端本身通过该连接的相关操作，不会影响服务器端连接；反之亦然！
- 经过实验，客户端通过标准输入向conn写入消息时，即使写入EOF，服务器端也还会继续等待客户端输入的到来，只有当客户端CloseWrite()后，服务器端才会放弃等待输入，继续后面的逻辑；反之服务器端写，客户端读，也是这种情况。似乎与练习要达到的效果有些出入（手动ctrl-d，文件EOF都不行）~