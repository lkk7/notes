+++
title = "A short note on iterables, iterators and generators in Python"
date = 2020-07-11T00:00:00+02:00
tags = ["python"]
aplayer = false
showLicense = false
+++

This subject may seem complicated: generators create generator iterators (which are iterables). To make that sentence sound understandable, here's a short note about the topic.
<!--more-->
**(Type hinting in every code snippet has been omitted for simplicity)**
### Iterables
The docs [1] briefly explain iterables:\
*An object capable of returning its members one at a time. Examples of iterables include all sequence types (such as list, str, and tuple) and some non-sequence types like dict, file objects \[...\] Iterables can be used in a for loop and in many other places where a sequence is needed (zip(), map(), …).*\
In essence, iterables are everything that you can loop over with a `for` loop.

An important detail is that they have a method `__iter__()` which returns an iterator for them (or an equivalent `iter()` built-in function).
### Iterators
An iterator is an object used to iterate over iterables. When we loop over a list with a `for` statement, the iterator is created behind the scenes and we don't have to worry about it. However, we can try to manually loop over the list with an iterator like this:
```python
numbers = [1, 2, 3]
it = iter(numbers)   # This is the iterator object
print(it.__next__()) # 1
print(it.__next__()) # 2
print(next(it))      # 3 (this is the same thing as __next__())
print(it.__next__()) # StopIteration exception
```
As we see, an iterator gets "worn out" after it has iterated over everything it had to. It throws a special StopIteration exception. We'd have to create a new iterator if we wanted to iterate again. Besides the `__next__()` method (which is synonymous to calling `next()` on the iterator, as seen in example above), the iterator should also have a method `__iter__` [1]. It just returns the iterator itself, making any iterator also an iterable (because, as I pointed out in the earlier section, every iterable has a `__next__()` method).
```python
a = iter([1, 2, 3])
print(a is a.__iter__())  # True
```
### Generators
A generator is a function which returns a generator iterator (just a special kind of iterator). This iterator is returned through usage of `yield` keyword. As the docs [1] say:\
*Each yield temporarily suspends processing, remembering the location execution state (including local variables and pending try-statements). When the generator iterator resumes, it picks up where it left off (in contrast to functions which start fresh on every invocation).*

Let's clear this up with an example:
```python
def double_range(start, stop):
    for i in range(start, stop):
	yield 2 * i
it = double_range(0, 3)  # our generator iterator, has the __next__ method
print(it)                # <generator object double at 0x10e900820>
print(it.__next__())     # 0
print(it.__next__())     # 2
print(next(it))          # 4
print(next(it))          # StopIteration exception
```
The shortest and simplest explanation is: the generator iterator returned from our `double_range` function is like a list, but we retrieve elements one at a time, when they are needed. That is called **lazy evaluation**. Subsequent calls to `__next__()` get subsequent elements. They aren't all calculated at `it = double_range(0, 3)` line. They would be calculated at definition time if we used a list:
```python
a = double_range(0, 3)
print(a)  # <generator object double_range at 0x104cd9200>
a = [2 * i for i in range(0, 3)]
print(a)  # [0, 2, 4]
```
In fact, we used list comprehension syntax there. Is there a "generator comprehension"? Yes, and we call it a generator expression:
```python
a = (2 * i for i in range(0, 3))
print(a)            # <generator object <genexpr> at 0x104cd9660>
print(next(a))      # 0
print(a.__next__()) # 2
print(next(a))      # 4
print(next(a))      # StopIteration exception
```
Definitely shorter syntax than a full function, and it does the same thing. However, it could be discussed if it's more readable. Also, do not mistake generator expressions for tuples.

### A very short note about use cases for generators
I believe that the information presented here is more than enough to start using generators. But why would we want to use them? They may seem like a worse list/tuple at first. It turns out there's a big advantage to them – memory usage. As we know now, generators calculate elements of a sequence one at a time. That may save memory and allow us to process large files, theoretically infinite sequences or "online" data streams.

The simplest example – creating infinite sequences over which we can iterate easily:
```python
def numbers():
    n = -999999999
    while True:
        yield n
        n += 1

for i in numbers():
    print(i)
# -999999999, -999999998, ...
```
## Sources:
- [1] [https://docs.python.org](https://docs.python.org) Python Software Foundation. Python Documentation