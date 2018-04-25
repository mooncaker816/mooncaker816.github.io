---
title: Go In Action 读书笔记（一）
date: 2017-09-25 17:00:00
categories:
- Golang
tags:
- Go In Action
- Go package
---

### 包

<!--more-->

- 目录即为包
- 同一目录下各个.go文件所属包名必须相同
- 包名可与目录名不同，但习惯保持一致（main包例外）
- 同一项目下，main包，main()函数，必须存在且唯一
- go build编译后的可执行程序名为main包代码所在目录的目录名
- 导入包，编译器会按GOROOT,GOPATH的顺序搜索包名依次导入，找到即会停止搜索，找不到就报错 
  ```go
  import (
      "fmt"
      "os"
  )
  ```

- 远程导入，支持直接从github，bitbucket等网站直接导入
  事实上， 这个导入路径代表一个 URL，指向 GitHub 上的代码库。如果路径包含 URL，可以使用 Go 工具链从 DVCS 获取包，并把包的源代码保存在 GOPATH 指向的路径里与 URL 匹配的目录里。这个获取过程 使用 go get 命令完成。go get 将获取任意指定的 URL 的包，或者一个已经导入的包所依赖的其 他包。由于go get的这种递归特性，这个命令会扫描某个包的源码树，获取能找到的所有依赖包。


- 命名导入，当包名冲突时，可重定义包名以示区别，使用当中只需使用重命名后当包名即可
  ```go
  package main
  import (
       "fmt"
       myfmt "mylib/fmt"
   )
  ```
  当重命名名称为下划线时,导入的包只执行init初始化函数，如为数据库注册驱动

  ```go
  package main
  import (
       "database/sql"
     _ "github.com/goinaction/code/chapter3/dbdriver/postgres" 
  )

  func main() {
   sql.Open("postgres", "mydb")
  }
  ```

- 导入的包必须被使用到，否则编译会报错






