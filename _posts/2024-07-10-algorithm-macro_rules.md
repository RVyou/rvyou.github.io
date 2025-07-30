---
title: rust 宏
author: rvyou
date: 2024-07-10 12:12:05
categories: [rust,macros]
tags: [rust,macros]
---

> 宏经常写老是忘记，记录一下方便记忆。回头重新看文档还是需要时间，形成自己记忆方式就不需要那么多时间。

[查看宏展开的代码](https://play.rust-lang.org/) tool  ----》expand macros

## 宏基础

### 宏类型

```rust
#[derive(Error, Debug)] // 派生宏
vec![];//声明宏 vec!();
#[error("this is error")] //属性宏 如果只想把属性施加到某个模块或者 item 上，就把 ! 去掉。参数：= "value"， (key = "value")，("value")，或者无参数
```

### 声明宏模板

```rust
macro_rules! $ident {
    ($matcher) => {$expansion}
}
```

### 声明宏操作符号

$(表达式)*    反复匹配0 到多次

*    表示重复 0 到多次。

+    表示重复 1 到多次。

?     表示重复 0 次或 1 次。

## 声明宏-元变量类型

### expr（常用）

expr：匹配表达式

```rust
mod aa {
    #[macro_export] //把 宏导出到当前crate root下了
    macro_rules! add {
        ($a:expr,$b:expr)=>{
            {
                $a+$b
            }
        };
        ($a:expr) =>{
            {
                $a
            }
        };
        ($($a:expr),*) =>{
            {
                0 //如果没有那么就是0，其实上面那些表达式可以不需要
                $(
                    +$a
                )*

            }
        };
    }
}
fn main() {
    println!("{:?}", add!(3,4));
    println!("{:?}", add!(3));
    println!("{:?}", add!(3,2,3));
}
```

### ty（常用）

`ty`：匹配类型

```rust
macro_rules! type_number {
    ($a:ty) => {
        let a = <$a>::MAX;
         println!("{}",a);
    };
}
fn main() {
    type_number!(i32);
    type_number!(u8);
    type_number!(f64);
}
```

### ident（常用）

`ident`: 分类符用于匹配任何形式的标识符 (identifier) 或者关键字。

```rust
macro_rules! ident_demo {
    ($a:ident) => {
        fn $a(){
         println!("111");
        };

    };
    ($a:ident,$b:ident) => {
        fn $a(){
            $b 0{
                _=>{
                    println!("match");
                }
            }
         println!("111");
        };

    };
}
fn main() {
    ident_demo!(aaa,match);
    aaa();
}
```

### item（常用）

`item`: 分类符只匹配 Rust 的 item 的 定义 (definitions)

```rust
macro_rules! item_demo {
    ($($a:item);*) => {
       ();
    };
}
fn main() {
    item_demo! (
   enum Bar {Baz};
   struct Foo{};
   impl Foo {}
    );
}
```

### tt（常用）

`tt` 分类符用于匹配标记树 (TokenTree)

```rust
//todo
```

### path（常用）

`path` 分类符用于匹配类型中的路径 ([TypePath](https://doc.rust-lang.org/reference/paths.html#paths-in-types))。

这包括函数式的 trait 形式。

```rust
macro_rules! paths {
    ($($path:path)*) => (
        $(
        use $path;
        )*
    );
}
pub mod aa{
    pub const  B:i32 = 11;
}

fn main() {
    paths! {
    aa::B
    };
    println!("{:?}",B);
    //paths! {
    //    ASimplePath
    //    ::A::B::C::D
    //    G::<eneri>::C
    //    FnMut(u32) -> ()
    //};
}
```

### stmt （常用）

`stmt` 分类符只匹配的 语句 ([statement](https://doc.rust-lang.org/reference/statements.html))。 除非 item 语句要求结尾有分号，否则 **不会** 匹配语句最后的分号。

```rust
macro_rules! statements {
    ($($stmt:stmt)*) => ($($stmt)*);
}

fn main() {
    statements! {
        struct Foo;
        fn foo() {}
        let zig = 3
        let zig = 3;
        3
        3;
        if true {} else {}
        {}
    }
}
```

### block

block 类型符只匹配 block 表达式。

```rust
macro_rules! block_demo {
    ($a:block) => {
         println!("{}",$a);
    };
}
fn main() {
    block_demo!({ println!("block");"aa".to_string()});
}});
}
```

### lifetime

`lifetime`分类符用于匹配生命周期注解或者标签 (lifetime or label)。 它与 ident 很像，但是 lifetime 会匹配到前缀 ''

```rust
macro_rules! lifetimes {
    ($($lifetime:lifetime)*) => ();
}

lifetimes! {
    'static
    'shiv
    '_
}
fn main() {}
```

### literal

`literal` 分类符用于匹配字面表达式 ([literal expression](https://doc.rust-lang.org/reference/expressions/literal-expr.html))。

```rust
macro_rules! literals {
    ($($literal:literal)*) => ();
}

literals! {
    -1
    "hello world"
    2.3
    b'b'
    true
}
fn main() {}
```

### meta

`meta` 分类符用于匹配属性 attribute， 准确地说是属性里面的内容。通常你会在 `#[$meta:meta]` 或 `#![$meta:meta]` 模式匹配中 看到这个分类符。

```rust
macro_rules! metas {
    ($($meta:meta)*) => ();
}

metas! {
    ASimplePath
    super::man
    path = "home"
    foo(bar)
}
fn main() {}
```

### pat

`pat`分类符用于匹配任何形式的模式 (pattern)，包括 2021 edition 开始的 or-patterns。

```rust
macro_rules! patterns {
    ($($pat:pat)*) => ();
}

patterns! {
    "literal"
    _
    0..5
    ref mut PatternsAreNice
    0 | 1 | 2 | 3 
}
fn main() {}
```

##### pat_param

从 2021 edition 起， or-patterns 模式开始应用，这让 `pat` 分类符不再允许跟随 `|`。

为了避免这个问题或者说恢复旧的 `pat` 分类符行为，你可以使用 `pat_param` 片段，它允许 `|` 跟在它后面，因为 `pat_param` 不允许 top level 或 or-patterns。

```rust
macro_rules! patterns {
    (pat: $pat:pat) => {
        println!("pat: {}", stringify!($pat));
    };
    (pat_param: $($pat:pat_param)|+) => {
        $( println!("pat_param: {}", stringify!($pat)); )+
    };
}
fn main() {
    patterns! {
       pat: 0 | 1 | 2 | 3
    }
    patterns! {
       pat_param: 0 | 1 | 2 | 3
    }
}
```

### vis

`vis` 分类符会匹配 **可能为空** 可见性修饰符 Visibility qualifier。

```rust
macro_rules! visibilities {
    //         ∨~~注意这个逗号，`vis` 分类符自身不会匹配到逗号
    ($($vis:vis,)*) => ();
}

visibilities! {
    , // 没有 vis 也行，因为 $vis 隐式包含 `?` 的情况
    pub,
    pub(crate),
    pub(in super),
    pub(in some_path),
}
fn main() {}
```

### 限制

- `stmt` 和 `expr`：`=>`、`,`、`;` 之一
- `pat`：`=>`、`,`、`=`、`if`、`in` 之一
- [`pat_param`]：`=>`、`,`、`=`、`|`、`if`、`in` 之一
- `path` 和 `ty`：`=>`、`,`、`=`、`|`、`;`、`:`、`>`、`>>`、`[`、`{`、`as`、`where` 之一； 或者 `block`型的元变量
- `vis`：`,`、除了 `priv` 之外的标识符、任何以类型开头的标记、 `ident`或 `ty` 或 `path` 型的元变量
- 其他片段分类符所跟的内容无限制

## 过程宏

`过程宏`允许在执行函数时创建句法扩展。过程宏有三种形式:

`类函数宏(function-like macros) `-> `custom!(...)`

`派生宏(derive macros)`-> `#[derive(CustomDerive)]`

`属性宏(attribute macros)` -> `#[CustomAttribute]`

```cargo
[lib]
proc-macro = true
[dependencies]
darling = "0.20.10"
proc-macro2 = "1.0.86"# proc_macro 的封装 官方
quote = "1" #配合syn，AST 变成 TokenSteam，从普通文本代码生成的 TokenSteam 中。
syn = { version = "2.0.70", features = ["full", "extra-traits"] } # "extra-traits"方便后续打印调试信标记流。
```

- `syn` — Rust 的语法解析器。这可以帮助您将输入标记流解析为 Rust AST。AST 是您在尝试编写自己的解释器或编译器时最常遇到的概念，但对于使用宏，基本的理解是必不可少的。毕竟，从某种意义上说，宏只是您为编译器编写的扩展。如果您有兴趣了解有关 AST 的更多信息，[请查看这个非常有用的介绍](https://dev.to/balapriya/abstract-syntax-tree-ast-explained-in-plain-english-1h38)。
- `quote` — 引用是，这是一个巨大的概括，一个帮助我们执行反向操作的板条箱`syn`。它帮助我们将 Rust 源代码转换为可以从宏中输出的标记流。
- `proc-macro2` — 标准库提供了一个`proc-macro`crate，但它提供的类型不能存在于过程宏之外。`proc-macro2`是标准库的包装器，使所有内部类型都可以在宏上下文之外使用。例如，这不仅允许`syn`和`quote`用于过程宏，还允许在常规 Rust 代码中使用，如果您有这样的需要的话。如果我们想对我们的宏或其扩展进行单元测试，我们确实会广泛使用它。
- `darling`–它有助于解析和使用宏参数，否则这将是一个繁琐的过程，因为必须手动从语法树中解析它。`darling`为我们提供了`serde`类似 - 的功能，可以自动将输入参数树解析为我们的参数结构。它还帮助我们处理无效参数、必需参数等错误

### 类函数宏(function-like macros)

```rust
//lib.rs
use proc_macro::TokenStream;
use syn::parse_macro_input;
use quote::quote;
#[proc_macro]
pub fn fn_macro(input: TokenStream) -> TokenStream {
    //TokenStream [
    //     Literal {
    //         kind: Str,
    //         symbol: "asd; asda",
    //         suffix: None,
    //         span: #0 bytes(87..98),
    //     },
    // ]
    eprintln! {"{:#?}", input};
    let s = parse_macro_input!(input as syn::LitStr);
    // LitStr {
    //     token: "aaa",
    // }
    eprintln! {"{:#?}", s};
    //转换会token
    quote!(let data = #s;println!(" input : {}", data);).into()
}
// main.rs
fn main() {
    fn_macro!("asd; asda");
    println!("{:?}",data);
}
```

### 派生宏(derive macros)

```rust
use proc_macro::TokenStream;
use darling::ast::NestedMeta;
use quote::quote;
use syn::Expr;
use darling::{FromMeta,Error};

#[derive(FromMeta)]
struct CachedParams {
    //key 对应 相应的内容
    ab: Option<Expr>,
    c: Option<()>,
}

#[proc_macro_attribute]
pub fn macro_attribute(attrs: TokenStream, input: TokenStream) -> TokenStream {
    // 过程宏的属性
    // eprintln!("attrs:{:#?}", attrs);
    // eprintln!("input:{:#?}", input);
    //attrs:TokenStream [
    //     Ident {
    //         ident: "ab",
    //         span: #0 bytes(48..50),
    //     },
    //     Punct {
    //         ch: ',',
    //         spacing: Alone,
    //         span: #0 bytes(50..51),
    //     },
    //     Ident {
    //         ident: "b",
    //         span: #0 bytes(51..52),
    //     },
    // ]
    // input:TokenStream [
    //     Ident {
    //         ident: "struct",
    //         span: #0 bytes(55..61),
    //     },
    //     Ident {
    //         ident: "A",
    //         span: #0 bytes(62..63),
    //     },
    //     Group {
    //         delimiter: Brace,
    //         stream: TokenStream [
    //             Ident {
    //                 ident: "B",
    //                 span: #0 bytes(69..70),
    //             },
    //             Punct {
    //                 ch: ':',
    //                 spacing: Alone,
    //                 span: #0 bytes(70..71),
    //             },
    //             Ident {
    //                 ident: "i64",
    //                 span: #0 bytes(71..74),
    //             },
    //         ],
    //         span: #0 bytes(63..76),
    //     },
    // ]
    let attr_args = match NestedMeta::parse_meta_list(attrs.into()) {
        Ok(v) => v,
        Err(e) => {

            return proc_macro::TokenStream::from(Error::from(e).write_errors());
        }
    };
    let data = match CachedParams::from_list(&attr_args) {
        Ok(params) => params,
        Err(error) => {
            return proc_macro::TokenStream::from(Error::from(error).write_errors());
        }
    };
    eprintln!("{:?}", data.ab);
    eprintln!("{:?}", data.c);
    //Some(Expr::Lit { attrs: [], lit: Lit::Int { token: 1 } })
    //Some(())
    quote!().into()
}
//main.rs
#[macro_attribute(ab ="1",c)]
struct A{
    B:i64
}
```



### 属性宏(attribute macros)

```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::Data;

#[proc_macro_derive(IntoHashMap)]
pub fn into_hash_map(item: TokenStream) -> TokenStream {
    let input = syn::parse_macro_input!(item as syn::DeriveInput);
    // eprintln!("ident:{:#?}", input.ident);
    // eprintln!("attrs:{:#?}", input.attrs);
    // eprintln!("vis:{:#?}", input.vis);
    // eprintln!("data:{:#?}", input.data);
    // eprintln!("generics:{:#?}", input.generics);
    //ident:Ident {
    //     ident: "User",
    //     span: #0 bytes(61..65),
    // }
    // attrs:[]
    // vis:Visibility::Public(
    //     Pub,
    // )
    // data:Data::Struct {
    //     struct_token: Struct,
    //     fields: Fields::Named {
    //         brace_token: Brace,
    //         named: [
    //             Field {
    //                 attrs: [],
    //                 vis: Visibility::Inherited,
    //                 mutability: FieldMutability::None,
    //                 ident: Some(
    //                     Ident {
    //                         ident: "username",
    //                         span: #0 bytes(72..80),
    //                     },
    //                 ),
    //                 colon_token: Some(
    //                     Colon,
    //                 ),
    //                 ty: Type::Path {
    //                     qself: None,
    //                     path: Path {
    //                         leading_colon: None,
    //                         segments: [
    //                             PathSegment {
    //                                 ident: Ident {
    //                                     ident: "String",
    //                                     span: #0 bytes(82..88),
    //                                 },
    //                                 arguments: PathArguments::None,
    //                             },
    //                         ],
    //                     },
    //                 },
    //             },
    //             Comma,
    //             Field {
    //                 attrs: [],
    //                 vis: Visibility::Inherited,
    //                 mutability: FieldMutability::None,
    //                 ident: Some(
    //                     Ident {
    //                         ident: "first_name",
    //                         span: #0 bytes(94..104),
    //                     },
    //                 ),
    //                 colon_token: Some(
    //                     Colon,
    //                 ),
    //                 ty: Type::Path {
    //                     qself: None,
    //                     path: Path {
    //                         leading_colon: None,
    //                         segments: [
    //                             PathSegment {
    //                                 ident: Ident {
    //                                     ident: "String",
    //                                     span: #0 bytes(106..112),
    //                                 },
    //                                 arguments: PathArguments::None,
    //                             },
    //                         ],
    //                     },
    //                 },
    //             },
    //             Comma,
    //             Field {
    //                 attrs: [],
    //                 vis: Visibility::Inherited,
    //                 mutability: FieldMutability::None,
    //                 ident: Some(
    //                     Ident {
    //                         ident: "last_name",
    //                         span: #0 bytes(118..127),
    //                     },
    //                 ),
    //                 colon_token: Some(
    //                     Colon,
    //                 ),
    //                 ty: Type::Path {
    //                     qself: None,
    //                     path: Path {
    //                         leading_colon: None,
    //                         segments: [
    //                             PathSegment {
    //                                 ident: Ident {
    //                                     ident: "String",
    //                                     span: #0 bytes(129..135),
    //                                 },
    //                                 arguments: PathArguments::None,
    //                             },
    //                         ],
    //                     },
    //                 },
    //             },
    //             Comma,
    //             Field {
    //                 attrs: [],
    //                 vis: Visibility::Inherited,
    //                 mutability: FieldMutability::None,
    //                 ident: Some(
    //                     Ident {
    //                         ident: "age",
    //                         span: #0 bytes(141..144),
    //                     },
    //                 ),
    //                 colon_token: Some(
    //                     Colon,
    //                 ),
    //                 ty: Type::Path {
    //                     qself: None,
    //                     path: Path {
    //                         leading_colon: None,
    //                         segments: [
    //                             PathSegment {
    //                                 ident: Ident {
    //                                     ident: "u32",
    //                                     span: #0 bytes(146..149),
    //                                 },
    //                                 arguments: PathArguments::None,
    //                             },
    //                         ],
    //                     },
    //                 },
    //             },
    //             Comma,
    //         ],
    //     },
    //     semi_token: None,
    // }
    // generics:Generics {
    //     lt_token: None,
    //     params: [],
    //     gt_token: None,
    //     where_clause: None,
    // }

    let struct_identifier = &input.ident;
    //判断数据是什么的 这里判断是否是结构体
    match &input.data {
        Data::Struct(syn::DataStruct { struct_token, fields, semi_token }) => {
            eprintln!("struct_token:{:#?}", struct_token);
            eprintln!("semi_token:{:#?}", semi_token);
            eprintln!("fields:{:#?}", fields);
            //struct_token:Struct
            // semi_token:None
            // fields:Fields::Named {
            //     brace_token: Brace,
            //     named: [
            //         Field {
            //             attrs: [],
            //             vis: Visibility::Inherited,
            //             mutability: FieldMutability::None,
            //             ident: Some(
            //                 Ident {
            //                     ident: "username",
            //                     span: #0 bytes(72..80),
            //                 },
            //             ),
            //             colon_token: Some(
            //                 Colon,
            //             ),
            //             ty: Type::Path {
            //                 qself: None,
            //                 path: Path {
            //                     leading_colon: None,
            //                     segments: [
            //                         PathSegment {
            //                             ident: Ident {
            //                                 ident: "String",
            //                                 span: #0 bytes(82..88),
            //                             },
            //                             arguments: PathArguments::None,
            //                         },
            //                     ],
            //                 },
            //             },
            //         },
            //         Comma,
            //         Field {
            //             attrs: [],
            //             vis: Visibility::Inherited,
            //             mutability: FieldMutability::None,
            //             ident: Some(
            //                 Ident {
            //                     ident: "first_name",
            //                     span: #0 bytes(94..104),
            //                 },
            //             ),
            //             colon_token: Some(
            //                 Colon,
            //             ),
            //             ty: Type::Path {
            //                 qself: None,
            //                 path: Path {
            //                     leading_colon: None,
            //                     segments: [
            //                         PathSegment {
            //                             ident: Ident {
            //                                 ident: "String",
            //                                 span: #0 bytes(106..112),
            //                             },
            //                             arguments: PathArguments::None,
            //                         },
            //                     ],
            //                 },
            //             },
            //         },
            //         Comma,
            //         Field {
            //             attrs: [],
            //             vis: Visibility::Inherited,
            //             mutability: FieldMutability::None,
            //             ident: Some(
            //                 Ident {
            //                     ident: "last_name",
            //                     span: #0 bytes(118..127),
            //                 },
            //             ),
            //             colon_token: Some(
            //                 Colon,
            //             ),
            //             ty: Type::Path {
            //                 qself: None,
            //                 path: Path {
            //                     leading_colon: None,
            //                     segments: [
            //                         PathSegment {
            //                             ident: Ident {
            //                                 ident: "String",
            //                                 span: #0 bytes(129..135),
            //                             },
            //                             arguments: PathArguments::None,
            //                         },
            //                     ],
            //                 },
            //             },
            //         },
            //         Comma,
            //     ],
            // }

            let field_identifiers = fields.iter().map(|item| item.ident.as_ref().unwrap()).collect::<Vec<_>>();

            quote! {
                impl From<#struct_identifier> for std::collections::HashMap<String, String> {
                    fn from(value: #struct_identifier) -> Self {
                        let mut hash_map = std::collections::HashMap::<String, String>::new();

                        #(
                            hash_map.insert(stringify!(#field_identifiers).to_string(), String::from(value.#field_identifiers));
                        )*

                        hash_map
                    }
                }
            }
        }
        _ => unimplemented!()
    }.into()
}
//main.rs
#[derive(IntoHashMap)]
pub struct User {
    username: String,
    first_name: String,
    last_name: String,
}

fn main() {
    let user = User {
        username: "username".to_string(),
        first_name: "First".to_string(),
        last_name: "Last".to_string(),
    };

    let hash_map = std::collections::HashMap::<String, String>::from(user);

    dbg!(hash_map);
}


```

## 参考

 [宏小册](https://zjp-cn.github.io/tlborm/introduction.html)

[Procedural Macros in Rust – A Handbook for Beginners](https://www.freecodecamp.org/news/procedural-macros-in-rust/)
