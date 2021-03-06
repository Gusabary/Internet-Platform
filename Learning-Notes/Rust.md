# Rust learning notes

## The Book Wrap-up

### Basic Concept

+ `expect` and `unwrap` method are defined on `Result` 

+ tuple type: `let tup = (500, 6.4, 1);`

  array type: `let a = [1, 2, 3, 4, 5];`

  array type with explicitly declared size: `let a: [i32; 5] = [1, 2, 3, 4, 5];`

  array type with initialization to same value: `let a = [3; 5];`

+ The block that's used to create a new scope is an expression, so we can write like this:

  ```rust
  let x = {
      let y = 1;
      y + 1
  };
  ```

+ `if` is also an expression:

  ```rust
  let number = if condition { 5 } else { 6 };
  ```

  note that `if` expression without `else` branch evaluates to `()`

+ `loop` is also an expression, whose value can be specified after `break`:

  ```rust
  let mut counter = 0;
  let result = loop {
      counter += 1;
      if counter == 10 {
          break counter * 2;
      }
  };
  ```

### Ownership

+ reference and slice don't take ownership

+ the type of variable that is initialized with string literals is `&str`, because it is essentially a reference to bytes stored in binary.

  string slices' type is `&str` while i32 slices' type is `&[i32]`.

+ string slice can be got from `[..]` on `String` or `str` while i32 slice can be got from `[..]` on `Vec<i32>` or `[i32]`.

### Struct

+ struct type variable can be initialized with a JSON-like format:

  ```rust
  let mut user1 = User {
      email: String::from("someone@example.com"),
      username: String::from("someusername123"),
      active: true,
      sign_in_count: 1,
  };
  ```

  if the filed name and variable name to fill that field are the same, we could write just once:

  ```rust
  User {
      email,
      username,
      active: true,
      sign_in_count: 1,
  }
  ```

  just like JavaScript, hah

+ struct update syntax:

  ```rust
  let user2 = User {
      email: String::from("another@example.com"),
      username: String::from("anotherusername567"),
      ..user1
  };
  ```

+ tuple struct is kinda like tuple, but they have different type even if types inside the tuple are all the same:

  ```rust
  struct Color(i32, i32, i32);
  struct Point(i32, i32, i32);
  
  let black = Color(0, 0, 0);
  let origin = Point(0, 0, 0);
  ```

+ add `#[derive(Debug)]` annotation to a struct to make it able to pretty-print with `{:?}` and `{:#?}`.

+ define methods inside `impl` block:

  ```rust
  impl Rectangle {
      fn area(&self) -> u32 {
          self.width * self.height
      }
  }
  ```

  actually the first parameter here `&self` can also be `self`, `mut self` or `&mut self`, and it doesn't need to specify the type explicitly because compiler knows the type is struct type.

+ when we invoke methods, we write like `p1.distance(&p2);` directly instead of `(&p1).distance(&p2);` or `->` like cpp, because rust supports **automatic referencing and dereferencing**, and calling methods is a scenario where this behavior applies.

+ define associated functions (aka. static methods) by not writing `self` parameter.

### Enum

+ enum can have an internal value for each enumerator:

  ```rust
  fn main() {
      enum IpAddr {
          V4(String),
          V6(String),
      }
  
      let home = IpAddr::V4(String::from("127.0.0.1"));
  
      let loopback = IpAddr::V6(String::from("::1"));
  }
  ```

+ the internal value type can even be anonymous struct:

  ```rust
  enum Message {
      Quit,
      Move { x: i32, y: i32 },
      Write(String),
      ChangeColor(i32, i32, i32),
  }
  ```

+ use `if let` to check whether the enumerator and its internal value are equal:

  ```rust
  if let Some(3) = some_u8_value {
      println!("three");
  }
  ```

### Crate and Module

+ a package can have 0 or 1 library crate and any number of binary crates. My understanding is that library crate contains key logic of the package and binary crate is just like entrance (how to use the library).

+ A create is a tree of modules.

+ Each file under `src/bin` directory will be a separate binary crate

+ Package could contain a library crate with the same name as the package and `src/lib.rs` is the crate root. Correspondingly, `src/main.rs` is root of the binary crate whose name is the same as the package.

+ absolute path starts with the crate name or `crate` literal while relative path starts with `self`, `super` or  an identifier in the current module.

+ An item can access its ancestor and private sibling, but not private child of its sibling.

+ each field in the struct can be specified as public separately, while privacy of enumerators always corresponds to enum.

+ ```rust
  use std::io::Result as IoResult;
  pub use crate::front_of_house::hosting;  // re-export
  use std::{cmp::Ordering, io};  // nested paths
  use std::io::{self, Write};
  ```

+ declaring mod in a file tells Rust to find its definition in another file whose filename is the same as the module name.

  `src/lib.rs`:

  ```rust
  mod front_of_house;
  ```

  `src/front_of_house.rs`:

  ```rust
  pub mod hosting;
  ```

  then there should be a file named `src/front_of_house/hosting.rs`.

### Collections

#### vector

+ create: `let v: Vec<i32> = Vec::new();` or `let v = vec![1, 2, 3];`
+ update: `let mut v = Vec::new(); v.push(3);`
+ read: `&v[1]` or `v.get(1)`, the latter returns an `Option`
+ iterate: `for i in &v` (if not reference here, `v` will get moved)

#### string

+ create: `::new()`, `::from()`
+ update: `push`, `push_str`, `operator+` or `format!`
+ read: use string slice `&s[0..2]`
+ iterate: `in s.chars()` or `in s.bytes()`

#### hashmap

`HashMap` is not in the prelude so we need to import it explicitly: `use std::collections::HashMap;`

+ create: `::new()`
+ update: `.insert()`
+ read: `.get()`
+ iterate: `for (k, v) in &map`

### Error handling

+ `panic!`

+ `operator?` to unwrap Ok value or return Err value:

  ```rust
  fn read_username_from_file() -> Result<String, io::Error> {
      let mut f = File::open("hello.txt")?;
      let mut s = String::new();
      f.read_to_string(&mut s)?;
      Ok(s)
  }
  ```

### Generics, Traits and Lifetimes

#### Generics

+ generic struct:

  ```rust
  struct Point<T> {
      x: T,
      y: T,
  }
  
  impl<T> Point<T> {
      fn x(&self) -> &T {
          &self.x
      }
  }
  ```

+ generic enum:

  ```rust
  enum Result<T, E> {
      Ok(T),
      Err(E),
  }
  ```

+ generic function:

  ```rust
  fn largest<T: PartialOrd + Copy>(list: &[T]) -> T;
  ```

#### Traits

+ trait is just like *interface* or *pure virtual class* (trait can also provide default implementations):

  ```rust
  pub trait Summary {
      fn summarize(&self) -> String;
  }
  ```

  implement a trait on a type:

  ```rust
  impl Summary for Tweet {
      fn summarize(&self) -> String {
          format!("{}: {}", self.username, self.content)
      }
  }
  ```

+ trait bound, i.e. the type that implements the trait:

  ```rust
  pub fn notify(item: &impl Summary) {
      println!("Breaking news! {}", item.summarize());
  }
  
  pub fn notify<T: Summary>(item: &T) {
      println!("Breaking news! {}", item.summarize());
  }
  
  pub fn notify<T>(item: &T) 
      where T: Summary
  {...}
  
  fn returns_summarizable() -> impl Summary {
      Tweet {
          username: String::from("horse_ebooks"),
          content: String::from(
              "of course, as you probably already know, people",
          ),
          reply: false,
          retweet: false,
      }
  }
  ```

+ trait bounds can be used to accomplish functionality similar to *template specialization* in cpp:

  ```rust
  struct Pair<T> {
      x: T,
      y: T,
  }
  
  impl<T> Pair<T> {
      fn new(x: T, y: T) -> Self {
          Self { x, y }
      }
  }
  
  impl<T: Display + PartialOrd> Pair<T> {
      fn cmp_display(&self) {
          if self.x >= self.y {
              println!("The largest member is x = {}", self.x);
          } else {
              println!("The largest member is y = {}", self.y);
          }
      }
  }
  ```

#### Lifetimes

+ Lifetime annotations don’t change how long any of the references live, they just help borrow checker finish the check.

+ like generic type, lifetime specifier also needs to declare before use:

  ```rust
  fn longest<'a>(x: &'a str, y: &'a str) -> &'a str;
  ```

+ lifetime annotation in struct:

  ```rust
  struct ImportantExcerpt<'a> {
      part: &'a str,
  }
  
  impl<'a> ImportantExcerpt<'a> {
      fn level(&self) -> i32 {
          3
      }
  }
  ```

+ [Lifetime elision rules](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html#lifetime-elision)

+ `'static` lifetime

### Tests

+ put unit tests alongside tested code.

  ```rust
  #[cfg(test)]
  mod tests {
      #[test]
      fn it_works() {
          assert_eq!(2 + 2, 4);
      }
  }
  ```

+ put integration tests under `tests/` directory separately.

  ```rust
  use adder;
  
  #[test]
  fn it_adds_two() {
      assert_eq!(4, adder::add_two(2));
  }
  ```

### Closures and Iterators

+ closures compared to functions:

  ```rust
  fn  add_one_v1   (x: u32) -> u32 { x + 1 }
  let add_one_v2 = |x: u32| -> u32 { x + 1 };
  let add_one_v3 = |x|             { x + 1 };
  let add_one_v4 = |x|               x + 1  ;
  ```

+ use `move` keyword before parameter list to force the closure to take ownership of variables it uses.

+ closures can implement the `Fn`, `FnMut` and `FnOnce` traits.

  ```rust
  struct Cacher<T>
  where
      T: Fn(u32) -> u32,
  {
      calculation: T,
      value: Option<u32>,
  }
  ```

+ all iterators implement the `Iterator` trait:

  ```rust
  pub trait Iterator {
      type Item;
  
      fn next(&mut self) -> Option<Self::Item>;
  
      // methods with default implementations elided
  }
  ```

## Something worth noting

+ when reading Chapter 12, the book, I tried that tiny project in VS Code. There is a red squiggle under `use minigrep::Config;` when first typing that, a workaround is to reload vscode.
+ type casting in Rust: `x as i32`
+ using a value to construct a field in the struct will move that value

## Q & A

+ Do the library crate (`src/lib.rs`) and binary crate (`src/main.rs`) have the same crate name?

  yes. (see section 7.1, *the Book*) actually, if you take a look at `target/debug` dir under the project, you will find that the binary crate is built into an executable `proj` (assuming that the package name is `proj`) while the library crate is built into a library `libproj.rlib`.

+ What is blanket implementation?

+ why is it not allowed that multiple mutable references exist at the same time?

+ what's trait `?Sized`

## Rust By Example Notes

+ `ToString` trait (which defines `to_string` method) is implemented automatically for those types which implements `Display` trait:

  ```rust
  impl<T> ToString for T
  where
      T: Display + ?Sized, 
  { /* ... */ }
  ```

  *[reference](https://stackoverflow.com/questions/27769681/should-i-implement-display-or-tostring-to-render-a-type-as-a-string)*

+ **borrow** means taking a reference of an object (without transfering ownership) `println!`-like macros just borrows the object (although we don't write `&` explicitly).

### Custom Types

+ use `std::mem::size_of` to get size of a type, note that the invocation should look like `size_of::<MyType>()` (that's so called **turbofish**). (`typeof` is reserved to get type of an object, but it's not implemented yet)

  and `std::mem::size_of_val` could be applied on an object instead of a type.

+ `(5u32,)` is a single-element tuple while `(5u32)` is just an integer.

+ arrays can be automatically borrowed as slices:

  ```rust
  fn analyze_slice(slice: &[i32]) {
      println!("first element of the slice: {}", slice[0]);
      println!("the slice has {} elements", slice.len());
  }
  
  analyze_slice(&[1, 2, 3]);
  analyze_slice(&[1, 2, 3][0..2]);
  ```

+ struct could also be destructured:

  ```rust
  struct S {
      x: i32,
      y: i32,
  }
  
  let s = S { x: 1, y: 2 };
  let S { x, y } = s;
  ```

  and the destructuring can even be nested.

+ ```rust
  enum E {
      EA(S),
      EB
  }
  
  match e {
      E::EA(s) => {},  // s has type S
      E::EB => {}
  }
  
  match &e {
      E::EA(s) => {},  // s has type &S
      E::EB => {}
  }
  
  match e {
      E::EA(ref s) => {},  // s has type &S
      E::EB => {}
  }
  ```

+ use `type Alias = i64` to give a type an alias, note that the alias is not a new type so different alias of a type share the same type also.

### Conversion

+ primitive types can be converted to each other through `as`-keyword casting, while custom types need `From` and `Into` traits.

+ after implementing `From` trait for our type like

  ```rust
  impl From<i32> for Number {
      fn from(item: i32) -> Self {
          Number { value: item }
      }
  }
  ```

  we get this for free:

  ```rust
  let num: Number = 5.into();
  ```

+ `TryFrom` and `TryInto` returns a `Result` instead of the object after conversion directly.

+ to convert an object to `String`, we need `ToString` trait, which is automatically provided if implement `Display` trait; to convert `String` to an object, we need `parse` method from `FromStr` trait, and integral type has implemented that.

  ```rust
  let turbo_parsed = "10".parse::<i32>().unwrap();
  ```

  note that `parse` method supports turbofish syntax while `into` method doesn't.

### Expressions

+ there are mainly two kinds of statements in Rust: variable binding with `let` and expression plus a `;`.

  blocks are expression, they are evaluated to the last expression in the block, if the last one is a statement, the block evaluates to `()`.

### Flow of Control

+ inner loops could `break` and `continue` outer loops with loop labels:

  ```rust
  'outer: loop {
      'inner: loop {
          break 'outer;
      }
  }
  ```

+ range notation `a..b` creates an iterator on the range `[a, b)` and `a..=b` creates `[a, b]`.

+ to convert a collection into iterators, there are three alternatives: `into_iter()`, `iter()` and `iter_mut()`, whose names are self-explanatory.

+ reference could be created at assigner side (right side of `=`) with `&` and also assignee side with `ref` keyword

  ```rust
  let ref a = &3;  // a is &&i32
  ```

+ when using `match` guard, compiler doesn't check the guard expression for whether all possible conditions have been covered, so in the last arm, `_` is needed.

+ `if let a = b` reads *if `b` could be destructured as `a`* (there is also a `while let`)

### Functions

+ `&self` in the method argument is just a sugar for `self: &Self`.

+ when the closure captures variables, it follows this order (more and more stricter):

  + by reference: `&T`
  + by mutable reference: `&mut T`
  + by value: `T`

  i.e. if capturing by reference is enough, it won't capture by mutable reference, not to mention capture by value

  we can force the closure to capture by value by adding a `move` keyword before `||`

+ when passing a closure as argument of a function. its type could be annotated through trait bound like:

  ```rust
  fn apply<F>(f: F) where
      F: FnOnce() {
      f();
  }
  ```

  there are three kinds of such trait: `Fn`, `FnMut` and `FnOnce`. there relationship is kinda like

  ```
  <-------------------Fn-------------------> (&T)
         <-----------FnMut-----------> (&mut T)
              <-----FnOnce-----> (T)
  
  ```

  that's to say, `Fn` is the most strict (a little bit counterintuitive, hah), which just applies to those closures which capture all variables by reference while `FnOnce` applies to all three kinds of closures.

+ Question: Do all functions (not closures) implement `Fn` trait? (since functions don't capture any variable)

+ closures could be returned from functions, but they must be denoted with `move` because captured variables won't have valid references out of function scope. So the function returning closures could be `Fn`, `FnMut` or `FnOnce`.

+ `into_iter()` for arrays unusually yields references.

+ diverging functions are those which never return (e.g. due to `panic!`),  actually their return type is marked as `!`, which could be cast to any other type.

### Modules, Crates and Cargo

+ **A crate is a compilation unit in Rust.** when running `rustc a.rs`, `a.rs` is treated as a crate file. mod declarations like `mod b;` will find the corresponding file (`b.rs` or `b/mod.rs`) and insert the content of that module file into where the mod declaration is found.
+ if using `rustc`, add `--extern` option to link with an extern lib crate.
+ cargo may run multiple tests concurrently, so mind the potential race condition.

### Attributes

+ `#![attribute]` applies to the whole crate while `#[attribute]` applies to a module or item.

+ `#[cfg]` attribute enables conditional compilation, e.g.:

  ```rust
  #[cfg(target_os = "linux")]
  ```

  while the `cfg!` macro supports check in run-time:

  ```rust
  if cfg!(target_os = "linux") {}
  ```

+ pass `--cfg` option to `rustc` to add definitions for conditional compilation. e.g. when running `cargo test`, I guess `--cfg test` is added so modules annotated by `#[cfg(test)]` will get compiled.

### Generics

+ items from traits can only be used if the trait is in scope. so if we want to invoke a method defined in a trait (e.g. `m::TR`), we need to use that trait first to bring it into scope (`use m::TR;`)

  this can explain the `rand::Rng` wondering in the chap.2, *the Book*

+ inside the trait, we can declare some associated items such as types (with `type` keyword, but not alias semantic) and override them in following `impl` blocks.

### Scoping rules

+ `Drop` trait should be implemented if a custom destructor is needed.

+ mutability of data can be changed after transferring the ownership:

  ```rust
  let immutable_box = Box::new(5u32);
  let mut mutable_box = immutable_box;
  ```

+ pattern bindings can take both by-reference and by-move forms:

  ```rust
  let Person { name, ref mut age } = person;
  ```

  *so-called pattern bindings take place when using `if let`, `match` and other destruction operations.*

  if we bind by reference and move at the same time, **partial move** will happen, which means part of the object is moved so we can no longer use the object as a whole, but still the unmoved part of the object:

  ```rust
  println!("{}", person);       // value borrowed here after partial move
  println!("{}", person.name);  // value borrowed here after move
  println!("{}", person.age);   // ok
  ```

+ `ref` keyword can be used only during pattern matching and destruction (most with `let` keyword) so it's not completely alternative to `&`.

#### lifetime specifier

+ the lifetime specifier specifies what the lifetime of a reference **should be**. it can apply to both functions and custom types, which will be explained below.

+ lifetime specifier on **functions** connects the relationship of parameters and return value in terms of lifetime.

  and lifetime specifier which is not constrained (only appears at return value) defaults to `'static`.

+ lifetime specifier on fields of **struct or enum** means the field must outlive the struct or enum:

  ```rust
  struct Borrowed<'a>(&'a i32);
  
  enum Either<'a> {
      Num(i32),
      Ref(&'a i32),
  }
  ```

+ lifetime can be bounded as well like `T: 'a`, which means the parameter of type `T` must outlive `'a`.

+ owned data always passes a `'static` bound while a reference generally doesn't:

  ```rust
  fn f(input: impl Debug + 'static) {}
  let i = 5;
  print_it(i);  // ok
  print_it(&i); // err
  ```

+ in a short word, the *lifetime elision rules* is to say that under such conditions you can emit lifetime specifier of reference parameter and return value:

  + the function is a method, which means the first parameter is `&self` or `&mut self`, then return value has the same lifetime with it
  + the function has only one parameter, then return value has the same lifetime with it as well.

### Traits

+ **The Rust compiler needs to know how much space the function's return type requires**, so if we want to return a trait `TR`, we cannot directly write like `-> TR`. instead, wrap it with a `Box` whose size is know at compile time.

  What's more, calls to the `TR` trait object are dynamically dispatched (recall the virtual table), so it needs to be highlighted by a `dyn` keyword. that's why the return value should be `Box<dyn TR>`

+ **operators are syntactic sugar for method calls**, `a + b` is actually `a.add(b)`. so we can implement different `Add<A>` traits for different type `B` to overload the addition operator.

+ `drop` method from `Drop` trait could also be manually invoked.

+ to implement `Iterator` trait on a custom type, we only need to define `next` method.

+ if the size of return type is known at compile time but the type itself cannot be written out (like closure) or concisely expressed, `impl Trait` as return type can be used.

  note that this is different from returning a `Box<dyn trait>`, which actual return type is not known until run-time.

+ to enable copy (stack-only) on a type, we need to derive both `Clone` and `Copy` trait for it.

+ although there is no inheritance in Rust, you can define a trait as a superset of another trait, which is called *supertraits*:

  ```rust
  trait A {}
  trait B: A {}
  ```

+ use fully qualified syntax to avoid disambiguating:

  ```rust
  <Type as Trait>::function(receiver_if_method, next_arg, ...);
  ```

### Macros

+ declarative macros are defined with `macro_rules!` while procedural macros are kinda like functions operating on `TokenStream` defined in `proc_macro` crate. we will focus on the former.

### Error handling

+ `Option` provides `map` method to map `Option<T>` to `Option<U>` by applying a function to the contained value whose type is `T`.

  It also provides `and_then` method to apply a function to the contained value and then return it (whose type is `U` now) without wrapping it with `Option`.

+ `Result` is a richer version of `Option` that describes possible errors instead of only possible absence.

+ `Result` could also be return type of `main`, in which condition, an error message will printed if `Error` is returned.

##### Last-modified date: 2021.3.29, 8 p.m.