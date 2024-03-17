# Homework 04

## Mobiles

给出了一个天文摆的模型 我觉得这玩意和树没什么区别就是了

必须有左子树和右子树 叶子结点必须是 `planet` 

区别是一个合法的 mobile 必须有左右子树 根节点不可能是叶子节点



#### Arms-length recursion

```python
# 和 mobile 模型无关 就是单纯的树 
def min_depth(t):
    if is_leaf(t):
        return 0
    h = float('inf')	# 规定一个上限 等待后面刷新 
    for branch in braches(t):
        # if is_leaf(branch):	违背了递归准则
        #     return 1
    	h = min(h, 1 + min_depth(branch))
    return h
```

这段就是想强调 虽然你加的这两行 if 判断可以减少一层递归提高代码的执行效率

但是违背了递归原则 是不推荐的



### Q1: Weights

```shell
python ok -q total_weight --local
```

根据给出的函数构造 `planet` 抽象数据类型

`planet` 构造函数 `size()` 成员函数



### Q2: Balanced

```shell
python ok -q balanced --local
```

判断该天文摆是否平衡：

1. 左右臂的力矩相等 长度 × 重量
2. 左右臂本身平衡

总之就是递归平衡	没想到自己搞出来了 有点开心



### Q3: Totals

```shell
python ok -q totals_tree --local
```

从它的语法检查来看 必须调用自己 所以是不能调用上面的 `total_weight` 的

利用已经存在的树的抽象数据结构对 `mobile` 进行映射

```python
def totals_tree(m):
    def mobile_map_tree(m):
        # tree 的构建只能发生在 mobile 结点
        if is_planet(m):
            return size(m)
        if is_arm(m):
            return mobile_map_tree(end(m))
        if is_mobile(m):
            branch_left = tree(mobile_map_tree(left(m)))
            branch_right = tree(mobile_map_tree(right(m)))
            return tree(total_weight(m), (branch_left, branch_right))
    return mobile_map_tree(m)
```

现在的问题是格式不太对，呃

你杀了我也不知道为什么外面会套中括号

```python
def totals_tree(m):
    def mobile_map_tree(m):
        if is_planet(m):
            # print("is_planet: " + str(size(m)))
            return tree(size(m))
        if is_arm(m):
            # print("is_arm")
            return tree(mobile_map_tree(end(m)))
        if is_mobile(m):
            # print("mobile")
            branch_left = tree(mobile_map_tree(left(m)))
            # print("---------------------")
            # print_tree(branch_left)
            # print("---------------------")
            branch_right = tree(mobile_map_tree(right(m)))
            return tree(total_weight(m), (branch_left, branch_right))
    return mobile_map_tree(m)
```



```python
def totals_tree(m):
    def mobile_map_tree(m):
        if is_planet(m):
            # print("is_planet: " + str(size(m)))
            return tree(size(m))
        if is_arm(m):
            # print("is_arm")
            return tree(mobile_map_tree(end(m)))
        if is_mobile(m):
            # print("mobile")
            branch_left = tree(total_weight(end(left(m))), tree(totals_tree(end(left(m)))))
            # print("---------------------")
            # print_tree(branch_left)
            # print("---------------------")
            branch_right = tree(total_weight(end(right(m))), tree(totals_tree(end(right(m)))))
            return tree(total_weight(m), (branch_left, branch_right))
    return mobile_map_tree(m)
```

我不知道我去看答案了 :（

```python
# 通过 end() 跳过 arm() 那层递归 确实我也觉得我那层判断写的没有意义
# 精简一下我的答案
def totals_tree(m):
    if is_planet(m):
        return tree(size(m))
    if is_mobile(m):
        branch_left = tree(total_weight(end(left(m))), tree(totals_tree(end(left(m)))))
        branch_right = tree(total_weight(end(right(m))), tree(totals_tree(end(right(m)))))
        # 可以看到问题出在 branch_left 和 branch_right 的 tree 层数上
        return tree(total_weight(m), [branch_left, branch_right])
```

自己写一遍之后就明白了，思维怎么能偏差到这个样子 明明就差一点点然后越走越远

主要是没有意识到 `totals()` 返回的也是 `tree` 结构



## Trees

### Q4: Replace Leaf

```shell
python ok -q replace_leaf --local
```

树中的每个节点都有 label 和之前的寻找叶结点的方式不一样 这里要遍历整棵树

一开始以为是整棵树的结点 label 都要替换 但是后面看到只要替换叶子节点

```python
def replace_leaf(t, find_value, replace_value):

    # 本次使用中序遍历的方式 注意是多叉树而不是二叉树 
    # for branch in branches(t):
    #     if not is_leaf(branch):
    #         return (label(branch), replace_leaf(branch,find_value,replace_value))
    #     else:
    #         if label(branch) == find_value:
    #             return tree(replace_value)
            
    if is_leaf(t) and label(t) == find_value:
        return tree(replace_value)
    return tree(label(t), [replace_leaf(c, find_value, replace_value) for c in branches(t)])
```

TMD 每道题都是这样 总感觉差一点点 但是肯定差的不是一点点 就想到但是写不出来 

观察了很久的答案 主要是我的思路对递归的入口不清晰

答案的递归是发生在 t 的内部然后汇总到 t ，而我的递归思路是对 t 的子树进行循环遍历

我的思路没有收紧的过程 所以虽然知道该做什么 但是没有结果（大概是这样的感觉？

![](D:\cs-61A\homework pic\hw04\0.png)



### Q5: Preorder

```shell
python ok -q preorder --local
```

多叉树前序遍历：根节点-左子树-右子树

核心是：除去第一次访问根节点后访问的根节点不能再打印

也有格式的问题，我真是吐了

```python
def preorder(t):
    # 访问根节点 
    if is_leaf(t):
        return label(t)
    # 循环访问子结点
    # 问题出在这里 每循环访问同层的下一个结点 就要递归调用 就会多一层中括号
    return [label(t),[preorder(branch) for branch in branches(t)]]
# [1, [2, [3, [4, 5]], [6, [7]]]]
```

我要被气坏了

```python
def preorder(t):
    if is_leaf(t):
        return [label(t)]
    res = []
    for b in branches(t):
        res += preorder(b)  # 在这里把该根的子结点变成同一层 差在这里 
    return [label(t)] + res
```



### Q6: Has Path

```python
python ok -q has_path --local
```

字典树 又称前缀树 `Trie tree` 多叉树（比如自动补全就用了这种功能）

递归调用的层数和 `phrase` 保持一致（ `phrase[1:]` ）

我终于成功了一个呜呜呜呜呜呜，我太感动了 😭😭😭



## Extra

### Q7: Interval Abstraction

```shell
python ok -q interval -u --local
```

在 power shell 的问答 我都忘了还有这个东西...

```shell
python ok -q interval --local
```

学到了一个 `str()` 表达 [Python format 格式化函数 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python/att-string-format.html) 

```python
# 有点像 C 的表达 会设定好占位符
# 属于 Python 的字符串格式化
return '{0} to {1}'.format(lower_bound(x), upper_bound(x))
```



```shell
python ok -q mul_interval -u --local
```

```shell
python ok -q mul_interval --local
```



### Q8: Sub Interval

```shell
python ok -q sub_interval -u --local
```

```shell
python ok -q sub_interval --local
```



### Q9: Div Interval

```shell
python ok -q div_interval -u --local
```

```python
# it is not clear what it means to divide by an interval that spans zero
# 对题目的理解有问题 这里不单指除 0 问题 而是跨度包含 0 的问题
# 只要区间包含 0 那么整个区间的除法都是不成立的
str_interval(div_interval(interval(-1, 2), interval(4, 8)))
# '-0.25 to 0.5'
str_interval(div_interval(interval(4, 8), interval(-1, 2)))
# AssertionError
```



```shell
python ok -q div_interval --local
```



### Q10: Multiple References

Eva Lu Ator,另外一个使用者，她发现使用区间计算法则时哪怕是代数上等价的式子，在实际上计算的时候，得到的结果却是不一样的。这时由于对不确定区间多重引用导致的。

多重引用问题（Multiple references problem）：对于代数上等价的公式，在使用区间计算法则的时候，表示不确定区间的变量重复引用得越少，最后得到的结果范围也越小。比如par1中`r1 * r2`，`r1 + r2`，`(r1 * r2) / (r1 + r2)`这三处运算都是对不确定区间的多重引用。

由此，她认为par2是计算并联电阻更好的方式。她说的对吗，请给出说明或解释。



Eva是正确的，问题在于，当我们对同一个区间进行计算的时候，从两个区间中选出的值是不同的，这会导致得到的结果比实际的值要更大。比如当我们计算[-1, 1]计算平方时，得到的结果是[-1, 1]，但实际上在实数集不可能存在一个数的平方小于0。

因此par2是更好的计算方法，在par2中与不确定的区间r1、r2计算的都是固定的区间[1, 1]，所以不会有重复引用问题。



### Q11: Quadratic

```shell
python ok -q quadratic --local
```

我没看懂.. 哈基米哈基米



### Q12: Par Diff

在Alyssa P完成了这份工作之后的很多年，有一个叫Lem E. Tweakit的人注意到并联电路电阻有两种表达方式：

```python
par1(r1, r2) = (r1 * r2) / (r1 + r2)
par2(r1, r2) = 1 / (1/r1 + 1/r2)
```

通过编程都实现了

```python
def par1(r1, r2):
    return div_interval(mul_interval(r1, r2), add_interval(r1, r2))

def par2(r1, r2):
    one = interval(1, 1)
    rep_r1 = div_interval(one, r1)
    rep_r2 = div_interval(one, r2)
    return div_interval(one, add_interval(rep_r1, rep_r2))
```

但是他在实验的时候发现，虽然代数上这两个公式是等价的，但是使用Alyssa P规则对区间进行运算之后得到的结果不同。

现在请举一个反例证明Lem是正确的

```shell
python ok -q check_par --local
```

看不懂思密达 我智力有问题可能性微存

[日拱一卒，伯克利CS61A，作业5，伯克利教你面向对象 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/507354981) 



# Done...

钰钰了 最后这几道题给我整的钰钰了 本来递归想着能解出两道还凑合，思路上有进步 最后直接干碎
