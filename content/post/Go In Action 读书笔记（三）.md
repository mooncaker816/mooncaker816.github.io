+++
Categories = ["Golang"]
Description = ""
Tags = ["Go In Action", "Go map"]
date = "2017-12-12T11:33:37+08:00"
title = "Go In Action 读书笔记（三）"

+++

# 映射
<!--more-->

- 映射用于存储一系列**无序的键值对**，能快速检索数据，键就像索引一样，指向与该键关联的值

- 键通过散列函数得到散列值，该散列值低位用于确定该键值对所在子集的位置，高位用于确认该键值对在该子集中所在的位置

  {% asset_img 1.png 映射散列表%}

- 创建，初始化

  ```go
  // 创建一个映射，键的类型是 string，值的类型是 int
  dict := make(map[string]int)
  // 创建一个映射，键和值的类型都是 string
  // 使用两个键值对初始化映射
  dict := map[string]string{"Red": "#da1337", "Orange": "#e95a22"}
  // 空map
  dict := map[string]int{}
  // nil map，无法直接赋值，需先进行初始化分配空间
  var dict map[string]int
  ```

- 映射的键必须可以进行`==`运算，**切片、函数以及包含切片的结构类型这些类型由于具有引用语义， 不能作为映射的键**，使用这些类型会造成编译错误

- 赋值

  ```go
  // 创建一个空映射，用来存储颜色以及颜色对应的十六进制代码
  colors := map[string]string{}
  // 将 Red 的代码加入到映射
  colors["Red"] = "#da1337"
  ```

- 判断键值是否存在

  ```go
  // 获取键 Blue 对应的值
  value, exists := colors["Blue"] 
  // 这个键存在吗？ 
  if exists { 
    fmt.Println(value) 
  }
  ```

- 迭代映射，由于映射是无序的，所以单纯的for range迭代得到的结果也是无序的，若需要按顺序迭代，需先按key进行排序，再根据key来取map中对应的值

  ```go
  // 显示映射里的所有颜色(无序)
  for key, value := range colors { 
    fmt.Printf("Key: %s Value: %s\n", key, value) 
  }
  // 有序迭代
  s1 := []string
  for key := range colors{
    s1 = append(s1,key)
  }
  sort.Strings(s1)
  for _,v := range s1{
    fmt.Printf("Key: %s Value: %s\n", v, colors[v])
  }
  ```

- 删除

  ```go
  // 删除键为 Coral 的键值对 
  delete(colors, "Coral")
  ```

- 函数间的传递 - **表现特点与切片类似，传输成本很小，不会复制底层数据结构**

  ​

