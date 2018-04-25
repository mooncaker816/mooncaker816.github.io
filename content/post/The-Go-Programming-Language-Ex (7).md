---
title: The Go Programming Language Ex（7）
date: 2017-12-29 00:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go Method
- Go BitMap
---

### Ex 6.1

Implement these additional methods:

func (*IntSet) Len() int // return the number of elements 

func (*IntSet) Remove(x int) //remove x from the set 

func (*IntSet) Clear() // remove all elements from the set 

func (*IntSet) Copy() *IntSet // return a copy of the set

<!--more-->

### Ex 6.2 

Deﬁne a variadic (*IntSet).AddAll(...int) method that allows a list of values to be added, such as s.AddAll(1, 2, 3).

### Ex 6.3

(*IntSet).UnionWith computes the union of two sets using |, the word-parallel bitwise OR operator. Implement methods for IntersectWith, DifferenceWith, and SymmetricDifference for the corresponding set operations. (The symmetric difference of two sets contains the elements present in one set or the other but not both.)

### Ex 6.4

Add a method Elems that returns a slice containing the elements of the set, suitable for iterating over with a range loop.

### Ex 6.5

The type of each word used by IntSet is uint64, but 64-bit arithmetic may be inefﬁcient on a 32-bit platform. Modify the program to use the uint type, which is the most efﬁcient unsigned integer type for the platform. Instead of dividing by 64, deﬁne a constant holding the effective size of uint in bits, 32 or 64. You can use the perhaps too-clever expression 32 << (^uint(0) >> 63) for this purpose.

```go
package main

import (
	"bytes"
	"fmt"
)

const wordlen = 32 << (^uint(0) >> 63) //当前计算机的位数

type IntSet struct {
	words []uint
}

func (s *IntSet) Add(x int) {
	word, bit := x/wordlen, uint(x%wordlen)
	for word >= len(s.words) {
		s.words = append(s.words, 0)
	}
	s.words[word] |= 1 << bit
}

func (s *IntSet) Has(x int) bool {
	word, bit := x/wordlen, uint(x%wordlen)
	return word < len(s.words) && s.words[word]&(1<<bit) != 0
}

func (s *IntSet) UnionWith(t *IntSet) {
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] |= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

func (s *IntSet) String() string {
	var buf bytes.Buffer
	buf.WriteByte('{')
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < wordlen; j++ {
			if word&(1<<uint(j)) != 0 {
				if buf.Len() > len("{") {
					buf.WriteByte(' ')
				}
				fmt.Fprintf(&buf, "%d", wordlen*i+j)
			}
		}
	}
	buf.WriteByte('}')
	return buf.String()
}

func (s *IntSet) Len() int {
	var count int
	for _, n := range s.words {
		for n != 0 {
			count++
			n = n & (n - 1)
		}
	}
	return count
}

func (s *IntSet) Remove(x int) {
	word, bit := x/wordlen, uint(x%wordlen)
	if word < len(s.words) {
		s.words[word] &= ^(1 << bit)
	}
}

func (s *IntSet) Clear() {
	s.words = []uint{0}
}
func (s *IntSet) Copy() *IntSet {
	ns := IntSet{make([]uint, len(s.words))}
	copy(ns.words, s.words)
	return &ns
}

func (s *IntSet) AddAll(x ...int) {
	for _, v := range x {
		s.Add(v)
	}
}

func (s *IntSet) IntersectWith(t *IntSet) *IntSet {
	tmp := make([]uint, 0)
	for i, tword := range t.words {
		if i < len(s.words) {
			tmp = append(tmp, s.words[i]&tword)
		}
	}
	return &IntSet{tmp}
}
func (s *IntSet) SymmetricDifference(t *IntSet) *IntSet {
	tmp := make([]uint, 0)
	if len(t.words) > len(s.words) {
		for i, tword := range t.words {
			if i < len(s.words) {
				tmp = append(tmp, s.words[i]^tword)
			} else {
				tmp = append(tmp, tword)
			}
		}
	} else {
		for i, sword := range s.words {
			if i < len(t.words) {
				tmp = append(tmp, t.words[i]^sword)
			} else {
				tmp = append(tmp, sword)
			}
		}
	}
	return &IntSet{tmp}
}

func (s *IntSet) DifferenceWith(t *IntSet) *IntSet {
	tmp := make([]uint, 0)
	for i, sword := range s.words {
		if i < len(t.words) {
			tmp = append(tmp, sword&^t.words[i])
		} else {
			tmp = append(tmp, sword)
		}
	}
	return &IntSet{tmp}
}
func (s *IntSet) Elems() []int {
	tmp := make([]int, 0)
	for i, word := range s.words {
		if word == 0 {
			continue
		}
		for j := 0; j < wordlen; j++ {
			if word&(1<<uint(j)) != 0 {
				tmp = append(tmp, wordlen*i+j)
			}
		}
	}
	return tmp
}

func main() {
	var set1, set2, set3 IntSet
	set1.Add(1)
	set1.Add(22)
	set2.Add(333)
	fmt.Println("set1:", &set1)
	fmt.Println("set1 是否包含1：", set1.Has(1), "set1 是否包含333：", set1.Has(333))
	fmt.Println("before union with {333}: ", &set1)
	fmt.Println("len: ", set1.Len())
	set1.UnionWith(&set2)
	fmt.Println("after union with {333}: ", &set1)
	fmt.Println("len: ", set1.Len())
	set1.Remove(22)
	fmt.Println("after remove 22: ", &set1)
	fmt.Println("len: ", set1.Len())
	set3 = *set1.Copy()
	fmt.Println("copy set1 to set3: ", &set3)
	set1.Clear()
	fmt.Println("after clear set1: ", &set1, &set3)
	set3.AddAll(4444, 55555)
	fmt.Println("after add all set3: ", &set3)
	set2.AddAll(55555, 666666)
	fmt.Println("se2:", &set2)
	fmt.Println("se3:", &set3)
	fmt.Println("set2交set3:", set2.IntersectWith(&set3))
	fmt.Println("set2 DifferenceWith set3:", set2.DifferenceWith(&set3))
	fmt.Println("set3 DifferenceWith set2:", set3.DifferenceWith(&set2))
	fmt.Println("set2 SymmetricDifference set3:", set2.SymmetricDifference(&set3))
	fmt.Println("set3 SymmetricDifference set2:", set3.SymmetricDifference(&set2))
	fmt.Println("elem slice in set2:", set2.Elems())
}
```

```bash
$ go run bitmap.go
set1: {1 22}
set1 是否包含1： true set1 是否包含333： false
before union with {333}:  {1 22}
len:  2
after union with {333}:  {1 22 333}
len:  3
after remove 22:  {1 333}
len:  2
copy set1 to set3:  {1 333}
after clear set1:  {} {1 333}
after add all set3:  {1 333 4444 55555}
se2: {333 55555 666666}
se3: {1 333 4444 55555}
set2交set3: {333 55555}
set2 DifferenceWith set3: {666666}
set3 DifferenceWith set2: {1 4444}
set2 SymmetricDifference set3: {1 4444 666666}
set3 SymmetricDifference set2: {1 4444 666666}
elem slice in set2: [333 55555 666666]
```

