---
title: Rust std(3) string
author: rvyou
date: 2021-09-17 19:22:10 +0800
categories: [Rust,std,string]
tags: [Rust,std,string]
---

### use std :: string :: String

- *with_capacity(capacity: usize) -> String  创建一个指定容量的string

- into_raw_parts(self) -> (*mut u8, usize, usize)   分解一个string内部数据返回

- into_bytes(self) -> Vec<u8, Global>  字符串转换u8 vector

- *as_str(&self) -> &str   返回借用

- *as_mut_str(&mut self) -> &mut str   返回可变借用

- *push_str(&mut self, string: &str)     追加字符

- extend_from_within<R>(&mut self, src: R)    追加指定字符(extend_from_within(2..))

- *capacity(&self) -> usize   返回字符串cap

- *reserve(&mut self, additional: usize)   重新设置 cap(最后比原来的大)

- reserve_exact(&mut self, additional: usize)  额外追加 cap

- try_reserve(&mut self, additional: usize) -> Result<(), TryReserveError>  额外追加 self.len() + additional 不大于cap 不返回错误

- try_reserve_exact( &mut self, additional: usize ) -> Result<(), TryReserveError>

- *shrink_to_fit(&mut self)  cap缩容到len长度 shrink_to(&mut self, min_capacity: usize)

- truncate(&mut self, new_len: usize) 缩容到指定长度 截取掉

- *push(&mut self, ch: char)  插入字符

- *pop(&mut self) -> Option<char> 弹出字符

- *as_bytes(&self) -> &[u8]  转换bytes

- remove(&mut self, idx: usize) -> char 返回指定下标字符

- *retain<F>(&mut self, f: F)   保留返回 闭包true 字符

- *insert(&mut self, idx: usize, ch: char)  指定赋值下标字符

- insert_str(&mut self, idx: usize, string: &str)

- *len(&self) -> usize 长度

- *is_empty(&self) -> bool   如果该字符串的长度为0，则返回true，否则返回false。

- split_off(&mut self, at: usize) -> String    保留 [0, at)  返回 [at, len)

- clear(&mut self) 清空 len 长度

- drain<R>(&mut self, range: R) -> Drain<'_>  迭代器删除指定字符 .collect(); 执行

- replace_range<R>(&mut self, range: R, replace_with: &str)  查找替换字符串

- into_boxed_str(self) -> Box<str, Global>  字符串装箱

### str(部分 有些重复的就不写了)

- is_char_boundary(&self, index: usize) -> bool 检查指定字符是否符合UTF-8 code

- as_ptr(&self) -> *const u8  原始u8 指针

- get<I>(&self, i: I) -> Option<&<I as SliceIndex<str>>::Output>  获取借用

- get_mut<I>( &mut self, i: I ) -> Option<&mut <I as SliceIndex<str>>::Output>  获取可变借用

- split_at(&self, mid: usize) -> (&str, &str) 分割字符串

- chars(&self) -> Chars<'_>   字符切片迭代器

- char_indices(&self) -> CharIndices<'_>  字符与位置迭代器

- bytes(&self) -> Bytes<'_>  二进制迭代器

- split_whitespace(&self) -> SplitWhitespace<'_> 空格迭代

- split_ascii_whitespace(&self) -> SplitAsciiWhitespace<'_>  单空格

- lines(&self) -> Lines<'_> 换行符迭代

- contains<'a, P>(&'a self, pat: P) -> bool  匹配是否一致

- starts_with<'a, P>(&'a self, pat: P) -> bool  匹配开始的字符

- ends_with<'a, P>(&'a self, pat: P) -> bool  结束的字符

- find<'a, P>(&'a self, pat: P) -> Option<usize>  查找字符 返回位置

- rfind<'a, P>(&'a self, pat: P) -> Option<usize> 从后面开始找

- split<'a, P>(&'a self, pat: P) -> Split<'a, P> 字符串分割 let v: Vec<&str> = "Mary had a little lamb".split(' ').collect();

- split_inclusive<'a, P>(&'a self, pat: P) -> SplitInclusive<'a, P> 包含分隔符

- split_terminator<'a, P>(&'a self, pat: P) -> SplitTerminator<'a, P>  去空

- splitn<'a, P>(&'a self, n: usize, pat: P) -> SplitN<'a, P>  查找几次

- split_once<'a, P>(&'a self, delimiter: P) -> Option<(&'a str, &'a str)>  查找一次

- matches<'a, P>(&'a self, pat: P) -> Matches<'a, P> 迭代指定字符 pat 返回都是pat

- match_indices<'a, P>(&'a self, pat: P) -> MatchIndices<'a, P>  返回位置信息

- trim(&self) -> &str  去空

- trim_matches<'a, P>(&'a self, pat: P) -> &'a str  去除指定字符

- strip_prefix<'a, P>(&'a self, prefix: P) -> Option<&'a str  去除指定前缀

- parse<F>(&self) -> Result<F, <F as FromStr>::Err> 转换类型

- is_ascii(&self) -> bool  是否是ascii

- eq_ignore_ascii_case(&self, other: &str) -> bool  大小写忽略匹配

- replace<'a, P>(&'a self, from: P, to: &str) -> String  查找替换返回string

- replacen<'a, P>(&'a self, pat: P, to: &str, count: usize) -> String  查找替换加次数

- to_lowercase(&self) -> String 小写

- to_uppercase(&self) -> String 大写

- repeat(&self, n: usize) -> String 重复字符串 多少次
