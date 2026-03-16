---
title: RUST学习日记之模块02
date: 2025-07-24 10:23:54
tags: RUST
categories: Rust
---

## 路径用于引用模块树中的项

来看一下 Rust 如何在模块树中找到一个项的位置，我们使用路径的方式，就像在文件系统使用路径一样。如果我们想要调用一个函数，我们需要知道它的路径。

路径有两种形式：

- **绝对路径**（*absolute path*）从 crate 根部开始，以 crate 名或者字面量 `crate` 开头。
- **相对路径**（*relative path*）从当前模块开始，以 `self`、`super` 或当前模块的标识符开头。

绝对路径和相对路径都后跟一个或多个由双冒号（`::`）分割的标识符。

让我们回到示例 7-1。我们如何调用 `add_to_waitlist` 函数？还是同样的问题，`add_to_waitlist` 函数的路径是什么？在示例 7-3 中，我们通过删除一些模块和函数，稍微简化了一下我们的代码。我们在 crate 根部定义了一个新函数 `eat_at_restaurant`，并在其中展示调用 `add_to_waitlist` 函数的两种方法。`eat_at_restaurant` 函数是我们 crate 库的一个公共 API，所以我们使用 `pub` 关键字来标记它。在[“使用 `pub` 关键字暴露路径”](https://rustwiki.org/zh-CN/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#使用-pub-关键字暴露路径)一节，我们将详细介绍 `pub`。注意，这个例子无法编译通过，我们稍后会解释原因。

```rust

mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

示例 7-3: 使用绝对路径和相对路径来调用 `add_to_waitlist` 函数

第一种方式，我们在 `eat_at_restaurant` 中调用 `add_to_waitlist` 函数，使用的是绝对路径。`add_to_waitlist` 函数与 `eat_at_restaurant` 被定义在同一 crate 中，这意味着我们可以使用 `crate` 关键字为起始的绝对路径。

在 `crate` 后面，我们持续地嵌入模块，直到我们找到 `add_to_waitlist`。你可以想象出一个相同结构的文件系统，我们通过指定路径 `/front_of_house/hosting/add_to_waitlist` 来执行 `add_to_waitlist` 程序。我们使用 `crate` 从 crate 根部开始就类似于在 shell 中使用 `/` 从文件系统根开始。

第二种方式，我们在 `eat_at_restaurant` 中调用 `add_to_waitlist`，使用的是相对路径。这个路径以 `front_of_house` 为起始，这个模块在模块树中，与 `eat_at_restaurant` 定义在同一层级。与之等价的文件系统路径就是 `front_of_house/hosting/add_to_waitlist`。以名称为起始，意味着该路径是相对路径。

选择使用相对路径还是绝对路径，还是要取决于你的项目。取决于你是更倾向于将项的定义代码与使用该项的代码分开来移动，还是一起移动。举一个例子，如果我们要将 `front_of_house` 模块和 `eat_at_restaurant` 函数一起移动到一个名为 `customer_experience` 的模块中，我们需要更新 `add_to_waitlist` 的绝对路径，但是相对路径还是可用的。然而，如果我们要将 `eat_at_restaurant` 函数单独移到一个名为 `dining` 的模块中，还是可以使用原本的绝对路径来调用 `add_to_waitlist`，但是相对路径必须要更新。我们更倾向于使用绝对路径，因为把代码定义和项调用各自独立地移动是更常见的。

> 官方依旧高速神言不说人话

我们用 C 语言的例子来类比一下 Rust 中“把代码定义和项调用各自独立地移动是更常见的”这句话。

在 C 语言中，我们没有像 Rust 这样的模块系统，但我们可以用**头文件（.h）** 和 **源文件（.c）** 来类比“定义”和“调用”以及“路径”。

### C 语言中的类比

想象你有两个 C 文件：

1. **`math_operations.h` 和 `math_operations.c`**：

   - `math_operations.h` (头文件)：声明了 `add` 函数。这就像 Rust 中一个模块的公开接口，告诉别人有这么一个函数。

     ```c
     // math_operations.h
     int add(int a, int b); // 这是函数的“定义”或“声明”
     ```

   - `math_operations.c` (源文件)：实现了 `add` 函数。这是函数的具体“定义”所在。

     ```c
     // math_operations.c
     #include "math_operations.h"
     
     int add(int a, int b) {
         return a + b;
     }
     ```

2. **`main.c`**：

   - 这个文件会**调用** `add` 函数。

#### 场景一：类似 Rust 中的**绝对路径**（使用完整路径/包含完整头文件）

在 `main.c` 中，如果你想要使用 `add` 函数，你会这么做：

```c
// main.c
#include "math_operations.h" // 告诉编译器去哪里找 add 函数的声明

int main() {
    int result = add(5, 3); // 调用 add 函数
    return 0;
}
```

这里，`#include "math_operations.h"` 就像 Rust 中的**绝对路径**。它明确地指明了 `add` 函数的声明在哪里可以找到。

- **如果我移动 `main.c` 文件（调用者）**：假设你把 `main.c` 从项目的一个子目录移动到另一个子目录，只要 `math_operations.h` 的相对位置没有改变，或者你通过编译器的 `-I` 选项（include path）告诉了编译器 `math_operations.h` 的新位置，那么 `main.c` 中 `add(5, 3)` 的调用**不需要修改**。因为 `add` 函数的定义（和声明）本身没有移动。这和 Rust 中绝对路径的稳定性类似。

#### 场景二：类似 Rust 中的**相对路径**（如果 C 语言有这种直接模块引用）

C 语言没有像 Rust 这样内建的相对模块路径引用机制。但我们可以想象一种情况，如果你需要在 `main.c` 中**直接引用** `add` 函数的定义，而不是通过头文件。这在 C 语言中是做不到的，但为了类比，我们可以假设：

如果 `main.c` 直接依赖于 `math_operations.c` 文件的**相对位置**来找到 `add` 函数的定义（这是个假设，C 实际编译不是这样做的）。那么：

- **如果我移动 `main.c` 文件（调用者）**：`main.c` 的位置变了，它与 `math_operations.c` 的相对位置就可能变了。那么，原先的“相对路径”就失效了，你**需要修改** `main.c` 中引用 `add` 函数的方式。

### “各自独立移动”的含义

回到 Rust 的语境：

“把代码定义和项调用各自独立地移动是更常见的”这句话意味着：

1. **你更常会移动和重构使用某个功能（`add` 函数）的代码文件（`main.c`），而不是改变那个功能本身（`math_operations.c` 和 `math_operations.h`）的存放位置。**
2. 当你移动 `main.c` 时，你希望它里面的 `add(5, 3)` 调用能够**保持不变**，而不需要每次移动都去修改它。

因此，使用 **绝对路径**（类似 C 语言中 `#include <library_header.h>` 或者通过 `-I` 选项找到的头文件）会更稳定。因为它从一个固定的起点（Rust 的 crate 根部）开始寻找，**不依赖于你当前调用代码（`main.c`）的位置**。即使你把 `main.c` 移动到项目的其他地方，只要 `add` 函数的定义没有从它原来的“绝对位置”移动，那么对它的引用就依然有效。

而 **相对路径**，由于它的起点是“当前模块”，一旦你移动了当前模块，相对位置就变了，路径就可能失效，需要你手动更新。

所以，Rust 建议我们倾向于使用绝对路径，因为它在代码重构时，特别是当调用方代码移动时，能够减少你需要修改的引用路径的数量，从而提高代码的维护性和稳定性。

> 简而言之就是绝对路径更好用,OK,接下来我们继续欣赏官方文档的高速神言

让我们试着编译一下示例 7-3，并查明为何不能编译！示例 7-4 展示了这个错误。

```

$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: module `hosting` is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^ private module
  |
note: the module `hosting` is defined here
 --> src/lib.rs:2:5
  |
2 |     mod hosting {
  |     ^^^^^^^^^^^

error[E0603]: module `hosting` is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^ private module
   |
note: the module `hosting` is defined here
  --> src/lib.rs:2:5
   |
2  |     mod hosting {
   |     ^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors
```

示例 7-4: 构建示例 7-3 出现的编译器错误

错误信息说 `hosting` 模块是私有的。换句话说，我们拥有 `hosting` 模块和 `add_to_waitlist` 函数的的正确路径，但是 Rust 不让我们使用，因为它不能访问私有片段。

模块不仅对于你组织代码很有用。他们还定义了 Rust 的 *私有性边界*（*privacy boundary*）：这条界线不允许外部代码了解、调用和依赖被封装的实现细节。所以，如果你希望创建一个私有函数或结构体，你可以将其放入模块。

Rust  中默认所有项（函数、方法、结构体、枚举、模块和常量）都是私有的。父模块中的项不能使用子模块中的私有项，但是子模块中的项可以使用他们父模块中的项。这是因为子模块封装并隐藏了他们的实现详情，但是子模块可以看到他们定义的上下文。继续拿餐馆作比喻，把私有性规则想象成餐馆的后台办公室：餐馆内的事务对餐厅顾客来说是不可知的，但办公室经理可以洞悉其经营的餐厅并在其中做任何事情。

Rust 选择以这种方式来实现模块系统功能，因此默认隐藏内部实现细节。这样一来，你就知道可以更改内部代码的哪些部分而不会破坏外部代码。你还可以通过使用 `pub` 关键字来创建公共项，使子模块的内部部分暴露给上级模块。

> 接下来主要就是引出PUB关键字,需要注意PUB关键字对于结构体,枚举,函数的作用是不太一样的,需要分辨一下

### 使用 `pub` 关键字暴露路径

让我们回头看一下示例 7-4 的错误，它告诉我们 `hosting` 模块是私有的。我们想让父模块中的 `eat_at_restaurant` 函数可以访问子模块中的 `add_to_waitlist` 函数，因此我们使用 `pub` 关键字来标记 `hosting` 模块，如示例 7-5 所示。

```rust
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

示例 7-5: 使用 `pub` 关键字声明 `hosting` 模块使其可在 `eat_at_restaurant` 使用

不幸的是，示例 7-5 的代码编译仍然有错误，如示例 7-6 所示。

```

$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: function `add_to_waitlist` is private
 --> src/lib.rs:9:37
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                                     ^^^^^^^^^^^^^^^ private function
  |
note: the function `add_to_waitlist` is defined here
 --> src/lib.rs:3:9
  |
3 |         fn add_to_waitlist() {}
  |         ^^^^^^^^^^^^^^^^^^^^

error[E0603]: function `add_to_waitlist` is private
  --> src/lib.rs:12:30
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                              ^^^^^^^^^^^^^^^ private function
   |
note: the function `add_to_waitlist` is defined here
  --> src/lib.rs:3:9
   |
3  |         fn add_to_waitlist() {}
   |         ^^^^^^^^^^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors
```

示例 7-6: 构建示例 7-5 出现的编译器错误

发生了什么？在 `mod hosting` 前添加了 `pub` 关键字，使其变成公有的。伴随着这种变化，如果我们可以访问 `front_of_house`，那我们也可以访问 `hosting`。但是 `hosting` 的 **内容**（*contents*） 仍然是私有的；这表明**使模块公有并不使其内容也是公有的。模块上的 `pub` 关键字只允许其父模块引用它。**

示例 7-6 中的错误说，`add_to_waitlist` 函数是私有的。私有性规则不但应用于模块，还应用于结构体、枚举、函数和方法。

让我们继续将 `pub` 关键字放置在 `add_to_waitlist` 函数的定义之前，使其变成公有。如示例 7-7 所示。

```rust

mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

示例 7-7: 为 `mod hosting` 和 `fn add_to_waitlist` 添加 `pub` 关键字使他们可以在 `eat_at_restaurant` 函数中被调用

现在代码可以编译通过了！让我们看看绝对路径和相对路径，并根据私有性规则，再检查一下为什么增加 `pub` 关键字使得我们可以在 `add_to_waitlist` 中调用这些路径。

在绝对路径，我们从 `crate`，也就是 crate 根部开始。然后 crate 根部中定义了 `front_of_house` 模块。`front_of_house` 模块不是公有的，不过因为 `eat_at_restaurant` 函数与 `front_of_house` 定义于同一模块中（即，`eat_at_restaurant` 和 `front_of_house` 是兄弟），我们可以从 `eat_at_restaurant` 中引用 `front_of_house`。接下来是使用 `pub` 标记的 `hosting` 模块。我们可以访问 `hosting` 的父模块，所以可以访问 `hosting`。最后，`add_to_waitlist` 函数被标记为 `pub` ，我们可以访问其父模块，所以这个函数调用是有效的！

在相对路径，其逻辑与绝对路径相同，除了第一步：不同于从 crate 根部开始，路径从 `front_of_house` 开始。`front_of_house` 模块与 `eat_at_restaurant` 定义于同一模块，所以从 `eat_at_restaurant` 中开始定义的该模块相对路径是有效的。接下来因为 `hosting` 和 `add_to_waitlist` 被标记为 `pub`，路径其余的部分也是有效的，因此函数调用也是有效的！

> 我们来仔细讨论一下上面这段话

**`front_of_house` 在你提供的代码中不是私有的，它是默认公开的，因为它是直接在 `crate` 根部定义的。**

Rust 的访问规则有点绕，我们来具体解释一下：

### `crate` 根部定义的模块默认是公开的

当你直接在 `src/lib.rs` (或者 `src/main.rs`) 文件中定义一个模块时，比如 `mod front_of_house {}`，这个模块就是定义在 `crate` 的**根部**。

**在 `crate` 根部定义的项（包括模块），默认情况下是可以在整个 `crate` 内部被直接访问到的。** 它们不需要 `pub` 关键字就可以被同一 `crate` 中的其他代码访问。`pub` 关键字的作用是让该项可以被**外部 `crate`** (比如你的库被其他项目引用时) 访问，或者让该项可以被**子模块之外的模块**通过完整路径访问。

所以，`front_of_house` 模块虽然没有显式地写 `pub mod front_of_house {}`，但因为它直接位于 `crate` 根部，所以：

1. **`eat_at_restaurant` 函数**（也在 `crate` 根部）可以直接看到并访问 `front_of_house`，因为它们是“兄弟”关系，都在最外层。
2. **`crate::`** 作为绝对路径的起点，它自然也能直接“看到”它根部定义的所有内容，包括 `front_of_house`。

### 为什么会产生误解？

你可能误解了“私有”的含义。在 Rust 中，一个模块（或任何项）如果没有 `pub` 关键字，它通常表示该项只在其**直接父模块内部**可见和可访问。但是，这个规则对于 **`crate` 根部**有点特殊。`crate` 根部本身就是最高层级，它里面定义的所有东西对于整个 `crate` 内部的代码来说，都是默认可访问的，除非它们被嵌套在某个私有模块内部。

### 总结

- **`front_of_house` 模块**：因为它直接定义在 `crate` 根部（`src/lib.rs` 的顶层），所以默认情况下，这个 `crate` 内部的所有代码都可以直接看到并引用它。它不是“私有的”，而是对 `crate` 内部可见的。
- **`pub mod hosting`**：这里 `pub` 的作用是让 `hosting` 模块可以被 `front_of_house` 之外的代码（比如 `eat_at_restaurant`，它位于 `front_of_house` 的父级 `crate` 根部）访问。如果没有 `pub`，那么 `hosting` 默认就是私有的，只有 `front_of_house` 内部的代码才能访问它。
- **`pub fn add_to_waitlist()`**：函数默认是私有的。如果你想从它所在的模块外面调用它，就必须给它加上 `pub`。

所以，`crate` 可以访问到 `front_of_house` 是因为 `front_of_house` 就在 `crate` 根部，并且对于 `crate` 内部是默认可见的。

> 依旧AI大法

------

是的，**默认情况下，模块外部无法直接访问 `front_of_house` 模块的内容。**

让我们用刚才的餐厅例子再来说明一下：

你的 Rust 项目是个大餐厅，`front_of_house` 是餐厅里“前厅”这个区域。

- **默认情况下（没有 `pub`）**：这个“前厅”是餐厅内部的一个区域，它里面的具体设施（比如某个柜台、某个角落）默认都是“不对外开放”的。只有当你身处“前厅”内部，或者你被“前厅”的主管（父模块）特许，才能使用里面的东西。外部的顾客（其他 `crate`）或者甚至餐厅内部其他区域的员工（其他模块）都不能直接走进“前厅”去拿里面的东西，除非前厅的某个门是打开的（`pub`）。
- **为什么 `crate::front_of_house` 可以访问到？**
  - 因为 `front_of_house` 是直接定义在你的 `crate`（也就是整个餐厅）的“大厅”（根部）里的。
  - 所以，当你从“大厅”出发（`crate::`）时，你自然能看到“大厅”里所有的区域划分，包括 `front_of_house`。这就像你站在餐厅大堂中央，能看到哪里是“前厅”，哪里是“厨房”。
  - 但是，能看到这个区域，不代表你就能直接使用这个区域里的所有东西。要使用里面的东西，里面的东西本身也必须是公开的。

所以，结论是：

如果你想让 **`front_of_house` 模块内部的** 某个函数或结构体被**`front_of_house` 模块外部**的代码访问（比如被 `eat_at_restaurant` 访问，或者被另一个 `crate` 访问），那么：

1. **`front_of_house` 模块**本身需要是可访问的（因为它直接在 `crate` 根部，所以它对 `crate` 内部是默认可见的）。
2. **`front_of_house` 内部的子模块或函数/结构体**也必须使用 **`pub` 关键字** 来声明，才能让外部通过完整路径访问到它们。

这就是为什么在示例中，你不仅要给 `hosting` 加上 `pub`，还要给 `add_to_waitlist` 加上 `pub`，才能让 `eat_at_restaurant` 最终调用到它。

### [使用 `super` 起始的相对路径](https://rustwiki.org/zh-CN/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#使用-super-起始的相对路径)

我们还可以使用 `super` 开头来构建从父模块开始的相对路径。这么做类似于文件系统中以 `..` 开头的语法。我们为什么要这样做呢？

考虑一下示例 7-8 中的代码，它模拟了厨师更正了一个错误订单，并亲自将其提供给客户的情况。`fix_incorrect_order` 函数通过指定的 `super` 起始的 `serve_order` 路径，来调用 `serve_order` 函数：

```rust

fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
```

示例 7-8: 使用以 `super` 开头的相对路径从父目录开始调用函数

`fix_incorrect_order` 函数在 `back_of_house` 模块中，所以我们可以使用 `super` 进入 `back_of_house` 父模块，也就是本例中的 `crate` 根。在这里，我们可以找到 `serve_order`。成功！我们认为 `back_of_house` 模块和 `serve_order` 函数之间可能具有某种关联关系，并且，如果我们要重新组织这个 crate 的模块树，需要一起移动它们。因此，我们使用 `super`，这样一来，如果这些代码被移动到了其他模块，我们只需要更新很少的代码。

### [创建公有的结构体和枚举](https://rustwiki.org/zh-CN/book/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#创建公有的结构体和枚举)

我们还可以使用 `pub` 来设计公有的结构体和枚举，不过有一些额外的细节需要注意。如果我们在一个结构体定义的前面使用了 `pub` ，这个结构体会变成公有的，但是这个结构体的字段仍然是私有的。我们可以根据情况决定每个字段是否公有。在示例 7-9 中，我们定义了一个公有结构体 `back_of_house::Breakfast`，其中有一个公有字段 `toast` 和私有字段 `seasonal_fruit`。这个例子模拟的情况是，在一家餐馆中，顾客可以选择随餐附赠的面包类型，但是厨师会根据季节和库存情况来决定随餐搭配的水果。餐馆可用的水果变化是很快的，所以顾客不能选择水果，甚至无法看到他们将会得到什么水果。

```rust

mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // 在夏天点一份黑麦面包作为早餐
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // 更改我们想要的面包
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // 如果取消下一行的注释，将会导致编译失败；我们不被允许
    // 看到或更改随餐搭配的季节水果
    // meal.seasonal_fruit = String::from("blueberries");
}
```

示例 7-9: 带有公有和私有字段的结构体

因为 `back_of_house::Breakfast` 结构体的 `toast` 字段是公有的，所以我们可以在 `eat_at_restaurant` 中使用点号来随意的读写 `toast` 字段。注意，我们不能在 `eat_at_restaurant` 中使用 `seasonal_fruit` 字段，因为 `seasonal_fruit` 是私有的。尝试去除那一行修改 `seasonal_fruit` 字段值的代码的注释，看看你会得到什么错误！

还请注意一点，因为 `back_of_house::Breakfast` 具有私有字段，所以这个结构体需要提供一个公共的关联函数来构造 `Breakfast` 的实例（这里我们命名为 `summer`）。如果 `Breakfast` 没有这样的函数，我们将无法在 `eat_at_restaurant` 中创建 `Breakfast` 实例，因为我们不能在 `eat_at_restaurant` 中设置私有字段 `seasonal_fruit` 的值。

与之相反，如果我们将枚举设为公有，则它的所有成员都将变为公有。我们只需要在 `enum` 关键字前面加上 `pub`，就像示例 7-10 展示的那样。

```rust

mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

示例 7-10: 设计公有枚举，使其所有成员公有

因为我们创建了名为 `Appetizer` 的公有枚举，所以我们可以在 `eat_at_restaurant` 中使用 `Soup` 和 `Salad` 成员。如果枚举成员不是公有的，那么枚举会显得用处不大；给枚举的所有成员挨个添加 `pub` 是很令人恼火的，因此枚举成员默认就是公有的。结构体通常使用时，不必将它们的字段公有化，因此结构体遵循常规，内容全部是私有的，除非使用 `pub` 关键字。

还有一种使用 `pub` 的场景我们还没有涉及到，那就是我们最后要讲的模块功能：`use` 关键字。我们将先单独介绍 `use`，然后展示如何结合使用 `pub` 和 `use`。
