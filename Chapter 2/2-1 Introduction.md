# 第二章节
## 面向对象编程
### 2.1.1、介绍
### 2.1.2、思想方法：数据抽象

**compound data types 复合数据类型**

### 2.1.1 对象隐喻（不是比喻）：编程领域 引入 “对象”这种模式，来解决问题。
`functions performed operations and data were operated upon. When we included function values among our data, we acknowledged that data too can have behavior. Functions could be operated upon like data, but could also be called to perform computation.`

function是一个行为，被引用到数据，数据也就有了行为；
函数可以向数据一样被操作，也可以被调用

`Objects represent information, but also _behave_ like the abstract concepts that they represent. The logic of how an object interacts with other objects is bundled along with the information that encodes the object's value.`
对象：数据和行为；
  **对象和对象的交互是用对象内的值（数据和函数）进行联系**

`Objects are both information and processes, bundled together to represent the properties, interactions, and behaviors of complex things.`
对象即是数据也是处理过程，用来表达属性、交互和行为


引用object解决复杂的数据

`The name `date` is bound to a _class_. A class represents a kind of object. Individual dates are called _instances_ of that class, and they can be _constructed_ by calling the class as a function on arguments that characterize the instance.`
class代表了一类的oebjce
**独立的对象（boject）叫做类的实例（instances）**


我的理解是
车是一个类
什么什么车是对象
某个具体的车是实例

`Objects have _attributes_, which are named values that are part of the object. In Python, we use dot notation to designated an attribute of an object.`
对象具有属性，属性是对象的一部分。
**属性（被命名的value）**

`> <expression> . <name>`
today.year理解
**today是表达式，表达式求值之后就是对象**
所以用expressioopn.name


`Objects also have _methods_, which are function-valued attributes. Metaphorically, the object "knows" how to carry out those methods. Methods compute their results from both their arguments and their object. For example, The `strftime` method of `today` takes a single argument that specifies how to display a date (e.g., `%A` means that the day of the week should be spelled out in full).`
对象也有方法，方法是函数的属性



### 2.1.2原始数据类型
`Every object in Python has a _type_.`
python里的每个对象都有一个类型
`So far, the only kinds of objects we have studied are numbers, functions, Booleans, and now dates.`
数字、函数、布尔、日期、集合、字符串

```
Native data types have the following properties:
1.  There are primitive expressions that evaluate to objects of these types, called _literals_.
2.  There are built-in functions, operators, and methods to manipulate these objects.
```
遵循2条规则
原始表达式对对象求值（标量）
内置函数、操作符、方法

```In fact, Python includes three native numeric types: integers (`int`), real numbers (`float`), and complex numbers (`complex`).```
python3中数值类型：整数、实数、复数






