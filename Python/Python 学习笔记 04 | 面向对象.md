[TOC]

## 类和实例

这里以动物（Animal）类为例，Python 提供关键字 `class` 来声明一个类：

```python
class Animal(object):
    pass
```

其中，`Animal` 是类名，通常类名的首字母采用大写（如果有多个单词，则每个单词的首字母大写），后面紧跟着 `(object)`，表示该类是从哪个类继承而来的，所有类最终都会继承自 `object` 类。

类定义好了，接下来我们就可以**创建实例**了：

```python
>>> animal = Animal()  # 创建一个实例对象
>>> animal
<__main__.Animal at 0x1030a44d0>
```

我们在创建实例的时候，还可以传入一些参数，以初始化对象的属性，为此，我们需要添加一个 `__init__` 方法：

```python
class Animal(object):
    def __init__(self, name):
        self.name = name
    def greet(self):
        print 'Hello, I am %s.' % self.name
```

然后，在创建实例的时候，传入参数：

```python
>>> animal = Aniaml('dog1')   # 传入参数 'dog1'
>>> animal.name               # 访问对象的 name 属性
'dog1'
>>> dog1.greet()
Hello, I am dog1.
```

现在，让我们做一下总结。我们在 Animal 类定义了两个方法：`__init__` 和 `greet`。`__init__` 是 Python 中的**特殊方法（special method）**，它用于对对象进行初始化，类似于 C++ 中的构造函数；`greet` 是我们自定义的方法。

注意到，我们在上面定义的两个**方法**有一个共同点，就是它们的第一个参数都是 `self`，指向实例本身，也就是说它们是和实例绑定的函数，这也是我们称它们为方法而不是函数的原因。

### 访问限制

在某些情况下，我们希望限制用户访问对象的属性或方法，也就是希望它是私有的，对外隐蔽。比如，对于上面的例子，我们希望 `name` 属性在外部不能被访问，我们可以**在属性或方法的名称前面加上两个下划线**，即 `__`，对上面的例子做一点改动：

```python
class Animal(object):
    def __init__(self, name):
        self.__name = name
    def greet(self):
        print 'Hello, I am %s.' % self.__name
```

```python
>>> dog1 = Animal('dog1')
>>> dog1.__name   # 访问不了
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-206-7f6730db631e> in <module>()
----> 1 dog1.__name

AttributeError: 'Animal' object has no attribute '__name'
>>> dog1.greet()   # 可以访问
Hello, I am dog1.
```

需要注意的是，在 Python 中，以双下划线开头，并且以双下划线结尾（即 `__xxx__`）的变量是特殊变量，特殊变量是可以直接访问的。所以，不要用 `__name__` 这样的变量名。

另外，如果变量名前面只有一个下划线 `_`，表示**不要随意访问这个变量**，虽然它可以直接被访问。



## 继承和多态

### 继承

```python
class Animal(object):
    def __init__(self, name):
        self.name = name
    def greet(self):
        print 'Hello, I am %s.' % self.name
```

```python
class Dog(Animal):
    def greet(self):
        print 'WangWang.., I am %s. ' % self.name
    def run(self):
        print 'I am running.I am running'
```

```python
>>> animal = Animal('animal')  # 创建 animal 实例
>>> animal.greet()
Hello, I am animal.
>>> 
>>> dog = Dog('dog')        # 创建 dog 实例
>>> dog.greet()
WangWang.., I am dog.
>>> dog.run()
I am running.I am running
```

### 多态

```python
class Cat(Animal):
    def greet(self):
        print 'MiaoMiao.., I am %s' % self.name
        
def hello(animal):
    animal.greet()
```

```python
>>> dog = Dog('dog')
>>> hello(dog)
WangWang.., I am dog.
>>>
>>> cat = Cat('cat')
>>> hello(cat)
MiaoMiao.., I am cat
```