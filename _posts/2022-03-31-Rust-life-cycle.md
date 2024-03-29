---
title: Rust 生命周期理解
author: rvyou
date: 2022-03-31 10:32:00 +0800
categories: [Rust,生命周期]
tags: [Rust,生命周期]
---

Rust 生命周期消除规则：

> - 每一个引用参数都会获得独自的生命周期
>
>   ```rust
>   fn foo(x: & i32, y: &i32){}//两个函数代码是等价
>   fn foo<'a, 'b>(x: &'a i32, y: &'b i32){//独立的生命周期
>   }//如果返回值是其中的某个参数，编译器不清楚返回那个生命周期
>   ```
>
> - 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期，也就是所有返回值的生命周期都等于该输入生命周期
>
>   ```rust
>   fn foo(x: &i32) -> &i32{}//两个函数代码是等价
>   fn foo<'a>(x: &'a i32) -> &'a i32{}
>   ```
>
> - 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期
>
>   ```rust
>   impl<'a> test<'a>{
>       fn foo<'b>(&'a self,s:&'b str)->&'a str{
>   }
>   }//等价与下面函数
>   impl test{
>       fn foo(& self,s:& str)->& str{
>       }
>   }
>   ```

## NLL(Non-Lexical Lifetime)

> 应用的声明周期从借用处开始到最后一次使用的地方(rust 从 1.31 版本开始)

## Reborrow 再借用

> 可以通过 &*v 进行再次借用，再借用生命周期内不能使用 v 的可变借用

```rust
let mut s = String::from("test");
let a = &mut s;
let aa = &*a;
//这里不能使用 a 的可变借用，但是可以使用不可变借用
println!("{}",aa);
//aa的声明周期结束 NLL 规则
println!("{}",a);
```

## ’static

> 静态生命周期直到程序终止

## &‘ static

> 和 ’static 一样，但是他是针对引用 (fn foo< T: 'static>( var: & T) 这样就是相等的 )
