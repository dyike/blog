---
title: Rust智能指针——Box
date: 2018-09-24 18:05:15
tags: Rust
---

在C++11中也有智能指针shared_ptr，unique_ptr，weak_ptr，在Rust中也有叫智能指针的东西，今天我们来讨论一下Box。现在我们要构建一个二叉树。在Python中实现是比较简单的：

```python 
class Tree:
    def __init__(self, root_val, left = None, right = None):
        self.root = root_val
        self.left = left
        self.right = right
        
        
t = Tree(12,
        Tree(10, None, Tree(14)),
        Tree(16, Tree(15), Tree(22))))
```

最终的树形结构：

```bash
                                    12 
                                  /    \
                                10      16                                    
                                 \      / \
                                  14  15   22
```

现在用Rust来实现一个？

```rust
struct Tree {
    root: i64,
    left: Tree,
    right: Tree,
}
```

在上一篇我们已经提到了，上面的代码Rust肯定不会让我们编译通过。已经提示我们使用`&`、`Box`、`Rc`。他们都是指针类型，都是指向内存上某一个位置。

* `&` 在Rust里面称作borrow类型。只是引用内存上某个位置的值，并不拥有所指的值。如果我们想使用`&`来修复这个问题，我们需要注意一个问题就是生命周期，borrow类型生命周期取决owner的生命周期。

```rust
struct Tree<'a> {
    root: i64,
    left: &'a Tree<'a>,
    right: &'a Tree<'a>,
}
```

* `Box`是一种智能指针，零运行时开销。拥有它所指向的数据。我们为什么称做智能的，是因为当执行过边界，它将drop掉它所指向的数据然后drop本身。不需要手动管理内存！！！

```rust
struct Tree {
  root: i64,
  left: Box<Tree>,
  right: Box<Tree>,
}
```

* `Rc`是另外一种智能指针，是`reference count`的简称。用于记录数据结构被引用的次数。当引用的数字减小到0时，就自行清理。如果在一个线程，对于同一个数据有多个owner的时候，我们选用`Rc`。对于多线程，我们有`Arc`（atomic reference count)

```rust
struct Tree {
  root: i64,
  left: Rc<Tree>,
  right: Rc<Tree>,
}
```


上面的三个方法都可以解决之前的问题，那我们该怎么选择呢，这取决于自己的使用场景。

### Make subtrees optional
紧接着我们又面临的问题就是无法实例化之前定义的tree结构。为什么呢？现有的数据结构，left和right子树都是Box<Tree>的类型，但是我们定义的tree节点有一些是空子树。在Python代码中，我们使用了`None`,但在Rust中有`Option`：

```rust
struct Tree {
  root: i64,
  left: Option<Box<Tree>>,
  right: Option<Box<Tree>>,
}
```

现在我们就可以创建我们第一个tree：
```rust
Tree {
    root: 12,
    left: Some(Box::new(Tree {
            root: 10,
            left: None,
            right: Some(Box::new(Tree {
                    root: 14,
                    left: None,
                    right: None,
            })),
    })),
    right: Some(Box::new(Tree {
            root: 16,
            left: Some(Box::new(Tree {
                    root: 15,
                    left: None,
                    right: None,
            })),
            right: Some(Box::new(Tree {
                    root: 22,
                    left: None,
                    right: None,
            })),
    })),
};

```

现在将代码做一些优化：

```rust
#[derive(Default)]
struct Tree {
  root: i64,
  left: Option<Box<Tree>>,
  right: Option<Box<Tree>>,
}

impl Tree {
  fn new(root: i64) -> Tree {
    Tree {
      root: root,
      ..Default::default()
    }
  }

  fn left(mut self, leaf: Tree) -> Self {
    self.left = Some(Box::new(leaf));
    self
  }

  fn right(mut self, leaf: Tree) -> Self {
    self.right = Some(Box::new(leaf));
    self
  }
}

```

```rust
Tree::new(12)
  .left(
    Tree::new(10)
      .right(Tree::new(14))
  )
  .right(
    Tree::new(16)
      .left(Tree::new(15))
      .right(Tree::new(22))
  );

```

那为什么在Python就能完美运行呢？Python在运行的时候为树对象动态分配内存，将所有内容包装在PyObject中，类似Rc。

遇到类似的情况我们该怎么处理和选择呢？我们需要了解所有可能的解决方案，如果可以，我们就远离使用智能指针，坚持简单借用。如果不能，我们就选择侵入性更小的一个。

