# Ownership

这是Rust最核心的概念，也是它和其它所有编程语言都不同的一点。

首先，要了解scope的概念，也就是一个变量的作用域。这个很好理解，就是这个变量有效的范围：

```rust
{                      // s is not valid here, it’s not yet declared
	let s = "hello";   // s is valid from this point forward

	// do stuff with s
}                      // this scope is now over, and s is no longer valid
```

然后要记住三个规则：

1. 每一个Rust变量都有一个owner；
2. 在同一时刻只能有一个owner；
3. 当owner离开了作用域之后，变量会被丢弃（dropped）。

上面的例子是一个栈内存的字符串（详见[[Study Log/rust_study/rust_book/rust_string|rust_string]]），所以处理起来和其它所有语言都是一样的。不一样的是分配在堆上的内存，既然没有GC，也不用手动free，内存什么时候回收？我们看下面的例子：

```rust
{
	let s = String::from("hello"); // s is valid from this point forward

	// do stuff with s
}                                  // this scope is now over, and s is no
								   // longer valid
```


在上面的例子中，走到`}`的时候，`s`的作用域结束了。此时编译器会自动帮我们调用一个函数叫做[`drop()`](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop)。这个时候String就会将内存返还回去。

下面我们来看一个更复杂的例子：

```rust
let s1 = String::from("hello");
let s2 = s1;
```

这里就是两个引用指向了堆内存中的同一片区域，和java是一样的。Rust的String由三部分组成：

1. 一个指向内存区域的指针；
2. 内存区域的长度；
3. 内存区域大小。

它们的分布是这样的：

![Three tables: tables s1 and s2 representing those strings on the stack, respectively, and both pointing to the same string data on the heap.|300](https://doc.rust-lang.org/book/img/trpl04-02.svg)

> [!attention]
> 这里我们能看到Rust的内存分配思路：**尽量将能放在栈上的都放在栈上**。在Rust中，指针、长度和size都是放在栈上的，而不是整个打包成对象，然后通过指针/引用去访问。这一点和C++是一样的，和Java是相反的。Java在调用`new String()`的时候，所有的数据都存储在堆上，只有返回的这个引用是在栈上的；而在Rust和C++中，会选择一部分常用的数据存储在栈上。另外，C++还有一个[SSO](https://pvs-studio.com/en/blog/terms/6658/)优化，可以将整个字符串都存储在栈上。
> 
> - [ ] #TODO tasktodo1747407401094 关于这些语言的内存分配细节，可以好好研究下，甚至可以直接研究原码，比如可以从Hotspot和Art虚拟机出发看Java的new String到底做了什么，这些数据到底都在哪里。 ➕ 2025-05-16 🔽 🆔 65m8rt 

但是，结合我们之前说的，在scope结束之后就会通过drop返还内存，可以发现，这里会有一个经典的double free（[[Study Log/os_study/2_virtualization/2_9_memory_api#2.9.4 Common Errors|OSTEP中有提到]]）问题。也就是s1和s2都释放了同一片内存。

这里的解决思路是，如果调用了`let s2 = s1`，则编译器就会认为`s1`已经是无效的了。所以，**上面的代码其实是报错的**。

我们之前介绍过[[Study Log/java_kotlin_study/java_kotlin_study_diary/copy|浅拷贝]]。这里的操作其实就像是一个浅拷贝。但是，此时s1是无效的，所以并不是严格意义上的浅拷贝，而是它的升级版：移动构造：

![Three tables: tables s1 and s2 representing those strings on the stack, respectively, and both pointing to the same string data on the heap. Table s1 is grayed out be-cause s1 is no longer valid; only s2 can be used to access the heap data.|300](https://doc.rust-lang.org/book/img/trpl04-04.svg)

下面是另一个例子：

```rust
let mut s = String::from("hello");
s = String::from("ahoy");

println!("{s}, world!");
```

这里不是两个变量了，而是给一个变量赋值两次。那么这里的问题就是，在赋值为ahoy的时候，hello就已经没用了：

![One table s representing the string value on the stack, pointing to the second piece of string data (ahoy) on the heap, with the original string data (hello) grayed out because it cannot be accessed anymore.|300](https://doc.rust-lang.org/book/img/trpl04-05.svg)

换句话说，hello此时已经离开作用域了。所以，这个时候hello的内存自然就被释放了，通过drop。

Java的默认拷贝也是浅拷贝，想要实现深拷贝，则调用clone()方法。rust也是一样的：

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {s1}, s2 = {s2}");
```

然后，是所有权和函数的结合。下面的代码：

```rust
pub fn ownership() {
    let s = String::from("hello");
    takes_ownership(s);
    s.push_str(" world");
}

fn takes_ownership(some_string: String) {
    println!("{}", some_string);
}
```

会报错，我们在调用`takes_ownership()`之后，就不能使用s了，此时s的所有权也没了。

但是，下面的代码：

```rust
pub fn ownership() {
    let x = 5;
    makes_copy(x);
    println!("{}", x);
}

fn makes_copy(some_integer: i32) {
    println!("{}", some_integer);
}
```

就是没问题的。也就是int可以，String不行。原因是，int实现了一个叫做`Copy`的trait，实现了这个trait之后，在赋值的时候就会是真正的拷贝，而不是移动。因此，在调用`makes_copy()`之后，x依然是有效的。

但是String没有实现Copy，所以，在进行传参的时候，就已经通过移动构造传给了`some_string`，之后s也就无效了。

返回值和作用域

下面的代码：

```rust
pub fn ownership() {
    let s1 = gives_ownership();
    let s2 = String::from("hello");
    let s3 = takes_and_gives_back(s2);
}

fn gives_ownership() -> String {
    let some_string = String::from("yours");
    some_string
}

fn takes_and_gives_back(a_string: String) -> String {
    a_string
}
```

我们直接问吧：在走到最后一行的时候，这三个变量都谁是还有效的？

首先看s1，通过`gives_ownership()`调用的返回值获得了所有权，也就是`some_string`的。之后一直没用，所以它最后是有效的。

然后是s2和s3，s2首先通过正常构造，然后又被传到了`takes_and_gives_back()`里，所以此时s2的所有权就交给`a_string`了。而最后又被返回给了s3。所以最后s2是无效的，s3是有效的。

那么有什么办法可以不这么麻烦呢？有一种直观的方法，既然所有权被转移到函数中了，那我最后再把原来的值传回来，就可以啦：

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{s2}' is {len}.");
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

但是，这样做多少有点蠢，一种更好的办法是，使用引用。
