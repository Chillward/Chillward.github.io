---
title: RUST学习日记之错误处理
date: 2025-07-25 18:15:52
tags: RUST
categories: Rust
---

> 以前一直是写C的，对于现代语言的错误处理一直是搞不太懂，不过目前看来Rust的错误处理看起来还是比Java顺眼一点的

## panic与不可恢复的错误

panic就类似于程序崩溃，程序会直接崩溃掉，这种错误是无法被程序处理的

有的时候代码出问题了，而你对此束手无策。对于这种情况，Rust 有 `panic!`宏。当执行这个宏时，程序会打印出一个错误信息，展开并清理栈数据，然后接着退出。出现这种情况的场景通常是检测到一些类型的 bug，而且开发者并不清楚该如何处理它。

> 当出现 panic 时，程序默认会开始 **展开**（*unwinding*），这意味着 Rust 会回溯栈并清理它遇到的每一个函数的数据，不过这个回溯并清理的过程有很多工作。另一种选择是直接 **终止**（*abort*），这会不清理数据就退出程序。那么程序所使用的内存需要由操作系统来清理。如果你需要项目的最终二进制文件越小越好，panic 时通过在 *Cargo.toml* 的 `[profile]` 部分增加 `panic = 'abort'`，可以由展开切换为终止。例如，如果你想要在release模式中 panic 时直接终止：
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

让我们在一个简单的程序中调用 `panic!`：

```rust
fn main() {
    panic!("crash and burn");
}
```

运行程序将会出现类似这样的输出：

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

最后两行包含 `panic!` 调用造成的错误信息。第一行显示了 panic 提供的信息并指明了源码中 panic 出现的位置：*src/main.rs:2:5* 表明这是 *src/main.rs* 文件的第二行第五个字符。

在这个例子中，被指明的那一行是我们代码的一部分，而且查看这一行的话就会发现 `panic!` 宏的调用。在其他情况下，`panic!` 可能会出现在我们的代码所调用的代码中。错误信息报告的文件名和行号可能指向别人代码中的 `panic!` 宏调用，而不是我们代码中最终导致 `panic!` 的那一行。我们可以使用 `panic!` 被调用的函数的 backtrace 来寻找代码中出问题的地方。下面我们会详细介绍 backtrace 是什么。

> backtrace宏的作用就是让我们可以找到“我们编写的代码中导致程序崩溃的地方”

~~~ rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
~~~

这是一个越界调用，在C语言中，这段代码会运行，然后给你一个错误的值，这是相当危险的

为了使程序远离这类漏洞，如果尝试读取一个索引不存在的元素，Rust 会停止执行并拒绝继续。尝试运行上面的程序会出现如下：

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', libcore/slice/mod.rs:2448:10
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

这指向了一个不是我们编写的文件，*libcore/slice/mod.rs*。其为 Rust 源码中 `slice` 的实现。这是当对 vector `v` 使用 `[]` 时 *libcore/slice/mod.rs* 中会执行的代码，也是真正出现 `panic!` 的地方。

接下来的几行提醒我们可以设置 `RUST_BACKTRACE` 环境变量来得到一个 backtrace。*backtrace* 是一个执行到目前位置所有被调用的函数的列表。Rust 的 backtrace 跟其他语言中的一样：阅读 backtrace 的关键是从头开始读直到发现你编写的文件。这就是问题的发源地。这一行往上是你的代码所调用的代码；往下则是调用你的代码的代码。这些行可能包含核心 Rust 代码，标准库代码或用到的 crate 代码。让我们将 `RUST_BACKTRACE` 环境变量设置为任何不是 0 的值来获取 backtrace 看看。示例 9-2 展示了与你看到类似的输出：

```text
$ RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', libcore/slice/mod.rs:2448:10
stack backtrace:
   0: std::sys::unix::backtrace::tracing::imp::unwind_backtrace
             at libstd/sys/unix/backtrace/tracing/gcc_s.rs:49
   1: std::sys_common::backtrace::print
             at libstd/sys_common/backtrace.rs:71
             at libstd/sys_common/backtrace.rs:59
   2: std::panicking::default_hook::{{closure}}
             at libstd/panicking.rs:211
   3: std::panicking::default_hook
             at libstd/panicking.rs:227
   4: <std::panicking::begin_panic::PanicPayload<A> as core::panic::BoxMeUp>::get
             at libstd/panicking.rs:476
   5: std::panicking::continue_panic_fmt
             at libstd/panicking.rs:390
   6: std::panicking::try::do_call
             at libstd/panicking.rs:325
   7: core::ptr::drop_in_place
             at libcore/panicking.rs:77
   8: core::ptr::drop_in_place
             at libcore/panicking.rs:59
   9: <usize as core::slice::SliceIndex<[T]>>::index
             at libcore/slice/mod.rs:2448
  10: core::slice::<impl core::ops::index::Index<I> for [T]>::index
             at libcore/slice/mod.rs:2316
  11: <alloc::vec::Vec<T> as core::ops::index::Index<I>>::index
             at liballoc/vec.rs:1653
  12: panic::main
             at src/main.rs:4
  13: std::rt::lang_start::{{closure}}
             at libstd/rt.rs:74
  14: std::panicking::try::do_call
             at libstd/rt.rs:59
             at libstd/panicking.rs:310
  15: macho_symbol_search
             at libpanic_unwind/lib.rs:102
  16: std::alloc::default_alloc_error_hook
             at libstd/panicking.rs:289
             at libstd/panic.rs:392
             at libstd/rt.rs:58
  17: std::rt::lang_start
             at libstd/rt.rs:74
  18: panic::main
```

这种信息一般是从下往上读，直到找到我们自己编写的代码位置

> RUST_BACKTRACE=1 cargo run

我们可以通过终端传递这个环境变量的值来进入BACKTRACE。

***

现在让我们回到C语言

~~~ C
#include <stdio.h>  // 用于 printf
#include <limits.h> // 用于 INT_MIN

/**
 * @brief 执行两个整数的除法。
 *
 * @param numerator 被除数。
 * @param denominator 除数。
 * @return int 如果除数不为零，返回计算出的商。
 * 如果除数为零，返回 INT_MIN 表示错误。
 */
int safe_divide(int numerator, int denominator) {
    if (denominator == 0) {
        fprintf(stderr, "Error: Division by zero is not allowed.\n");
        return INT_MIN; // 传播错误：返回一个特殊值表示失败
    }
    return numerator / denominator; // 成功：返回计算结果
}

int main() {
    int result;

    printf("--- 正常除法示例 ---\n");
    result = safe_divide(10, 2);
    if (result != INT_MIN) { // 检查返回值是否是错误码
        printf("10 / 2 = %d\n", result); // 成功处理
    } else {
        printf("Error occurred during division.\n"); // 错误处理
    }

    printf("\n--- 除数为零示例 ---\n");
    result = safe_divide(10, 0);
    if (result != INT_MIN) { // 检查返回值是否是错误码
        printf("10 / 0 = %d\n", result); // 理论上不会执行到这里
    } else {
        printf("Error occurred during division. Cannot divide by zero.\n"); // 错误处理
    }

    printf("\n--- 另一个正常除法示例 ---\n");
    result = safe_divide(-15, 3);
    if (result != INT_MIN) {
        printf("-15 / 3 = %d\n", result);
    } else {
        printf("Error occurred during division.\n");
    }

    return 0;
}
~~~

当我们写一个除法程序，当我们传"0"为除数，程序很明显不应该崩溃，而是该返回一个标记值来提醒调用者出错了，然后调用者再进行处理，这就是**可恢复的错误**以及**错误的传播**。

C语言中错误的传播相对原始，一般都是通过返回值层层传递，一旦有一层忘记处理这种情况，整个程序的稳定性就会受到极大的影响。 

在讨论Rust中错误的传播前，我们先来讨论Rust对于这种可恢复错误的处理方式。

------

在 Rust 中，`Option<T>` 和 `Result<T, E>` 是两个非常核心的枚举（enum），它们是 Rust 强大的**错误处理**和**存在性（presence）管理**机制的基石。它们的设计理念是强制你在编译时处理可能缺失的值或可能发生的错误，从而避免了其他语言中常见的空指针异常和未处理的运行时错误。

### `Option<T>`：处理值可能缺失的情况

`Option<T>` 枚举用来表示一个值**可能存在，也可能不存在**的情况。它的定义如下：

```Rust
enum Option<T> {
    None, // 值不存在
    Some(T), // 值存在，并包含类型 T 的数据
}
```

**它解决什么问题？**

在许多其他语言（如 C++、Java、Python 等）中，你可能会使用 `NULL`、`null` 或 `None` 来表示一个变量没有值。然而，直接使用这些“空”值往往会导致运行时错误，比如著名的**空指针异常（Null Pointer Exception）**。因为你可能会忘记检查一个值是否为 `null`，然后尝试对其进行操作。

`Option<T>` 强制你在编译时就处理值存在或不存在的两种情况。如果你尝试直接使用一个 `Option<T>` 中的值而不先确定它是否是 `Some(T)`，编译器会报错。

### `Result<T, E>`：处理可能发生的错误

`Result<T, E>` 枚举用来表示一个操作**可能成功并返回一个值，也可能失败并返回一个错误**。它的定义如下：

```Rust
enum Result<T, E> {
    Ok(T),  // 操作成功，并包含类型 T 的结果数据
    Err(E), // 操作失败，并包含类型 E 的错误数据
}
```

**它解决什么问题？**

在 C 语言中，你通常通过返回值和错误码来表示函数成功或失败。在 Java/Python 等语言中，则通常使用异常（exceptions）。然而，这些方式都有其弊端：

- **C 语言的错误码**：容易被忽略，需要手动检查，且错误信息有限。
- **异常**：虽然方便，但可能会导致控制流难以预测（“goto 式的错误处理”），且编译器通常不会强制你捕获或声明异常，可能导致未处理的运行时崩溃。

`Result<T, E>` 强制你在编译时就考虑并处理成功和失败的两种情况，使得错误处理成为你代码类型系统的一部分。

**如何使用？**

与 `Option` 类似，你通常会使用 `match` 表达式、`if let` 或 `Result` 提供的各种方法（如 `is_ok()`, `is_err()`, `unwrap()`, `expect()`, `map_err()`, `and_then()`, **`?` 运算符**等）来处理 `Result` 值。

**示例：**

```Rust
use std::fs::File; // 引入文件系统模块

fn main() {
    // 尝试打开一个不存在的文件
    // File::open 返回一个 Result<File, std::io::Error>
    let greeting_file = File::open("hello.txt");
    let greeting_file = match greeting_file {
      Ok(file) => file,
      Err(error) => {
         panic!("Probled opening the file {:?}",error) 
        }, 
    };

    // 如果上面一行没有 panic，说明文件成功打开了
    println!("文件 'hello.txt' 已成功打开！");

    // 注意：如果 hello.txt 不存在，上面的 println! 永远不会执行
}
```

这是Rust中最简单的错误处理，通过match匹配Result<>成员来实现对于错误的处理，但是在上面那段程序中，当打开文件失败时，程序直接崩溃掉了，这很明显不是我们想要的结果，接下来我们引入下一个知识点 **错误的匹配**

~~~ rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
~~~

`File::open` 返回的 `Err` 成员中的值类型 `io::Error`，它是一个标准库中提供的结构体。这个结构体有一个返回 `io::ErrorKind` 值的 `kind` 方法可供调用。`io::ErrorKind` 是一个标准库提供的枚举，它的成员对应 `io` 操作可能导致的不同错误类型。我们感兴趣的成员是 `ErrorKind::NotFound`，它代表尝试打开的文件并不存在。这样，`match` 就匹配完 `f` 了，不过对于 `error.kind()` 还有一个内层 `match`。

我们希望在内层 `match` 中检查的条件是 `error.kind()` 的返回值是否为 `ErrorKind`的 `NotFound` 成员。如果是，则尝试通过 `File::create` 创建文件。然而因为 `File::create` 也可能会失败，还需要增加一个内层 `match` 语句。当文件不能被打开，会打印出一个不同的错误信息。外层 `match` 的最后一个分支保持不变，这样对任何除了文件不存在的错误会使程序 panic。

match确实是强大的，但是有时候我们确实是希望当出现错误时直接panic!掉，此时再写match会有点麻烦，Rust为我们提供了两个简写方法。

### unwrap` 和 `expect

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

如果调用这段代码时不存在 *hello.txt* 文件，我们将会看到一个 `unwrap` 调用 `panic!` 时提供的错误信息：

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
src/libcore/result.rs:906:4
```

还有另一个类似于 `unwrap` 的方法它还允许我们选择 `panic!` 的错误信息：`expect`。使用 `expect` 而不是 `unwrap` 并提供一个好的错误信息可以表明你的意图并更易于追踪 panic 的根源。`expect` 的语法看起来像这样：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

`expect` 与 `unwrap` 的使用方式一样：返回文件句柄或调用 `panic!` 宏。`expect` 在调用 `panic!` 时使用的错误信息将是我们传递给 `expect` 的参数，而不像 `unwrap` 那样使用默认的 `panic!` 信息。它看起来像这样：

```text
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }', src/libcore/result.rs:906:4
```

因为这个错误信息以我们指定的文本开始，`Failed to open hello.txt`，将会更容易找到代码中的错误信息来自何处。如果在多处使用 `unwrap`，则需要花更多的时间来分析到底是哪一个 `unwrap` 造成了 panic，因为所有的 `unwrap` 调用都打印相同的信息。

接下来我们就可以进入到下一个知识点 **错误的传播**，C语言一般都是通过返回值的层层传递来实现错误的传播。传播错误的好处就是这样能更好地控制代码调用，因为比起你代码所拥有的上下文，调用者可能拥有更多信息或逻辑来决定应该如何处理错误。

简单点说错误的传播就是把可能发生的错误返回给调用者，让调用者来处理而不是由被调用的函数来处理

~~~ rust
#![allow(unused)]
fn main() {
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
}
~~~

首先让我们看看函数的返回值：`Result<String, io::Error>`。这意味着函数返回一个 `Result<T, E>` 类型的值，其中泛型参数 `T` 的具体类型是 `String`，而 `E` 的具体类型是 `io::Error`。如果这个函数没有出任何错误成功返回，函数的调用者会收到一个包含 `String` 的 `Ok` 值 —— 函数从文件中读取到的用户名。如果函数遇到任何错误，函数的调用者会收到一个 `Err` 值，它储存了一个包含更多这个问题相关信息的 `io::Error` 实例。这里选择 `io::Error` 作为函数的返回值是因为它正好是函数体中那两个可能会失败的操作的错误返回值：`File::open` 函数和 `read_to_string` 方法。

函数体以 `File::open` 函数开头。接着使用 `match` 处理返回值 `Result`，类似于示例 9-4 中的 `match`，唯一的区别是当 `Err` 时不再调用 `panic!`，而是提早返回并将 `File::open` 返回的错误值作为函数的错误返回值传递给调用者。如果 `File::open` 成功了，我们将文件句柄储存在变量 `f` 中并继续。

接着我们在变量 `s` 中创建了一个新 `String` 并调用文件句柄 `f` 的 `read_to_string` 方法来将文件的内容读取到 `s` 中。`read_to_string` 方法也返回一个 `Result` 因为它也可能会失败：哪怕是 `File::open` 已经成功了。所以我们需要另一个 `match` 来处理这个 `Result`：如果 `read_to_string` 成功了，那么这个函数就成功了，并返回文件中的用户名，它现在位于被封装进 `Ok` 的 `s` 中。如果 `read_to_string` 失败了，则像之前处理 `File::open` 的返回值的 `match` 那样返回错误值。不过并不需要显式的调用 `return`，因为这是函数的最后一个表达式。

调用这个函数的代码最终会得到一个包含用户名的 `Ok` 值，或者一个包含 `io::Error` 的 `Err` 值。我们无从得知调用者会如何处理这些值。例如，如果他们得到了一个 `Err` 值，他们可能会选择 `panic!` 并使程序崩溃、使用一个默认的用户名或者从文件之外的地方寻找用户名。我们没有足够的信息知晓调用者具体会如何尝试，所以将所有的成功或失败信息向上传播，让他们选择合适的处理方法。

> 这种写法是相当常见的，Rust也为我们提供了这种情况下可供使用的简写

### 传播错误的简写：`?` 运算符

示例 9-7 展示了一个 `read_username_from_file` 的实现，它实现了与示例 9-6 中的代码相同的功能，不过这个实现使用了 `?` 运算符：

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

示例 9-7：一个使用 `?` 运算符向调用者返回错误的函数

`Result` 值之后的 `?` 被定义为与示例 9-6 中定义的处理 `Result` 值的 `match` 表达式有着完全相同的工作方式。如果 `Result` 的值是 `Ok`，这个表达式将会返回 `Ok` 中的值而程序将继续执行。如果值是 `Err`，`Err` 将作为整个函数的返回值，就好像使用了 `return` 关键字一样，这样错误值就被传播给了调用者。

示例 9-6 中的 `match` 表达式与问号运算符所做的有一点不同：`?` 运算符所使用的错误值被传递给了 `from` 函数，它定义于标准库的 `From` trait 中，其用来将错误从一种类型转换为另一种类型。当 `?` 运算符调用 `from` 函数时，收到的错误类型被转换为由当前函数返回类型所指定的错误类型。这在当函数返回单个错误类型来代表所有可能失败的方式时很有用，即使其可能会因很多种原因失败。只要每一个错误类型都实现了 `from` 函数来定义如何将自身转换为返回的错误类型，`?` 运算符会自动处理这些转换。

在示例 9-7 的上下文中，`File::open` 调用结尾的 `?` 将会把 `Ok` 中的值返回给变量 `f`。如果出现了错误，`?` 运算符会提早返回整个函数并将一些 `Err` 值传播给调用者。同理也适用于 `read_to_string` 调用结尾的 `?`。

当你在一个不返回 `Result` 的函数中需要调用返回 `Result` 的函数时，文本提供了两种主要的修复方法：

1. **修改当前函数的返回值类型为 `Result<T, E>`**： 这是最常见和推荐的方法，特别是当你的函数确实需要传播错误时。你将函数的签名从默认的 `()` 修改为 `Result<T, E>`，使得它能够兼容 `?` 运算符传播的错误。

   **示例：**

   Rust

   ```
   use std::error::Error; // 引入 Error trait
   use std::fs::File;
   
   // 将 main 函数的返回值类型修改为 Result<(), Box<dyn Error>>
   fn main() -> Result<(), Box<dyn Error>> {
       let f = File::open("hello.txt")?; // 现在 ? 运算符可以正常工作了
   
       // ... 其他操作 ...
   
       Ok(()) // 如果所有操作成功，返回 Ok(())
   }
   ```

   - **`-> Result<(), Box<dyn Error>>`**：这里 `main` 函数被声明为返回一个 `Result`。
     - `Ok(())` 表示成功，没有具体返回值。
     - `Err(Box<dyn Error>)` 表示失败，并包含一个**错误对象**。
   - **`Box<dyn Error>`**：这被称为 **“trait 对象”**。它的作用是允许你返回**任何实现了 `std::error::Error` 这个 trait 的错误类型**。这是 Rust 处理“多种可能错误类型”的一种通用方法。在这里，你可以简单地理解为 `main` 函数现在可以返回任何类型的错误，只要这个错误实现了 `Error` trait。

2. **在当前函数内使用 `match` 或 `Result` 的其他方法处理错误**： 如果你不希望函数传播错误，或者函数不能修改返回值类型（例如，一些回调函数），那么你就必须在当前函数内部显式地处理 `Result`，而不是使用 `?` 运算符。

   ```Rust
   use std::fs::File;
   
   fn main() { // main 函数的返回值仍然是 ()
       // 使用 match 显式处理 File::open 返回的 Result
       let f = match File::open("hello.txt") {
           Ok(file) => file,
           Err(e) => {
               eprintln!("Error opening file: {}", e); // 打印错误到标准错误输出
               return; // 如果发生错误，直接从 main 函数返回，程序终止
           }
       };
   
       println!("File opened successfully!");
       // ... 继续使用 f ...
   }
   ```

   在这个例子中，我们使用 `match` 语句来检查 `File::open` 的结果。如果它返回 `Err`，我们就打印错误信息并使用 `return;` 提前退出 `main` 函数。这样就没有错误需要被“传播”出 `main` 函数了。