# Iterators Generators OOP

## Iterators

没什么好说的



## Generators

### 2.1

我居然搞出来一道我好激动（虽然正确答案可能是把找子列的方法写在了里面，但是那样真的超出我的能力了

附带几个思考子列的失败案例

```python
# def find_sub_list(num_list):
#     # 找到从 None->num 的所有子串   递归
#     if len(num_list) == 0:
#         return []
#     return [[i] + [num_list[-1]] for i in find_sub_list(num_list[:-1])]

# 会直接退出返回 [] 对比了一下 lab-7 的实现 问题出现对递归终点的判断
# 如果 len(num_list) == 0 那么在递归的循环中会直接结束 从而退出 无法继续后面的递归

# def find_sub_list(num_list):
#     # 找到从 None->num 的所有子串   递归
#     if len(num_list) <= 1:
#         return [[], num_list] if num_list != [] else [[]]
#     return [i.append(num_list[-1]) for i in find_sub_list(num_list[:-1])]


# 还有个问题就是没有把目前找到的子串加上    看来辅助函数不可避免
def add_last(elem, sub_list):
    return [i + [elem] for i in sub_list]

def find_sub_list(num_list):
    # 找到从 None->num 的所有子串   递归
    if len(num_list) <= 1:
        return [[], num_list] if num_list != [] else [[]]
    temp = find_sub_list(num_list[:-1])
    return add_last(num_list[-1], temp) + temp

def call_find_sub_list(num=0):
    # 根据 num 返回一个列表调用
    return sorted(find_sub_list(list(range(num))))

def generate_subsets():
    n = 0
    while True:
        yield call_find_sub_list(n)
        n += 1

subsets = generate_subsets()
for _ in range(3):
    print(next(subsets))
```

我找着答案了 TMD 我跟个傻卵一样

```python
def generatoe_subsets():
    subsets = [[]]
    n = 1
    while True:
        yield subsets
        subsets = subsets + [s + [n] for s in subsets]
        n += 1
```

我 TM 怎么想不出来这种简洁的方法，而且也很好懂



### 2.2

因为一开始很难从生成器的角度是思考，所以先想想一般做法

```python
def sum_paths(t):
    # 从 gen 比较难想 先从普通的函数执行开始    问题是无法把结果储存下来
    if is_leaf(t):
        return [label(t)]
    for branch in branches(t):
        for elem in sum_paths(branch):  # [3]
            elem += label(t)
```

说实话我看了答案，也许差不多，但是执行思路上差了很多，大体方向是找到叶子节点，然后逐步递归向上求出和

但是不使用 yield 关键字的话，中间变量无法保存会直接退出，也就是这道题只能用生成器去做

但是生成器的执行过程是完全不同的，会保存父函数帧的状态，并继续运行

（我 Debug 了一遍也没懂它是怎么保存变量的 去 pythontutor 看一下 step_88

```python
def sum_paths(t):
    if is_leaf(t):
        yield label(t)
	for branch in branches(t):
        for elem in sum_path(branch):
            yield elem + label(t)
```

~~我明白了，它是通过 yield 保存的，比如在第一个分支中 yield 3 就会直接保存下来（step_133 返回 5~~

和我最开始想的不太一样，这里是每次都计算到了最终结果，再开始下一个分支的运算

```python
# 中间的递归调用并不会停下 能返回的是最外层的 yield
print(next(sum_paths_gen(t2)))	# 6
```

只有最外层（根节点调用的 yield）才会返回，那是主函数帧（可以这么认为吧）使用的 yield 而在执行该 yield 之前的递归过程并不会因为 yield 停下，所以调用 `next()` 会返回 6 （递归又发生在循环函数体的上一步，所以就会紧跟着执行循环体，加上上一层的 label）

在 step_128 的时候执行到叶子节点 3 开始返回，回到上一层进行求和的操作，得到 5，然后并不会停下，回到更上一层，step_137 也就是 root 层，求和得到 6 并且返回该值，因为 `sorted()` 的作用，继续执行.

(理论上来说这个时候结束可以进入下一个分支，但是从 step_140 为什么中间还有一段求 3 的过程 要返回 None 保证最上层该轮的循环结束，能看到这个列表里面已经有第一个 s = 3，本质是在求第二个 s，这里发现没有第二个 s 已经跳出内层循环了，回到第一层循环找叶子节点 3 的分支（当然找不到，返回 None )

step_148 其实是从最顶层 (t=1) 突然跳回第二层 (t=2) 继续执行，有点迷惑	可以这么想，因为第二层的循环还没有执行结束，所以还要回来，不过里面具体的执行过程确实没看懂，只能说从整体上理解，以后再看看吧

```python
def tree(label, branches=[]):
    for branch in branches:
        assert is_tree(branch)
    return [label] + list(branches)

# Selector
def label(tree):
    return tree[0]

# Selector
def branches(tree):
    return tree[1:]

def is_tree(tree):
    """Returns True if the given tree is a tree, and False otherwise."""
    if type(tree) != list or len(tree) < 1:
        return False
    for branch in branches(tree):
        if not is_tree(branch):
            return False
    return True

# For convenience
def is_leaf(tree):
    return not branches(tree)

def sum_paths_gen(t):
    if is_leaf(t):
        yield label(t)
    for branch in branches(t):
        for s in sum_paths_gen(branch):
            yield s + label(t)      # 通过在循环中持续运行保存变量 不需要考虑返回

t2 = tree(1,[tree(2, [tree(3), tree(4)]), tree(9)])
print(sorted(sum_paths_gen(t2)))
```



## OOP

### 3.1

就没啥好说的

```python
class Student:
    students = 0
    def __init__(self, name, ta):
        self.name = name
        self.understanding = 0
        Student.students += 1
        print("There are now", Student.students, "students")
        ta.add_student(self)

    def visit_office_hours(self, staff):
        staff.assist(self)

class Professor:
    def __init__(self, name):
        self.name = name
        self.students = {}

    def add_student(self, student):
        self.students[student.name] = student

    def assist(self, student):
        student.understanding += 1

callahan = Professor("Callahan")
elle = Student("ella", callahan)

elle.visit_office_hours(callahan)

elle.visit_office_hours(Professor("Paulette"))

print(elle.understanding)

print([name for name in callahan.students])

x = Student("Vivian", Professor("Stromwell")).name
print(x)

print([name for name in callahan.students])
```



### 3.2

就是在 Client 的实现中，初始化要向 Server 提交自己的信息，完成注册，自己写的时候漏掉了

```python
# 通信流程 clientA -> Server -> ClientB (email)

class Email:
    """
    Every email object has 3 instance sttributes: message, sender_name, recipient_name
    """
    def __init__(self, msg, sender_name, recipient_name):
        self.msg = msg
        self.sender_name = sender_name
        self.recipient_name = recipient_name

class Server:
    """
    Each server has an instance attribute clients, which is a dict that associates clinets names with client object
    """
    def __init__(self):
        self.client = {}    # clinet_name: client_obj
    
    def send(self, email):
        """
        Take an email and put it in the index of the client
        """
        receiver_name = email.recipient_name
        receiver = self.client[receiver_name]
        receiver.receive(email)

    def register_client(self, client, client_name):
        """
        Take a clint obj and client_name and add them to the clients instance attribute
        """
        self.client[client_name] = client

class Client:
    """
    Every Clint has instance attribute name , server, and inbox(a list of all emails the client has received)
    """
    def __init__(self, server, name):
        self.inbox = []     # email obj
        self.name = name
        self.server = server

        self.server.register_client(server, self, name) # !miss 如果不注册就没办法用

    def compose(self, msg, recipient_name):
        """
        Send an email with the given message to the given recipient client
        """
        email = Email(msg, self.name, recipient_name)
        self.server.send(email)
    
    def receive(self, email):
        """
        Take an email and add it to the inbox of this client
        """
        self.inbox.append(email)
```



## Inheritance

### 4.1

```python
# Inheritance
class Pet():
    def __init__(self, name, owner):
        self.is_alive = True
        self.name = name
        self.owner = owner

    def eat(self, thing):
        print(self.name + "ate a " + str(thing) + "!")

    def talk(self):
        print(self.name)

class Cat(Pet):
    def __init__(self, name, owner, lives = 9):
        self.lives = lives
        Pet.__init__(self, name, owner)     # ! 这里要提供 self 变量
    
    def talk(self):
        """
        Print out a cat's greeting
        """
        print(self.name + " says meow!")

    def lose_lives(self):
        """
        Decrements a cat's life by 1, when reach zero, 'is_alive' becomes False
        If called after lives reaches 0, print out the cat has no more lives to lose
        """
        # 这里的逻辑先后有点问题
        # if self.lives == 0:
        #     print(self.name + "has no more lives to lose")
        #     self.is_alive = False
        #     return
        # self.lives -= 1

        if self.lives >= 1:
            self.lives -= 1
            if self.lives == 0:
                self.is_alive = False
        else:
            print(self.name + "has no more lives to lose")
```



### 4.2

完成吵闹猫

```python
class NoisyCat(Cat):
    """
    A cat repeats things twice
    """
    def __init__(self, name, owner, lives = 9):
        Cat.__init__(self, name, owner, lives)

    def talk(self):
        Cat.talk(self)
        Cat.talk(self)
```



### 4.3

```python
class A:
    def f(self):
        return 2
    def g(self, obj, x):
        if x == 0:
            return A.f(obj)
        return obj.f() + self.g(self, x - 1)

class B(A):
    def f(self):
        return 4
    
x, y = A(), B()
print(x.f())
print(B.f())

print(x.g(x, 1))
print(y.g(x, 2))
```



# Done!

找到答案舒服多了，加油吧
