---
title: Go In Action 读书笔记（四）
date: 2017-12-14 09:39:07
categories:
- Golang
tags:
- Go In Action
- Go type
- Go method
- Go interface
---

### 用户类型

<!--more-->

- 所谓用户类型，就是由一个或多个基本类型或用户类型组合而成的一个新的结构类型。

  ```go
  //多个基本类型组合而成
  type user struct {
    name        string
    email       string
    ext         int
    privileged  bool
  }
  //一个基本类型，亦可称为别称
  type Duration int64
  ```

- 定义，初始化

  ```go
  //定义一个类型为user,初始值为零值的结构变量bill
  var bill user
  //定义一个类型为user,且有初始值的结构变量bill
  bill := user{"Lisa", "lisa@email.com", 123, true}
  bill := user{
    name :	"Lisa",
    email :	"lisa@email.com",
    ext :		123,
    privileged :	true,
  }
  ```

- **别称和他的基础类型不能直接赋值，因为Go认为他们不是同一类型**

### 方法

- 函数是没有接收者的，不属于任何用户类型或基本类型

  ```go
  //没有接收者
  func add(a int, b int) int{
      return a + b
  }
  ```

- 方法是有接收者（用户类型）的函数

  ```go
  type score struct {
    math		int
    english	int
    chinese	int
  }
  //有接收者
  func (s score) sum() int{
      return s.math + s.english + s.chinese
  }
  ```

- 方法的调用者和接收者之间的关系

  1. 当调用者和接收者同类型时，正常调用
  2. 当调用者为值，接收者为指针时，Go会自动取调用者的地址再调用方法，如(&a).func
  3. 当调用者为指针，接收者为值时，Go会自动对调用者解引用再调用方法，如(*a).func

    **开发者无需过多的考虑，调用者和接收者是否匹配的问题，Go会自动处理**

  ```go
  type user struct {
      name	 string
      email	 string
  }
  // notify 使用值接收者实现了一个方法
  func (u user) notify(){
    fmt.Printf("Sending User Email To %s<%s>\n",u.name,u.email)
  }
  // changeEmail 使用指针接收者实现了一个方法
  func (u *user) changeEmail(email string) {
      u.email = email
  }
  //初始化user类型的值bill
  bill := user{"Bill", "bill@email.com"}
  //初始化指向user类型的指针lisa
  lisa := &user{"Lisa", "lisa@email.com"}
  //值bill调用值为接收者的方法notify
  bill.notify() //Sending User Email To Bill<bill@email.com>
  //值bill调用指针为接收者的方法changeEmail
  bill.changeEmail("bill@newdomain.com") // success
  bill.notify() //Sending User Email To Bill<bill@newdomain.com>
  //指针lisa调用值为接收者的方法notify()
  lisa.notify() //Sending User Email To Lisa<lisa@email.com>
  //指针lisa调用指针为接收者的方法changeEmail
  lisa.changeEmail("lisa@newdomain.com") // success
  lisa.notify() //Sending User Email To Lisa<lisa@newdomain.com>
  ```

- 方法定义的接收者究竟应该是指针还是值，需按该方法的需求决定。

### 接口

- 接口是一种抽象的类型，没有具体实现，是一组方法的集合

- 多态是指代码可以根据类型的具体实现采取不同行为的能力。如果一个类型实现了某个接口，所有使用这个接口的地方，都可以支持这种类型的值。换句话说，如果用户定义的类型实现了某个接口类型声明的一组方法，那么这个用户定 义的类型的值就可以赋给这个接口类型的值。这个赋值会把用户定义的类型的值存入接口类型 的值。

- 接口的内部实现

  接口值是一个两个字长度的数据结构，第一个字包含一个指向内部表的指针。这个内部表叫作 iTable，包含了所存储的值的类型信息。iTable包含了已存储的值的类型信息以及与这个值相关联的一组方法。第二个字是一个指向所存储值的指针。

  ![实体值赋值后接口值](http://oumnldfwl.bkt.clouddn.com/%E5%AE%9E%E4%BD%93%E5%80%BC%E8%B5%8B%E5%80%BC%E5%90%8E%E6%8E%A5%E5%8F%A3%E5%80%BC.png)

  ![实体指针赋值后接口值](http://oumnldfwl.bkt.clouddn.com/%E5%AE%9E%E4%BD%93%E6%8C%87%E9%92%88%E8%B5%8B%E5%80%BC%E5%90%8E%E6%8E%A5%E5%8F%A3%E5%80%BC.png)

- 方法集

  * 方法集定义了一组关联到给定类型的值或者指针的方法
  * T 类型的值的方法集只包含值接收者声明的方法，指向 T 类型的指针的方法集既包含值接收者声明的方法，也包含指针接收者声明的方法
  * 如果使用**<u>指针接收者</u>**来实现一个接口，那么只有指向那个类型的指针才能够实现对应的接口
  * 如果使用**<u>值接收者</u>**来实现一个接口，那么那个类型的值和指针都能够实现对应的接口

| Values | Methods Receivers |
| ------ | ----------------- |
| T      | (t T)             |
| \*T    | (t T) and (t \*T) |

| Methods Receivers | Values    |
| ----------------- | --------- |
| (t T)             | T and \*T |
| (t \*T)           | \*T       |

**因为不是总能获取一个值的地址，所以值的方法集只包括了使用值接收者实现的方法。**

- 结论：

  **实体类型以值接收者实现接口的时候，不管是实体类型的值，还是实体类型值的指针，都实现了该接口**。

  ```go
  type notifier interface {
    notify()
  }
  type user struct {
      name 	string
      email 	string
  }
  func (u user) notify() {
    fmt.Printf("Sending user email to %s<%s>\n",u.name,u.email)
  }
  //接收一个实现notifier接口的值
  func sendNotification(n notifier) {
    n.notify()
  }

  func main() {
    bill := user{"Bill", "bill@email.com"}
    lisa := user{"Lisa", "lisa@email.com"}
    sendNotification(bill)// Sending User Email To Bill<bill@email.com>
    sendNotification(&lisa)// Sending User Email To Lisa<lisa@email.com>
  }
  ```

  **实体类型以指针接收者实现接口的时候，只有指向这个类型的指针才被认为实现了该接口**

  ```go
  type notifier interface {
    notify()
  }
  type user struct {
      name 	string
      email 	string
  }
  func (u *user) notify() {
    fmt.Printf("Sending user email to %s<%s>\n",u.name,u.email)
  }
  //接收一个实现notifier接口的值
  func sendNotification(n notifier) {
    n.notify()
  }

  func main() {
    bill := user{"Bill", "bill@email.com"}
    lisa := user{"Lisa", "lisa@email.com"}
    sendNotification(bill)// ERROR
    sendNotification(&lisa)// Sending User Email To Lisa<lisa@email.com>
  }
  ```


  ​