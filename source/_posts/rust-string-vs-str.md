---
title: Rust中的String VS str
date: 2018-09-22 14:34:14
tags: Rust
---

最近在写C++，业余时间在学习Rust，这好像是我第二次学了，第一次学习对有的概念还是很懵逼的，最近写了C++之后，好多东西理解起来不是那么吃力了，里面的概念超级多，好记性不如烂笔头，还是记录一下吧。


在Rust的官方文档References and Borrowing [https://doc.rust-lang.org/book/second-edition/ch04-02-references-and-borrowing.html] 用到了三种不同类型的string变量：String，&String 和 &str


首先看一下str 和 String之间的区别：String是一个可变的、堆上分配的UTF-8的字节缓冲区。而str是一个不可变的固定长度的字符串，如果是从String解引用而来的，则指向堆上，如果是字面值，则指向静态内存。

### 举例说明 String 和 &str


```rust
let a = "hello world";
let b = "OK";
let mut s = String::from("Hello Rust");
println!("{}", s.capacity());    // prints 12
s.push_str("Here I come!");
println!("{}", s.len()); // prints 24
let s = "Hello, Rust!";
println!("{}", s.capacity()); // compile error: no method named `capacity` found for type `&str`
println!("{}", s.len()); // prints 12
```


* 上面的a, b 是&str，不是String，&str更像一个固定的数组，String像一个可变的数组。

* String保留了一个len()和capacity()，但str只有一个len()。

* &str 是 str的一个的borrowed 类型，可以称为一个字符串切片，一个不可变的string。



### 关于&String

&String 是String的borrowed类型，这只不过是一个指针类型，可以传递而不放弃ownership。事实上，一个&String可以当做是&str。
```rust 
fn main() {
    let s = String::from("Hello, Rust!");
    foo(&s);
}

fn foo(s: &str) {
    println!("{}", s);
}
```


foo()可以使用string slice或者borrowed String类型。

如果我们想修改字符串的内容，只需要传递一个可变引用就行了。
```rust 
fn main() {
    let mut s = String::from("Hello, Rust!");
    foo(&mut s);
}

fn foo(s: &mut String) {
    s.push_str("appending foo..");
    println!("{}", s);
}
```



### 相互转换


#### &str => String
```rust 
let c = a.to_string();
let d = String::from(b);
let d = a.to_owned();
```

#### String => &str
```rust 
let e = &String::from("Hello Rust");
// 或使用as_str()
let e_tmp = String::from("Hello Rust");
let e = e_tmp.as_str();
// 不能直接这样使用 
// let e = String::from("Hello Rust").as_str();

```

#### String + &str => String 

String后面接上N个&str
```rust 
let mut strs = "Hello".to_string();
// let mut strs = String::from("Hello");
strs.push_str(" Rust");
println!("{}", strs);

```

### 总结

如果只想要一个字符串的只读视图，或者&str作为一个函数的参数，那就首选&str。如果想拥有所有权，想修改字符串那就用String吧。



