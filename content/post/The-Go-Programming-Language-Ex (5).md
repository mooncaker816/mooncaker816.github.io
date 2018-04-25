---
title: The Go Programming Language Ex（5）
date: 2017-12-24 19:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go map
- Go template
- Go JSON
---

### Ex 4.8

Modify charcount to count letters, digits, and so on in their Unicode categories, using functions like unicode.IsLetter.

<!--more-->

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"unicode"
)

func main() {
	counts := make(map[string]int)                       // counts of Unicode characters
	invalid, letter, digit, mark, other := 0, 0, 0, 0, 0 // count of invalid UTF-8 characters

	in := bufio.NewReader(os.Stdin)
	for {
		r, n, err := in.ReadRune() // returns rune, nbytes, error
		if err == io.EOF {
			break
		}
		if err != nil {
			fmt.Fprintf(os.Stderr, "charcount: %v\n", err)
			os.Exit(1)
		}
		if r == unicode.ReplacementChar && n == 1 {
			invalid++
			continue
		} else if unicode.IsLetter(r) {
			letter++
		} else if unicode.IsDigit(r) {
			digit++
		} else if unicode.IsMark(r) {
			mark++
		} else {
			other++
			fmt.Println(r)
		}
	}
	counts["invalid"] = invalid
	counts["letter"] = letter
	counts["digit"] = digit
	counts["mark"] = mark
	counts["other"] = other

	for c, n := range counts {
		fmt.Printf("%s\t:%d\n", c, n)
	}
}
```

```bash
./ex4.8
sdfwwer 1434jipi三是奇偶i问
32
10
invalid	:0
letter	:17
digit	:4
mark	:0
other	:2
```

- **显然汉字也被认为是letter**
- **终端模拟EOF 为 ctrl + d**

### Ex 4.9

Write a program wordfreq to report the frequency of each word in an input text ﬁle. Call input.Split(bufio.ScanWords) before the ﬁrst call to Scan to break the input into words instead of lines.

```go
package main

import (
	"bufio"
	"fmt"
	"strings"
)

func main() {
	words := make(map[string]int)
	input := "foo  bar   baz foo 我们 foo 我们"
	scanner := bufio.NewScanner(strings.NewReader(input))
	scanner.Split(bufio.ScanWords)
	for scanner.Scan() {
		//fmt.Println(scanner.Text())
		words[scanner.Text()]++
	}
	for k, v := range words {
		fmt.Printf("%s : %d\n", k, v)
	}
}
```

```bash
$ go run wordfreq.go
baz : 1
我们 : 2
foo : 3
bar : 1
```

### Ex4.13

The JSON-based web service of the Open Movie Database lets you s https://omdbapi.com/ for a movie by name and download its poster image. Write a tool poster that downloads the poster image for the movie named on the command line.

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"os"
)

const OmdbAPI = "http://www.omdbapi.com/?i=tt3896198&apikey=7b94e6bd&t="

type Movie struct {
	Title    string
	Year     string
	Poster   string
	Response string
	Errormsg string `json:"Error,omitempty"`
}

func main() {
	for i := 1; i < len(os.Args); i++ {
		url := OmdbAPI + os.Args[i]
		resp, err := http.Get(url)
		if err != nil {
			fmt.Println("http get error ", err)
			return
		}
		defer resp.Body.Close()
		var result Movie
		err = json.NewDecoder(resp.Body).Decode(&result)
		if err != nil {
			fmt.Println("decode failed")
			return
		}
		fmt.Printf("%#v\n", result)
		fmt.Printf("%v\n", result)
		if result.Response == "false" {
			fmt.Println(result.Errormsg)
		} else {
			fmt.Println("starting download poster")
			result.Poster = "https://gss1.bdstatic.com/9vo3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike92%2C5%2C5%2C92%2C30/sign=5a60b1a4cf8065386fe7ac41f6b4ca21/fd039245d688d43f4ee7a518781ed21b0ff43b89.jpg"
			posterres, err := http.Get(result.Poster)
			if err != nil {
				fmt.Println("get poster url error, download failed", err)
				return
			} else {
				filename := result.Title + ".jpg"
				f, err := os.Create(filename)
				if err != nil {
					fmt.Println("create file error")
					return
				}
				io.Copy(f, posterres.Body)
			}
		}
	}
}
```

```bash
$ ./ex4.13 triangle triangleeeee
main.Movie{Title:"Triangle", Year:"2009", Poster:"https://images-na.ssl-images-amazon.com/images/M/MV5BY2VlODI5ZmMtZDExYS00OWI4LWJiMWItZWZkZWRkNzlmZWI2XkEyXkFqcGdeQXVyMTQxNzMzNDI@._V1_SX300.jpg", Response:"True", Errormsg:""}
{Triangle 2009 https://images-na.ssl-images-amazon.com/images/M/MV5BY2VlODI5ZmMtZDExYS00OWI4LWJiMWItZWZkZWRkNzlmZWI2XkEyXkFqcGdeQXVyMTQxNzMzNDI@._V1_SX300.jpg True }
starting download poster
download finished
main.Movie{Title:"", Year:"", Poster:"", Response:"False", Errormsg:"Movie not found!"}
{   False Movie not found!}
Movie not found!
```

![Triangle.jpg](http://oumnldfwl.bkt.clouddn.com/ex4.13.png)

### Ex 4.14

Create a web server that queries GitHub once and then allows navigation of the list of bug reports, milestones, and users.

```go
package main

import (
	"encoding/json"
	"fmt"
	temp2 "html/template"
	"log"
	"net/http"
	"net/url"
	"os"
	"strings"
	temp1 "text/template"
	"time"
)

const IssuesURL = "https://api.github.com/search/issues"

//!+template
const templ1 = `{{.TotalCount}} issues:
{{range .Items}}----------------------------------------
Number: {{.Number}}
User:   {{.User.Login}}
Title:  {{.Title | printf "%.64s"}}
Age:    {{.CreatedAt | daysAgo}} days
Milestone: {{.Milestone.Title}}
{{end}}`
const templ2 = `
<h1>{{.TotalCount}} issues</h1>
<table>
<tr style='text-align: left'>
  <th>#</th>
  <th>State</th>
  <th>User</th>
  <th>Title</th>
  <th>Milestone</th>
</tr>
{{range .Items}}
<tr>
  <td><a href='{{.HTMLURL}}'>{{.Number}}</a></td>
  <td>{{.State}}</td>
  <td><a href='{{.User.HTMLURL}}'>{{.User.Login}}</a></td>
  <td><a href='{{.HTMLURL}}'>{{.Title}}</a></td>
  <td><a href='{{.Milestone.HTMLURL}}'>{{.Milestone.Title}}</a></td>
</tr>
{{end}}
</table>
`

//!-template

type IssuesSearchResult struct {
	TotalCount int `json:"total_count"`
	Items      []*Issue
}

type Issue struct {
	Number    int
	HTMLURL   string `json:"html_url"`
	Title     string
	State     string
	User      *User
	CreatedAt time.Time `json:"created_at"`
	Body      string    // in Markdown format
	Milestone *Milestone
}

type User struct {
	Login   string
	HTMLURL string `json:"html_url"`
}

type Milestone struct {
	Title   string
	HTMLURL string `json:"html_url"`
}

var report1 = temp1.Must(temp1.New("issuelist1").
	Funcs(temp1.FuncMap{"daysAgo": daysAgo}).
	Parse(templ1))
var report2 = temp2.Must(temp2.New("issuelist2").
	Funcs(temp2.FuncMap{"daysAgo": daysAgo}).
	Parse(templ2))

func main() {
	result, err := SearchIssues(os.Args[1:])
	if err != nil {
		log.Fatal(err)
	}
	f1, err := os.Create("report.txt")
	if err := report1.Execute(f1, result); err != nil {
		log.Fatal(err)
	}
	f2, err := os.Create("report.html")
	if err := report2.Execute(f2, result); err != nil {
		log.Fatal(err)
	}
}

func daysAgo(t time.Time) int {
	return int(time.Since(t).Hours() / 24)
}

// SearchIssues queries the GitHub issue tracker.
func SearchIssues(terms []string) (*IssuesSearchResult, error) {
	q := url.QueryEscape(strings.Join(terms, " "))
	resp, err := http.Get(IssuesURL + "?q=" + q)
	if err != nil {
		return nil, err
	}
	// We must close resp.Body on all execution paths.
	// (Chapter 5 presents 'defer', which makes this simpler.)
	if resp.StatusCode != http.StatusOK {
		resp.Body.Close()
		return nil, fmt.Errorf("search query failed: %s", resp.Status)
	}
	var result IssuesSearchResult
	if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
		resp.Body.Close()
		return nil, err
	}
	resp.Body.Close()
	return &result, nil
}
```

```text
//report.txt
41 issues:
----------------------------------------
Number: 23188
User:   ianlancetaylor
Title:  cmd/compile: incorrect order of evaluation according to spec
Age:    7 days
Milestone: Go1.11
----------------------------------------
Number: 23183
User:   terinjokes
Title:  net/http: muxer no longer redirects host patterns in go1.10
Age:    7 days
Milestone: Go1.10
----------------------------------------
Number: 23181
User:   hirochachacha
Title:  cmd/buildid: rewrite algorithm is broken
Age:    7 days
Milestone: Go1.10
----------------------------------------
Number: 23180
User:   aarzilli
Title:  cmd/go: go test -c does not apply specified gcflags to all packa
Age:    7 days
Milestone: Go1.10
----------------------------------------
Number: 23166
User:   mvdan
Title:  x/tools/go/ssa/interp: tests consistently failing on darwin
Age:    8 days
Milestone: Go1.10
----------------------------------------
Number: 23146
User:   bradfitz
Title:  cmd/vet: stderr spam during testing
Age:    12 days
Milestone: Go1.10
----------------------------------------
Number: 23122
User:   rsc
Title:  all: remove support for OS X 10.8 and 10.9
Age:    13 days
Milestone: Go1.11
----------------------------------------
Number: 23098
User:   mikioh
Title:  runtime: loop over allp causes a nil pointer dereference crash
Age:    15 days
Milestone: Go1.10
----------------------------------------
Number: 23037
User:   zolotov
Title:  cmd/go: add JSON output for building package failures while runn
Age:    19 days
Milestone: Go1.10
----------------------------------------
Number: 23036
User:   zolotov
Title:  cmd/test2json: filtering out testing service messages or mark th
Age:    19 days
Milestone: Go1.10
----------------------------------------
Number: 23011
User:   bradfitz
Title:  build: announce end of support for old macOS releases
Age:    21 days
Milestone: Go1.11
----------------------------------------
Number: 23010
User:   rsc
Title:  net/http: ResponseWriter panics in WriteHeaders that were former
Age:    21 days
Milestone: Go1.10
----------------------------------------
Number: 23009
User:   rsc
Title:  net/http/httputil: ReverseProxy change conflicts with future Rev
Age:    21 days
Milestone: Go1.10
----------------------------------------
Number: 22984
User:   rsc
Title:  cmd/go: test -json not cached
Age:    22 days
Milestone: Go1.10
----------------------------------------
Number: 22924
User:   chipaca
Title:  syscall: on linux 386 doesn't support syscalls that don't fail
Age:    27 days
Milestone: Go1.11
----------------------------------------
Number: 22781
User:   pciet
Title:  runtime: fatal error: sweep increased allocation count, go1.9.x
Age:    39 days
Milestone: Go1.9.3
----------------------------------------
Number: 22637
User:   ianlancetaylor
Title:  crypto: examine and probably remove OpenSSL derived code
Age:    48 days
Milestone: Go1.10
----------------------------------------
Number: 22487
User:   tklauser
Title:  lib/time: update tzdata before 1.10 release
Age:    58 days
Milestone: Go1.10
----------------------------------------
Number: 22475
User:   rsc
Title:  cmd/go: include GOROOT in linkActionID hash
Age:    59 days
Milestone: Go1.10
----------------------------------------
Number: 22472
User:   rsc
Title:  cmd/go: implement gccgo support for content-based staleness
Age:    60 days
Milestone: Go1.10
----------------------------------------
Number: 22444
User:   griesemer
Title:  cmd/compile: missing wrapper function for call of literal method
Age:    62 days
Milestone: Go1.11
----------------------------------------
Number: 22429
User:   TheTincho
Title:  cmd/compile: invalid instruction error for FMOVD when compiling 
Age:    63 days
Milestone: Go1.9.3
----------------------------------------
Number: 22349
User:   alexbrainman
Title:  net: ipStackCapabilities.probe creates sockets that can escape i
Age:    68 days
Milestone: Go1.10
----------------------------------------
Number: 22224
User:   siebenmann
Title:  cmd/go: build failure on amd64 Linux with an error in TestTwoGop
Age:    76 days
Milestone: Go1.10
----------------------------------------
Number: 22204
User:   tmm1
Title:  runtime: sigpanic during GC on android/arm64
Age:    77 days
Milestone: Go1.10
----------------------------------------
Number: 22155
User:   rsc
Title:  cmd/go: GOROOT override using linker -X flag is probably not rig
Age:    82 days
Milestone: Go1.10
----------------------------------------
Number: 21431
User:   josharian
Title:  runtime: stack split at a bad time on mipsle
Age:    135 days
Milestone: Go1.10
----------------------------------------
Number: 21282
User:   dsnet
Title:  cmd/compile: incorrect type assertions on conflicting package na
Age:    146 days
Milestone: Go1.11
----------------------------------------
Number: 21221
User:   vibhavp
Title:  cmd/compile: internal compiler error: constant type mismatch whe
Age:    150 days
Milestone: Go1.11
----------------------------------------
Number: 20790
User:   mikioh
Title:  net: DefaultResolver.Lookup{Host,IPAddr} and LookupHost fail to 
Age:    185 days
Milestone: Go1.11
```

```html
//report.html

<h1>41 issues</h1>
<table>
<tr style='text-align: left'>
  <th>#</th>
  <th>State</th>
  <th>User</th>
  <th>Title</th>
  <th>Milestone</th>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23188'>23188</a></td>
  <td>open</td>
  <td><a href='https://github.com/ianlancetaylor'>ianlancetaylor</a></td>
  <td><a href='https://github.com/golang/go/issues/23188'>cmd/compile: incorrect order of evaluation according to spec</a></td>
  <td><a href='https://github.com/golang/go/milestone/62'>Go1.11</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23183'>23183</a></td>
  <td>open</td>
  <td><a href='https://github.com/terinjokes'>terinjokes</a></td>
  <td><a href='https://github.com/golang/go/issues/23183'>net/http: muxer no longer redirects host patterns in go1.10</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23181'>23181</a></td>
  <td>open</td>
  <td><a href='https://github.com/hirochachacha'>hirochachacha</a></td>
  <td><a href='https://github.com/golang/go/issues/23181'>cmd/buildid: rewrite algorithm is broken</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23180'>23180</a></td>
  <td>open</td>
  <td><a href='https://github.com/aarzilli'>aarzilli</a></td>
  <td><a href='https://github.com/golang/go/issues/23180'>cmd/go: go test -c does not apply specified gcflags to all packages</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23166'>23166</a></td>
  <td>open</td>
  <td><a href='https://github.com/mvdan'>mvdan</a></td>
  <td><a href='https://github.com/golang/go/issues/23166'>x/tools/go/ssa/interp: tests consistently failing on darwin</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23146'>23146</a></td>
  <td>open</td>
  <td><a href='https://github.com/bradfitz'>bradfitz</a></td>
  <td><a href='https://github.com/golang/go/issues/23146'>cmd/vet: stderr spam during testing</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23122'>23122</a></td>
  <td>open</td>
  <td><a href='https://github.com/rsc'>rsc</a></td>
  <td><a href='https://github.com/golang/go/issues/23122'>all: remove support for OS X 10.8 and 10.9</a></td>
  <td><a href='https://github.com/golang/go/milestone/62'>Go1.11</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23098'>23098</a></td>
  <td>open</td>
  <td><a href='https://github.com/mikioh'>mikioh</a></td>
  <td><a href='https://github.com/golang/go/issues/23098'>runtime: loop over allp causes a nil pointer dereference crash</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23037'>23037</a></td>
  <td>open</td>
  <td><a href='https://github.com/zolotov'>zolotov</a></td>
  <td><a href='https://github.com/golang/go/issues/23037'>cmd/go: add JSON output for building package failures while running go test on directory with several packages</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23036'>23036</a></td>
  <td>open</td>
  <td><a href='https://github.com/zolotov'>zolotov</a></td>
  <td><a href='https://github.com/golang/go/issues/23036'>cmd/test2json: filtering out testing service messages or mark them in a special way</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23011'>23011</a></td>
  <td>open</td>
  <td><a href='https://github.com/bradfitz'>bradfitz</a></td>
  <td><a href='https://github.com/golang/go/issues/23011'>build: announce end of support for old macOS releases</a></td>
  <td><a href='https://github.com/golang/go/milestone/62'>Go1.11</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23010'>23010</a></td>
  <td>open</td>
  <td><a href='https://github.com/rsc'>rsc</a></td>
  <td><a href='https://github.com/golang/go/issues/23010'>net/http: ResponseWriter panics in WriteHeaders that were formerly ignored</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/23009'>23009</a></td>
  <td>open</td>
  <td><a href='https://github.com/rsc'>rsc</a></td>
  <td><a href='https://github.com/golang/go/issues/23009'>net/http/httputil: ReverseProxy change conflicts with future ReverseProxy plans</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22984'>22984</a></td>
  <td>open</td>
  <td><a href='https://github.com/rsc'>rsc</a></td>
  <td><a href='https://github.com/golang/go/issues/22984'>cmd/go: test -json not cached</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22924'>22924</a></td>
  <td>open</td>
  <td><a href='https://github.com/chipaca'>chipaca</a></td>
  <td><a href='https://github.com/golang/go/issues/22924'>syscall: on linux 386 doesn&#39;t support syscalls that don&#39;t fail</a></td>
  <td><a href='https://github.com/golang/go/milestone/62'>Go1.11</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22781'>22781</a></td>
  <td>open</td>
  <td><a href='https://github.com/pciet'>pciet</a></td>
  <td><a href='https://github.com/golang/go/issues/22781'>runtime: fatal error: sweep increased allocation count, go1.9.x</a></td>
  <td><a href='https://github.com/golang/go/milestone/63'>Go1.9.3</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22637'>22637</a></td>
  <td>open</td>
  <td><a href='https://github.com/ianlancetaylor'>ianlancetaylor</a></td>
  <td><a href='https://github.com/golang/go/issues/22637'>crypto: examine and probably remove OpenSSL derived code</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22487'>22487</a></td>
  <td>open</td>
  <td><a href='https://github.com/tklauser'>tklauser</a></td>
  <td><a href='https://github.com/golang/go/issues/22487'>lib/time: update tzdata before 1.10 release</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22475'>22475</a></td>
  <td>open</td>
  <td><a href='https://github.com/rsc'>rsc</a></td>
  <td><a href='https://github.com/golang/go/issues/22475'>cmd/go: include GOROOT in linkActionID hash</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22472'>22472</a></td>
  <td>open</td>
  <td><a href='https://github.com/rsc'>rsc</a></td>
  <td><a href='https://github.com/golang/go/issues/22472'>cmd/go: implement gccgo support for content-based staleness</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22444'>22444</a></td>
  <td>open</td>
  <td><a href='https://github.com/griesemer'>griesemer</a></td>
  <td><a href='https://github.com/golang/go/issues/22444'>cmd/compile: missing wrapper function for call of literal method expression</a></td>
  <td><a href='https://github.com/golang/go/milestone/62'>Go1.11</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22429'>22429</a></td>
  <td>open</td>
  <td><a href='https://github.com/TheTincho'>TheTincho</a></td>
  <td><a href='https://github.com/golang/go/issues/22429'>cmd/compile: invalid instruction error for FMOVD when compiling for 387</a></td>
  <td><a href='https://github.com/golang/go/milestone/63'>Go1.9.3</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22349'>22349</a></td>
  <td>open</td>
  <td><a href='https://github.com/alexbrainman'>alexbrainman</a></td>
  <td><a href='https://github.com/golang/go/issues/22349'>net: ipStackCapabilities.probe creates sockets that can escape into child process</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22224'>22224</a></td>
  <td>open</td>
  <td><a href='https://github.com/siebenmann'>siebenmann</a></td>
  <td><a href='https://github.com/golang/go/issues/22224'>cmd/go: build failure on amd64 Linux with an error in TestTwoGopathShlibsGccgo from CL 69831  onward</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22204'>22204</a></td>
  <td>open</td>
  <td><a href='https://github.com/tmm1'>tmm1</a></td>
  <td><a href='https://github.com/golang/go/issues/22204'>runtime: sigpanic during GC on android/arm64</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/22155'>22155</a></td>
  <td>open</td>
  <td><a href='https://github.com/rsc'>rsc</a></td>
  <td><a href='https://github.com/golang/go/issues/22155'>cmd/go: GOROOT override using linker -X flag is probably not right</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/21431'>21431</a></td>
  <td>open</td>
  <td><a href='https://github.com/josharian'>josharian</a></td>
  <td><a href='https://github.com/golang/go/issues/21431'>runtime: stack split at a bad time on mipsle</a></td>
  <td><a href='https://github.com/golang/go/milestone/56'>Go1.10</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/21282'>21282</a></td>
  <td>open</td>
  <td><a href='https://github.com/dsnet'>dsnet</a></td>
  <td><a href='https://github.com/golang/go/issues/21282'>cmd/compile: incorrect type assertions on conflicting package names</a></td>
  <td><a href='https://github.com/golang/go/milestone/62'>Go1.11</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/21221'>21221</a></td>
  <td>open</td>
  <td><a href='https://github.com/vibhavp'>vibhavp</a></td>
  <td><a href='https://github.com/golang/go/issues/21221'>cmd/compile: internal compiler error: constant type mismatch when comparing two unsafe.Pointer rvalues</a></td>
  <td><a href='https://github.com/golang/go/milestone/62'>Go1.11</a></td>
</tr>

<tr>
  <td><a href='https://github.com/golang/go/issues/20790'>20790</a></td>
  <td>open</td>
  <td><a href='https://github.com/mikioh'>mikioh</a></td>
  <td><a href='https://github.com/golang/go/issues/20790'>net: DefaultResolver.Lookup{Host,IPAddr} and LookupHost fail to parse a literal IPv6 address w/ zone</a></td>
  <td><a href='https://github.com/golang/go/milestone/62'>Go1.11</a></td>
</tr>

</table>
```

![ex4.14 report.html](http://oumnldfwl.bkt.clouddn.com/ex4.14.png)

