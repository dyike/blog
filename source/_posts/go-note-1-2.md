---
title: Go语言学习笔记：【Go中字符串小记】
tags:
  - Golang
date: 2015-07-26 17:01:49
---

 Go语言中的字符串

**字符串是用一对双号(“”)或者单引号(‘’)括起来的定义的，它的类型是srting。**

值得注意的是，在go语言中字符串是不可变的，有这样一段代码，编译时会报错。

```go
var s string = "hello"

s[0] = 'c'
```

但是想要修改怎么办呢？方法如下：

```go
s := "hello"

c := []byte(s) // 将字符串 s转换为 []byte类型

c[0] = 'c'

s2 := string(c) // 再转换回 string类型

fmt.Printf("%s\n", s2)
```

修改也可以这样：

```go
s := "hello"

s = "c" + s[1:] // 字符串虽不能更改，但可进行切片操作

fmt.Printf("%s\n", s)
```

go中也可声明多行字符串，可以通过`来声明：

```go
m := hello
world
```


---