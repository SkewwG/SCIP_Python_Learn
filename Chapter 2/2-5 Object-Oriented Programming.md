# 2.5 Object-Oriented Programming
【*0】类、方法、继承和点运算符
---

【*1】面向对象和数据抽象的3条相同点：↓
---

**1、Like abstract data types, objects create an abstraction barrier between the use and implementation of data
与抽象数据类型一样, 对象在数据的使用和实现之间创建一个抽象边界**

**2、objects respond to behavioral requests.
对象响应行为请求**

**3、objects have local state that is not directly accessible from the global environment.
对象拥有局部状态，并且不能直接从全局环境访问**

OOP：将数据抽象实现出来

Each object bundles together local state and behavior in a way that hides the complexity of both behind a data abstraction.

每个对象以一种隐藏数据抽象背后的复杂性的方式将局部状态和行为绑定在一起。

Not only do objects pass messages, they also share behavior among other objects of the same type and inherit characteristics from related types.

对象不仅投递消息, 它们还在同一类型的其他对象之间共享行为, 并从相关类型继承特征。

We have seen that an object is a data value that has methods and attributes, accessible via dot notation.

我们已经看到, 一个对象是一个数据值, 它具有方法和属性, 可通过点表示法访问。

## 2.5.1 Objects and Classes（对象和类）

A class serves as a template for all objects whose type is that class. Every object is an instance of some particular class.

类是所有对象的模板, 其类型为该类。每个对象都是某个特定类的实例。

The act of creating a new object instance is known as _instantiating_ the class.

创建对象的行为叫做对象实例化

An _attribute_ of an object is a name-value pair associated with the object, which is accessible via dot notation.

对象的属性是与该对象关联的名称-值对, 可通过点表示法访问它。

instance attributes may also be called _fields_, _properties_, or _instance variables_.

实例属性：字段、属性、实例变量

The side effects and return value of a method can depend upon, and change, other attributes of the object.

产生的副作用和方法的返回值依赖于对象的其他属性。


## 2.5.2 Defining Classes（类的定义）

When a class statement is executed, a new class is created and bound to &lt;name&gt; in the first frame of the current environment. The suite is then executed. Any names bound within the &lt;suite&gt; of a class statement, through def or assignment statements, create or modify attributes of the class.

**当类语句执行后，一个新的类被创建，并且在当前环境的第一个栈帧上绑定到name。然后suite语句执行，在class语句内的suite里，任何name绑定。通过def或者赋值语句，创建或者修改类的属性**

Classes are typically organized around manipulating instance attributes,

类通常围绕操作实例属性来组织

The class specifies the instance attributes of its objects by defining a method for initializing new objects.

类通过定义初始化新对象的方法来指定对象的实例属性
 
The method that initializes objects has a special name in Python,

初始化对象的方法在 Python 中有一个特殊的名称,

**Identity**
Object identity is compared using the `is` and `is not` operators.

身份认证：通过is 和 is not去比较

As usual, binding an object to a new name using assignment does not create a new object.

**通过赋值语句给一个对象重新绑定一个新的变量名是不会创建一个对象的，只是对象的引用。**

New objects that have user-defined classes are only created when a class (such as `Account`) is instantiated with call expression syntax.

**当我们定义的那些class，在通过调用表达式语法来调用的时候，才会去初始化一个类的对象。**


**Methods.** Object methods are also defined by a `def` statement in the suite of a `class` statement.

方法.对象方法通过 def 语句定义。

Each method definition again includes a special first parameter `self`, which is bound to the object on which the method is invoked.

每个方法定义都包含一个特殊的第一个参数 self, 它绑定到调用该方法的对象。

【*2】self，把函数绑定给self，self就是类自己本身：什么时候绑定？函数调用的时候才绑定。
---

When a method is invoked via dot notation, the object itself (bound to `tom_account`, in this case) plays a dual role. First, it determines what the name `withdraw` means; `withdraw` is not a name in the environment, but instead a name that is local to the `Account` class. Second, it is bound to the first parameter `self` when the `withdraw` method is invoked. The details of the procedure for evaluating dot notation follow in the next section.

当方法通过点调用，对象扮演了2个角色
1、检查withdraw是什么
2、当调用方法时, 它绑定到第一个参数 self。


## 2.5.3 Message Passing and Dot Expressions

The central idea in message passing was that data values should have behavior by responding to messages that are relevant to the abstract type they represent. Dot notation is a syntactic feature of Python that formalizes the message passing metaphor. 

消息传递中的中心思想是, 数据值应该具有响应与它们所表示的抽象类型相关的消息的行为。点表示法是 Python 的一种句法特征, 它形式化了消息传递的隐喻。

```
<expression> . <name>
```
**<name>必须是个简单的名称（而不是求值为name的表达式）
 
例如：正确的：list.append       错误的：list.('append')**

Using `getattr`, we can look up an attribute using a string, just as we did with a dispatch dictionary.

使用 "getattr", 我们可以使用字符串查找属性, 就像我们使用调度字典一样。

```
>>> getattr(tom_account, 'balance')
10

```

We can also test whether an object has a named attribute with `hasattr`.

我们还可以测试一个对象是否具有具有 hasattr 的名字属性。

```
>>> hasattr(tom_account, 'deposit')
True
```

**Method and functions.**

 Method：the object is bound to the parameter `self`.
 
【*3】如果和self绑定了，那就是method，没绑定就是function。Method指代一种特殊的Function，本质都是Function
---

function：     `class.函数名称`
method ：      `obejct.函数名称`


As an attribute of a class, a method is just a function, but as an attribute of an instance, it is a bound method:

作为类的一个属性, 方法只是一个函数, 但作为一个实例的属性, 它是一个绑定方法:

``` python
>>> class A:
    def hello():
        pass

>>> A.hello()
>>> type(A.hello)       # 这是类本身调用，没有绑定，所以是function
<class 'function'>
>>> type(A().hello)     # 这是对象调用，绑定了，所以是Method
<class 'method'>
```

【*4】@staticmethod
---

<font color=#FF0000  size=5  face="楷体">静态方法就是function，没有绑定类。</font>
**静态方法与一般的函数没有区别，所有参数没有一个和class相关。
因为在有些复杂类的创建过程中，想要用的那个function和这个类的关系不一定需要绑定关系，只是纯粹作为一个工具供你这个类使用，**
<font color=#FF0000  size=5  face="楷体">静态方法可以被所有的实例所调用，也可以被类调用</font>
``` python
>>> class B:
    @staticmethod
    def hi(a):
        return a

>>> type(B.hi)
<class 'function'>
>>> type(B().hi)
<class 'function'>
>>> B.hi(1)                     # 类调用
1
>>> B().hi(2)                   # 实例调用
2
```
【*5】@staticmethod用途(跟类有关系的功能但在运行时又不需要实例和类参与的情况下:即将函数放在类中, 但不能访问该类的实例.注意：@staticmethod定义的函数没有self参数)：↓
---
经常有一些跟类有关系的功能但在运行时又不需要实例和类参与的情况下需要用到静态方法**.** 比如更改环境变量或者修改其他类的属性等能用到静态方法**.** 这种情况可以直接用函数解决, 但这样同样会扩散类内部的代码，造成维护困难<meta charset="utf-8">

比如这样:

```

IND = 'ON'
def checkind():
    return (IND == 'ON')
class Kls(object):
     def __init__(self,data):
        self.data = data
def do_reset(self):
    if checkind():
        print('Reset done for:', self.data)
def set_db(self):
    if checkind():
        self.db = 'new db connection'
        print('DB connection made for:',self.data)
ik1 = Kls(12)
ik1.do_reset()
ik1.set_db()
```

**如果使用@staticmethod就能把相关的代码放到对应的位置了**

```

IND = 'ON'
class Kls(object):
    def __init__(self, data):
        self.data = data
    @staticmethod
    def checkind():
        return (IND == 'ON')
    def do_reset(self):
        if self.checkind():
            print('Reset done for:', self.data)
    def set_db(self):
        if self.checkind():
            self.db = 'New db connection'
        print('DB connection made for: ', self.data)
ik1 = Kls(12)
ik1.do_reset()
ik1.set_db()

```


【*6】@classmethod
---
**类方法就是Method，都绑定了类。可以被所有的实例所调用，也可以被类调用，**
**是被所有的实例共用的，它的状态和类相关，而不是说和实例相关。**
**比如：银行的利率是所有账户都一样的，所以可以用classmethod、**
``` python
>>> class C:
    @classmethod
    def ci(self, a):
        return a

>>> type(C.ci)
<class 'method'>
>>> type(C().ci)
<class 'method'>
>>> C().ci(1)                   # 实例调用
1
>>> C.ci(2)                     # 类调用
2
```

【*7】@classmethod用处(只在类中运行而不在实例中运行的方法：即在func2方法内可以使用类及其属性, 而不是特定的实例。注意：@classmethod定义的方法有一个参数是cls，代表类本身)：↓
---

``` python
class A(object):
    bar = 1
    def func1(self):  
        print ('foo') 
    @classmethod
    def func2(cls):
        print ('func2')
        print (cls.bar)
        cls().func1()   # 调用 foo 方法
 
A.func2()               # 不需要实例化
```
<meta charset="utf-8">

如果现在我们想写一些仅仅与类交互而不是和实例交互的方法会怎么样呢？我们可以在类外面写一个简单的方法来做这些，但是这样做就扩散了类代码的关系到类定义的外面. 如果像下面这样写就会导致以后代码维护的困难:

```

def get_no_of_instances(cls_obj):
    return cls_obj.no_inst
class Kls(object):
    no_inst = 0
    def __init__(self):
        Kls.no_inst = Kls.no_inst + 1
ik1 = Kls()
ik2 = Kls()
print(get_no_of_instances(Kls))

```
@classmethod
我们要写一个只在类中运行而不在实例中运行的方法 如果我们想让方法不在实例中运行，可以这么做:

使用@classmethod装饰器来创建类方法**.**

```

class Kls(object):
    no_inst = 0
    def __init__(self):
        Kls.no_inst = Kls.no_inst + 1
    @classmethod
    def get_no_of_instance(cls_obj):
        return cls_obj.no_inst
ik1 = Kls()
ik2 = Kls()
print ik1.get_no_of_instance()
print Kls.get_no_of_instance()

```

输出:
2
2
这样的好处是: 不管这个方式是从实例调用还是从类调用，它都用第一个参数把类传递过来

【*8】英文解释：↓
---

``` python
@classmethod means: when this method is called, we pass the class as the first argument instead of the instance of that class (as we normally do with methods). This means you can use the class and its properties inside that method rather than a particular instance
@staticmethod means: when this method is called, we don't pass an instance of the class to it (as we normally do with methods). This means you can put a function inside a class but you can't access the instance of that class (this is useful when your method does not use the instance).
@classmethod 意味着: 当调用此方法时, 我们将类作为第一个参数传递, 而不是该类的实例 (我们通常使用方法)。这意味着您可以在该方法中使用类及其属性, 而不是特定的实例
@staticmethod 意味着: 当调用此方法时, 我们不会将类的实例传递给它 (正如我们通常使用方法那样)。这意味着可以将函数放在类中, 但不能访问该类的实例 (当方法不使用该实例时, 这很有用)。
```


We can call `deposit` in two ways: as a function and as a bound method. In the former case, we must supply an argument for the `self` parameter explicitly. In the latter case, the `self` parameter is bound automatically.
```
>>> Account.deposit(tom_account, 1001)  # The deposit function requires 2 arguments
1011
>>> tom_account.deposit(1000)           # The deposit method takes 1 argument
2011
```
【*9】调用方法2种：↓
---
1、类名.方法(实例的对象，参数值)
---
2、实例的对象.方法(参数值)
---


In some cases, there are instance variables and methods that are related to the maintenance and consistency of an object that we don't want users of the object to see or use. They are not part of the abstraction defined by a class, but instead part of the implementation. Python's convention dictates that if an attribute name starts with an underscore, it should only be accessed within methods of the class itself, rather than by users of the class.

【*10】下划线
---
**“单下划线” 开始的成员变量叫做保护变量，意思是只有类对象和子类对象自己能访问到这些变量；**
**“双下划线” 开始的是私有成员，意思是只有类对象自己能访问，连子类对象也不能访问到这个数据。**


## 2.5.4 Class Attributes（类属性）

【*11】类属性（静态变量）：定义在suite里面，即在Class里面，在Method外面。
---
Some attribute values are shared across all objects of a given class.

**类属性：一些属性在所有对象中共享**

Class attributes are created by assignment statements in the suite of a `class` statement, outside of any method definition.

**定义在suite里面，即在Class里面，在Method外面。**

class attributes may also be called class variables or static variables.

**类属性也可以称为类变量或静态变量。**

This attribute can still be accessed from any instance of the class.

**可以被这个类的任何实例访问**

However, a single assignment statement to a class attribute changes the value of the attribute for all instances of the class.

【*11】对类属性的单个赋值语句会更改该类的所有实例的属性的数值。
---

To evaluate a dot expression:

1.  Evaluate the `&lt;expression&gt;` to the left of the dot, which yields the _object_ of the dot expression.
2.  `&lt;name&gt;` is matched against the instance attributes of that object; if an attribute with that name exists, its value is returned.
3.  If `&lt;name&gt;` does not appear among instance attributes, then `&lt;name&gt;` is looked up in the class, which yields a class attribute value.
4.  That value is returned unless it is a function, in which case a bound method is returned instead.

**为了求解点表达式：**
1.  求出点左边的`<expression>`，会产生点运算符的对象。
2.  `<name>`会和对象的实例属性匹配；如果该名称的属性存在，会返回它的值。
3.  如果`<name>`不存在于实例属性，那么会在类中查找`<name>`，这会产生类的属性值。
4.  这个值会被返回，如果它是个函数，则会返回绑定方法。

## 2.5.5 Inheritance（继承）

Two classes may have similar attributes, but one represents a special case of the other.

两个类可能拥有相似的属性，但是一个表示另一个的特殊情况。

base class    基类

subclass      子类

重载：方法名相同，参数个数相同，但参数类型可能不同

重写：方法名相同，参数个数也可能不同

With inheritance, we only specify what is different between the subclass and the base class. Anything that we leave unspecified in the subclass is automatically assumed to behave just as it would for the base class.

**在继承的基础上, 我们只指定子类和基类之间的不同。我们在子类中保留的任何未指定的内容都将自动假定为与基类一样的行为。**

## 2.5.6 Using Inheritance（使用继承）

We can define this procedure recursively. To look up a name in a class.

1.  If it names an attribute in the class, return the attribute value.
2.  Otherwise, look up the name in the base class, if there is one.

我们通过将基类放置到类名称后面的圆括号内来指定继承。

1.  如果类中有带有这个名称的属性，返回属性值。

2.  否则，如果有基类的话，在基类中查找该名称。


## 2.5.7 Multiple Inheritance（多重继承）
AsSeenOnTVAccount, CheckingAccount, SavingsAccount, Account, object
[![](https://github.com/wizardforcel/sicp-py-zh/raw/master/img/multiple_inheritance.png)

【*12】多继承的查找顺序：
---

对于像这样的简单“菱形”，Python 从左到右解析名称，之后向上。这个例子中，Python 按下列顺序检查名称，直到找到了具有该名称的属性：当前类，上一层的从左到右找，再上一层找
---

``` python
class D(A, B, C)
D.__mro__    (D, A, B, C)
```
