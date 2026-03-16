---
title: RUST学习日记之枚举和结构匹配
date: 2025-07-21 16:34:40
tags :RUST
categories: Rust
---

RUST中的枚举远比C语言中的强大的多。

~~~rust
enum IpAddrKind {
    V4,
    V6,
}

fn main() {
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;

    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
}

fn route(ip_kind: IpAddrKind) {}
~~~

第一个巨大的不同之处就是:RUST的枚举可以关联不同类型和数量的数据

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));
```

这无疑是非常灵活方便的，当然RUST的枚举可以关联结构体，甚至可以再关联一个枚举。

RUST还允许我们像给结构体定义方法一样给枚举定义方法
~~~rust
fn main() {
    enum Message {
        Quit,
        Move { x: i32, y: i32 },
        Write(String),
        ChangeColor(i32, i32, i32),
    }

    impl Message {
        fn call(&self) {
            // 在这里定义方法体
        }
    }
    let m = Message::Write(String::from("hello"));
    m.call();
}
~~~



### Option枚举

RUST中没有类似于C语言的NULL这种东西，在C语言中，假设你malloc/calloc一个内存区域，如果失败了，会返回一个NULL，调用者需要手动处理NULL的情况，不然程序就会出现bug，而RUST通过Option枚举避免了这种问题。

问题不在于概念而在于具体的实现。为此，Rust 并没有空值，不过它确实拥有一个可以编码存在或不存在概念的枚举。这个枚举是 `Option<T>`，而且它[定义于标准库中](https://doc.rust-lang.org/std/option/enum.Option.html)，如下：

```rust
#![allow(unused)]
fn main() {
enum Option<T> {
    None,
    Some(T),
}
}
```

`Option<T>` 枚举是如此有用以至于它甚至被包含在了 prelude 之中，无需将其显式引入作用域。另外，它的变体也是如此：可以不需要 `Option::` 前缀来直接使用 `Some` 和 `None`。即便如此 `Option<T>` 也仍是常规的枚举，`Some(T)` 和 `None` 仍是 `Option<T>` 的变体。

`<T>` 语法是一个我们还未讲到的 Rust 功能。它是一个泛型类型参数，第十章会更详细的讲解泛型。目前，所有你需要知道的就是 `<T>` 意味着 `Option` 枚举的 `Some` 变体可以包含任意类型的数据，同时每一个用于 `T` 位置的具体类型使得 `Option<T>` 整体作为不同的类型。这里是一些包含数字类型和字符类型 `Option` 值的例子：

```rust
    let some_number = Some(5);
    let some_char = Some('e');

    let absent_number: Option<i32> = None;
```

`some_number` 的类型是 `Option<i32>`。`some_char` 的类型是 `Option<char>`，是不同于 `some_number` 的类型。因为我们在 `Some` 变体中指定了值，Rust 可以推断其类型。对于 `absent_number`，Rust 需要我们指定 `Option` 整体的类型，因为编译器只通过 `None` 值无法推断出 `Some` 变体保存的值的类型。这里我们告诉 Rust 希望 `absent_number` 是 `Option<i32>` 类型的。

当有一个 `Some` 值时，我们就知道存在一个值，而这个值保存在 `Some` 中。当有个 `None` 值时，在某种意义上，它跟空值具有相同的意义：并没有一个有效的值。那么，`Option<T>` 为什么就比空值要好呢？

简而言之，因为 `Option<T>` 和 `T`（这里 `T` 可以是任何类型）是不同的类型，编译器不允许像一个肯定有效的值那样使用 `Option<T>`。例如，这段代码不能编译，因为它尝试将 `Option<i8>` 与 `i8` 相加：

```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
```

如果运行这些代码，将得到类似这样的错误信息：

```console
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`
  = help: the following other types implement trait `Add<Rhs>`:
            `&i8` implements `Add<i8>`
            `&i8` implements `Add`
            `i8` implements `Add<&i8>`
            `i8` implements `Add`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `enums` (bin "enums") due to 1 previous error
```

很好！事实上，错误信息意味着 Rust 不知道该如何将 `Option<i8>` 与 `i8` 相加，因为它们的类型不同。当在 Rust 中拥有一个像 `i8` 这样类型的值时，编译器确保它总是有一个有效的值。我们可以自信地使用而无需做空值检查。只有当使用 `Option<i8>`（或者任何用到的类型）的时候需要担心可能没有值，而编译器会确保我们在使用值之前处理了为空的情况。

换句话说，在对 `Option<T>` 进行运算之前必须将其转换为 `T`。通常这能帮助我们捕获到空值最常见的问题之一：假设某值不为空但实际上为空的情况。

消除了错误地假设一个非空值的风险，会让你对代码更加有信心。为了拥有一个可能为空的值，你必须要显式的将其放入对应类型的 `Option<T>` 中。接着，当使用这个值时，必须明确的处理值为空的情况。只要一个值不是 `Option<T>` 类型，你就**可以**安全的认定它的值不为空。这是 Rust 的一个经过深思熟虑的设计决策，来限制空值的泛滥以增加 Rust 代码的安全性。

那么当有一个 `Option<T>` 的值时，如何从 `Some` 变体中取出 `T` 的值来使用它呢？`Option<T>` 枚举拥有大量用于各种情况的方法：你可以查看[它的文档](https://doc.rust-lang.org/std/option/enum.Option.html)。熟悉 `Option<T>` 的方法将对你的 Rust 之旅非常有用。

总的来说，为了使用 `Option<T>` 值，需要编写处理每个变体的代码。你想要一些代码只当拥有 `Some(T)` 值时运行，允许这些代码使用其中的 `T`。也希望一些代码只在值为 `None` 时运行，这些代码并没有一个可用的 `T` 值。`match` 表达式就是这么一个处理枚举的控制流结构：它会根据枚举的变体运行不同的代码，这些代码可以使用匹配到的值中的数据。

除了match跟if let外，还有以下方法可以提取Option<>中的值

###  `unwrap()` 和 `expect()`

`unwrap()` 和 `expect()` 是提取 `Option` 值的直接方法，但它们是不安全的，因为它们在 `Option` 是 `None` 时会**panic**（导致程序崩溃）。

- **`unwrap()`**: 如果 `Option` 是 `Some(T)`，它会返回 `T`。如果 `Option` 是 `None`，它会 panic。

  Rust

  ```
  fn main() {
      let some_value = Some(100);
      let value = some_value.unwrap(); // value is 100
      println!("Unwrapped value: {}", value);
  
      let none_value: Option<i32> = None;
      // let another_value = none_value.unwrap(); // 这行代码会导致 panic!
      // println!("Another unwrapped value: {}", another_value);
  }
  ```

- **`expect("自定义错误信息")`**: 和 `unwrap()` 类似，但允许你提供一个自定义的 panic 错误信息，这在调试时很有用。

  Rust

  ```
  fn main() {
      let file_content = Some(String::from("Some text in the file."));
      let content = file_content.expect("文件内容不存在！");
      println!("文件内容: {}", content);
  
      let empty_file: Option<String> = None;
      // let empty_content = empty_file.expect("读取文件失败，文件为空或不存在。"); // 这行代码会导致 panic!
  }
  ```

**重要提示：** 除非你百分之百确定 `Option` 永远不会是 `None`（或者在 `None` 的情况下程序崩溃是可以接受的），否则**应避免使用 `unwrap()` 和 `expect()`**。它们主要用于原型开发、测试或在明确知道 `None` 是一个不可恢复的错误时。

### `unwrap_or()`, `unwrap_or_default()`, `unwrap_or_else()`

这些方法提供了在 `Option` 是 `None` 时提供一个默认值或通过闭包计算一个默认值的方式，而不会 panic。

- **`unwrap_or(default_value)`**: 如果 `Option` 是 `Some(T)`，返回 `T`。如果 `Option` 是 `None`，返回提供的 `default_value`。

  ```rust
  fn main() {
      let user_input = Some("Hello".to_string());
      let result = user_input.unwrap_or("Guest".to_string());
      println!("欢迎: {}", result); // 输出: 欢迎: Hello
  
      let no_input: Option<String> = None;
      let result_none = no_input.unwrap_or("Guest".to_string());
      println!("欢迎: {}", result_none); // 输出: 欢迎: Guest
  }
  ```

- **`unwrap_or_default()`**: 如果 `Option` 是 `Some(T)`，返回 `T`。如果 `Option` 是 `None`，返回 `T` 类型的默认值（要求 `T` 实现 `Default` trait）。

  ```rust
  fn main() {
      let count = Some(5);
      let num = count.unwrap_or_default(); // num 是 5
      println!("Count: {}", num);
  
      let no_count: Option<u32> = None;
      let default_num = no_count.unwrap_or_default(); // default_num 是 0 (u32 的默认值)
      println!("Default Count: {}", default_num);
  }
  ```

- **`unwrap_or_else(|| { /\* 闭包计算默认值 \*/ })`**: 如果 `Option` 是 `Some(T)`，返回 `T`。如果 `Option` 是 `None`，执行提供的闭包并返回其结果。这在计算默认值比较复杂或有副作用时很有用。

  ```rust
  fn get_default_username() -> String {
      println!("正在生成默认用户名...");
      "Anonymous".to_string()
  }
  
  fn main() {
      let username = Some("Alice".to_string());
      let final_username = username.unwrap_or_else(|| get_default_username());
      println!("用户: {}", final_username);
  
      let no_username: Option<String> = None;
      let final_no_username = no_username.unwrap_or_else(|| get_default_username());
      println!("用户: {}", final_no_username);
  }
  ```

### 5. 其他辅助方法

`Option` 还提供了许多其他有用的方法，例如：

- **`is_some()`**: 返回 `true` 如果是 `Some`，否则返回 `false`。
- **`is_none()`**: 返回 `true` 如果是 `None`，否则返回 `false`。
- **`map(|value| new_value)`**: 如果是 `Some(T)`，应用闭包到 `T` 并返回 `Some(U)`；如果是 `None`，返回 `None`。用于转换 `Option` 内的值类型。
- **`and_then(|value| Option<U>)`**: 如果是 `Some(T)`，应用闭包到 `T`（闭包返回另一个 `Option`）并返回结果；如果是 `None`，返回 `None`。常用于链式处理多个可能失败的操作。

## [`match` 控制流结构](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#match-控制流结构)

Rust 有一个叫做 `match` 的极为强大的控制流运算符，它允许我们将一个值与一系列的模式相比较，并根据相匹配的模式执行相应代码。模式可由字面值、变量、通配符和许多其他内容构成；[第十九章](https://kaisery.github.io/trpl-zh-cn/ch19-00-patterns.html)会涉及到所有不同种类的模式以及它们的作用。`match` 的力量来源于模式的表现力，以及编译器能够确认所有可能情况均已被覆盖。

可以把 `match` 表达式想象成某种硬币分类器：硬币滑入有着不同大小孔洞的轨道，每一个硬币都会掉入符合它大小的孔洞。同样地，值也会通过 `match` 的每一个模式，并且在遇到第一个 “符合” 的模式时，值会进入相关联的代码块并在执行中被使用。

因为刚刚提到了硬币，让我们用它们来作为一个使用 `match` 的例子！我们可以编写一个函数来获取一个未知的美国硬币，并以一种类似验钞机的方式，确定它是何种硬币并返回它的美分值，如示例 6-3 中所示。

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

示例 6-3：一个枚举和一个以枚举变体作为模式的 `match` 表达式

拆开 `value_in_cents` 函数中的 `match` 来看。首先，我们列出 `match` 关键字后跟一个表达式，在这个例子中是 `coin` 的值。这看起来非常像 `if` 所使用的条件表达式，不过这里有一个非常大的区别：对于 `if`，表达式必须返回一个布尔值，而这里它可以是任何类型的。例子中的 `coin` 的类型是示例 6-3 中定义的 `Coin` 枚举。

接下来是 `match` 的分支。一个分支有两个部分：一个模式和一些代码。第一个分支的模式是值 `Coin::Penny` 而之后的 `=>` 运算符将模式和将要运行的代码分开。这里的代码就仅仅是值 `1`。每一个分支之间使用逗号分隔。

当 `match` 表达式执行时，它将结果值按顺序与每一个分支的模式相比较。如果模式匹配了这个值，这个模式相关联的代码将被执行。如果模式并不匹配这个值，将继续执行下一个分支，非常类似一个硬币分类器。可以拥有任意多的分支：示例 6-3 中的 `match` 有四个分支。

每个分支相关联的代码是一个表达式，而表达式的结果值将作为整个 `match` 表达式的返回值。

如果分支代码较短的话通常不使用大括号，正如示例 6-3 中的每个分支都只是返回一个值。如果想要在分支中运行多行代码，可以使用大括号，而分支后的逗号是可选的。例如，如下代码在每次使用`Coin::Penny` 调用时都会打印出 “Lucky penny!”，同时仍然返回代码块最后的值，`1`：

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### [绑定值的模式](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#绑定值的模式)

匹配分支的另一个有用的功能是可以绑定匹配的模式的部分值。这也就是如何从枚举变体中提取值的。

作为一个例子，让我们修改枚举的一个变体来存放数据。1999 年到 2008 年间，美国在 25 美分的硬币的一侧为 50 个州的每一个都印刷了不同的设计。其他的硬币都没有这种区分州的设计，所以只有这些 25 美分硬币有特殊的价值。可以将这些信息加入我们的 `enum`，通过改变 `Quarter` 变体来包含一个 `State` 值，示例 6-4 中完成了这些修改：

```rust
#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

示例 6-4：`Quarter` 变体也存放了一个 `UsState` 值的 `Coin` 枚举

想象一下我们的一个朋友尝试收集所有 50 个州的 25 美分硬币。在根据硬币类型分类零钱的同时，也可以报告出每个 25 美分硬币所对应的州名称，这样如果我们的朋友没有的话，他可以将其加入收藏。

在这些代码的匹配表达式中，我们在匹配 `Coin::Quarter` 变体的分支的模式中增加了一个叫做 `state` 的变量。当匹配到 `Coin::Quarter` 时，变量 `state` 将会绑定 25 美分硬币所对应州的值。接着在那个分支的代码中使用 `state`，如下：

```rust
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {state:?}!");
            25
        }
    }
}
```

如果调用 `value_in_cents(Coin::Quarter(UsState::Alaska))`，`coin` 将是 `Coin::Quarter(UsState::Alaska)`。当将值与每个分支相比较时，没有分支会匹配，直到遇到 `Coin::Quarter(state)`。这时，`state` 绑定的将会是值 `UsState::Alaska`。接着就可以在 `println!` 表达式中使用这个绑定了，像这样就可以获取 `Coin` 枚举的 `Quarter` 变体中内部的州的值。

### [匹配 `Option`](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#匹配-optiont)

我们在之前的部分中使用 `Option<T>` 时，是为了从 `Some` 中取出其内部的 `T` 值；我们还可以像处理 `Coin` 枚举那样使用 `match` 处理 `Option<T>`！只不过这回比较的不再是硬币，而是 `Option<T>` 的变体，但 `match` 表达式的工作方式保持不变。

比如我们想要编写一个函数，它获取一个 `Option<i32>` ，如果其中含有一个值，将其加一。如果其中没有值，函数应该返回 `None` 值，而不尝试执行任何操作。

得益于 `match`，编写这个函数非常简单，它将看起来像示例 6-5 中这样：

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
```

示例 6-5：一个在 `Option<i32>` 上使用 `match` 表达式的函数

让我们更仔细地检查 `plus_one` 的第一行操作。当调用 `plus_one(five)` 时，`plus_one` 函数体中的 `x` 将会是值 `Some(5)`。接着将其与每个分支比较。

```rust
            None => None,
```

值 `Some(5)` 并不匹配模式 `None`，所以继续进行下一个分支。

```rust
            Some(i) => Some(i + 1),
```

`Some(5)` 与 `Some(i)` 匹配吗？当然匹配！它们是相同的变体。`i` 绑定了 `Some` 中包含的值，所以 `i` 的值是 `5`。接着匹配分支的代码被执行，所以我们将 `i` 的值加一并返回一个含有值 `6` 的新 `Some`。

接着考虑下示例 6-5 中 `plus_one` 的第二个调用，这里 `x` 是 `None`。我们进入 `match` 并与第一个分支相比较。

```rust
            None => None,
```

匹配成功！这里没有值来加一，所以程序结束并返回 `=>` 右侧的值 `None`，因为第一个分支就匹配到了，其他的分支将不再比较。

将 `match` 与枚举相结合在很多场景中都是有用的。你会在 Rust 代码中看到很多这样的模式：`match` 一个枚举，绑定其中的值到一个变量，接着根据其值执行代码。这在一开始有点复杂，不过一旦习惯了，你会希望所有语言都拥有它！这一直是用户的最爱。

### [匹配是穷尽的](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#匹配是穷尽的)

`match` 还有另一方面需要讨论：这些分支必须覆盖了所有的可能性。考虑一下 `plus_one` 函数的这个版本，它有一个 bug 并不能编译：

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),
        }
    }
```

我们没有处理 `None` 的情况，所以这些代码会造成一个 bug。幸运的是，这是一个 Rust 知道如何处理的 bug。如果尝试编译这段代码，会得到这个错误：

```console
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0004]: non-exhaustive patterns: `None` not covered
 --> src/main.rs:3:15
  |
3 |         match x {
  |               ^ pattern `None` not covered
  |
note: `Option<i32>` defined here
 --> /rustc/4eb161250e340c8f48f66e2b929ef4a5bed7c181/library/core/src/option.rs:572:1
 ::: /rustc/4eb161250e340c8f48f66e2b929ef4a5bed7c181/library/core/src/option.rs:576:5
  |
  = note: not covered
  = note: the matched value is of type `Option<i32>`
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
  |
4 ~             Some(i) => Some(i + 1),
5 ~             None => todo!(),
  |

For more information about this error, try `rustc --explain E0004`.
error: could not compile `enums` (bin "enums") due to 1 previous error
```

Rust 知道我们没有覆盖所有可能的情况甚至知道哪些模式被忘记了！Rust 中的匹配是 **穷尽的**（*exhaustive*）：必须穷举到最后的可能性来使代码有效。特别的在这个 `Option<T>` 的例子中，Rust 防止我们忘记明确的处理 `None` 的情况，这让我们免于假设拥有一个实际上为空的值，从而使之前提到的价值亿万的错误不可能发生。

### [通配模式和 `_` 占位符](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#通配模式和-_-占位符)

使用枚举，我们也可以针对少数几个特定值执行特殊操作，而对其他所有值采取默认操作。想象我们正在玩一个游戏，如果你掷出骰子的值为 3，角色不会移动，而是会得到一顶新奇的帽子。如果你掷出了 7，你的角色将失去一顶新奇的帽子。对于其他的数值，你的角色会在棋盘上移动相应的格子。这是一个实现了上述逻辑的 `match`，骰子的结果是硬编码而不是一个随机值，其他的逻辑部分使用了没有函数体的函数来表示，实现它们超出了本例的范围：

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

对于前两个分支，匹配模式是字面值 `3` 和 `7`，最后一个分支则涵盖了所有其他可能的值，模式是我们命名为 `other` 的一个变量。`other` 分支的代码通过将其传递给 `move_player` 函数来使用这个变量。

即使我们没有列出 `u8` 所有可能的值，这段代码依然能够编译，因为最后一个模式将匹配所有未被特殊列出的值。这种通配模式满足了 `match` 必须被穷尽的要求。请注意，我们必须将通配分支放在最后，因为模式是按顺序匹配的。如果我们在通配分支后添加其他分支，Rust 将会警告我们，因为此后的分支永远不会被匹配到。

Rust 还提供了一个模式，当我们不想使用通配模式获取的值时，请使用 `_` ，这是一个特殊的模式，可以匹配任意值而不绑定到该值。这告诉 Rust 我们不会使用这个值，所以 Rust 也不会警告我们存在未使用的变量。

让我们改变游戏规则：现在，当你掷出的值不是 3 或 7 的时候，你必须再次掷出。这种情况下我们不需要使用这个值，所以我们改动代码使用 `_` 来替代变量 `other` ：

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

这个例子也满足穷尽性要求，因为我们在最后一个分支中显式地忽略了其它值。我们没有忘记处理任何东西。

最后，让我们再次改变游戏规则，如果你掷出 3 或 7 以外的值，你的回合将无事发生。我们可以使用单元值（在[“元组类型”](https://kaisery.github.io/trpl-zh-cn/ch03-02-data-types.html#元组类型)一节中提到的空元组）作为 `_` 分支的代码：

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
```

在这里，我们明确告诉 Rust 我们不会使用与前面模式不匹配的值，并且这种情况下我们不想运行任何代码。



### `_`（下划线）



`_` 在 `match` 语句中是一个**通配符模式**。它的作用是：

- **匹配任何值，但不会将该值绑定到任何变量**。这意味着你不在乎具体匹配到的值是什么，只要模式匹配成功就执行对应的代码块。
- **表示剩余情况的穷尽匹配**。它通常作为 `match` 表达式的最后一个分支，用来捕获所有之前没有明确处理的模式。

**`_` 的主要特点：**

- **不绑定**：匹配到的值不会绑定到任何变量，因此你不能在该分支的代码块中使用它。
- **不可驳斥**：它总是会匹配成功。
- **常用于默认或“包罗万象”的情况**。

**示例：**

Rust

```
fn process_number(x: i32) {
    match x {
        1 => println!("数字是1！"),
        2 => println!("数字是2！"),
        _ => println!("其他数字！"), // 匹配任何其他 i32 值
    }
}

fn main() {
    process_number(1);
    process_number(5);
}
```

在这个例子中，`_` 处理了所有不是 `1` 或 `2` 的 `i32` 值。



### `other`（或任何其他标识符）

当你使用 `other`（或 `x`、`value`、`remainder` 等）这样的标识符在 `match` 语句中时，它充当一个**变量绑定模式**。它的作用是：

- **匹配任何值并将其绑定到一个新的变量**，变量名就是你指定的标识符。这允许你在该分支的代码块中使用匹配到的值。
- **捕获值**以便进一步处理。

**标识符（如 `other`）的主要特点：**

- **绑定**：匹配到的值会绑定到指定的变量，然后可以在该分支的代码中使用。
- **可驳斥（但可以作为包罗万象的情况）**：虽然如果放在最后它也可以作为包罗万象的分支，但它的主要目的是绑定值。
- **当需要处理匹配到的值时非常有用**。

```rust
enum Result {
    Success(String),
    Error(u32),
    Loading,
}

fn handle_result(res: Result) {
    match res {
        Result::Success(message) => println!("成功：{}", message),
        Result::Error(code) => println!("错误码：{}", code),
        other => println!("接收到其他状态：{:?}", other), // 将剩余的 Result 值绑定到 'other'
    }
}

fn main() {
    handle_result(Result::Success("操作完成".to_string()));
    handle_result(Result::Error(404));
    handle_result(Result::Loading); // 'Loading' 将被绑定到 'other'
}
```

在这个例子中，当匹配到 `Result::Loading` 时，`Loading` 变体本身被绑定到 `other` 变量，然后你可以在代码中打印或使用 `other`。

------



### 区别总结 📊



| 特征         | `_`（通配符）                | `other`（变量绑定）                      |
| ------------ | ---------------------------- | ---------------------------------------- |
| **目的**     | 忽略值；作为“包罗万象”的模式 | 将值绑定到变量以供使用                   |
| **值的使用** | 不能在该分支中使用匹配到的值 | 可以使用匹配到的值（通过变量）在该分支中 |
| **绑定**     | 不发生绑定                   | 将匹配到的值绑定到标识符                 |
| **常见用例** | 默认情况，忽略模式的特定部分 | 捕获和处理匹配到的值                     |



## 什么是“愉快路径”（Happy Path）？



首先，理解“愉快路径”的概念很重要。在编程中，“愉快路径”指的是程序在**没有遇到错误、异常或意外情况**时，按照预期顺利执行的流程。就像你在一条平坦的路上开车，没有堵车，没有故障，一路畅通。

相反，如果出现错误或不匹配的情况，我们就需要处理**“不愉快路径”**，比如报错、返回空值、退出程序等。

------



## `let...else` 的核心思想 



`let...else` 的核心目的是为了让你的代码在处理可能失败的操作时，能够**清晰地把“愉快路径”的代码放在主线上，而把“不愉快路径”的退出逻辑快速处理掉**。它就像一个“快速出口”，当条件不满足时，直接从函数中跳出，避免让主逻辑变得复杂。



### 为什么需要 `let...else`？（C 语言类比：繁琐的错误检查）



在 C 语言中，当你调用一个可能失败的函数，或者需要检查一个指针是否为 `NULL` 时，你通常会看到这样的模式：

```c
// C 语言示例：模拟一个可能失败的函数调用
int* get_data() {
    // 假设这个函数可能返回 NULL 表示失败
    if (rand() % 2 == 0) { // 模拟随机失败
        return NULL;
    }
    int* data = (int*)malloc(sizeof(int));
    *data = 100;
    return data;
}

void process_data() {
    int* data = get_data(); // 调用可能失败的函数

    // 经典 C 语言的错误检查模式：if (data == NULL) { return; }
    if (data == NULL) { // 检查“不愉快路径”
        printf("获取数据失败，提前返回。\n");
        return; // 从函数中提前返回
    }

    // 走到这里，data 肯定不是 NULL，这是“愉快路径”
    printf("成功获取数据: %d\n", *data);
    free(data); // 释放资源
}

int main() {
    srand(time(NULL)); // 初始化随机数种子
    process_data();
    process_data();
    return 0;
}
```

在上面的 C 语言代码中，`if (data == NULL)` 就是处理“不愉快路径”的代码。它打断了主逻辑（处理数据）的流畅性，因为你必须先进行检查，如果失败就 `return`。

如果有很多这样的检查，或者你需要从多个函数中获取数据并检查，你的代码可能会变成这样：

```c
// C 语言中嵌套的错误检查可能很丑陋
TypeA* objA = get_obj_a();
if (objA == NULL) {
    return ERROR_A;
}

TypeB* objB = get_obj_b(objA);
if (objB == NULL) {
    return ERROR_B;
}

TypeC* objC = get_obj_c(objB);
if (objC == NULL) {
    return ERROR_C;
}

// 只有当所有都成功时，才能执行核心逻辑
process_final_data(objA, objB, objC);
```

这就是 `if let` 有时显得“繁琐”或“不对称”的原因。它虽然能绑定值，但在处理不匹配时，如果你想提前返回，就需要在 `else` 块中明确写 `return`，让控制流看起来有点跳跃：

```rust
// 对应上面 C 语言的 if (data == NULL) { return; }
// Rust 的 if let 模拟：
fn describe_state_quarter_if_let(coin: Coin) -> Option<String> {
    let state = if let Coin::Quarter(s) = coin {
        s // 匹配成功，绑定 state
    } else {
        return None; // 匹配失败，直接返回 None
    };
    // 只有匹配成功，才会执行到这里
    // 接下来是处理 state 的“愉快路径”代码
    // ...
    Some(format!("{state:?}"))
}
```

这段 `if let` 的代码虽然可以实现功能，但你会感觉 `let state = if let ... else { return None; };` 这一行有点别扭。成功的逻辑是赋值，失败的逻辑是返回，两种控制流类型不一样。



### `let...else` 如何简化？（C 语言类比：更直接的错误处理）



`let...else` 的出现就是为了让这种“如果模式匹配就绑定值，否则直接退出”的场景变得更简洁、更符合“愉快路径”的直觉。它强制 `else` 块必须包含一个**非局部退出**（Non-local Return），也就是跳出当前函数、循环或者直接使程序中断。

我们可以将 `let...else` 类比为 C 语言中结合了**宏**或**特定的错误处理约定**来简化这种“检查-退出”模式：

想象一下，在 C 语言中，你可能会定义一个宏来做这样的事情（虽然不完全一样，但思想相似）：

```c
// 伪 C 语言宏类比 let...else
#define CHECK_AND_GET_DATA(ptr_var, func_call) \
    ptr_var = func_call;                     \
    if (ptr_var == NULL) {                   \
        printf("错误，提前退出！\n");         \
        return;                              \
    }

void process_data_simplified() {
    int* data;
    CHECK_AND_GET_DATA(data, get_data()); // 使用宏，如果 get_data 失败就直接返回

    // 走到这里，data 肯定有效，直接处理“愉快路径”
    printf("成功获取数据（简化版）: %d\n", *data);
    free(data);
}
```

Rust 的 `let...else` 就是把这种模式内置到了语言层面，让它安全且优雅。

```rust
fn describe_state_quarter_let_else(coin: Coin) -> Option<String> {
    // let Coin::Quarter(state) = coin else { ... };
    // 尝试将 coin 匹配为 Coin::Quarter。
    // 如果匹配成功，那么 state 变量就会被绑定，程序会继续往下执行（“愉快路径”）。
    let Coin::Quarter(state) = coin else {
        return None; // 如果不匹配（比如 coin 是 Coin::Dime 或 Coin::Nickel），直接从函数返回 None。
    };

    // 只有当 coin 确实是 Coin::Quarter 时，代码才会执行到这里。
    // 此时 state 变量已经包含了 Quarter 中的 UsState 值。
    // 这就是我们的“愉快路径”：直接使用 state 进行后续操作。
    if state.existed_in(1900) {
        Some(format!("{state:?} is pretty old, for America!"))
    } else {
        Some(format!("{state:?} is relatively new."))
    }
}
```

**`let...else` 的优点：**

1. **保持“愉快路径”的简洁性**：它允许你的主要逻辑（即成功时执行的代码）保持在左对齐的、不被中断的块中。那些会导致函数退出的“不愉快路径”逻辑被清晰地隔离在 `else` 块里，而且这个块必须执行一个非局部返回，避免了遗漏。
2. **更清晰的控制流**：一眼就能看出如果模式不匹配，函数会立即退出，避免了 `if let` 某些情况下可能出现的控制流跳跃或分支逻辑不一致的问题。这使得代码更易读、易懂。

简而言之，`let...else` 是 Rust 语言为处理“如果能成功解构就继续，否则立即退出”这种常见模式提供的一个语法糖，让代码在面对潜在失败时，依然能保持“愉快路径”的简洁和直观。
