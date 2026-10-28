# Python Notes

## Links

* [Google Dev Python](https://developers.google.com/edu/python/)
* [Python 3.6 Docs](https://docs.python.org/3.6/)
* [Stack Overflow - Python yield](https://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do)

## Quick style guide

```python
# 79 column rule
# module_name, package_name, ClassName, method_name,
# ExceptionName, function_name, GLOBAL_CONSTANT_NAME,
# global_var_name, instance_var_name, function_parameter_name,
# local_var_name
```

### Doc string

[Google example](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html)

## Basics

```python
print(__name__)
if __name__ == '__main__':
    print 'match'
```

One of the use of this functionality to write various kind of unit tests within the same module. For example, you can just import many different modules you have written and call specific functions.

### Scoping

* `global` statement can be used to indicate that particular variables live in the global scope and should be rebound there.
* `nonlocal` statement indicates that particular variables live in an enclosing scope and should be rebound there.

```python
def scope_test():
    def do_local():
        spam = "local spam"

    def do_nonlocal():
        nonlocal spam
        spam = "nonlocal spam"

    def do_global():
        global spam
        spam = "global spam"

    spam = "test spam"
    do_local()
    print("After local assignment:", spam)
    do_nonlocal()
    print("After nonlocal assignment:", spam)
    do_global()
    print("After global assignment:", spam)

scope_test()
print("In global scope:", spam)

# After local assignment: test spam
# After nonlocal assignment: nonlocal spam
# After global assignment: nonlocal spam
# In global scope: global spam
```

### Sequences

There are three sequence types:

* [list](https://docs.python.org/3.3/library/stdtypes.html#list) - these are mutable sequences that are used to store collections of homogeneous items.
* [tuple](https://docs.python.org/3.3/library/stdtypes.html#tuple) - these are immutable sequences and assignment to its members aren't possible however they may contain mutable objects like lists.
* [range](https://docs.python.org/3.3/library/stdtypes.html#range) - the `range` type is similar to Python 2.7 `xrange` where it has constant space cost - small memory usage like generators.

### Collections

* [deque](https://docs.python.org/3/library/collections.html#collections.deque) - supports fewer operations but faster for adding/removing from the head/tail of the queue - `O(1)`. See [the docs](https://docs.python.org/3/library/collections.html#collections.deque).
* [tuple](https://docs.python.org/3/builtins/stdtypes.html#tuple) - are immutable sequences, typically used to store collections of heterogeneous data (such as the 2-tuples produced by the [`enumerate()`](https://docs.python.org/3/builtins/functions.html#enumerate) built-in)
* [set](https://docs.python.org/3/builtins/stdtypes.html#set) - an unordered collection of distinct [hashable](https://docs.python.org/3/glossary.html#term-hashable) objects. Common uses include membership testing, removing duplicates from a sequence, and computing mathematical operations such as intersection, union, difference, and symmetric difference.

### Misc

```python
# a if condition else b
x = 100 if True else 0

# lists
l = [1, 2] + [2, 3]  # l += list(range(10))
l[-1]                # last element

# slicing
# sublist will include the element at the lower bound
# but exclude the element at the upper bound
sublist = list(range(1, 11))[3: -1]

# *args, **kwargs
def foo(parg, *args, **kwargs):
    print("Positional argument: {0}".format(parg))
    for i, a in enumerate(args):
        print("Arg {0}: {1}".format(i, a))
    for k, v in kwargs.items():
        print("{0}: {1}".format(k, v))

# now call the function and do cool things with unpacking
list_args = ['arg2', 'arg3', 'arg4']
dict_args = {'karg1': 1, 'karg2': 2, 'karg3': 3}
foo('pos1 arg', *list_args, **dict_args)

# print formatting
print("Float with 2 decimal places: {0:.2f}".format(10.111))
print("Decimal padded (left justified): {0:10d}".format(100))
print("Decimal padded (right justified): {0:>10d}".format(100))
print("String padded: {0:20}, end of padding.".format("Short String"))
```

### `is` vs `==`

* `is` checks that 2 arguments refer to the **same object**.
* `==` checks that 2 arguments have the **same value**.

```python
1 == True  # True
1 is True  # False

# dir(1) == dir(True)
# these return the namespace list from both instances
# both lists contain the same values
# BUT they are not the same instance
# hence why the *is* will fail

# another example
list(range(1) == list(range(1))  # True
list(range(1) is list(range(1))  # False
```

### iterables and iterators

See the [Stack Overflow post](https://stackoverflow.com/questions/9884132/what-exactly-are-pythons-iterator-iterable-and-iteration-protocols) for more explanations.

An **iterable** is:

* anything that can be looped over (i.e. you can loop over a string or file) or
* anything that can appear on the right-side of a for-loop: `for x in iterable:`
* anything you can call with `iter()` that will return an **iterator** e.g. `iter(obj)`
* an object that defines `__iter__` that returns a fresh **iterator**, or it may have a `__getitem__` method suitable for indexed lookup.
* **no** state!

An **iterator** is an **object**:

* with _state_ that remembers where it is during iteration
* with a `__next__` method that:
  * returns the next value in the iteration
  * updates the state to point at the next value
  * signals when it is done by raising `StopIteration`
  * is self-iterable (meaning that it has an `__iter__` method that returns `self`).
  * `next(iterator)` calls `__next__` on the iterator.

In a `for` loop:

```python
iterable_obj = 'abcde'  # immutable, no state
iterator_obj = iter(iterable_obj)  # this calls iterable_obj.__iter__()

# now emulate the for loop
while True:
    try:
        next(iterator_obj)  # calls iterator_obj.__next__()
    except StopIteration:
        break
```

### View objects

They provide a **dynamic** view on the dictionary's entries, which means that when the dictionary changes, the view **reflects** these changes.

See [the docs](https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects) for more info.

### LEGB-rule

`Local -> Enclosed -> Global -> Built-in`

Where the arrows should denote the direction of the namespace-hierarchy search order.

1. **Local** can be inside a function or class method, for example.
2. **Enclosed** can be its enclosing function, e.g., if a function is wrapped inside another function.
3. **Global** refers to the uppermost level of the executing script itself

### For loops

* `range(0, 10)` returns an immutable `range` type - iterator, so you need to cast it as `list(range(10))`.
* Use of other built-ins like `reversed()`, `sorted()`, `list.sort()`.

```python
num_list1 = list(range(10, 20)) # Python 3
num_list2 = list(range(20, 30))

# get index and value
for i, v in enumerate(num_list1):
    print('index: {0}, value: {1}'.format(i, v))

# iterate over both lists at the same time
for x, y in zip(num_list1, num_list2):
    print('item1: {0}, item2: {1}'.format(x, y))
```

### Copy

Assignment statements in Python **do not** copy objects, they create **bindings** between a target and an object. For collections that are **mutable or contain mutable items**, a copy is sometimes needed so one can change one copy without changing the other.

Basically:

* A **shallow copy** constructs a new compound object and then (to the extent possible) inserts references into it to the objects found in the original. This is is okay for flat mutable objects.
* A **deep copy** constructs a new compound object and then, recursively, inserts copies into it of the objects found in the original. Use this for nested lists.

```python
import copy as c

# for flat immutable objects
l1 = list(range(10))
l2 = c.copy(l1)  # create a shallow copy
l3 = l1[:]  # create a shallow copy

# for nested
l1 = [list(range(10)), 10, 11, 12]

l2 = c.copy(l1)  # shallow copy
l2[0][0] = 111111  # this also changes l1
print(l1)  # [[111111, 1, 2, 3, 4, 5, 6, 7, 8, 9], 10, 11, 12]

# solution
l1 = [list(range(10)), 10, 11, 12]
l3 = c.deepcopy(l1)  # deepcopy recursively copies objects
l3[0][0] = 111111
print(l1)  # [[0, 1, 2, 3, 4, 5, 6, 7, 8, 9], 10, 11, 12]
```

### Lists, generators and other interesting built-ins

For Python 3.x:

> Removed reduce(). Use functools.reduce() if you really need it; however, 99 percent of the time an explicit for loop is more readable.

```python
# built-in functions
q, r = divmod(9, 4)  # 9//4 == 2, 9-(4*2) == 1

# returns the concatenation of the strings in the sequence seq.
dash_delimted_str = '-'.join(['a', 'b', 'c'])

number_list = list(range(10))

# basic list comprehension
# some_list = [ <output value> for <element> in <list> <optional criteria> ]
#
squared_list = [x**x for x in number_list if x < 5]
filtered_list = [x if x < 10 else -1 for x in number_list]

# using built-in filter
def less_than_five(x):
    return x < 5

filtered_list = list(filter(less_than_five, number_list))
# using lambda
filtered_list = list(filter(lambda x: x < 5, number_list))
```

#### map, filter, reduce

```python
# filter(function, sequence)
# applies the function to each element of the sequence
def three_five(x):
    return x % 3 == 0 or x % 5 == 0

filter(three_five, range(100))
filter(lambda x: x % 3 == 0 or x % 5 == 0, range(100))

# map(function, sequence)
# calls function(element) of the sequence and returns the result as a list
def expd(x):
    return x**x

map(expd, range(10))

# functools.reduce(function, sequence)
# returns a single value constructed by calling the binary function function
# on the first two items of the sequence, then on the result and
# the next item, and so on
def sum(seq):
    def add(x, y):
        return x + y
    return reduce(add, seq)

# sum is a built-in, but this is how it's implemented
sum(range(1, 11))

def total(x, y):
     return x + y

reduce(total, range(1, 11))
```

### Dictionaries

Thank god Python 3 reduced the number of functions for iterating over a dictionary!

```python
dict.items()
dict.keys()
dict.values()

# Return an **view** over the dictionary’s (k, v) pair, keys and values.
# Remember, these are dynamic views of the dictionary itself!

# other
dict.pop(key)
dict.clear()
```

### Bit manipulation

```python
"cd".encode("hex")  # '6364'
hex(10)  # 0xa
```

### Logging

* [More from the docs](https://docs.python.org/3/howto/logging.html)

```python
import logging

logging.basicConfig(filename='logfile.log',
                    filemode='a',
                    format='%(asctime)s,%(msecs)d %(name)s '
                           '%(levelname)s %(message)s',
                    datefmt='%H:%M:%S',
                    level=logging.DEBUG)
```



## Time complexity

Some of the optimized data structures Python for better performance summarized from the [Python wiki](https://wiki.python.org/moin/TimeComplexity).

For additional information for time complexity, see my other notes: [https://cse-docs.gitbook.io/notes/fundamentals/sorting-and-searching#time-complexity](https://cse-docs.gitbook.io/notes/fundamentals/sorting-and-searching#time-complexity).

### list

| Operation        | Average Case | Amortized Worst Case |
| ---------------- | ------------ | -------------------- |
| Copy             | `O(n)`       | `O(n)`               |
| Append\[1]       | `O(1)`       | `O(1)`               |
| Pop last         | `O(1)`       | `O(1)`               |
| Pop intermediate | `O(k)`       | `O(k)`               |
| Insert           | `O(n)`       | `O(n)`               |
| Get Item         | `O(1)`       | `O(1)`               |
| Set Item         | `O(1)`       | `O(1)`               |
| Delete Item      | `O(n)`       | `O(n)`               |
| Iteration        | `O(n)`       | `O(n)`               |
| Get Slice        | `O(k)`       | `O(k)`               |
| Del Slice        | `O(n)`       | `O(n)`               |
| Set Slice        | `O(k+n)`     | `O(k+n)`             |
| Extend\[1]       | `O(k)`       | `O(k)`               |
| Sort             | `O(n log n)` | `O(n log n)`         |
| Multiply         | `O(nk)`      | `O(nk)`              |
| x in s           | `O(n)`       |                      |
| `min(s), max(s)` | `O(n)`       |                      |
| Get Length       | `O(1)`       | `O(1)`               |

### collections.deque

| Operation  | Average Case | Amortized Worst Case |
| ---------- | ------------ | -------------------- |
| copy       | `O(n)`       | `O(n)`               |
| append     | `O(1)`       | `O(1)`               |
| appendleft | `O(1)`       | `O(1)`               |
| pop        | `O(1)`       | `O(1)`               |
| popleft    | `O(1)`       | `O(1)`               |
| extend     | `O(k)`       | `O(k)`               |
| extendleft | `O(k)`       | `O(k)`               |
| remove     | `O(n)`       | `O(n)`               |

### set

| Operation                           | Average Case            | Amortized Worst Case                            | Notes                                      |
| ----------------------------------- | ----------------------- | ----------------------------------------------- | ------------------------------------------ |
| x in s                              | `O(1)`                  | `O(n)`                                          |                                            |
| Union `s\|t`                        | `O(len(s)+len(t))`      |                                                 |                                            |
| Intersection `s&t`                  | `O(min(len(s), len(t))` | `O(len(s) * len(t))`                            | replace "min" with "max" if t is not a set |
| Multiple intersection `s1&s2&..&sn` |                         | `(n-1)*O(l) where l is max(len(s1),..,len(sn))` |                                            |
| Difference s-t                      | `O(len(s))`             |                                                 |                                            |

### dict

| Operation     | Average Case | Amortized Worst Case |
| ------------- | ------------ | -------------------- |
| Copy\[2]      | O(n)         | O(n)                 |
| Get Item      | O(1)         | O(n)                 |
| Set Item\[1]  | O(1)         | O(n)                 |
| Delete Item   | O(1)         | O(n)                 |
| Iteration\[2] | O(n)         | O(n)                 |

\[1] = These operations rely on the "Amortized" part of "Amortized Worst Case". Individual actions may take surprisingly long, depending on the history of the container.

\[2] = For these operations, the worst case `n` is the _maximum size_ the container ever achieved, rather than just the current size. For example, if `N` objects are added to a dictionary, then `N-1` are deleted, the dictionary will still be sized for `N` objects (at least) until another insertion is made.

## Classes

* [Docs](https://docs.python.org/3/tutorial/classes.html)

Terms:

* data attributes: instance variables (don't need to be declared)
* method attributes

When a class definition is entered, a **new namespace** is created, and used as the local scope — thus, all assignments to local variables go into this new namespace. In particular, function definitions bind the name of the new function here.

When a class definition is left normally (via the end), a class object is created. This is basically a **wrapper** around the contents of the namespace created by the class definition. The original local scope (the one in effect just before the class definition was entered) is reinstated, and the class object is **bound** here to the **class name** given in the class definition header.

Also to note, when changing **class members**, these are remembered when creating **new instances** of that class e.g.

```python
class Foo():
    static_var = 0  # i realise this should be called class member...

print(Foo.static_var)  # 0
f = Foo()
print(f.static_var)  # 0

Foo.static_var = 100

print(f.static_var)  # 100
f.static_var = 99  # instance variable change
print(f.static_var)  # 99

ff = Foo()
print(ff.static_var)  # 100
```

### Inheritance

```python
# if an attribute is not found in the derived class namespace
# it is searched in the baseClassName and so on recursively
class DerivedClassName(modname.BaseClassName):
    class_var = 0  # class attribute

    def __init__(self):
        self.instance_var = 0

# multiple levels of inheritance
# attributes will be search DFS, left-right (simplified)
class DerivedClassName(Base1, Base2, Base3):

    # remember this will override the Base1 constructors!
    def __init__(self):
        # super can be used to access parent class members
        suer
```

* `isinstance(child, parent)`: if the child **type** is the same or some class derived from parent.
* `issubclass(child, parent)`: if child instance is a **subclass** (inheritance) of parent. e.g. `bool` is subclass of `int`.

### Private variables

Python doesn't support private variables (class-private members) but has some implementation detail and limited support for such mechanism called **name mangling**.

* `_var`: are by convention used for **semiprivate** variables for e.g. non-public api.
  * something minor, when doing `from foo import *`, `_names` are not included.
* `__var`: are by convention to indicate **private** variables.
  * they cannot be accessed normally e.g. `instance.__var` - exception
  * they are replaced with `_classname__var` to prevent **accidental** access.

### Misc

* you can make an **instance** callable - not class - by implementing the `__call__` method.

## Decorators

### Obvious ones

* `@classmethod`: function belongs to the **class** that it is in.
* `@staticmethod`: function is its **own** function - used for grouping/style e.g. like nested functions. It is useful for not polluting a module's namespace with free functions for example.

```python
class A(object):
    def foo(self,x):
        print("executing foo({0},{1})".format(self,x))

    @classmethod
    def class_foo(cls,x):
        print("executing class_foo({0},{1})".format(cls,x))

    @staticmethod
    def static_foo(x):
        print("executing static_foo({0})".format(x))
a = A()

a.foo(10)  # executing foo(<__main__.A object at 0x0000021ACE738F60>,10)
print(a.foo)  # <bound method A.foo of <__main__.A object at 0x0000021ACE738F60>>

a.class_foo(100)  # executing class_foo(<class '__main__.A'>,100)
print(a.class_foo)  # <bound method A.class_foo of <class '__main__.A'>>

a.static_foo(1000)  # executing static_foo(1000)
print(a.static_foo)  # <function A.static_foo at 0x0000021ACE73B9D8>
```

### Creating your own

Remember that decorators return the wrapped function i.e. a modified version of the function that was passed in as an argument. This can be resolved with `functools.wraps`. There are a few ways to use them:

* as classes
  * you must be aware that the `help()` may **not** work correctly
  * the workaround for decorators as wrappers is `functools.update_wrapper()`. This sets the attribute on the **instance** but `help()` looks at the class as it is always called as `help(instance)`. See more [here on stackoverflow](https://stackoverflow.com/questions/6394511/python-functools-wraps-equivalent-for-classes).
* as nested functions
* with/without arguments

Each method has a slightly different syntax that you should be aware of. Useful links:

* [Python 3 decorators - better example](https://python-3-patterns-idioms-test.readthedocs.io/en/latest/PythonDecorators.html)
* [Python 3 decorators - quick examples](https://book.pythontips.com/en/latest/decorators.html)

#### Decorators with classes

```python
import functools

class decorator_without_arguments(object):

    def __init__(self, f):
        """If there are *no* decorator arguments, the function
        to be decorated is passed to the constructor.
        """
        print("Inside __init__()")
        self.f = f
        functools.update_wrapper(self, f)  # update __name__ etc. attributes

    def __call__(self, *args, **kwargs):
        """The __call__ method is not called until the
        decorated function is called.
        """
        print("Inside __call__()")
        self.f(*args, **kwargs)
        print("After self.f(*args, **kwargs)")

@decorator_without_arguments
def sayHello(a1, a2, a3, a4):
    print('sayHello arguments:', a1, a2, a3, a4)
```

```python
from functools import wraps
# notice the __init__ doesn't contain func
# notice there are 2 decorator args vs 3 function args
class decorator_with_arguments(object):

    def __init__(self, arg1, arg2):
        """If there are decorator arguments, the function
        to be decorated is *not* passed to the constructor!
        """
        print("Inside __init__()")
        self.arg1 = arg1
        self.arg2 = arg2

    def __call__(self, f):
        """If there are decorator arguments, __call__() is only called
        once, as part of the decoration process! You can only give
        it a single argument, which is the function object.
        """
        print("Inside __call__()")
        @wraps(f)  # updates __name__, __doc__ attributes etc.
        def wrapped_f(*args, **kwargs):
            print("Inside wrapped_f()")
            print("Decorator arguments:", self.arg1, self.arg2)
            f(*args, **kwargs)
            print("After f(*args, **kwargs)")
        return wrapped_f

@decorator_with_arguments("hello", "world")
def sayHello(a1, a2, a3):
    print('sayHello arguments:', a1, a2, a3)

sayHello("say", "hello", "argument")
sayHello("a", "different", "set of args")

# Inside __init__()
# Inside __call__()  # after decoration

# Inside wrapped_f()  # sayHello is called here
# Decorator arguments: hello world
# sayHello arguments: say hello argument list
# After f(*args)  # after first sayHello() call

# Inside wrapped_f()  # sayHello is called here
# Decorator arguments: hello world 42
# sayHello arguments: a different set of args
# After f(*args)  # after second sayHello() call
```

### Decorators with Functions

```python
def decorator_without_arguments(func):
    @wraps(func)
    def wrapped_func(*args, **kwargs):
        print('Inside wrapped_func - wrapping:', func.__name__)
        func(*args, **kwargs)
        print('Finished func(*args, **kwargs)')
    return wrapped_func
```

```python
def decorator_with_arguments(arg1, arg2):
    def wrapper(func):
        @wraps(func)
        def wrapped_func(*args, **kwargs):
            print('Inside wrapped_func - wrapping:', func.__name__)
            func(*args, **kwargs)
            print('Finished func(*args, **kwargs)')
        return wrapped_func
    return wrapper
```

### Modern type hints

[Official typing reference](https://docs.python.org/3/library/typing.html)

Hints describe expected values to editors and type checkers. Python does **not** automatically validate or convert arguments, return values, or collection contents.

| Meaning                  | Modern spelling   | Available since |
| ------------------------ | ----------------- | --------------- |
| List of strings          | `list[str]`       | 3.9             |
| String keys, int values  | `dict[str, int]`  | 3.9             |
| Fixed two-item tuple     | `tuple[str, int]` | 3.9             |
| Any-length tuple of ints | `tuple[int, ...]` | 3.9             |
| Either of two types      | `str \| bytes`    | 3.10            |
| Value may be absent      | `str \| None`     | 3.10            |

Prefer these over `typing.List`, `typing.Dict`, `typing.Tuple`, `Union`, and `Optional` in modern code. The older spellings still matter when supporting older interpreters.

#### Nullable versus an optional argument

```python
# Python 3.10+
def describe(label: str | None) -> str:
    return "Unnamed" if label is None else label

def describe_default(label: str | None = None) -> str:
    return describe(label)

describe(None)       # "Unnamed"
# describe()        # TypeError: argument still required
describe_default()  # "Unnamed"
```

`T | None` allows a value of `None`; a default permits omission of the argument.

`Any` allows unchecked operations to spread through code. Prefer `object` for an unknown value when you intend to narrow its type before using it:

```python
# Python 3.10+
def normalise_label(value: object) -> str:
    if isinstance(value, str):
        return value.strip()  # Narrowed to str here.
    raise TypeError("label must be a string")

normalise_label("  Ready  ")  # "Ready"
```

#### Typed dictionaries: records with named fields

[Official TypedDict specification](https://typing.python.org/en/latest/spec/typeddict.html)

```python
# Python 3.11+; TypedDict itself arrived in 3.8.
from typing import Literal, NotRequired, TypedDict

class Delivery(TypedDict):
    reference: str
    state: Literal["pending", "sent"]
    tracking_url: NotRequired[str]

parcel: Delivery = {"reference": "P-104", "state": "pending"}
parcel.get("tracking_url")  # None
isinstance(parcel, dict)    # True: an ordinary dictionary at runtime.
```

* `Literal` restricts the values accepted by a type checker; it does not enforce them at runtime.
* `NotRequired[str]` allows the **key to be missing**. If present, its value must be a string.
* `str | None` allows a **present key with a None value**. These are different contracts.
* `Required` marks a required key inside a `TypedDict` declared with `total=False`.
* Validate external JSON separately; assigning an annotation does not check its contents.

#### Protocols: depend on capabilities

[Official protocol specification](https://typing.python.org/en/latest/spec/protocol.html)

```python
# Python 3.8+
from io import StringIO
from typing import Protocol

class TextReader(Protocol):
    def read(self) -> str: ...


def read_title(source: TextReader) -> str:
    return source.read().splitlines()[0]

read_title(StringIO("Release notes\nDetails"))  # "Release notes"
```

`StringIO` need not inherit from `TextReader`. A compatible `read()` method satisfies the contract for static checking. This lets a function accept files, in-memory streams, or test doubles without depending on a concrete class. This example assumes a non-empty document.

#### Generics and type aliases

[Python 3.12: type parameters and aliases](https://docs.python.org/3/whatsnew/3.12.html#type-parameter-syntax)

```python
# Python 3.12+
from collections.abc import Sequence

# T connects the element type of the input to the return type.
def first[T](items: Sequence[T]) -> T:
    return items[0]

first([10, 20])        # 10; inferred return type: int
first(["red", "blue"]) # "red"; inferred return type: str
# first([])           # IndexError: typing does not guarantee non-empty input.

type Coordinate = tuple[float, float]
type Lookup[T] = dict[str, T]

origin: Coordinate = (0.0, 0.0)
stock: Lookup[int] = {"pencils": 12}
```

`type Coordinate = ...` creates a type alias, not a new runtime class. Use `collections.abc` for interfaces such as `Iterable`, `Sequence`, `Mapping`, and `Callable`; use concrete types when callers need their specific operations.

Useful names to recognise, without memorising the entire typing library:

| Name        | Since | Purpose                                                        |
| ----------- | ----- | -------------------------------------------------------------- |
| `Final`     | 3.8   | Checker flags reassignment; no runtime immutability.           |
| `Annotated` | 3.9   | Attach metadata; a consuming tool decides how to interpret it. |
| `Self`      | 3.11  | Return the same class type, including subclasses.              |
| `override`  | 3.12  | Checker verifies a method overrides an inherited member.       |

`typing_extensions` backports many typing names. It cannot make new grammar such as `def first[T]` parse on an older interpreter. For older targets, use `TypeVar` and older alias syntax instead.

### Dataclasses: concise value objects

[Official dataclass reference](https://docs.python.org/3/library/dataclasses.html)

```python
# Python 3.10+; dataclass itself arrived in 3.7.
from dataclasses import dataclass, field, replace

@dataclass(frozen=True, slots=True, kw_only=True)
class Document:
    title: str
    revision: int = 1
    tags: list[str] = field(default_factory=list)

first_draft = Document(title="Runbook")
second_draft = replace(first_draft, revision=2)

first_draft.revision   # 1
second_draft.revision  # 2
Document(title="A") == Document(title="A")  # True
# first_draft.revision = 3  # FrozenInstanceError
```

| Setting           | Result                                                     |
| ----------------- | ---------------------------------------------------------- |
| `@dataclass`      | Generates initialiser, representation, and field equality. |
| `default_factory` | Calls a factory for each instance's default value.         |
| `frozen=True`     | Blocks ordinary field reassignment; shallow immutability.  |
| `slots=True`      | Generates slots; normally prevents extra instance fields.  |
| `kw_only=True`    | Generated initialiser requires field arguments by keyword. |

`first_draft.tags.append("draft")` still works: freezing the object does not freeze its list. Use `tuple[str, ...]` for immutable tags. `replace()` is shallow too: unchanged fields refer to the same objects. Annotations do not validate field types. Ordering comparisons are not generated unless requested with `order=True`.

### Argument restrictions and assignment expressions

[Python 3.8 language additions](https://docs.python.org/3/whatsnew/3.8.html#new-features)

```python
# Python 3.8+
def resize(image, /, width, *, preserve_ratio=True):
    return image, width, preserve_ratio

resize("photo.png", 640)                       # ("photo.png", 640, True)
resize("photo.png", width=640, preserve_ratio=False)
# resize(image="photo.png", width=640)         # TypeError
# resize("photo.png", 640, False)              # TypeError
```

* Before `/`: positional-only. Here, `image` cannot be passed by keyword.
* Between `/` and `*`: positional or keyword. Here, `width` accepts either.
* After bare `*`: keyword-only. Here, `preserve_ratio=False` must be named.
* `/` in ordinary Python function definitions arrived in 3.8. Keyword-only parameters already existed in 3.0.

`:=` assigns a value inside an expression when keeping that value is useful:

```python
# Python 3.8+
import re

line = "order=482"
if match := re.fullmatch(r"order=(\d+)", line):
    order_number = int(match.group(1))
    print(f"{order_number=}")  # order_number=482; debug f-string syntax.
```

Use it to avoid duplicate work, rather than compressing several steps. Parentheses matter in comparisons: `if (count := len(items)) > 5:` stores the length; `if count := len(items) > 5:` stores a Boolean.

### Dictionary and string conveniences

[Python 3.9 additions](https://docs.python.org/3/whatsnew/3.9.html)

```python
# Python 3.9+
defaults = {"timeout": 10, "retries": 2}
overrides = {"timeout": 30}
settings = defaults | overrides  # {"timeout": 30, "retries": 2}
defaults                        # {"timeout": 10, "retries": 2}; unchanged.
defaults |= overrides           # Updates defaults in place.

"report.csv".removesuffix(".csv")  # "report"
"prefix_name".removeprefix("prefix_")  # "name"
"report.txt".removesuffix(".csv")  # "report.txt"; unmatched stays unchanged.
```

Dictionary union is shallow and the right-hand value wins for duplicate keys. It does not recursively merge nested dictionaries. `removeprefix` and `removesuffix` remove one exact string; `strip`, `lstrip`, and `rstrip` remove characters from a set.

Dictionary insertion order became a language guarantee in **3.7**; it was a CPython implementation detail in 3.6. Replacing a value keeps its key's position; deleting and reinserting moves it to the end. Dictionary equality still ignores order.

Python **3.12** also relaxed f-string parsing: expressions can reuse the outer quote style, contain comments, and use backslashes. The basic f-string syntax itself dates to 3.6.

```python
# Python 3.12+
person = {"name": "Ada"}
f"Hello, {person["name"]}!"  # "Hello, Ada!"
```

### Structural pattern matching

[Official match tutorial](https://docs.python.org/3/tutorial/controlflow.html#match-statements)

```python
# Python 3.10+
def describe_event(event: object) -> str:
    match event:
        case {"kind": "move", "position": [int(x), int(y)]}:
            return f"Move to ({x}, {y})"
        case {"kind": "message", "text": str(text)} if text.strip():
            return text.strip()
        case _:
            return "Unsupported event"

describe_event({"kind": "move", "position": [2, 5], "id": 99})
# "Move to (2, 5)"; extra mapping keys are allowed.
describe_event({"kind": "message", "text": "  Hello  "})  # "Hello"
```

* Matches structure and binds names; useful for several distinct input shapes.
* `int(x)` checks and captures an integer; it does not convert a string to one. `bool` is an `int` subclass, so this pattern also accepts booleans.
* Bare names usually **capture**, rather than compare to existing variables. Use literals or qualified constants such as `State.READY` to compare.
* `_` is the wildcard. The first matching case whose guard succeeds runs; there is no fall-through.
* A sequence pattern can match lists or tuples, but not strings or bytes.

### Safer iteration and useful standard-library additions

[Iterator helpers](https://docs.python.org/3/library/itertools.html) · [Python 3.11 additions](https://docs.python.org/3/whatsnew/3.11.html)

```python
# Python 3.10+
names = ["Ada", "Lin"]
scores = [8, 9]
dict(zip(names, scores, strict=True))  # {"Ada": 8, "Lin": 9}
# list(zip(["Ada"], [8, 9], strict=True))  # ValueError: lengths differ.

from itertools import pairwise
list(pairwise([10, 15, 12]))  # [(10, 15), (15, 12)]
```

Normal `zip` silently stops at the shortest input. `strict=True` raises when a mismatch is encountered **during iteration**, not necessarily when the iterator is created.

```python
# Python 3.12+
from itertools import batched

list(batched(range(5), 2))  # [(0, 1), (2, 3), (4,)]
# Python 3.13+: strict=True rejects an incomplete final batch.
```

```python
# Python 3.11+
import tomllib

config = tomllib.loads('[storage]\nlimit = 25\n')
config["storage"]["limit"]  # 25
# For files: with open("settings.toml", "rb") as f: tomllib.load(f)
```

`tomllib` reads TOML; it does not write it. Another useful addition is `zoneinfo.ZoneInfo` (3.9) for IANA time zones such as `Europe/London`, using system time-zone data or the separately installed `tzdata` package. Prefer aware datetimes for real timestamps; `datetime.utcnow()` is deprecated since 3.12 in favour of `datetime.now(timezone.utc)`.

#### Caching repeated pure calculations

[Official functools reference](https://docs.python.org/3/library/functools.html#functools.cache)

```python
# Python 3.9+
from functools import cache

@cache
def count_routes(rows: int, columns: int) -> int:
    # Grid routes using only right/down steps; positive dimensions required.
    if rows < 1 or columns < 1:
        raise ValueError("dimensions must be positive")
    if rows == 1 or columns == 1:
        return 1
    return count_routes(rows - 1, columns) + count_routes(rows, columns - 1)

count_routes(3, 3)  # 6
count_routes(3, 3)  # 6; reused from cache.
count_routes.cache_clear()
```

`cache` is unbounded: arguments and results remain referenced until cleared. Arguments must be hashable. Use `lru_cache(maxsize=...)` for a bounded cache. Avoid caching changing external state or returning mutable results that callers may alter. Memoisation reduces repeated work here, but does not remove recursion depth limits.

### Async tasks and grouped failures

[Official asyncio tasks reference](https://docs.python.org/3/library/asyncio-task.html)

```python
# Python 3.11+
import asyncio

async def check_name(name: str) -> str:
    await asyncio.sleep(0)  # Stand-in for asynchronous I/O; yields control.
    return name.upper()

async def main() -> list[str]:
    async with asyncio.TaskGroup() as group:
        first = group.create_task(check_name("alpha"))
        second = group.create_task(check_name("beta"))
    # Both tasks have completed successfully before these results are read.
    return [first.result(), second.result()]

asyncio.run(main())  # ["ALPHA", "BETA"]
```

* Calling an `async def` function creates a coroutine; await or schedule it to execute it.
* `TaskGroup` waits for its tasks. A task failure other than cancellation normally cancels siblings and raises an exception group after cleanup.
* `asyncio.timeout(seconds)` (3.11) limits a block's running time through cancellation.
* `asyncio.to_thread(sync_function, ...)` (3.9) can offload blocking I/O. A blocking call directly inside an async function blocks the event loop.
* Do not swallow `CancelledError`. Call `asyncio.run()` from synchronous entry-point code; in an already running loop, await instead.

Python 3.11 added `ExceptionGroup` and `except*` for handling matching subgroups of multiple exceptions. Ordinary `except ValueError` does not unpack an exception group. Learn the mechanism if working with concurrent failures; a normal single-error function rarely needs it.

### Newest features: recognise before adopting

[Python 3.13](https://docs.python.org/3/whatsnew/3.13.html) · [Python 3.14](https://docs.python.org/3/whatsnew/3.14.html) · [Python 3.15](https://docs.python.org/3/whatsnew/3.15.html)

#### Python 3.13

Type parameters can have defaults, for example `class Box[T = str]: ...`. New typing tools include `ReadOnly` for TypedDict fields and `TypeIs` for user-defined narrowing. These are type-checker contracts, not runtime freezing or validation.

Free-threaded builds and the JIT first appeared experimentally in 3.13. Free-threaded Python became officially supported, but optional, in 3.14. Do not assume an ordinary interpreter has the GIL disabled, or that concurrent access to shared mutable state is safe.

#### Python 3.14

Annotations are deferred by default. This differs from `from __future__ import annotations`, which stores annotation strings. For runtime inspection, prefer the supported `annotationlib` APIs rather than assuming `__annotations__` contains already-evaluated types; evaluating annotations may execute code.

```python
# Python 3.14+
name = "Ada"
message = t"Hello, {name}!"
message.strings                 # ("Hello, ", "!")
message.interpolations[0].value  # "Ada"
```

A t-string produces a `Template`, not a `str`. Its interpolation expressions are evaluated when created; formatting/processing is separate. It is not automatically safe SQL, HTML, or shell escaping.

#### Python 3.15

```python
# Python 3.15+
options = frozendict(timeout=30, retries=2)
options["timeout"]  # 30
# options["timeout"] = 10  # TypeError

groups = [["red", "blue"], ["green"]]
flattened = [*group for group in groups]  # ["red", "blue", "green"]
```

`frozendict` is shallowly immutable; it is hashable only when its keys and values are hashable. `lazy import json` defers loading until first use, including import failures and side effects; this syntax is restricted to module scope outside `try` blocks. UTF-8 is now the default text encoding; explicit encodings still clarify file contracts. A built-in `sentinel` type supports distinct “not supplied” values. These features require 3.15; do not assume assessment support.

### Corrections to carry into older notes

| Older assumption                       | Accurate Python 3 behaviour                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------- |
| `range` is an iterator                 | It is an iterable sequence; `iter(range(...))` creates an iterator.                   |
| `map` and `filter` return lists        | They return iterators; materialise with `list(...)` only when needed.                 |
| An iterable is necessarily stateless   | Iterators are iterable too, and consumption changes their state.                      |
| Lists must contain one element type    | Runtime lists may mix types; `list[T]` expresses a static contract.                   |
| Multiple inheritance uses plain DFS    | Method lookup uses the C3 method resolution order; inspect `Class.__mro__`.           |
| `issubclass` accepts an instance first | Pass a class; use `isinstance` for an instance.                                       |
| `print 'text'`, `<>`, `.encode('hex')` | Use `print('text')`, `!=`, and `b'cd'.hex()` respectively.                            |
| `reduce` is built in                   | Import it from `functools`; prefer a dedicated operation such as `sum` when suitable. |
| Regex `\d` and `\w` are ASCII-only     | String patterns are Unicode-aware by default; `re.ASCII` restricts these classes.     |
| Regex repetition is written `{1-4}`    | Use `{1,4}` for one to four repetitions.                                              |

## Operator precedence

| Operator                                                                                                                                   | Description                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| `**`                                                                                                                                       | Exponentiation (raise to the power)                                            |
| `~` `+` `-`                                                                                                                                | Complement, unary plus and minus (method names for the last two are +@ and -@) |
| `*` `/` `%` `//`                                                                                                                           | Multiply, divide, modulo and floor division                                    |
| `+` `-`                                                                                                                                    | Addition and subtraction                                                       |
| `>>` `<<`                                                                                                                                  | Right and left bitwise shift                                                   |
| `&`                                                                                                                                        | Bitwise AND                                                                    |
| `^` `\|`                                                                                                                                   | Bitwise exclusive 'OR' and regular 'OR'                                        |
| `<=` `<` `>` `>=`                                                                                                                          | Comparison operators                                                           |
| `<>` `==` `!=`                                                                                                                             | Equality operators                                                             |
| <p><code>=</code> <code>%=</code> <code>/=</code><br><code>//=</code> <code>-=</code> <code>+=</code> <code>*=</code> <code>**=</code></p> | Assignment operators                                                           |
| `is` `is not`                                                                                                                              | Identity operators                                                             |
| `in` `not in`                                                                                                                              | Membership operators                                                           |
| `not` `or` `and`                                                                                                                           | Logical operators                                                              |
