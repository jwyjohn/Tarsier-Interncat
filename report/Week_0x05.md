# Jwy's Report Week 0x05

## BJ67 工作

- [x] 参与 [isrc-cas/tarsier-meta](https://github.com/isrc-cas/tarsier-meta) 的维护 见 [PR](https://github.com/isrc-cas/tarsier-meta/pull/5)

## BJ61 工作

- [x] 熟悉 The Little Schemer 和 NbE（笔记待整理）

# The Little Schemer

## Commandments

### The First Commandment
When recurring on a list of atoms, *lat*, ask two questions about it: *(null? lat)* and *else*.

When recurring on a number, n, ask two questions about it: *(zero? n)* and *else*.

When recurring on a list of S-expressions, *l*, ask three questions about it: *(null? l), (atom? (car l))*, and  else.

### The Second Commandment
Use *cons* to build lists.

### The Third Commandment
When building a list, describe the first typical element, and then *cons* it onto the natural recursion.

### The Fourth Commandment
Always change at least one argument while recurring. When recurring on a list of atoms, *lat*, use *(cdr lat)*. When recurring on a number, *n*, use *(sub1 n)*. And when recurring on a list of S-expressions, l, use *(car l)* and *(cdr l)* if neither *(null? l)* nor *(atom? (car l))* are true.

It must be changed to be closer to termination. The changing argument must be tested in the termination condition: when using *cdr*, test termination will *null?* and when  using *sub1*, test termination with *zero?*

### The Fifth Commandment
When building a value with +, always use 0 for the value of the termination line, for adding 0 does not change the value of an addition.

When building a value with x, always use 1 for the value of the terminating line, for multiplying by 1 does not change the value of multiplication.

When building a value with *cons*, always consider () for the value of the terminating line.

### The Sixth Commandment
Simplify only after the function is correct.

### The Seventh Commandment
Recur on the *subparts* that are of the same nature:

* On the sub-lists of a list.

* On the sub-expressions of an arithmetic expression.

### The Eight Commandment
Use help functions to abstract from representations.

### The Ninth Commandment
Abstract common patterns with a new function.

### The Tenth Commandment
Build functions to collect more than one value at a time.

## Rules

### The Law of Car
The primitive *car* is defined only for non-empty lists.

### The Law of Cdr
The primitive *cdr* is defined only on non-empty lists. The *cdr* of any non-empty list is always another list.

### The Law of Null?
The primitive *null?* is defined only for lists.

### The Law of Eq?
The primitive *eq?* takes two arguments. Each must be a non-numeric atom.