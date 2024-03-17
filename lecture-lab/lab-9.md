# Lab 9 Linked Lists, Mutable Trees

#### Linked Lists

a recursive object that only stores two things: first value and a reference to the rest of the list



#### Mutable Trees



## Linked Lists

### Q1: WWPD: Linked Lists

```shell
python ok -q link -u --local
```

```
<function ...> => Function
Wrong => Error
None => Nothing
```

```python
>>> link = Link(1000, 2000)
# Correct: Error => rest is not Link type

>>> link = Link(1000, Link())
# Correct: Error => rest is empty
```

```python
>>> link = Link(1)		# List_obj
>>> link.rest = link	# 通过赋值 object 完成了递归调用
>>> link.rest.rest.rest.rest.first

# (link.rest).rest.rest.rest.first
# link.rest.rest.rest.first	=> (link.rest).rest.rest.first
# link.rest.rest.first => (link.rest).rest.first
# link.rest.first => (link.rest).first
# link.first => 1

# 无论执行多少层 都是在调用自己
```



### Q2: Convert Link

```shell
python ok -q convert_link --local
```

面向测试用例编程了，有一说一，这样不好

```python
# 循环版本
def convert_link(link):
    result_list = []
    if link == Link.empty:
        return result_list
    while link.rest is not Link.empty:
        result_list.append(link.first)
        link = link.rest
    result_list.append(link.first)
    return result_list
```

```python
def convert_link(link):
    if link == Link.empty:
        return []
    if link.rest == Link.empty:
        return [link.first]
    return [link.first] + convert_link(link.rest)
```



### Q3: Every Other

```shell
python ok -q every_other --local
```

修改原有列表，删掉奇数的元素，但是如果只有两个元素的话，保持不变

```python
# 递归实现
def every_other(s):
    if s.rest is not Link.empty and s.rest.rest is not Link.empty:
        s.rest = every_other(s.rest.rest)
    else:
        s.rest = Link.empty
        return s
    
singleton = Link(4)
every_other(singleton)	# Error: 由于递归的 return 有 Link(4) 打印
singleton
Link(4)

# 如果想解决这个问题 感觉只能用循环 使用递归就避免不了 return
# 但是循环的话 想不到 s 更新后的重新指向的问题
```

看了看答案，确实还是用递归实现的，只是巧妙的避免了 `return s` 语句

```python
def every_other(s):
    # 还是很巧妙的
    if s == Link.empty or s.rest == Link.empty:
        return
    s.rest = s.rest.rest
    return every_other(s.rest)	# 1 2 3 4 => 1 3
```

和我的思路还不太一样	MD 仔细想想感觉这种思路很正常，我原本的思路才是不正常的，为什么 TMD

确实是每两个元素，两个元素的去递归，我是被第一个元素的特殊性束缚了思考算是，并没有去考虑全空情况

（因为我从初始的思考上就向后错了一位，所以需要赋值后再返回



## Mutable Trees

### Q4: Square

```shell
python ok -q label_squarer --local
```

在原树的数据上修改，递归实现



### Q5: Cumulative Mul

```shell
python ok -q cumulative_mul --local
```

父节点的值成为当前自己的值和下一层子节点的乘积，过了测试但是感觉自己写的不太对

```python
def cumulative_mul(t):
    def wait_for_branch(t):
        if t.is_leaf():
            return t.label
        else:
            for branch in t.branches:
                t.label *= wait_for_branch(branch)
            return t.label
    for branch in t.branches:
        t.label *= wait_for_branch(branch)
```



# Optional Questions

### Q6: Cycles

```shell
python ok -q has_cycle --local
```

传递列表对象构成递归，形成了一个环，这个函数的任务是检测目标是否有环，有个比较偷鸡的办法检测 `type` 类型，坏了，不行，都一样

很笨逼的方法，就是用一个列表记录目前已存在的 Link 对象地址，判断是否有重合，嗯，能做

```python
def has_cycle(link):
    address_table = [id(link)]
    while link.rest is not Link.empty:
        if id(link.rest) in address_table:
            return True
        else:
            address_table.append(id(link.rest))
            link = link.rest
    return False
```

#### Extra Challenge

```shell
python ok -q has_cycle_constant --local
```

constant space：要求 `O(1)` 的常数级空间，emm... 一下子想不太到呢

快慢指针：快指针总是能提前，如果有环的话，总能追上慢指针，但是这也不是常数级的 × 确实是快慢指针，是对空间有要求而不是时间，只需要两个指针空间

```python
def has_cycle_constant(link):
    slow_index = link
    fast_index = link
    while link.rest is not Link.empty and link.rest.rest is not Link.empty:
        slow_index = slow_index.rest
        fast_index = fast_index.rest.rest
        if slow_index == fast_index:
            return True
        else:
            link = link.rest
    return False
```



### Q7: Reverse Other

```shell
python ok -q reverse_other --local
```

奇数层深度的结点值反转....（我无力吐槽了已经，我例题都 TM 没看懂	在第二个例子中为什么同层的 (2,8) 也交换了

```py
def reverse_other(t):
    if t.is_leaf():
        return 
    label_list = []
    for b in t.branches:
        # 先把该层的所有结点存起来		每次只处理相对于 t 的下一层的结点 实现奇数层的效果
        label_list.append(b.label)
    for b, new_label in zip(t.branches, reversed(label_list)):
        # 2，8 应该就是在 reversed() 这里交换的 虽然我不知道为什么要交换它
        b.label = new_label
        for bb in b.branches:
            reverse_other(bb)
```

我连答案也没看明白... 😭



# Done!

就算完成了吧，除了最后一题都还好
