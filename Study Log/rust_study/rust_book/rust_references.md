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
> 其实，这样的模式才是Java中String的默认方式。

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