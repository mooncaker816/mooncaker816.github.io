---
title: The Go Programming Language Ex（6）
date: 2017-12-28 00:00:00
categories:
- Golang
tags:
- The Go Programming Language Ex
- Go Func
- Go crawler
---

### Ex 5.1

Change the findlinks program to traverse the n.FirstChild linked list using recursive calls to visit instead of a loop.

<!--more-->

```go
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)

func main() {
	doc, err := html.Parse(os.Stdin)
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlinks1: %v\n", err)
		os.Exit(1)
	}
	for _, link := range visit(nil, doc) {
		fmt.Println(link)
	}
}

// visit appends to links each link found in n and returns the result.
func visit(links []string, n *html.Node) []string {

	if n == nil {
		return links
	} else {
		if n.Type == html.ElementNode && n.Data == "a" {
			for _, a := range n.Attr {
				if a.Key == "href" {
					links = append(links, a.Val)
				}
			}
		}
		links = visit(links, n.FirstChild)
		links = visit(links, n.NextSibling)
	}
	return links
}
```

```bash
$ ./ex1.7 http://www.baidu.com | ./ex5.1
/
javascript:;
javascript:;
javascript:;
/
javascript:;
https://passport.baidu.com/v2/?login&tpl=mn&u=http%3A%2F%2Fwww.baidu.com%2F
http://news.baidu.com
http://www.hao123.com
http://map.baidu.com
http://v.baidu.com
http://tieba.baidu.com
http://xueshu.baidu.com
https://passport.baidu.com/v2/?login&tpl=mn&u=http%3A%2F%2Fwww.baidu.com%2F
http://www.baidu.com/gaoji/preferences.html
http://www.baidu.com/more/
http://news.baidu.com/ns?cl=2&rn=20&tn=news&word=
http://tieba.baidu.com/f?kw=&fr=wwwt
http://zhidao.baidu.com/q?ct=17&pn=0&tn=ikaslist&rn=10&word=&fr=wwwt
http://music.baidu.com/search?fr=ps&ie=utf-8&key=
http://image.baidu.com/search/index?tn=baiduimage&ps=1&ct=201326592&lm=-1&cl=2&nc=1&ie=utf-8&word=
http://v.baidu.com/v?ct=301989888&rn=20&pn=0&db=0&s=25&ie=utf-8&word=
http://map.baidu.com/m?word=&fr=ps01000
http://wenku.baidu.com/search?word=&lm=0&od=0&ie=utf-8
//www.baidu.com/more/
//www.baidu.com/cache/sethelp/help.html
http://home.baidu.com
http://ir.baidu.com
http://e.baidu.com/?refer=888
http://www.baidu.com/duty/
http://jianyi.baidu.com/
http://www.beian.gov.cn/portal/registerSystemInfo?recordcode=11000002000001
```

### Ex 5.2

Write a function to populate a mapping from element names—p, div, span, and so on—to the number of elements with that name in an HTML document tree.

```go
package main

import (
	"fmt"
	"os"

	"golang.org/x/net/html"
)

func main() {
	doc, err := html.Parse(os.Stdin)
	if err != nil {
		fmt.Fprintf(os.Stderr, "findlinks1: %v\n", err)
		os.Exit(1)
	}
	m := make(map[string]int)
	for k, v := range check(m, doc) {
		fmt.Printf("%s:%d\n", k, v)
	}
}

func check(m map[string]int, n *html.Node) map[string]int {
	if n == nil {
		return m
	} else {
		if n.Type == html.ElementNode {
			m[n.Data]++
		}
		m = check(m, n.FirstChild)
		m = check(m, n.NextSibling)
	}
	return m
}
```

```bash
$ ./ex1.7 http://www.baidu.com | ./ex5.2
body:1
img:2
map:1
area:1
a:32
i:3
head:1
title:1
style:3
ul:1
meta:4
link:11
form:1
span:5
li:4
p:3
html:1
script:13
noscript:1
div:21
input:14
b:2
```

### Ex 5.5

Implement countWordsAndImages. (See Exercise 4.9 for word-splitting.)

```go
package main

import (
	"bufio"
	"fmt"
	"net/http"
	"strings"

	"golang.org/x/net/html"
)

func main() {
	url := "http://www.baidu.com"
	w, i, err := CountWordsAndImages(url)
	if err != nil {
		fmt.Println("CountWordsAndImages error: ", err)
	}
	fmt.Printf("words = %d,images = %d\n", w, i)
}

func CountWordsAndImages(url string) (words, images int, err error) {
	resp, err := http.Get(url)
	if err != nil {
		return
	}
	doc, err := html.Parse(resp.Body)
	resp.Body.Close()
	if err != nil {
		err = fmt.Errorf("parsing HTML: %s", err)
		return
	}
	words, images = countWordsAndImages(doc)
	return
}

func countWordsAndImages(n *html.Node) (words, images int) {
	if n == nil {
		return
	} else {
		if n.Type == html.ElementNode {
			if n.Data == "img" {
				images++
			}
		} else if n.Type == html.TextNode {
			scanner := bufio.NewScanner(strings.NewReader(n.Data))
			scanner.Split(bufio.ScanWords)
			for scanner.Scan() {
				words++
			}
		}
		w1, i1 := countWordsAndImages(n.FirstChild)
		words += w1
		images += i1
		w2, i2 := countWordsAndImages(n.NextSibling)
		words += w2
		images += i2
	}
	return
}
```

```bash
$ go run countwi.go
words = 2805,images = 2
```

### Ex 5.6

Modify the corner function in gopl.io/ch3/surface (§3.2) to use named results and a bare return statement.

```go
package main

import (
	"fmt"
	"math"
	"os"
)

const (
	width, height = 600, 320            // canvas size in pixels
	cells         = 100                 // number of grid cells
	xyrange       = 30.0                // axis ranges (-xyrange..+xyrange)
	xyscale       = width / 2 / xyrange // pixels per x or y unit
	zscale        = height * 0.4        // pixels per z unit
	angle         = math.Pi / 6         // angle of x, y axes (=30°)
)

var sin30, cos30 = math.Sin(angle), math.Cos(angle) // sin(30°), cos(30°)

func main() {
	//fmt.Printf("<svg xmlns='http://www.w3.org/2000/svg' "+
	f, err := os.Create("test.svg")
	if err != nil {
		fmt.Println("create file error ", err)
		return
	}
	fmt.Fprintf(f, "<svg xmlns='http://www.w3.org/2000/svg' "+
		"style='stroke: grey; fill: white; stroke-width: 0.7' "+
		"width='%d' height='%d'>", width, height)
	for i := 0; i < cells; i++ {
		for j := 0; j < cells; j++ {
			ax, ay := corner(i+1, j)
			bx, by := corner(i, j)
			cx, cy := corner(i, j+1)
			dx, dy := corner(i+1, j+1)
			fmt.Fprintf(f, "<polygon points='%g,%g %g,%g %g,%g %g,%g'/>\n",
				ax, ay, bx, by, cx, cy, dx, dy)
		}
	}
	fmt.Fprintln(f, "</svg>")
}

func corner(i, j int) (sx, sy float64) {
	// Find point (x,y) at corner of cell (i,j).
	x := xyrange * (float64(i)/cells - 0.5)
	y := xyrange * (float64(j)/cells - 0.5)

	// Compute surface height z.
	z := f(x, y)

	// Project (x,y,z) isometrically onto 2-D SVG canvas (sx,sy).
	sx = width/2 + (x-y)*cos30*xyscale
	sy = height/2 + (x+y)*sin30*xyscale - z*zscale
	return
}

func f(x, y float64) float64 {
	r := math.Hypot(x, y) // distance from (0,0)
	return math.Sin(r) / r
}
```

### Ex 5.7

Develop startElement and endElement into a general HTML pre Print comment nodes, text nodes, and the attributes of each element (<a href='...'>). Use short forms like <img/> instead of <img></img> when an element has no children. Write a test to ensure that the output can be parsed successfully. (See Chapter 11.)

```go
package main

import (
	"fmt"
	"net/http"
	"os"

	"golang.org/x/net/html"
)

func main() {
	for _, url := range os.Args[1:] {
		outline(url)
	}
}

func outline(url string) error {
	resp, err := http.Get(url)
	if err != nil {
		return err
	}
	defer resp.Body.Close()
	doc, err := html.Parse(resp.Body)
	if err != nil {
		return err
	}

	forEachNode(doc, startElement, endElement)

	return nil
}

func forEachNode(n *html.Node, pre, post func(n *html.Node)) {
	if pre != nil {
		pre(n)
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		forEachNode(c, pre, post)
	}
	if post != nil {
		post(n)
	}
}

var depth int

func startElement(n *html.Node) {
	if n.Type == html.ElementNode ||
		n.Type == html.TextNode ||
		n.Type == html.CommentNode {
		if n.DataAtom != 0 { //去除无法识别标签的节点，如空白的textnode等
			var attr string
			for _, v := range n.Attr {
				attr += " " + v.Key + "='" + v.Val + "'"
			}
			if n.FirstChild == nil {
				fmt.Printf("%*s<%s/>\n", depth*2, "", n.Data)
			} else {
				fmt.Printf("%*s<%s%s>\n", depth*2, "", n.Data, attr)
			}
			depth++
		}
	}
}
func endElement(n *html.Node) {
	if n.Type == html.ElementNode ||
		n.Type == html.TextNode ||
		n.Type == html.CommentNode {
		if n.DataAtom != 0 {
			depth--
			if n.FirstChild != nil {
				fmt.Printf("%*s</%s>\n", depth*2, "", n.Data)
			}
		}
	}
}
```

```bash
$ ./ex5.7 http://gopl.io
<html xmlns='http://www.w3.org/1999/xhtml'>
  <head>
    <meta/>
    <title>
    </title>
    <script>
    </script>
    <link/>
    <style>
    </style>
  </head>
  <body>
    <table>
      <tbody>
        <tr>
          <td>
            <a href='http://www.informit.com/store/go-programming-language-9780134190440'>
              <img/>
            </a>
            <br/>
            <div style='text-align: center'>
              <a href='http://www.amazon.com/dp/0134190440'>
                <img/>
              </a>
              <a href='http://www.informit.com/store/go-programming-language-9780134190440'>
                <img/>
              </a>
              <a href='http://www.barnesandnoble.com/w/1121601944'>
                <img/>
              </a>
            </div>
            <br/>
          </td>
          <td width='500'>
            <h1 class='center'>
            </h1>
            <p class='biblio center'>
              <br/>
              <br/>
              <br/>
              <tt>
              </tt>
              <tt>
              </tt>
              <tt>
              </tt>
            </p>
            <div id='toc'>
              <table>
                <tbody>
                  <tr>
                    <td>
                      <h1>
                        <a href='ch1.pdf'>
                        </a>
                      </h1>
                      <h1>
                        <a href='ch1.pdf'>
                        </a>
                      </h1>
                      <h1>
                        <a href='ch1.pdf'>
                        </a>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                    </td>
                    <td>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                      </h1>
                      <h1>
                        <a href='ch1.pdf'>
                        </a>
                      </h1>
                    </td>
                  </tr>
                  <tr>
                    <td colspan='2'>
                      <h1>
                        <a href='https://github.com/adonovan/gopl.io/'>
                        </a>
                        <a href='reviews.html'>
                        </a>
                        <a href='translations.html'>
                        </a>
                        <a href='errata.html'>
                        </a>
                      </h1>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
            <p class='bio'>
              <a href='http://golang.org/s/oracle-user-manual'>
                <code>
                </code>
              </a>
              <a href='http://golang.org/lib/godoc/analysis/help.html'>
                <code>
                </code>
              </a>
              <a href='https://github.com/golang/tools/blob/master/refactor/eg/eg.go'>
                <code>
                </code>
              </a>
              <a href='https://github.com/golang/tools/blob/master/refactor/rename/rename.go'>
                <code>
                </code>
              </a>
            </p>
            <p class='bio'>
              <a href='http://www.amazon.com/dp/0131103628?tracking_id=disfordig-20'>
              </a>
              <a href='http://www.amazon.com/dp/020161586X?tracking_id=disfordig-20'>
              </a>
            </p>
          </td>
        </tr>
      </tbody>
    </table>
  </body>
</html>
```

- 遍历TextNode时，含有很多空节点（其实就是空格，也当成了一个节点），或许应该在parse阶段就抛弃这些不是节点的节点。本习题中，我暂时借用了元素节点的字段`DataAtom  atom.Atom`，若为0，则说明该节点标签无法识别。

  >// A Node consists of a NodeType and some Data (tag name for element nodes,
  >// content for text) and are part of a tree of Nodes. Element nodes may also
  >// have a Namespace and contain a slice of Attributes. Data is unescaped, so
  >// that it looks like "a<b" rather than "a&lt;b". For element nodes, DataAtom
  >// is the atom for Data, or zero if Data is not a known tag name.

### Ex 5.8

Modify forEachNode so that the pre and post functions return a boolean re indicating whether to continue the traversal. Use it to write a function ElementByID with the following signature that ﬁnds the ﬁrst HTML element with the speciﬁed id attribute. The function should stop the traversal as soon as a match is found.

`func ElementByID(doc *html.Node, id string) *html.Node`

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"os"

	"golang.org/x/net/html"
)

var custID = "css_index_result"

//var custID = "toc"

func main() {
	for _, url := range os.Args[1:] {
		outline(url)
	}
}

func outline(url string) error {
	resp, err := http.Get(url)
	if err != nil {
		return err
	}
	defer resp.Body.Close()
	doc, err := html.Parse(resp.Body)
	if err != nil {
		return err
	}
	//forEachNode(doc, "", startElement, endElement)
	fmt.Printf("%#v:\n", getElementByID(doc, custID))
	return nil
}

func forEachNode(n *html.Node, id string, pre, post func(n *html.Node, id string) bool) *html.Node {
	var rtn *html.Node
	if pre != nil {
		if !pre(n, id) {
			rtn = n
		}
	}
	for c := n.FirstChild; c != nil && rtn == nil; c = c.NextSibling {
		rtn = forEachNode(c, id, pre, post)
	}
	if post != nil {
		if !post(n, id) {
			rtn = n
		}
	}
	return rtn
}

var depth int

func startElement(n *html.Node, id string) bool {
	if n.Type == html.ElementNode ||
		n.Type == html.TextNode ||
		n.Type == html.CommentNode {
		if n.DataAtom != 0 { //去除无法识别标签的节点，如空白的textnode等
			var attr string
			for _, v := range n.Attr {
				attr += " " + v.Key + "='" + v.Val + "'"
			}
			if n.FirstChild == nil {
				fmt.Printf("%*s<%s/>\n", depth*2, "", n.Data)
			} else {
				fmt.Printf("%*s<%s%s>\n", depth*2, "", n.Data, attr)
			}
			depth++
		}
		if n.Type == html.ElementNode {
			for _, v := range n.Attr {
				if v.Key == "id" && v.Val == id {
					log.Printf("%#v\n", n)
					return false
				}
			}
		}
	}
	return true
}
func endElement(n *html.Node, id string) bool {
	if n.Type == html.ElementNode ||
		n.Type == html.TextNode ||
		n.Type == html.CommentNode {
		if n.DataAtom != 0 {
			depth--
			if n.FirstChild != nil {
				fmt.Printf("%*s</%s>\n", depth*2, "", n.Data)
			}
		}
	}
	return true
}

func getElementByID(doc *html.Node, id string) *html.Node {
	return forEachNode(doc, id, startElement, endElement)
}
```

```bash
$ ./ex5.7 http://www.baidu.com
<html>
  <head>
    <meta/>
    <meta/>
    <meta/>
    <meta/>
    <link/>
    <link/>
    <link/>
    <link/>
    <link/>
    <link/>
    <link/>
    <link/>
    <link/>
    <link/>
    <link/>
    <title>
    </title>
    <style id='css_index' index='index' type='text/css'>
    </style>
    <style data-for='debug'>
    </style>
    <style data-for='result' id='css_index_result' type='text/css'>
2018/01/02 17:53:11 &html.Node{Parent:(*html.Node)(0xc4201322a0), FirstChild:(*html.Node)(0xc4201336c0), LastChild:(*html.Node)(0xc4201336c0), PrevSibling:(*html.Node)(0xc4201335e0), NextSibling:(*html.Node)(0xc420133730), Type:0x3, DataAtom:0x6f905, Data:"style", Namespace:"", Attr:[]html.Attribute{html.Attribute{Namespace:"", Key:"data-for", Val:"result"}, html.Attribute{Namespace:"", Key:"id", Val:"css_index_result"}, html.Attribute{Namespace:"", Key:"type", Val:"text/css"}}}
    </style>
  </head>
</html>
&html.Node{Parent:(*html.Node)(0xc4201322a0), FirstChild:(*html.Node)(0xc4201336c0), LastChild:(*html.Node)(0xc4201336c0), PrevSibling:(*html.Node)(0xc4201335e0), NextSibling:(*html.Node)(0xc420133730), Type:0x3, DataAtom:0x6f905, Data:"style", Namespace:"", Attr:[]html.Attribute{html.Attribute{Namespace:"", Key:"data-for", Val:"result"}, html.Attribute{Namespace:"", Key:"id", Val:"css_index_result"}, html.Attribute{Namespace:"", Key:"type", Val:"text/css"}}}:
```

### Ex 5.12

The startElement and endElement functions in gopl.io/ch5/outline2 (§5.5) share a global variable, depth. Turn them into anonymous functions that share a variable local to the outline function.

```go
// Copyright © 2016 Alan A. A. Donovan & Brian W. Kernighan.
// License: https://creativecommons.org/licenses/by-nc-sa/4.0/

// See page 133.

// Outline prints the outline of an HTML document tree.
package main

import (
	"fmt"
	"net/http"
	"os"

	"golang.org/x/net/html"
)

func main() {
	for _, url := range os.Args[1:] {
		outline(url)
	}
}

func outline(url string) error {
	resp, err := http.Get(url)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	doc, err := html.Parse(resp.Body)
	if err != nil {
		return err
	}
	var depth int
	startElement := func(n *html.Node) {
		if n.Type == html.ElementNode {
			fmt.Printf("%*s<%s>\n", depth*2, "", n.Data)
			depth++
		}
	}

	endElement := func(n *html.Node) {
		if n.Type == html.ElementNode {
			depth--
			fmt.Printf("%*s</%s>\n", depth*2, "", n.Data)
		}
	}
	//!+call
	forEachNode(doc, startElement, endElement)
	//!-call

	return nil
}

//!+forEachNode
// forEachNode calls the functions pre(x) and post(x) for each node
// x in the tree rooted at n. Both functions are optional.
// pre is called before the children are visited (preorder) and
// post is called after (postorder).
func forEachNode(n *html.Node, pre, post func(n *html.Node)) {
	if pre != nil {
		pre(n)
	}

	for c := n.FirstChild; c != nil; c = c.NextSibling {
		forEachNode(c, pre, post)
	}

	if post != nil {
		post(n)
	}
}
```

### Ex 5.13

Modify crawl to make local copies of the pages it ﬁnds, creating directories as necessary. Don’t make copies of pages that come from a different domain. For example, if the original page comes from golang.org, save all ﬁles from there, but exclude ones from vimeo.com.

```go
package main

import (
	"fmt"
	"io"
	"log"
	"net/http"
	"net/url"
	"os"
	"strconv"
	"strings"
	"time"

	"golang.org/x/net/html"
	"gopl.io/ch5/links"
)

// Extract makes an HTTP GET request to the specified URL, parses
// the response as HTML, and returns the links in the HTML document.
func Extract(urlstr string) ([]string, error) {
	resp, err := http.Get(urlstr)
	if err != nil {
		return nil, err
	}
	if resp.StatusCode != http.StatusOK {
		resp.Body.Close()
		return nil, fmt.Errorf("getting %s: %s", urlstr, resp.Status)
	}
	doc, err := html.Parse(resp.Body)
	resp.Body.Close()
	if err != nil {
		return nil, fmt.Errorf("parsing %s as HTML: %v", urlstr, err)
	}
	var links []string
	visitNode := func(n *html.Node) {
		if n.Type == html.ElementNode && n.Data == "a" {
			for _, a := range n.Attr {
				if a.Key != "href" {
					continue
				}
				link, err := resp.Request.URL.Parse(a.Val)
				if err != nil {
					continue // ignore bad URLs
				}
				links = append(links, link.String())
			}
		}
	}
	forEachNode(doc, visitNode, nil)
	return links, nil
}

//!-Extract

// Copied from gopl.io/ch5/outline2.
func forEachNode(n *html.Node, pre, post func(n *html.Node)) {
	if pre != nil {
		pre(n)
	}
	for c := n.FirstChild; c != nil; c = c.NextSibling {
		forEachNode(c, pre, post)
	}
	if post != nil {
		post(n)
	}
}

func breadthFirst(f func(item string, host []string) []string, worklist []string) {
	seen := make(map[string]bool)
	host := make([]string, len(worklist))
	for i, v := range worklist {
		u, err := url.Parse(v)
		if err != nil {
			fmt.Println("initial url parse failed")
			return
		}
		host[i] = u.Host
	}
	for len(worklist) > 0 {
		items := worklist
		worklist = nil
		for _, item := range items {
			if !seen[item] {
				seen[item] = true
				worklist = append(worklist, f(item, host)...)
			}
		}
	}
}

//!-breadthFirst

//!+crawl
func crawl(urlstr string, sl []string) []string {
	fmt.Println(urlstr)
	copycontent(urlstr, sl)
	list, err := links.Extract(urlstr)
	if err != nil {
		log.Print(err)
	}
	return list
}

func copycontent(s string, sl []string) {
	u, err := url.Parse(s)
	if err != nil {
		fmt.Println("url parse failed")
		return
	}
	for _, v := range sl {
		if u.Host == v {
			resp, err := http.Get(s)
			if err != nil {
				fmt.Println(err)
				return
			}
			if resp.StatusCode != http.StatusOK {
				resp.Body.Close()
				return
			}
			//fmt.Println(path)
			dir, _ := os.Getwd() //当前的目录
			var filename, filepart, dirpart string
			ns := strings.Count(u.Path, "/")
			if ns == 0 || ns == 1 && len(u.Path) == 1 {
				filepart = ""
				filename = strconv.FormatInt(time.Now().Unix(), 10) + ".html"
				dirpart = "/"
			} else {
				filepart = u.Path[strings.LastIndex(u.Path, "/")+1:]
				dirpart = u.Path[:strings.LastIndex(u.Path, "/")+1]
				if strings.Contains(filepart, ".") {
					filename = filepart
				} else {
					dirpart = u.Path
					filename = strconv.FormatInt(time.Now().Unix(), 10) + ".html"
				}
			}

			fullpath := dir + "/" + u.Host + dirpart
			_, err = os.Stat(fullpath)
			if err != nil {
				err = os.MkdirAll(fullpath, os.ModePerm) //在当前目录下生成md目录
				if err != nil {
					fmt.Println("create folder failed! ", fullpath, err)
					return
				}
			}
			filename = dir + "/" + u.Host + dirpart + "/" + filename
			f, err := os.Create(filename)
			if err != nil {
				fmt.Println("create file error:", err, s)
				return
			}
			_, err = io.Copy(f, resp.Body)
			if err != nil {
				fmt.Println("failed in copy")
				return
			}
			resp.Body.Close()
		}
	}
}
func main() {
	breadthFirst(crawl, os.Args[1:])
}
```

