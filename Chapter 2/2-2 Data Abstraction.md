# 2.2 数据抽象
`a?_compound data_?value`
复合数据值

---
模块化
某种数据结构、某种抽象数据。
分层次、分结构、大问题化为小问题

`The general technique of isolating the parts of a program that deal with how data are represented from the parts of a program that deal with how those data are manipulated is a powerful design methodology called?_data abstraction_. Data abstraction makes programs much easier to design, maintain, and modify.`

**一种技术：将程序从展示到处理分割成俩部分。被叫做数据抽象**

---

`data abstraction is a methodology that enables us to isolate how a compound data object is used from the details of how it is constructed.`

*数据抽象是一个方法论：从（使用、构造）分割开复合数据对象*

---

抽象鸡肋：对方向的利用

```The basic idea of data abstraction is to structure programs so that they operate on abstract data. That is, our programs should use data in such a way as to make as few assumptions about the data as possible. At the same time, a concrete data representation is defined, independently of the programs that use the data. The interface between these two parts of our system will be a set of functions, called selectors and constructors, that implement the abstract data in terms of the concrete representation. To illustrate this technique, we will consider how to design a set of functions for manipulating rational numbers.```

**不先去构造对象，而是先去考虑怎么使用。

个人理解：写类的时候，先不写具体的函数内容，而是先定义一个个自己想要使用的功能（方法）。后期到用的时候再去完善方法内容**

---

选择器（a.b方法）、构造器（init）

---

## 2.2.1有理数算数 rational（比率）
`<numerator>/<denominator>`
`Rational numbers are important in computer science`
`Thus, working with rational numbers should, in principle, allow us to avoid approximation errors in our arithmetic.`

```we would like to keep the numerator and denominator separate for the sake of precision, but treat them as a single unit.```

有理数一定是精确的，用有限数的小数表达它是不精确的

在计算机里做精度运算的时候，更倾向于把分子和分母都保留下来，而不把它求值。（防止精度损失）

---

```
We know from using functional abstractions that we can start programming productively before we have an implementation of some parts of our program. Let us begin by assuming that we already have a way of constructing a rational number from a numerator and a denominator. We also assume that, given a rational number, we have a way of extracting (or selecting) its numerator and its denominator. Let us further assume that the constructor and selectors are available as the following three functions:

make_rat(n, d)?returns the rational number with numerator?`n`?and denominator?`d`.
numer(x)?returns the numerator of the rational number?`x`.
denom(x)?returns the denominator of the rational number?`x`.
```

3个假装：

1、已经有了构造有理数的方法

2、拥有另外的一种方法，给定一个有理数，分解为分子和分母

3、从有理数选择分子和分母的方法

make_rat(n,d) 给分子分母，返回有理数

number(x) 给有理数，返回分子

denom(y) 给有理数，返回分母

---

## 2.2.2元组
`multiple assignment.`
多重赋值：
``` python
pair =(1, 2)
a,b = pari
```

---

```Tuples in Python (and sequences in most other programming languages) are 0-indexed, meaning that the index?`0`?picks out the first element, index?`1`?picks out the second, and so on. One intuition that underlies this indexing convention is that the index represents how far an element is offset from the beginning of the tuple.```

**索引以0开始（间隔距离）：这个值和第一个元素的距离间隔**

getitem，从序列里取出元素

---

读一下Further reading
`Further reading.?The?str_rat?implementation above uses?_format strings_, which contain placeholders for values. The details of how to use format strings and the?format?method appear in the?[formatting strings](http://diveintopython3.ep.io/strings.html#formatting-strings)?section of Dive Into Python 3.`

实现的str_rat格式字符串这个方法包含了占位符的值。有关如何使用格式字符串和格式方法的细节将出现在Python 3的格式化字符串部分中。

---

## 2.2.3 抽象界限
```In general, the underlying idea of data abstraction is to identify for each type of value a basic set of operations in terms of which all manipulations of values of that type will be expressed, and then to use only those operations in manipulating the data.```

only背后的意义：个人理解

**一个class的属性和方法只能被自己的实例所调用，不能被别的类实例调用**

---

![](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/barriers.png)

**问题域这层只关心如何使用加法、乘法**

**如何实现加分，乘法（如何实现make_rat，如何得到分子、得到分母）**

**如何实现分子、实现分母、实现有理数，如何使用tuple、getitem**

---

The fewer functions that depend on a particular representation, the fewer changes are required when one wants to change that representation.

**依赖越少，以后修改的时候也就越少**

---

## 2.2.4 The Properties of Data
```In general, we can think of an abstract data type as defined by some collection of selectors and constructors, together with some behavior conditions. As long as the behavior conditions are met (such as the division property above), these functions constitute a valid representation of the data type.```

**一个抽象数据：选择器、构造器、行为条件。当行为条件触发的时候，能够正确的执行行为。**
