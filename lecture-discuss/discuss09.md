# Linked Lists, Trees

## Linked Lists

```python
class Link:
    empty = ()

    def __init__(self, first, rest=empty):
        assert rest is Link.empty or isinstance(rest, Link)
        self.first = first
        self.rest = rest

    def __repr__(self):
        if self.rest is not Link.empty:
            rest_repr = ', ' + repr(self.rest)
        else:
            rest_repr = ''
        return 'Link(' + repr(self.first) + rest_repr + ')'

    def __str__(self):
        string = '<'
        while self.rest is not Link.empty:
            string += str(self.first) + ' '
            self = self.rest
        return string + str(self.first) + '>'
```



### 1.1

```python
def sum_nums(lnk):
    sum = 0
    while lnk is not Link.empty:
        sum += lnk.first
        lnk = lnk.rest
    return sum
```



### 1.2

传入一组列表，根据长度最短的列表，返回各个位置列表相加的结果

我做出来了啊啊啊啊啊啊啊啊啊啊啊啊啊，不然我感觉我就要碎掉了

```python
def multiply_lnks(lst_of_lnks):
    mul_res = 1
    for lnk in range(len(lst_of_lnks)):
        if lst_of_lnks[lnk] is Link.empty:
            return Link.empty
        else:
            mul_res *= lst_of_lnks[lnk].first
            lst_of_lnks[lnk] = lst_of_lnks[lnk].rest
    a = Link(mul_res)
    a.rest = multiply_lnks(lst_of_lnks)
    return a

a = Link(2, Link(3, Link(5)))
b = Link(6, Link(4, Link(2)))
c = Link(4, Link(1, Link(0, Link(2))))
p = multiply_lnks([a, b, c])
print(p)
```



### 1.3

返回一个生成器，返回函数执行结果为 True 的值

```python
link = Link(1, Link(2, Link(3)))
g = filter_link(link, lambda x: x % 2 == 0)
print(next(g))
print(next(g))
print(list(filter_link(link, lambda x: x % 2 != 0)))
```

用循环和非循环各自实现一次

```python
# while-loop
def filter_link(link, f):
    while link is not Link.empty:
        if f(link.first) is True:
            yield link.first
        link = link.rest
```

```python
# no-iter
def filter_link(link, f):
    if link is Link.empty:
        return
    elif f(link.first) is True:
        yield link.first
    yield from filter_link(link.rest, f)
```



## Trees

```python
class Tree:
    def __init__(self, label, branches=[]):
        for b in branches:
            assert isinstance(b, Tree)
        self.label = label
        self.branches = list(branches)

    def is_leaf(self):
        return not self.branches

    def map(self, fn):
        self.label = fn(self.label)
        for b in self.branches:
            b.map(fn)

    def __contains__(self, e):
        if self.label == e:
            return True
        for b in self.branches:
            if e in b:
                return True
        return False

    def __repr__(self):
        if self.branches:
            branch_str = ', ' + repr(self.branches)
        else:
            branch_str = ''
        return 'Tree({0}{1})'.format(self.label, branch_str)

    def __str__(self):
        def print_tree(t, indent=0):
            tree_str = '  ' * indent + str(t.label) + "\n"
            for b in t.branches:
                tree_str += print_tree(b, indent + 1)
            return tree_str
        return print_tree(self).rstrip()

```



### 2.1

所有奇数值的结点加一变成偶数值结点，本身为偶数值结点的保持不变

```python
def make_even(t):
    if t.is_leaf() and (t.label % 2 == 1):
        t.label += 1
        return
    elif t.is_leaf() and (t.label % 2 == 0):
        return
    else:
        if t.label % 2 == 1:
            t.label += 1
        for b in t.branches:
            make_even(b)

t = Tree(1, [Tree(2, [Tree(3)]), Tree(4), Tree(5)])
make_even(t)
```



### 2.2

将这棵树中的结点值全部平方

```python
def square_tree(t):
    if t.is_leaf():
        t.label = t.label ** 2
        return
    else:
        t.label = t.label ** 2
        for b in t.branches:
            square_tree(b)

tree_ex = Tree(2, [Tree(7, [Tree(3), Tree(6, [Tree(5), Tree(11)])]), Tree(1)])
square_tree(tree_ex)
print(tree_ex)
```



### 2.3

假设这棵树的每个节点值唯一，并且有唯一一条走到该结点的路，以列表的形式打印

我之前好像做过一道一样的，那个用的好像还是生成器什么的

没想到这个还是没搞出来，去看看

```python
def find_path(tree_ex, entry):
    if tree_ex.label == entry:
        return [tree_ex.label]
    for b in tree_ex.branches:
        # 实际上是靠递归一层层对失败的路径返回 None 进行判断的
        # 注意两个分支都没有写 else 只考虑成功路径 自然就不用考虑失败节点的弹出
        path = find_path(b, entry)
        if path:
            return [tree_ex.label] + path
        
tree_ex = Tree(2, [Tree(7, [Tree(3), Tree(6, [Tree(5), Tree(11)])]), Tree(1)])
print(find_path(tree_ex, 5))
```

这个答案是在 sol_05.pdf 中，原本用纯数据结构实现，稍微改一下就是对象实现



### 2.4

求总数和平均值

```python
def average(t):

    def sum_helper(t):
        total, count = 0, 0
        if t.branches == []:
            return t.label, 1
        for b in t.branches:
            a, b = sum_helper(b)    # 上一个分支的计算结果在这里被更新了 要想办法完成 +=
            total += a
            count += b
        return total + t.label, count + 1
    
    total, count = sum_helper(t)
    print(total, count)
    return total / count

t0 = Tree(0, [Tree(1), Tree(2, [Tree(3)])])
print(average(t0))

t1 = Tree(8, [t0, Tree(4)])
print(average(t1))
```

花了一些时间，但是还能做出来



### 2.5

把两个结构完全相同的树的各个结点进行 combiner 操作，并将返回值赋予一棵完全相同的新树

```python
def combine_tree(t1, t2, combiner):
    if t1.is_leaf():
        return Tree(combiner(t1.label, t2.label))
    else:
        a = Tree(combiner(t1.label, t2.label))
        for para in zip(t1.branches, t2.branches):
            a.branches = [combine_tree(para[0], para[1], combiner)]
        return a    # Tree(4, Tree(10, Tree(18)))

a = Tree(1, [Tree(2, [Tree(3)])])
b = Tree(4, [Tree(5, [Tree(6)])])
from operator import mul
combined = combine_tree(a, b, mul)
# 能得到正确的树 但是在打印上出了点问题     没有在 branches 的返回上加 [] 导致的
print(combined)
```



### 2.6

每间隔一层，对该层的所有结点值执行函数

```python
def alt_tree_map(t, map_fn):
    depth = -1
    def helper(sub_t, sub_map_fn):
        nonlocal depth
        depth += 1
        if depth % 2 == 0:
            sub_t.label = map_fn(sub_t.label)
        if sub_t.is_leaf():
            return
        else:
            for b in sub_t.branches:
                helper(b, sub_map_fn)
    return helper(t, map_fn)

t = Tree(1, [Tree(2, [Tree(3)]), Tree(4)])
negate = lambda x: -x
alt_tree_map(t, negate)
print(t)
```

还行，直面曾经的困难居然解决了



# Done!

还行，不错，基本都做出来了，比我想象的好很多（也可能是熟练了hhhh
