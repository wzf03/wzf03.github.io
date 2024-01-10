---
title: 《Rust by Example》读书笔记
date: 2022-07-28 10:00:04 +0800
categories: [Rust]
tags: [rust, programming]
image: assets/img/covers/rust.png
---

## Hello World

```rust
// 一个标准的Hello World程序
fn main() {
    // 这是一个宏，可以把文本输出到控制台
    println!("Hello World!");
}
```

### 注释

- **普通注释**，其内容将被编译器忽略掉：
  - `// 单行注释，注释内容直到行尾。`
  - `/* 块注释，注释内容一直到结束分隔符。 */`
- **文档注释**，其内容将被解析成 HTML 帮助[文档](https://rustwiki.org/zh-CN/rust-by-example/meta/doc.html)：
  - `/// 为接下来的项生成帮助文档。`
  - `//! 为注释所属于的项（译注：如 crate、模块或函数）生成帮助文档。`

```rust
fn main() {
    // 行注释，一般而言推荐行注释

    /*
     * 块注释，在临时注释大量代码块时特别有用
     * /* 块注释可以嵌套 */
     */

    /*
    另一种风格的块注释
    */

    // 这种块注释也是成立的，行注释则无法达到这种效果
    let x = 5 + /* 90 + */ 5;
}
```

### 格式化输出

`std::fmt`里面定义的一系列宏处理格式化输出

- `format!`：将格式化文本写到[字符串](https://rustwiki.org/zh-CN/rust-by-example/std/str.html)。
- `print!`：与 `format!` 类似，但将文本输出到控制台（io::stdout）。
- `println!`: 与 `print!` 类似，但输出结果追加一个换行符。
- `eprint!`：与 `print!` 类似，但将文本输出到标准错误（io::stderr）。
- `eprintln!`：与 `eprint!` 类似，但输出结果追加一个换行符。

```rust
fn main() {
    // 通常情况下，`{}` 会被任意变量内容替换
    // 会检查使用到的参数数量是否正确
    println!("{} days", 31);

    // 也可以使用位置参数
    println!("{0}, this is {1}. {1}, this is {0}", "Alice", "Bob");

    // 也可以使用命名参数
    println!("Hello {name}", name="Bob");

    // 可以在 `:` 后面指定格式
    println!("The binary form of 12 is {:b}", 12);

    // 右对齐 `文本`
    println!("{number:>width$}", number=1, width=6);

    // 在 `数字` 左边补0
    println!("{number:>0width$}", number=1, width=6);
}
```

#### 调试（debug）

使用`std::fmt`打印的类型必须实现一个可打印的`traits`。`std`库中的类型提供了自动实现，其他类型需要**手动**实现

- 所有类型都可以**自动创建**`fmt::Debug`这个`trait`，采用 `{:?}` 或 `{:#?}` 标记

- `fmt::Display`需要**手动**实现，采用 `{}` 标记

```rust
// 使用 `derive` 属性自动创建所需的实现
#[derive(Debug)]
struct Structure(i32);

#[derive(Debug)]
struct Person<'a> {
    name: &'a str,
    age: u8
}

fn main() {
    // 使用 `{:?}` 进行 `fmt::Debug` 打印
    println!("{:?}", Structure(3));

    // 同样可以指定位置
    println!("{1:?}{0:?}", Structure(3), Structure(2));

    // 美化打印
    println!("{:#?}", Person{ name: "Peter", age: 27});
}
```

#### 显示（display）

```rust
// 导入 `fmt` 模块使 `fmt::Display` 可用
use std::fmt;

struct Structure(i32);

// 为了使用 `{}` 标记，必须手动为类型实现 `fmt::Display` trait。
impl fmt::Display for Structure {
    // 这个 trait 要求 `fmt` 使用与下面的函数完全一致的函数签名
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // 仅将 self 的第一个元素写入到给定的输出流 `f`。返回 `fmt:Result`，此
        // 结果表明操作成功或失败。注意 `write!` 的用法和 `println!` 很相似。
        write!(f, "{}", self.0)
    }
}
```

因为没有一种合适的样式适用于泛型容器（generic container）中的所有类型，所有对于 `Vec<T>` 或其他任意泛型容器，
`fmt::Display` 都没有实现，因此要使用 `fmt::Debug`

#### 测试实例：List

```rust
use std::fmt; // 导入 `fmt` 模块。

// 定义一个包含单个 `Vec` 的结构体 `List`。
struct List(Vec<i32>);

impl fmt::Display for List {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        // 使用元组的下标获取值，并创建一个 `vec` 的引用。
        let vec = &self.0;
        // 使用 `?` 对 `write!` 进行尝试。如果发生错误则返回相应的错误
        // 否则继续执行后面的语句
        // try!(write!(f, "{}", value)); // 与这种写法等价
        write!(f, "[")?;

        // 使用 `v` 对 `vec` 进行迭代，并用 `count` 记录迭代次数。
        for (count, v) in vec.iter().enumerate() {
            // 对每个元素（第一个元素除外）加上逗号。
            // 使用 `?` 或 `try!` 来返回错误。
            if count != 0 { write!(f, ", ")?; }
            write!(f, "{}", v)?;
        }

        // 加上配对中括号，并返回一个 fmt::Result 值。
        write!(f, "]")
    }
}

fn main() {
    let v = List(vec![1, 2, 3]);
    println!("{}", v);
}
```

## 原生类型（Primitives）

- 标量类型（scalar type）
  - 有符号整数（signed integers）：`i8`、`i16`、`i32`、`i64`、`i128` 和 `isize`（指针宽度）
  - 无符号整数（unsigned integers）： `u8`、`u16`、`u32`、`u64`、`u128` 和 `usize`（指针宽度）
  - 浮点数（floating point）： `f32`、`f64`
  - `char`（字符）：单个 **Unicode** 字符，如 `'a'`，`'α'` 和 `'∞'`（每个**都是 4 字节**）
  - `bool`（布尔型）：只能是 `true` 或 `false`
  - 单元类型（unit type）：`()`。其唯一可能的值就是 `()` 这个空元组，不被认为是复合类型
- 复合类型（compound type）
  - 数组（array）：如 `[1, 2, 3]`
  - 元组（tuple）：如 `(1, true)`

```rust
fn main() {
    // 变量可以显示地给出类型声明（type annotation）
    let logical: bool = true;

    // 通过后缀（suffix）方式声明
    let an_integer = 5i64;

    // 默认方式决定类型
    let default_float   = 3.0; // `f64`
    let default_integer = 7;   // `i32`

    // 通过上下文自动推断类型
    let mut inferred_type = 12;
    inferred_type = 4294967296i64;

    // 可变（mutable）变量
    let mut mutable = 12;
    mutable = 21

    // 可以用遮蔽（shadow）来覆盖前面的变量
    let mutable = true;
}
```

### 字面量和运算符

- 通过加前缀 `0x`、`0o`、`0b`，数字可以用十六进制、八进制或二进制记法表示

- 为了改善可读性，可以在数值字面量中插入下划线，比如：`1_000` 等同于 `1000`，`0.000_001` 等同于 `0.000001`

```rust
fn main() {
    // 整数相加
    println!("1 + 2 = {}", 1u32 + 2);

    // 整数相减
    println!("1 - 2 = {}", 1i32 - 2);

    // 短路求值的布尔逻辑
    println!("true AND false is {}", true && false);
    println!("true OR false is {}", true || false);
    println!("NOT true is {}", !true);

    // 位运算
    println!("0011 AND 0101 is {:04b}", 0b0011u32 & 0b0101);
    println!("0011 OR 0101 is {:04b}", 0b0011u32 | 0b0101);
    println!("0011 XOR 0101 is {:04b}", 0b0011u32 ^ 0b0101);
    println!("1 << 5 is {}", 1u32 << 5);
    println!("0x80 >> 2 is 0x{:x}", 0x80u32 >> 2);
}
```

### 元组

元组可以有任意多个值，使用`()`构造（construct），每个元组自身是一个类型标记为`(T1, T2, ...)`的值。

```rust
// 元组可以充当函数的参数和返回值
fn reverse(pair: (i32, bool)) -> (bool, i32) {
    // 可以使用 `let` 把一个元组的成员绑定到一些变量
    let (integer, boolean) = pair;

    (boolean, integer)
}

#[derive(Debug)]
struct Matrix(f32, f32, f32, f32);

fn main() {
    // 包含各种不同类型的元组
    let long_tuple = (1u8, 2u16, 3u32, 4u64,
                      -1i8, -2i16, -3i32, -4i64,
                      0.1f32, 0.2f64,
                      'a', true);

    // 通过元组的下标来访问具体的值
    println!("long tuple first value: {}", long_tuple.0);
    println!("long tuple second value: {}", long_tuple.1);

    // 元组也可以充当元组的元素
    let tuple_of_tuples = ((1u8, 2u16, 2u32), (4u64, -1i8), -2i16);

    // 元组可以打印
    println!("tuple of tuples: {:?}", tuple_of_tuples);

    // 但很长的元组无法打印
    // let too_long_tuple = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13);
    // println!("too long tuple: {:?}", too_long_tuple);

    let pair = (1, true);
    println!("pair is {:?}", pair);

    println!("the reversed pair is {:?}", reverse(pair));

    // 创建单元素元组需要一个额外的逗号，这是为了和被括号包含的字面量作区分。
    println!("one element tuple: {:?}", (5u32,));
    println!("just an integer: {:?}", (5u32));

    // 元组可以被解构（deconstruct），从而将值绑定给变量
    let tuple = (1, "hello", 4.5, true);

    let (a, b, c, d) = tuple;
    println!("{:?}, {:?}, {:?}, {:?}", a, b, c, d);

    let matrix = Matrix(1.1, 1.2, 2.1, 2.2);
    println!("{:?}", matrix)

}
```

### 数组与切片

- 数组（array）是一组拥有相同类型 `T` 的对象的集合，在内存中连续存储。使用 `[]` 创建，**大小在编译时确定**。类型标记为 `[T; length]`
- 切片（slice）**大小在编译时不确定**。是一个上了双字对象（two-word object），
  第一个字是指向数据的指针，第二个字是切片的长度。“字”的宽度和 `usize` 相同。可以用来**借用数组的一部分**。类型标记为 `&[T]`

```rust
use std::mem;

// 此函数借用一个 slice
fn analyze_slice(slice: &[i32]) {
    println!("first element of the slice: {}", slice[0]);
    println!("the slice has {} elements", slice.len());
}

fn main() {
    // 定长数组（类型标记是多余的）
    let xs: [i32; 5] = [1, 2, 3, 4, 5];

    // 所有元素可以初始化成相同的值，这里为0
    let ys: [i32; 500] = [0; 500];

    // 下标从 0 开始
    println!("first element of the array: {}", xs[0]);
    println!("second element of the array: {}", xs[1]);

    // `len` 返回数组的大小
    println!("array size: {}", xs.len());

    // 数组是在栈中分配的
    println!("array occupies {} bytes", mem::size_of_val(&xs));

    // 数组可以自动被借用成为 slice
    println!("borrow the whole array as a slice");
    analyze_slice(&xs);

    // slice 可以指向数组的一部分
    println!("borrow a section of the array as a slice");
    analyze_slice(&ys[1 .. 4]);

    // 越界的下标会引发致命错误（panic）
    // println!("{}", xs[5]);
}
```

## 自定义类型（Custom Types）

### 结构体（structure）

- 元组结构体（tuple struct），就是具名元组而已
- 经典的 [C 语言风格结构体](<https://en.wikipedia.org/wiki/Struct_(C_programming_language)>)
  （C struct）
- 单元结构体（unit struct），不带字段，在泛型中很有用

```rust
// C struct
struct Point {
    x: f32,
    y: f32,
}

// unit struct
struct Unit;

// tuple struct
struct Pair(i32, i32);

// 结构体可以作为另一个结构体的字段
#[allow(dead_code)]
struct Rectangle {
    top_left: Point,
    bottom_right: Point,
}

fn main(){
    // 实例化结构体 `Point`
    let point: Point = Point { x: 10.3, y: 0.4 };

    // 访问 point 的字段
    println!("point coordinates: ({}, {})", point.x, point.y);

    // 使用结构体更新语法创建新的 point，
    // 这样可以用到之前的 point 的字段
    // 即`bottom_right.y` 与 `point.y` 一样
    let bottom_right = Point { x: 5.2, ..point };

    // 使用 `let` 绑定来解构 point
    let Point { x: left_edge, y: top_edge } = point;

    let _rectangle = Rectangle {
        // 结构体的实例化也是一个表达式
        top_left: Point { x: left_edge, y: top_edge },
        bottom_right: bottom_right,
    };

    // 实例化一个单元结构体
    let _unit = Unit;

    // 实例化一个元组结构体
    let pair = Pair(1, 0.1);

    // 访问元组结构体的字段
    println!("pair contains {:?} and {:?}", pair.0, pair.1);

    // 解构一个元组结构体
    let Pair(integer, decimal) = pair;

    println!("pair contains {:?} and {:?}", integer, decimal);
}
```

### 枚举（enum）

```rust
// 该属性用于隐藏对未使用代码的警告。
#![allow(dead_code)]

// 创建一个 `enum`（枚举）来对 web 事件分类。
// 取值的种类：`PageLoad` 不等于 `PageUnload`，`KeyPress(char)` 不等于
// `Paste(String)`。各个取值不同，互相独立。
enum WebEvent {
    // 一个 `enum` 可以是单元结构体（称为 `unit-like` 或 `unit`），
    PageLoad,
    PageUnload,
    // 或者一个元组结构体，
    KeyPress(char),
    Paste(String),
    // 或者一个普通的结构体。
    Click { x: i64, y: i64 }
}

// 此函数将一个 `WebEvent` enum 作为参数，无返回值。
fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::PageUnload => println!("page unloaded"),
        // 从 `enum` 里解构出 `c`。
        WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
        WebEvent::Paste(s) => println!("pasted \"{}\".", s),
        // 把 `Click` 解构给 `x` and `y`。
        WebEvent::Click { x, y } => {
            println!("clicked at x={}, y={}.", x, y);
        },
    }
}

fn main() {
    let pressed = WebEvent::KeyPress('x');
    // `to_owned()` 从一个字符串切片中创建一个具有所有权的 `String`。
    let pasted  = WebEvent::Paste("my text".to_owned());
    let click   = WebEvent::Click { x: 20, y: 80 };
    let load    = WebEvent::PageLoad;
    let unload  = WebEvent::PageUnload;

    inspect(pressed);
    inspect(pasted);
    inspect(click);
    inspect(load);
    inspect(unload);
}
```

```rust
// 类型别名
enum VeryVerboseEnumOfThingsToDoWithNumbers {
    Add,
    Subtract,
}

impl VeryVerboseEnumOfThingsToDoWithNumbers {
    fn run(&self, x: i32, y: i32) -> i32 {
        match self {
            // 使用 `Self` 类型别名
            Self::Add => x + y,
            Self::Subtract => x - y,
        }
    }
}

// 创建一个类型别名
type Operations = VeryVerboseEnumOfThingsToDoWithNumbers;

fn main() {
    // We can refer to each variant via its alias,
    // not its long and inconvenient name.
    let x = Operations::Add;
}
```

#### 使用 use

使用 `use` 声明可以不写出完整的路径

```rust
// 该属性用于隐藏对未使用代码的警告。
#![allow(dead_code)]

enum Status {
    Rich,
    Poor,
}

enum Work {
    Civilian,
    Soldier,
}

fn main() {
    // 显式地 `use` 各个名称使他们直接可用，而不需要指定它们来自 `Status`。
    use Status::{Poor, Rich};
    // 自动地 `use` `Work` 内部的所有名称。
    use Work::*;

    // `Poor` 等价于 `Status::Poor`。
    let status = Poor;
    // `Civilian` 等价于 `Work::Civilian`。
    let work = Civilian;

    match status {
        // 注意这里没有用完整路径，因为上面显式地使用了 `use`。
        Rich => println!("The rich have lots of money!"),
        Poor => println!("The poor have no money..."),
    }

    match work {
        // 再次注意到没有用完整路径。
        Civilian => println!("Civilians work!"),
        Soldier  => println!("Soldiers fight!"),
    }
}
```

#### C 风格用法

```rust
// 该属性用于隐藏对未使用代码的警告。
#![allow(dead_code)]

// 拥有隐式辨别值（implicit discriminator，从 0 开始）的 enum
enum Number {
    Zero,
    One,
    Two,
}

// 拥有显式辨别值（explicit discriminator）的 enum
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}

fn main() {
    // `enum` 可以转成整型。
    println!("zero is {}", Number::Zero as i32);
    println!("one is {}", Number::One as i32);

    println!("roses are #{:06x}", Color::Red as i32);
    println!("violets are #{:06x}", Color::Blue as i32);
}
```

### 常量

- `const` 不可改变值
- `static` 具有 `'static` 生命周期的，可以是可变的变量

```rust
// 全局变量是在所有其他作用域之外声明的。
static LANGUAGE: &'static str = "Rust";
const  THRESHOLD: i32 = 10;

fn is_big(n: i32) -> bool {
    // 在一般函数中访问常量
    n > THRESHOLD
}

fn main() {
    let n = 16;

    // 在 main 函数（主函数）中访问常量
    println!("This is {}", LANGUAGE);
    println!("The threshold is {}", THRESHOLD);
    println!("{} is {}", n, if is_big(n) { "big" } else { "small" });

    // 报错！不能修改一个 `const` 常量。
    // THRESHOLD = 5;
}
```

## 变量绑定（Variable Bindings）

```rust
fn main() {
    let an_integer = 1u32;
    let a_boolean = true;
    let unit = ();

    // 将 `an_integer` 复制到 `copied_integer`
    let copied_integer = an_integer;

    println!("An integer: {:?}", copied_integer);
    println!("A boolean: {:?}", a_boolean);
    println!("Meet the unit value: {:?}", unit);

    // 编译器会对未使用的变量绑定产生警告；可以给变量名加上下划线前缀来消除警告。
    let _unused_variable = 3u32;
}
```

### 可变变量

变量绑定默认是不可变的（immutable），加上 `mut` 修饰语后就变为可变的（mutable）

```rust
fn main() {
    let mut mutable_binding = 1;

    println!("Before mutation: {}", mutable_binding);

    mutable_binding += 1;

    println!("After mutation: {}", mutable_binding);
}
```

### 作用域与遮蔽

变量绑定有一个作用域（scope），被限定在一个**代码块**（block）中生存（live）。代码块是一个被 `{}` 包围的语句集合。

```rust
fn main() {
    // 此绑定生存于 main 函数中
    let long_lived_binding = 1;

    // 这是一个代码块，比 main 函数拥有更小的作用域
    {
        // 此绑定只存在于本代码块
        let short_lived_binding = 2;

        println!("inner short: {}", short_lived_binding);

        // 此绑定`遮蔽（shadow）`了外面的绑定
        let long_lived_binding = 5_f32;

        println!("inner long: {}", long_lived_binding);
    }
    // 代码块结束

    // 报错！`short_lived_binding` 在此作用域上不存在
    // println!("outer short: {}", short_lived_binding);

    println!("outer long: {}", long_lived_binding);

    // 此绑定同样*遮蔽*了前面的绑定
    let long_lived_binding = 'a';

    println!("outer long: {}", long_lived_binding);
}
```

### 变量先声明

先声明（declare）变量绑定，后面才将它们初始化（initialize）。

```rust
fn main() {
    // 声明一个变量绑定
    let a_binding;

    {
        let x = 2;

        // 初始化一个绑定
        a_binding = x * x;
    }

    println!("a binding: {}", a_binding);

    let another_binding;

    // 报错！使用了未初始化的绑定
    // println!("another binding: {}", another_binding);

    another_binding = 1;

    println!("another binding: {}", another_binding);
}
```

### 冻结

当数据被相同的名称不变地绑定时，它还会**冻结**（freeze）

```rust
fn main() {
    let mut _mutable_integer = 7i32;

    {
        // 被不可变的 `_mutable_integer` 遮蔽
        let _mutable_integer = _mutable_integer;

        // 报错！`_mutable_integer` 在本作用域被冻结
        // _mutable_integer = 50;

        // `_mutable_integer` 离开作用域
    }

    // 正常运行！ `_mutable_integer` 在这个作用域没有冻结
    _mutable_integer = 3;
}
```

## 类型系统（Types）

### 5.1. 类型转换

Rust 不提供原生类型之间的隐式类型转换（coercion），但可以使用 `as` 关键字进行显示类型转换（casting）。

```rust
// 不显示类型转换产生的溢出警告。
#![allow(overflowing_literals)]

fn main() {
    let decimal = 65.4321_f32;

    // 错误！不提供隐式转换
    // let integer: u8 = decimal;

    // 可以显式转换
    let integer = decimal as u8;
    let character = integer as char;

    println!("Casting: {} -> {} -> {}", decimal, integer, character);

    // 当把任何类型转换为无符号类型 T 时，会不断加上或减去 (std::T::MAX + 1)
    // 直到值位于新类型 T 的范围内。

    // 1000 已经在 u16 的范围内
    println!("1000 as a u16 is: {}", 1000 as u16);

    // 1000 - 256 - 256 - 256 = 232
    // 事实上的处理方式是：从最低有效位（LSB，least significant bits）开始保留
    // 8 位，然后剩余位置，直到最高有效位（MSB，most significant bit）都被抛弃。
    // 最左边一位和最右边一位。
    println!("1000 as a u8 is : {}", 1000 as u8);
    // -1 + 256 = 255
    println!("  -1 as a u8 is : {}", (-1i8) as u8);

    // 对正数，这就和取模一样。
    println!("1000 mod 256 is : {}", 1000 % 256);

    // 当转换到有符号类型时，（位操作的）结果就和 “先转换到对应的无符号类型，
    // 如果 MSB 是 1，则该值为负” 是一样的。

    // 当然如果数值已经在目标类型的范围内，就直接把它放进去。
    println!(" 128 as a i16 is: {}", 128 as i16);
    // 128 转成 u8 还是 128，但转到 i8 相当于给 128 取八位的二进制补码，其值是：
    println!(" 128 as a i8 is : {}", 128 as i8);
}
```

### 字面量

见`原生类型（Primitives）`

### 类型推断

见`原生类型（Primitives）`

### 别名

用 `type` 语句给已有的类型取一个新的名字。类型的名字必须遵守**驼峰命名法**（除一些原生类型外 ：`usize` 、`f32`）

```rust
// `NanoSecond` 是 `u64` 的新名字。
type NanoSecond = u64;
type Inch = u64;

fn main() {
    // `NanoSecond` = `Inch` = `u64_t` = `u64`.
    let nanoseconds: NanoSecond = 5 as u64_t;
    let inches: Inch = 2 as u64_t;

    // 注意类型别名*并不能*提供额外的类型安全，因为别名*并不是*新的类型。
    println!("{} nanoseconds + {} inches = {} unit?",
             nanoseconds,
             inches,
             nanoseconds + inches);
}
```

## 类型转换（Conversion）

### `From` 和 `Into`

`From` 和 `Into` 两个 trait 是内部相关联的。如果我们能够从类型 B 得到类型 A，那么很容易相信我们也能把类型 B 转换为类型 A。

### 6.1.1. From

`From` trait 允许定义一种类型“怎么根据另一种类型*生成自己*”。

比如，把 `str` 转换成 `String`

```rust
let my_str = "hello";
let my_string = String::from(my_str);
```

也可以自己定义类型转换机制

```rust
use std::convert::From;

#[derive(Debug)]
struct Number {
    value: i32,
}

impl From<i32> for Number {
    fn from(item: i32) -> Self {
        Number { value: item }
    }
}

fn main() {
    let num = Number::from(30);
    println!("My number is {:?}", num);
}
```

#### Into

`Into` trait 就是把 `From` trait 倒过来而已。如果为类型实现了 `From`，同时就免费获得了 `Into`。

使用 `Into` trait 通常要指明要转换到的类型，因为编译器大多数时候不能推断它。

```rust
use std::convert::From;

#[derive(Debug)]
struct Number {
    value: i32,
}

impl From<i32> for Number {
    fn from(item: i32) -> Self {
        Number { value: item }
    }
}

fn main() {
    let int = 5;
    let num: Number = int.into();
    println!("My number is {:?}", num);
}
```

### `TryFrom` 和 `TryInto`

`TryFrom` 和 `TryInto` 用于易出错的转换，其返回值是 `Result` 型。

```rust
use std::convert::TryFrom;
use std::convert::TryInto;

#[derive(Debug, PartialEq)]
struct EvenNumber(i32);

impl TryFrom<i32> for EvenNumber {
    type Error = ();

    fn try_from(value: i32) -> Result<Self, Self::Error> {
        if value % 2 == 0 {
            Ok(EvenNumber(value))
        } else {
            Err(())
        }
    }
}

fn main() {
    // TryFrom

    assert_eq!(EvenNumber::try_from(8), Ok(EvenNumber(8)));
    assert_eq!(EvenNumber::try_from(5), Err(()));

    // TryInto

    let result: Result<EvenNumber, ()> = 8i32.try_into();
    assert_eq!(result, Ok(EvenNumber(8)));
    let result: Result<EvenNumber, ()> = 5i32.try_into();
    assert_eq!(result, Err(()));
}
```

### `ToString` 和 `FromStr`

#### `ToString`

实现 `ToString` trait 则可把任何类型转换成 `String`。

```rust
use std::string::ToString;

struct Circle {
    radius: i32
}

impl ToString for Circle {
    fn to_string(&self) -> String {
        format!("Circle of radius {:?}", self.radius)
    }
}

fn main() {
    let circle = Circle { radius: 6 };
    println!("{}", circle.to_string());
}
```

实现 `fmt::Display` 也会自动提供 `ToString`。

```rust
use std::fmt;

struct Circle {
    radius: i32
}

impl fmt::Display for Circle {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Circle of radius {}", self.radius)
    }
}

fn main() {
    let circle = Circle { radius: 6 };
    println!("{}", circle.to_string());
}
```

#### 解析字符串

实现 `FromStr` 后，就可使用 `parse` 函数解析字符串。要提供目标类型或使用 `<>` （turbo fish）实现。

标准库中已经给多种类型实现了 `FromStr` ，也可手动实现 `FromStr`。

```rust
fn main() {
    let parsed: i32 = "5".parse().unwrap();
    let turbo_parsed = "10".parse::<i32>().unwrap();

    let sum = parsed + turbo_parsed;
    println!{"Sum: {:?}", sum};
}
```

## 表达式（Expressions）

语句类型：

- 声明变量绑定
- 表达式带上 `;`

```rust
fn main() {
    // 语句
    let x= 5;
    println!("One is {}", 1);

    // 表达式
    // 代码块也是表达式，值为代码块中最后执行的表达式的值
    {
        // 表达式，没有分号
        2
    }
}
```
