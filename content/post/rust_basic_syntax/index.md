---
title: rust基本语法特性
description: 这是一个副标题
date: 2020-11-19 00:04:32 +0800
slug: rust
image: matt-le-unsplash.jpg
categories:
    - 代码
tags:
    - rust
---
 ### println！宏

 println！宏中的格式化形式列表如下：

*   nothing代表Display，比如println！("{}",2)。

*   ？代表Debug，比如println！("{:?}",2)。

*   o代表八进制，比如println！("{: o}",2)）。

*   x代表十六进制小写，比如println！("{:x}",2)。

*   X代表十六进制大写，比如println！("{:X}",2)。

*   p代表指针，比如println！("{:p}",2)。

*   b代表二进制，比如println！("{:b}",2)。

*   e代表指数小写，比如println！("{:e}",2)。

*   E代表指数大写，比如println！("{:E}",2)。


### const_fn
 CTEF机制（Compile-Time Function Execution）

 在编译时函数执行

 2018版本不需要加#![feature(const_fn)]

 使用const fn定义的函数必须可以确定值，不能存在歧义。

 与fn的区别在于，const fn可以强制编译器在编译期执行函数，其中const一般用于定义全局变量。
```
 #![feature(const_fn)]
 const fn init_len()->i32{
     return 5;
 }

 fn main(){
     let arr = [0,init_len()];
 }
```


### never、闭包

 never类型表示永远不可能有返回值的计算类型，比如线程退出的时候，就不可能有返回值。

 使用＃！[feature（never_type）]特性，这是因为当前never类型属于实验特性，所以必须在Nightly版本下使用该特性，才可以显式地使用never类型。

 never类型可以强制转换为其它任何类型

 闭包：

*   可以像匿名函数一样被调用

*   可以捕获上下文环境中的自由变量

*   可以自动推断输入和返回类型

 闭包实际上就是一个匿名结构体和trait来组合实现的，闭包同样也可以作为返回值

<span class='redtext'> 注意：闭包默认按引用捕获变量，如果将闭包作为返回值，则引用也会跟着返回，但是在整个函数调用完毕之后函数内的本地变量会被销毁。</span>

           随闭包返回的本地变量会成为悬垂指针，所以使用move关键字将捕获本地变量的所有权转移到闭包中，就不会按引用捕获变量，使闭包安全返回。

```
 fn main(){
     let out =42;
     fn add( i:i32, j:i32)->i32{ i + j + out}
     let closure_annotated = |i: i32, j: i32|  -> i32{ i + j + out};
     let out =42;
     let closure_inferred = |i, j |  i + j + out;
     //闭包作为参数
     let a = 2;
     let b = 3;
     let res = math(|| a + b);
     //闭包作为返回值
     let result = two_times_impl();

 }

 //使用impl Fn(i32)-> i32作为函数的返回值
 fn two_times_impl() -> impl Fn(i32) -> i32 {
     let i = 2;
     move | j | j * i;
 }

 //参数是一个泛型F，并且该泛型受Fn() -> i32 trait的限定
 fn math<F: Fn() -> i32>( op : F) ->i32{
     op()
 }

```


*   如果闭包中没有捕获任何环境变量，则默认自动实现Fn。

*   如果闭包中捕获了复制语义类型的环境变量，则：
	*   如果不需要修改环境变量，无论是否使用move关键字，均会自动实现Fn。
	*   如果需要修改环境变量，则自动实现FnMut。

*   如果闭包中捕获了移动语义类型的环境变量，则:

	*   如果不需要修改环境变量，且没有使用move关键字，则自动实现FnOnce。

	*   如果不需要修改环境变量，且使用了move关键字，则自动实现FnOnce。

	*   如果需要修改环境变量，则自动实现FnMut。

*   对于FnMut的闭包使用move关键字，如果捕获的变量是复制语义类型的，则闭包会自动实现Copy/Clone，否则不会实现Copy/Clone。

 ### 复合数据类型

 4种复合数据类型，分别是：

*   元组（Tuple） 

*   结构体（Struct）

	*   具名结构体（Named-Field Struct）

	*   元组结构体（Tuple-Like Struct）

	*   单元结构体（Unit-Like Struct）

*   枚举体（Enum）

*   联合体（Union）

 元祖：元素可以不同类型，固定长度

```
 let tuple : (&'static str,i32,char) = ("hello",5,'c');

 assert_eq!(tuple.0,"hello");

 assert_eq!(tuple.1,5);

 assert_eq!(tuple.2,'c');
```


 结构体：

 结构体遵循驼峰命名规则

 结构体中字段默认不可变

 字段可以是任意类型甚至结构体本身
```
#[derive(Debug,PartialEq)]  //可以 让结构体自动实现Debug trait和PartialEq trait，它们的功能是允许对结构 体实例进行打印和比较。

 struct People{
     name: &'static str,
     gender: u32
 }

 impl People{

     fn new(name: &'static str,gender:u32)->Self{
         return People{name : name,gender:gender};
     }

     fn set_name(&mut self,name : &'static str)->(){
         self.name = name;
     }

     fn get_name(&self) -> String{
         return self.name.to_string();
     }
 }

 fn main() {
     let mut  people = People::new("aa", 2);
     println!("{}",people.get_name());
     people.set_name("bb");
     println!("{}",people.get_name());
 }
```
 

 //单元结构体

 //x y z在 Release 编译模式下，单元结构体实例会被优化为同一 个对象。而在 Debug模式下，则不会进行这样的优化。

```
 struct Empty;

 fn main() {
     let x = Empty;
     println!("{:p}",&x);
     let y = x;
     println!("{:p}",&y);
     let z = Empty;
     println!("{:p}",&z);
     assert_eq!((..),std::ops::RangeFull);

 }
```



 · 线性序列：向量（Vec）、双端队列（VecDeque）、链表（LinkedList）。

 · Key-Value映射表：无序哈希表（HashMap）、有序哈希表（BTreeMap）。

 · 集合类型：无序集合（HashSet）、有序集合（BTreeSet）。

 · 优先队列：二叉堆（BinaryHeap）。

 双端队列VecDeque实现了两种push方法，push_front和push_back。

 push_front的行为像栈，push_back的行为像队列。通过get方法加索引值

 可以获取队列中相应的值。

 使用Vec或VecDeque类型比链表更加快速，内存访问效率更高，并且可以更好地利用CPU缓存。

### 泛型和trait

```
 fn matct_option<T : Debug>(op : Option<T>)->(){
     match op {
         Some(o) => println!("{:?}",o),
         None =>println!("没有值")
     }
 }

 fn main() {
     let a = Some(1);
     matct_option(a);
     let b : Option<u8> = None;
     matct_option(b);
 }
```

 fly_static这种调用方式在 Rust 中叫静态分发。Rust 编译器会为fly_static::<Pig>(pig)和fly_static::<Duck>(duck)这两个具

 体类型的调用生成特殊化的代码。也就是说，对于编译器来说，这种抽象并不存在，因为在编译阶段，泛型已经被展开为具体类型的代码。

 fly_dyn(&Duck)，也可以实现同样的效果。但是 fly_dyn 函数是动态分发方式的，它会在运行时查找相应类型的方法，会带来一定的运行时开销，不过这种开销很小。

```
 trait Fly{
     fn is_fly(&self)-> bool;
 }

 struct Duck;

 struct Pig;

 impl Fly for Duck{

     fn is_fly(&self) -> bool {
         println!("鸭子会飞");
         return true;
     }
 }

 impl Fly for Pig{

     fn is_fly(&self) -> bool {
         println!("猪不会飞");
         return false;
     }

 }

 fn fly_static<T :Fly>(f : T)-> String{
     if f.is_fly() { return "飞起来了".to_string() }else {return  "飞不动".to_string() };
 }

 fn fly_dyn(f : &Fly)->String{ if f.is_fly() { return "飞起来了".to_string() }else { return "飞不动".to_string() };}

 fn matct_option<T : Debug>(op : Option<T>)->(){
     match op {
         Some(o) => println!("{:?}",o),
         None =>println!("没有值")
     }
 }

 fn main() {
     let res = fly_static::<Pig>(Pig);
     println!("{}",res);
     let res = fly_static::<Duck>(Duck);
     println!("{}",res);
     let res = fly_dyn(&Pig);
     println!("{}",res);

 }
```


 Rust一共提供了5个重要的标签trait，都被定义在标准库std：：

 marker模块中。它们分别是：

*   Sized trait，用来标识编译期可确定大小的类型。

*   Unsize trait，目前该trait为实验特性，用于标识动态大小类型(DST)。

*   Copy trait，用来标识可以按位复制其值的类型。

*   Send trait，用来标识可以跨线程安全通信的类型。

*   Sync trait，用来标识可以在线程间安全共享引用的类型。

 除此之外，Rust标准库还在增加新的标签trait以满足变化的需求。

 Sized trait 非常重要，编译器用它来识别可以在编译期确定大小的

 类型。

*** ＃[fundamental] 属性，代表trait不受孤儿规则的限制。

 通过＃[derive（Debug，Copy，Clone）]属性实现Copy，使结构体可以按位复制。如果结构体中 还有移动语义类型的成员，则无法实现Copy。因为按位复制可能会引发 内存安全问题。***

### 引用与借用 

 引用（Reference）是 Rust 提供的一种指针语义。引用是基于指针 的实现，它与指针的区别是，指针保存的是其指向内存的地址，而引用 可以看作某块内存的别名（Alias），使用它需要满足编译器的各种安 全检查规则。引用也分为不可变引用和可变引用。使用&符号进行不可 变引用，使用&mut符号进行可变引用。 在所有权系统中，引用&x也可称为x的借用（Borrowing），通过 &操作符来完成所有权租借。既然是借用所有权，那么引用并不会造成 绑定变量所有权的转移。但是借用所有权会让所有者（owner）受到如 下限制： 

  · 在不可变借用期间，所有者不能修改资源，并且也不能再进行可 变借用。

  · 在可变借用期间，所有者不能访问资源，并且也不能再出借所有 权。

 引用在离开作用域之时，就是其归还所有权之时。使用借用，与 直接使用拥有所有权的值一样自然，而且还不需要转移所有权。

 借用规则 为了保证内存安全，借用必须遵循以下三个规则。 

 · 规则一：借用的生命周期不能长于出借方（拥有所有权的对象） 的生命周期。 

 · 规则二：可变借用（引用）不能有别名（Alias），因为可变借用 具有独占性。

 · 规则三：不可变借用（引用）不能再次出借为可变借用。 

     规则一是为了防止出现悬垂指针。

 规则二和三可以总结为一条核心 的原则：共享不可变，可变不共享。Rust编译器会做严格的借用检查， 违反以上规则的行为均无法正常通过编译。 

 规则一很好理解。如果出借方已经被析构了，但借用依然存在，就 会产生一个悬垂指针，这是Rust绝对不允许出现的情况。 

 规则二和规则三描述的不可变借用和可变借用就相当于内存的读写 锁，同一时刻，只能拥有一个写锁，或者多个读锁，不能同时拥有，

 Cell<T> ： 给实现了Copy类型T提供get和set方法

 RefCell<T>: 对于没有实现Copy类型，提供borrow/borrow_mut方法对应Cell的get/set方法，维护一个运行时借用检查器，

 如果再运行时出现了违反借用规则的情况，比如持有多个可变借用，就会引发线程panic而退出当前线程

 Cow<T>：写时复制（Copy on write）

*   Borrowed 用于包裹引用

*   Owned 用于包裹所有者

 to_mut方法获取可变借用，该方法产生克隆，但仅克隆一次，如果调用多次，则只会使用第一次的克隆对象。如果T本身

 有所有权，则不会发生克隆

 into_owned获取一个拥有所有权的对象。如果T是借用类型，会发生克隆并创建新的所有权对象。如果T是所有权对象，则

 会将所有权转移到新的克隆对象。

 ### 字符串常用方法

*   is_digit（16），用于判断给定字符是否属于十六进制形式。如果 参数为10，则判断是否为十进制形式。

*   to_digit（16），用于将给定字符转换为十六进制形式。如果参数 为10，则将给定字符转换为十进制形式。

*   is_lowercase，用于判断给定字符是否为小写的。作用于 Unicode 字符集中具有Lowercase属性的字符。

*   is_uppercase，用于判断给定字符是否为大写的。作用于 Unicode 字符集中具有Uppercase属性的字符。

*   to_lowercase，用于将给定字符转换为小写的。作用于Unicode字 符集中具有Lowercase属性的字符。

*   to_uppercase，用于将给定字符转换为大写的。作用于Unicode字 符集中具有Uppercase属性的字符。

*   is_whitespace，用于判断给定字符（或十六进制形式的码点）是否 为空格字符。

*   is_alphabetic，用于判断给定字符是否为字母。汉字也算是字母。

*   is_alphanumeric，用于判断给定字符是否为字母、数字。 · is_control，用于判断给定字符是否为控制符。

*   is_numeric，用于判断给定字符是否为数字。

*   escape_default，用于转义\t、\r、\n、单引号、双引号、反斜杠等 特殊符号。