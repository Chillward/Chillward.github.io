---
title: C语言实现lambdas
date: 2025-08-03 09:06:16
tags: lambdas C
categories: C语言
---

在C语言中，原生并没有像C++或JavaScript那样内置的**箭头函数（lambda）**支持。但是，我们可以通过一些技巧和模式来模拟实现类似的功能。

下面介绍两种常见的实现方式：

### 1. 使用宏和`typeof`关键字

这种方法是利用C语言的宏预处理器和GCC/Clang编译器特有的`typeof`扩展。这种方式的优点是代码简洁，看起来最接近真正的lambda表达式，但缺点是它依赖于特定的编译器，可移植性较差。

```C
#include <stdio.h>

// 定义一个宏，用于创建类似lambda的函数
// 参数：
// type: 函数的返回类型
// args: 函数的参数列表，用括号括起来
// body: 函数体
#define LAMBDA(type, args, body) \
({ \
    type __fn__ args body \
    __fn__; \
})

int main() {
    // 定义一个lambda，求两个数的和
    int (*sum_lambda)(int, int) = LAMBDA(int, (int a, int b), {
        return a + b;
    });

    printf("Sum: %d\n", sum_lambda(5, 3)); // 输出 Sum: 8

    // 定义一个lambda，判断一个数是否为偶数
    int (*is_even_lambda)(int) = LAMBDA(int, (int num), {
        return num % 2 == 0;
    });

    printf("Is 10 even? %d\n", is_even_lambda(10)); // 输出 Is 10 even? 1
    printf("Is 7 even? %d\n", is_even_lambda(7));   // 输出 Is 7 even? 0

    return 0;
}
```

**工作原理：**

- `LAMBDA`宏创建了一个匿名函数，并立即返回这个函数的地址。
- `({})` 这种语法是GCC的**语句表达式（Statement Expressions）**扩展，它允许在一个表达式中使用代码块。代码块的最后一条语句的值就是整个表达式的值。
- `typeof`在这里没有直接使用，但这个模式的核心是利用了GCC/Clang对局部函数（nested functions）的支持，并在宏中返回了其地址。上述代码中的`typeof`是隐藏在宏的实现细节中，用来自动推断函数指针类型的。

### 2. 使用结构体和函数指针

这是一种更通用、更具可移植性的方法，可以在任何遵循C标准的编译器上使用。它的核心思想是将**函数指针**和**捕获的变量**（被lambda引用的外部变量）封装在一个**结构体**中。

C

```C
#include <stdio.h>

// 定义一个结构体，用于表示lambda
typedef struct {
    void *context; // 用于存储捕获的变量
    int (*function)(void *context, int arg); // 函数指针
} Lambda;

// 实际执行的函数，它接收一个上下文和一个参数
int adder_function(void *context, int arg) {
    int *base = (int *)context;
    return (*base) + arg;
}

// 创建一个lambda
Lambda create_adder(int base) {
    // 分配内存来存储捕获的变量
    int *context = (int *)malloc(sizeof(int));
    *context = base;

    Lambda l = {
        .context = context,
        .function = adder_function
    };
    return l;
}

int main() {
    // 创建一个lambda，它会捕获外部变量 'x'
    int x = 10;
    Lambda add_x = create_adder(x);

    // 调用lambda
    int result = add_x.function(add_x.context, 5); // 10 + 5
    printf("Result: %d\n", result); // 输出 Result: 15

    // 释放捕获变量的内存
    free(add_x.context);

    return 0;
}
```

**工作原理：**

- 我们定义了一个`Lambda`结构体，它包含两个成员：
  - `context`：一个**`void*`指针**，用于存储lambda需要访问的外部变量（即“捕获的变量”）。
  - `function`：一个**函数指针**，它接收`context`指针和实际的参数。
- 当我们创建一个`lambda`时，我们会分配内存来存储外部变量，并将其地址赋给`context`。
- 在调用时，我们将`Lambda`结构体的`context`和实际参数一起传递给`function`指针指向的函数。
- 这种方式更复杂，但**非常灵活**，并且是C语言中实现闭包（closure）的经典模式。它也是很多库（例如：libdispatch）在C语言中实现异步回调和闭包的基础。
