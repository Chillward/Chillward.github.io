---
title: RUST学习日记之模块01
date: 2025-07-23 21:25:01
tags: RUST
categories: Rust
---

 在C语言编程时，我们会包含多个头文件，然后就可以使用头文件中定义的函数和数据类型。但是C语言这一套容易出现一些问题:

1. 隐式依赖
2. 全局命名空间污染
3. 重复包含问题

RUST的模块系统很好的避免了这些问题

> 依旧是熟悉的复制粘贴

  到目前为止，我们编写的程序都在一个文件的一个模块中。伴随着项目的增长，你应该通过将代码分解为多个模块和多个文件来组织代码。一个包（package）可以包含多个二进制 crate 项和一个可选的库 crate。伴随着包的增长，你可以将包中的部分代码提取出来，做成独立的 crate，这些 crate 则作为外部依赖项。本章将会涵盖所有这些概念。对于一个由一系列相互关联的包组成的超大型项目，Cargo 提供了**工作空间**（*workspaces*）这一功能，我们将在第十四章的 [“Cargo Workspaces”](https://kaisery.github.io/trpl-zh-cn/ch14-03-cargo-workspaces.html) 对此进行讲解。

  我们也会讨论封装来实现细节，这可以让你在更高层面重用代码：你实现了一个操作后，其他的代码可以通过该代码的公共接口来进行调用，而不需要知道它是如何实现的。你在编写代码时可以定义哪些部分是其他代码可以使用的公共部分，以及哪些部分是你有权更改实现细节的私有部分。这是另一种减少你在脑海中记住项目内容数量的方法。

  这里有一个需要说明的概念 “作用域（scope）”：代码所在的嵌套上下文有一组定义为 “in scope” 的名称。当阅读、编写和编译代码时，程序员和编译器需要知道特定位置的特定名称是否引用了变量、函数、结构体、枚举、模块、常量或者其他有意义的项。你可以创建作用域，以及改变哪些名称在作用域内还是作用域外。同一个作用域内不能拥有两个相同名称的项；可以使用一些工具来解决名称冲突。

  Rust 有许多功能可以让你管理代码的组织，包括哪些细节可以被公开，哪些细节作为私有部分，以及程序中各个作用域中有哪些名称。这些特性，有时被统称为 “模块系统（the module system）”，包括：

- **包**（*Packages*）：Cargo 的一个功能，它允许你构建、测试和分享 crate。
- **Crates** ：一个模块的树形结构，它形成了库或可执行文件项目。
- **模块**（*Modules*）和 **use**：允许你控制作用域和路径的私有性。
- **路径**（*path*）：一个为例如结构体、函数或模块等项命名的方式。

## 包和Create

crate 是 Rust 在编译时最小的代码单位。即使你用 `rustc` 而不是 `cargo` 来编译一个单独的源代码文件（正如我们在第 1 章“编写并运行 Rust 程序”中所做的那样），编译器还是会将那个文件视为一个 crate。crate 可以包含模块，模块可以定义在其他文件，然后和 crate 一起编译，我们会在接下来的章节中遇到。

crate 有两种形式：二进制 crate 和库 crate。**二进制 crate**（*Binary crates*）可以被编译为可执行程序，比如命令行程序或者服务端。它们必须有一个名为 `main` 函数来定义当程序被执行的时候所需要做的事情。目前我们所创建的 crate 都是二进制 crate。

**库 crate**（*Library crates*）并没有 `main` 函数，它们也不会编译为可执行程序。相反它们定义了可供多个项目复用的功能模块。比如 [第二章](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html#生成一个随机数) 的 `rand` crate 就提供了生成随机数的功能。大多数时间 `Rustaceans` 说的 “crate” 指的都是库 crate，这与其他编程语言中 “library” 概念一致。

###### *crate root* 是一个源文件，Rust 编译器以它为起始点，并构成你的 crate 的根模块。



**包（package）是提供一系列功能的一个或者多个 crate 的捆绑。**一个包会包含一个 *Cargo.toml* 文件，阐述如何去构建这些 crate。Cargo 实际上就是一个包，它包含了用于构建你代码的命令行工具的二进制 crate。其他项目也依赖 Cargo 库来实现与 Cargo 命令行程序一样的逻辑。

包中可以包含至多一个库 crate(library crate)。包中可以包含任意多个二进制 crate(binary crate)，但是必须至少包含一个 crate（无论是库的还是二进制的）。

让我们来看看创建包的时候会发生什么。首先，我们输入命令 `cargo new my-project`：

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

运行了这条命令后，我们先用 `ls` （译者注：此命令为 Linux 平台的指令，Windows 下可用 dir）来看看 Cargo 给我们创建了什么，Cargo 会给我们的包创建一个 *Cargo.toml* 文件。查看 *Cargo.toml* 的内容，会发现并没有提到 *src/main.rs*，**因为 Cargo 遵循的一个约定：*src/main.rs* 就是一个与包同名的二进制 crate 的 crate 根。**同样的，Cargo 知道如果包目录中包含 *src/lib.rs*，则包带有与其同名的库 crate，且 *src/lib.rs* 是 crate 根。crate 根文件将由 Cargo 传递给 `rustc` 来实际构建库或者二进制项目。

~~~
[package]
name = "Functions"
version = "0.1.0"
edition = "2024"

[dependencies]
//假如我们要添加main跟lib之外的其他依赖
[[bin]]
name = "another-app" # 这是你希望这个额外二进制 crate 的名称
path = "src/bin/another_app.rs" # 这是它的源文件路径
~~~

在此，我们有了一个只包含 *src/main.rs* 的包，意味着它只含有一个名为 `my-project` 的二进制 crate。如果一个包同时含有 *src/main.rs* 和 *src/lib.rs*，则它有两个 crate：一个二进制的和一个库的，且名字都与包相同。通过将文件放在 *src/bin* 目录下，一个包可以拥有多个二进制 crate：每个 *src/bin* 下的文件都会被编译成一个独立的二进制 crate。

> 来点人话

### 单一二进制 Crate

最简单的 Rust 包结构是一个只包含 `src/main.rs` 文件的包。在这种情况下，你的包将编译成一个名为 `my-project` 的**二进制 crate**（其中 `my-project` 是你的包名）。所有代码都将包含在 `main.rs` 文件中。

```
my-project/
├── src/
│   └── main.rs
└── Cargo.toml
```

### 二进制 Crate 和库 Crate 共存

如果一个包同时包含 `src/main.rs` 和 `src/lib.rs`，那么它将拥有两个 crate：一个**二进制 crate** 和一个**库 crate**。这两个 crate 的名字都默认与包名相同。

- `src/main.rs` 中的代码会编译成可执行程序。
- `src/lib.rs` 中的代码会编译成库，可以被其他项目（包括当前包的二进制 crate）引用。

```
my-project/
├── src/
│   ├── main.rs
│   └── lib.rs
└── Cargo.toml
```

在这种结构中，`main.rs` 通常会使用 `use my_project;` 或 `use crate;`（在 Rust 2018 版及更高版本中）来引用 `lib.rs` 中定义的模块和函数。

### 多个二进制 Crate

Rust 包可以通过将文件放置在 `src/bin` 目录下拥有多个**二进制 crate**。`src/bin` 目录下的每个文件都会被编译成一个独立的二进制 crate。每个文件的名称（不包括 `.rs` 后缀）将成为对应二进制 crate 的名称。

```
my-project/
├── src/
│   ├── main.rs         // 默认的二进制 crate
│   ├── lib.rs          // 库 crate
│   └── bin/
│       ├── my_tool.rs  // 额外的二进制 crate: `my_tool`
│       └── another_app.rs // 额外的二进制 crate: `another_app`
└── Cargo.toml
```

在这个例子中：

- `cargo run` 默认会编译并运行 `src/main.rs` 对应的二进制 crate。

- 你可以使用 `cargo run --bin my_tool` 来运行 `src/bin/my_tool.rs` 对应的二进制 crate。

- 同样，`cargo run --bin another_app` 会运行 `src/bin/another_app.rs`。

  

  当 Rust 包包含多个二进制 crate 时，**`src/main.rs` 中的内容仍然是默认运行的二进制 crate**。

  即使你在 `src/bin` 目录下添加了其他的 `.rs` 文件来创建额外的二进制 crate，`cargo run` 命令在没有额外指定的情况下，依然会查找并运行 `src/main.rs` 编译而成的可执行文件。

  如果你想运行 `src/bin` 目录下特定的二进制 crate，你需要使用 `cargo run --bin <crate-name>` 命令，其中 `<crate-name>` 是你想要运行的二进制 crate 的文件名（不包括 `.rs` 后缀）。

  **举个例子：**

  假设你有这样的项目结构：

  ```
  my-project/
  ├── src/
  │   ├── main.rs         // 默认的二进制 crate
  │   └── bin/
  │       └── my_tool.rs  // 另一个二进制 crate: `my_tool`
  └── Cargo.toml
  ```

  - 运行 `cargo run` 会编译并执行 `src/main.rs` 中的代码。
  - 运行 `cargo run --bin my_tool` 则会编译并执行 `src/bin/my_tool.rs` 中的代码。

  所以，`src/main.rs` 就像是你的项目主要的或默认的入口点，而 `src/bin` 中的文件则提供了额外的、可独立运行的工具或应用程序。

  > 暂时不知道有什么用

### 模块的组织

在上述基本结构的基础上，你可以在 `src` 目录下创建子目录来进一步组织你的模块。每个 `mod.rs` 文件或与目录同名的 `.rs` 文件（例如 `my_module/mod.rs` 或 `my_module.rs`）都代表一个模块。

例如，如果你在 `src/lib.rs` 中有一个复杂的库，你可以将其拆分为多个文件和目录：

```
my-project/
├── src/
│   ├── main.rs
│   └── lib.rs
│   │   // lib.rs 的内容可能包含 `mod data;` 和 `mod utils;`
│   ├── data/          // 对应 `data` 模块
│   │   ├── mod.rs     // data 模块的根文件
│   │   └── processing.rs // data 模块下的子模块 `processing`
│   └── utils/         // 对应 `utils` 模块
│       └── mod.rs     // utils 模块的根文件
└── Cargo.toml
```

在这种结构中：

- `src/lib.rs` 可以通过 `mod data;` 和 `mod utils;` 声明并引入 `data` 和 `utils` 模块。
- `src/data/mod.rs` 是 `data` 模块的入口点，它可以通过 `mod processing;` 声明 `processing` 子模块。

------

**总结来说，Rust 的文件目录结构和模块系统紧密配合，主要遵循以下原则：**

- **`src/main.rs`**: 默认的二进制 crate 入口。
- **`src/lib.rs`**: 默认的库 crate 入口。
- **`src/bin/` 目录**: 用于存放额外的二进制 crate。
- **子目录和 `mod.rs` (或与目录同名的 `.rs` 文件)**: 用于组织更复杂的模块结构。

理解这些约定将帮助你有效地组织和管理你的 Rust 项目，无论它们是简单的工具还是大型的应用程序。

## Crate 根是什么东西？

在 Rust 里，**Crate 根 (Crate Root)** 可以理解为你整个 Rust 项目（或者说一个 **Crate**）的“总入口文件”。当 Rust 编译器开始编译你的代码时，它就是从这个文件开始读起，然后根据里面的声明，一步步地找到并包含你项目里所有其他的模块和代码。

可以把它想象成一本书的“扉页”或者一个网站的“首页”。

### Crate 根在哪里？

Crate 根文件的位置是约定俗成的：

1. **对于一个库 Crate (Library Crate)**：
   - 它的 Crate 根文件是 **`src/lib.rs`**。
   - 这样的 Crate 编译出来的是一个库（就像一个工具箱，提供很多功能给其他代码使用），而不是一个独立运行的程序。
2. **对于一个二进制 Crate (Binary Crate)**：
   - 它的 Crate 根文件是 **`src/main.rs`**。
   - 这样的 Crate 编译出来的是一个可执行程序（就像一个应用程序，可以直接运行）。

### Crate 根的作用是什么？

Crate 根文件主要有以下几个作用：

- **编译起点**：编译器总是从这里开始扫描和解析你的代码。
- **模块声明**：你会在 Crate 根文件中使用 `mod` 关键字来声明你 Crate 中所有顶级的模块。比如，在 `src/main.rs` 中写 `mod garden;`，就是告诉编译器：“我有一个叫做 `garden` 的模块，请去 `src/garden.rs` 或 `src/garden/mod.rs` 找到它的代码。”
- **公共接口（对于库 Crate）**：如果你的 Crate 是一个库，`src/lib.rs` 就定义了这个库向外部暴露的公共 API。其他项目在使用你的库时，会通过这个文件来访问你的功能。
- **程序入口（对于二进制 Crate）**：如果你的 Crate 是一个二进制程序，`src/main.rs` 中的 `fn main()` 函数就是程序执行的起点。

------

**简单来说：**

**Crate 根**就是你的 Rust 项目（一个 Crate）的“大脑”或者“指挥中心”，它告诉编译器如何找到并组织你所有的代码。没有它，编译器就不知道从何开始构建你的程序或库。

> 官方文档依旧不说人话

## 定义模块来控制作用域与私有性

在本节，我们将讨论模块和其它一些关于模块系统的部分，如允许你命名项的 *路径*（*paths*）；用来将路径引入作用域的 `use` 关键字；以及使项变为公有的 `pub` 关键字。我们还将讨论 `as` 关键字、外部包（external packages）和 glob 运算符（glob operator）。

首先，我们将从一系列的规则开始，在你未来组织代码的时候，这些规则可被用作简单的参考。接下来我们将会详细的解释每条规则。

## [模块小抄（Cheat Sheet）](https://kaisery.github.io/trpl-zh-cn/ch07-02-defining-modules-to-control-scope-and-privacy.html#模块小抄cheat-sheet)

在深入了解模块和路径的细节之前，这里提供一个简单的参考，用来解释模块、路径、`use`关键词和`pub`关键词如何在编译器中工作，以及大部分开发者如何组织他们的代码。我们将在本章中举例说明每条规则，但这是回顾模块工作原理的绝佳参考。

- **从 crate 根节点开始**: 当编译一个 crate, 编译器首先在 crate 根文件（通常，对于一个库 crate 而言是 *src/lib.rs*，对于一个二进制 crate 而言是 *src/main.rs*）中寻找需要被编译的代码。

- 声明模块

  : 在 crate 根文件中，你可以声明一个新模块；比如，用

   

  ```rust
  mod garden;
  ```

  声明了一个叫做

  ```rust
  garden
  ```

  的模块。编译器会在下列路径中寻找模块代码：
  
  - 内联，用大括号替换 `mod garden` 后跟的分号
- 在文件 *src/garden.rs*
  - 在文件 *src/garden/mod.rs*

- 声明子模块

  : 在除了 crate 根节点以外的任何文件中，你可以定义子模块。比如，你可能在src/garden.rs 中声明

  ```
  mod vegetables;
  ```

  。编译器会在以父模块命名的目录中寻找子模块代码：

  - 内联，直接在 `mod vegetables` 后方不是一个分号而是一个大括号
- 在文件 *src/garden/vegetables.rs*
  - 在文件 *src/garden/vegetables/mod.rs*

- **模块中的代码路径**: 一旦一个模块是你 crate 的一部分，你可以在隐私规则允许的前提下，从同一个 crate 内的任意地方，通过代码路径引用该模块的代码。举例而言，一个 garden vegetables 模块下的 `Asparagus` 类型可以通过 `crate::garden::vegetables::Asparagus` 访问。

- **私有 vs 公用**: 一个模块里的代码默认对其父模块私有。为了使一个模块公用，应当在声明时使用 `pub mod` 替代 `mod`。为了使一个公用模块内部的成员公用，应当在声明前使用`pub`。

- **`use` 关键字**: 在一个作用域内，`use`关键字创建了一个项的快捷方式，用来减少长路径的重复。在任何可以引用 `crate::garden::vegetables::Asparagus` 的作用域，你可以通过 `use crate::garden::vegetables::Asparagus;` 创建一个快捷方式，然后你就可以在作用域中只写 `Asparagus` 来使用该类型。

这里我们创建一个名为`backyard`的二进制 crate 来说明这些规则。该 crate 的路径同样命名为`backyard`，该路径包含了这些文件和目录：

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

这个例子中的 crate 根文件是 *src/main.rs*，该文件包含了：

文件名：src/main.rs

```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {plant:?}!");
}
```

`pub mod garden;` 行告诉编译器将 *src/garden.rs* 中发现的代码包含进来：

文件名：src/garden.rs

```rust
pub mod vegetables;
```

在此处，`pub mod vegetables;` 意味着在 *src/garden/vegetables.rs* 中的代码也应该被包含。这些代码是：

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

现在让我们深入了解这些规则的细节并在实践中演示它们！

### 在模块中对相关代码进行分组

**模块**让我们可以将一个 crate 中的代码进行分组，以提高可读性与重用性。因为一个模块中的代码默认是私有的，所以还可以利用模块控制项的**私有性**（*privacy*）。私有项是不可为外部使用的内在详细实现。我们也可以将模块和它其中的项标记为公开的，这样，外部代码就可以使用并依赖于它们。

作为示例，让我们编写一个提供餐厅功能的库 `crate`。我们将定义函数的签名，但将其函数体留空以便将注意力集中在代码的组织结构上而不是餐厅实现的细节。

在餐饮业，餐馆中会有一些地方被称之为**前台**（*front of house*），还有另外一些地方被称之为**后台**（*back of house*）。前台是招待顾客的地方；这包括接待员为顾客安排座位、服务员接受点单和付款、调酒师制作饮品的地方。后台则是厨师和烹饪人员在厨房工作、洗碗工清理餐具，以及经理处理行政事务的区域。

为了以这种方式构建我们的 `crate`，我们可以将其功能组织到嵌套模块中。通过执行 `cargo new restaurant --lib` 来创建一个新的名为 `restaurant` 的库。然后将示例 7-1 中所罗列出来的代码放入 *src/lib.rs* 中，来定义一些模块和函数签名；这段代码即为前台部分。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}
```

示例 7-1：一个包含了其他内置了函数的模块的 `front_of_house` 模块

我们使用 `mod` 关键字来定义模块，后跟模块名（本例中叫做 `front_of_house`），并且用花括号包围模块的主体。在模块内，我们还可以定义其它的模块，就像本例中的 `hosting` 和 `serving` 模块。模块还可以保存一些定义的其它项，比如结构体、枚举、常量、trait、或者如示例 7-1 所示的函数。

通过使用模块，我们可以将相关的定义分组到一起，并指出它们为什么相关。程序员可以通过使用这段代码，更加容易地找到他们想要的定义，因为他们可以基于分组来对代码进行导航，而不需要阅读所有的定义。程序员向这段代码中添加一个新的功能时，他们也会知道代码应该放置在何处，可以保持程序的组织性。

在前面我们提到了，`src/main.rs` 和 `src/lib.rs` 叫做 crate 根。之所以这样叫它们是因为这两个文件的内容都分别在 crate 模块结构的根组成了一个名为 `crate` 的模块，该结构被称为**模块树**（*module tree*）。

示例 7-2 展示了示例 7-1 中模块树的结构。

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

示例 7-2: 示例 7-1 中代码的模块树

这个树展示了一些模块是如何被嵌入到另一个模块的（例如，`hosting` 嵌套在 `front_of_house` 中）。这个树还展示了一些模块是互为**兄弟**（*siblings*）的，这意味着它们定义在同一模块中；`hosting` 和 `serving` 被一起定义在 `front_of_house` 中。继续沿用家庭关系的比喻，如果一个模块 A 被包含在模块 B 中，我们将模块 A 称为模块 B 的 **子**（*child*）模块，模块 B 则是模块 A 的 **父**（*parent*）模块。注意，整个模块树都植根于名为 `crate` 的隐式模块下。

这个模块树可能会令你想起电脑上文件系统的目录树；这是一个非常恰当的类比！就像文件系统的目录，你可以使用模块来组织你的代码。并且，就像目录中的文件，我们需要一种方法来找到模块。

> 上面这段话多少看的人有点懵逼,我们现在让AI来说点人话

### **1. Rust 代码的起点：从“大门”开始找**

你的整个 Rust 项目，无论多大，都有一个固定的“大门”。

- 如果你写的是一个可执行程序（像一个 `.exe` 文件），大门通常是 **`src/main.rs`**。
- 如果你写的是一个库（给别人用的代码包），大门就是 **`src/lib.rs`**。

编译器盖房子时，第一步就是从这个“大门”开始，一步步往里找代码。

### **2. 声明模块：给你的代码分房间**

就像家里有客厅、卧室、厨房一样，你的代码也需要分门别类。在“大门”文件里（比如 `src/main.rs`），你可以用 `mod 房间名;` 来声明一个新的“房间”。

比如：`mod garden;`

这告诉编译器：“嘿，我有一个叫 `garden` 的房间！” 编译器就会去以下几个地方找这个房间的具体图纸：

- **直接在大括号里：** 你可以直接在 `mod garden` 后面跟一对 `{}`，把所有代码写在里面，就像一个“开放式厨房”。
- **在同级文件里：** 编译器会去 `src/garden.rs` 这个文件里找。
- **在同级文件夹里：** 编译器会去 `src/garden/mod.rs` 这个文件里找（`mod.rs` 文件就像这个文件夹的“总说明书”）。

### **3. 声明子模块：房间里再隔小间**

如果你的“房间”里内容太多，还可以再隔出“小间”。比如，你已经在 `src/garden.rs`（也就是 `garden` 房间的图纸）里，想再分一个 `vegetables` 的“小间”。

你可以写：`mod vegetables;`

这时，编译器就知道要去 `garden` 这个“房间”里找 `vegetables` 这个“小间”的图纸了。它会去：

- **直接在大括号里：** 和上面一样，可以直接写在大括号里。
- **在父房间的子文件夹里：** 编译器会去 `src/garden/vegetables.rs` 这个文件里找。
- **在父房间的子文件夹里的 `mod.rs` 文件：** 编译器会去 `src/garden/vegetables/mod.rs` 这个文件里找。

### **4. 模块中的代码路径：怎么找到某个东西？**

想象你在一个大房子里，要描述某个东西在哪儿。你会说“在客厅里沙发的旁边”。在 Rust 里，你要引用一个模块里的东西，也需要指明它的“路径”。

比如，如果你有个叫 `Asparagus` 的类型，它在 `garden` 房间的 `vegetables` 小间里，那么它的完整路径就是： **`crate::garden::vegetables::Asparagus`**

- `crate::` 就代表你这个项目的根目录（也就是那个“大门”）。
- 后面跟着的就是模块一层层嵌套的名字。

### **5. 私有 vs 公用：谁能看到我的东西？**

这是关于隐私的规则：

- **默认是私有的：** 你在一个模块里写的东西，默认只有这个模块自己能用，外面是看不到的。就像你卧室里的东西，客厅里的人看不到。
- **`pub mod` 公开模块：** 如果你想让整个 `garden` 房间都能被外面的人看到，你需要在声明它的时候加上 `pub`，写成 `pub mod garden;`。
- **`pub` 公开模块内部成员：** 即使一个房间是公开的，里面的家具（比如结构体、函数）默认还是私有的。如果你想让 `Asparagus` 这个“家具”也能被外面看到，你也要在它前面加上 `pub`。

### **6. `use` 关键字：走捷径，少写路名！**

想象你每次去厨房都要说“去客厅，然后穿过餐厅，再到厨房”。这太麻烦了！

`use` 关键字就是用来走捷径的。比如：

```
use crate::garden::vegetables::Asparagus;
```

这行代码就是告诉 Rust：“以后在这个地方，我提到 `Asparagus`，就直接指代 `crate::garden::vegetables::Asparagus` 这个东西，不用每次都写那么长一串路径了！”

这样，你以后在这个文件里直接写 `Asparagus` 就可以了，方便很多。

### **用 `backyard` 例子来说明：**

现在，我们把上面的规则套到那个 `backyard` 项目里，就像搭建一个真实的小花园：

1. **`src/main.rs` (主屋，程序的入口)：**
   - `pub mod garden;`：主屋里声明了一个**公开**的 `garden` 房间（所以编译器去 `src/garden.rs` 里找这个房间的图纸）。
   - `use crate::garden::vegetables::Asparagus;`：这里设置了一个捷径！以后我直接写 `Asparagus`，就知道指的是 `garden` 房间里 `vegetables` 小间里的 `Asparagus` 这个植物。
   - `fn main() { ... }`：这是程序真正运行的地方，就像主屋里的客厅。它创建了一个 `Asparagus` 植物，并打印出来。
2. **`src/garden.rs` (花园的图纸)：**
   - `pub mod vegetables;`：花园里声明了一个**公开**的 `vegetables` 小间（所以编译器去 `src/garden/vegetables.rs` 里找这个小间的图纸）。
3. **`src/garden/vegetables.rs` (蔬菜小间的图纸)：**
   - `pub struct Asparagus {}`：这里定义了一个**公开**的 `Asparagus` 植物结构体。

通过这些规则，整个项目就被组织得井井有条，各个部分各司其职，又可以通过明确的路径相互引用。

`src/garden.rs` **不是**一个独立的库 crate。它只是一个**模块文件**，是 `backyard` 这个二进制 crate 的一部分。

### `src/garden.rs` 为什么不在 `lib` 目录下？

`src/garden.rs` 之所以直接放在 `src` 目录下，是因为它被 `src/main.rs` 中的 `pub mod garden;` 这行代码**声明并包含了进来**。

当你在 `src/main.rs`（或 `src/lib.rs`）中声明 `mod garden;` 时，Rust 编译器会按照你之前看到的规则去寻找 `garden` 模块的代码：

1. 在 `src/garden.rs` 文件中。
2. 在 `src/garden/mod.rs` 文件中。

在这个例子中，它找到了 `src/garden.rs`，并将其中的代码（包括它声明的 `vegetables` 模块）**作为 `main.rs` 所属的这个二进制 crate 的一部分**编译进去。所以，`garden.rs` 仅仅是 `backyard` 这个二进制 crate 内部的一个模块，用来组织代码。

### 那 `lib` 目录是干啥用的？

`lib` 目录（具体来说是 `src/lib.rs` 文件）是用来创建**库 crate** 的。

- **库 crate** 是一种可以被其他项目（甚至是你自己项目的二进制 crate）**引用和复用**的代码包。它不直接生成可执行文件，而是生成一个库文件（例如 `.rlib` 文件）。
- 当你的项目包含 `src/lib.rs` 时，`Cargo` 会自动将其视为一个库 crate 的根文件。

**区别在于：**

- **`src/lib.rs`**：定义的是整个**库 crate** 的入口和结构，它是一个独立的编译单元，旨在被其他代码导入使用。
- **`src/garden.rs`**：定义的是当前**二进制 crate**（或库 crate）内部的一个**模块**，它只是用来组织该 crate 内部的代码，而不是一个独立的、可被其他外部项目直接引用的 crate。

**简而言之：**

`src/lib.rs` 相当于你的项目要对外提供的“工具箱”的“总说明书”。而 `src/garden.rs` 只是你这个工具箱（或你的主程序）内部的一个“抽屉”，用来存放相关工具。

为了更好地理解，我们再回顾一下 Rust 包中不同类型的 Crate 和模块文件的角色：

1. **默认二进制 Crate (`src/main.rs`)**:
   - 这是你运行 `cargo run` 时默认会编译和执行的程序。
   - 必须包含一个 `fn main()` 函数。
   - 它可以声明和使用内部的模块文件（比如本例中的 `src/garden.rs`），这些模块文件最终都成为了这个二进制 Crate 的一部分。
2. **库 Crate (`src/lib.rs`)**:
   - 这是为了提供可复用的代码库而存在的。它不包含 `main` 函数。
   - 编译后生成的是一个库文件（例如 `.rlib` 或 `.so`/`.dll`），而不是可执行文件。
   - 这个库 Crate 可以被其他项目（包括当前包的默认二进制 Crate `src/main.rs` 或其他位于 `src/bin` 目录下的二进制 Crate）引用和使用。
3. **额外二进制 Crate (`src/bin/\*.rs`)**:
   - 如果你的项目需要除了 `src/main.rs` 之外的**其他独立可执行程序**，你就会把它们放在 `src/bin/` 目录下。
   - `src/bin` 目录下的每个 `.rs` 文件都会被编译成一个独立的二进制 Crate，并且每个文件都需要包含一个 `fn main()` 函数。
   - 例如，如果你有 `src/bin/my_tool.rs`，那么你可以用 `cargo run --bin my_tool` 来运行它。

当然可以。在这个餐厅项目中，你可以完全按照 Backyard 项目的模式来组织代码，将餐厅功能模块化，但不是作为一个独立的库 crate，而是作为主二进制 crate (`src/main.rs`) 的一部分来包含。

这意味着：

1. **没有 `src/lib.rs` 文件**：你的项目将不会包含一个独立的库 crate。
2. **`src/main.rs` 作为主入口**：所有功能将最终通过 `src/main.rs` 来使用。
3. **模块文件直接在 `src/` 目录下或其子目录中**：像 `src/garden.rs` 在 Backyard 项目中一样，你可以在 `src/` 目录下创建 `restaurant_logic.rs` 或者 `front_of_house.rs` 等文件，这些文件会通过 `mod` 声明被 `src/main.rs` 包含进来。

**修改后的餐厅项目结构（类似 Backyard）**：

首先，创建一个新的二进制项目 (如果你之前创建的是库项目，可以新建一个)：

Bash

```
cargo new restaurant_app
cd restaurant_app
```

然后，修改 `src/main.rs` 来声明和使用模块。

**项目文件结构：**

```
restaurant_app/
├── Cargo.toml
└── src/
    ├── front_of_house.rs  # 对应 Backyard 项目中的 garden.rs
    ├── front_of_house/    # 对应 Backyard 项目中的 garden/ 目录
    │   └── hosting.rs     # 对应 Backyard 项目中的 garden/vegetables.rs
    └── main.rs
```

**`src/main.rs` 的内容：**

```rust
// 声明 front_of_house 模块。因为它是顶级模块，并且我们想让它可以在文件中定义，所以用 `mod 模块名;`
// 它会去查找 src/front_of_house.rs 或 src/front_of_house/mod.rs
mod front_of_house; //

fn main() {
    // 假设 front_of_house::hosting::add_to_waitlist 是公开的
    front_of_house::hosting::add_to_waitlist(); //
    println!("顾客已添加到等位列表！");
}
```

**解释 `src/main.rs`：**

- `mod front_of_house;` 这行告诉编译器，有一个名为 `front_of_house` 的模块，它的代码不在 `main.rs` 中内联，而是要去 `src/front_of_house.rs` 或 `src/front_of_house/mod.rs` 中寻找。在这个例子中，我们选择在 `src/front_of_house.rs` 中定义它。
- `fn main()` 中通过 `front_of_house::hosting::add_to_waitlist()` 来调用功能。这要求 `front_of_house` 模块及其内部的 `hosting` 模块和 `add_to_waitlist` 函数都被声明为 `pub`，才能被 `main` 函数访问。

**`src/front_of_house.rs` 的内容：**

```rust
// 在 front_of_house 模块内声明 hosting 和 serving 子模块
// 它们会去查找 src/front_of_house/hosting.rs 或 src/front_of_house/hosting/mod.rs
// 以及 src/front_of_house/serving.rs 或 src/front_of_house/serving/mod.rs
pub mod hosting; //
pub mod serving; //

// 你也可以在这里直接定义一些 front_of_house 模块特有的函数或结构体
// pub fn greet_customer() {
//     println!("欢迎光临！");
// }
```

**解释 `src/front_of_house.rs`：**

- 这里我们声明了 `hosting` 和 `serving` 这两个子模块，并使用了 `pub` 关键字，表示它们是公开的，可以被它们的父模块（`front_of_house`）和通过路径访问到 `front_of_house` 的其他代码（例如 `main.rs`）使用。
- 由于是在 `src/front_of_house.rs` 中声明的，Rust 会继续在 `src/front_of_house/` 目录下查找 `hosting.rs` 和 `serving.rs`。

**`src/front_of_house/hosting.rs` 的内容：**

Rust

```rust
pub fn add_to_waitlist() { //
    // 实际的等位逻辑
    println!("顾客已加入等位列表。");
}

pub fn seat_at_table() { //
    println!("顾客已入座。");
}
```

**`src/front_of_house/serving.rs` 的内容：**

```rust
pub fn take_order() { //
    println!("正在点餐...");
}

pub fn serve_order() { //
    println!("正在上菜...");
}

pub fn take_payment() { //
    println!("正在付款...");
}
```

运行：

现在你可以运行 cargo run，它会编译并执行 src/main.rs 中的代码，从而调用你组织在不同文件和模块中的餐厅功能。

这种方式的好处是，你不需要一个单独的库 crate 就能将代码分解到多个文件中，保持项目的整洁和组织性。所有的代码最终都编译成一个单一的二进制可执行文件。

### 两种模块化的优劣(独立库 Crate (推荐)单一二进制 Crate (简化))

这两种实现模块化的方式（即：将功能代码放在一个独立的库 crate 中供二进制 crate 使用，或者将所有模块都作为单一二进制 crate 的一部分）没有绝对的“孰优孰劣”，而是**取决于你的项目需求和目标**。

以下是它们的优缺点对比：

### 1. 将功能代码作为独立库 Crate 的一部分（`src/lib.rs` + `src/main.rs`）

**优点：**

- **代码复用性强**：
  - 这是最主要的好处。如果你的核心功能将来可能被其他项目、其他二进制工具或测试用例使用，将其封装在一个库 crate 中是最佳选择。其他项目可以直接添加你的库 crate 作为依赖。
  - 你的二进制 crate (`src/main.rs` 或 `src/bin/*.rs`) 可以像使用任何外部依赖一样 `use` 这个库 crate 中的功能。
- **清晰的职责分离**：库 crate 专注于提供核心功能，而二进制 crate 专注于提供可执行的应用程序入口或命令行界面。这种分离使得代码结构更清晰，更易于理解和维护。
- **更好的测试粒度**：你可以独立地对库 crate 进行单元测试和集成测试，因为它不依赖于 `main` 函数或特定的程序入口，这使得测试更纯粹和高效。
- **减少主二进制文件大小（潜在）**：如果你的库 crate 包含大量可选功能，并且你的二进制 crate 只使用其中一部分，那么在某些编译优化下，最终的二进制文件可能更精简。
- **便于分享**：如果你想将你的核心功能发布到 crates.io 上供全球 Rust 开发者使用，那么它必须是一个库 crate。

**缺点：**

- **初始设置略复杂**：你需要管理 `src/lib.rs` 和 `src/main.rs` 两个入口文件，并确保它们之间的 `use` 路径正确。
- **额外的层级**：对于非常简单的项目，引入一个独立的库 crate 可能会显得有点“过度设计”，增加了不必要的抽象层级。



### 2. 将所有模块都作为单一二进制 Crate 的一部分（只有 `src/main.rs` 和其内部模块文件）



**优点：**

- **简单直接**：对于小型或中型项目，如果代码不会被其他项目复用，这种方式是最直接和简单的。你只需要管理一个主要的 `main.rs` 入口文件。
- **更容易快速启动**：当你开始一个新项目，不确定哪些代码将来会复用时，这种结构更容易开始。
- **所有代码编译成一个文件**：最终生成一个单一的可执行文件，没有额外的库文件需要管理。

**缺点：**

- **代码复用性差**：如果将来需要将某些功能用于其他项目，或者想将其作为独立的库发布，你就需要手动将这部分代码提取出来，并重新组织成一个库 crate，这会涉及到代码的迁移和路径的调整。
- **职责不明确**：所有的功能都被包含在同一个二进制 Crate 中，可能导致在大型项目中，核心逻辑和应用程序入口的边界变得模糊。
- **测试可能更复杂**：如果你想针对非 `main` 函数的某个内部模块进行独立的单元测试，你可能需要编写一些辅助代码或者使用特定的测试宏，而不能像库 crate 那样直接暴露接口进行测试。



### 总结与选择建议：



| 特性/场景      | 独立库 Crate (推荐)                      | 单一二进制 Crate (简化)                   |
| -------------- | ---------------------------------------- | ----------------------------------------- |
| **复用性**     | 极佳，核心逻辑可被其他 Crate 引用        | 差，功能不易被外部项目复用                |
| **项目规模**   | 适合中大型项目，或有复用需求的模块化设计 | 适合小型或一次性工具，无需对外暴露库功能  |
| **代码组织**   | 清晰的职责分离，库归库，应用归应用       | 所有模块都在一个 Crate 内，结构可能更扁平 |
| **测试**       | 便于独立测试核心库功能                   | 内部模块测试可能需要更多设置              |
| **发布**       | 可发布到 crates.io，供他人使用           | 通常只发布可执行文件，不作为库发布        |
| **未来扩展性** | 高，方便拆分功能和引入新二进制程序       | 低，后续功能拆分或复用成本较高            |

**我的建议是：**

- **如果你不确定将来是否需要复用代码，或者项目规模可能会增长，** **优先选择“独立库 Crate + 二进制 Crate”的模式**。即使是现在看起来很小的功能，未来也可能被需要。这种结构一开始可能多了一点点设置，但从长远来看可以节省大量重构时间。
- **如果你确信这是一个非常小、一次性、且永远不会被其他项目复用的工具或脚本，** 那么只使用单一二进制 Crate 的方式会更简单直接。

大多数成熟的 Rust 项目，即使是最终产生一个可执行程序的项目，也会倾向于将其核心逻辑放在 `src/lib.rs` 中，并通过 `src/main.rs` 来调用这个库，从而获得更好的模块化、复用性和可测试性。

在我们写C语言项目时,经常会有src,inc目录,我们自己编写或者下载好的.h.c文件塞进去,将它们添加进编译列表,然后就可以在项目中使用它们了,RUST中的模块系统是与之类似的,以下是AI给出的两者之间的联系以及区别

## Rust 模块系统与 C 语言文件组织对比

在 C 语言项目中，我们通常将源代码 (`.c` 文件) 放在 `src` 目录，头文件 (`.h` 文件) 放在 `inc` (或 `include`) 目录。Rust 的模块系统与 C 语言这种文件组织方式在**逻辑上有一些相似之处，但在实现和理念上存在显著不同**。

### 相似之处

1. **代码组织与复用：** 无论是 C 语言的头文件/源文件分离，还是 Rust 的模块，核心目的都是为了更好地组织代码、提高可读性，并实现代码的复用。通过将相关功能封装在一起，可以避免命名冲突，并使得项目结构更加清晰。
2. **隐藏实现细节：** C 语言中，头文件声明了函数和变量的接口，而 `.c` 文件包含具体实现。外部用户只需要包含头文件，不需要关心 `.c` 文件内部的实现细节。Rust 的模块系统也提供了类似的能力，通过 `pub` 关键字控制可见性，可以隐藏内部实现，只暴露公共 API。
3. **编译单元的概念：** C 语言的 `.c` 文件是独立的编译单元，通过 `#include` 预处理器指令将头文件内容插入到编译单元中。Rust 的模块在编译时也会被编译器处理，形成逻辑上的编译单元。

### 不同之处

1. **声明与定义的绑定方式：**
   - **C 语言：** C 语言通过预处理器宏 `#include` 将头文件的内容（声明）复制到源文件中。编译时，编译器需要同时看到声明和定义，或者通过链接器在链接阶段解析符号。这种方式相对松散，容易出现头文件循环引用、重复定义等问题。
   - **Rust：** Rust 的模块系统是语言级别内置的。它通过 `mod` 关键字定义模块，并通过 `use` 关键字引入路径。Rust 编译器在编译时会理解模块的层级结构和可见性规则，不需要像 C 语言那样手动管理头文件依赖。这使得 Rust 的模块管理更加健壮和安全。
2. **可见性控制：**
   - **C 语言：** C 语言主要通过头文件来控制可见性。头文件中声明的符号默认是外部可见的，`.c` 文件中定义的 `static` 变量或函数才是文件私有的。
   - **Rust：** Rust 提供了更细粒度的可见性控制。默认情况下，模块中的项（函数、结构体、枚举等）是私有的 (`private`)，只有通过 `pub` 关键字明确声明的项才是公共的。此外，Rust 还支持 `pub(crate)`（当前 crate 可见）、`pub(super)`（父模块可见）等更灵活的可见性修饰符，极大地增强了封装性。
3. **命名空间与路径：**
   - **C 语言：** C 语言没有内置的命名空间机制，通常通过前缀（例如 `my_library_function`）来避免不同库之间的命名冲突。头文件的路径管理也相对简单，就是文件系统的路径。
   - **Rust：** Rust 的模块系统本身就提供了强大的命名空间管理。每个模块都有自己的路径，例如 `crate::module_name::sub_module::item`。通过 `use` 关键字，我们可以引入特定的项到当前作用域，或者使用 `as` 关键字为引入的项创建别名，从而解决潜在的命名冲突。这种基于路径的命名空间机制使得代码组织更加清晰和安全。
4. **文件与模块的对应关系：**
   - **C 语言：** 通常一个 `.c` 文件对应一个独立的编译单元，并可能有一个对应的 `.h` 文件用于声明接口。
   - **Rust：** 在 Rust 中，一个文件可以包含多个模块，或者一个模块可以分散在多个文件中（通过 `mod name;` 语法）。Rust 的文件系统布局约定是模块系统的一部分，例如，一个 `mod foo;` 声明会查找 `foo.rs` 文件或 `foo/mod.rs` 目录来加载模块内容。这种灵活的对应关系使得代码组织可以更好地反映逻辑结构。
5. **宏与预处理器：**
   - **C 语言：** 严重依赖预处理器宏（`#define`, `#ifdef` 等）进行条件编译、代码生成等操作。
   - **Rust：** Rust 拥有功能更强大的宏系统（过程宏、声明宏），它们在编译时执行，并且具有更好的类型安全性。Rust 强调零成本抽象，通过类型系统和编译器在编译阶段捕获错误，而不是依赖运行时检查或预处理器。
