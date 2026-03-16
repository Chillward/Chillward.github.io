---
title: RUST学习日记之控制流
date: 2025-07-20 09:21:17
tags: RUST
categories: Rust
---

### if表达式

rust中的if语句跟C语言中的是类似的，基本格式如下

~~~rust
fn main() {
    let number = 3;
    if (number < 5) {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
~~~

但也有不同的地方：

1. if后紧跟的条件必须是一个bool值，如果不是的话，程序会无法编译
2. 在rust中，if是一个表达式而不是语句，二者之间的区别就是语句仅仅只是执行一个动作而不返回值，表达式会在执行后返回一个值。

~~~rust
fn main() {
    let x = 10;

    let y = if x > 5 {
        20 // 这个块的值是 20
    } else {
        30 // 这个块的值是 30
    }; // 注意这里的分号，表示表达式的结束

    println!("y = {}", y); // 输出 y = 20

    // 另一个例子：隐式返回
    let description = if x > 100 {
        "非常大"
    } else if x > 50 {
        "比较大"
    } else {
        "不大"
    };
    println!("x 的值: {}", description); // 输出 x 的值: 不大
}
~~~

**关键点：**

1. **返回类型一致性**：当 `if` 作为表达式使用时，所有分支（`if`、`else if` 和 `else`）必须返回**相同类型**的值。

   ```rust
   let result = if true {
       10 // 返回 i32
   } else {
       "hello" // 错误！返回 &str，类型不匹配
   };
   ```

   编译器会报错，因为 `10` 是整数，而 `"hello"` 是字符串切片，它们的类型不同。

2. **块的最后一个表达式是返回值**：在 Rust 中，一个代码块的值是其最后一个表达式的值（没有分号），没法使用return ***;的形式返回。

   ```rust
   let value = if condition {
       let temp = 10;
       temp + 5 // 这个表达式的值 15 就是整个 if 分支的返回值
   } else {
       20
   };
   ```

***

### 循环

rust中的循环分3种:

1. loop
2. while
3. for

需要注意的是，rust中是没有类似于C语言中do while()这种循环结构的，等效的功能可以由loop实现。

#### loop:

loop循环类似于C语言中while(1)或者for(;;)，不手动用Break退出的话就会一直循环下去。

~~~rust
fn main() {
    loop {
        println!("again!");
    }
}
~~~

需要注意的是,rust中有一个C语言中没有的特性:**循环标签**

##### 循环标签(loop labels)：

语法:

~~~rust
'label_name: loop {
    // 循环体
}

'label_name: while condition {
    // 循环体
}

'label_name: for item in collection {
    // 循环体
}
~~~

循环标签的语法是在 `loop`、`while` 或 `for` 关键字之前加上一个**单引号开头的名称**，后面紧跟着一个**冒号**，然后在使用 `break` 或 `continue` 时指定这个标签，从而控制跳出或继续执行哪个具体的循环，即使它不是最内层循环。

~~~rust
fn main() {
    let mut count = 0;

    'outer_loop: loop { // 这是一个名为 'outer_loop 的循环
        println!("Outer loop count: {count}");
        let mut inner_count = 0;

        'inner_loop: loop { // 这是一个名为 'inner_loop 的循环
            println!("Inner loop count: {inner_count}");

            if inner_count >= 2 {
                break 'inner_loop; // 跳出 'inner_loop
            }
            if count >= 1 {
                break 'outer_loop; // 跳出 'outer_loop
            }
            inner_count += 1;
        }
        count += 1;
        if count >= 3 {
            break; // 跳出当前（外层）循环，这里其实效果和 break 'outer_loop 一样，因为这是最外层循环
        }
    }
    println!("Loop finished. Final count: {count}");
}
~~~

rust的这种特性很好的避免了C语言中需要跳出复杂循环时需要设置标志位以及单独判断的情况。

~~~c
int done = 0;
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        if (i == 1 && j == 1) {
            done = 1; // 设置标志
            break;    // 跳出内层循环
        }
        printf("i: %d, j: %d\n", i, j);
    }
    if (done) {
        break; // 根据标志跳出外层循环
    }
}
~~~

***

#### while:

~~~rust
fn main(){
    while(true){
        println!("Hello World")
    }
}
~~~

while循环的语法跟特性与C语言都是类似的，唯一的不同就是它的条件必须是一个bool量而不能是其他类型。

***

#### for:

Rust的for循环与C语言的for循环有较大不同，Rust中的for循环类似于其他现代语言中的"for each"。

## 核心差异总结



| 特性       |                 C 语言的 `for` 循环                  |                   Rust 的 `for` 循环                   |
| ---------- | :--------------------------------------------------: | :----------------------------------------------------: |
| **基础**   |        基于**计数器和条件**，手动控制迭代过程        |          基于**迭代器**，遍历集合中的每个元素          |
| **语法**   |            `for (init; condition; step)`             |               `for element in iterator`                |
| **安全性** |      容易出现索引越界和“差一错误”，需要手动管理      |     Rust 的所有权和借用系统保证安全，避免越界访问      |
| **简洁性** |  对于简单计数循环简洁，但遍历集合时需要额外索引管理  |   遍历集合和范围时非常简洁，不需要手动索引或长度计算   |
| **灵活性** | 可以实现任何循环逻辑，包括非标准迭代（但可能不清晰） | 通过迭代器适配器（`map`, `filter` 等）实现复杂迭代逻辑 |
| **返回值** |                   循环本身不返回值                   |     循环本身不返回值，但可以结合 `break` 来返回值      |

语法如下:

~~~ rust
for 元素 in 迭代器 {
    // 循环体
}
~~~

下面是简单的示例:

~~~rust
fn main() {
    // Rust 示例：遍历向量（Vector）
    let numbers = vec![10, 20, 30, 40, 50];

    // .iter() 方法返回一个迭代器，它产出对元素的不可变引用
    for number in numbers.iter() {
        print!("{} ", number);
    }
    println!(); // 输出: 10 20 30 40 50

    // Rust 示例：使用范围（Range）
    for i in 0..5 { // 范围 [0, 5) 不包含 5
        print!("{} ", i);
    }
    println!(); // 输出: 0 1 2 3 4

    // 如果需要索引，可以使用 .enumerate()
    for (index, number) in numbers.iter().enumerate() {
        println!("Index: {}, Value: {}", index, number);
    }
}
~~~



**注意**:Rust是可以通过break;在循环(for while loop)中返回值的，例子如下：

~~~rust
fn main() {
    let mut counter = 0;

    // `loop` 作为一个表达式，通过 `break` 返回值
    let result = loop { // 整个 loop 表达式将被赋值给 result
        counter += 1;

        if counter == 10 {
            break counter * 2; // 当 counter 等于 10 时，跳出循环，并将 counter * 2 作为 loop 表达式的值
        }
    }; // 注意这里的分号，表示表达式的结束

    println!("The result is: {}", result); // 输出: The result is: 20
    println!("The final counter value is: {}", counter); // 输出: The final counter value is: 10
}
~~~

