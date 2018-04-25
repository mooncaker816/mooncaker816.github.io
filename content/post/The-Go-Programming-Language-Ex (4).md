---
title: The Go Programming Language Ex（4）
date: 2017-12-23 19:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go slice
---

### Ex 4.1

Write a function that counts the number of bits that are different in two SHA256 hashes. (See PopCount from Section 2.6.2.)

<!--more-->

```go
package main

import (
	"crypto/sha256"
	"fmt"
)

func main() {
	str1 := "abc"
	str2 := "xyz"
	hash1 := sha256.Sum256([]byte(str1))
	hash2 := sha256.Sum256([]byte(str2))
	var dif int
	for i := 0; i < 32; i++ {
		dif += popcount(hash1[i] ^ hash2[i])
	}
	fmt.Printf("there are %d different bits between these 2 hashes\n", dif)
}

func popcount(x byte) int {
	var count int
	for x != 0 {
		count++
		x &= x - 1
	}
	return count
}
```

```bash
$ go run difhash.go
there are 121 different bits between these 2 hashes
```

### Ex 4.2

Write a program that prints the SHA256 hash of its standard input by default but supports a command-line ﬂag to print the SHA384 or SHA512 hash instead.

```go
package main

import (
	"crypto/sha256"
	"crypto/sha512"
	"flag"
	"fmt"
	"log"
)

func main() {
	method := flag.String("sha", "256", "default is SHA256, optional 384,512")
	flag.Parse()
	//去掉flag参数后剩余的输入参数，此处为需要计算hash的字符串
	fmt.Printf("there are %d non-flag input params\n", flag.NArg())
	for i, str := range flag.Args() {
		switch *method {
		case "256":
			c := sha256.Sum256([]byte(str))
			fmt.Printf("#%d:SHA256 of %s is %x\n", i, str, c)
		case "384":
			c := sha512.Sum384([]byte(str))
			fmt.Printf("#%d:SHA384 of %s is %x\n", i, str, c)
		case "512":
			c := sha512.Sum512([]byte(str))
			fmt.Printf("#%d:SHA512 of %s is %x\n", i, str, c)
		default:
			log.Fatal("not support")
		}
	}
}
```

```bash
$ ./ex4.2 -sha 256 a b c abc
there are 4 non-flag input params
#0:SHA256 of a is ca978112ca1bbdcafac231b39a23dc4da786eff8147c4e72b9807785afee48bb
#1:SHA256 of b is 3e23e8160039594a33894f6564e1b1348bbd7a0088d42c4acb73eeaed59c009d
#2:SHA256 of c is 2e7d2c03a9507ae265ecf5b5356885a53393a2029d241394997265a1a25aefc6
#3:SHA256 of abc is ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
$ ./ex4.2 -sha 384 a b c abc
there are 4 non-flag input params
#0:SHA384 of a is 54a59b9f22b0b80880d8427e548b7c23abd873486e1f035dce9cd697e85175033caa88e6d57bc35efae0b5afd3145f31
#1:SHA384 of b is 98a906182cdcfb1eb4eb47117600f68958e2ddd140248b47984f4bde6587b89c8215c3da895a336e94ad1aca39015c40
#2:SHA384 of c is 40f98a05660bf871802ee59964de1945bd731a45cc7f48e4dadd92f34a7eeec089e149ad8c2434f11792e588b740d997
#3:SHA384 of abc is cb00753f45a35e8bb5a03d699ac65007272c32ab0eded1631a8b605a43ff5bed8086072ba1e7cc2358baeca134c825a7
$ ./ex4.2 -sha 512 a b c abc
there are 4 non-flag input params
#0:SHA512 of a is 1f40fc92da241694750979ee6cf582f2d5d7d28e18335de05abc54d0560e0f5302860c652bf08d560252aa5e74210546f369fbbbce8c12cfc7957b2652fe9a75
#1:SHA512 of b is 5267768822ee624d48fce15ec5ca79cbd602cb7f4c2157a516556991f22ef8c7b5ef7b18d1ff41c59370efb0858651d44a936c11b7b144c48fe04df3c6a3e8da
#2:SHA512 of c is acc28db2beb7b42baa1cb0243d401ccb4e3fce44d7b02879a52799aadff541522d8822598b2fa664f9d5156c00c924805d75c3868bd56c2acb81d37e98e35adc
#3:SHA512 of abc is ddaf35a193617abacc417349ae20413112e6fa4e89a97ea20a9eeee64b55d39a2192992a274fc1a836ba3c23a3feebbd454d4423643ce80e2a9ac94fa54ca49f
$ ./ex4.2 -sha 777 a b c abc
there are 4 non-flag input params
2017/12/22 20:53:41 not support
$./ex4.2 a b c abc
there are 4 non-flag input params
#0:SHA256 of a is ca978112ca1bbdcafac231b39a23dc4da786eff8147c4e72b9807785afee48bb
#1:SHA256 of b is 3e23e8160039594a33894f6564e1b1348bbd7a0088d42c4acb73eeaed59c009d
#2:SHA256 of c is 2e7d2c03a9507ae265ecf5b5356885a53393a2029d241394997265a1a25aefc6
#3:SHA256 of abc is ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
```

### Ex 4.3

Rewrite reverse to use an array pointer instead of a slice.

```go
package main

import (
	"fmt"
)

func main() {
	s := [5]byte{1, 2, 3, 4, 5}
	fmt.Println(s)
	reverse(&s)
	fmt.Println(s)
}

func reverse(a *[5]byte) {
	for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
		a[i], a[j] = a[j], a[i]
	}
}
```

```bash
$ go run reverse.go
[1 2 3 4 5]
[5 4 3 2 1]
```

### Ex 4.4

Write a version of rotate that operates in a single pass.

```go
package main

import (
	"fmt"
	"log"
)

func main() {
	s := []int{0, 1, 2, 3, 4, 5, 6, 7, 8}
	rotate(&s, 3)
	fmt.Println(s)
}

func rotate(s *[]int, n int) {
	if n > len(*s) {
		log.Fatal("invalid rotate num")
	}
	var ls []int
	ls = append(ls, (*s)[n:]...)
	ls = append(ls, (*s)[:n]...)
	*s = ls
}
```

```bash
$ go run rotate.go
[3 4 5 6 7 8 0 1 2]
```

### Ex 4.5

Write an in-place function to eliminate adjacent duplicates in a []string slice.

```go
package main

import (
	"fmt"
)

func main() {
	s1 := []string{"a", "b", "b", "c", "d", "d"}
	s11 := s1
	dup(&s1)
	dup1(&s11)
	fmt.Println(s1, s11)
	s2 := []string{"a", "b", "c"}
	s21 := s2
	dup(&s2)
	dup(&s21)
	fmt.Println(s2, s21)
}

func dup(s *[]string) {
	var ls []string
	ls = append(ls, (*s)[0])
	for i := 0; i < len(*s)-1; i++ {
		if (*s)[i] != (*s)[i+1] {
			ls = append(ls, (*s)[i+1])
		}
	}
	*s = ls
}

func dup1(s *[]string) {
	j := 0
	var temp string
	for i := range *s {
		(*s)[j] = (*s)[i]
		j++
		if i > 0 && temp == (*s)[i] {
			j--
		}
		temp = (*s)[i]
	}
	*s = (*s)[:j]
}
```

```bash
$ go run dup.go
[a b c d] [a b c d]
[a b c] [a b c]
```

### Ex 4.6

Write an in-place function that squashes each run of adjacent Unicode spac (see unicode.IsSpace) in a UTF-8-encoded []byte slice into a single ASCII space.

```go
package main

import (
	"fmt"
	"unicode"
	"unicode/utf8"
)

func main() {
	str := "\t世界a \na\r\t\vb   "
	s1 := []byte(str)
	s2 := []byte(str)
	//s1 := []byte{'a', 'b', ' ', '	', '\n', 'c', ' '}
	fmt.Printf("before:%d %[1]v %[1]s\n", s1)
	utf8space(&s1)
	fmt.Printf("after:%d %[1]v %[1]s\n", s1)
	//s2 := []byte{'1', '2', ' ', '	', '\n', '3', ' ', '\t'}
	fmt.Printf("before:%d %[1]v %[1]s\n", s2)
	utf8space2(&s2)
	fmt.Printf("after:%d %[1]v %[1]s\n", s2)
}

func utf8space(s *[]byte) {
	var ls []byte
	runes := []rune(string(*s))
	spaceInd := false
	for _, r := range runes {
		if !unicode.IsSpace(r) {
			spaceInd = false
			ls = append(ls, []byte(string(r))...)
		} else if unicode.IsSpace(r) && !spaceInd {
			spaceInd = true
			ls = append(ls, byte(32))
		}
	}
	*s = ls
}

func utf8space2(s *[]byte) {
	str := string(*s)
	spaceInd := false
	j := 0
	for i, r := range str {
		bytelen := utf8.RuneLen(r)
		if !unicode.IsSpace(r) {
			spaceInd = false
			copy((*s)[j:], (*s)[i:i+bytelen])
			j += bytelen
		} else if unicode.IsSpace(r) && !spaceInd {
			spaceInd = true
			bytelen = 1
			copy((*s)[j:], " ")
			j += bytelen
		}
	}
	*s = (*s)[:j]
}
```

```bash
$ go run utf8space.go
before:[9 228 184 150 231 149 140 97 32 10 97 13 9 11 98 32 32 32] [9 228 184 150 231 149 140 97 32 10 97 13 9 11 98 32 32 32] 	世界a
a
        b
after:[32 228 184 150 231 149 140 97 32 97 32 98 32] [32 228 184 150 231 149 140 97 32 97 32 98 32]  世界a a b
before:[9 228 184 150 231 149 140 97 32 10 97 13 9 11 98 32 32 32] [9 228 184 150 231 149 140 97 32 10 97 13 9 11 98 32 32 32] 	世界a
a
        b
after:[32 228 184 150 231 149 140 97 32 97 32 98 32] [32 228 184 150 231 149 140 97 32 97 32 98 32]  世界a a b
```

**for range遍历<u>`字符串`</u>的时候，i为当前字符开始的位置，v为当前字符。换句话说，字符串包含多字节utf8字符时，i不连续，因为遍历的单位不是byte，而是rune（unicode）。**

![range loop for string](http://oumnldfwl.bkt.clouddn.com/range%20loop%20for%20string.png)

### Ex 4.7 

Modify reverse to reverse the characters of a []byte slice that represents a UTF-8-encoded string, in place. Can you do it without allocating new memory?

```go
package main

import (
	"fmt"
	"unicode"
)

func main() {
	str := "\U00000080world,世界,hello,你好"
	s1 := []byte(str) //每个元素根据utf8的格式，确定utf8表达字符的字节数，或为单字节或为多字节
	s2 := []rune(str) //一个元素就是一个unicode
	fmt.Printf("% x \n % x\n", s1, s2)
	fmt.Println(Reverse(str))
	reverse(s2)
	fmt.Println(string(s2))
}

// Reverse reverses the input while respecting UTF8 encoding and combined characters
func Reverse(text string) string {
	textRunes := []rune(text)
	textRunesLength := len(textRunes)
	if textRunesLength <= 1 {
		return text
	}

	i, j := 0, 0
	for i < textRunesLength && j < textRunesLength {
		j = i + 1
		for j < textRunesLength && isMark(textRunes[j]) {
			j++
		}

		if isMark(textRunes[j-1]) {
			// Reverses Combined Characters
			reverse(textRunes[i:j])
		}

		i = j
	}

	// Reverses the entire array
	reverse(textRunes)

	return string(textRunes)
}

func reverse(runes []rune) {
	for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
		runes[i], runes[j] = runes[j], runes[i]
	}
}

// isMark determines whether the rune is a marker
func isMark(r rune) bool {
	return unicode.Is(unicode.Mn, r) || unicode.Is(unicode.Me, r) || unicode.Is(unicode.Mc, r)
}
```

```bash
$ go run utf8reverse.go
c2 80 77 6f 72 6c 64 2c e4 b8 96 e7 95 8c 2c 68 65 6c 6c 6f 2c e4 bd a0 e5 a5 bd
 [ 80  77  6f  72  6c  64  2c  4e16  754c  2c  68  65  6c  6c  6f  2c  4f60  597d]
好你,olleh,界世,dlrow
好你,olleh,界世,dlrow
```

Reverse：确保了组合字符（多个unicode组合成一个字符）翻转的正确性

reverse：不考虑组合字符，翻转正确