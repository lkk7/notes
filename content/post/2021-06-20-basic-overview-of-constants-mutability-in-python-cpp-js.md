+++
title = "Basic overview of constants/mutability in Python, C++ and JavaScript"
date = 2021-04-21T00:00:00+02:00
description = "Constants/mutability in Python, C++ and JavaScript"
tags = ["python", "c++", "javascript"]
aplayer = false
showLicense = false
+++

Here I compare the concept of constants, mutability and things around it – across Python, C++ and JavaScript. Of course, this note does not describe the topic in its entirety.

<!--more-->
**(For Python, type hinting in every code snippet has been omitted for simplicity)**
## Python
### Data model, objects
Python's reference [1] sums up its concept of objects in a concise way:\
*"Objects are Python’s abstraction for data. All data in a Python program is represented by objects or by relations between objects. Every object has an identity, a type and a value. An object’s identity never changes once it has been created; you may think of it as the object’s address in memory. The `is` operator compares the identity of two objects; the `id()` function returns an integer representing its identity. [...]\
The value of some objects can change. Objects whose value can change are said to be mutable; objects whose value is unchangeable once they are created are called immutable."*

We can check that every object has its identity, represented by an ID. Note that this is guaranteed to be a memory address in CPython, but not in other implementations.
```python
>>> number = 1
>>> id(number)
4402092336
```
### Constants, mutability
There is no concept of `const` in Python, so we cannot simply define a `const` variable like in many other languages.
However, as said in earlier section, objects can be mutable or immutable.\
Let's try to modify an object of type `list` (it is mutable) and see if it affects its identity.
```python
>>> numbers = [1, 2, 3, 4, 5]
>>> id(numbers)
4404611136
>>> numbers[0] = 123
>>> numbers
[123, 2, 3, 4, 5]
>>> id(numbers)
4404611136
```
As we see, modifying a mutable object does not change its identity.
Let's try to do the same thing with an object of type `str`, which is immutable.
```python
>>> letters = "abcdefg"
>>> letters[0] = "!"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'str' object does not support item assignment
```
We cannot modify immutable objects in-place. If we want to modify a variable which holds a string, we have to assign a new string object to that variable.
```python
>>> letters = "abcdefg"
>>> id(letters)
4404659376
>>> # Replace the occurence of 'a' with '!' by assigning a new string here
>>> letters = letters.replace('a', '!', 1)
>>> id(letters)
4404660336
```
Here are the essential data types divided on their mutability:
- Immutable: `str`, `tuple`, `int`, `float`, `frozenset` (an immutable `set`), `bytes`
- Mutable: `list`, `dict`, `set`, `bytearray` (a mutable version of `bytes`), user-defined classes.

If an immutable sequence like a `tuple` contains a mutable object (or really a reference to it) like a `list`, then this list can be modified in-place. We can't modify the reference itself in the sequence, but we can modify the object it points to.
```python
>>> immutable = (1, 2, 3, [100, 200, 300])
>>> id(immutable[3])
4302923264
>>> immutable[3] = [10, 20, 30] # We can't modify the reference
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
>>> immutable[3][2] = 999 # But we can get what it points to and modify that
>>> id(immutable[3])
4302923264
>>> immutable
(1, 2, 3, [100, 200, 999])
```
This points us to the fact that "immutability" doesn't really mean "constness".
We can't change the immutable object itself, but if it holds references to something mutable, we can modify that thing being pointed to.\
Moreover, we can just change what the variable holds.
```python
>>> immutable = (1, 2, 3) # Variable points to an immutable object
>>> immutable = (3, 2, 1) # So what, we can just change that
```

### Creating "constants" in Python
We can emulate a constant like this:
```python
>>> def GET_X():
...     return "a1scdf8sfacd4cl"
... 
>>> GET_X()
'a1scdf8sfacd4cl'
>>> GET_X() = 100
  File "<stdin>", line 1
    GET_X() = 100
    ^
SyntaxError: cannot assign to function call
```
But it requires us to remember about the function call parentheses. Also, it has the overhead of a function.\
For classes, we can create read-only properties.
```python
>>> class Something:
...     def __init__(self, x):
...         self._x = x
...     @property
...     def x(self):
...         return self._x
...
>>> something = Something(1)
>>> something.x
1
>>> something.x = 2
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
AttributeError: can't set attribute
```
### An interesting case
Python's reference [1] says:\
*Default parameter values are evaluated from left to right when the function definition is executed. This means that the expression is evaluated once, when the function is defined, and that the same “pre-computed” value is used for each call. This is especially important to understand when a default parameter is a mutable object, such as a list or a dictionary: if the function modifies the object (e.g. by appending an item to a list), the default value is in effect modified.*

That means that this is possible:
```python
>>> def print_mirrored(numbers=[1,2,3]):
...     numbers += numbers[::-1]
...     print(numbers)
... 
>>> print_mirrored()
[1, 2, 3, 3, 2, 1]
>>> print_mirrored()
[1, 2, 3, 3, 2, 1, 1, 2, 3, 3, 2, 1]
```
We see that changes to the default `numbers` parameter are saved between calls. If we don't want that effect, we can choose to use `None`:
```python
>>> def print_mirrored(numbers=None):
...     if numbers is None:
...         numbers = [1, 2, 3]
...     numbers += numbers[::-1]
...     print(numbers)
... 
>>> print_mirrored()
[1, 2, 3, 3, 2, 1]
>>> print_mirrored()
[1, 2, 3, 3, 2, 1]
```

## C++
### Const type qualifier and const-qualified member functions
In contrast to Python, we can make objects constant and we do that by using a constant type qualifier. Cppreference [2] tells us what is a const object:\
*An object whose type is const-qualified, or a non-mutable subobject of a const object. Such object cannot be modified: attempt to do so directly is a compile-time error, and attempt to do so indirectly (e.g., by modifying the const object through a reference or pointer to non-const type) results in undefined behavior.*

We declare the constness together with the type by the `const` keyword:
```cpp
const int a = 0;
// a = 10;       // Error
```
But the `const` keyword can also be used to const-qualify a member function (a function that is a member of a class) [2]. In such case, inside the body of that function `*this` is seen as const-qualified. Thus, we can't modify any other members here and only member functions that are const-qualified can be called here.
```cpp
class Something {
    int member = 1;
    void modify_member();
    void modify_member_const() const;
    void non_const_function() {}
    void const_function() const {};
};

void Something::modify_member() {
    member = 2;           // Will work
    non_const_function(); // Will work
}

// Will not work
void Something::modify_member_const() const {
    const_function();        // Will work
    // non_const_function(); // Won't work (error)
    // member = 2;           // Won't work (error)
}
```
There are some tools that make the compiler "forget" for a while that something is const in a specific context.

The first one is a `mutable` keyword. When we declare a class member as `mutable`, it can be modified even in const-qualified member functions.
```cpp
class Something {
    int member = 1;
    mutable int mutable_member = 1;
    void const_function() const;
};

void Something::const_function() const{
    mutable_member = 2; // Will work
    // member = 2;         // Won't work (error)
}
```
As cppreference says [2]: *Mutable is used to specify that the member does not affect the externally visible state of the class*. Example usage includes mutexes.

Another tool used to "forget" the constness of something is `const_cast`. It can be used to remove constness from const pointers/references that point to a non-const object.
```cpp
int i = 1;
const int* ptr = &i;
*const_cast<int*>(ptr) = 2;
```
**If the object (`i`) pointed to was actually `const`, this would lead to undefined behavior.** `const_cast` is used rather rarely.

### constexpr
There's also the great `constexpr`, about which cppreference [2] says:\
*The constexpr specifier declares that it is possible to evaluate the value of the function or variable at compile time. Such variables and functions can then be used where only compile time constant expressions are allowed (provided that appropriate function arguments are given).*
The "at-compile-time" thing is still evolving, with `consteval` and `constinit` coming in the future. That's why I will probably focus on these things in a different post.

### An interesting case
Const declarations might get tricky when using type aliases.
```cpp
using int_ptr = int*;

int x = 1;
const int* first = &x;     // const int* as expected
const int_ptr second = &x; // This is actually int* const
```
The `first` pointer will be of type `const int*` (non-const pointer to a const int). However, compiler will make the `second` one a `int* const` (const pointer to a non-const int). Why is this the case? It's because the `second` pointer gets evaluated to `int_ptr const second`, which gives `int* const second`.

### A Good FAQ about constness in C++
https://isocpp.org/wiki/faq/const-correctness

## JavaScript
### Constants, mutability
All primitive data types (string, number, bigint, boolean, undefined, symbol, null) are immutable [3]. Objects and arrays are mutable. As in Python, variables pointing to immutable data can just start pointing to a new value.
```js
let immutable = "x";   // the "x" is immutable
immutable = "y";       // but we can just abandon the "x"

let x = {a: 1};
x.a = 100;             // We can modify an object
console.log(x);        // {a: 100}
```
ES6 introduced the 'const' keyword for variables. This prevents reassignment:
```js
const value = 1;
value = 2;
// Uncaught TypeError: Assignment to constant variable.
```
We have to initialize a `const` variable immediately:
```js
const x; // Uncaught SyntaxError: Missing initializer in const declaration
```
`const`, just like `let`, has block scope:
```js
const x = 1;
{
    const x = 2;
    console.log(x); // 2
}
console.log(x);     // 1
```
Important: `const` does not make the object pointed to immutable – it just prevents reassignment:
```js
const x = {a: 1};
x.a = 100;
console.log(x); // {a: 100}
```
To make an object immutable, we can use `Object.freeze`. This does not work on nested objects:
```js
let x = {a: 1, nested: {b: 1}};
Object.freeze(x);
x.a = 2;
x.new = "new";
x.nested.b = 2;
console.log(x); // {a: 1, nested: {b: 2}}
```
As we can see, this situation is very similar to Python's case (immutable `tuple` holding a mutable `list`).
### An interesting case
Function declarations get hoisted to current scope's top, so the function can be used before the declaration.
```js
console.log(linearFunction(2, 1, 10)); // 21
function linearFunction(a, b, x) {
    return a * x + b;
}
```
A `const` arrow function prevents this (because it is block-scoped) while also preventing reassignment:
```js
console.log(linearFunction(2, 1, 10));          // Error
const linearFunction = (a, b, x) => a * x + b;
console.log(linearFunction(2, 1, 10));          // 21
linearFunction = () => {}                       // Error
```
Another option is a mix of these two options:
```js
console.log(linearFunction(2, 1, 10));                    // Error  
const linearFunction = function linearFunction(a, b, x) {
    return a * x + b;
}
console.log(linearFunction(2, 1, 10));                    // 21
linearFunction = function linearFunction(a, b, x) {};     // Error
```
Which approach is better depends on the situation. Arrow functions have their limitations [as seen here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions).

## Sources:
- [1] [https://docs.python.org/3/reference](https://docs.python.org/3/reference) Python Software Foundation. The Python Language Reference
- [2] [https://en.cppreference.com](https://en.cppreference.com) cppreference.com – C and C++ reference
- [3] [https://developer.mozilla.org](https://developer.mozilla.org) – MDN Web Docs
