---
title: 详解闭包:Rust中的函数式编程
date: 2023-04-02 21:05:39
tags:
    - Rust
    - 开发
    - 函数式
    - 教程
categories:
    - Rust
    - 开发
---


全文长度：2300词
阅读时长：约15分钟


本节参考：
- [Rust语言圣经](https://course.rs/advance/functional-programing/closure.html)
- [RF-闭包表达式](https://rustwiki.org/zh-CN/reference/expressions/closure-expr.html)
- [RF-闭包类型](https://rustwiki.org/zh-CN/reference/types/closure.html)
- [Rust中的闭包与关键字Move](https://zhuanlan.zhihu.com/p/341815515)

Rust中函数式编程的大梁由四大天王顶起：
- 模式匹配
- 枚举
- 迭代器
- 闭包

使用 **闭包（Closure）** 可以做到将一系列语句和表达式赋值给变量，因此也可以将语句和表达式作为参数传递，将语句和表达式作为函数返回值返回，它具有如此一系列神奇的特性。闭包的使用很简单，但其中一些细节需要仔细推敲。
下面，我们从闭包如何捕获环境，闭包如何使用捕获值，以及闭包实现的角度，来介绍这个编程利器。


## 开始——捕获环境
---
Rust中的函数是无法捕获其所在环境的。对于以下代码：
```rust
fn main() {
	let num = 0;
	println!("{num}");
	fn func() {
		println!("{num}");
	}
}
```
即使该函数定义在 `main()` 内部，它仍然无法访问到自己被定义的环境中定义的变量。若要访问这些变量，只能通过传递函数参数的方式。需要注意，`static` 和 `const` 这样的量具有静态生命周期，是可以访问的。
而使用闭包，便可以起到捕获环境的作用。
那么如何定义一个闭包呢？Rust中通过闭包表达式定义一个闭包类型，在其他语言中也称为 **lambda表达式**。

闭包表达式的**句法规则**是：可选的 `move` ，后跟由 `||` 围住的参数模式列表（可以省略类型标注），后跟可选的返回值标注 `-> type` ，后跟一个块表达式（无返回值标注时，若块内只有一个表达式则可以直接写在 `||` 后）。例如：
```rust
fn main() {  
	// 函数
    fn func(a: i32, b: i32) -> i32 {  
        a + b  
    }  
    
    // 闭包定义1
    let func = |a: i32, b: i32| -> i32 {  
        a + b  
    };
    // 闭包定义2
    let func = |a, b| {  
        a + b  
    };  
    // 闭包定义3
    let func = |a, b| a + b;  
    // 闭包定义4
    let func = move |a, b| a + b;  
  
    // 相同的调用方式 
    let res = func(1, 2);  
    assert_eq!(res, 3);  
}
```


## 闭包捕获环境的方式 
---
闭包是可以捕获环境的，捕获的方式有这几种（不捕获环境的闭包见后文）：
- 不可变引用 `&T`
- 可变引用 `&mut T`
- 移动语义（获取所有权） `T`
当在 `||` 前使用 `move` 时，将强制闭包以移动语义（move）捕获值，获取值的所有权。对于实现了 `Copy Trait` 的类型，则使用 `Copy` **复制语义**。当没有使用 `move` 时，编译器会按照如下顺序进行检查，选择捕获方式，直到遇到第一个能通过编译的选项：
1. 不可变引用
2. 唯一不可变引用
3. 可变引用
4. 移动语义
此处，**唯一不可变引用** 是基于借用规则而出现的一种特殊的捕获方式。对于下述代码：
```rust
let mut a = 0;  
let b = &mut a;  
{  
    let mut x = || { *b = 1; };  
    // 下行代码不正确  
    // let y = &b;  
    x();  
    println!("{y}");   // 由于NLL，这里需要使用 y
}  
let z = &b;
```
代码中 `b` 是对 `a` 的可变借用，因此可以通过解引用 `b` 来修改 `a` 的值。但在这里我们将修改的操作放在一个闭包中，其中使用了 `b`，因此闭包需要捕获它。由于 `b` 本身不是 `mut` 的，因此无法以可变引用的形式捕获。但若以不可变引用的形式捕获，那么就会获得对可变引用的引用 `& &mut`，它将不是唯一的，这违反了借用规则。
这时，闭包便使用**唯一不可变引用**的方式来捕获变量，即它会对 `b` 进行不可变引用，同时会确保对 `b` 的引用只有一个。


## 3种闭包Trait 
---
这里需要做一区别，闭包**如何捕获环境**，和闭包**如何使用捕获到的值**，两者是不同的。

Rust编译器会根据闭包 **如何使用** 捕获到的值，来决定为闭包实现哪些**闭包Trait**。
或者说，编译器通过这3种Trait来描述和分类不同的闭包：
- `FnOnce` ：闭包可能会消耗掉捕获值的所有权，表示闭包至少能使用一次，因此所有的闭包均实现了该Trait。
- `FnMut` ：闭包不会消耗掉捕获值的所有权，同时会对捕获值进行修改。
- `Fn` ：闭包不会消耗掉捕获值的所有权，同时不会对捕获值进行修改。
所有闭包都 **至少** 实现了 `FnOnce`。

所有类型的闭包中，有些闭包可能会消耗掉捕获值的所有权，这种闭包在调用一次后无法再次调用（要处理的值已经不见了），因此对于所有的闭包来说，闭包最少是可以使用一次的，使用 `FnOnce` 描述。如果闭包并不消耗掉捕获值的所有权，便可以多次被调用，它对捕获值的操作，只可能是修改或者不修改，前者使用 `FnMut` 描述，后者使用 `Fn` 描述。

因此可以说， **3种闭包Trait，是在闭包如何使用捕获值的角度上，对闭包的分类。**

现在观察这3种Trait的定义签名（简化）：
```rust
pub trait Fn<Args> : FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
}
pub trait FnMut<Args> : FnOnce<Args> {
    extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
}
pub trait FnOnce<Args> {
    type Output;
    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
}
```
可以看到，实现 `FnMut` 的条件是，已经实现了 `FnOnce`，而实现 `Fn` 的条件是已经实现了 `FnMut`，因此，闭包对这3种Trait的实现有这三种情况：
1. 只实现了 `FnOnce`
2. 实现了 `FnOnce` 和 `FnMut`
3. 实现了 `FnOnce` ，`FnMut` 和 `Fn`
分别对应上述三种Trait的情况。


## 函数式编程：作为参数和返回值
---
由于Rust中的闭包实现了上文介绍的几种闭包特征，因此可以使用特征约束的方法让闭包作为函数参数或返回值来使用，例如：
```rust
// 接收一个 FnOnce() 类型的闭包并调用
fn function<F> (f: F) 
where F: FnOnce() {
	f();
}

// 返回一个 FnOnce() -> &'static str 类型的闭包
fn some_func() -> impl FnOnce() -> &'static str {  
    || { "666" }  
}
// 返回一个特征对象，不常用
fn dyn_func() -> Box<dyn FnOnce() -> &'static str> {
	Box::new(|| { "999" })  
}
```
对于函数而言，只要符合特征约束，也可以作为其他函数的参数：
```rust
// 将要接收函数和闭包作为参数的函数
fn call_me<F: Fn()>(f: F) {
    f()
}
// 一个函数
fn function() {
    println!("I'm a function!");
}
fn main() {
	// 一个闭包
    let closure = || println!("I'm a closure!");
    
    call_me(closure);
    call_me(function);
}
```



## 闭包的实际类型 
---
当使用闭包表达式定义一个闭包时，编译器会隐式生成一个匿名结构体，结构体中的字段会存储闭包捕获的变量。同时，会为该结构体实现闭包特征，并由此实现闭包的函数功能。
例如，对于以下闭包：
```rust
fn closure<F> (f: F)  
where F: FnOnce() -> &'static str  
{  
    println!("closure: {}", f());  
}  
  
fn main() {  
    let s = || { "Hello" };  
    closure(s);  
}
```
编译器会大致生成如下的代码：
```rust
struct ClosureSome {
	a: &'static str,
}

impl FnOnce() for ClosureSome {
	type Output = &'static str;
	fn call_once(self) -> &'static str {
		"Hello"
	}
}
```
因此每个闭包都具有自己独特的类型，且无法被写出。
由此可以看出，当传递一个闭包时，**传递的实际上是一个结构体**，而调用一个闭包时，则是调用相应Trait定义的方法。

上文中介绍了编译器根据闭包如何使用捕获到的值而实现不同的闭包特征，而对于 **闭包没有捕获值** 的情况，该闭包可以被 **自动强转** 为函数指针：
```rust
fn main() {
	let add = |x, y| x + y;
	let mut x = add(5,7);

	type Binop = fn(i32, i32) -> i32;
	let bo: Binop = add;
	
	x = bo(5,7);
}
```


## 总结
---
Rust中的闭包可以实现一些函数式编程的功能，它与函数类似，但也不同，主要便在于**闭包可以捕获环境**。

闭包 **捕获环境** 的方式分为三种，即 `&T` `&mut T` 和 `T`，当闭包不捕获环境时，可以被自动强转为函数指针。
闭包 **使用捕获值** 的方式也分为三种，即消耗所有权，不消耗所有权并进行修改，不消耗所有权且不修改。与此对应的，有三种闭包特征，即 `FnOnce`， `FnMut` 和 `Fn`，实现了后一个特征则肯定实现了前一个特征，如一个闭包实现了 `Fn`，它肯定实现了 `FnMut` 和 `FnOnce`。

通过使用特征约束，利用3种Trait，可以将闭包作为参数传递，或作为返回值返回。
最后，闭包实现这样一系列功能，它的真实类型便是一个编译器自动生成的匿名结构体，结构体的字段存储着闭包捕获的环境，编译器为它实现相应的Trait，并将闭包包含的语句和表达式作为具体的实现。