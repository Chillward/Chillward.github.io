---
title: RUST学习日记之模块03
date: 2025-07-24 18:11:52
tags: RUST
categories: Rust
---

> 我们依旧先来欣赏一下官方的高速神言,这一节官方主要就是讲了一下 use as还有pub use这三个东西

## [使用 `use` 关键字将名称引入作用域](https://rustwiki.org/zh-CN/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#使用-use-关键字将名称引入作用域)

到目前为止，似乎我们编写的用于调用函数的路径都很冗长且重复，并不方便。例如，示例 7-7 中，无论我们选择 `add_to_waitlist` 函数的绝对路径还是相对路径，每次我们想要调用 `add_to_waitlist` 时，都必须指定 `front_of_house` 和 `hosting`。幸运的是，有一种方法可以简化这个过程。我们可以使用 `use` 关键字将路径一次性引入作用域，然后调用该路径中的项，就如同它们是本地项一样。

在示例 7-11 中，我们将 `crate::front_of_house::hosting` 模块引入了 `eat_at_restaurant` 函数的作用域，而我们只需要指定 `hosting::add_to_waitlist` 即可在 `eat_at_restaurant` 中调用 `add_to_waitlist` 函数。

```rust

mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

示例 7-11: 使用 `use` 将模块引入作用域

在作用域中增加 `use` 和路径类似于在文件系统中创建软连接（符号连接，symbolic link）。通过在 crate 根增加 `use crate::front_of_house::hosting`，现在 `hosting` 在作用域中就是有效的名称了，如同 `hosting` 模块被定义于 crate 根一样。通过 `use` 引入作用域的路径也会检查私有性，同其它路径一样。

你还可以使用 `use` 和相对路径来将一个项引入作用域。示例 7-12 展示了如何指定相对路径来取得与示例 7-11 中一样的行为。

```rust

mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

示例 7-12: 使用 `use` 和相对路径将模块引入作用域

### [创建惯用的 `use` 路径](https://rustwiki.org/zh-CN/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#创建惯用的-use-路径)

在示例 7-11 中，你可能会比较疑惑，为什么我们是指定 `use crate::front_of_house::hosting`，然后在 `eat_at_restaurant` 中调用 `hosting::add_to_waitlist`，而不是通过指定一直到 `add_to_waitlist` 函数的 `use` 路径来得到相同的结果，如示例 7-13 所示。

```rust

mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```

示例 7-13: 使用 `use` 将 `add_to_waitlist` 函数引入作用域，这并不符合习惯

虽然示例 7-11 和 7-13 都完成了相同的任务，但示例 7-11 是使用 `use` 将函数引入作用域的习惯用法。使用 `use` 将函数的父模块引入作用域意味着我们必须在调用函数时指定父模块，这样可以清晰地表明函数不是在本地定义的，同时使完整路径的重复度最小化。示例 7-13 中的代码则未表明 `add_to_waitlist` 是在哪里被定义的。

另一方面，使用 `use` 引入结构体、枚举和其他项时，习惯是指定它们的完整路径。示例 7-14 展示了将 `HashMap` 结构体引入二进制 crate 作用域的习惯用法。

```rust

use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

示例 7-14: 将 `HashMap` 引入作用域的习惯用法

这种习惯用法背后没有什么硬性要求：它只是一种惯例，人们已经习惯了以这种方式阅读和编写 Rust 代码。

> 其实也就说了这么些内容

**使用 `use` 引入函数时，习惯上是将函数的父模块引入作用域。** 这样，在调用函数时，你需要写 `父模块::函数名()`，比如 `hosting::add_to_waitlist()`。这样做的好处是，可以清晰地表明这个函数不是当前模块定义的，同时避免了每次都写完整的冗长路径。

**使用 `use` 引入结构体、枚举或其他项时，习惯上是直接引入到该项本身。** 这样，你可以直接使用该项的名称，比如 `HashMap::new()`。

这个习惯用法有一个例外，那就是我们想使用 `use` 语句将两个具有相同名称的项带入作用域，因为 Rust 不允许这样做。示例 7-15 展示了如何将两个具有相同名称但不同父模块的 `Result` 类型引入作用域，以及如何引用它们。



```rust

use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

示例 7-15: 使用父模块将两个具有相同名称的类型引入同一作用域

如你所见，使用父模块可以区分这两个 `Result` 类型。如果我们是指定 `use std::fmt::Result` 和 `use std::io::Result`，我们将在同一作用域拥有了两个 `Result` 类型，当我们使用 `Result` 时，Rust 则不知道我们要用的是哪个。

> 接下来就是引入 as

~~~ RUST
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
~~~

　这个其实看两眼也就明白了,不用什么复杂的解释

然后还有pub use,我的理解就是你自己编写的模块中通过use引用了别的模块的内容,想要使这些内容可以被引用你模块的人使用的话,就需要pub use语句

> 以下是ai给出的解释

`pub use` 的核心作用，正是你描述的这个场景：当你编写一个模块（或者库）时，你可能会**在内部 `use` 其他模块或类型**来方便自己使用。但是，如果你希望**使用你这个模块的外部代码**也能通过你的模块路径来访问这些被你 `use` 进来的内容，那么你就需要使用 `pub use`。

简单来说：

- **`use`：** 仅仅是你自己模块内部的“快捷方式”，方便你在本模块中使用。外部模块无法通过你的模块来访问这些被 `use` 进来的东西。
- **`pub use`：** 不仅创建了内部的“快捷方式”，还**公开了**这个快捷方式。这样，外部模块就可以通过你的模块路径来访问这个被重导出的内容，就好像这些内容就是你模块自己定义的一样。

这就像是你开了一家商店（你的模块），你从批发商那里进了些商品（其他模块的内容）。

- 如果你只是自己用（`use`），这些商品就在你的库房里，只有你能看到。
- 但如果你想把这些商品摆在你的店里卖（`pub use`），那么顾客（外部代码）就可以通过你的商店来购买这些商品了，他们甚至不需要知道你最初是从哪个批发商那里进的货。

`pub use` 允许你提供一个更清晰、更简洁的 API 给你的用户，同时隐藏了你内部的组织结构,语法如下:

~~~　rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
fn main() {}

~~~

还剩下一些杂七杂八的东西

下面是这段内容讲的三个主要“玩意儿”：

- 你问得很好！这段内容主要围绕着如何在 Rust 项目中使用外部包（crate）以及如何优化 `use` 语句来管理这些引入的依赖。

  下面是这段内容讲的三个主要“玩意儿”：

  ### 1. 如何使用外部包（Crate）

  Rust 项目使用 **Cargo** 这个构建系统和包管理器来管理依赖。

  - **声明依赖：** 当你需要使用一个外部包时（比如 `rand`），你需要在项目的 `Cargo.toml` 文件中的 `[dependencies]` 部分添加这个包的名称和版本号。Cargo 会自动从 **crates.io**（Rust 社区的包注册表）下载这个包及其所有必要的依赖。

    Ini, TOML

    ```
    [dependencies]
    rand = "0.8.3" # 示例：声明对rand包的依赖
    ```
  
  - **引入作用域：** 仅仅声明依赖还不够。你还需要使用 `use` 关键字，以 **绝对路径** 的方式将外部包中你想要使用的特定项（函数、结构体、trait 等）引入到你的代码的作用域中，这样你才能直接使用它们。
  
    Rust
  
    ```
    use rand::Rng; // 将rand包中的Rng trait引入作用域
    ```

    值得注意的是，**标准库 (`std`)** 也是一个特殊的外部 crate。虽然你不需要在 `Cargo.toml` 中声明它，但你仍然需要使用 `use` 语句来引入 `std` 中你想要使用的项，例如 `use std::collections::HashMap;`。

  ### 2. 优化 `use` 语句：嵌套路径
  
  当你的代码需要从同一个模块或同一个包中引入多个项时，逐行 `use` 会让代码变得冗长。Rust 提供了 **嵌套路径** 的语法来简化这个过程。

  - **基本嵌套：** 如果多个项共享同一个路径前缀，你可以将它们用大括号 `{}` 包裹起来。

    Rust

    ```
    // 之前：
    // use std::cmp::Ordering;
    // use std::io;
    
    // 之后：一行搞定
    use std::{cmp::Ordering, io};
    ```
  
  - **`self` 关键字：** 当你想引入一个模块本身，同时又想引入这个模块下的某个子项时，可以在嵌套路径中使用 `self` 关键字。
  
    Rust
  
    ```
    // 之前：
    // use std::io;
    // use std::io::Write;
    
    // 之后：一行搞定，同时引入了io模块和io::Write
    use std::io::{self, Write};
    ```
  
  ### 3. 引入所有公有定义：Glob 运算符 `*`
  
  如果你想将一个模块下所有 **公有** 的项都引入到当前作用域，可以使用 **glob 运算符 `\*`**。
  
  - **用法：** 在路径的末尾加上 `*`。
  
    Rust
  
    ```
    use std::collections::*; // 引入std::collections模块中所有的公有项
    ```

  - **注意事项：** 尽管方便，但使用 glob 运算符时需要**谨慎**。它会使代码的可读性降低，因为你很难一眼看出某个名称是来自哪里，可能会导致名称冲突。它通常在测试模块（为了方便测试所有功能）或特定模式（如 `prelude` 模式）中使用。

  这里还有，并提供了两种技巧 (`{}` 嵌套路径和 `*` glob 运算符) 来更简洁地管理 `use` 语句，从而提高代码的可读性和维护性。

  这段代码展示了 Rust 中更细粒度的可见性控制，也就是如何使用 `pub(in path)`、`pub(self)` 和 `pub(super)` 来限制一个项（比如函数）的可见范围。

  ### 深入理解 Rust 的可见性控制

  ~~~ rust
  // 一个名为 `my_mod` 的模块
  mod my_mod {
      // 模块中的项默认具有私有的可见性
      fn private_function() {
          println!("called `my_mod::private_function()`");
      }
  
      // 使用 `pub` 修饰语来改变默认可见性。
      pub fn function() {
          println!("called `my_mod::function()`");
      }
  
      // 在同一模块中，项可以访问其它项，即使它是私有的。
      pub fn indirect_access() {
          print!("called `my_mod::indirect_access()`, that\n> ");
          private_function();
      }
  
      // 模块也可以嵌套
      pub mod nested {
          pub fn function() {
              println!("called `my_mod::nested::function()`");
          }
  
          #[allow(dead_code)]
          fn private_function() {
              println!("called `my_mod::nested::private_function()`");
          }
  
          // 使用 `pub(in path)` 语法定义的函数只在给定的路径中可见。
          // `path` 必须是父模块（parent module）或祖先模块（ancestor module）
          pub(in crate::my_mod) fn public_function_in_my_mod() {
              print!("called `my_mod::nested::public_function_in_my_mod()`, that\n > ");
              public_function_in_nested()
          }
  
          // 使用 `pub(self)` 语法定义的函数则只在当前模块中可见。
          pub(self) fn public_function_in_nested() {
              println!("called `my_mod::nested::public_function_in_nested");
          }
  
          // 使用 `pub(super)` 语法定义的函数只在父模块中可见。
          pub(super) fn public_function_in_super_mod() {
              println!("called my_mod::nested::public_function_in_super_mod");
          }
      }
  
      pub fn call_public_function_in_my_mod() {
          print!("called `my_mod::call_public_funcion_in_my_mod()`, that\n> ");
          nested::public_function_in_my_mod();
          print!("> ");
          nested::public_function_in_super_mod();
      }
  
      // `pub(crate)` 使得函数只在当前 crate 中可见
      pub(crate) fn public_function_in_crate() {
          println!("called `my_mod::public_function_in_crate()");
      }
  
      // 嵌套模块的可见性遵循相同的规则
      mod private_nested {
          #[allow(dead_code)]
          pub fn function() {
              println!("called `my_mod::private_nested::function()`");
          }
      }
  }
  
  fn function() {
      println!("called `function()`");
  }
  
  fn main() {
      // 模块机制消除了相同名字的项之间的歧义。
      function();
      my_mod::function();
  
      // 公有项，包括嵌套模块内的，都可以在父模块外部访问。
      my_mod::indirect_access();
      my_mod::nested::function();
      my_mod::call_public_function_in_my_mod();
  
      // pub(crate) 项可以在同一个 crate 中的任何地方访问
      my_mod::public_function_in_crate();
  
      // pub(in path) 项只能在指定的模块中访问
      // 报错！函数 `public_function_in_my_mod` 是私有的
      //my_mod::nested::public_function_in_my_mod();
      // 试一试 ^ 取消该行的注释
  
      // 模块的私有项不能直接访问，即便它是嵌套在公有模块内部的
  
      // 报错！`private_function` 是私有的
      //my_mod::private_function();
      // 试一试 ^ 取消此行注释
  
      // 报错！`private_function` 是私有的
      //my_mod::nested::private_function();
      // 试一试 ^ 取消此行的注释
  
      // Error! `private_nested` is a private module
      //my_mod::private_nested::function();
      // 试一试 ^ 取消此行的注释
  }
  
  ~~~
  
  
  
  默认情况下，Rust 中的所有项（函数、结构体、枚举、模块等）都是私有的。要让它们在当前作用域之外可见，你需要使用 `pub` 关键字。然而，简单的 `pub` 意味着对**所有**外部代码都可见，这在某些情况下可能过于宽松。
  
  `pub(in path)`、`pub(self)` 和 `pub(super)` 提供了一种方式，让你能够更精确地控制项的可见性，而不是简单地“完全公开”或“完全私有”。
  
  #### 1. `pub(in crate::my_mod)`：指定路径可见性
  
  这个语法允许你将一个项的可见性限制在指定的路径（模块）内部。
  
  - **`pub(in crate::my_mod) fn public_function_in_my_mod()`**
  
    - 这意味着 `public_function_in_my_mod` 这个函数只在 **`crate::my_mod` 模块及其子模块内部可见和可调用**。
    - 在 `my_mod` 外部的代码，即使 `my_mod` 本身是公开的，也无法直接调用 `public_mod::nested::public_function_in_my_mod`。
    - `path` 必须是该项的父模块或祖先模块。你不能指定一个与当前项没有继承关系的模块。
  
    **想象一下：** 你有一个家族企业，`my_mod` 是总公司。这个函数就像是只有总公司内部的员工（包括子公司员工）才能使用的特定工具。外部客户即使知道总公司存在，也无法直接使用这个工具。
  
  #### 2. `pub(self)`：当前模块可见性
  
  `pub(self)` 将可见性限制在定义该项的**当前模块**内部。
  
  - **`pub(self) fn public_function_in_nested()`**
  
    - 这意味着 `public_function_in_nested` 这个函数只在 `my_mod::nested` **模块内部可见**。
    - 即使在 `my_mod` 模块内部（`nested` 的父模块），也无法直接调用 `public_function_in_nested`。只有在 `nested` 模块内部的代码才能调用它。
  
    **想象一下：** `nested` 是公司里的一个特定部门。`public_function_in_nested` 就像是这个部门内部的专用流程，只有这个部门的员工才能使用。总公司或其他部门的员工都不能直接调用这个流程。
  
  #### 3. `pub(super)`：父模块可见性
  
  `pub(super)` 将可见性限制在定义该项的**父模块**内部。
  
  - **`pub(super) fn public_function_in_super_mod()`**
  
    - 这意味着 `public_function_in_super_mod` 这个函数只在 `my_mod` **模块内部可见**。
    - 尽管它定义在 `nested` 模块中，但它只对 `nested` 的父模块（即 `my_mod`）可见。`nested` 模块内部也能调用它，因为父模块的可见性范围包含了子模块。
  
    **想象一下：** 仍然是公司里的一个部门 `nested`。`public_function_in_super_mod` 就像是这个部门为总公司（`my_mod`）提供的内部服务，总公司可以直接调用，但其他部门或外部客户不能直接调用。
  
  ~~~ rust
  fn function() {
      println!("called `function()`");
  }
  
  mod cool {
      pub fn function() {
          println!("called `cool::function()`");
      }
  }
  
  mod my {
      fn function() {
          println!("called `my::function()`");
      }
      
      mod cool {
          pub fn function() {
              println!("called `my::cool::function()`");
          }
      }
      
      pub fn indirect_call() {
          // 让我们从这个作用域中访问所有名为 `function` 的函数！
          print!("called `my::indirect_call()`, that\n> ");
          
          // `self` 关键字表示当前的模块作用域——在这个例子是 `my`。
          // 调用 `self::function()` 和直接调用 `function()` 都得到相同的结果，
          // 因为他们表示相同的函数。
          self::function();
          function();
          
          // 我们也可以使用 `self` 来访问 `my` 内部的另一个模块：
          self::cool::function();
          
          // `super` 关键字表示父作用域（在 `my` 模块外面）。
          super::function();
          
          // 这将在 *crate* 作用域内绑定 `cool::function` 。
          // 在这个例子中，crate 作用域是最外面的作用域。
          {
              use crate::cool::function as root_function;
              root_function();
          }
      }
  }
  
  fn main() {
      my::indirect_call();
  }
  
  ~~~
  
  
  
  
  
  ### 总结
  
  这些细粒度的可见性控制非常有用，它们允许你：
  
  - **封装内部实现细节：** 将一些只应该在特定范围内部使用的函数或数据隐藏起来，避免外部滥用或不当修改。
  - **构建清晰的 API：** 明确哪些部分是库的公共接口，哪些是内部辅助功能。
  - **增强代码安全性：** 限制了对某些敏感操作的访问。
  
  RUST的这个模块管理系统真的挺复杂的,估计得在实战中用它两回才能真正学得会.
