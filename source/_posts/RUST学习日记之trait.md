---
title: RUST学习日记之trait
date: 2025-07-28 16:46:13
tags: Rust 
categories: Rust
---

Trait类似于java语言中的接口

### 定义 trait

一个类型的行为由其可供调用的方法构成。如果可以对不同类型调用相同的方法的话，这些类型就可以共享相同的行为了。trait 定义是一种将方法签名组合起来的方法，目的是定义一个实现某些目的所必需的行为的集合。

例如，这里有多个存放了不同类型和属性文本的结构体：结构体 `NewsArticle` 用于存放发生于世界各地的新闻故事，而结构体 `Tweet` 最多只能存放 280 个字符的内容，以及像是否转推或是否是对推友的回复这样的元数据。

我们想要创建一个多媒体聚合库用来显示可能储存在 `NewsArticle` 或 `Tweet` 实例中的数据的总结。每一个结构体都需要的行为是他们是能够被总结的，这样的话就可以调用实例的 `summarize` 方法来请求总结。示例 10-12 中展示了一个表现这个概念的 `Summary` trait 的定义：

文件名: src/lib.rs

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

示例 10-12：`Summary` trait 定义，它包含由 `summarize` 方法提供的行为

这里使用 `trait` 关键字来声明一个 trait，后面是 trait 的名字，在这个例子中是 `Summary`。在大括号中声明描述实现这个 trait 的类型所需要的行为的方法签名，在这个例子中是 `fn summarize(&self) -> String`。

在方法签名后跟分号，而不是在大括号中提供其实现。接着每一个实现这个 trait 的类型都需要提供其自定义行为的方法体，编译器也会确保任何实现 `Summary` trait 的类型都拥有与这个签名的定义完全一致的 `summarize` 方法。

trait 体中可以有多个方法：一行一个方法签名且都以分号结尾。

### 为类型实现 trait

现在我们定义了 `Summary` trait，接着就可以在多媒体聚合库中需要拥有这个行为的类型上实现它了。示例 10-13 中展示了 `NewsArticle` 结构体上 `Summary` trait 的一个实现，它使用标题、作者和创建的位置作为 `summarize` 的返回值。对于 `Tweet` 结构体，我们选择将 `summarize` 定义为用户名后跟推文的全部文本作为返回值，并假设推文内容已经被限制为 280 字符以内。

文件名: src/lib.rs

```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

示例 10-13：在 `NewsArticle` 和 `Tweet` 类型上实现 `Summary` trait

在类型上实现 trait 类似于实现与 trait 无关的方法。区别在于 `impl` 关键字之后，我们提供需要实现 trait 的名称，接着是 `for` 和需要实现 trait 的类型的名称。在 `impl` 块中，使用 trait 定义中的方法签名，不过不再后跟分号，而是需要在大括号中编写函数体来为特定类型实现 trait 方法所拥有的行为。

一旦实现了 trait，我们就可以用与 `NewsArticle` 和 `Tweet` 实例的非 trait 方法一样的方式调用 trait 方法了：

```rust
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

这会打印出 `1 new tweet: horse_ebooks: of course, as you probably already know, people`。

注意因为示例 10-13 中我们在相同的 *lib.rs* 里定义了 `Summary` trait 和 `NewsArticle` 与 `Tweet` 类型，所以他们是位于同一作用域的。如果这个 *lib.rs* 是对应 `aggregator` crate 的，而别人想要利用我们 crate 的功能为其自己的库作用域中的结构体实现 `Summary` trait。首先他们需要将 trait 引入作用域。这可以通过指定 `use aggregator::Summary;` 实现，这样就可以为其类型实现 `Summary` trait 了。`Summary` 还必须是公有 trait 使得其他 crate 可以实现它，这也是为什么示例 10-12 中将 `pub` 置于 `trait` 之前。

### Trait 的可见性

当你定义一个 `trait` 时，你需要考虑它是否应该对其他 `crate` 可用。

- 如果一个 `trait` 只是在你自己的 `crate` 内部使用，那么它不需要是公共的。
- 但如果你希望其他 `crate` 能够为它们自己的类型实现你定义的 `trait`，那么这个 `trait` 必须是 **`pub`（公共的）**。这就是为什么在示例 10-12 中，`Summary` trait 前面有 `pub` 关键字，表示它是公共的，其他 `crate` 才能看到并实现它。

### 实现 Trait 的作用域规则

当你想要为一个类型实现某个 `trait` 时，有一个重要的限制：**你只能为你自己的 `crate` 本地定义的类型实现 `trait`，或者为你自己的 `crate` 本地定义的 `trait` 实现类型。**

让我们用例子来说明：

- **可以实现的情况：**
  - **为你自己的类型实现标准库中的 `trait`：** 比如，你在 `aggregator` 这个 `crate` 中定义了一个自定义类型 `Tweet`。你可以为这个 `Tweet` 类型实现标准库中的 `Display` trait（这个 `trait` 用于控制类型如何打印输出）。这是因为 `Tweet` 是你 `aggregator` `crate` 本地定义的类型。
  - **为标准库类型实现你自己的 `trait`：** 同样地，你可以在 `aggregator` `crate` 中为标准库类型 `Vec<T>`（一个向量类型）实现你自定义的 `Summary` trait。这是因为 `Summary` `trait` 是你 `aggregator` `crate` 本地定义的 `trait`。
- **不能实现的情况（外部类型实现外部 trait）：**
  - 你**不能**在 `aggregator` `crate` 中为标准库类型 `Vec<T>` 实现标准库中的 `Display` trait。为什么呢？因为 `Display` 和 `Vec<T>` 这两个东西都**不是**你 `aggregator` `crate` 本地定义的。它们都来自标准库，对于你的 `aggregator` `crate` 来说，它们都是“外部”的。

### 孤儿规则（Orphan Rule）的意义

这个限制被称为**相干性（coherence）**，更具体地说是**孤儿规则（orphan rule）**。这条规则是为了避免潜在的冲突和混乱：

- **避免冲突：** 如果没有这条规则，想象一下：`crate A` 为 `Vec<T>` 实现了 `Display`，而 `crate B` 也为 `Vec<T>` 实现了 `Display`。当你的代码同时依赖 `crate A` 和 `crate B` 时，Rust 就会感到困惑，不知道当你想打印 `Vec<T>` 时，到底应该使用 `crate A` 的 `Display` 实现还是 `crate B` 的 `Display` 实现。
- **保证代码稳定性：** 孤儿规则确保了“别人编写的代码不会破坏你的代码，反之亦然”。它防止了不同 `crate` 对同一外部类型实现相同外部 `trait` 导致的行为不确定性。

简而言之，孤儿规则保证了每个 `trait` 实现都必须至少有一个“亲属”在当前 `crate` 中，要么是你实现了这个 `trait` 的类型是你自己定义的，要么是你实现的这个 `trait` 是你自己定义的。

### 默认实现

有时为 trait 中的某些或全部方法提供默认的行为，而不是在每个类型的每个实现中都定义自己的行为是很有用的。这样当为某个特定类型实现 trait 时，可以选择保留或重载每个方法的默认行为。

示例 10-14 中展示了如何为 `Summary` trait 的 `summarize` 方法指定一个默认的字符串值，而不是像示例 10-12 中那样只是定义方法签名：

文件名: src/lib.rs

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

示例 10-14：`Summary` trait 的定义，带有一个 `summarize` 方法的默认实现

如果想要对 `NewsArticle` 实例使用这个默认实现，而不是定义一个自己的实现，则可以通过 `impl Summary for NewsArticle {}` 指定一个空的 `impl` 块。

虽然我们不再直接为 `NewsArticle` 定义 `summarize` 方法了，但是我们提供了一个默认实现并且指定 `NewsArticle` 实现 `Summary` trait。因此，我们仍然可以对 `NewsArticle` 实例调用 `summarize` 方法，如下所示：

```rust
let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
    author: String::from("Iceburgh"),
    content: String::from("The Pittsburgh Penguins once again are the best
    hockey team in the NHL."),
};

println!("New article available! {}", article.summarize());
```

这段代码会打印 `New article available! (Read more...)`。

为 `summarize` 创建默认实现并不要求对示例 10-13 中 `Tweet` 上的 `Summary` 实现做任何改变。其原因是重载一个默认实现的语法与实现没有默认实现的 trait 方法的语法一样。

默认实现允许调用相同 trait 中的其他方法，哪怕这些方法没有默认实现。如此，trait 可以提供很多有用的功能而只需要实现指定一小部分内容。例如，我们可以定义 `Summary` trait，使其具有一个需要实现的 `summarize_author` 方法，然后定义一个 `summarize` 方法，此方法的默认实现调用 `summarize_author` 方法：

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

为了使用这个版本的 `Summary`，只需在实现 trait 时定义 `summarize_author` 即可：

```rust
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

一旦定义了 `summarize_author`，我们就可以对 `Tweet` 结构体的实例调用 `summarize` 了，而 `summarize` 的默认实现会调用我们提供的 `summarize_author` 定义。因为实现了 `summarize_author`，`Summary` trait 就提供了 `summarize` 方法的功能，且无需编写更多的代码。

```rust
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

这会打印出 `1 new tweet: (Read more from @horse_ebooks...)`。

请注意，无法从相同方法的重载实现中调用默认方法。

核心思想是：**`trait` 定义了行为，`trait bound` 则限制了泛型类型必须拥有这些行为。**

### 1. 将 Trait 作为函数参数

想象一下，你有一个 `Summary` trait，它规定了任何实现它的类型都应该有一个 `summarize` 方法。

- **`impl Trait` 语法（简单写法）：**

  ```Rust
  pub fn notify(item: impl Summary) {
      println!("Breaking news! {}", item.summarize());
  }
  ```

  这就像在说：“嘿，`notify` 函数，我不在乎你收到的是 `NewsArticle` 还是 `Tweet`，**只要**它能 `summarize` 就行！” 这种写法简洁明了，编译器会确保你传递进来的类型确实实现了 `Summary`。如果传了 `String` 或 `i32`，编译就会失败，因为它们没有 `summarize` 方法。

### 2. Trait Bound 语法（更详细的写法）

`impl Trait` 只是一个语法糖，它的背后是更正式的 **Trait Bound** 语法。

- **基本等价写法：**

  ```Rust
  pub fn notify<T: Summary>(item: T) {
      println!("Breaking news! {}", item.summarize());
  }
  ```

  这和上面的 `impl Trait` 效果一样，但是明确引入了一个**泛型类型 `T`**，并用 `<T: Summary>` 表示 `T` 必须实现 `Summary`。

- **强制多个参数类型一致：** `Trait Bound` 的强大之处在于它可以让你强制多个泛型参数是**同一个具体类型**。

  - 如果你用 `impl Trait`：

    ```Rust
    pub fn notify(item1: impl Summary, item2: impl Summary) { /* ... */ }
    ```

    `item1` 可以是 `NewsArticle`，`item2` 可以是 `Tweet`，只要它们都能 `summarize` 就行。它们**可以是不同类型**。

  - 如果你用 `Trait Bound`：

    ```Rust
    pub fn notify<T: Summary>(item1: T, item2: T) { /* ... */ }
    ```

    这里 `item1` 和 `item2` 都被指定为泛型类型 `T`。这意味着，如果你给 `item1` 传了一个 `NewsArticle`，那么 `item2` **也必须**是 `NewsArticle`。它们**必须是相同类型**。

### 3. `+` 指定多个 Trait Bound

如果一个类型需要同时实现多个 `trait` 呢？用 `+` 符号连接它们！

- **使用 `impl Trait`：**

  ```Rust
  pub fn notify(item: impl Summary + Display) { /* ... */ }
  ```

  这表示 `item` 既要能 `summarize`，也要能被 `Display`（也就是可以被格式化打印出来）。

- **使用 `Trait Bound`：**

  ```Rust
  pub fn notify<T: Summary + Display>(item: T) { /* ... */ }
  ```

  效果一样，只是写法不同。

### 4. `where` 从句简化 Trait Bound

当你的函数有很多泛型参数，并且每个参数都有多个 `trait bound` 时，函数签名会变得非常长，难以阅读。`where` 从句就是为了解决这个问题。

- **没有 `where` 从句：**

  ```Rust
  fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 { /* ... */ }
  ```

  你看，尖括号里一堆东西，函数名和参数列表离得好远。

- **使用 `where` 从句：**

  ```Rust
  fn some_function<T, U>(t: T, u: U) -> i32
      where T: Display + Clone,
            U: Clone + Debug
  { /* ... */ }
  ```

  这样就清晰多了！`trait bound` 被移到了单独的 `where` 从句中，函数签名本身就简洁了。

### 5. 返回实现了 Trait 的类型

你也可以让函数返回一个**实现了特定 `trait` 的类型**，而不需要暴露具体的类型是什么。

- **语法：**

  ```Rust
  fn returns_summarizable() -> impl Summary {
      Tweet { /* ... */ }
  }
  ```

  这个函数承诺它会返回一个可以被“总结”的东西（`impl Summary`），但具体是 `Tweet` 还是 `NewsArticle`，函数调用方并不需要知道。这对于**闭包**和**迭代器**特别有用，因为它们的实际类型可能非常复杂，用 `impl Trait` 就能大大简化代码。

- **重要限制：** `returns_summarizable()` 这种写法**只能返回单一的具体类型**。

  ```Rust
  // 错误示例！不能编译！
  fn returns_summarizable(switch: bool) -> impl Summary {
      if switch {
          NewsArticle { /* ... */ }
      } else {
          Tweet { /* ... */ } // 这里返回了不同的类型
      }
  }
  ```

  **你不能根据条件返回不同的实现了 `Summary` 的类型（比如有时返回 `NewsArticle`，有时返回 `Tweet`）。如果需要这样做，你得使用 **`trait object`（特性对象）**，这是第 17 章会讲到的更高级概念。 **

### 6. 修复 `largest` 函数

回到最开始 `largest` 函数的错误：

- **问题一：不能比较 `T` 类型（`>` 运算符）。**

  - 因为 `>` 运算符是 `std::cmp::PartialOrd` 这个 `trait` 提供的。

  - **解决方案：** 给 `T` 加上 `PartialOrd` 的 **trait bound**：

    ```Rust
    fn largest<T: PartialOrd>(list: &[T]) -> T { /* ... */ }
    ```

- **问题二：不能移动非 `Copy` 类型的值（`list[0]` 和 `for &item`）。**

  - `list[0]` 会把第一个元素“移动”出来，`for &item` 也尝试“解引用并移动”。但如果 `T` 没有实现 `Copy` trait，这种移动是不被允许的（因为移动后原位置就“空”了，而切片 `&[T]` 不允许这种操作）。像 `i32` 和 `char` 这种栈上数据默认是 `Copy` 的，但其他复杂类型可能不是。

  - **解决方案一（简单粗暴）：** 给 `T` 再加上 `Copy` 的 **trait bound**。这样就限制了 `largest` 只能用于那些可以简单复制的类型。

    ```Rust
    fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
        let mut largest = list[0]; // 现在 T 保证是 Copy 的
        for &item in list.iter() { // 现在 &item 可以拷贝一份
            if item > largest {
                largest = item;
            }
        }
        largest
    }
    ```

  - **解决方案二（更通用但可能慢）：** 如果不想限制 `Copy`，可以要求 `T` 实现 `Clone`，然后在需要的时候**显式地克隆**数据。但这可能会涉及堆内存分配，效率较低。

  - **解决方案三（最佳实践）：** 返回一个**引用** `&T`！这样就避免了移动或拷贝数据，直接操作原始数据的引用。你会被鼓励尝试自己实现这个版本。

### 7. 有条件地实现方法和 Blanket Implementations

- **有条件实现方法：**

  - 你可以给一个泛型结构体（比如 `Pair<T>`）的 `impl` 块加上 **`trait bound`**。

  - 这意味着：`Pair<T>` 总是有一个 `new` 方法，但它只有在 `T` 类型同时实现了 `Display` 和 `PartialOrd` 时，才会有 `cmp_display` 方法。

    ```Rust
      impl<T: Display + PartialOrd> Pair<T> {
          fn cmp_display(&self) { /* ... */ } // 只有当 T 满足条件时才有
      }
    ```

- **Blanket Implementations（毯子实现/覆盖实现）：**

  - 这是一个非常强大的功能！它允许你**为所有满足特定 `trait bound` 的类型实现另一个 `trait`**。

  - 标准库中就有很多例子，比如：

    ```Rust
    impl<T: Display> ToString for T { /* ... */ }
    ```

    这句话的意思是：“**任何**实现了 `Display` trait 的类型，都自动实现 `ToString` trait。”

  - 这解释了为什么你可以直接对一个整数（比如 `3`）调用 `.to_string()`：`3` 是 `i32` 类型，`i32` 实现了 `Display`，所以根据这个“毯子实现”，`i32` 也就自动实现了 `ToString`。

### 总结一下：

`trait` 和 `trait bound` 是 Rust 泛型系统的基石。它们让你能够编写出：

- **灵活的代码：** 可以处理多种不同类型。
- **安全的代码：** 编译器在编译时就检查类型是否满足所需行为，而不是等到运行时才报错。
- **高性能的代码：** 因为编译时已经确定了类型和行为，运行时不需要额外的检查开销。

这就像是给你的泛型函数或类型打上了“能力标签”，只有拥有这些标签的类型才能被使用，确保了代码的正确性。

