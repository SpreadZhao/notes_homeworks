# References

rust中有引用，没有指针。引用可以保证，在整个引用的生命周期内，它指向的内存区域，一定是一个有效的，确定的值。i32就是i32，f32就是f32，不会像C语言那样内存会被别人改掉。 ^8efd00

下面的代码：

```rust
pub fn reference() {
    let s1 = String::from("hello");
    let len = calculate_len(&s1);
    println!("The length of '{s1}' is {len}.");
}

fn calculate_len(s: &String) -> usize {
    return s.len();
}
```

在内存中是这样的：

![[Study Log/rust_study/rust_book/resources/Pasted image 20250519004423.png|400]]

> [!note]
> 其实，这样的创建模式才是Java中String的默认方式。但是内存情况又和java不同。

`&s1`的作用是，对`s1`**创建**一个引用。引用是不会对数据有所有权的。因此，将引用传递过去并没有发生所有权转移。看下面的代码：

```rust
pub fn reference() {
    let s1 = String::from("hello");
    let r = &s1;
    let len = calculate_len(r);
    println!("The length of '{s1}' is {len}, ref is {}", r);
}

fn calculate_len(s: &String) -> usize {
    return s.len();
}
```

我们让`r`是`s1`的引用，并将`r`传递了进去。之后，又使用了`r`。但是它依然还能使用。证明在传参的过程中，并没有发生所有权的转移。还是那句话，**reference does not have ownership of what it refers to**.

我们把创建一个引用的行为叫做borrowing。

然后是，如何对引用指向的内容做修改？首先，只有可变类型才能修改。其次，可变类型也只能由可变引用指向：

```rust
pub fn reference() {  
    let mut s = String::from("hello");  
    change(&mut s);  
    println!("{}", s);  
    println!("len: {}", s.len());  
}  
  
fn calculate_len(s: &String) -> usize {  
    return s.len();  
}
```

这里变量和引用都加了`mut`，才能做修改。

可变引用有一个重要的限制：**如果对一个变量有了一个可变引用，那么这个变量就不能再有其它的引用了**：

```rust
let mut s = String::from("hello");
let r1 = &mut s;
let r2 = &mut s;
println!("{}, {}", r1, r2);
```

上面的代码会报错，提示我们不能同时借用一个变量两次。

那怎么办？没得办！rust就是不让你写这样的代码，因为会产生data race。所以，我们能做的，就只能是让其中一个引用先“**消失掉**”。换句话说，要让编译器认为，这个引用已经不会再被使用了。这样另一个引用才能生效。

```rust
let mut s = String::from("hello");
{
	let r1 = &mut s;
}
let r2 = &mut s;
```

下面还有个问题，注意一下我们说的可变引用的限制：如果对一个变量有了一个可变引用，那么这个变量就不能再有其它的引用了。

这句话里暗含了一点，有了可变引用，其它所有的引用都不能存在。所以下面的代码：

```rust
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM

println!("{}, {}, and {}", r1, r2, r3);
```

这里r1和r2都是正常的不可变引用，都没问题。但是突然多了个r3。那这个时候，本来没问题的r1和r2也有问题了。所以上面的代码依然是不能执行的！

那么这个时候有什么办法呢？除了上面用大括号来**显式**地表示出一个引用的作用域，还有一个暗含的规则：**一个引用的生命周期是，从它创建那刻开始，到它最后一次被使用为止**。

这里就暗含一点，如果一直不用，那么就一直有效。所以，上面的代码中，在r3被**创建**的时候，r1和r2依然是有效的。就出现了问题。

所以，我们要做的是，让r3在使用之前，使用一下r1和r2。这样，r1和r2就有了一次使用，也就会被编译器认为在这个时候就不用了。那么这样再使用r3就没问题了：

```rust
let r1 = &s;
let r2 = &s;
println!("{}, {}", r1, r2);
let r3 = &mut s;
// println!("{}, {}", r1, r2);
println!("{}", r3)
```

上面的代码就不会报错。但是，注意注释的那一句。如果把注释打开，就又报错了。因为在r3被创建的时候，发现r1和r2在之后还会被使用，也就是依然有效，那就不行了。

下面是野指针的逻辑。c++里会有野指针的问题，比如你返回了一个栈内存的引用/指针。在函数调用结束之后，这块内存就被回收了，所以你拿回去也没用。

rust也能写出类似的代码：

```rust
fn dangle() -> &String {
    let s = String::from("hello");
    &s
}
```

会报两个错。一个是lifetime相关的，我们先不用管。另一个就是不能返回本地引用的错误。

这里我们要注意，和java是不一样的。java中new的String都是分配在堆上的，所以你返回local ref没问题，因为这个ref底层指向的内容是不会变的。

但是，看看rust的内存情况，还是之前那张图：

![[Study Log/rust_study/rust_book/resources/Pasted image 20250519004423.png|400]]

注意，这里的s,s1都是栈上的。上面代码里的s对应的是这里的s1，上面代码里的`&s`对应的是这里的s。也就是说，在`dangle()`结束之后，s1会被回收。那么，s就变成一个野引用了。这里的本质是，rust在这种情况下创建出来的引用，指向的还是栈内存。

- [ ] #TODO tasktodo1748014659797 看看c++这里是啥情况。 ➕ 2025-05-23 🔺 🆔 6s6uoo 

所以，这种情况下，直接返回string就好，也就是上图中的s1。然后会遵循浅拷贝的原则，长度等属性直接复制，然后堆内存依然是复用的。