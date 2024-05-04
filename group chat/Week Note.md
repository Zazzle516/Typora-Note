# 2024/3/17

大概交流了一下截至到第二章的进度

- get 了一个抽象语法树的网址：[AST explorer](https://astexplorer.net/) 

- get 了 lua 这种脚本语句，用于各种语言之间的转换
  - ​	类似的有提到 duktape 用于在 C++ 和 JavaScript 之间做转换
  - Rust ffi 在 C 语言之间进行交互

​	codegen：[首页 - CodeGen工具箱](https://cloud.codegen.cc/) 看起来很实用



我顺便贴一些我想放上来的链接

不推荐，答案不好找：[CS 61A Summer 2020 (berkeley.edu)](https://inst.eecs.berkeley.edu/~cs61a/su20/) 

推荐：[CS 61A Fall 2021 (berkeley.edu)](https://inst.eecs.berkeley.edu/~cs61a/fa21/) 

可视化执行过程：https://pythontutor.com/render.html



在示例程序上添加了一些注释，通信依赖的是对**外层函数环境**的调用

我写了大致的执行流程，可以看一下

```python
def make_instance(cls):
    # 属于某个特定实例的变量和实例变量的访问方法
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
    # 只提供最基础的访问
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
    # class_method and static_attribues
    # 属于 account 类的特有方法和静态变量
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
# return cls

# 调用 make_account_class() 中传递的 attributes 
jim_account = Account['new']('Jim')
# make_class().new() -> init_instance().make_instance() -> make_instance().attributes
# init: 从 make_account_class() 中得到 __init__() 函数体
# 通过 cls 执行 make_class().get_value('__init__') -> attributes[__init__]
> 这里是因为最开始的 Account = make_account_class() 这里传递了 attributes 参数进去
> 而 jim_account 又是通过 Account 去访问的 所以可以读取到 make_account_class().attributes
# 为 jim_account 分配独立的变量空间
# return make_instance().instance	!注意返回的内容

jim_account['get_value']('holder')
# 直接找到 make_instance().instance 函数中的 get_value() 函数体 (通过地址)
# 因为处于该函数的运行环境中 可以访问到属于 jim_account 的变量 
# 通过 HOF 进行二次传参 'holder' attributes[name] = 'Jim'

jim_account['get_value']('interest')
# ! 这里看着和上面差不多 但是执行的过程完全不同 因为这里访问的是静态变量 在类中定义的变量 !
# 同样是执行 make_instace().get_value() 但是这里会进入 else-branch 因为没有 interest 存储
# 其余的类似
```



# 2024/3/24

hls 直播

trmp

QUIC UDP



# 2024/4/6

git push ssh 和 http 的 get 区别

smart protocal / http protocal

git clone 过程

第一次请求

从客户端先发起一次 get 请求 info/refs

第二次请求

POST Github 更偏向业务层 /git-upload-pack



用户鉴权 http basic auth



SSH

git 本身就等价于 ssh git 只是 ssh 的包装

ssh鉴权，公钥私钥，ssh 双工通信

极狐gitlib（只要装了 open-ssh 就支持 22 端口



gitdiff to html



控制台 控制协议 ANSI escape code



# 2024/5/4

直接提前计算出一些 AST 分支，简化汇编的压力，不过一般不用，很难控制

P2P 也是会用的，比如舍友看 B 站视频，我看相应的视频就不用从 B 站拉信息了
