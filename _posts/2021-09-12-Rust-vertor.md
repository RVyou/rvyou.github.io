---
title: Rust std(2) vertor
author: rvyou
date: 2021-09-12 13:32:00 +0800
categories: [Rust,std,vertor]
tags: [Rust,std,vertor]
---

## 常用

### with_capacity(capacity: usize) -> Vec<T, Global>

设置 cap 返回 vec

### capacity(&self) -> usize

获取容量 cap

### append(&mut self, other: &mut Vec<T, A>)

### pop(&mut self) -> Option<T>

### push(&mut self, value: T)

### reserve(&mut self, additional: usize)

### clear(&mut self)

### len(&self) -> usize

### is_empty(&self) -> bool

### try_reserve(&mut self, additional: usize) -> Result<(), TryReserveError>

 为在给定的Vec<T>中插入至少更多的元素保留容量.大于self.len() + reserve， 如果容量已经足够，则什么也不做

### reserve_exact(&mut self, additional: usize)

### try_reserve_exact( &mut self, additional: usize ) -> Result<(), TryReserveError>

预留最小容量 reserve_exact，在调用 reserve_exact 之后，容量将大于或等于 self.len() + reserve_exact， 如果容量已经足够，则什么也不做。

### shrink_to_fit (&mut self)

缩容(由系统决定缩容值)

### shrink_to (&mut self, min_capacity: usize )

指定缩容，如果没有足够的空间就不做操作

### into_boxed_slice (self) -> Box < [ T ] , A>

 将向量转换为Box<[T]>. 请注意，这将减少任何多余的容量。

### truncate(&mut self, len: usize)

保留第一个len元素并删除其余元素,如果len大于向量的当前长度，则无效。cap不影响

### as_slice (&self) -> &[T]

提取包含整个 vec 的切片。相当于`&s[..]`

### as_mut_slice (&mut self) -> &mut [T]

提取整个向量的可变切片。 相当于`&mut s[..]`

### swap_remove(&mut self, index: usize) -> T

o(1) 不保留排序，index 超出范围会 painc

### remove(&mut self, index: usize) -> T

移除并返回矢量中索引位置的元素，将其之后的所有元素向左移动。O(n)

### insert(&mut self, index: usize, element: T)

指定位置插入元素

### retain<F>(&mut self, f: F) where F: FnMut(&T) -> bool

遍历元素 根据函数返回 true 进行保留

### retain_mut<F>(&mut self, f: F) where F: FnMut(&mut T) -> bool

遍历元素 跟 retain 一样true 保留

### dedup_by_key<F, K>(&mut self, key: F) where F: FnMut(&mut T) -> K, K: PartialEq<K>

删除向量中解析为相同键的所有连续元素，但第一个元素除外。

如果向量已排序，则会删除所有重复项(连续元素)。

### dedup_by(&mut self, same_bucket: F) where F: FnMut(&mut T, &mut T) -> bool

( next , current) true 删除 next ，flase 保留

### drain<R>(&mut self, range: R) -> Drain<'_, T, A>ⓘ where R: RangeBounds<usize>

从向量中批量移除指定的范围，并将所有移除的元素作为一个迭代器返回。如果迭代器在被完全消耗之前被丢弃，它会丢弃剩余的移除元素。

```rust
let u: Vec<_> = v.drain(1..).collect();
```

### split_off(&mut self, at: usize) -> Vec<T, A>

返回一个新分配的向量，包含范围[at, len]内的元素。调用后，原向量将被留下，包含元素[0, at]，其先前的容量没有改变 。

### resize(&mut self, new_len: usize, value: T)

调整Vec的大小，使len等于new_len

如果new_len大于len，则Vec被扩展，每一个额外的槽被填满。

如果new_len小于len，那么Vec就被简单地截断了。

### resize_with<F>(&mut self, new_len: usize, f: F) where F: FnMut() -> T

如果new_len大于len，那么Vec被扩展，每一个额外的槽都被调用闭包f的结果所填充。

如果new_len小于len，Vec将被简单地截断。

Default::default 可以作为第二个参数

### leak<'a>(self) -> &'a mut [T]

泄漏可变vec。从Rust 1.57开始，这个方法不会重新分配或缩小Vec，所以泄露的分配可能包括未使用的容量，而这些容量不属于返回的片断。

### spare_capacity_mut(&mut self) -> &mut [MaybeUninit<T>]

返回cap容量可修改，可以配合set_len 使用

### as_ptr (&self) -> *const T

### as_mut_ptr(&mut self) -> *mut T

###

## 不常用

### unsafe fn from_raw_parts( ptr: *mut T, length: usize, capacity: usize ) -> Vec<T, Global>

unsafe  根据 数据源地址，len ，cap 进行设置 vec

### unsafe fn set_len(&mut self, new_len: usize)

必须小于等于 cap

### leak<'a>(self) -> &'a mut [T]

消耗并泄露Vec，返回对其内容的可变引用，&'a mut [T]。注意，类型T必须超过所选择的生命周期'a'。如果该类型只有静态引用，或者根本没有，那么可以选择 "静态"。

从Rust 1.57开始，这个方法不会重新分配或缩小Vec，所以泄露的分配可能包括未使用的容量，而这些容量不属于返回的片断。

### spare_capacity_mut(&mut self) -> &mut [MaybeUninit<T>]

返回向量的剩余容量，作为 MaybeUninit<T>的一个片断。

在使用set_len方法将数据标记为初始化之前，返回的片断可以用来向向量中填充数据（例如从文件中读取）。

```rust
// Allocate vector big enough for 10 elements.
let mut v = Vec::with_capacity(10);

// Fill in the first 3 elements.
let uninit = v.spare_capacity_mut();
uninit[0].write(0);
uninit[1].write(1);
uninit[2].write(2);

// Mark the first 3 elements of the vector as being initialized.
unsafe {
    v.set_len(3);
}

assert_eq!(&v, &[0, 1, 2]);
```

### split_at_spare_mut(&mut self) -> (&mut [T], &mut [MaybeUninit<T>])

将vec的内容作为T的一个片断返回，同时将vec的剩余容量作为 MaybeUninit<T>的一个片断返回。

### extend_from_slice(&mut self, other: &[T])

遍历vec元素追到调用的vec里
