# Slice

slice是一种特殊的reference。它可以只引用变量的一部分。最常见的就是string slice。简单来讲，就是类似于substring：

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

上面的代码很好理解。但是背后的原理关系到rust的核心思想。

不只是string，包括list等集合，我们都经常会有这样的需求，取出其中的一部分做某些事情。比如在java中：

```java
List list = new ArrayList<Student>();
// init list
list[3].xxx;
```

很常见吧。但是，一般情况下，线上的业务代码，都得check一下bound。如果你取不到那就毁了！

而slice可以将这种错误提前到编译期。

我们看下面的代码：

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```

这个函数的作用是找到一个string的第一个单词。比如`hello world`就会返回`hello`。逻辑很好看懂，唯一需要解释一个小点。

我们好像创建了一个引用的引用？传入的参数是s，本身就是一个引用类型。然后，我们返回的时候，又创建了一个引用。这样有什么问题吗？答案是没问题。最后它们都会指向同一个string。这点和java是一样的，所有的引用最后都会做拷贝，指向同一段内存。不过要注意，引用的引用和变量的引用，在borrow这一点是一样的，不能又有可变的又有不可变的。这个自己写代码测试去吧。

最后，还有一个最重要的点：注意这个函数的返回值，居然是`&str`！这代表什么？代表着：**其实`&str`类型就是string slice**。rust中只有一个string类型，那就是string slice。而String其实并不是一个核心类型，它类似java的StringBuilder和StringBuffer。

看看[[Study Log/rust_study/rust_book/rust_string|rust_string]]里的这张图：

![[Study Log/rust_study/rust_book/resources/Pasted image 20250516212709.png]]

这里的第三行，定义一个字符串常量，类型就是`&str`，这也证明了一点。`const_str`就是一个string slice。它是一个不可变的引用。只不过这个slice包含了全部的字符串而已。

现在回到刚才的问题。这个slice是一个不可变引用，也就意味着，**在使用完这个引用之前，String是不能有不可变引用的**。所以下面的代码：

```rust
fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);
    s.clear(); // error!
    println!("the first word is: {word}");
}
```

会报错。为啥？`word`是一个string slice，指向了`hello world`的一部分。然而，我们在**word依然有效**的时候，调用了`s.clear()`。用jb想都能想明白，clear既然要清空字符串，那肯定要改变s，也就一定会借用s，即创建s的可变引用。这就违反了之前介绍的规则。

这个机制，就保证了所有的string slice都可以安全使用。因为只要有人想使坏（并非故意。。）修改String中的内容，都能被编译器发现并阻止。

