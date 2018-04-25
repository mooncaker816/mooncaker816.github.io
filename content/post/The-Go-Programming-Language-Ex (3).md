---
title: The Go Programming Language Ex（3）
date: 2017-12-22 17:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
---

### Ex 3.10

Write a non-recursive version of comma, using bytes.Buffer instead of str concatenation.

<!--more-->

```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"os"
)

func main() {
	for _, v := range os.Args[1:] {
		s := []byte(v)
		var buf bytes.Buffer
		for i, j := len(s)-1, 0; i >= 0; i-- {
			err := buf.WriteByte(s[i])
			if err != nil {
				log.Fatal("write buf failed")
			}
			if j++; j != len(s) && j%3 == 0 {
				err = buf.WriteByte(',')
			}
		}
		fmt.Println(Reverse(buf.String()))
	}

}

func Reverse(s string) string {
	r := []rune(s)
	for i, j := len(r)-1, 0; i > j; i, j = i-1, j+1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

```bash
$ ./ex3.10 12345678 829356934592162 23 7777 333
12,345,678
829,356,934,592,162
23
7,777
333
```

### Ex 3.11

Enhance comma so that it deals correctly with ﬂoating-point numbers and an optional sign.

```go
package main

import (
	"bytes"
	"fmt"
	"log"
	"os"
)

func main() {
	for _, v := range os.Args[1:] {
		var lbuf, rbuf bytes.Buffer
		var lstr, rstr, sign string
		if v[0] == '+' || v[0] == '-' {
			sign = string(v[0])
			v = v[1:]
		}
		chars := bytes.SplitAfterN([]byte(v), []byte("."), 2)
		for ix, pc := range chars {
			if ix == 0 {
				if pc[len(pc)-1] == '.' {
					pc = pc[:len(pc)-1]
					err := lbuf.WriteByte('.')
					if err != nil {
						log.Fatal("write '.' in buf failed")
					}
				}
				if len(pc) > 0 {
					for i, j := len(pc)-1, 0; i >= 0; i-- {
						err := lbuf.WriteByte(pc[i])
						if err != nil {
							log.Fatal("write int part in buf failed")
						}
						if j++; j != len(pc) && j%3 == 0 {
							err = lbuf.WriteByte(',')
						}
					}
					lstr = Reverse(lbuf.String())
				}
			}
			if ix == 1 {
				if len(pc) > 0 {
					for i, j := 0, 0; i < len(pc); i++ {
						err := rbuf.WriteByte(pc[i])
						if err != nil {
							log.Fatal("write float part in buf failed")
						}
						if j++; j != len(pc) && j%3 == 0 {
							err = rbuf.WriteByte(',')
						}
					}
					rstr = rbuf.String()
				} else {
					rstr = "0"
				}
			}
		}
		fmt.Println(sign + lstr + rstr)
	}
}

func Reverse(s string) string {
	r := []rune(s)
	for i, j := len(r)-1, 0; i > j; i, j = i-1, j+1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```

```bash
$ go run comma.go +123456.4563 -2342335352234.234 234235235211 +1243535353434 24533. 454545.345354456
+123,456.456,3
-2,342,335,352,234.234
234,235,235,211
+1,243,535,353,434
24,533.0
454,545.345,354,456
```

### Ex 3.12

Write a function that reports whether two strings are anagrams of each other, that is, they contain the same letters in a different order.

```go
package main

import (
	"fmt"
	"os"
	"sort"
)

func main() {
	s1 := []rune(os.Args[1])
	s2 := []rune(os.Args[2])
	var ss1 = make([]string, len(s1))
	var ss2 = make([]string, len(s2))
	for i, v := range s1 {
		ss1[i] = string(v)
	}
	for i, v := range s2 {
		ss2[i] = string(v)
	}
	sort.Strings(ss1)
	sort.Strings(ss2)
	if len(ss1) != len(ss2) {
		fmt.Println("they are NOT anagrams strings")
		return
	}
	for i := 0; i < len(ss1); i++ {
		if ss1[i] != ss2[i] {
			fmt.Println("they are NOT anagrams strings")
			return
		}
	}
	fmt.Println("they are anagrams strings")
}
```

```bash
$ go run anagrams.go abcdefg acdefgb
they are anagrams strings
$ go run anagrams.go aabbccdd dcbadcba
they are anagrams strings
$ go run anagrams.go abcde dbca
they are NOT anagrams strings
$ go run anagrams.go abcde dbcaf
they are NOT anagrams strings
```

### Ex 3.13

Write const declarations for KB, MB, up through YB as compactly as you can.

```go
package main

import (
	"fmt"
)

const (
	_   = int64(1) << (10 * iota)
	KiB // 1024        2e1
	MiB // 1048576     2e20
	GiB // 1073741824  2e30
	TiB // 1099511627776
	PiB // 1125899906842624
	EiB // 1152921504606846976
	//ZiB // 1180591620717411303424  overflow
	//YiB // 1208925819614629174706176 overflow
	KB = int64(1000)
	MB = 1000 * KB
	GB = 1000 * MB
	TB = 1000 * GB
	PB = 1000 * TB
	EB = 1000 * PB
)

func main() {
	fmt.Println(KiB, MiB, GiB, TiB, PiB, EiB)
	fmt.Println(KB, MB, GB, TB, PB, EB)
}
```

```bash
$ go run const.go
1024 1048576 1073741824 1099511627776 1125899906842624 1152921504606846976
1000 1000000 1000000000 1000000000000 1000000000000000 1000000000000000000
```

