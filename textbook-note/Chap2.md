# Abstract Object

用生成器表达式的时候，注意变量的意义

```python
def sum_even_fib(n):
    # return sum(fib(i) for i in range(1,n + 1) if is_even(i) )
    return sum(fib(i) for i in range(1,n + 1) if is_even( fib(i) ))

# 被注释掉的是我自己写的 后面的 is_even(i) 判断的是下标为偶数的 fib 数的和
# 而不是偶数 fib 的和
```



多重继承的相关问题

[Python多重继承问题-MRO和C3算法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/151856162) 



### 函数式编程实现面向对象

```python
def make_instance(cls):
    def get_value(name):
        if name in attributes:
            return attributes[name]
        else:
            value = cls['get_value'](name)
            return bind_method(value, instance)

    def set_value(name, value):
        attributes[name] = value

    attributes = {}
    instance = {'get_value': get_value, 'set_value': set_value}
    return instance

def bind_method(value, instance):
    if callable(value):
        def method(*args):
            return value(instance, *args)
        return method
    else:
        return value
    
def make_class(attributes, base_class = None):
    def get_value(name):
        if name in attributes:
            return attributes[name]
        elif base_class is not None:
            return base_class['get_value'](name)

    def set_value(name, value):
        attributes[name] = value

    def new(*args):
        return init_instance(cls, *args)
    
    cls = {'get_value': get_value,
           'set_value': set_value,
           'new': new}
    
    return cls

def init_instance(cls, *args):
    instance = make_instance(cls)
    init = cls['get_value']('__init__')
    if init:
        init(instance, *args)
    return instance

def make_account_class():
    def __init__(self, account_holder):
        self['set_value']('holder', account_holder)
        self['set_value']('balance', 0)
    
    def deposit(self, amount):
        new_balance = self['get_value']('balance') + amount
        self['set_value']('balance', new_balance)
        return self['get_value']('balance')
    
    def withdraw(self, amount):
        balance = self['get_value']('balance')
        if amount > balance:
            return 'Insufficient funds'
        self['set_value']('balance', balance - amount)
        return self['get_value']('balance')
    
    return make_class({'__init__': __init__,
                       'deposit': deposit,
                       'withdraw': withdraw,
                       'interest': 0.02})

Account = make_account_class()

jim_account = Account['new']('Jim')

jim_account['get_value']('holder')
jim_account['get_value']('interest')

print(jim_account['get_value']('deposit')(20))

print(jim_account['get_value']('withdraw')(5))

Ann_account = Account['new']('Ann')

print(Ann_account['get_value']('deposit')(50))
```





针对 `attribute` 变量的确定问题，GPT 给了不错的回答

```
在 `make_account_class()` 函数内部，`attributes` 变量作为一个局部变量，被定义在函数的作用域中。然后，`get_value()` 函数被定义为 `make_account_class()` 函数内部的一个内部函数。由于 Python 中的闭包特性，内部函数可以访问外部函数的局部变量。这意味着 `get_value()` 函数可以访问 `make_account_class()` 函数中定义的 `attributes` 变量，即使 `get_value()` 被调用时已经离开了 `make_account_class()` 函数的作用域。

所以，`get_value()` 函数在执行时可以直接访问到 `make_account_class()` 函数中定义的 `attributes` 变量，因为它们之间形成了一个闭包关系，`attributes` 变量被捕获并保存在 `get_value()` 函数的闭包中。
```



```
当一个外部函数通过字典调用 `get_value()` 函数时，会在该外部函数的函数帧中创建一个新的仅包含 `get_value()` 函数的子函数帧，而不是在全局函数帧中创建。这是因为字典中存储的函数是函数对象的引用，当被调用时，Python 会在当前的函数帧中创建一个新的局部函数帧以执行这个函数。

具体地说，当外部函数调用字典中存储的函数时，Python 解释器会在外部函数的函数帧中创建一个新的子函数帧来执行这个函数。这个子函数帧将继承外部函数的局部变量和其他执行上下文，但它会拥有自己的局部命名空间和执行环境。这样做可以确保在函数调用时，每个函数都有自己独立的执行环境，不会影响到其他函数的执行。
```



### TypeClass	祖先类

所有的 Python User_defined_Classes 都是 Type_class 的实例





### Meta-class	元类

允许在定义类的时候动态控制类的创建过程，可以对类进行定制化，扩展功能

并在类的实例化过程中执行额外的操作

元类：用来创建类的类

语法：通过 `type()` 函数进行创建，平时在创建类的时候其实是编译器隐式调用了 `type()` 

```python
def custom_init(self, name):
    self.name = name

CustomClass = type('CustomClass', (object,), {'__init__': custom_init})
```

元类的应用：

- 框架开发
- ORM

#### A

```python
class ModelMetaClass(type):
    def __new__(cls, name, bases, attrs):
        if name != 'BaseModel':
            attrs['table_name'] = name.lower()
        return super().__new__(cls, name, bases, attrs)

class BaseModel(metaclass=ModelMetaClass):
    # metaclass 是用来指定元类的特殊关键字
    pass

class User(BaseModel):
    pass

# 元类同样可以影响继承类
print(User.table_name)  # 输出：user
```



#### 接口规范

通过元类定义接口规范

```python
class InterfaceMetaClass(type):
    def __new__(cls, name, bases, attrs):
        if '__abstractmethods__' not in attrs:
            abstractmethods = set()
            for base in bases:
                abstractmethods.update(getattr(base, '__abstractmethods__', set()))
            for attr_name, attr_value in attrs.items():
                if callable(attr_value) and attr_name not in abstractmethods:
                    raise TypeError(f"Class '{name}' does not implement required method '{attr_name}'")
        return super().__new__(cls, name, bases, attrs)

class Interface(metaclass=InterfaceMetaClass):
    pass

class MyInterface(Interface):
    def method1(self):
        pass

class MyClass(MyInterface):
    def method1(self):
        pass

class InvalidClass(MyInterface):
    pass
```



### Dispatching Ty

[cs61a 课时笔记 对象的抽象_cs61a 抽象障碍-CSDN博客](https://blog.csdn.net/qq_23869697/article/details/106037460) 
