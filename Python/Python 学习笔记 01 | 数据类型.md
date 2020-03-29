[TOC]

## 序列

Python 最基本的数据结构 - **序列（sequence）**

**序列**的一个特点就是根据索引（index，即元素的位置）来获取序列中的元素，第一个索引是 0，第二个索引是 1，以此类推。

所有序列类型都可以进行某些通用的操作，比如：

- 索引（indexing）
- 分片（sliceing）
- 迭代（iteration）
- 加（adding）
- 乘（multiplying）

除了上面这些，我们还可以检查某个元素是否属于序列的成员，计算序列的长度等等。

## 常用数据类型

- 列表（list）
- 元组（tuple）
- 字符串（string）
- 字典（dict）
- 集合（set）

其中，**列表、元组和字符串都属于序列类型，它们可以进行某些通用的操作**，比如索引、分片等；字典属于**映射**类型，每个元素由键（key）和值（value）构成；集合是一种特殊的类型，它所包含的元素是不重复的。

### 通用的序列操作

#### 索引

序列中的元素可以通过索引获取，索引从 0 开始。看看下面的例子：

```python
>>> nums = [1, 2, 3, 4, 5]   # 列表
>>> nums[0]
1
>>> nums[1]
2
>>> nums[-1]                 # 索引 -1 表示最后一个元素
5
>>> s = 'abcdef'             # 字符串
>>> s[0]
'a'
>>> s[1]
'b'
>>>
>>> a = (1, 2, 3)            # 元组
>>> a[0]
1
>>> a[1]
2
```

注意到，-1 则代表序列的最后一个元素，-2 代表倒数第二个元素，以此类推。

#### 分片

**索引**用于获取序列中的单个元素，而**分片**则用于获取序列的部分元素。分片操作需要提供两个索引作为边界，中间用冒号相隔，比如：

```python
>>> numbers = [1, 2, 3, 4, 5, 6]
>>> numbers[0:2]                   # 列表分片
[1, 2]
>>> numbers[2:5]
[3, 4, 5]
>>> s = 'hello, world'             # 字符串分片
>>> s[0:5]
'hello'
>>> a = (2, 4, 6, 8, 10)           # 元组分片
>>> a[2:4]
(6, 8)
```

下面列举使用分片的一些技巧。（https://funhacks.gitbooks.io/explore-python/Datatypes/）

- 访问最后几个元素
- 使用步长

这里总结一下使用分片操作的一些方法，分片的使用形式是：

```python
# 左索引:右索引:步长
left_index:right_index:step
```

要牢牢记住的是：

- 左边索引的元素包括在结果之中，右边索引的元素不包括在结果之中；
- 当使用一个负数作为步长时，必须让左边索引大于右边索引；
- 对正数步长，从左向右取元素；对负数步长，从右向左取元素；

## 列表（list）

<font color="red">字符串和元组是不可变的，而列表是可变（mutable）的，可以对它进行随意修改</font>。我们还可以将字符串和元组转换成一个列表，只需使用 `list` 函数，比如：

```python
>>> s = 'hello'
>>> list(s)
['h', 'e', 'l', 'l', 'o']
>>> a = (1, 2, 3)
>>> list(a)
[1, 2, 3]
```

#### 列表的常用方法

https://funhacks.gitbooks.io/explore-python/Datatypes/list.html

| 方法    | 作用                                                         |
| ------- | ------------------------------------------------------------ |
| index   | 从列表中找出某个元素的位置，如果有多个相同的元素，则返回第一个元素的位置 |
| count   | count 方法用于统计某个元素在列表中出现的次数                 |
| append  | append 方法用于在列表末尾增加新的元素                        |
| extend  | extend 方法将一个新列表的元素添加到原列表中                  |
| insert  | insert 方法用于将某个元素添加到某个位置                      |
| pop     | pop 方法用于移除列表中的一个元素（默认是最后一个），并且返回该元素的值 |
| remove  | remove 方法用于移除列表中的某个匹配元素，如果有多个匹配，则移除第一个 |
| reverse | reverse 方法用于将列表中的元素进行反转                       |
| sort    | sort 方法用于对列表进行排序，注意该方法会改变原来的列表，而不是返回新的排序列表，另外，sort 方法的返回值是空 |

- 虽然 append 和 extend 可接收一个列表作为参数，但是 append 方法是将其作为一个元素添加到列表中，而 extend 则是将新列表的元素逐个添加到原列表中。

- 如果我们不想改变原列表，而是希望返回一个排序后的列表，可以使用 sorted 函数
- 不管是 sort 方法还是 sorted 函数，默认排序都是升序排序。如果你想要降序排序，就需要指定排序参数了。比如，对 sort 方法，可以添加一个 reverse 关键字参数 `a.sort(reverse=True) ` `sorted(a, reverse=True)`
- 更多排序知识，参考https://wiki.python.org/moin/HowTo/Sorting

## 元组（tuple）

在 Python 中，**元组是一种不可变序列**，它使用圆括号来表示：

```python
>>> a = (1, 2, 3)    # a 是一个元组
>>> a
(1, 2, 3)
>>> a[0] = 6         # 元组是不可变的，不能对它进行赋值操作
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment
```

<font color="red">创建一个值的元组需要在值后面再加一个逗号，这个比较特殊，需要牢牢记住</font>

```python
>>> a = (12,)   # 在值后面再加一个逗号
>>> a
(12,)
>>> type(a)
<type 'tuple'>
>>>
>>> b = (12)    # 只是使用括号括起来，而没有加逗号，不是元组，本质上是 b = 12
>>> b
12
>>> type(b)
<type 'int'>
```

元组也是一种序列，因此也可以对它进行索引、分片等。由于它是不可变的，因此就没有类似列表的 append, extend, sort 等方法。

## 字符串（string）

字符串也是一种序列，因此，通用的序列操作，比如索引，分片，加法，乘法等对它同样适用。

但需要注意的是，字符串和元组一样，也是不可变的，所以你不能对它进行赋值等操作。

除了通用的序列操作，字符串还有自己的方法，比如 join, lower, upper 等。字符串的方法特别多，这里只介绍一些常用的方法

#### 字符串的常用方法

https://funhacks.gitbooks.io/explore-python/Datatypes/string.html

| 方法        | 作用                                                         |
| ----------- | ------------------------------------------------------------ |
| find        | find 方法用于在一个字符串中查找子串，它返回子串所在位置的最左端索引，如果没有找到，则返回 -1 |
| split       | split 方法用于将字符串分割成序列                             |
| join        | join 方法可以说是 split 的逆方法，它用于将序列中的元素连接起来 |
| strip       | strip 方法用于移除字符串左右两侧的空格，但不包括内部，当然也可以指定需要移除的字符串 |
| replace     | replace 方法用于替换字符串中的**所有**匹配项                 |
| translate   | translate 方法和 replace 方法类似，也可以用于替换字符串中的某些部分，但 **translate 方法只处理单个字符** |
| lower/upper | lower/upper 用于返回字符串的大写或小写形式                   |



## 字典（dict）

字典是 Python 中唯一的映射类型，每个元素由键（key）和值（value）构成，<font color="red"> 键必须是不可变类型</font>，比如数字、字符串和元组。

#### 创建字典

字典可以通过下面的方式创建：

```python
>>> d0 = {}    # 空字典
>>> d0
{}
>>> d1 = {'name': 'ethan', 'age': 20}
>>> d1
{'age': 20, 'name': 'ethan'}
>>> d1['age'] = 21          # 更新字典
>>> d1
{'age': 21, 'name': 'ethan'}
>>> d2 = dict(name='ethan', age=20)    # 使用 dict 函数
>>> d2
{'age': 20, 'name': 'ethan'}
>>> item = [('name', 'ethan'), ('age', 20)]
>>> d3 = dict(item)
>>> d3
{'age': 20, 'name': 'ethan'}
```

#### 遍历字典

遍历字典有多种方式，这里先介绍一些基本的方式，后文会介绍一些高效的遍历方式。

```python
>>> d = {'name': 'ethan', 'age': 20}
>>> for key in d:
...     print '%s: %s' % (key, d[key])
...
age: 20
name: ethan
>>> d['name']
'ethan'
>>> d['age']
20
>>> for key in d:
...     if key == 'name':
...         del d[key]         # 要删除字典的某一项
...
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
RuntimeError: dictionary changed size during iteration
>>>
>>> for key in d.keys():  # python2 应该使用这种方式, python3 使用 list(d.keys())
...     if key == 'name':
...         del d[key]
...
>>> d
{'age': 20}
```

在上面，我们介绍了两种遍历方式：`for key in d` 和 `for key in d.keys()`，如果在遍历的时候，要删除键为 key 的某项，使用第一种方式会抛出 RuntimeError，使用第二种方式则不会。

#### 判断键是否在字典里面

有时，我们需要判断某个键是否在字典里面，这时可以用 `in` 进行判断，如下：

```python
>>> d = {'name': 'ethan', 'age': 20}
>>> 'name' in d
True
>>> d['score']             # 访问不存在的键，会抛出 KeyError
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'score'
>>> 'score' in d           # 使用 in 判断 key 是否在字典里面
False
```

#### 字典的常用方法

https://funhacks.gitbooks.io/explore-python/Datatypes/dict.html

| 方法              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| clear             | clear 用于清空字典中的所有项，这是个原地操作，所以无返回值（或者说是 None） |
| copy              | copy 方法实现的是浅复制（shallow copy）它具有以下特点：对可变对象的修改保持同步；对不可变对象的修改保持独立； |
| get               | 当我们试图访问字典中不存在的项时会出现 KeyError，但使用 get 就可以避免这个问题 |
| setdefault        | setdefault 方法用于对字典设定键值                            |
| update            | update 方法用于将一个字典添加到原字典，如果存在相同的键则会进行覆盖 |
| items/iteritems   | items 方法将所有的字典项以列表形式返回，这些列表项的每一项都来自于（键，值）。我们也经常使用这个方法来对字典进行遍历 |
| keys/iterkeys     | keys 方法将字典的键以列表形式返回，iterkeys 则返回针对键的迭代器 |
| values/itervalues | values 方法将字典的值以列表形式返回，itervalues 则返回针对值的迭代器 |
| pop               | pop 方法用于将某个键值对从字典移除，并返回给定键的值         |
| popitem           | popitem 用于随机移除字典中的某个键值对。                     |

#### 推荐的字典遍历方法

```python
>>> d = {'name': 'ethan', 'age': 20}
>>> d.iteritems()
<dictionary-itemiterator object at 0x109cf2d60>
>>> for k, v in d.iteritems():
...     print '%s: %s' % (k, v)
...
age: 20
name: ethan
```

#### 对元素为字典的列表排序

事实上，我们很少直接对字典进行排序，而是对元素为字典的列表进行排序。

比如，存在下面的 students 列表，它的元素是字典：

```python
students = [
    {'name': 'john', 'score': 'B', 'age': 15},
    {'name': 'jane', 'score': 'A', 'age': 12},
    {'name': 'dave', 'score': 'B', 'age': 10},
    {'name': 'ethan', 'score': 'C', 'age': 20},
    {'name': 'peter', 'score': 'B', 'age': 20},
    {'name': 'mike', 'score': 'C', 'age': 16}
]

# 按 score 从小到大排序
>>> sorted(students, key=lambda stu: stu['score'])

# 按 score 从大到小排序
>>> sorted(students, key=lambda stu: stu['score'], reverse=True)  # reverse 参数

# 按 score 从小到大，再按 age 从小到大
>>> sorted(students, key=lambda stu: (stu['score'], stu['age']))

# 按 score 从小到大，再按 age 从大到小
>>> sorted(students, key=lambda stu: (stu['score'], -stu['age']))
```



## 集合（set）

集合（set）和字典（dict）类似，它是一组 key 的集合，但不存储 value。集合的特性就是：key 不能重复。

### 集合常用操作

#### 创建集合

set 的创建可以使用 `{}` 也可以使用 set 函数：

```python
>>> s1 = {'a', 'b', 'c', 'a', 'd', 'b'}   # 使用 {}
>>> s1
set(['a', 'c', 'b', 'd'])
>>>
>>> s2 = set('helloworld')                # 使用 set()，接收一个字符串
>>> s2
set(['e', 'd', 'h', 'l', 'o', 'r', 'w'])
>>>
>>> s3 = set(['.mp3', '.mp4', '.rmvb', '.mkv', '.mp3'])   # 使用 set()，接收一个列表
>>> s3
set(['.mp3', '.mkv', '.rmvb', '.mp4'])
```

#### 遍历集合

```python
>>> s = {'a', 'b', 'c', 'a', 'd', 'b'}
>>> for e in s:
...     print e
...
a
c
b
d
```

#### 添加元素

`add()` 方法可以将元素添加到 set 中，可以重复添加，但没有效果。

```python
>>> s = {'a', 'b', 'c', 'a', 'd', 'b'}
>>> s
set(['a', 'c', 'b', 'd'])
>>> s.add('e')
>>> s
set(['a', 'c', 'b', 'e', 'd'])
>>> s.add('a')
>>> s
set(['a', 'c', 'b', 'e', 'd'])
>>> s.add(4)
>>> s
set(['a', 'c', 'b', 4, 'd', 'e'])
```

#### 删除元素

`remove()` 方法可以删除集合中的元素，但是删除不存在的元素，会抛出 KeyError，可改用 `discard()`。

```python
>>> s = {'a', 'b', 'c', 'a', 'd', 'b'}
>>> s
set(['a', 'c', 'b', 'd'])
>>> s.remove('a')           # 删除元素 'a'
>>> s
set(['c', 'b', 'd'])
>>> s.remove('e')           # 删除不存在的元素，会抛出 KeyError
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'e'
>>> s.discard('e')          # 删除不存在的元素, 不会抛出 KeyError
```

#### 交集 / 并集 / 差集

Python 中的集合也可以看成是数学意义上的无序和无重复元素的集合，因此，我们可以对两个集合作交集、并集等。

```python
>>> s1 = {1, 2, 3, 4, 5, 6}
>>> s2 = {3, 6, 9, 10, 12}
>>> s3 = {2, 3, 4}
>>> s1 & s2            # 交集
set([3, 6])
>>> s1 | s2            # 并集
set([1, 2, 3, 4, 5, 6, 9, 10, 12])
>>> s1 - s2            # 差集
set([1, 2, 4, 5])
>>> s3.issubset(s1)    # s3 是否是 s1 的子集
True
>>> s3.issubset(s2)    # s3 是否是 s2 的子集
False
>>> s1.issuperset(s3)  # s1 是否是 s3 的超集
True
>>> s1.issuperset(s2)  # s1 是否是 s2 的超集
False
```