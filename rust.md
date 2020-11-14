# RUST



## 猜数游戏

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number");
    let secret_number = rand::thread_rng().gen_range(1, 101);
    println!("The secret number is: {}", secret_number);
    // 循环
    loop {
        println!("Please input your guess.");
        // 初始化可变量
        let mut guess = String::new();
        io::stdin()
            .read_line(&mut guess) // 读取标准输入
            .expect("Failed to read line"); // 异常处理

        // let guess: u32 = guess.trim().parse().expect("Please type a number.");
        // 对 io::Result 的 match 匹配应用
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(msg) => {
                println!("{}", msg);
                continue;
            }
        };

        println!("You guessed: {}", guess);
        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("you win!");
                break;
            }
        }
    }
}
```

## minigrep

// src/main.rs

```rust
use std::env;
use std::process;

// 当前项目名为minigrep
use minigrep::Config;

fn main() {
    // 读取参数
    // 参数-0: 执行文件
    let args: Vec<String> = env::args().collect();

    // 注意使用 minigrep::Config 结构与 minigrep::run 方法的前缀不同
    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    println!("searching for {}", config.query);
    println!("in file {}", config.filename);

    // 使用lib.rs
    // 注意使用 minigrep::Config 结构与 minigrep::run 方法的前缀不同
    if let Err(e) = minigrep::run(config) {
        println!("Application error: {}", e);
        process::exit(1);
    }
}
```

// src/lib.rs

```rust
use std::env;
use std::error::Error;
use std::fs;

// 注意 pub 提供包外访问
pub struct Config {
    pub query: String,
    pub filename: String,
    pub case_sensitive: bool,
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }
        let query = args[1].clone();
        let filename = args[2].clone();

        let case_sensitive = env::var("CASE_INSENSITIVE") // 读取环境变量
            .is_err(); // 转换为bool值

        Ok(Config {
            query,
            filename,
            case_sensitive,
        })
    }
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.filename)?;

    let results = if config.case_sensitive {
        search(&config.query, &contents)
    } else {
        search_case_insensitive(&config.query, &contents)
    };

    for line in results {
        println!("{}", line)
    }

    Ok(())
}

// 单元测试可以混合写入
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn one_result() {
        let query = "main1";
        let contents = "\
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        process::exit(1);
    });


    if let Err(e) = minigrep::run(config) {
        process::exit(1);
    }
}";
        assert_eq!(vec!["fn main() {"], search(query, contents));
    }
}

// 区分大小写
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();
    for line in contents.lines() {
        // contains 只接受借用参数
        if line.contains(&query) {
            results.push(line);
        }
    }
    results
}



// 不区分大小写
pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    // 为忽略大小写
    let query = query.to_lowercase();
    let mut results = Vec::new();
    for line in contents.lines() {
        // contains 只接受借用参数
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }
    results
}

```

### 优化 clone

```rust
// main.rs
fn main() {
  	// 传递所有权
    let config = Config::new(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}

// lib.rs -- 1
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        // trucate filename
        args.next();
      	// get query
        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };
        // get filename
        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };
        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();
        Ok(Config { query, filename, case_sensitive })
    }
}

// 更简洁的search
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```



## 安装

```shell
# download
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh

# 1
source $HOME/.cargo/env

# 2
export PATH="$HOME/.cargo/bin:$PATH"
```

## 核心概念

### 栈

> 栈以放入值的顺序存储值并以相反顺序取出值。这也被称作 **后进先出**（*last in, first out*），**入栈比在堆上分配内存要快**，因为（入栈时）操作系统无需为存储新数据去搜索内存空间；其位置**总是在栈顶**。

### 堆

> 栈中的所有数据都必须占用已知且固定的大小。在编译时大小未知或大小可能变化的数据，要改为存储在堆上。堆是缺乏组织的：当向堆放入数据时，你要请求一定大小的空间。操作系统在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的 **指针**（*pointer*）。这个过程称作 **在堆上分配内存**（*allocating on the heap*），有时简称为 “分配”（allocating）。将数据推入栈中并不被认为是分配。因为指针的大小是已知并且固定的，你可以将指针存储在栈上，不过当需要实际数据时，必须访问指针。

### 所有权规则

1. Rust 中的每一个值都有一个被称为其 **所有者**（*owner*）的变量。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者（变量）离开作用域，这个值将被丢弃。
4. Rust 永远也不会自动创建数据的 “深拷贝”。因此，任何 **自动** 的复制可以被认为对运行时性能影响较小。
5. 像**整型这样的在编译时已知大小的类型**被整个存储在栈上，所以拷贝其实际的值是快速的。这意味着没有理由在创建变量 `y` 后使 `x` 无效。
6. Rust 有一个叫做 `Copy` trait 的特殊注解，可以用在类似整型这样的存储在栈上的类型上。Rust 不允许自身或其任何部分实现了 `Drop` trait 的类型使用 `Copy` trait。

```rust
// 将 5 绑定到 x；接着生成一个值 x 的拷贝并绑定到 y”。现在有了两个变量，x 和 y，都等于 5。这也正是事实上发生了的，因为整数是有已知固定大小的简单值，所以这两个 5 被放入了栈中。
let x = 5;
let y = x;

// 与其尝试拷贝被分配的内存，Rust 则认为 s1 不再有效，因此 Rust 不需要在 s1 离开作用域后清理任何东西。看看在 s2 被创建之后尝试使用 s1 会发生什么；这段代码不能运行：
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

### 什么类型是 Copy 的？

- 所有整数类型，比如 `u32`。
- 布尔类型，`bool`，它的值是 `true` 和 `false`。
- 所有浮点数类型，比如 `f64`。
- 字符类型，`char`。
- 元组，当且仅当其包含的类型也都是 `Copy` 的时候。比如，`(i32, i32)` 是 `Copy` 的，**但 `(i32, String)` 就不是**。

### 所有权与函数 

```rust
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 所以不会有特殊操作

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。不会有特殊操作
```

### 返回值与作用域

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership 将返回值
                                        // 移给 s1

    let s2 = String::from("hello");     // s2 进入作用域

    let s3 = takes_and_gives_back(s2);  // s2 被移动到
                                        // takes_and_gives_back 中,
                                        // 它也将返回值移给 s3
} // 这里, s3 移出作用域并被丢弃。s2 也移出作用域，但已被移走，
  // 所以什么也不会发生。s1 移出作用域并被丢弃

fn gives_ownership() -> String {             // gives_ownership 将返回值移动给
                                             // 调用它的函数

    let some_string = String::from("hello"); // some_string 进入作用域.

    some_string                              // 返回 some_string 并移出给调用的函数
}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String { // a_string 进入作用域

    a_string  // 返回 a_string 并移出给调用的函数
}



```

## 引用与借用

> 基于基本所有权规则的代码内容

```rust
// 引用的前言
// 我们必须将 String 返回给调用函数，以便在调用 calculate_length 后仍能使用 String，因为 String 被移动到了 calculate_length 内。
fn main() {
    let s1 = String::from("hello");
    let (s2, len) = calculate_length(s1);
    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度
    (s, length)
}
```

> 改为引用后的代码内容:  <u>这些 & 符号就是 **引用**，它们允许你使用值但不获取其所有权，我们将获取引用作为函数参数称为 **借用**（*borrowing*）。</u>

```rust
fn main() {
    let s1 = String::from("hello");
		// &s1 语法让我们创建一个 指向 值 s1 的引用，但是并不拥有它。因为并不拥有这个值，当引用离开作用域时其指向的值也不会被丢弃。
    let len = calculate_length(&s1);
    println!("The length of '{}' is {}.", s1, len);
}
// 同理，函数签名使用 & 来表明参数 s 的类型是一个引用。让我们增加一些解释性的注释：
fn calculate_length(s: &String) -> usize {
  	// 借用无法修改变量
  	// s.push_str(" ,world");  // error
    s.len()
}
```

## 可变引用

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

> 在特定作用域中的特定数据只能有一个可变引用。这些代码会失败：

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);
```

> 使用 {} 创建新的作用域来允许拥有多个可变引用

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用

let r2 = &mut s;
```

> 同时使用可变与不可变引用中。这些代码会导致一个错误：

```rust
// 我们 也 不能在拥有不可变引用的同时拥有可变引用。
let mut s = String::from("hello");
let r1 = &s; // 没问题
let r2 = &s; // 没问题
let r3 = &mut s; // 大问题
println!("{}, {}, and {}", r1, r2, r3);

// 多个不可变引用是可以的，因为没有哪个只能读取数据的人有能力影响其他人读取到的数据。
// 注意一个引用的作用域从声明的地方开始一直持续到最后一次使用为止。例如，因为最后一次使用不可变引用在声明可变引用之前，所以如下代码是可以编译的：
// 不可变引用 r1 和 r2 的作用域在 println! 最后一次使用之后结束，这也是创建可变引用 r3 的地方。它们的作用域没有重叠，所以代码是可以编译的。
let mut s = String::from("hello");
let r1 = &s; // 没问题
let r2 = &s; // 没问题
println!("{} and {}", r1, r2);
// 此位置之后 r1 和 r2 不再使用
let r3 = &mut s; // 没问题
println!("{}", r3);
```

### 悬垂引用 

> 在具有指针的语言中，很容易通过释放内存时保留指向它的指针而错误地生成一个 **悬垂指针**（*dangling pointer*），所谓悬垂指针是其指向的内存可能已经被分配给其它持有者。相比之下，在 Rust 中编译器确保引用永远也不会变成悬垂状态：当你拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域。

## Slices (无所有权)

> 字符串

```rust
// “字符串 slice” 的类型声明写作 &str
// 更好的方式： fn first_word(s: &str) -> &str {
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }
    &s[..]
}
```

> 数值 slice / &[i32]

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```

## 结构

```rust
// 定义
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
//struct User {
//    username: &str,
//    email: &str,
//    sign_in_count: u64,
//    active: bool,
//}

// 实例
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

// 可变实例
// 注意整个实例必须是可变的；Rust 并不允许只将某个字段标记为可变。
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
user1.email = String::from("anotheremail@example.com");

// 结构更新语法
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

### 元组结构体

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

### 通过派生 trait 增加实用功能

```rust
// 增加注解派生
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("rect1 is {:?}", rect1);
	  // rect1 is Rectangle { width: 30, height: 50 }
  	println!("rect1 is {:#?}", rect1);
  	//rect1 is Rectangle {
    //		width: 30,
    //		height: 50
		//}
}
```

### 结构体方法

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
  	// 使用 &self 来替代 rectangle: &Rectangle，
    // 因为该方法位于 impl Rectangle 上下文中所以 Rust 知道 self 的类型是 Rectangle。
    // 注意仍然需要在 self 前面加上 &，就像 &Rectangle 一样。方法可以选择获取 self 的所有权，
    // 或者像我们这里一样不可变地借用 self，或者可变地借用 self，就跟其他参数一样
    fn area(&self) -> u32 {
        self.width * self.height
    }
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

// 可以使用多个 impl 块重写
impl Rectangle {
    // let sq = Rectangle::square(3);
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

## 枚举 *enums*

```rust
// 基本定义
enum IpAddrKind {
    V4,
    V6,
}
let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

// 将数据直接放进每一个枚举成
enum IpAddr {
    V4(String),
    V6(String),
}
let home = IpAddr::V4(String::from("127.0.0.1"));

// 不同类型
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}
let home = IpAddr::V4(127, 0, 0, 1);


```

## Option

> Option::Some / Option::None 使用非常频繁，所以被独立出来使用。

```rust
enum Option<T> {
    Some(T),
    None,
}
// 
let some_number = Some(5);
let some_string = Some("a string");
let absent_number: Option<i32> = None;
```



## 宏 `!`

> `!`  println!

## Cargo

```shell
# create a project
cargo new project_name

# create a lib
cargo new --lib lib_name

# 
cargo run 

#
cargo build

# 
cargo check

#
cargo update
```

## 导入

```rust
use std::io;
```

## （可）变量

```rust
// a string
let mut guess = String::new();

// ...
let foo = 5; // immutable
let mut bar = 5; // mutable

// 隐藏
let foo = 5;
let foo = "string"
```

## 数据类型

| 长度    | 有符号  | 无符号  |
| ------- | ------- | ------- |
| 8-bit   | `i8`    | `u8`    |
| 16-bit  | `i16`   | `u16`   |
| 32-bit  | `i32`   | `u32`   |
| 64-bit  | `i64`   | `u64`   |
| 128-bit | `i128`  | `u128`  |
| arch    | `isize` | `usize` |
| f32     |         |         |
| f64     |         |         |

### 数值

> 每一个有符号的变体可以储存包含从 -(2n - 1) 到 2n - 1 - 1 在内的数字，这里 *n* 是变体使用的位数。所以 `i8` 可以储存从 -(27) 到 27 - 1 在内的数字，也就是从 -128 到 127。无符号的变体可以储存从 0 到 2n - 1 的数字，所以 `u8` 可以储存从 0 到 28 - 1 的数字，也就是从 0 到 255。

### 字面值

| Decimal (十进制)              | `98_222`      |
| ----------------------------- | ------------- |
| Hex (十六进制)                | `0xff`        |
| Octal (八进制)                | `0o77`        |
| Binary (二进制)               | `0b1111_0000` |
| Byte (单字节字符)(仅限于`u8`) | `b'A'`        |

## 常量

```rust
const MAX_POINTS: u32 = 100_000; // 下划线提高可读性
```

## 元组

### 赋值与分解

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

### 索引访问

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

## 组数与Vector

> 数组不能改变长度，而vector可以。

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
let a = [3; 5];
let a = [1, 2, 3, 4, 5];

let first = a[0];
let second = a[1];
```

## 函数

### 返回值

> 返回值只存在于表达式，而一个函数就是一个表达式，所以返回值（表达式）不能写成一个语句。概念上与lisp相似。(数字本身就是一个表达式)

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1
    // x + 1;  - help: consider removing this semicolon // 错误
}
```



## 表达式

```rust
fn main() {
    let x = 5;
		// 这个值作为 let 语句的一部分被绑定到 y 上。注意结尾没有分号的那一行 x+1，
    // 与你见过的大部分代码行不同。表达式的结尾没有分号。如果在表达式的结尾加上分号，
    // 它就变成了语句，而语句不会返回值。在接下来探索具有返回值的函数和表达式时要谨记这一点。
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

## 控制流

### if

> 比较特别的地方是能和 let 配合

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

### match

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
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

### 匹配 Option

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

// _ 通配符
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

### if let

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
// 使用 if let 优化
if let Some(3) = some_u8_value {
    println!("three");
}

let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
// 使用 if let 优化
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```





### loop & while

> loop 配合 let 使用 break 返回值 / while?

```rust
// loop
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
// while
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```

### for .. in ..

> 如何获得下标？

```rust
// 遍历
fn main() {
    let a = [10, 20, 30, 40, 50];
	
    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
// 反转遍历
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```



## io::Result type

```rust
// io::result 是一个枚举
// Ok 与 Err 是正确与失败的两种操作
// expect 是 Result 的异常捕获方法
```

## 添加依赖

```toml
# Filename: Cargo.toml
[dependencies]
rand = "0.5.5"
```

```bash
cargo build
```

## 特征

```rust
// 大写特征？
use rand::Rng;
```



## 包管理

### pub

```rust
mod front_of_house {
  	// 不加 pub 关键字无法
    pub mod hosting {
        // 同上
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

### super

```rust
fn serve_order() {}
mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
      	// serve_order 的 mod 是 back_of_house
        // back_of_house 的父级是根，
        // 根的 serve_order();
        super::serve_order();
    }
    fn cook_order() {}
}
```

### use / as

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```

> pub use <u>它的意思是这样的，把深层的 item 导出到上层目录中，使调用的时候，更方便。接口设计中会大量用到这个技术。</u>

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

### 嵌套路径导入

```rust
use std::{cmp::Ordering, io}; // 导入 cmp::Ordering 和 io

use std::io;
use std::io::Write;
// 改进
use std::io::{self, Write};

// 所有公有定义
use std::collections::*;
```

### 分割进不同文件中

```rust
// src/lib.rs
// 申明 front_of_house 引入模块同名文件内容
mod front_of_house;
pub use crate::front_of_house::hosting;
pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}

// src/front_of_house.rs
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

## Vector

```rust
// 遍例与修改值
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50; // i = &i, 所以需要解引用
}

// 与枚举配合存储任意类型
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

## HashMap

> 利用 collect 与 vector 生成 hash map

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

> hash map 将获得 Copy trait 的拷贝或其它类型的所有权

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// 这里 field_name 和 field_value 不再有效，
// 尝试使用它们看看会出现什么编译错误！
```

> 遍历 & get

```rust
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name);
    println!("{:?}", score); // 是一个Option

    for (key, value) in &scores {
        println!("{:?}: {:?}", key, value);
    }
}
// Some(10)
// "Yellow": 50
// "Blue": 10
```

> 只在键没有对应值时插入

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

> 初始化并持续更新

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

## 错误处理

> 如果你需要项目的最终二进制文件越小越好

```toml
# Cargo.toml
[profile.release]
panic = 'abort'
```

> 显示一个调用栈

```rust
RUST_BACKTRACE=1 cargo run
```

> 匹配不同错误

```rust
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

// 减少 match 的改写方法
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

### unwrap 与 expect

> unwrap 触发 panic!
>
> expect 处理（显示）成自定义显示

### 向上传播Result与 `？ ` 的作用。

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");
    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e), // 此处向上传播代码
    };
    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),    // 同下
        Err(e) => Err(e),  // 最后的表达示，无需写 return
    }
}

// 优化过后的代码
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}

// 进阶链式调用
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
// 最短的常用版本
use std::io;
use std::fs;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```

### 关于 `？` 的细节

> 当你期望在不返回 `Result` 的函数中调用其他返回 `Result` 的函数时使用 `?` 的话，有两种方法修复这个问题。一种技巧是将函数返回值类型修改为 `Result<T, E>`，如果没有其它限制阻止你这么做的话。另一种技巧是通过合适的方法使用 `match` 或 `Result` 的方法之一来处理 `Result<T, E>`。

```rust
// 编译出错版本
use std::fs::File;
fn main() {
    let f = File::open("hello.txt")?;
}

// 编译正常版本
use std::error::Error;
use std::fs::File;
// 目前可以理解 Box<dyn Error> 为使用 ? 时 main 允许返回的 “任何类型的错误”。
fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;
    Ok(())
}
```

## 泛型

> Rust 实现了泛型，使得使用泛型类型参数的代码相比使用具体类型并没有任何速度上的损失


### 使用泛型 (暂不可用)

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("The largest number is {}", result);
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

### 泛型与结构

```rust
// 单个泛型
struct Point<T> {
    x: T,
    y: T,
}
fn main() {
    // 类型必须一样
    // let wont_work = Point { x: 5, y: 4.0 }; // error
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}

// 多个泛型
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

### 枚举与泛型

```rust
// Option
enum Option<T> {
    Some(T),
    None,
}

// Result
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### 方法中定义泛型

```rust
// 单个泛型
struct Point<T> {
    x: T,
    y: T,
}
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.x());
}

// 多个泛型
struct Point<T, U> {
    x: T,
    y: U,
}
impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};
    let p3 = p1.mixup(p2);
    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

## trait

### 定义

```rust
// 定义 trait
pub trait Summary {
    fn summarize(&self) -> String;
}

// 实现 trait
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

// 调用 trait
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

### 默认的实现

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}
// 空 impl 块
impl Summary for NewsArticle {}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}
// 空 impl 块
impl Summary for Tweet {}

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
```

### trait 作为参数

```rust
// trait bound 写法（更适合较复杂的情况）
// 例： pub fn notify<T: Summary>(item1: T, item2: T) {
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
// 语法糖
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

### 多个 trait bound 组合

```rust
// 语法糖写法
pub fn notify(item: impl Summary + Display) {}
  
// trait bound 写法
pub fn notify<T: Summary + Display>(item: T) {}
```

### 使用 where 简化 trait bound 写法

```rust
// 常规写法
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {}

// 使用 where
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### trait 可以作为一个返回值类型

```rust
fn returns_summarizable() -> impl Summary {}
```

### 用例

```rust
// PartialOrd 支持 i32
// Copy 支持 char
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("The largest number is {}", result);
    let char_list = vec!['y', 'm', 'a', 'q'];
    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

### 有条件实现

```rust
// 标准库为任何实现了 Display trait 的类型实现了 ToString trait。
// 以 Display 为条件，实现 ToString trait
// 链式捆绑？
impl<T: Display> ToString for T {
    // --snip--
}
```

## 生命周期与注解

> 编译器采用三条规则来判断引用何时不需要明确的注解。第一条规则适用于输入生命周期，后两条规则适用于输出生命周期。如果编译器检查完这三条规则后仍然存在没有计算出生命周期的引用，编译器将会停止并生成错误。这些规则适用于 `fn` 定义，以及 `impl` 块。
>
> 第一条规则是每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。
>
> 第二条规则是如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。
>
> 第三条规则是如果方法有多个输入生命周期参数并且其中一个参数是 `&self` 或 `&mut self`，说明是个对象的方法(method)(译者注： 这里涉及rust的面向对象参见17章), 那么所有输出生命周期参数被赋予 `self` 的生命周期。第三条规则使得方法更容易读写，因为只需更少的符号。

### 不能编译的函数

```rust
// ->&str 是一个引用，
// 在返回时无法确定是x还是y
// 所以在编译期（借用检查器）无法进行检查而无法通过所有权规则
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

```shell
error[E0106]: missing lifetime specifier
 --> src/main.rs:1:33
  |
1 | fn longest(x: &str, y: &str) -> &str {
  |                                 ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the
signature does not say whether it is borrowed from `x` or `y`
```

> 带有生命周期注解的例子

```rust
// 此例说明了 x与y 拥有相同的生命周期
// 返回值一样
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### 举例

```rust
// 编译正常
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}

// 编译错误
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

### 举例（结构）

```rust
// 这个结构体有一个字段，part，它存放了一个字符串 slice，这是一个引用。
// 类似于泛型参数类型，必须在结构体名称后面的尖括号中声明泛型生命周期参数，以便在结构体定义中使用生命周期参数。
// 这个注解意味着 ImportantExcerpt 的实例不能比其 part 字段中的引用存在的更久。
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

### 方法定义中的生命周期注解

> 当为带有生命周期的结构体实现方法时。声明和使用生命周期参数的位置依赖于生命周期参数是否同结构体字段或方法参数和返回值相关。

```rust
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

### 静态生命周期

> 这里有一种特殊的生命周期值得讨论：`'static`，其生命周期**能够**存活于整个程序期间。所有的字符串字面值都拥有 `'static` 生命周期，我们也可以选择像下面这样标注出来：

```rust
let s: &'static str = "I have a static lifetime.";
```

### 综合`生命周期`与`泛型`、`trait` 混合使用的例子

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

## 测试用例

> 创建一个lib，自带测试用例

```rust
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

### assert! / assert_eq! / assert_ne!

> Src/lib.rs

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        // 使用 assert_eq!
        assert_eq!(2 + 2, 4);
      	// 使用 assert_ne!
      	assert_ne!(2 + 2, 5);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
  
  	#[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { width: 8, height: 7 };
        let smaller = Rectangle { width: 5, height: 1 };
				// 使用 assert!
        assert!(larger.can_hold(&smaller));
      
      	// 增加更多有用的信息
      	assert!(
        	result.contains("Carol"),
	        "Greeting did not contain name, value was `{}`", result
  		  );
    }
}
```



### 使用 should_panic 检查 panic

```rust
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
  	
  	#[test]
    #[ignore]
    fn expensive_test() {
        // 被忽略的测试函数
    }
}
```



### 使用 Result<T,E> 用于测试

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```



### 相关命令

```shell
cargo test
cargo test -- --test-threads=1 # 默认多线程单元测试
cargo test -- --nocapture      # 将测试通过内容打印出来，类型go test -v
cargo test one_hundred         # one_hundred 为一个测试函数
cargo test add                 # add 可自动匹配含`add`的测试函数
cargo test -- --ignored        # 运行注解为忽略的测试函数
```

## 闭包

```rust
// 函数定义
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
// 闭包演化
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

### 推断为两个不同类型闭包时不可用(但申明时有效)

```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

### Fn trait / FnMut trait / FnOnce trait

- `FnOnce` 消费从周围作用域捕获的变量，闭包周围的作用域被称为其 **环境**，*environment*。为了消费捕获到的变量，闭包必须获取其所有权并在定义闭包时将其移动进闭包。其名称的 `Once` 部分代表了闭包不能多次获取相同变量的所有权的事实，所以它只能被调用一次。
- `FnMut` 获取可变的借用值所以可以改变其环境
- `Fn` 从其环境获取不可变的借用值

```rust
struct Cacher<T>
where
    T: Fn(u32) -> u32,
{
    // 定义存储闭包的属性
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
where
    T: Fn(u32) -> u32,
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            }
        }
    }
}
```

### 闭包捕获其环境

```rust
// 这里，即便 x 并不是 equal_to_x 的一个参数，equal_to_x 闭包也被允许使用变量 x，
// 因为它与 equal_to_x 定义于相同的作用域。
fn main() {
    let x = 4;
    let equal_to_x = |z| z == x;
    let y = 4;
    assert!(equal_to_x(y));
}
```

```rust
fn main() {
    let x = 4;
    // 这不是一个闭包，这是一个函数
    // 编译会出错
    fn equal_to_x(z: i32) -> bool { z == x }
    let y = 4;
    assert!(equal_to_x(y));
}
```

### 获得所有权

```rust
fn main() {
    let x = vec![1, 2, 3];
  	// move 关键字获得所有权
    let equal_to_x = move |z| z == x;
    // 去掉println!可编译通过
    println!("can't use x here: {:?}", x);
    let y = vec![1, 2, 3];
    assert!(equal_to_x(y));
}
```

### 迭代器

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
  	// 获得一个迭代器
    let v1_iter = v1.iter();
     // 在迭代器上调用 sum 求和方法
    let total: i32 = v1_iter.sum();
    assert_eq!(total, 6);
  
  let v1: Vec<i32> = vec![1, 2, 3];
  // v1.iter().map(|x| x + 1) 必须被消费所以使用 collect()
	let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
	assert_eq!(v2, vec![2, 3, 4]);
}
```

### 实现 Iterator trait

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}

#[test]
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new().zip(Counter::new().skip(1)) // [{1,2},{2,3}...{4,5}] // {5,None}
                                 .map(|(a, b)| a * b)         // [2,6...20]
                                   .filter(|x| x % 3 == 0)    // [6,12]
                                 .sum();											// 6+12
    assert_eq!(18, sum);
}
```

## 智能指针

> `String` 与 `Vec<T>` 都属于智能指针

- `Box<T>`，用于在堆上分配值
- `Rc<T>`，一个引用计数类型，其数据可以有多个所有者
- `Ref<T>` 和 `RefMut<T>`，通过 `RefCell<T>` 访问。（ `RefCell<T>` 是一个在运行时而不是在编译时执行借用规则的类型）。

### Box

- 当有一个在编译时未知大小的类型，而又想要在需要确切大小的上下文中使用这个类型值的时候
- 当有大量数据并希望在确保数据不被拷贝的情况下转移所有权的时候
- 当希望拥有一个值并只关心它的类型是否实现了特定 trait 而不是其具体类型的时候

### 不同的使用场景

- `Rc<T>` 允许相同数据有多个所有者；`Box<T>` 和 `RefCell<T>` 有单一所有者。
- `Box<T>` 允许在编译时执行不可变或可变借用检查；`Rc<T>`仅允许在编译时执行不可变借用检查；`RefCell<T>` 允许在运行时执行不可变或可变借用检查。
- 因为 `RefCell<T>` 允许在运行时执行可变借用检查，所以我们可以在即便 `RefCell<T>` 自身是不可变的情况下修改其内部的值。

### 必须使用递归Box的场景

```rust
use crate::List::{Cons, Nil};
enum List {
    Cons(i32, List),
    Nil,
}
fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
  	// 使用Box后，可
    //  let list = Cons(1,
    //    Box::new(Cons(2,
    //        Box::new(Cons(3,
    //            Box::new(Nil))))));
}
```
> 这个错误表明这个类型 “有无限的大小”。其原因是 `List` 的一个成员被定义为是递归的：它直接存放了另一个相同类型的值。这意味着 Rust 无法计算为了存放 `List` 值到底需要多少空间。
```bash
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^ recursive type has infinite size
2 |     Cons(i32, List),
  |               ----- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to
  make `List` representable
```

### Deref trait 实现自定义 `*` 解引用

- 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
- 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
- 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。<u>从可变引用到不可变引用</u>

```rust
use std::ops::Deref;
// 定义一个结构没有定义属性就存放一个slice
struct MyBox<T>(T);
impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
// 实现 Deref trait
impl<T> Deref for MyBox<T> {
  	// 语法定义了用于此 trait 的关联类型。
    type Target = T;
    fn deref(&self) -> &T {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);
		// 像 `*` 一个使用解引用
    assert_eq!(5, x);
    // 相当于 *(y.deref())
    assert_eq!(5, *y);
}
```

### 强制多态（递归解引用）

> 这里使用 `&m` 调用 `hello` 函数，其为 `MyBox<String>` 值的引用。因为示例 15-10 中在 `MyBox<T>` 上实现了 `Deref` trait，Rust 可以通过 `deref` 调用将 `&MyBox<String>` 变为 `&String`。标准库中提供了 `String` 上的 `Deref` 实现，其会返回字符串 slice，这可以在 `Deref` 的 API 文档中看到。Rust 再次调用 `deref` 将 `&String` 变为 `&str`，这就符合 `hello` 函数的定义了。

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}


```

> 如果 Rust 没有实现解引用强制多态，为了使用 `&MyBox<String>` 类型的值调用 `hello`，则不得不编写示例 15-13 中的代码来代替示例 15-12：

```rust
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

### Drop trait

> 后进先出（LIFO）,<u>并且不允许显示调用 drop ,因为在析构函数中会自动调用而导致 double free。</u>

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") }; // 3
    let d = CustomSmartPointer { data: String::from("other stuff") }; // 2
    println!("CustomSmartPointers created."); // 1
}
```

> 通过 `prelude` 的 `std::men::drop` ,提前丢弃值

```rust
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    drop(c); // 提前丢弃
    println!("CustomSmartPointer dropped before the end of main.");
}
```

### 计数引用  (*reference counting*)

> 无法被共享的 `Box<T>` 写法

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5,Box::new(Cons(10,Box::new(Nil))));
    let b = Cons(3, Box::new(a)); // a moved b
    let c = Cons(4, Box::new(a)); // error
}
```

> 使用 `Rc<T>` 解决

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a)); // Rc::clone 只是增加计数，区别于 a.clone() 深拷贝
    let c = Cons(4, Rc::clone(&a));
}
```

### `RefCell<T>` 内部可变性模式

> 对于引用和 `Box<T>`，借用规则的不可变性作用于<u>**编译时**</u>。对于 `RefCell<T>`，这些不可变性作用于 <u>**运行时**</u>。对于引用，如果违反这些规则，会得到一个编译错误。而对于 `RefCell<T>`，如果违反这些规则程序会 panic 并退出。

```rust
pub trait Messenger {
    fn send(&self, msg: &str);  // 定义中已经确定了 self 的不可变性
}

#[cfg(test)]
mod tests {
    use super::*;
    struct MockMessenger {
        // 因为 self 不具有可变性，所以该值无法被修改
        // sent_messages: Vec<String>,
      
      	// 使用 RefCell 
        sent_messages: RefCell<Vec<String>>,
    }
    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: vec![] }
        }
    }
    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            // 此处需要 self 具有可变性，才可以使用 push 方法
            // self.sent_messages.push(String::from(message));
          
            // 使用 RefCell 的 borrow_mut 方法获取 RefCell 中的可变引用
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }
    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

> 编译时不会有任何错误，不过测试会失败：<u>这会在相同作用域中创建两个可变引用，这是不允许的。</u>

```rust
impl Messenger for MockMessenger {
    fn send(&self, message: &str) {
        let mut one_borrow = self.sent_messages.borrow_mut();
        let mut two_borrow = self.sent_messages.borrow_mut();

        one_borrow.push(String::from(message));
        two_borrow.push(String::from(message));
    }
}
```

### `Rc<T>` 与 `RefCell<T>` 结合

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));
    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));
    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

### `weak<T>` 弱引用实例

+ 调用 `Rc::downgrade` 时会得到 `Weak<T>` 类型的智能指针。
+ 调用 `Weak<T>` 实例的 `upgrade` 方法，这会返回 `Option<Rc<T>>`

```rust
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}", // leaf strong = 1, weak = 0
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
		
  	// 进入作用域
    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}", // branch strong = 1, weak = 1
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}", // leaf strong = 2, weak = 0
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }
    // 离开作用域 leaf parent 弱引用被丢弃

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade()); // leaf parent = None
    println!(
        "leaf strong = {}, weak = {}", // leaf strong = 1, weak = 0
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}

```

## 并发与线程

### spawn 创建新线程

```rust
use std::thread;
use std::time::Duration;
fn main() {
  	// 传入闭包函数
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });
    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
  	// 等待线程结束
  	handle.join().unwrap();
}
```

### move 闭包获得所有权

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

### 消息传递

> `mpsc` 是 **多个生产者，单个消费者**（*multiple producer, single consumer*）的缩写

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
  	// 创建通道
    let (tx, rx) = mpsc::channel();
		// 获得发送端所有权
    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
		// 阻塞式接收数据（recv） 
  	// 非阻塞式接收数据 (try_recv)
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

### 多个生产者

```rust
let (tx, rx) = mpsc::channel();
let tx1 = mpsc::Sender::clone(&tx); // 克隆一个发送端
```

### 共享状态 `Mutex<T>`

```rust
use std::sync::Mutex;
fn main() {
    let m = Mutex::new(5);
    {
        // 如果另一个线程拥有锁，并且那个线程 panic 了，
        // 则 lock 调用会失败。在这种情况下，没人能够再获取锁，
        // 所以这里选择 unwrap 并在遇到这种情况时使线程 panic。
        let mut num = m.lock().unwrap();
        *num = 6;
    }
    println!("m = {:?}", m);
}
```

### 多线程与多所有权 `Arc`

> `Rc<T>` 并不能安全的在线程间共享。所幸 `Arc<T>` **正是** 这么一个类似 `Rc<T>` 并可以安全的用于并发环境的类型。字母 “a” 代表 **原子性**（*atomic*）

```rust
//use std::rc::Rc;
//use std::sync::Mutex;
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    // 将互斥锁放入计数引用中
    // let counter = Rc::new(Mutex::new(0));
  	let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    for _ in 0..10 {
      	// 添加计数
        // counter 隐藏了之前的 counter
        // let counter = Rc::clone(&counter);
      	let counter = Arc::clone(&counter);
        // counter 的所有权之一被移入 thread
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

### `RefCell`/`Rc` 与 `Mutex`/`Arc` 的相似性

> 你可能注意到了，因为 `counter` 是不可变的，不过可以获取其内部值的可变引用；这意味着 `Mutex<T>` 提供了内部可变性，就像 `Cell` 系列类型那样。正如第十五章中使用 `RefCell<T>` 可以改变 `Rc<T>` 中的内容那样，同样的可以使用 `Mutex<T>` 来改变 `Arc<T>` 中的内容。

## 面向对象 ？？？？

> https://kaisery.github.io/trpl-zh-cn/ch17-00-oop.html

## 模式匹配

### match

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y), // 引入了覆盖变量 y
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

#### 多个模式

```rust
let x = 1;
match x {
    1 | 2 => println!("one or two"), // 多个模式
    3 => println!("three"),
    _ => println!("anything"),
}
```

#### 范围匹配 `..=`

```rust
let x = 5;

match x {
    1..=5 => println!("one through five"), // 范围匹配
    _ => println!("something else"),
}

let x = 'c';
match x {
    'a'..='j' => println!("early ASCII letter"), // char 和 数字值是 Rust 仅有的可以判断范围是否为空的类型。
    'k'..='z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```

#### 匹配守卫

> **匹配守卫**（*match guard*）是一个指定于 `match` 分支模式之后的额外 `if` 条件，

```rust
fn main() {
    let x = Some(5);
    let y = 10;
    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {}", n),  // 一种使用方式，额外条件
        _ => println!("Default case, x = {:?}", x),
    }
    println!("at the end: x = {:?}, y = {}", x, y);
}

// 混合使用
let x = 4;
let y = false;
match x {
    4 | 5 | 6 if y => println!("yes"), // 守卫 y 值
    _ => println!("no"),
}
```

#### 绑定并测试 `@`

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
  	// 这里我们希望测试 Message::Hello 的 id 字段是否位于 3...7 范围内，
    // 同时也希望能将其值绑定到 id_variable 变量中以便此分支相关联的代码可以使用它。
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```



### if let 

```rust
let age: Result<u8, _> = "34".parse();
// 引用了新的覆盖变量 (age) ,不能与if let 组合，只能在代码块中
// age 是一个 Result 对象
if let Ok(age) = age {
		if age > 30 {
      
  	}   
}

```

### while let

```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);
// stack.pop() 最终返回的是一个 None
while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

### for

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

### 函数中的模式

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

### Refutability `可反驳性`

> 模式有两种形式：refutable（可反驳的）和 irrefutable（不可反驳的）。能匹配任何传递的可能值的模式被称为是 **不可反驳的**（*irrefutable*）。一个例子就是 `let x = 5;` 语句中的 `x`，因为 `x` 可以匹配任何值所以不可能会失败。对某些可能的值进行匹配会失败的模式被称为是 **可反驳的**（*refutable*）。一个这样的例子便是 `if let Some(x) = a_value` 表达式中的 `Some(x)`；如果变量 `a_value` 中的值是 `None` 而不是 `Some`，那么 `Some(x)` 模式不能匹配。

```rust
// -- snipt --
let Some(x) = some_option_value; // 报错，let只接受 irrefutable 类型

// -- snipt --
// 警告: if 在处理一个 refutable 类型
if let x = 5 {
    println!("{}", x);
};
```

### 解构分解值

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };
    let Point { x: a, y: b } = p; // 解构出 a,b 值
    // let Point { x, y } = p;    // 简写方式
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

### 解构枚举

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        }
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```

#### 解构嵌套

```rust
enum Color {
   Rgb(i32, i32, i32),
   Hsv(i32, i32, i32),
}
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color), // 嵌套
}
fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change the color to red {}, green {}, and blue {}",r,g,b)
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => { // 解构嵌套
            println!("Change the color to hue {}, saturation {}, and value {}",h,s,v)
        }
        _ => ()
    }
}
```

### 忽略值 `_`

> 虽然 `_` 模式作为 `match` 表达式最后的分支特别有用，也可以将其用于任意模式，包括函数参数中

```rust

// 函数中使用
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}
fn main() {
    foo(3, 4);
}

// 配合 Some 使用，匹配却不使用值
let mut setting_value = Some(5);
let new_setting_value = Some(10);
match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}

println!("setting is {:?}", setting_value);
```

### 通过在名字前以一个下划线开头来忽略未使用的变量（但会获得所有权）

```rust
fn main() {
    let _x = 5; // 忽略未使用警告
    let y = 10;
}
```

###  `..` 忽略剩余值

```rust
// 解构忽略
struct Point {
    x: i32,
    y: i32,
    z: i32,
}
let origin = Point { x: 0, y: 0, z: 0 };
match origin {
    Point { x, .. } => println!("x is {}", x),
}

// 忽略区间
fn main() {
    let numbers = (2, 4, 8, 16, 32);
    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }
}
```

## 高级特征

### unsafe

#### 裸指针

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

#### 函数与方法

```rust
unsafe fn dangerous() {} // 不安全函数
unsafe {
    dangerous(); // 在不安全代码块中调用，否则非法
}
```

#### 代码块中使用 unsafe

```rust
// 编译器无法知道 slice[..mid] 与 slice[mid..] 不重叠，所以需要人为干涉成 unsafe
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    assert!(mid <= len);
    (&mut slice[..mid],
     &mut slice[mid..])
}

// unsafe 解决
use std::slice;
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();
    assert!(mid <= len);
    unsafe {
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.add(mid), len - mid))
    }
}
```

### 全局变量 `static`

> 读取或修改一个可变静态变量是不安全的

```rust
static mut COUNTER: u32 = 0;
fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}
fn main() {
    add_to_count(3);
    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

### Trait

> 在 Trait 中使用 type item 提供 `关联类型占位符 ` , 解决要在 Trait 中实现类似泛型的概念。通过关联类型，则无需标注类型因为不能多次实现这个 trait。

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

#### 默认参数类型

> 尖括号中的 `RHS=Self`：这个语法叫做 **默认类型参数**（*default type parameters*）。`RHS` 是一个泛型类型参数（“right hand side” 的缩写），它用于定义 `add` 方法中的 `rhs` 参数。

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}
```

##### 默认定义

```rust
// 大部分情况，都是使用相同类型相加
trait Add<RHS=Self> {
    type Output;
    fn add(self, rhs: RHS) -> Self::Output;
}
```

##### 不使用默认定义

```rust
use std::ops::Add;
struct Millimeters(u32);
struct Meters(u32);
// 此处使用了非默认类型
impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

#### 调用相同名称的方法

##### 含 `self` 传参

```rust
trait Pilot {
    fn fly(&self);
}
trait Wizard {
    fn fly(&self);
}
struct Human;
impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}
impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}
impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}

fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

##### 完全限定 

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

##### trait 中使用某个 trait 功能

```rust
use std::fmt;
// OutlinePrint 依赖 Display 的实现。才可以使用 to_string();
trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
struct Point {
    x: i32,
    y: i32,
}
impl OutlinePrint for Point {}
// -- 以上实现依然编译出错，除非实现了以下代码 -- //
use std::fmt;
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

##### newtype 定义

```rust
use std::fmt;

// 这是一个newtype定义，类似golang，因为是新的 type 所以可以对其原有的Display trait重定义
// 注意 newtype 的类型继承
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

```rust
// 定义一个类型别名
type Kilometers = i32;
let x: i32 = 5;
let y: Kilometers = 5;
// 与 golang 不同 原类型相同，所以可以相加。
println!("x + y = {}", x + y);

```

```rust
use std::io::Error;
use std::fmt;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
// -- 优化过后 -- //
type Result<T> = std::result::Result<T, std::io::Error>;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>; // 此处少写了Error，类似C的宏
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: Arguments) -> Result<()>;
}
```

### never type `!`

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue, // never type 可以强转为任何其他类型,因为 continue 并不真正返回一个值
};
```

### 返回闭包

> 闭包表现为 trait，这意味着不能直接返回闭包。
>
> Rust 并不知道需要多少空间来储存闭包。不过我们在上一部分见过这种情况的解决办法：可以使用 trait 对象

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
  	// 因为 RUST 不知道大小所以要放到堆上
    // |x| x + 1
    Box::new(|x| x + 1)
}
```

## 宏 

### macro_rules! 的声明宠用于通用元编程

```rust
#[macro_export]     // #[macro_export] 注解说明宏应该是可用的。 如果没有该注解，这个宏不能被引入作用域。
macro_rules! vec {  // 使用 macro_rules! 和宏名称开始宏定义，且所定义的宏并 不带 感叹号
    // 首先，一对括号包含了全部模式。
    // 接下来是后跟一对括号的美元符号（ $ ），其通过替代代码捕获了符合括号内模式的值。
    // $() 内则是 $x:expr ，其匹配 Rust 的任意表达式或给定 $x 名字的表达式。
    // $() 之后的逗号说明一个逗号分隔符可以有选择的出现代码之后，这段代码与在 $() 中所捕获的代码相匹配。
    // 紧随逗号之后的 * 说明该模式匹配零个或多个 * 之前的任何模式。
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

```rust
// -- 调用 -- //
vec![1, 2, 3];

// -- 展开 -- //
let mut temp_vec = Vec::new();
temp_vec.push(1);
temp_vec.push(2);
temp_vec.push(3);
temp_vec
```

### derive 宏

> `#[derive(HelloMacro)]` 注解他们的类型来得到 `hello_macro` 函数的默认实现。

#### 调用方

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
    // Hello, Macro! My name is Pancakes!
}
```

#### 实现方

```rust
// cargo new hello_macro --lib
// src/lib.rs
pub trait HelloMacro {
    fn hello_macro();
}
// src/main.rs
use hello_macro::HelloMacro;

struct Pancakes;

impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!("Hello, Macro! My name is Pancakes!");
    }
}

fn main() {
    Pancakes::hello_macro();
}

// cargo new hello_macro_derive --lib
// hello_macro_derive/Cargo.toml
[lib]
proc-macro = true // 定义过程宏？

[dependencies]
syn = "0.14.4"
quote = "0.6.3"

// hello_macro_derive/src/lib.rs
extern crate proc_macro; // 1.31.0 必须

use crate::proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // 构建 Rust 代码所代表的语法树
    // 以便可以进行操作
    let ast = syn::parse(input).unwrap();

    // 构建 trait 实现
    impl_hello_macro(&ast)
}
```

