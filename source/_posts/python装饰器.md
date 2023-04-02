---
title: python装饰器
tags:
  - python
categories:
  - python
abbrlink: 26366
date: 2020-10-28 07:05:07
---

# 装饰器

## 基本使用

```python
def logger(func):
    def decorator(*args, **kwargs):
        print('before...')
        res = func(*args, **kwargs)
        print('after')
        return res
    return decorator


@logger
def func():
    print('func...')

func()
```


## 保存元数据

包装了以后，一些函数元数据就丢失了

- `__name__`
- `__doc__`
- `__annotations__`
- `__defaults__`
- `__closure__`

- 可以手动的进行设置，但是可能费事费力还不全
- 可以使用 `functions.update_wrapper` 方法，一次性将多个数据保留
- 可以使用 `@functools.wraps`,将元数据保留，不过底层也是 `upodate_wrapper`实现的
- 后两个方式还能使用 `__wrapper__` 属性获取原始被包装的方法

```python
def logger(func):
    def decorator(*args, **kwargs):
        print('before...')
        res = func(*args, **kwargs)
        print('after')
        return res
    return decorator


@logger
def func():
    print('func...')

func()
```

## 带参数的装饰器

- 带参数的装饰器本质上是一个装饰器工厂
- 工厂生产出来的装饰器才是真正给方法用的


```python
def a(*ty_args, **ty_kwargs):
    def decorator(func):
        def wrap(*args, **kwargs):
            return func(*args, **kwargs)
        return wrap
    return decorator
@a(1,2,3)
def xxx(a, b):
    print('func...')


xxx(1, "")
```

## 不带括号的

实现一个装饰器 `c` 可以 `@c(a=1,b=2)`使用 也可以 `@c` 使用


简单的带参装饰器:

```python
def c(a=1, b=2):
    def d(func):
        def e(*args, **kwargs):
            print(f'a={a}, b={b}')
            return func(*args, **kwargs)
        return e
    return d

@c(a=1,b=2)
def f():
    print('f')
```

这样的方式是正确的，但如果使用 `@c` 的方式，则会抛出异常

得利用装饰器的基本性质来分得

- `func` 是放在装饰器的第一个参数传进来的
- 而现在的 `c` 不是一个装饰器，而是一个生产装饰器的工厂
- 实际的所用的装饰器是 `d`
- 上述代码的本质使用应该如下:
```python
def c(a=1, b=2):
    def d(func):
        def e(*args, **kwargs):
            print(f'a={a}, b={b}')
            return func(*args, **kwargs)
        return e
    return d

def f():
    print('f')

f = c(a=2,b=3)(f)
```
- 也就是说我们可以既把 `c` 当做装饰器工厂，又把 `c` 当做装饰器本身
- 什么时候当做工厂，什么时候当装饰器，由是否传入函数可以知道

```python
def c(method=None, *, a=1, b=2):
    def d(func):
        def e(*args, **kwargs):
            print(f'a={a}, b={b}')
            return func(*args, **kwargs)
        return e
    if method is None:
        return d
    return d(method)

@c
def f():
    print('f')
```

还有另外一种更简单的写法:

```python
from functools import partial


def c(method=None, *, a=1, b=2):
    if method is None:
        return partial(c, a=a, b=b)

    def e(*args, **kwargs):
        print(f'a={a}, b={b}')
        return method(*args, **kwargs)
    return e

@c
def f():
    print('f')
```

## 装饰器类

可以先给类添加 `__call__` 方法: 

```python
class A:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print('before call...')
        res = self.func(*args, **kwargs)
        print('after call...')
        return res

@A
def f():
    print('f')
f()
```

上面的代码确实可以成功运行，如果用于其他类方法的装饰器时候就会出错:

```python
class B:
    @A
    def fb(self, k):
        print('fb')

b = B()
b.fb(1)

TypeError: fb() missing 1 required positional argument: 'k'
```

问题分析

- 异常说是缺少 `k` 这个参数，但实际上我们已经传了
- 考虑到这是个实例变量，很有可能是因为 `self` 没有自动传入成功,传入的 `1` 交给了 `self` 变量，导致 `k` 没有找到
- 进一步分析，当 `A` 装饰器实例化的到时候，得到的 `fb` 并没有携带 `self`
- 而只是单纯的 `B.fb`,所以这就导致了后面调用的时候，缺少 `self` 的问题

解决思路:

- 由于实例化 `A` 的时候，确实没有 `B` 的实例化对象，因此不应该在构造函数修改
- 再看看调用流程 `b.fb` 是 `A` 的一个实例, `b.fb()` 实际上在是在调用 调用 A的 `__call__`函数
- 而这时候的的 `a` 实例是绑定在 `b` 上面的
- 这样就好办了，此时可以考虑使用属性描述器协议，使用绑定方法进行调用
- 将当前实例 `b` 绑定到 实例(方法) `a`上面去调用, 这样每次 `a` 被调用的时候，`b` 实例都会是第一个,达到传递 `self` 的效果
- 但是为了兼容装饰的是普通函数，需要一次 `instance is Noe` 的判断

```python
class A:
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print('before call...')
        print(*args, **kwargs)
        res = self.func(*args, **kwargs)
        print('after call...')
        return res

    def __get__(self, instance, owner):
        if instance is None:
            return self
        from types import MethodType
        return MethodType(self, instance)

class B:
    @A
    def fb(self, k):
        print('fb')

b = B()
b.fb(1)
```

To be continue....