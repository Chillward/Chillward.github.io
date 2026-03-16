---
title: RUST学习日记之常见集合
date: 2025-07-25 07:56:03
tags: RUST
categories: Rust
---

> RUST标准库中给我们提供了很多集合,这里介绍三种最常见的:

1. vector
2. String
3. hash map

可能是类似于CPP中的STL,标准库提供了一些别的数据类型,方便我们使用.

##　Vector

第一个类型是 `Vec<T>`，也被称为 *vector*。vector 允许我们在一个单独的数据结构中储存多个值，所有值在内存中彼此相邻排列。vector 只能储存相同类型的值。它们在拥有一系列项的场景下非常实用，例如文件中的文本行或购物车中商品的价格。

### Vector的初始化

1. `  ler v:Vec[i32] = Vec::new();` 新建一个空的Vec来存储i32类型的值.
2. ` let v = vec![1,2,3]; ` 利用vec!宏来使用初始值来创建一个Vec.此时Rust可以自动推断出v的类型.

### Vector的更新

~~~ rust
#![allow(unused)]
fn main() {
let mut v = Vec::new();
v.push(5);
v.push(6);
v.push(7);
v.push(8);
}

~~~

很明显,v.push是将值推入Vec中的方法

我们需要注意的一点是:与其他结构类似,当vector离开他的作用域时会被销毁,并且当存在一个有效的不可变引用时,是没办法通过push向vec中新增值的.

~~~ RUST
let mut v = vec![1, 2, 3, 4, 5];
let first = &v[0];
v.push(6);
println!("The first element is: {}", first);
~~~



### Vector元素的读取

1. 通过索引来读取值
2. 通过.get来读取值

~~~ rust

#![allow(unused)]
fn main() {
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
println!("The third element is {}", third);

match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}
}

~~~

这里有两个需要注意的地方。首先，我们使用索引值 `2` 来获取第三个元素，索引是从 0 开始的。其次，这两个不同的获取第三个元素的方式分别为：使用 `&` 和 `[]` 返回一个引用；或者使用 `get` 方法以索引作为参数来返回一个 `Option<&T>`。

> 使用.get方法可能是更优的,因为当索引越界时,.get方法会返回None,然后你就可以处理而不至于使程序崩溃

###　遍历Vector中的元素

如果想要依次访问 vector 中的每一个元素，我们可以遍历其所有的元素而无需通过索引一次一个的访问。

```RUST
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

我们也可以遍历可变 vector 的每一个元素的可变引用以便能改变他们。

```RUST
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

为了修改可变引用所指向的值，在使用 `+=` 运算符之前必须使用解引用运算符（`*`）获取 `i` 中的值。

### 使用枚举来储存多种类型

在本章的开始，我们提到 vector 只能储存相同类型的值。这是很不方便的；绝对会有需要储存一系列不同类型的值的用例。幸运的是，枚举的成员都被定义为相同的枚举类型，所以当需要在 vector 中储存不同类型值时，我们可以定义并使用一个枚举！

例如，假如我们想要从电子表格的一行中获取值，而这一行的有些列包含数字，有些包含浮点值，还有些是字符串。我们可以定义一个枚举，其成员会存放这些不同类型的值，同时所有这些枚举成员都会被当作相同类型，那个枚举的类型。接着可以创建一个储存枚举值的 vector，这样最终就能够储存不同类型的值了。

```RUST

enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

Rust 在编译时就必须准确的知道 vector 中类型的原因在于它需要知道储存每个元素到底需要多少内存。第二个好处是可以准确的知道这个  vector 中允许什么类型。如果 Rust 允许 vector 存放任意类型，那么当对 vector  元素执行操作时一个或多个类型的值就有可能会造成错误。使用枚举外加 `match` 意味着 Rust 能在编译时就保证总是会处理所有可能的情况，正如第 6 章讲到的那样。

如果在编写程序时不能确切无遗地知道运行时会储存进 vector 的所有类型，枚举技术就行不通了。相反，你可以使用 trait 对象，第 17 章会讲到它。

现在我们了解了一些使用 vector 的最常见的方式，请一定去看看标准库中 `Vec` 定义的很多其他实用方法的 API 文档。例如，除了 `push` 之外还有一个 `pop` 方法，它会移除并返回 vector 的最后一个元素。

## String类型

在 Rust 中，字符串并非单一类型，而是通常指两种主要的 UTF-8 编码类型：**`str`（字符串 slice）** 和 **`String`**。

具体来说，它涵盖了以下几个要点：

- **`str` (字符串 slice)**：
  - 这是 Rust **核心语言**中唯一的字符串类型。
  - 它通常以 **被借用的形式 `&str`** 出现。
  - `&str` 是对储存在**别处**（例如程序二进制输出中的字符串字面量）的 UTF-8 编码字符串数据的**引用**。它们是不可变且固定大小的。
- **`String`**：
  - 这是由 Rust **标准库提供**的类型，不属于核心语言。
  - 它是一种**可增长的、可变的、有所有权**的 UTF-8 编码字符串类型。
  - `String` 类型的数据存储在堆上，可以根据需要进行扩展和修改。
- **“字符串”的通用指代**：
  - 当 Rust 开发者（Rustacean）提到“字符串”时，通常指的是 **`String` 和 `&str` 这两种类型**，而不是特指其中一个。
  - 这两种类型在 Rust 标准库中都非常常用，且都使用 **UTF-8 编码**。
- **其他字符串类型**：
  - 除了 `String` 和 `&str`，Rust 标准库还提供了其他字符串类型，如 `OsString`、`OsStr`、`CString` 和 `CStr`。
  - 这些类型通常以 `String` 或 `Str` 结尾，分别对应其所有权（`String` 变体）和可借用（`Str` 变体）的特性。
  - 它们可能用于存储不同编码或内存表示形式的文本内容（例如与操作系统或 C 语言交互时）。

> 官方文档下头这段写得还是很像人话的,我们之间复制过来,需要注意的是,rust中的字符串默认是UTF-8而不是ASCII,所以会出现一些在C语言中不会出现的问题,C语言中与之对应的知识是wchar,即宽字符.

### [新建字符串](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#新建字符串)

很多 `Vec` 可用的操作在 `String` 中同样可用，从 `new` 函数创建字符串开始，如示例 8-11 所示。

```rust
let mut s = String::new();
```

示例 8-11：新建一个空的 `String`

这新建了一个叫做 `s` 的空的字符串，接着我们可以向其中装载数据。通常字符串会有初始数据，因为我们希望一开始就有这个字符串。为此，可以使用 `to_string` 方法，它能用于任何实现了 `Display` trait 的类型，字符串字面量也实现了它。示例 8-12 展示了两个例子。

```rust
let data = "initial contents";

let s = data.to_string();

// 该方法也可直接用于字符串字面量：
let s = "initial contents".to_string();
```

示例 8-12：使用 `to_string` 方法从字符串字面量创建 `String`

这些代码会创建包含 `initial contents` 的字符串。

也可以使用 `String::from` 函数来从字符串字面量创建 `String`。示例 8-13 中的代码等同于使用 `to_string`。

```rust
let s = String::from("initial contents");
```

示例 8-13：使用 `String::from` 函数从字符串字面量创建 `String`

因为字符串应用广泛，这里有很多不同的用于字符串的通用 API 可供选择。其中一些可能看起来多余，不过都有其用武之地！在这个例子中，`String::from` 和 `to_string` 最终做到了完全相同的事情，所以如何选择，就是风格问题了。

请记住，字符串是 UTF-8 编码的，所以可以包含任何正确编码的数据，如示例 8-14 所示。

```rust
let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("Hello");
let hello = String::from("שָׁלוֹם");
let hello = String::from("नमस्ते");
let hello = String::from("こんにちは");
let hello = String::from("안녕하세요");
let hello = String::from("你好");
let hello = String::from("Olá");
let hello = String::from("Здравствуйте");
let hello = String::from("Hola");
```

示例 8-14：在字符串中储存不同语言的问候语

所有这些都是有效的 `String` 值。

### [更新字符串](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#更新字符串)

`String` 的大小可以增加，其内容也可以改变，就像可以放入更多数据来改变 `Vec` 的内容一样。另外，可以方便的使用 `+` 运算符或 `format!` 宏来拼接 `String` 值。

#### [使用 `push_str` 和 `push` 附加字符串](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#使用-push_str-和-push-附加字符串)

可以通过 `push_str` 方法来附加字符串 slice，从而使 `String` 变长，如示例 8-15 所示。

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

示例 8-15：使用 `push_str` 方法向 `String` 附加字符串 slice

执行这两行代码之后，`s` 将会包含 `foobar`。`push_str` 方法采用字符串 slice，因为我们并不需要获取参数的所有权。例如，示例 8-16 展示了如果将 `s2` 的内容附加到 `s1` 之后，自身不能被使用就糟糕了。

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {}", s2);
```

示例 8-16：将字符串 slice 的内容附加到 `String` 后使用它

如果 `push_str` 方法获取了 `s2` 的所有权，就不能在最后一行打印出其值了。好在代码如我们期望那样工作！

`push` 方法被定义为获取一个单独的字符作为参数，并附加到 `String` 中。示例 8-17 展示了使用 `push` 方法将字母 *l* 加入 `String` 的代码。

```rust
let mut s = String::from("lo");
s.push('l');
```

示例 8-17：使用 `push` 将一个字符加入 `String` 值中

执行这些代码之后，`s` 将会包含 `lol`。

#### [使用 `+` 运算符或 `format!` 宏拼接字符串](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#使用--运算符或-format-宏拼接字符串)

通常你会希望将两个已知的字符串合并在一起。一种办法是像这样使用 `+` 运算符，如示例 8-18 所示。

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // 注意 s1 被移动了，不能继续使用
```

示例 8-18：使用 `+` 运算符将两个 `String` 值合并到一个新的 `String` 值中

执行完这些代码之后，字符串 `s3` 将会包含 `Hello, world!`。`s1` 在相加后不再有效的原因，和使用 `s2` 的引用的原因，与使用 `+` 运算符时调用的函数签名有关。`+` 运算符使用了 `add` 函数，这个函数签名看起来像这样：

```rust
fn add(self, s: &str) -> String {
```

这并不是标准库中实际的签名；标准库中的 `add` 使用泛型定义。这里我们看到的 `add` 的签名使用具体类型代替了泛型，这也正是当使用 `String` 值调用这个方法会发生的。第 10 章会讨论泛型。这个签名提供了理解 `+` 运算那微妙部分的线索。

首先，`s2` 使用了 `&`，意味着我们使用第二个字符串的 **引用** 与第一个字符串相加。这是因为 `add` 函数的 `s` 参数：只能将 `&str` 和 `String` 相加，不能将两个 `String` 值相加。不过等一下 —— 正如 `add` 的第二个参数所指定的，`&s2` 的类型是 `&String` 而不是 `&str`。那么为什么示例 8-18 还能编译呢？

之所以能够在 `add` 调用中使用 `&s2` 是因为 `&String` 可以被 **强转**（*coerced*）成 `&str`。当 `add` 函数被调用时，Rust 使用了一个被称为 **解引用强制转换**（*deref coercion*）的技术，你可以将其理解为它把 `&s2` 变成了 `&s2[..]`。第 15 章会更深入的讨论解引用强制转换。因为 `add` 没有获取参数的所有权，所以 `s2` 在这个操作后仍然是有效的 `String`。

其次，可以发现签名中 `add` 获取了 `self` 的所有权，因为 `self` **没有** 使用 `&`。这意味着示例 8-18 中的 `s1` 的所有权将被移动到 `add` 调用中，之后就不再有效。虽然 `let s3 = s1 + &s2;` 看起来就像它会复制两个字符串并创建一个新的字符串，而实际上这个语句会获取 `s1` 的所有权，附加上从 `s2` 中拷贝的内容，并返回结果的所有权。换句话说，它看起来好像生成了很多拷贝，不过实际上并没有：这个实现比拷贝要更高效。

如果想要级联多个字符串，`+` 的行为就显得笨重了：

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

这时 `s` 的内容会是 `tic-tac-toe`。在有这么多 `+` 和 `"` 字符的情况下，很难理解具体发生了什么。对于更为复杂的字符串连接，可以使用 `format!` 宏：

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

这些代码也会将 `s` 设置为 “tic-tac-toe”。`format!` 与 `println!` 的工作原理相同，不过不同于将输出打印到屏幕上，它返回一个带有结果内容的 `String`。这个版本就好理解的多，并且不会获取任何参数的所有权。

### [索引字符串](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#索引字符串)

在很多语言中，通过索引来引用字符串中的单独字符是有效且常见的操作。然而在 Rust 中，如果你尝试使用索引语法访问 `String` 的一部分，会出现一个错误。考虑一下如示例 8-19 中所示的无效代码。

```rust
let s1 = String::from("hello");
let h = s1[0];
```

示例 8-19：尝试对字符串使用索引语法

这段代码会导致如下错误：

```
error[E0277]: the trait bound `std::string::String: std::ops::Index<{integer}>` is not satisfied
 -->
  |
3 |     let h = s1[0];
  |             ^^^^^ the type `std::string::String` cannot be indexed by `{integer}`
  |
  = help: the trait `std::ops::Index<{integer}>` is not implemented for `std::string::String`
```

错误和提示说明了全部问题：Rust 的字符串不支持索引。那么接下来的问题是，为什么不支持呢？为了回答这个问题，我们必须先聊一聊 Rust 是如何在内存中储存字符串的。

#### [内部表现](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#内部表现)

`String` 是一个 `Vec<u8>` 的封装。让我们看看示例 8-14 中一些正确编码的字符串的例子。首先是这一个：

```rust
let len = String::from("Hola").len();
```

在这里，`len` 的值是 4 ，这意味着储存字符串 “Hola” 的 `Vec` 的长度是 4 个字节：这里每一个字母的 UTF-8 编码都占用 1 个字节。那下面这个例子又如何呢？（注意这个字符串中的首字母是西里尔字母的 Ze，而不是阿拉伯数字 3 。）

```rust
let len = String::from("Здравствуйте").len();
```

当问及这个字符是多长的时候有人可能会说是 12。然而，Rust 的回答是 24。这是使用 UTF-8 编码 “Здравствуйте”  所需要的字节数，这是因为每个 Unicode 标量值需要 2 个字节存储。因此一个字符串字节值的索引并不总是对应一个有效的 Unicode  标量值。作为演示，考虑如下无效的 Rust 代码：

```rust
let hello = "Здравствуйте";
let answer = &hello[0];
```

`answer` 的值应该是什么呢？它应该是第一个字符 `З` 吗？当使用 UTF-8 编码时，`З` 的第一个字节 `208`，第二个是 `151`，所以 `answer` 实际上应该是 `208`，不过 `208` 自身并不是一个有效的字母。返回 `208` 可不是一个请求字符串第一个字母的人所希望看到的，不过它是 Rust 在字节索引 0 位置所能提供的唯一数据。用户通常不会想要一个字节值被返回，即便这个字符串只有拉丁字母： 即便 `&"hello"[0]` 是返回字节值的有效代码，它也应当返回 `104` 而不是 `h`。为了避免返回意外的值并造成不能立刻发现的 bug，Rust 根本不会编译这些代码，并在开发过程中及早杜绝了误会的发生。

#### [字节、标量值和字形簇！天呐！](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#字节标量值和字形簇天呐)

这引起了关于 UTF-8 的另外一个问题：从 Rust 的角度来讲，事实上有三种相关方式可以理解字符串：字节、标量值和字形簇（最接近人们眼中 **字母** 的概念）。

比如这个用梵文书写的印度语单词 “नमस्ते”，最终它储存在 vector 中的 `u8` 值看起来像这样：

```
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164, 224, 165, 135]
```

这里有 18 个字节，也就是计算机最终会储存的数据。如果从 Unicode 标量值的角度理解它们，也就像 Rust 的 `char` 类型那样，这些字节看起来像这样：

```
['न', 'म', 'स', '्', 'त', 'े']
```

这里有六个 `char`，不过第四个和第六个都不是字母，它们是发音符号本身并没有任何意义。最后，如果以字形簇的角度理解，就会得到人们所说的构成这个单词的四个字母：

```
["न", "म", "स्", "ते"]
```

Rust 提供了多种不同的方式来解释计算机储存的原始字符串数据，这样程序就可以选择它需要的表现方式，而无所谓是何种人类语言。

最后一个 Rust 不允许使用索引获取 `String` 字符的原因是，索引操作预期总是需要常数时间 (O(1))。但是对于 `String` 不可能保证这样的性能，因为 Rust 必须从开头到索引位置遍历来确定有多少有效的字符。

### [字符串 slice](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#字符串-slice)

索引字符串通常是一个坏点子，因为字符串索引应该返回的类型是不明确的：字节值、字符、字形簇或者字符串  slice。因此，如果你真的希望使用索引创建字符串 slice 时，Rust 会要求你更明确一些。为了更明确索引并表明你需要一个字符串  slice，相比使用 `[]` 和单个值的索引，可以使用 `[]` 和一个 range 来创建含特定字节的字符串 slice：

```rust
let hello = "Здравствуйте";
let s = &hello[0..4];
```

这里，`s` 会是一个 `&str`，它包含字符串的头 4 个字节。早些时候，我们提到了这些字母都是 2 个字节长的，所以这意味着 `s` 将会是 “Зд”。

如果获取 `&hello[0..1]` 会发生什么呢？答案是：Rust 在运行时会 panic，就跟访问 vector 中的无效索引时一样：

```
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/libcore/str/mod.rs:2188:4
```

你应该小心谨慎的使用这个操作，因为这么做可能会使你的程序崩溃。

### [遍历字符串的方法](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#遍历字符串的方法)

幸运的是，这里还有其他获取字符串元素的方式。

如果你需要操作单独的 Unicode 标量值，最好的选择是使用 `chars` 方法。对 “नमस्ते” 调用 `chars` 方法会将其分开并返回六个 `char` 类型的值，接着就可以遍历其结果来访问每一个元素了：

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

这些代码会打印出如下内容：

```rust
न
म
स
्
त
े
```

`bytes` 方法返回每一个原始字节，这可能会适合你的使用场景：

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

这些代码会打印出组成 `String` 的 18 个字节：

```rust
224
164
// --snip--
165
135
```

不过请记住有效的 Unicode 标量值可能会由不止一个字节组成。

从字符串中获取字形簇是很复杂的，所以标准库并没有提供这个功能。[crates.io](https://crates.io) 上有些提供这样功能的 crate。

### [字符串并不简单](https://rustwiki.org/zh-CN/book/ch08-02-strings.html#字符串并不简单)

总而言之，字符串还是很复杂的。不同的语言选择了不同的向开发者展示其复杂性的方式。Rust 选择了以准确的方式处理 `String` 数据作为所有 Rust 程序的默认行为，这意味着开发者们必须更多的思考如何预先处理 UTF-8 数据。这种权衡取舍相比其他语言更多的暴露出了字符串的复杂性，不过也使你在开发生命周期后期免于处理涉及非 ASCII 字符的错误。

> 哈希map,我在C语言编程时没怎么接触过这个东西,所以不太熟悉,等用到的时候再回来看吧

## [哈希 map 储存键值对](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#哈希-map-储存键值对)

最后介绍的常用集合类型是 **哈希 map**（*hash map*）。`HashMap<K, V>` 类型储存了一个键类型 `K` 对应一个值类型 `V` 的映射。它通过一个 **哈希函数**（*hashing function*）来实现映射，决定如何将键和值放入内存中。很多编程语言支持这种数据结构，不过通常有不同的名字：哈希、map、对象、哈希表或者关联数组，仅举几例。

哈希 map 可以用于需要任何类型作为键来寻找数据的情况，而不是像 vector 那样通过索引。例如，在一个游戏中，你可以将每个团队的分数记录到哈希 map 中，其中键是队伍的名字而值是每个队伍的分数。给出一个队名，就能得到他们的得分。

本章我们会介绍哈希 map 的基本 API，不过还有更多吸引人的功能隐藏于标准库在 `HashMap<K, V>` 上定义的函数中。一如既往请查看标准库文档来了解更多信息。

### [新建一个哈希 map](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#新建一个哈希-map)

可以使用 `new` 创建一个空的 `HashMap`，并使用 `insert` 增加元素。在示例 8-20 中我们记录两支队伍的分数，分别是蓝队和黄队。蓝队开始有 10 分而黄队开始有 50 分：

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

示例 8-20：新建一个哈希 map 并插入一些键值对

注意必须首先 `use` 标准库中集合部分的 `HashMap`。在这三个常用集合中，`HashMap` 是最不常用的，所以并没有被 prelude 自动引用。标准库中对 `HashMap` 的支持也相对较少，例如，并没有内建的构建宏。

像 vector 一样，哈希 map 将它们的数据储存在堆上，这个 `HashMap` 的键类型是 `String` 而值类型是 `i32`。类似于 vector，哈希 map 是同质的：所有的键必须是相同类型，值也必须都是相同类型。

另一个构建哈希 map 的方法是使用一个元组的 vector 的 `collect` 方法，其中每个元组包含一个键值对。`collect` 方法可以将数据收集进一系列的集合类型，包括 `HashMap`。例如，如果队伍的名字和初始分数分别在两个 vector 中，可以使用 `zip` 方法来创建一个元组的 vector，其中 “Blue” 与 10 是一对，依此类推。接着就可以使用 `collect` 方法将这个元组 vector 转换成一个 `HashMap`，如示例 8-21 所示：

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

示例 8-21：用队伍列表和分数列表创建哈希 map

这里 `HashMap<_, _>` 类型标注是必要的，因为 `collect` 有可能当成多种不同的数据结构，而除非显式指定否则 Rust 无从得知你需要的类型。但是对于键和值的类型参数来说，可以使用下划线占位，而 Rust 能够根据 vector 中数据的类型推断出 `HashMap` 所包含的类型。

### [哈希 map 和所有权](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#哈希-map-和所有权)

对于像 `i32` 这样的实现了 `Copy` trait 的类型，其值可以拷贝进哈希 map。对于像 `String` 这样拥有所有权的值，其值将被移动而哈希 map 会成为这些值的所有者，如示例 8-22 所示：

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// 这里 field_name 和 field_value 不再有效，
// 尝试使用它们看看会出现什么编译错误！
```

示例 8-22：展示一旦键值对被插入后就为哈希 map 所拥有

当 `insert` 调用将 `field_name` 和 `field_value` 移动到哈希 map 中后，将不能使用这两个绑定。

如果将值的引用插入哈希 map，这些值本身将不会被移动进哈希 map。但是这些引用指向的值必须至少在哈希 map 有效时也是有效的。第 10 章 [“生命周期与引用有效性”](https://rustwiki.org/zh-CN/book/ch10-03-lifetime-syntax.html#validating-references-with-lifetimes) 部分将会更多的讨论这个问题。

### [访问哈希 map 中的值](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#访问哈希-map-中的值)

可以通过 `get` 方法并提供对应的键来从哈希 map 中获取值，如示例 8-23 所示：

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

示例 8-23：访问哈希 map 中储存的蓝队分数

这里，`score` 是与蓝队分数相关的值，应为 `Some(10)`。因为 `get` 返回 `Option<V>`，所以结果被装进 `Some`；如果某个键在哈希 map 中没有对应的值，`get` 会返回 `None`。这时就要用某种第 6 章提到的方法之一来处理 `Option`。

可以使用与 vector 类似的方式来遍历哈希 map 中的每一个键值对，也就是 `for` 循环：

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

这会以任意顺序打印出每一个键值对：

```rust
Yellow: 50
Blue: 10
```

### [更新哈希 map](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#更新哈希-map)

尽管键值对的数量是可以增长的，不过任何时候，每个键只能关联一个值。当我们想要改变哈希 map 中的数据时，必须决定如何处理一个键已经有值了的情况。可以选择完全无视旧值并用新值代替旧值。可以选择保留旧值而忽略新值，并只在键 **没有** 对应值时增加新值。或者可以结合新旧两值。让我们看看这分别该如何处理！

#### [覆盖一个值](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#覆盖一个值)

如果我们插入了一个键值对，接着用相同的键插入一个不同的值，与这个键相关联的旧值将被替换。即便示例 8-24 中的代码调用了两次 `insert`，哈希 map 也只会包含一个键值对，因为两次都是对蓝队的键插入的值：

```rust

use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

示例 8-24：替换以特定键储存的值

这会打印出 `{"Blue": 25}`。原始的值 `10` 则被覆盖了。

#### [只在键没有对应值时插入](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#只在键没有对应值时插入)

我们经常会检查某个特定的键是否有值，如果没有就插入一个值。为此哈希 map 有一个特有的 API，叫做 `entry`，它获取我们想要检查的键作为参数。`entry` 函数的返回值是一个枚举，`Entry`，它代表了可能存在也可能不存在的值。比如说我们想要检查黄队的键是否关联了一个值。如果没有，就插入值 50，对于蓝队也是如此。使用 entry API 的代码看起来像示例 8-25 这样：

```rust

use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

示例 8-25：使用 `entry` 方法只在键没有对应一个值时插入

`Entry` 的 `or_insert` 方法在键对应的值存在时就返回这个值的可变引用，如果不存在则将参数作为新值插入并返回新值的可变引用。这比编写自己的逻辑要简明的多，另外也与借用检查器结合得更好。

运行示例 8-25 的代码会打印出 `{"Yellow": 50, "Blue": 10}`。第一个 `entry` 调用会插入黄队的键和值 `50`，因为黄队并没有一个值。第二个 `entry` 调用不会改变哈希 map 因为蓝队已经有了值 `10`。

#### [根据旧值更新一个值](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#根据旧值更新一个值)

另一个常见的哈希 map 的应用场景是找到一个键对应的值并根据旧的值更新它。例如，示例 8-26  中的代码计数一些文本中每一个单词分别出现了多少次。我们使用哈希 map  以单词作为键并递增其值来记录我们遇到过几次这个单词。如果是第一次看到某个单词，就插入值 `0`。

```rust

use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

示例 8-26：通过哈希 map 储存单词和计数来统计出现次数

这会打印出 `{"world": 2, "hello": 1, "wonderful": 1}`，`or_insert` 方法事实上会返回这个键的值的一个可变引用（`&mut V`）。这里我们将这个可变引用储存在 `count` 变量中，所以为了赋值必须首先使用星号（`*`）解引用 `count`。这个可变引用在 `for` 循环的结尾离开作用域，这样所有这些改变都是安全的并符合借用规则。

### [哈希函数](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#哈希函数)

`HashMap` 默认使用一种 “密码学安全的”（“cryptographically strong” ）[1](https://rustwiki.org/zh-CN/book/ch08-03-hash-maps.html#siphash) 哈希函数，它可以抵抗拒绝服务（Denial of Service, DoS）攻击。然而这并不是可用的最快的算法，不过为了更高的安全性值得付出一些性能的代价。如果性能监测显示此哈希函数非常慢，以致于你无法接受，你可以指定一个不同的 *hasher* 来切换为其它函数。hasher 是一个实现了 `BuildHasher` trait 的类型。第 10 章会讨论 trait 和如何实现它们。你并不需要从头开始实现你自己的 hasher；[crates.io](https://crates.io) 有其他人分享的实现了许多常用哈希算法的 hasher 的库。
