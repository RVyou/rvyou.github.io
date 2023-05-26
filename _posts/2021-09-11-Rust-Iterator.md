---
title: Rust std(1) Iterator
author: rvyou
date: 2021-09-11 13:32:00 +0800
categories: [Rust,std,Iterator]
tags: [Rust,std,Iterator]
---

这几天写算法题发现自己对标准库熟悉度是非常的低。所以加强一下。

## 常用

### next

迭代器的下一个元素返回 Option

```rust
let i = [1, 2, 3];
let s = vec!["1","2","3"];
let v:Vec<i32> = Vec::new();
println!("{:?}--{:?}--{:?}",i.iter().next(),s.iter().next(),v.iter().next());
//Some(1)--Some("1")--None
```

### collect

执行迭代器

### nth

返回迭代器第n个元素，且迭代器也会从这里进行 next

```rust
let s = vec!["1","2","3"];
let mut si = s.iter();
println!("{:?}",si.nth(1));
println!("{:?}",si.next());
//Some("2")
//Some("3")
```

### skip

迭代器跳到指定位置，跟 nth 的区别是不返回跳到的元素

### rev

反向迭代

### take

创建一个迭代器 指定元素迭代最大个数

### scan

一个初始值，它是内部状态的种子，以及一个有两个参数的闭包，第一个是对内部状态的可变引用，第二个是迭代器元素

### chain

链接2个迭代器形成链表

```rust
let s1 = &[1, 2, 3];
let s2 = &[4, 5, 6];

let mut iter = s1.iter().chain(s2);

assert_eq!(iter.next(), Some(&1));
assert_eq!(iter.next(), Some(&2));
assert_eq!(iter.next(), Some(&3));
assert_eq!(iter.next(), Some(&4));
```

### zip

链接2个迭代器形成同时迭代，任何一个迭代器多出来元素忽略掉

```rust
let a1 = [1, 2, 3,7];
let a2 = [4, 5, 6];

let mut iter = a1.iter().zip(a2.iter());

assert_eq!(iter.next(), Some((&1, &4)));
assert_eq!(iter.next(), Some((&2, &5)));
assert_eq!(iter.next(), Some((&3, &6)));
println!("{:?}",iter.next());
//None
```

### unzip

```rust
let a = [(1, 2), (3, 4), (5, 6)];

let (left, right): (Vec<_>, Vec<_>) = a.iter().cloned().unzip();

assert_eq!(left, [1, 3, 5]);
assert_eq!(right, [2, 4, 6]);

// you can also unzip multiple nested tuples at once
let a = [(1, (2, 3)), (4, (5, 6))];

let (x, (y, z)): (Vec<_>, (Vec<_>, Vec<_>)) = a.iter().cloned().unzip();
assert_eq!(x, [1, 4]);
assert_eq!(y, [2, 5]);
assert_eq!(z, [3, 6]);
```

### copied

实现 copy 进行复制

### cloned

实现 clone   进行复制

### map

对迭代器进行闭包调用，惰性的

### flat_map

map 的闭包为每个元素返回一个处理后的元素，而 flat_map() 的闭包为每个元素返回一个迭代器。类似 map(f).flatten()

### flatten

相当于把 `2` 层迭代器或`2`可以迭代的变成一层

```rust
let d3 = [[[1, 2], [3, 4]], [[5, 6], [7, 8]]];

let d2 = d3.iter().flatten().collect::<Vec<_>>();
assert_eq!(d2, [&[1, 2], &[3, 4], &[5, 6], &[7, 8]]);

let d1 = d3.iter().flatten().flatten().collect::<Vec<_>>();
assert_eq!(d1, [&1, &2, &3, &4, &5, &6, &7, &8]);
```

### fuse

创建一个迭代器，在第一个None之后结束

### filter

迭代器闭包过滤是否输出 true

### filter_map

相当于 map().filter().map() 其实就是帮你解多一层

### enumerate

创建迭代器 返回值 (序号，值)

### peekable

创建一个迭代器 peek 和peek_mut 不消耗迭代次数 next

### skip_while

忽略闭包返回 true 的元素，第一次返回 flase 就不在调用

### take_while

返回闭包返回 true 的元素，第一次返回 flase 就不在调用

### map_while

相当于 take_while 自动解应用返回 Some(4)

### for_each

跟map 差不多，但是它不是惰性的，且无返回值

### fold

fold() 需要两个参数：一个初始值，和一个有两个参数的闭包：一个'累加器'，和一个元素。闭包返回累加器在下一次迭代时应具有的值

### reduce

reduce() 相比 fold 少了一个初始值

### max

查找迭代器最大值

### min

查找迭代器最小值

### try_fold

闭包一个初始值，以及一个有两个参数的闭包：一个 "累加器 "和一个元素。闭包要么成功返回，为下一次迭代提供累加器的值，要么返回失败，提供一个错误值。

返回错误的迭代会停止，剩下不会进行迭代，返回值为 none

```rust
let a = [10, 20, 30, 100, 40, 50];
let mut it = a.iter();
let sum = it.try_fold(0i8, |acc, &x| acc.checked_add(x));
assert_eq!(sum, None);
assert_eq!(it.len(), 2);
assert_eq!(it.next(), Some(&40));
```

### try_for_each

第一个错误处停止并返回该错误

for_each() 的易变形式，或者是 try_fold() 的无状态版本

```rust
use std::fs::rename;
use std::io::{stdout, Write};
use std::path::Path;

let data = ["no_tea.txt", "stale_bread.json", "torrential_rain.png"];

let res = data.iter().try_for_each(|x| writeln!(stdout(), "{}", x));
assert!(res.is_ok());

let mut it = data.iter().cloned();
let res = it.try_for_each(|x| rename(x, Path::new(x).with_extension("old")));
assert!(res.is_err());
// It short-circuited, so the remaining items are still in the iterator:
assert_eq!(it.next(), Some("stale_bread.json"));
```

### inspect

对迭代器的每个元素进行处理，并将值传递给它。 一般作为调试调用

### by_ref

借用一个迭代器，可变借用

### partition

通过闭包将迭代器变成 2 个集合 一个 true 元素合集， 一个 false 元素合集

is_partitioned() 和 partition_in_place(）单独返回

### all

all() 接收一个返回真或假的闭包，并执行迭代返回 全部元素为 true 返回 true (消耗迭代器次数，返回 false )

```rust
let a = [1, 2, 3];

let mut iter = a.iter();
println!("{:?}",iter.all(|&x| x != 2));
println!("{:?}",iter.next());
//false
//Some(3)
```

### any

和 all 相反 返回 flase 继续

### find

搜索一个元素，找到返回元素，闭包参数是一个双重引用

### find_map

f 提供返回值为Some(value) ，map 只能是值

相当于 iter.filter_map(f).next() 。filter_map 不执行迭代器，find_map 执行迭代器

### position

搜索迭代器的元素返回索引(会消耗迭代器)

### rposition

搜索迭代器的元素返回索引(会消耗迭代器，从右开始)

### sum

求和

### product

乘法求乘积 溢出会 painc

### cmp

两个迭代器比较是否相等 (cmp_by nightly-only)

```rust
use std::cmp::Ordering;

assert_eq!([1].iter().cmp([1].iter()), Ordering::Equal);
assert_eq!([1].iter().cmp([1, 2].iter()), Ordering::Less);
assert_eq!([1, 2].iter().cmp([1].iter()), Ordering::Greater);
```

### eq

和另一个迭代器进行比较(大小和迭代器个数相等)(eq_by  nightly-only)

### ne

和另一个迭代器进行比较 不等 返回true

### lt

小于

### le

小于等于

### ge

大于

### gt

大于等于

### is_sorted

是否已经排序 要实现 PartialOrd （nightly-only is_sorted_by is_sorted_by_key）

## 不常用

### size_hint

返回迭代器当前剩余的元素，返回类型元组(usize 当前剩余元素,usize上限)

```rust
let s = vec!["1","2","3"];
let mut si = s.iter();
si.next();
println!("{:?}",si.size_hint());
//(2, Some(2))
```

### count

返回迭代器还有多少个未迭代的元素

```rust
let s = vec!["1","2","3"];
let mut si = s.iter();
si.next();
println!("{:?}",si.count());
```

### last

消耗迭代器返回最后一个元素

```rust
let s = vec!["1","2","3"];
let mut si = s.iter();
si.next();
println!("{:?}",si.last());
//Some("3")
```

### advance_by

迭代器推进n个元素 （nightly-only）

```rust
```rust
let s = vec!["1","2","3"];
let mut si = s.iter();
si.advance_by(2);
println!("{:?}",si.next());
//Some("3")
```

```
### step_by

调整迭代器的步长(原本为1)

### intersperse_with

迭代器添加偶数分割返回

```rust
iter.intersperse_with(|| 99);
```

### try_collect

nightly-only

### max_by_key

返回最大值(多个是相同最大值是最后一个)，参数是闭包可以进行筛选

### min_by_key

跟 max_by_key 相反

### max_by

返回最大值(多个是相同最大值是最后一个)，参数是闭包可以进行筛选(闭包是2个参数)

### min_by

跟 max_by 相反

### cycle

无形循环进行迭代

###
