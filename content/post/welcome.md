+++
Categories = ["Development", "GoLang"]
Description = ""
Tags = ["Development", "golang"]
date = "2015-09-23T16:30:37+08:00"
title = "你好，Hugo"

+++

这是使用Hugo创建的站点中的第一篇文章。
<!--more-->

```go
package main

import (
	"bufio"
	"fmt"
	"os"
	"strconv"
)

func main() {
	fmt.Printf("%b", 5)
	r := bufio.NewReader(os.Stdin)
	f, _ := strconv.ParseFloat(readline(r), 64)
	p1, _ := strconv.Atoi(readline(r))
	p2, _ := strconv.Atoi(readline(r))
	total := int(f * (1 + float64(p1+p2)/100))
	fmt.Printf("The total meal cost is %d dollars.\n", total)
}

func readline(r *bufio.Reader) (s string) {
	for {
		line, prefix, _ := r.ReadLine()
		s = s + string(line)
		if !prefix {
			break
		}
	}
	return
}
```

```bash
#!/bin/bash

###### CONFIG
ACCEPTED_HOSTS="/root/.hag_accepted.conf"
BE_VERBOSE=false

if [ "$UID" -ne 0 ]
then
 echo "Superuser rights required"
 exit 2
fi

genApacheConf(){
 echo -e "# Host ${HOME_DIR}$1/$2 :"
}
```