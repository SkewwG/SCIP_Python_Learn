# 2.3Sequences（序列）
A sequence is an ordered collection of data values

数据值的有序集合

there are many kinds of sequences, but they all share certain properties

这里有很多类型的序列，但是他们有共同的属性

【*0】序列的2个共有属性：长度、元素选择
---

Sequences provide a layer of abstraction that may hide the details of exactly which sequence type is being manipulated by a particular program.

作用：序列提供了一个抽象层, 它可以隐藏特定程序所操作的序列类型的详细信息。

## 2.3.1 Nested Pairs（嵌套的值对）
A standard way to visualize a pair --- in this case, the pair `(1,2)` --- is called _box-and-pointer_ notation. Each value, compound or primitive, is depicted as a pointer to a box. The box for a primitive value contains a representation of that value. For example, the box for a number contains a numeral. The box for a pair is actually a double box: the left part contains (an arrow to) the first element of the pair and the right part contains the second.

可视化偶对的一个标准方法 -- 这里也就是偶对`(1,2)`-- 叫做盒子和指针记号

偶对的盒子实际上是两个盒子：左边的部分（箭头指向的）包含偶对的第一个元素，右边的部分包含第二个。

---
![](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/pair.png)
用python表达
```
>>> ((1, 2), (3, 4))
((1, 2), (3, 4))
```
![](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/nested_pairs.png)

Our ability to use tuples as the elements of other tuples provides a new means of combination in our programming language

在编程语言中提供了一个新的组合手段：使用元组作为其它元组的元素

---
a method for combining data values satisfies the closure property if the result of combination can itself be combined using the same method. Closure is the key to power in any means of combination because it permits us to create hierarchical structures --- structures made up of parts, which themselves are made up of parts, and so on. We will explore a range of hierarchical structures in Chapter 3\. For now, we consider a particularly important structure.

封闭性：自己包含自己（自包含），组合起来，允许我们去创建层次结构

---
## 2.3.2 Recursive Lists（递归列表）
A non-empty sequence can be decomposed into:
*   its first element, and
*   the rest of the sequence.

一个非空序列可以划分为：
*   它的第一个元素，以及
*   序列的其余部分。

---
These two selectors, one constructor, and one constant together implement the recursive list abstract data type. The single behavior condition for a recursive list is that, like a pair, its constructor and selectors are inverse functions.

这两个选择器和一个构造器，以及一个常量共同实现了抽象数据类型的递归列表。递归列表唯一的行为条件是，就像偶对那样，它的构造器和选择器是相反的函数。

*   If a recursive list `s` was constructed from element `f` and list `r`, then `first(s)` returns `f`, and `rest(s)` returns `r`.

We can use the constructor and selectors to manipulate recursive lists.

```
>>> counts = make_rlist(1, make_rlist(2, make_rlist(3, make_rlist(4, empty_rlist))))
>>> first(counts)
1
>>> rest(counts)
(2, (3, (4, None)))
```
---
*   如果一个递归列表`s`由元素`f`和列表`r`构造，那么`first(s)`返回`f`，并且`rest(s)`返回`r`。

我们可以使用构造器和选择器来操作递归列表。

```source-python
>>> counts = make_rlist(1, make_rlist(2, make_rlist(3, make_rlist(4, empty_rlist))))
>>> first(counts)
1
>>> rest(counts)
(2, (3, (4, None)))
```
---

The recursive list can store a sequence of values in order, but it does not yet implement the sequence abstraction. Using the abstract data type we have defined, we can implement the two behaviors that characterize a sequence: length and element selection.

递归列表可以按顺序存储一系列值, 但它还没有实现序列抽象。使用我们定义的抽象数据类型, 我们可以实现两个序列的特征行为: 长度和元素选择。

```
>>> def len_rlist(s):
        """Return the length of recursive list s."""
        length = 0
        while s != empty_rlist:
            s, length = rest(s), length + 1
        return length

```

```
>>> def getitem_rlist(s, i):
        """Return the element at index i of recursive list s."""
        while i > 0:
            s, i = rest(s), i - 1
        return first(s)

```

Now, we can manipulate a recursive list as a sequence:

现在, 我们可以将递归列表操作为一个序列:
```
>>> len_rlist(counts)
4
>>> getitem_rlist(counts, 1)  # The second item has index 1
2
```
---

## 2.3.3 Tuples II（元组）
Tuples can have arbitrary length, and they exhibit the two principal behaviors of the sequence abstraction: length and element selection. Below, `digits` is a tuple with four elements.
元组具有任意的长度，并且也拥有序列抽象的两个基本行为：长度和元素选择。下面的`digits`是一个四元素元组。
```
>>> digits = (1, 8, 2, 8)
>>> len(digits)
4
>>> digits[3]
8
```

Additionally, tuples can be added together and multiplied by integers. For tuples, addition and multiplication do not add or multiply elements, but instead combine and replicate the tuples themselves. That is, the `add`function in the `operator` module (and the `+` operator) returns a new tuple that is the conjunction of the added arguments. The `mul` function in `operator` (and the `*` operator) can take an integer `k` and a tuple and return a new tuple that consists of `k` copies of the tuple argument.
此外，元组可以彼此相加以及与整数相乘。对于元组，加法和乘法操作并不对元素相加或相乘，而是组合和重复元组本身。也就是说，`operator`模块中的`add`函数（以及`+`运算符）返回两个被加元组连接成的新元组。`operator`模块中的`mul`函数（以及`*`运算符）接受整数`k`和元组，并返回含有元组参数`k`个副本的新元组。

【*1】加法和乘法：↓
---
元组可以加其它的元组
元组可以乘以整数


```source-python
>>> (2, 7) + digits * 2
(2, 7, 1, 8, 2, 8, 1, 8, 2, 8)
```
**Mapping.** A powerful method of transforming one tuple into another is by applying a function to each element and collecting the results.
**映射。**map(函数，序列) 序列里的每个元素都当做函数的参数传递给函数，一一调用函数得到结果。
ptyhon3返回的是map类型，可以通过tuple或者list强制类型转换

```source-python
>>> alternates = (-1, 2, -3, 4, -5)
>>> tuple(map(abs, alternates))
(1, 2, 3, 4, 5)
```

The `map` function is important because it relies on the sequence abstraction: we do not need to be concerned about the structure of the underlying tuple; only that we can access each one of its elements individually in order to pass it as an argument to the mapped function (`abs`, in this case).

---
【*2】map函数非常重要，因为它依赖于序列抽象：我们不需要关心底层元组的结构，只需要能够独立访问每个元素，以便将它作为参数传入用于映射的函数中（这里是abs）。
---

## 2.3.4 Sequence Iteration（序列的迭代）
Mapping is itself an instance of a general pattern of computation: iterating over all elements in a sequence.
一次一个访问序列里的元素，直到访问完序列里的元素
Python has an additional control statement to process sequential data: the `for`statement.
Python 添加控制语句来处理序列数据：`for`语句

---
Consider the problem of counting how many times a value appears in a sequence. We can implement a function to compute this count using a `while` loop.
考虑计算一个值在序列中出现了多少次的问题，。我们可以使用`while`循环实现一个函数来计算这个数量。

```
>>> def count(s, value):
        """Count the number of occurrences of value in sequence s."""
        total, index = 0, 0
        while index < len(s):
            if s[index] == value:
                total = total + 1
            index = index + 1
        return total

```

```
>>> count(digits, 8)
2

```

The Python `for` statement can simplify this function body by iterating over the element values directly, without introducing the name `index` at all. `For` example (pun intended), we can write:

Python `for`语句可以通过直接迭代元素值来简化这个函数体，完全不需要引入`index`。我们可以写成：
```
>>> def count(s, value):
        """Count the number of occurrences of value in sequence s."""
        total = 0
        for elem in s:
            if elem == value:
                total = total + 1
        return total

```

```
>>> count(digits, 8)
2
```
【*3】循环用while语句，迭代用for语句
---
---

A `for` statement consists of a single clause with the form:

for 语句的组成

```
for <name> in <expression>:
    <suite>

```

A `for` statement is executed by the following procedure:

`for`语句按照以下过程来执行：

1.  Evaluate the header `&lt;expression&gt;`, which must yield an iterable value.
<expression>必须产生一个可迭代的值
2.  For each element value in that sequence, in order:
    1.  Bind `&lt;name&gt;` to that value in the local environment.
序列里的每一个值，绑定到name局部变量里
    2.  Execute the `&lt;suite&gt;`.
然后执行suite

An important consequence of this evaluation procedure is that `&lt;name&gt;` will be bound to the last element of the sequence after the `for` statement is executed.

【*4】这个求值过程的一个重要结果是，在for语句执行完毕之后，name会绑定到序列的最后一个元素上。
---

**Sequence unpacking（序列解包）.**
A common pattern in programs is to have a sequence of elements that are themselves sequences, but all of a fixed length. `For` statements may include multiple names in their header to "unpack" each element sequence into its respective elements. For example, we may have a sequence of pairs (that is, two-element tuples),
程序中的一个常见模式是，序列的元素本身就是序列，但是具有固定的长度。`for`语句可在头部中包含多个名称，将每个元素序列“解构”为各个元素。例如，我们拥有一个偶对（也就是二元组）的序列：
```
>>> pairs = ((1, 2), (2, 2), (2, 3), (4, 4))

```

and wish to find the number of pairs that have the same first and second element.

```
>>> same_count = 0

```

The following `for` statement with two names in its header will bind each name `x` and `y` to the first and second elements in each pair, respectively.
【*5】下面的`for`语句的头部带有两个名词，会将每个名称`x`和`y`分别绑定到每个偶对的第一个和第二个元素上。
---

```
>>> for x, y in pairs:
        if x == y:
            same_count = same_count + 1

```

```
>>> same_count
2

```

This pattern of binding multiple names to multiple values in a fixed-length sequence is called _sequence unpacking_; it is the same pattern that we see in assignment statements that bind multiple names to multiple values.
**这个绑定多个名称到定长序列中多个值的模式，叫做序列解包。它的模式和我们在赋值语句中看到的，将多个名称绑定到多个值的模式相同。**
【*6】Range也是一个序列。
---
**Ranges.** A `range` is another built-in type of sequence in Python, which represents a range of integers. Ranges are created with the `range` function, which takes two integer arguments: the first number and one beyond the last number in the desired range.
`range`是另一种 Python 的内建序列类型，它表示一个整数范围。范围可以使用`range`函数来创建，它接受两个整数参数：所得范围的第一个数值和最后一个数值加一。

---
```
>>> range(1, 10)  # Includes 1, but not 10
range(1, 10)

```

Calling the `tuple` constructor on a range will create a tuple with the same elements as the range, so that the elements can be easily inspected.

在范围上调用`tuple`构造器会创建与范围具有相同元素的元组，使元素易于查看。
```
>>> tuple(range(5, 8))
(5, 6, 7)
```
If only one argument is given, it is interpreted as one beyond the last value for a range that starts at 0.

如果只提供了一个元素，它会解释为最后一个数值加一，范围开始于 0。
```
>>> tuple(range(4))
(0, 1, 2, 3)

```

Ranges commonly appear as the expression in a `for` header to specify the number of times that the suite should be executed:

```
>>> total = 0
>>> for k in range(5, 8):
        total = total + k

```

```
>>> total
18

```

A common convention is to use a single underscore character for the name in the `for` header if the name is unused in the suite:

【*7】约定俗成：如果不用变量，用下划线（_）赋值。
---
```
>>> for _ in range(3):
        print('Go Bears!')

Go Bears!
Go Bears!
Go Bears!

```

Note that an underscore is just another name in the environment as far as the interpreter is concerned, but has a conventional meaning among programmers that indicates the name will not appear in any expressions.
对解释器来说，下划线是另一个名称，但是在程序员中具有固定含义，它表明这个名称不应出现在任何表达式中。

## 2.3.5 Sequence Abstraction（序列抽象）
We have now introduced two types of native data types that implement the sequence abstraction: tuples and ranges. Both satisfy the conditions with which we began this section: length and element selection. Python includes two more behaviors of sequence types that extend the sequence abstraction.
**我们已经介绍了两种原生数据类型，它们实现了序列抽象：元组和Range。两个都满足这一章开始时的条件：长度和元素选择。**
``` python
>>> a = (1,2,3)
>>> len(a)             #序列的长度
3
>>> a[2]               #序列的元素选择
3
>>> b = range(10)
>>> b[1]               #序列的元素选择
1
>>> len(b)             #序列的长度
10
```
**Python 还包含了两种序列类型的行为，它们扩展了序列抽象。**

【*8】序列的2个行为：成员测试、切片
--
**成员测试：判断一个元素是否在序列当中（in | not in）**
**切片**
**Membership（成员测试）.** A value can be tested for membership in a sequence. Python has two operators `in` and `not in` that evaluate to `True` or `False` depending on whether an element appears in a sequence.
可以测试一个值在序列中的成员性。Python 拥有两个操作符`in`和`not in`，取决于元素是否在序列中出现而求值为`True`和`False`。

```
>>> digits
(1, 8, 2, 8)
>>> 2 in digits
True
>>> 1828 not in digits
True

```

All sequences also have methods called `index` and `count`, which return the index of (or count of) a value in a sequence.

**所有序列都有叫做`index`和`count`的方法，它会返回序列中某个值的下标（或者数量）。**
``` python
>>> a = (1,1,1,2,3,4,4,5)
>>> a.count(1)
3
```
**Slicing（切片）.** 
In Python, sequence slicing is expressed similarly to element selection, using square brackets. A colon separates the starting and ending indices. Any bound that is omitted is assumed to be an extreme value: 0 for the starting index, and the length of the sequence for the ending index.
**Python 中，序列切片的表示类似于元素选择，使用方括号。冒号分割了起始和结束下标。任何边界上的省略都被当作极限值：起始下标为 0，结束下标是序列长度。**

```
>>> digits[0:2]
(1, 8)
>>> digits[1:]
(8, 2, 8)
```

## 2.3.6 Strings（字符串）
The native data type for text in Python is called a string, and corresponds to the constructor `str`.
Python 中原生的文本数据类型叫做字符串，相应的构造器是`str`。

String literals can express arbitrary text, surrounded by either single or double quotation marks.
字符串可以表达任意文本，被单引号或者双引号包围。

```
>>> 'I am string!'
'I am string!'
>>> "I've got an apostrophe"
"I've got an apostrophe"
>>> '您好'
'您好'

```
Strings satisfy the two basic conditions of a sequence that we introduced at the beginning of this section: they have a length and they support element selection.
字符串满足两个基本的序列条件，我们在这一节开始介绍过它们：它们拥有长度和元素选择

```
>>> city = 'Berkeley'
>>> len(city)
8
>>> city[3]
'k'

```

The elements of a string are themselves strings that have only a single character. A character is any single letter of the alphabet, punctuation mark, or other symbol. Unlike many other programming languages, Python does not have a separate character type; any text is a string, and strings that represent single characters have a length of 1.
字符串的元素本身就是包含单一字符的字符串。字符是字母表中的任意单一字符，标点符号，或者其它符号。不像许多其它编程语言那样，Python 没有单独的字符类型，任何文本都是字符串，表示单一字符的字符串长度为 1

Like tuples, strings can also be combined via addition and multiplication.
【*10】字符串也可以通过加法和乘法来组合：
---
```
>>> 'Berkeley' + ', CA'
'Berkeley, CA'
>>> 'Shabu ' * 2
'Shabu Shabu '

```

**Membership.** The behavior of strings diverges from other sequence types in Python. The string abstraction does not conform to the full sequence abstraction that we described for tuples and ranges. In particular, the membership operator `in` applies to strings, but has an entirely different behavior than when it is applied to sequences. It matches substrings rather than elements.
字符串的行为不同于 Python 中其它序列类型。字符串抽象没有实现我们为元组和Ranges描述的完整序列抽象。特别地，字符串上实现了成员性运算符`in`，但是与序列上的实现具有完全不同的行为。它匹配子字符串而不是元素。
```
>>> 'here' in "Where's Waldo?"
True

```

Likewise, the `count` and `index` methods on strings take substrings as arguments, rather than single-character elements. The behavior of `count` is particularly nuanced; it counts the number of non-overlapping occurrences of a substring in a string.
与之相似，字符串上的`count`和`index`方法接受子串作为参数，而不是单一字符。`count`的行为有细微差别，它统计字符串中非重叠字串的出现次数。
【*11】count方法
---
```
>>> 'Mississippi'.count('i')
4
>>> 'Mississippi'.count('issi')
1
```
---
**Multiline Literals.** Strings aren't limited to a single line. Triple quotes delimit string literals that span multiple lines. We have used this triple quoting extensively already for docstrings.
**多行文本。**字符串并不限制于单行文本，三个引号分隔的字符串可以跨越多行。我们已经在文档字符串中使用了三个引号。

```
>>> """The Zen of Python
claims, Readability counts.
Read more: import this."""
'The Zen of Python\nclaims, "Readability counts."\nRead more: import this.'

```

In the printed result above, the `\n` (pronounced "_backslash en_") is a single element that represents a new line. Although it appears as two characters (backslash and "n"), it is considered a single character for the purposes of length and element selection.

【*12] 三个引号的多行文本：\n（叫做“反斜杠加 n”）是表示新行的单一元素。虽然它表示为两个字符（反斜杠和 n）。它在长度和元素选择上被认为是单个字符。
---
``` python
#多行文本：
>>> '''asdfasdf
zxcvzxcv
qweqweqwe
'''
'asdfasdf\nzxcvzxcv\nqweqweqwe\n'
换行地方，默认加上了换行符！
结果是：11111111111111\n222222222222222
```

【*13】超长字符串表达方式，用括号括起来
---
``` python
#超长字符串：("11111111111"
            "22222222222")
结果是：1111111111122222222222
```

**String Coercion.** A string can be created from any object in Python by calling the `str` constructor function with an object value as its argument. This feature of strings is useful for constructing descriptive strings from objects of various types.
字符串可以从 Python 的任何对象通过以某个对象值作为参数调用`str`构造函数来创建，这个字符串的特性对于从多种类型的对象中构造描述性字符串非常实用。

```
>>> str(2) + ' is an element of ' + str(digits)
'2 is an element of (1, 8, 2, 8)'

```
【*14】Methods（方法）
---

```
>>> '1234'.isnumeric()
True
>>> 'rOBERT dE nIRO'.swapcase()
'Robert De Niro'
>>> 'snakeyes'.upper().endswith('YES')
True
```

## 2.3.7 Conventional Interfaces（常规接口）

【*15】序列计算模式：枚举、映射、筛选、计算的模式
---
```
 enumerate     map    filter  accumulate
-----------    ---    ------  ----------
naturals(n)    fib    iseven     sum
```

The filter function takes a sequence and returns those elements of a sequence for which a predicate is true
传递一个序列，返回序列中断言为真的元素。 返回的是filter对象，可以转换为元组

【*16】Generator expressions 生成器表达式：处理序列这种数据结构的方法
---
```
<map expression> for <name> in <sequence expression> if <filter expression>
```
(x*2 for x in range(5) if x%2==0) 返回的是generator生成器类型
list(x*2 for x in range(5) if x%2==0) list强制转换生成器成列表

``` python
# <map expression> for <name> in <sequence expression> if <filter expression>

a = (x * 2 for x in range(10) if x % 2 == 0)
print(a)            # <generator object <genexpr> at 0x0000000002A69E10>
b = list((x * 2 for x in range(10) if x % 2 == 0))
print(b)            # [0, 4, 8, 12, 16]

def double(x):
    return x * 2

c = list((double(x) for x in range(10) if x % 2 == 0))
print(c)            # [0, 4, 8, 12, 16]

d = list(lambda x : x * 2 for x in range(10) if x % 2 == 0)
print(d)            # 错误

e = lambda x : x * 2
f = list(d for x in range(10) if x % 2 == 0)

# 总结：使用生成器表达式，要先把函数定义出来！
```

[17*] reduce(函数，序列) 
---
Python includes `reduce` in the `functools` module, which applies a two-argument function cumulatively to the elements of a sequence from left to right, to reduce a sequence to a value.
Python 在`functools`模块中包含`reduce`，它将序列中的元素先取出2个，传递给函数得到的值与序列的下一个元素传递给函数，依次直到取完所有元素，得到最终值。
``` python
reduce(lambda x, y:x*2+y*2, (1,2,3,4))
先传递1和2，再传递6和3，最后传递18和4
```
