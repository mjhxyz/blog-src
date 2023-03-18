---
title: python神奇的迭代
date: 2020-09-25 04:46:43
tags:
- python
categories:
- python

---

# Python 神奇的迭代

## 迭代器使用

迭代器基本使用，就如下代码一样

```python
a = [1, 2, 3, 4, 5]
for n in a:
    print(n)
```

## for in 原理

使用 `for n in xx` 时

- 要求循环迭代的对象必须得是 `collections.Iterable`(可迭代对象) 的一个实例
- 然后使用内置方法 `inter(xx)` 获取到`迭代器`由可迭代器对象生成
- 本质上调用了 `xx.__iter__()` 方法, 这也是 `Iterable` 的抽象方法
- 拿到的迭代器才是真正迭代
- 传入 next 中调用 `next(it)` 则会调用 `it.__next__()` 方法
- 当 `next(it)` 一直调用到没有元素的时候，会抛出 `StopIteration` 异常

注意的是

- 迭代器只能用一次，下次用重新生成
- 两个迭代器之间没有干扰
- 迭代器也可以用 `for` 遍历,说明迭代器对象也是可迭代对象，只不过返回了 `self`

## 撸一个可迭代对象

场景:

用户对象有自己的一些属性，比如说姓名，年龄，性别。现在希望能对用户对象遍历，获取用户对象的相关信息

```python
class User:
    def __init__(self, name, age, gender):
        self.name = name
        self.age = age
        self.gender = gender
        self.index = 0

    def __iter__(self):
        self.index = 0
        return self

    def __next__(self):
        self.index += 1
        if self.index > 3:
            raise StopIteration
        if self.index == 1:
            return self.name
        elif self.index == 2:
            return self.age
        else:
            return self.gender


a = User('mao', 18, 'n')
for n in a:
    print(n)
        
```

相当于用户对象自己是可迭代对象，同时也是迭代器，当然创建一个专门当迭代器的类也可以。



上面的代码是我们手动使用了一个 `index` 变量来维护迭代器。也可以用生成器来自动维护得带器 如下代码:

```python
class User:
    def __init__(self, name, age, gender):
        self.name = name
        self.age = age
        self.gender = gender

    def __iter__(self):
        yield self.name
        yield self.age
        yield self.gender
```

## 反向迭代

### 基本概念

有一个变量 `a = [1, 2, 3, 4, 5]`,要对这个变量进行反向迭代操作

- 使用 `a.reverse()` 方法,将 a 逆序再迭代，但修改了源数组
- 使用切片操作 `a[::-1]` 得到新的逆序列表，再迭代,但浪费的空间
- 正常操作需要使用内置函数 `reversed`:
```python
k = reversed(a)  # 会得到反向迭代器对象,和 inter() 刚好想法

# 但传入 reversed 的函数的实例必须实现 __reversed__ 方法
```

### 整一个支持反向迭代的类

直接在上面的普通迭代对象加一个 `__reversed__` 方法即可

```python
class User:
    def __init__(self, name, age, gender):
        self.name = name
        self.age = age
        self.gender = gender

    def __iter__(self):
        yield self.name
        yield self.age
        yield self.gender

    def __reversed__(self):
        yield self.gender
        yield self.age
        yield self.name


u = User('mao', 18, 'n')
for v in reversed(u):
    print(v)
```

## Iterable 进行切片操作

### 切片操作使用

```python
a = [1, 2, 3, 4, 5, 6]

for n in a[2:5]:
    print(n)
# 输出 3 4 5
```

### 切片实现

- `[]` 运算符重载的方法是 `__getitem__` 即 `a[2]` <=> `a.__getitem__`
- `[]` 切片也是一样的, `a[2:4:1]` <=> `a.__getitem__(slice(2,4[,1]))`
    - `slice` 是 python 的一个内置函数
    - 参数为 起始值，结束值，步进
- 使用 `itertools.islice()` 方法, 将一个`可迭代`对象转换为一个`切片对象`
- 切片对象也可迭代


```python
def counter(n):
    while True:
        yield n
        n += 1


from itertools import islice

s = islice(counter(0), 100, 200)

for i in s:
    print(i)
# 输出 100 - 199 的数
```

**islice 原理**

- 从最前开始依次读取数据，但不返回数据
- 直到`起始值`的时候，才开始返回结果
- 超过了`结束值`,就结束


### 实现一个 islice


```python
def counter(n):
    while True:
        yield n
        n += 1


def mslice(it, s, e, j=1):
    cur = 0
    while cur < e:
        d = next(it)
        if cur >= s:
            # 此时 s <= cur < s 符合要求
            yield d
        cur += 1


s = mslice(counter(0), 100, 200)

for i in s:
    print(i)
# 一样输出 100 - 199 的数
```