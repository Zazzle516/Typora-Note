# Homework05

## Nonlocal

### Q1: Make Bank

```shell
python ok -q make_bank --local
```

类似于教科书中看的，通过函数和 `dict dispatching` 的方式实现面向对象编程

提供的成员函数包括：

- withdraw
- deposit

对无效消息的判断要怎么实现呢

****

又仔细看了一下啊，不是的，根据 `message` 判断就可以，傻了



### Q2: Password Protected Account

```shell
python ok -q make_withdraw --local
```

只针对取钱进行加密保护，密码错误相对于存款不足优先判断，当密码错误达到 3 次后，任何尝试都会被冻结，哪怕密码正确也不允许再取款。



## Iterators and Generators

对这块语法完全不熟悉....先去学习一下

### Q3: Repeated

```shell
python ok -q repeated --local
```

返回列表中连续出现 k 次的元素，保证每个列表都存在答案，根据顺序进行返回

比如存在两个结果，第一次调用返回第一个，第二次调用返回第二个（呃，可以直接修改原列表达成这种效果）

可以用快慢指针的思路。

```python
def repeated(t, k):
    index_slow = 0
    index_fast = 1
    target_num = None
    while index_fast < len(t):
        flag = 0
        for i in range(0, k - 1):
            if t[index_fast + i] == t[index_slow]:
                flag += 1
            else:
                index_slow = index_fast
                index_fast += 1
        if flag == k - 1:
            target_num = t[index_slow]
            for i in range(0 ,k):
                t.pop(index_slow)
            break
    return target_num

lst = [10, 9, 10, 9, 9, 10, 8, 8, 8, 7]
print(repeated(lst, 2))
print(lst)		# [10, 9, 10, 10, 8, 8, 8, 7]
```

这种方法只能说可以从算法的角度实现，虽然也有问题，实际上 iterator 的类型并不支持 list 相关操作，比如 `len()` `pop()` 之类的，所以也能看到后面的测试用例输入的是 List 类型。

从这个测试中能看到最后的列表有 2 个原本分开的 10 现在拼在了一起，会影响后续的函数二次调用

这种语法上的问题我是真的没想太到....

```python
# Python grammar
# iter(): 迭代器类型
for i in iter(lst):
    print(i)
```



作用到迭代器上，只能采用 `for` 循环，这题的意思难道是让我重写 `__iter__()` 方法？应该不是吧

******

我去看答案了（太好笑了，我现在满脑子是会赢吗🤣

```python
def repeated(t, k):
    # (last_item, item)
    count = 1			# 等效于 index_slow 也占用一个位置
    last_item = None	# 用额外变量记录遍历的上一个元素
    while True:
        item = next(t)
        if item == last_item:
            count += 1
        else:
            last_item = item	# 更新 index_slow
            count = 1
        if count == k:
            return item
```

思路还是相同的，就是从下标换成用元素本身去对比，算是思路盲点的训练

#### 迭代器状态

注意其中的两个测试用例

```shell
>>> s = iter([3, 2, 2, 2, 1, 2, 1, 4, 4, 5, 5, 5])
>>> repeated(s, 3)
2
# 这里二次调用但是返回不一样的值 因为迭代器会保存自己的状态 并不会因为二次调用就更新
# 感觉这一堆测试用例就是为了说明这一点包的饺子
>>> repeated(s, 3)
5
```

很奇怪，为什么迭代器本身不会因为调用函数更新自己的状态呢

是否可以认为迭代器是一个指针，指向了自己的函数帧？

```
# GPT
是的，迭代器内部会有一个指针，用来指向当前迭代的位置。这个指针会随着迭代器的迭代过程不断更新，以便迭代器能够正确地返回下一个元素。当迭代器被创建时，这个指针通常会指向序列的第一个元素，随着迭代器的不断迭代，指针会移动到下一个元素的位置。这个指针是迭代器内部状态的一部分，因此每个迭代器对象都有自己的指针，它们的状态是相互独立的。
```

迭代器是通过 `__iter__()` 声明的，所以是在全局环境中保存了自己的状态

但是迭代器又是从可迭代对象中调用 `__iter__()` 生成的，在调用的时候又重新传入了可迭代对象，但是没有对迭代器的状态造成影响。

```python
def repeated(k):
    # 如果把 s 定义在内部 每次函数调用都会创建一个迭代器 每个都会保存自己的状态
    count = 1
    index_slow = None
    s = iter([3, 2, 2, 2, 1, 2, 1, 4, 4, 5, 5, 5])
    while True:
        index_fast = next(s)
        if index_fast == index_slow:
            count += 1
        if index_slow != index_fast:
            index_slow = index_fast
            count = 1   # 记得更新
        if count == k:
            return index_slow
        
print(repeated(3))
print(repeated(3))
```



独立的迭代器在多线程环境下的访问

```
# GPT
在 Python 中，迭代器本身不是线程安全的，因此不能将其视为锁。如果多个线程同时访问同一个迭代器，并且在迭代过程中修改迭代器的状态，可能会导致意外的行为或错误。

如果需要在多线程环境下安全地使用迭代器，可以考虑使用线程安全的数据结构或者在迭代器的访问上添加适当的同步机制，比如使用互斥锁（mutex）。通过在迭代器的访问过程中使用锁来确保在同一时间只有一个线程可以访问迭代器，可以有效地避免竞态条件和数据不一致性问题。
```



### Q4: Merge

```shell
python ok -q merge --local
```

有关生成器特性的，输入两个有序生成器，可以认为不包含重复元素但是可以是流的形式，无穷的状态

返回的是不包含重复元素的，两个序列融合后的有序生成器。

给出了一个自定义的 `next()` 可以有返回值，因为 Python 的 `next()` 没有任何返回值，该函数返回一个 `generator` 。

```python
# 这个测试用例也很有意思
def big(n):
    k = 0
    while True:
        yield k;
        k += n

# 通过每次调用 big() 逐次推进排序进度 每次接受的参数数量任意
m = merge(big(2), big(3))
```

至于怎么做... 呃 :）仅从算法上看其实很简单，但是生成器怎么做，呃

思路上：从两个迭代器中各取一个元素，比较元素的大小，把小的元素放进去然后大的保留，较小的 `next()` 

写了一般，但是在处理返回结果的时候不对，不能每次声明一个迭代器返回，因为这样无法保存当前的执行位置需要通过 `yield` 返回

```python
def merge(incr_a, incr_b):
    iter_a, iter_b = iter(incr_a), iter(incr_b)
    next_a, next_b = next(iter_a, None), next(iter_b, None)
    "*** YOUR CODE HERE ***"
    # 不能用列表来储存结果
    v = []
    while iter_b is not None and iter_a is not None:
        if iter_a <= iter_b:
            v.append(iter_a)
            iter_a = next(iter_a)
        else:
            v.append(iter_b)
            iter_b = next(iter_b)
    while iter_a is not None:
        v.append(iter_a)
        iter_a = next(iter_a)
    while iter_b is not None:
        v.append(iter_b)
        iter_a = next(iter_b)
    return iter(v)
```

本质上还是没有弄懂 `next_a()` 和 `next_b()` 函数的意义

```shell
# 普通的 next()
>>> a = [1,2]
>>> b = iter(a)
>>> next(b)
1

# 迭代器对象并不会随着 a 的访问结束而消失
# 改变发生在 iter().special_variable() 指针的指向保持不变
a = [11, 2]
b = iter(a)	# e.g. <list_iterator object at 0x0000019FF2466B20>
print(next(b,None))	# 11
print(next(b,None))	# 2
print(next(b,None))	# None
print(next(b,None))	# None
```

```python
# 正确答案
def merge(incr_a, incr_b):
    # 通过可迭代对象生成迭代器 因为是在函数内部生成 所以每次调用都会更新
    # 调用两次会返回两次相同的结果
    iter_a, iter_b = iter(incr_a), iter(incr_b)
    # 通过传递第二个参数 None 取消触发 StopIteration 异常
    # 同时在这里获取了两个列表的第一个元素 进行初始化 
    next_a, next_b = next(iter_a, None), next(iter_b, None)
    "*** YOUR CODE HERE ***"
    # 通过 yield 返回 generator 类型 也能满足流式文件的读取
    # ! 注意这里是根据列表的元素值进行判断 迭代器始终不为空
    while next_a is not None and next_b is not None:
        val_a, val_b = next_a, next_b   # 直接调用已经给出的 function handle
        if val_a < val_b:
            yield val_a
            next_a = next(iter_a, None)
        elif val_a == val_b:
            yield val_a
            next_a = next(iter_a, None)
            next_b = next(iter_b, None)
        else:
            yield val_b
            next_b = next(iter_b, None)
    while next_a is not None:
        yield val_a
        next_a = next(iter_a, None)
    while next_b is not None:
        yield val_b
        next_b = next(iter_b, None)
```

```shell
>>> print(type(m))  # 为什么这里没有进入函数而是直接出结果了呢
>>> print(list(m))  # 反而是这里进入了 merge() 函数

# GPT
在Python中，生成器对象是惰性求值的。这意味着当你调用生成器函数时，它不会立即执行函数体内的代码，而是返回一个生成器对象，该对象保存了函数的状态以及如何执行函数的信息。因此，当你调用 `merge(lst1, lst2)` 返回生成器对象 `m` 时，实际上并没有立即执行函数体内的代码。

生成器对象只有在你迭代它时（比如使用 `list()` 函数转换为列表时），才会逐步执行函数体内的代码，直到遇到 `yield` 语句或者函数结束。因此，虽然在调用 `print(type(m))` 时没有显式调用函数，但由于生成器对象已经被创建，所以仍然可以获取到它的类型信息，即 `<class 'generator'>`。

>>> # m 是当时函数 merge() 的一个结果，并没有实际求值，只是该结果可以被 type() 识别

是的，您说得对。在调用生成器函数时，它并不会立即执行函数体内的代码，而是返回一个生成器对象，该对象包含了生成器函数的定义以及生成器当前的状态信息。这个过程类似于一个“准备就绪”的状态，生成器对象中包含了可以执行生成器函数的一切所需。

因此，当您调用 `type(m)` 时，Python 解释器能够识别 `m` 是一个生成器对象，并返回其类型为 `generator`。

只有在对生成器对象进行迭代时（比如使用 `list()` 函数），才会开始执行生成器函数内的代码，逐步产生生成器的值。这就是惰性求值的体现。
```

返回的是程序运行的一个切片吧，可以这么认为



# Extra Questions

### Q5: Joint Account

```shell
python ok -q make_joint --local
```

创建共享账户，可以用不同的密码访问，同样在三次访问失败后锁死，利用 `Q2: make_withdraw` 的解决

新密码是新函数 `make_joint` 独享的，只能通过 `make_joint` 访问，旧密码两个函数都可以访问，返回的肯定是 `withdraw()` 函数

```python
# my code
def make_withdraw(balance, password):
    error_count = 0
    error_list = []
    def try_withdraw(amount, type_password):
        nonlocal balance
        nonlocal error_count
        nonlocal error_list
		
        # 这里因为有的时候 password 是 list 所以修改了
        while error_count <= 2:
            if type(password) == list:
                for pass_ in password:
                    if type_password == pass_:
                        if balance >= amount:
                            balance -= amount
                            return balance
                        else:
                            return 'Insufficient funds'
                error_count += 1
                error_list.append(type_password)
                return 'Incorrect password'
            else:
                if type_password != password:
                    error_count += 1
                    error_list.append(type_password)
                    return 'Incorrect password'
                else:
                    if balance >= amount:
                        balance -= amount
                        return balance
                    else:
                        return 'Insufficient funds'
        return 'Too many incorrect attempts. Attempts: ' + str(error_list)
    return try_withdraw


def make_joint(withdraw, old_pass, new_pass):
    # 只负责添加一个新密码 并且共享 balance
    new_password = []
    new_password.append(old_pass)
    # 通过取 0 元判断旧密码的正确性 不存在余额不足的情况
    is_correct_account = withdraw(0, old_pass)
    if type(is_correct_account) == str:
        # 旧密码错误
        return is_correct_account
    else:
        # 旧密码正确
        new_password.append(new_pass)
        # 怎么把判断密码更新为新密码
        joint_account = make_withdraw(is_correct_account, new_password)
        return joint_account

w = make_withdraw(100, 'hax0r')
print("75", w(25, 'hax0r'))  # 75 调用 try_withdraw 环境包含了 true_password
make_joint(w, 'my', 'secret')   # Incorrect password
j = make_joint(w, 'hax0r', 'secret')
print("Incorrect password", w(25, 'secret'))  # Incorrect password
print("50", j(25, 'secret'))  # 50
print("25", j(25, 'hax0r'))   # 25

j2 = make_joint(j, 'secret', 'code')
print("20", j2(5, 'code'))
print("15", j2(5, 'secret'))
print("10", j2(5, 'hax0r'))
# 在这里出问题 无法匹配到最原始的密码 因为在更新的时候是根据 old_pass 更新的而不是通过对象

# 想改对这一点肯定要在里面写一个子函数 然后把 new_password 声明为 Nonlocal
```

```python
# 又改了一版 虽然还是没改对 而且考虑到修改了 make_withdraw 肯定不对就是了
def make_joint(withdraw, old_pass, new_pass):
    def sub():
        nonlocal old_pass
        nonlocal new_password
        new_password.append(new_pass)
        joint_account = make_withdraw(is_correct_account, new_password)
        return joint_account
    # 只负责添加一个新密码 并且共享 balance
    new_password = []
    new_password.append(old_pass)
    is_correct_account = withdraw(0, old_pass)
    if type(is_correct_account) == str:
        return is_correct_account
    else:
        return sub()

# 还是在同样的地方出问题 并不能保留 old_pass
```

准备去看答案了，把答案放在可视化上看了一下

怀疑自己和别人有智力上的差距.... 其实不是 nonlocal 不懂，还是子函数的环境和父函数的问题

答案的思路是通过父函数的调用，把当前的 `new_pass` 往前走一步，换成上个环境的 `new_pass` 也就是当前环境的 `old_pass` 这里一路换上去会替换为最原始密码，去调用 `try_withdraw` 函数

[Python Tutor code visualizer: Visualize code in Python, JavaScript, C, C++, and Java](https://pythontutor.com/render.html#mode=display) 

```python
def make_withdraw(balance, password):
    error_count = 0
    error_list = []
    def try_withdraw(amount, type_password):
        nonlocal balance
        nonlocal error_count
        nonlocal error_list

        while error_count <= 2:
            if type_password != password:
                error_count += 1
                error_list.append(type_password)
                return 'Incorrect password'
            else:
                if balance >= amount:
                    balance -= amount
                    return balance
                else:
                    return 'Insufficient funds'
        return 'Too many incorrect attempts. Attempts: ' + str(error_list)
    return try_withdraw


def make_joint(withdraw, old_pass, new_pass):
    def joint_withdraw(amount, password):
        nonlocal old_pass, new_pass
        # 关键在这里
        # TMD 我其实最开始就是想要不要替换成 old_pass 去调用
        # 但是仍然需要对 new_pass 进行判断 所以后面就放弃了这个想法
        # 所以其实还是没想到要利用父函数的环境去替换密码
        return withdraw(amount, old_pass if password == new_pass else password)
    check = withdraw(0, old_pass)
    if type(check) == str:
        return check
    return joint_withdraw

w = make_withdraw(100, 'hax0r')
print("75", w(25, 'hax0r'))
make_joint(w, 'my', 'secret')
j = make_joint(w, 'hax0r', 'secret')
print("Incorrect password", w(25, 'secret'))
print("50", j(25, 'secret'))
print("25", j(25, 'hax0r'))
print("---")
j2 = make_joint(j, 'secret', 'code')
print("20", j2(5, 'code'))
print("15", j2(5, 'secret'))
```

不管怎么样还是坚持吧...



### Q6: Remainder Generator

```shell
python ok -q remainders_generator --local
```

生成器可以像函数一样有 `higher-order` 的规则，接受一个整数 `m` ，并返回 `m` 个不同的生成器

根据被 4 相除的余数进行返回，当然题目只是以 4 举例，内部的子函数可以直接调用 `naturals` 函数

```python
def remainders_generator(m):
    def intern_generator():
        n = 1
        nonlocal i
        while True:
            if m % n == i:
                yield n
            n += 1
    for i in range(0, m):
        # 枚举 m 的余数 [0, m - 1]
        # Q: 因为 HOF 不能调用 只能返回函数对象 但是怎么才能把 i 传到子函数里
        yield intern_generator
        
# 这里 remainders_generator() 返回的是 function 而不是 generator 
# 这里还没有看答案 我也不知道怎么改了 我觉得问题在主函数 子函数应该问题不大?
```

唉，看看

```python
def naturals():
    i = 1
    while True:
        yield i
        i += 1
        
def remainders_generator(m):
    def remainder(n):
        while True:
            for natural in naturals():
                yield natural*m + n

    yield remainder(0)
    for i in range(-m+1,0):
        yield remainder(i)
    

import types
for gen in remainders_generator(5):
    # yield 返回的值被视为 generator
    print(isinstance(gen, types.GeneratorType))

remainders_four = remainders_generator(4)   # 惰性求值

for i in range(4):
    print("First 3 natural numbers with remainder {0} when divided by 4:".format(i))
    gen = next(remainders_four)     # remainder(i)
    for _ in range(3):
        print(next(gen))
```

有点知道问题在哪，这里的 `yield` 还是使用参数进行了传递，为什么

这里和生成器惰性求值有关吗	无关，单纯就是 `yield` 语法问题，如果希望返回值类型为 `generator` 那么`yield` 必须是函数调用结果

主要是我有一个观念不对，`next()` 并不是严格意义上的函数调用，因为并没有参数 `i` 的问题，它只是让函数继续运行了。



# Done..?

这节的 homework 说真的有点.. :(
