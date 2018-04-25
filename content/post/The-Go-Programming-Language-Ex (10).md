---
title: The Go Programming Language Ex（10）
date: 2018-01-03 00:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go goroutine
- Go breadth first crawler
- Go select
- Go chat room
---

### Ex 8.6

Add depth-limiting to the concurrent crawler. That is, if the user sets -depth=3, then only URLs reachable by at most three links will be fetched.

<!--more-->

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// See page 241.

// Crawl2 crawls web links starting with the command-line arguments.
//
// This version uses a buffered channel as a counting semaphore
// to limit the number of concurrent calls to links.Extract.
package main

import (
	"fmt"
	"log"
	"os"

	"gopl.io/ch5/links"
)

//!+sema
// tokens is a counting semaphore used to
// enforce a limit of 20 concurrent requests.
var tokens = make(chan struct{}, 20)

func crawl(url string, deepth int, f *os.File) work {
	tokens <- struct{}{} // acquire a token
	list, err := links.Extract(url)
	fmt.Fprintln(f, deepth, " ", url, " ", len(list))
	<-tokens // release the token
	deepth++
	if err != nil {
		log.Print(err)
	}
	return work{list, deepth}
}

//!-sema
type work struct {
	worklist   []string
	workdeepth int
}

const DEEPTH = 2

//!+
func main() {
	wkpool := make(chan work)
	var n int // number of pending sends to worklist
	f, _ := os.Create("stat")
	defer f.Close()
	// Start with the command-line arguments.
	n++
	go func() { wkpool <- work{os.Args[1:], 0} }()

	// Crawl the web concurrently.
	seen := make(map[string]bool)
	for ; n > 0; n-- {
		wk := <-wkpool
		if wk.workdeepth > DEEPTH {
			continue
		}
		for _, link := range wk.worklist {
			if !seen[link] {
				seen[link] = true
				n++
				go func(link string, deepth int, f *os.File) {
					wkpool <- crawl(link, deepth, f)
				}(link, wk.workdeepth, f)
			}
		}
	}
}
```

### Ex 8.8

Using a select statement, add a timeout to the echo server from Section 8.3 so that it disconnects any client that shouts nothing within 10 seconds.

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
	log.Println("connected!")
	done := make(chan struct{})
	go func() {
		io.Copy(os.Stdout, conn) // NOTE: ignoring errors
		log.Println("done")
		done <- struct{}{} // signal the main goroutine
	}()
	go func() {
		mustCopy(conn, os.Stdin)
		//conn.Close()
		err = conn.(*net.TCPConn).CloseWrite() //经实验：必须要关，否则无法通知服务端输入已结束（但不应该是服务端根据EOF来判断吗？）
		log.Println("write finished in client: ", err)
	}()
	<-done // wait for background goroutine to finish
	log.Printf("local in server: %#v\n", conn.LocalAddr())
	log.Printf("remote in server: %#v\n", conn.RemoteAddr())
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
	msg := make(chan string)
	go func(c net.Conn) {
		//for {
		input := bufio.NewScanner(c)
		for input.Scan() {
			fmt.Println("getting: ", input.Text())
			msg <- input.Text()
		}
		msg <- "eof"
		//}
	}(c)
	ticker := time.NewTicker(5 * time.Second)
	var wg sync.WaitGroup
label1:
	for {
		select {
		case <-ticker.C:
			ticker.Stop()
			c.Close()
			fmt.Println("connect closed!!")
			return
		case s := <-msg:
			if s == "eof" {
				break label1
			}
			ticker.Stop()
			fmt.Println(s)
			wg.Add(1)
			go echo(c, s, 1*time.Second, &wg)
		}
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
2018/01/03 17:01:15.985187 connected!
2018/01/03 17:01:20.989292 done
2018/01/03 17:01:20.989515 local in server: &net.TCPAddr{IP:net.IP{0x7f, 0x0, 0x0, 0x1}, Port:49954, Zone:""}
2018/01/03 17:01:20.989576 remote in server: &net.TCPAddr{IP:net.IP{0x7f, 0x0, 0x0, 0x1}, Port:8000, Zone:""}
2018/01/03 17:01:20.989596 finish print
2018/01/03 17:01:20.989774 <nil>
```

```bash
//server
./ex8.8
2018/01/03 17:01:20.989248 connect closed!! 127.0.0.1:8000 127.0.0.1:49954
```

### Ex 8.9

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// See page 250.

// The du3 command computes the disk usage of the files in a directory.
package main

// The du3 variant traverses all directories in parallel.
// It uses a concurrency-limiting counting semaphore
// to avoid opening too many files at once.

import (
	"flag"
	"io/ioutil"
	"log"
	"os"
	"path/filepath"
	"sync"
	"time"
)

var vFlag = flag.Bool("v", false, "show verbose progress messages")

func init() {
	log.SetFlags(log.Ldate | log.Lmicroseconds)
}

//!+
func main() {
	// ...determine roots...

	//!-
	flag.Parse()

	// Determine the initial directories.
	roots := flag.Args()
	if len(roots) == 0 {
		roots = []string{"."}
	}

	//!+
	// Traverse each root of the file tree in parallel.
	//fileSizes := make(chan int64)
	var m sync.WaitGroup
	for _, root := range roots {
		var tick <-chan time.Time
		if *vFlag {
			tick = time.Tick(100 * time.Millisecond)
		}
		var fileSizes chan int64
		var n sync.WaitGroup
		fileSizes = make(chan int64)
		n.Add(1)
		go walkDir(root, &n, fileSizes)
		go func() {
			n.Wait()
			close(fileSizes)
		}()
		m.Add(1)
		go func(root string, tick <-chan time.Time) {
			defer m.Done()
			var nfiles, nbytes int64
		loop:
			for {
				select {
				case size, ok := <-fileSizes:
					if !ok {
						break loop // fileSizes was closed
					}
					nfiles++
					nbytes += size
				case <-tick:
					printDiskUsage(root, nfiles, nbytes)
				}
			}
			printDiskUsage(root, nfiles, nbytes) // final totals
		}(root, tick)
	}
	m.Wait()
}

//!-

func printDiskUsage(root string, nfiles, nbytes int64) {
	log.Printf("%s: %d files  %.1f GB\n", root, nfiles, float64(nbytes)/1e9)
}

// walkDir recursively walks the file tree rooted at dir
// and sends the size of each found file on fileSizes.
//!+walkDir
func walkDir(dir string, n *sync.WaitGroup, fileSizes chan<- int64) {
	defer n.Done()
	for _, entry := range dirents(dir) {
		if entry.IsDir() {
			n.Add(1)
			subdir := filepath.Join(dir, entry.Name())
			go walkDir(subdir, n, fileSizes)
		} else {
			fileSizes <- entry.Size()
		}
	}
}

//!-walkDir

//!+sema
// sema is a counting semaphore for limiting concurrency in dirents.
var sema = make(chan struct{}, 20)

// dirents returns the entries of directory dir.
func dirents(dir string) []os.FileInfo {
	sema <- struct{}{}        // acquire token
	defer func() { <-sema }() // release token
	// ...
	//!-sema

	entries, err := ioutil.ReadDir(dir)
	if err != nil {
		//fmt.Fprintf(os.Stderr, "du: %v\n", err)
		return nil
	}
	return entries
}
```

```bash
$ ./ex8.9 -v /etc /usr /bin /var
2018/01/03 20:36:21.368337 /bin: 36 files  0.0 GB
2018/01/03 20:36:21.370797 /etc: 308 files  0.0 GB
2018/01/03 20:36:21.483464 /var: 2270 files  6.1 GB
2018/01/03 20:36:21.483846 /usr: 4322 files  1.9 GB
2018/01/03 20:36:21.571401 /var: 2449 files  6.3 GB
2018/01/03 20:36:21.599092 /usr: 9322 files  2.1 GB
2018/01/03 20:36:21.678850 /usr: 18130 files  2.6 GB
2018/01/03 20:36:21.782177 /usr: 47807 files  3.2 GB
2018/01/03 20:36:21.872339 /usr: 65859 files  3.7 GB
2018/01/03 20:36:21.920744 /usr: 89008 files  3.9 GB
```

### Ex 8.12

Make the broadcaster announce the current set of clients to each new arrival. This requires that the clients set and the entering and leaving channels record the client name too.

### Ex 8.13

Make the chat server disconnect idle clients, such as those that have sent no messages in the last ﬁve minutes. Hint: calling conn.Close() in another goroutine unblocks active Read calls such as the one done by input.Scan().

### Ex 8.14

Change the chat server’s network protocol so that each client provides its name on entering. Use that name instead of the network address when preﬁxing each message with its sender’s identity.

只需要添加自定义协议，在压包解包的时候按协议进行就行，待做。。。

### Ex 8.15

Failure of any client program to read data in a timely manner ultimately causes all clients to get stuck. Modify the broadcaster to skip a message rather than wait if a client writer is not ready to accept it. Alternatively, add buffering to each client’s outgoing message channel so that most messages are not dropped; the broadcaster should use a non-blocking send to this channel.

将broadcaster中向各个客户端发送消息的部分写入go routine，并将客户端发送通道定义为缓存通道

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// See page 254.
//!+

// Chat is a server that lets clients chat with each other.
package main

import (
	"bufio"
	"fmt"
	"log"
	"net"
	"time"
)

//!+broadcaster
type client chan<- string // an outgoing message channel

var (
	entering = make(chan client)
	leaving  = make(chan client)
	messages = make(chan string) // all incoming client messages
)

func broadcaster() {
	clients := make(map[client]bool) // all connected clients
	for {
		select {
		case msg := <-messages:
			// Broadcast incoming message to all
			// clients' outgoing message channels.
			for cli := range clients {
				cli <- msg
			}

		case cli := <-entering:
			clients[cli] = true

		case cli := <-leaving:
			delete(clients, cli)
			close(cli)
		}
	}
}

//!-broadcaster

//!+handleConn
func handleConn(conn net.Conn) {
	inittime := time.Now()
	itimestr := "[" + inittime.Format("2006-01-02 15:04:05") + "]"
	ch := make(chan string) // outgoing client messages
	go clientWriter(conn, ch)
	lastmsgtime := make(chan time.Time)
	who := conn.RemoteAddr().String()
	go func(t time.Time) {
		for {
			select {
			case t = <-lastmsgtime:
			default:
			}
			if time.Now().After(t.Add(20 * time.Second)) {
				ch <- fmt.Sprintf("[%s] no msg in last 20 sec since %s, closing connect!", time.Now().Format("2006-01-02 15:04:05"), t.Format("2006-01-02 15:04:05"))
				time.Sleep(1 * time.Second)
				conn.Close()
				//leaving <- ch
				//messages <- who + " has left"
				break
			}
		}
	}(inittime)
	ch <- itimestr + " " + "You are " + who
	messages <- itimestr + " " + who + " has arrived"
	entering <- ch

	input := bufio.NewScanner(conn)
	for input.Scan() {
		messages <- "[" + time.Now().Format("2006-01-02 15:04:05") + "]" + " " + who + ": " + input.Text()
		lastmsgtime <- time.Now()
	}
	// NOTE: ignoring potential errors from input.Err()

	leaving <- ch
	messages <- "[" + time.Now().Format("2006-01-02 15:04:05") + "]" + " " + who + " has left"
	conn.Close()
}

func clientWriter(conn net.Conn, ch <-chan string) {
	for msg := range ch {
		fmt.Fprintln(conn, msg) // NOTE: ignoring network errors
	}
}

//!-handleConn

//!+main
func main() {
	listener, err := net.Listen("tcp", "localhost:8000")
	if err != nil {
		log.Fatal(err)
	}

	go broadcaster()
	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Print(err)
			continue
		}
		go handleConn(conn)
	}
}
```

```bash
$ ./netcat1
[2018-01-04 01:09:14] You are 127.0.0.1:55627
[2018-01-04 01:09:18] 127.0.0.1:55630 has arrived
[2018-01-04 01:09:20] 127.0.0.1:55630: sdf
[2018-01-04 01:09:24] 127.0.0.1:55635 has arrived
[2018-01-04 01:09:28] 127.0.0.1:55635: ththth
[2018-01-04 01:09:34] no msg in last 20 sec since 2018-01-04 01:09:14, closing connect!
2018/01/04 01:09:35 done
$ ./netcat2
[2018-01-04 01:09:18] You are 127.0.0.1:55630
sdf
[2018-01-04 01:09:20] 127.0.0.1:55630: sdf
[2018-01-04 01:09:24] 127.0.0.1:55635 has arrived
[2018-01-04 01:09:28] 127.0.0.1:55635: ththth
[2018-01-04 01:09:35] 127.0.0.1:55627 has left
[2018-01-04 01:09:40] no msg in last 20 sec since 2018-01-04 01:09:20, closing connect!
2018/01/04 01:09:41 done
$ ./netcat3
[2018-01-04 01:09:24] You are 127.0.0.1:55635
ththth
[2018-01-04 01:09:28] 127.0.0.1:55635: ththth
[2018-01-04 01:09:35] 127.0.0.1:55627 has left
[2018-01-04 01:09:41] 127.0.0.1:55630 has left
[2018-01-04 01:09:48] no msg in last 20 sec since 2018-01-04 01:09:28, closing connect!
2018/01/04 01:09:49 done
```

