---
title: Higher Kind Type
date: 2023/01/25 17:15
excerpt: 小孩子不懂事，随手写的 HKT
---

随手写着玩的 HKT

```rust
pub trait HKT {  
    type Inner;  
    type With<T>: HKT<Inner = T>;  
}  
  
pub trait Functor: HKT {  
    fn fmap<T, F>(&self, f: F) -> Self::With<T>  
    where  
        F: Fn(&Self::Inner) -> T;  
}  
  
pub trait Applicative : Functor {  
    fn pure(a: Self::Inner) -> Self;  
    fn ap<I, O>(&self, i: Self::With<I>) -> Self::With<O>  
        where  
            Self::Inner: Fn(&I) -> O;  
}  
  
pub trait Monad: Applicative {  
    fn bind<O, F>(&self, f: F) -> Self::With<O>  
    where  
        F: Fn(&Self::Inner) -> Self::With<O>;  
}  
  
impl<T> HKT for Option<T> {  
    type Inner = T;  
    type With<U> = Option<U>;  
}  
  
impl<T> Functor for Option<T> {  
    fn fmap<U, F>(&self, f: F) -> Self::With<U>  
    where  
        F: Fn(&Self::Inner) -> U,  
    {  
        match self {  
            Some(x) => Some(f(x)),  
            None => None,  
        }  
    }  
}  
  
impl<T> Applicative for Option<T> {  
    fn pure(a: Self::Inner) -> Self {  
        Some(a)  
    }  
  
    fn ap<I, O>(&self, i: Self::With<I>) -> Self::With<O>  
    where  
        Self::Inner: Fn(&I) -> O,  
    {  
        match self {  
            Some(f) => match i {  
                Some(x) => Some(f(&x)),  
                None => None,  
            }  
            None => None,  
        }  
    }  
}  
  
impl<T> Monad for Option<T> {  
    fn bind<U, F>(&self, f: F) -> Self::With<U>  
    where  
        F: Fn(&Self::Inner) -> Self::With<U>,  
    {  
        match self {  
            Some(x) => f(&x),  
            None => None,  
        }  
    }  
}  
  
impl<T> HKT for Vec<T> {  
    type Inner = T;  
    type With<U> = Vec<U>;  
}  
  
impl<T> Functor for Vec<T> {  
    fn fmap<U, F>(&self, f: F) -> Self::With<U>  
    where  
        F: Fn(&Self::Inner) -> U,  
    {  
        let mut v = Vec::new();  
  
        for x in self {  
            v.push(f(x));  
        }  
  
        v  
    }  
}  
  
impl<T> Applicative for Vec<T> {  
    fn pure(a: Self::Inner) -> Self {  
        vec![a]  
    }  
  
    fn ap<I, O>(&self, i: Self::With<I>) -> Self::With<O>  
    where  
        Self::Inner: Fn(&I) -> O,  
    {  
        let mut v = Vec::new();  
  
        for f in self {  
            for x in &i {  
                v.push(f(x));  
            }  
        }  
  
        v  
    }  
}  
  
impl<T> Monad for Vec<T> {  
    fn bind<U, F>(&self, f: F) -> Self::With<U>  
    where  
        F: Fn(&Self::Inner) -> Self::With<U>,  
    {  
        let mut v = Vec::new();  
  
        for x in self {  
            v.extend(f(x));  
        }  
  
        v  
    }  
}
```