# 2.4 Mutable Data(可变数据)

we need strategies to help us structure large systems so that they will be _modular_, that is, so that they can be divided "naturally" into coherent parts that can be separately developed and maintained.

模块化：分成一序列的各个部分，每个部分都是相对独立的开发与维护

One powerful technique for creating modular programs is to introduce new kinds of data that may change state over time.

创建模块化程序引入一种新型的数据类型，这种数据类型是随着时间的变化而变化

a single data object can represent something that evolves independently of the rest of the program.

单个数据对象可以表示与程序的其余部分无关的东西

【*1】存储作用：状态的延续
---

**纯函数是无状态的，调用一次和调用二次都是一样的结果。**

 numbers, Booleans, tuples, ranges, and strings --- are all types of _immutable_ objects. While names may change bindings to different values in the environment during the course of execution, the values themselves do not change.
 
数字、布尔值、元组、范围和字符串---都是不可变对象的类型。在执行过程中, 名称可能会绑定环境中不同的值, 但这些值本身不会更改。

【*2】数字、布尔、元组、Ranges、字符串都是不可变对象
---

【*3】变化的只是绑定关系，而不存在这个值本身发生变化
---

【*4】可变数据类型：允许在程序的运行过程中，改变数据类型这个值的本身
---

## 2.4.1 Local State（局部状态）
【5*】一个函数可以拥有自己的局部状态，使用带有局部状态（nonlocal）的函数，我们就能实现可变数据类型
---

We will do so by creating a function called `withdraw`, which takes as its argument an amount to be withdrawn. If there is enough money in the account to accommodate the withdrawal, then `withdraw` should return the balance remaining after the withdrawal. Otherwise, `withdraw`should return the message `'Insufficient funds'`. For example, if we begin with $100 in the account, we would like to obtain the following sequence of return values by calling withdraw:

我们通过创建叫做`withdraw`的函数来实现它，它将要取出的金额作为参数。如果账户中有足够的钱来取出，`withdraw`应该返回取钱之后的余额。否则，`withdraw`应该返回消息`'Insufficient funds'`。例如，如果我们以账户中的`$100`开始，我们希望通过调用`withdraw`来得到下面的序列：

```
>>> withdraw(25)
75
>>> withdraw(25)
50
>>> withdraw(60)
'Insufficient funds'
>>> withdraw(15)
35
```

Observe that the expression `withdraw(25)`, evaluated twice, yields different values. This is a new kind of behavior for a user-defined function: it is non-pure. Calling the function not only returns a value, but also has the side effect of changing the function in some way, so that the next call with the same argument will return a different result. All of our user-defined functions so far have been pure functions, unless they called a non-pure built-in function. They have remained pure because they have not been allowed to make any changes outside of their local environment frame!

观察表达式`withdraw(25)`，**求值了两次，产生了不同的值。这是一种用户定义函数的新行为：它是非纯函数。调用函数不仅仅返回一个值，同时具有以一些方式修改函数的副作用，使带有相同参数的下次调用返回不同的结果。**我们所有用户定义的函数，到目前为止都是纯函数，除非他们调用了非纯的内建函数。它们仍旧是纯函数，因为它们并**不允许修改任何在局部环境帧之外的东西。**

pure because they have not been allowed to make any changes outside of their local environment frame!

【*6】纯函数在执行期间不允许修改任何在局部环境栈帧之外的东西
---

【*7】非纯函数除了改变局部环境的栈帧之外，还会改变除它这个局部栈帧之外的东西
---

is a higher-order function that takes a starting balance as an argument. The function `withdraw` is its return value.

`make_withdraw`函数是个高阶函数，接受起始余额作为参数，`withdraw`函数是它的返回值。


```
>>> withdraw = make_withdraw(100)

```

An implementation of `make_withdraw` requires a new kind of statement: a `nonlocal` statement. When we call `make_withdraw`, we bind the name `balance` to the initial amount. We then define and return a local function, `withdraw`, which updates and returns the value of `balance` when called.

`make_withdraw`的实现需要新类型的语句：

**`nonlocal`语句。当我们调用`make_withdraw`时，我们将名称`balance`绑定到初始值上。之后我们定义并返回了局部函数，`withdraw`，它在调用时更新并返回`balance`的值。**

```
>>> def make_withdraw(balance):
        """Return a withdraw function that draws down balance with each call."""
        def withdraw(amount):
            nonlocal balance                 # Declare the name "balance" nonlocal
            if amount > balance:
                return 'Insufficient funds'
            balance = balance - amount       # Re-bind the existing balance name
            return balance
        return withdraw

```

The novel part of this implementation is the `nonlocal` statement, which mandates that whenever we change the binding of the name `balance`, the binding is changed in the first frame in which `balance` is already bound. Recall that without the `nonlocal` statement, an assignment statement would always bind a name in the first frame of the environment. The `nonlocal` statement indicates that the name appears somewhere in the environment other than the first (local) frame or the last (global) frame.

`nonlocal`语句，无论什么时候我们修改了名称`balance`的绑定，绑定都会在`balance`之前已经绑定的第一个帧中修改。回忆一下，在没有`nonlocal`语句的情况下，赋值语句总是会在环境的第一个帧中绑定名称。`nonlocal`语句表明，名称出现在环境中不是第一个（局部）帧，或者最后一个（全局）帧的其它地方。

【*8】nonlocal语句的变量作用域在外层函数和内层函数之间。
---

![](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/nonlocal_def.png)

Our definition statement has the usual effect: it creates a new user-defined function and binds the name `make_withdraw` to that function in the global frame.

建了新的用户定义函数，并且将名称`make_withdraw`在全局帧中绑定到那个函数上

Next, we call `make_withdraw` with an initial balance argument of `20`.

下面，我们使用初始的余额参数`20`来调用`make_withdraw`。

```
>>> wd = make_withdraw(20)

```

This assignment statement binds the name `wd` to the returned function in the global frame.

这个赋值语句将名称`wd`绑定到全局帧中的返回函数上：

![](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/nonlocal_assign.png)

The returned function, (intrinsically) called _withdraw_, is associated with the local environment for the _make_withdraw_ invocation in which it was defined. The name `balance` is bound in this local environment. Crucially, there will only be this single binding for the name `balance` throughout the rest of this example.

**名称`balance`绑定后，只有这一个绑定！**

Next, we evaluate an expression that calls _withdraw_ on an amount `5`.

```
>>> wd(5)
15

```

The name `wd` is bound to the _withdraw_ function, so the body of _withdraw_ is evaluated in a new environment that extends the environment in which _withdraw_ was defined. 

**名称`wd`绑定到了`withdraw`函数上，因为每次调用一个函数，都会产生新的栈帧，所以`withdraw`的函数体在新的环境中求值，而新的环境从`withdraw`定义所在的环境中扩展而来。**

![](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/nonlocal_call.png)

The assignment statement in _withdraw_ would normally create a new binding for `balance` in _withdraw_'s local frame. Instead, because of the `nonlocal` statement, the assignment finds the first frame in which `balance`was already defined, and it rebinds the name in that frame. If `balance` had not previously been bound to a value, then the `nonlocal` statement would have given an error.

**`withdraw`的赋值语句通常在`withdraw`的局部帧中为`balance`创建新的绑定。由于`nonlocal`语句，赋值运算找到了`balance`定义位置的第一帧，并在那里重新绑定名称。**

【*9】nonlocal理解（自己的理解）：因为调用了withdraw函数，所以在withdraw的局部栈帧中本来打算为balance创建一个新的绑定，但因为nonlocal语句，balance之前绑定过，所以找到了之前balance绑定的那个第一帧，然后在之前那个地方重新绑定。
---


By virtue of changing the binding for `balance`, we have changed the _withdraw_ function as well. The next time _withdraw_ is called, the name `balance` will evaluate to `15` instead of `20`.

通过修改`balance`绑定的行为，我们也修改了`withdraw`函数。下次`withdraw`调用的时候，名称`balance`会求值为`15`而不是`20`。

When we call `wd` a second time,

当我们第二次调用`wd`时，

```
>>> wd(3)
12

```

we see that the changes to the value bound to the name `balance` are cumulative across the two calls.

![img/nonlocal_recall.png](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/nonlocal_recall.png)

【*10】从图可以看到，当第二次调用withdraw函数时，又创建了一个新的局部栈帧，amount绑定到3，第一次调用而创建的局部栈帧中amount绑定到5，但是，balance的绑定一直都是在make_withdraw函数创建的时候产生的局部栈帧中，并没有因为withdraw的调用而改变。但是如果make_withdraw再次调用，它会创建单独的帧，banlance将会绑定到这个新创建的帧。
---


By introducing `nonlocal` statements, we have created a dual role for assignment statements. Either they change local bindings, or they change nonlocal bindings. In fact, assignment statements already had a dual role: they either created new bindings or re-bound existing names. The many roles of Python assignment can obscure the effects of executing an assignment statement. It is up to you as a programmer to document your code clearly so that the effects of assignment can be understood by others.

【*11】赋值语句扮演了2中角色
---

1、创建了一个新的变量名绑定
---

2、解除绑定，重新绑定一个新的变量名
---


## 2.4.2 The Benefits of Non-Local Assignment（非局部赋值的好处）
If _make_withdraw_ is called again, then it will create a separate frame with a separate binding for `balance`.

如果`make_withdraw`再次调用，它会创建单独的帧，banlance将会绑定到这个新创建的帧。

```
>>> wd2 = make_withdraw(7)

```

 The name `wd` is still bound to a _withdraw_ function with a balance of `12`, while `wd2` is bound to a new _withdraw_ function with a balance of `7`.
 
 名称`wd`仍旧绑定到余额为`12`的`withdraw`函数上，而`wd2`绑定到了余额为`7`的新的`withdraw`函数上。

![](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/nonlocal_def2.png)

wd的局部栈帧和wd2的局部栈帧完全是不一样的。wd2是重新创建了一次make_withdraw的栈帧。

Finally, we call the second _withdraw_ bound to `wd2`:

最后，我们调用绑定到`wd2`上的第二个`withdraw`函数：

```
>>> wd2(6)
1

```

This call changes the binding of its nonlocal `balance` name, but does not affect the first _withdraw_ bound to the name `wd` in the global frame.

这个调用修改了非局部名称`balance`的绑定，但是不影响在全局帧中绑定到名称`wd`的第一个`withdraw`。

![img/nonlocal_call2.png](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/nonlocal_call2.png)

**从图中可以看到，balance的绑定是在另一个局部环境中，而不是之前的那个环境。所以说wd2是调用make_withdraw函数后创建的一个新的局部变量！**


【12*】当外部的函数make_withdraw重新调用，会创建另一个自己的栈帧，
---



## 2.4.3 The Cost of Non-Local Assignment（非局部赋值的代价）

Previously, our values did not change; only our names and bindings changed. When two names `a` and `b`were both bound to the value `4`, it did not matter whether they were bound to the same `4` or different `4`'s. As far as we could tell, there was only one `4` object that never changed.

之前，我们的值并没有改变，仅仅是我们的名称和绑定发生了变化。当两个名称`a`和`b`绑定到`4`上时，它们绑定到了相同的`4`还是不同的`4`并不重要。我们说，只有一个`4`对象，并且它永不会改变。

However, functions with state do not behave this way. When two names `wd` and `wd2` are both bound to a _withdraw_ function, it _does_ matter whether they are bound to the same function or different instances of that function. Consider the following example, which contrasts the one we just analyzed.

但是，带有状态的函数不是这样的。当两个名称`wd`和`wd2`都绑定到`withdraw`函数时，它们绑定到相同函数还是函数的两个不同实例，就很重要了。考虑下面的例子，它与我们之前分析的那个正好相反：

```
>>> wd = make_withdraw(12)
>>> wd2 = wd
>>> wd2(1)
11
>>> wd(1)
10

```

In this case, calling the function named by `wd2` did change the value of the function named by `wd`, because both names refer to the same function. The environment diagram after these statements are executed shows this fact.

这里，通过`wd2`调用函数会修改名称为`wd`的函数的值，因为两个名称都指向相同的函数。这些语句执行之后的环境图示展示了这个现象：

![img/nonlocal_corefer.png](https://wizardforcel.gitbooks.io/sicp-in-python/content/img/nonlocal_corefer.png)

The key to correctly analyzing code with non-local assignment is to remember that only function calls can introduce new frames. Assignment statements always change bindings in existing frames. In this case, unless _make_withdraw_ is called twice, there can be only one binding for `balance`.


【13*】正确分析带有nonlocal赋值代码的关键是
---

1、只有函数调用会产生新的栈帧
---

2、赋值语句只改变已存在栈帧的绑定关系。
---


**这里，除非`make_withdraw`调用了两次，`balance`还是只有一个绑定。**

## 2.4.4 Lists(列表)

All of these _mutation operations_ change the value of the list; they do not create new list objects.
列表是可变数据类型


【*14】is 和 is not判断是否为同一个对象
---

两个对象当且仅当在内存中的位置相同时为同一个对象
---

 Python includes two comparison operators, called `is` and `is not`, that test whether two expressions in fact evaluate to the identical object. Two objects are identical if they are equal in their current value, and any change to one will always be reflected in the other. Identity is a stronger condition than equality.

Python 引入了两个比较运算符，叫做`is`和`is not`，测试了两个表达式实际上是否求值为同一个对象。如果两个对象的当前值相等，并且一个对象的改变始终会影响另一个，那么两个对象是同一个对象。身份是个比相等性更强的条件。

```
>>> suits is nest[0]
True
>>> suits is ['heart', 'diamond', 'spade', 'club']
False
>>> suits == ['heart', 'diamond', 'spade', 'club']
True

```

The final two comparisons illustrate the difference between `is` and `==`. The former checks for identity, while the latter checks for the equality of contents.

**最后的两个比较展示了`is`和`==`的区别，前者检查身份，而后者检查内容的相等性。**


【*15】List comprehensions.（列表推导式）
---

A list comprehension uses an extended syntax for creating lists, analogous to the syntax of generator expressions.

表推导式使用扩展语法来创建列表，与生成器表达式的语法相似。

``` python
a = [(x,y) for x in range(10) for y in range(10) if x % 2 == 0 if y % 2 != 0]
```

Our mutable list is a dispatch function, just as our functional implementation of a pair was a dispatch function. It checks the input "message" against known messages and takes an appropriate action for each different input. Our mutable list responds to five different messages. The first two implement the behaviors of the sequence abstraction. The next two add or remove the first element of the list. The final message returns a string representation of the whole list contents.

我们的可变列表是个调度函数，就像我们偶对的函数式实现也是个调度函数。它检查输入“信息”是否为已知信息，并且对每个不同的输入执行相应的操作。我们的可变列表可响应五个不同的信息。前两个实现了序列抽象的行为。接下来的两个添加或删除列表的第一个元素。最后的信息返回整个列表内容的字符串表示。

【*16】怎么通过函数实现列表
---
 
```
>>> def make_mutable_rlist():
        """Return a functional implementation of a mutable recursive list."""
        contents = empty_rlist
        def dispatch(message, value=None):
            nonlocal contents
            if message == 'len':
                return len_rlist(contents)
            elif message == 'getitem':
                return getitem_rlist(contents, value)
            elif message == 'push_first':
                contents = make_rlist(value, contents)
            elif message == 'pop_first':
                f = first(contents)
                contents = rest(contents)
                return f
            elif message == 'str':
                return str(contents)
        return dispatch

```

We can also add a convenience function to construct a functionally implemented recursive list from any built-in sequence, simply by adding each element in reverse order.

我们也可以添加一个辅助函数，来从任何内建序列中构建函数式实现的递归列表。只需要以递归顺序添加每个元素。

```
>>> def to_mutable_rlist(source):
        """Return a functional list with the same contents as source."""
        s = make_mutable_rlist()
        for element in reversed(source):
            s('push_first', element)
        return s

```

In the definition above, the function `reversed` takes and returns an iterable value; it is another example of a function that uses the conventional interface of sequences.

在上面的定义中，函数`reversed`接受并返回可迭代值。它是使用序列的接口约定的另一个示例。

At this point, we can construct a functionally implemented lists. Note that the list itself is a function.

这里，我们可以构造函数式实现的列表，要注意列表自身也是个函数。

```
>>> s = to_mutable_rlist(suits)
>>> type(s)
<class 'function'>
>>> s('str')
"('heart', ('diamond', ('spade', ('club', None))))"

```

In addition, we can pass messages to the list `s` that change its contents, for instance removing the first element.

另外，我们可以像列表`s`传递信息来修改它的内容，比如移除第一个元素。

```
>>> s('pop_first')
'heart'
>>> s('str')
"('diamond', ('spade', ('club', None)))"

```

In principle, the operations `push_first` and `pop_first` suffice to make arbitrary changes to a list. We can always empty out the list entirely and then replace its old contents with the desired result.

## 2.4.5 Dictionaries（字典）

 A dictionary contains key-value pairs, where both the keys and values are objects. The purpose of a dictionary is to provide an abstraction for storing and retrieving values that are indexed not by consecutive integers, but by descriptive keys.
 
 字典包含了键值对，其中键和值都可以是对象。字典的目的是提供一种抽象，用于储存和获取下标不是整数，而是描述性的键的值。
 
 Looking up values by their keys uses the element selection operator that we previously applied to sequences.
 
 我们可以使用元素选择运算符，来通过键查找值，我们之前将其用于序列。

```
>>> numerals['X']
10

```

A dictionary can have at most one value for each key. Adding new key-value pairs and changing the existing value for a key can both be achieved with assignment statements.

字典的每个键最多只能拥有一个值。添加新的键值对或者修改某个键的已有值，可以使用赋值运算符来完成。

```
>>> numerals['I'] = 1
>>> numerals['L'] = 50
>>> numerals
{'I': 1, 'X': 10, 'L': 50, 'V': 5}
```
Notice that `'L'` was not added to the end of the output above. Dictionaries are unordered collections of key-value pairs.

要注意，`'L'`并没有添加到上面输出的末尾。字典是无序的键值对集合。

Dictionaries do have some restrictions:

*   A key of a dictionary cannot be an object of a mutable built-in type.
*   There can be at most one value for a given key.

【*16】约束：
---

1、key必须是不可变对象
---

2、每一个key只能有一个value
---

Dictionaries also have a comprehension syntax analogous to those of lists and generator expressions. Evaluating a dictionary comprehension yields a new dictionary object.

【*17】字典推导式：字典也拥有推导式语法，和列表推导式和生成器表达式类似。求解字典推导式会产生新的字典对象。
---


```
>>> {x: x*x for x in range(3,6)}
{3: 9, 4: 16, 5: 25}
```
**Implementation.** We can implement an abstract data type that conforms to the dictionary abstraction as a list of records, each of which is a two-element list consisting of a key and the associated value.

我们可以实现一个抽象数据类型，它是一个记录的列表，与字典抽象一致。每个记录都是两个元素的列表，包含键和相关的值。

```
def make_dict():
    """Return a functional implementation of a dictionary."""
    records = []

    def getitem(key):
        for k, v in records:
            if k == key:
                return v

    def setitem(key, value):
        for item in records:
            if item[0] == key:
                item[1] = value
                return
        records.append([key, value])

    def dispatch(message, key=None, value=None):
        if message == 'getitem':
            return getitem(key)
        elif message == 'setitem':
            setitem(key, value)
            #print(records)
        elif message == 'keys':
            return tuple(k for k, _ in records)
        elif message == 'values':
            return tuple(v for _, v in records)

    return dispatch

d = make_dict()
print(d('setitem', 3, 9))
print(d('setitem', 4, 16))
print(d('getitem', 3))
print(d('keys'))
print(d('values'))
```
