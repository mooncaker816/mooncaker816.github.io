---
title: The Go Programming Language Ex（2）
date: 2017-12-21 17:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
---

### Ex 2.1

Add types, constants, and functions to tempconv for processing temperatures in the Kelvin scale, where zero Kelvin is −273.15°C and a difference of 1K has the same magnitude as 1°C.

<!--more-->

```go
package tempconv

import "fmt"

type Celsius float64
type Fahrenheit float64
type Kelvin float64

const (
	AbsoluteZeroC Celsius = -273.15
	FreezingC     Celsius = 0
	BoilingC      Celsius = 100
)

func (c Celsius) String() string    { return fmt.Sprintf("%g°C", c) }
func (f Fahrenheit) String() string { return fmt.Sprintf("%g°F", f) }
func (k Kelvin) String() string     { return fmt.Sprintf("%g°K", k) }

```

```go
package tempconv

// CToF converts a Celsius temperature to Fahrenheit.
func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

// FToC converts a Fahrenheit temperature to Celsius.
func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }

//CToK converts a Celsius temperature to Kelvin.
func CToK(c Celsius) Kelvin { return Kelvin(c + 273.15) }

//KToC converts a Kelvin temperature to Celsius.
func KToC(k Kelvin) Celsius { return Celsius(k - 273.15) }

//FToK converts a Fahrenheit temperature to Kelvin.
func FToK(f Fahrenheit) Kelvin { return Kelvin((f-32)*5/9 + 273.15) }

// KToF converts a Kelvin temperature to Fahrenheit.
func KToF(k Kelvin) Fahrenheit { return Fahrenheit((k-273.15)*9/5 + 32) }

```

```go
package main

import (
	"fmt"
	"The_Go_Programming_Language_Exercises/CH2/ex2.1/tempconv"
)

func main() {
	fmt.Println(tempconv.CToK(tempconv.BoilingC))
	k1 := tempconv.Kelvin(273)
	fmt.Println(tempconv.KToC(k1))
	fmt.Println(tempconv.KToF(k1))
}
```

```bash
$ go run main.go
373.15°K
-0.14999999999997726°C
31.73000000000004°F
```

### Ex 2.2

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"strconv"
	"strings"

	"The_Go_Programming_Language_Exercises/CH2/ex2.2/unitconv"
)

func main() {
	var s []string
	if len(os.Args) > 1 {
		s = os.Args[1:]
	} else {
		input, err := os.Open("a")
		if err != nil {
			fmt.Println("happend a error when opening", err)
		}
		defer input.Close()
		buf := bufio.NewReader(input)
		for {
			line, err := buf.ReadString('\n')
			line = strings.TrimSpace(line)
			s = append(s, line)
			if err != nil {
				if err == io.EOF {
					break
				}
				fmt.Fprintf(os.Stderr, "reading error %v\n", err)
				os.Exit(1)
			}
		}
	}
	for _, arg := range s {
		t, err := strconv.ParseFloat(arg, 64)
		if err != nil {
			fmt.Fprintf(os.Stderr, "invalid input: %v\n", err)
			os.Exit(1)
		}
		f := unitconv.Fahrenheit(t)
		c := unitconv.Celsius(t)
		p := unitconv.Pounds(t)
		kg := unitconv.Kilograms(t)
		fe := unitconv.Feet(t)
		m := unitconv.Meter(t)
		fmt.Printf("%s = %s, %s = %s\n", f, unitconv.FToC(f), c, unitconv.CToF(c))
		fmt.Printf("%s = %s, %s = %s\n", p, unitconv.PToKg(p), kg, unitconv.KgToP(kg))
		fmt.Printf("%s = %s, %s = %s\n", fe, unitconv.FeToM(fe), m, unitconv.MToFe(m))
	}
}
```

```go
//file a
11
22
33
```

```bash
$ ./ex2.2
11°F = -11.666666666666666°C, 11°C = 51.8°F
11 pounds = 4.9895160700000005 kilograms, 11 kilograms = 24.2508488403368 pounds
11 feets = 3.3528000000000002 meters, 11 meters = 36.08913 feets
22°F = -5.555555555555555°C, 22°C = 71.6°F
22 pounds = 9.979032140000001 kilograms, 22 kilograms = 48.5016976806736 pounds
22 feets = 6.7056000000000004 meters, 22 meters = 72.17826 feets
33°F = 0.5555555555555556°C, 33°C = 91.4°F
33 pounds = 14.968548210000002 kilograms, 33 kilograms = 72.7525465210104 pounds
33 feets = 10.0584 meters, 33 meters = 108.26738999999999 feets

$ ./ex2.2 11 22 33 44.5
11°F = -11.666666666666666°C, 11°C = 51.8°F
11 pounds = 4.9895160700000005 kilograms, 11 kilograms = 24.2508488403368 pounds
11 feets = 3.3528000000000002 meters, 11 meters = 36.08913 feets
22°F = -5.555555555555555°C, 22°C = 71.6°F
22 pounds = 9.979032140000001 kilograms, 22 kilograms = 48.5016976806736 pounds
22 feets = 6.7056000000000004 meters, 22 meters = 72.17826 feets
33°F = 0.5555555555555556°C, 33°C = 91.4°F
33 pounds = 14.968548210000002 kilograms, 33 kilograms = 72.7525465210104 pounds
33 feets = 10.0584 meters, 33 meters = 108.26738999999999 feets
44.5°F = 6.944444444444445°C, 44.5°C = 112.1°F
44.5 pounds = 20.184860465 kilograms, 44.5 kilograms = 98.10570667227161 pounds
44.5 feets = 13.563600000000001 meters, 44.5 meters = 145.996935 feets
```

### Ex 2.3 2.4 2.5

Rewrite PopCount to use a loop instead of a single expression. Compare the performance of the two versions. (Section 11.4 shows how to compare the performance of different implementations systematically.)

Write a version of PopCount that counts bits by shifting its argument through 64 bit positions, testing the rightmost bit each time. Compare its performance to the tablelookup version.

The expression x&(x-1) clears the rightmost non-zero bit of x. Write a version of PopCount that counts bits by using this fact, and assess its performance.

```go
package popcount_test

import (
	"fmt"
	"io/ioutil"
	"testing"
)

func init() {
	for i := range pc {
		pc[i] = pc[i/2] + byte(i&1)
	}
	for i := 0; i < 256; i++ {
		n[i] = i
	}
}

var pc [256]byte
var n = make([]int, 256)

//通过v&(v-1)，将v对应的二进制中最右的非零位置零，直到v=0为止，此时置零的次数即为popcount的值
func BenchmarkPopCount1(b *testing.B) {
	for i := 0; i < b.N; i++ {
		for _, v := range n {
			var count int
			for v != 0 {
				count++
				v &= v - 1
			}
			fmt.Fprintln(ioutil.Discard, count)
			//fmt.Fprintln(os.Stdout, count)
		}
	}
}

//把64位平均分为8份，每份对应一个字节，读出该字节的数值，去预先准备好的单字节(8位)所能存储的所有数值对应的popcount的表中查询出每份的popcount，再相加8份的popcount
func BenchmarkPopCount2(b *testing.B) {
	for i := 0; i < b.N; i++ {
		for _, x := range n {
			count := int(pc[byte(x>>(0*8))] +
				pc[byte(x>>(1*8))] +
				pc[byte(x>>(2*8))] +
				pc[byte(x>>(3*8))] +
				pc[byte(x>>(4*8))] +
				pc[byte(x>>(5*8))] +
				pc[byte(x>>(6*8))] +
				pc[byte(x>>(7*8))])
			fmt.Fprintln(ioutil.Discard, count)
			//fmt.Fprintln(os.Stdout, count)
		}
	}
}

//从最右开始一位一位检查是否为1
func BenchmarkPopCount3(b *testing.B) {
	for i := 0; i < b.N; i++ {
		for _, v := range n {
			var count int
			for ; v != 0; v >>= 1 {
				count += v & 1
			}
			fmt.Fprintln(ioutil.Discard, count)
			//fmt.Fprintln(os.Stdout, count)
		}
	}
}
```

```bash
$ go test -v -run="none" -bench=. -benchtime="3s" -benchmem
BenchmarkPopCount1-4   	  200000	     23154 ns/op	    2048 B/op	     256 allocs/op
BenchmarkPopCount2-4   	  200000	     21941 ns/op	    2048 B/op	     256 allocs/op
BenchmarkPopCount3-4   	  200000	     24450 ns/op	    2048 B/op	     256 allocs/op
PASS
ok  	The_Go_Programming_Language_Exercises/CH2/ex2.3	14.646s
```

**n&(n-1) ：将n的二进制最右边的非零位置零**

平均来说，此3种方法中查表最快，n&(n-1)居中，一位一位数最慢