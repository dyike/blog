---
title: Rust中的递归struct的注意点
date: 2018-09-24 14:34:31
tags: Rust
---

> 原文链接：https://stackoverflow.com/questions/25296195/why-are-recursive-struct-types-illegal-in-rust

我们来看一个例子吧：

```rust
struct Person {
    mother: Option<Person>,
    father: Option<Person>,
    partner: Option<Person>,
}

fn main() {
    let susan = Person {
        mother: None,
        father: None,
        partner: None,
    };

    let john = Person {
        mother: None,
        father: None,
        partner: Some(susan),
    };
}
```

编译会出错：
```
error[E0072]: recursive type `Person` has infinite size
 --> recursive.rs:1:1
  |
1 | struct Person {
  | ^^^^^^^^^^^^^ recursive type has infinite size
2 |     mother: Option<Person>,
  |     ---------------------- recursive without indirection
3 |     father: Option<Person>,
  |     ---------------------- recursive without indirection
4 |     partner: Option<Person>,
  |     ----------------------- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to make `Person` representable
  
```

出错的意思：Person是无限大小的。

在Rust中该怎么修复呢？上面也提示了使用`Box`, `Rc`, `&`。 将`Person`放进一个`Box`,然后就正常work了。
```rust
struct Person {
    mother: Option<Box<Person>>,
    father: Option<Box<Person>>,
    partner: Option<Box<Person>>,
}

fn main() {
    let susan = Person {
        mother: None,
        father: None,
        partner: None,
    };

    let john = Person {
        mother: None,
        father: None,
        partner: Some(Box::new(susan)),
    };
}
```

这背后究竟是为什么呢？

我们都知道Rust在编译的时候就需要知道一个类型究竟该分配多少内存。如果一个类型的内存不知道是多少的话，比如说上面的recursive就是其中一种，需要无限的空隙间，Rust就会在编译阶段直接报错。

但是`Box`是知道空间大小的，上面的例子中就在一个递归中插入一个box。Susan有一个mother，father和partner，他们每一个都有一个mother，father，partner......而Box使用一个指针，这个指针是固定大小动态内存分配的。


在`structs`,`enums`（tuples）中的数据是直接内联存储在struct值的内存中的。给定一个struct：

```rust
struct Recursive {
    x: u8,
    y: Option<Recursive>
}
```

现在我们来计算该大小：size_of::<Recursive>()。很明显，x字段需要1byte， Option也需要1byte作为判别，然后就是struct中包含的Recursive的size_of::<Recursive>()
```
size_of::<Recursive>() = 2 + size_of::<Recursive>()
```

这个size将会变得无限的大：
```
Recursive ==
(u8, Option<Recursive>) ==
(u8, Option<(u8, Option<Recursive>)>) ==
(u8, Option<(u8, Option<(u8, Option<Recursive>)>)>) ==
...
```

而`Box<T>`是一个指针，有固定的大小的，所以(u8, Option<Box<Recursive>>)的大小就是 1 + 8 bytes，此时Rust编译就知道该分配多少内存。



