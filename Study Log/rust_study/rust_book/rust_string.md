# String in Rust

在Rust中，String类型是可变的：

```rust
let mut str = String::from("hello");
str.push_str(", world");
```

这里为什么需要加mut才行呢？这个和所有权有关系，其实，String本身就是一个可变的结构，内部实现就是一个vector。但是rust的编译期会加上限制，**只要你的变量没有mut修饰，你就不能调用String的`push_str()`函数，即使这个函数本身是公开可访问的**。

这个特性是其它语言所没有的，其它任何语言中，只要一个函数存在于一个类/结构体中，那么本身就是可以访问的。而Rust通过所有权加了这样一个限制，来保证内存安全。

而对于hard code的String，其实它是另一种类型，这个借助ide就能看出来：

![[Study Log/rust_study/rust_book/resources/Pasted image 20250516212709.png]]

一个是String类型，另一个是\&str类型。后者是不可变的，存在栈上的数据。

所以，和Java相比，Java的String对应到Rust，应该是\&str，都是不可变的；而Rust的String对应到Java，应该是StringBuilder和StringBuffer。