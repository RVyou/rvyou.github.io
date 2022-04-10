---
title: Rust async/await
author: rvyou
date: 2022-04-10 16:23:10
categories: [Rust,异步并发,async/await,(1)]
tags: [Rust,异步并发,async/await]
---

最近看来一篇 async/await 文章，做一下总结

```rust
#[inline(never)]
async fn x() -> usize {
    5
}
```

等价代码：

```rust
#[inline(never)]
fn x() -> impl Future<Output = usize> {
    async { 5 }
}
```

async fn 就是会返回一个 Future trait 的函数，future 是惰性的，想要运行需要 .await 或者其他运行时。

调用其 poll 方法获取 Future 的运行结果（GeneratorState） [yield](https://github.com/rust-lang/rust/blob/42313dd29b3edb0ab453a0d43d12876ec7e48ce0/library/core/src/ops/generator.rs#L70) 进行让出执行权直到状态就绪

```rust
#[lang = "from_generator"]
#[doc(hidden)]
#[unstable(feature = "gen_future", issue = "50547")]
#[rustc_const_unstable(feature = "gen_future", issue = "50547")]
#[inline]
pub const fn from_generator<T>(gen: T) -> impl Future<Output = T::Return>
where
    T: Generator<ResumeTy, Yield = ()>,
{
    #[rustc_diagnostic_item = "gen_future"]
    struct GenFuture<T: Generator<ResumeTy, Yield = ()>>(T);

    // We rely on the fact that async/await futures are immovable in order to create
    // self-referential borrows in the underlying generator.
    impl<T: Generator<ResumeTy, Yield = ()>> !Unpin for GenFuture<T> {}

    impl<T: Generator<ResumeTy, Yield = ()>> Future for GenFuture<T> {
        type Output = T::Return;
        fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
            // SAFETY: Safe because we're !Unpin + !Drop, and this is just a field projection.
            let gen = unsafe { Pin::map_unchecked_mut(self, |s| &mut s.0) };

            // Resume the generator, turning the `&mut Context` into a `NonNull` raw pointer. The
            // `.await` lowering will safely cast that back to a `&mut Context`.
            match gen.resume(ResumeTy(NonNull::from(cx).cast::<Context<'static>>())) {
                GeneratorState::Yielded(()) => Poll::Pending,
                GeneratorState::Complete(x) => Poll::Ready(x),
            }
        }
    }

    GenFuture(gen)
}
```

# 参考

- [iPotato | Rust 的 async/await 语法是怎样工作的](https://ipotato.me/article/70)
