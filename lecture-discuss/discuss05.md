# Lists, Trees, Mutability

## Trees

### 环境

```python
def tree(label, branches=[]):
    if change_abstraction.changed:
        for branch in branches:
            assert is_tree(branch), 'branches must be trees'
        return {'label': label, 'branches': list(branches)}
    else:
        for branch in branches:
            assert is_tree(branch), 'branches must be trees'
        return [label] + list(branches)

def label(tree):
    if change_abstraction.changed:
        return tree['label']
    else:
        return tree[0]

def branches(tree):
    if change_abstraction.changed:
        return tree['branches']
    else:
        return tree[1:]

def is_tree(tree):
    if change_abstraction.changed:
        if type(tree) != dict or len(tree) != 2:
            return False
        for branch in branches(tree):
            if not is_tree(branch):
                return False
        return True
    else:
        if type(tree) != list or len(tree) < 1:
            return False
        for branch in branches(tree):
            if not is_tree(branch):
                return False
        return True

def is_leaf(tree):
    return not branches(tree)

def change_abstraction(change):
    change_abstraction.changed = change

change_abstraction.changed = False

def print_tree(t, indent=0):
    print('  ' * indent + str(label(t)))
    for b in branches(t):
        print_tree(b, indent + 1)

def copy_tree(t):
    return tree(label(t), [copy_tree(b) for b in branches(t)])

def is_branch(brunch):
    for item in brunch:
        if is_tree(item):
            return True
        else:
            continue
```

### 1.1

可能是因为太久没碰递归，有了一些和 `nonlocal` 混淆的搞笑错误思路

```python
# def height(t):
#     """
#     Return the height of a tree: from root to leaf
#     >>> t = tree(3, [tree(5, [tree(1)]), tree(2)])
#     """
#     brunch_heigh = []
#     def sub_tree(brunch=None):
#         nonlocal brunch_heigh
#         current_heigh = 0
#         def sub_heigh(brunch=None):
#             nonlocal  current_heigh
#             if is_branch(brunch) and brunch is not None:
#                 current_heigh += 1
#                 for sub_brunch in brunch:
#                     sub_heigh(branches(sub_brunch))
#                     brunch_heigh.append(current_heigh)
#             else:
#                 return 0
#             return max(brunch_heigh)
              # 在 [3,[4,5]] 递归完成后会突然退出到这里然后再去执行 [6, [7]] 很奇怪，我不理解
              # 因为 [4, 5] 所在的层遍历结束返回
#         return sub_heigh(branches(t))
#     return sub_tree(branches(t))
```

下面是答案：

```python
def height(t):
    if is_leaf(t):
        return 0
    else:
        # 我会想的那么复杂还是没有理解 lst 在递归中的作用 在每一层递归中 lst 就已经是仅在当下作用域生效的 也可以添加回去
        # 所以不需要 nonlocal 
        lst = []
        for sub in branches(t):
            current_height = 1 + height(sub)
            lst.append(current_height)
        return max(lst)
        # lst = [1 + height(sub) for sub in branches(t)]
        # return max(lst)
```



### 1.2

还行，这个倒是写出来了

```python
# 无法在原树的基础上修改 只能建立一棵新数
def square_tree(t):
    if is_leaf(t):
        new_value = label(t) ** 2
        return tree(new_value, [])
    else:
        branch = []
        for sub in branches(t):
            branch += tree(square_tree(sub))
        return tree(label(t) ** 2, branch)
```



### 1.3

主要是对递归中返回值的效果不清晰，废了点劲

```python
def find_path_A(tree, x):
    # 如何确定已经到了终点但是还没有找到的情况
    if label(tree) == x:
        return x
    
    if is_leaf(tree):
        # 该分支到叶节点也没有找到
        return False
    
    path = []
    path.append(label(tree))
    for sub in branches(tree):
        path.append(label(sub))
        if find_path_A(sub, x) == False:
            path.pop()
        else:
            return path + [find_path_A(sub, x)]
    return None

def find_path_B(tree, x):
    # 对比一下正确答案：递归的思路问题不大
    if label(tree) == x:
        return [x]
    if is_leaf(tree):
        # Q: 为什么这里不返回会造成 path 元素缺失呢 叶子节点本身应该不会造成影响才对
        # A: 因为结果是统一作用在 find_path_B() 上的 值本身确实没有意义 但是需要 type 保持正确
        # return [label(tree)]
        return []
    for sub in branches(tree):
        # 调用 find_path_B 的时候不需要中括号了 函数的返回值已经包含
        path = [label(tree)] + find_path_B(sub, x)
        # 如果发生了 return 的情况
        if path[-1] == x:
            return path
        
def find_path(t, x):
    if label(t) == x or is_leaf(t):
        return [label(t)]
    for _ in branches(t):
        path = [label(t)] + find_path(_, x)
        if path[-1] == x:
            return path
    return
```



## Mutation

### 2.1

没想到这里要注意的点很多 基础是很重要的啊啊啊啊啊

（当然语言特性相对没有那么重要就是了 :）

```

```



### 2.2

这个简单

```python
def add_this_many(x, el, lst):
    add_time = 0
    for elem in lst:
        if x == elem:
            add_time += 1
    for i in range(0, add_time):
        lst.append(el)
    return lst
```



### 2.3

这个也

```python
def group_by(s, fn):
    group_dict = {}
    for elem in s:
        key = fn(elem)
        if key in group_dict:
            group_dict[key].append(elem)
        else:
            group_dict[key] = []
            group_dict[key].append(elem)
    return group_dict
```



## Quiz

### A

感受到了递归难以名状的过程

```python
# 和这个很像的之前的一道题 卡了很久 结果现在看见怂了 没怎么思考直接看了答案 现在又后悔 唉
# 其实当时主要是卡在最多走 k 级台阶上 无法列举完全  一开始也想过分成 (max, rest) 去处理 但还是没想出来
# 现在看着答案是很清晰的 通过每轮的递归逐次减少 k 达到枚举的效果
def partition_options(total, biggest):
    # 一共有 m 级台阶 每次最多走 k 阶
    # 假设是倒放视频 这个人已经走上去了 在他走最后一步的台阶之前的路都已经确定了
    if total <= 0 or biggest == 0:
        return [[]]
    if biggest == 1:
        return [[1 for _ in range(0,total)]]
    if total == 1:
        return [[1]]
    path1 = [i + [biggest] for i in partition_options(total - biggest, biggest)]    # 这里的递归就是在求他之前走的路 当下这步是确定的 biggest
    path2 = partition_options(total, biggest - 1)
    return path1 + path2
```



### B

这个是相对简单的，也写出来了

```python
def min_elements(T, lst):
    if T <= 0:
        return 0
    if max(lst) <= T :
        return 1 + min_elements(T - max(lst), lst)
    return min_elements(T, sorted(lst)[:-1])

# 相比上面那个没有考虑取最小值的情况
def min_elements_A(T, lst):
    if T <= 0:
        return 0
    return min(1 + min_elements(T - max(lst), lst), min_elements(T, sorted(lst)[:-1]))
```



# Done!

最大的收获果然还得是递归，很庆幸自己一道题都没放弃，最起码也是过了一遍
