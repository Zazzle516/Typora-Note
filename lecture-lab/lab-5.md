# Lab 5 Python Lists, Trees

## Lists

### Q1: Coordinates

```shell
python ok -q coords --local
```

```python
[<expression> for <element> in <sequence> if <conditional>]
# 外面的中括号是必须的 
# 注意题目要求的返回格式
```



### Q2: Riffle Shuffle

```shell
python ok -q riffle --local
```

```
# 模拟洗牌过程中牌的位置 把中间的牌放在顶部的牌的后面
# 假设有偶数张牌
Hint: use k%2 to alternatively access the beginning and the middle of the list
# 注意 Python 声明数组一定要提前声明长度 否则只能 append() 直接使用会显示数组越界
```



## Trees

有一个自定义抽象数据类型

构造函数

​	`tree(label,branches = [])` 默认 `branches[]` 为空 作为可选选项

选择器

​	`label(tree)` 返回树的根节点的值 

​	`branches(tree)` 根据 `tree` 返回对应的子树列表

成员函数

​	`is_leaf(tree)` 判断是否是空树（因为树的递归性质）更多是在判断是否是叶节点

​	`print_tree(t,indent = 0)` 接收一棵树作为参数 每层默认缩进 2 空格



### Q3: Finding Berries!

```shell
python ok -q berry_finder --local
```

问题是 子树发现了 berry 如何把结果传给上层 因为上层的结果会覆盖，呃...

直接 return True 会被父节点的 False 覆盖

```python
def berry_finder(t):
    if label(t) is 'berry':
        return True
    else:
      for sub_tree in branches(t):
          berry_finder(sub_tree)
    return False
```

明明思路上已经OK了，但是实现不出来.. 还是自己能力不行啊，害	考虑通过子函数实现 但是应该用不到

看了答案，MD，自己脑部供血不足导致的 真是笨比了我

```python
def berry_finder(t):
    if label(t) is 'berry':
        return True
    else:
      for sub_tree in branches(t):
          if berry_finder(sub_tree):
                return True
    return False        
```



### Q4: Sprout leaves

```shell
python ok -q sprout_leaves --local
```

把分支加在叶子结点上构建一颗新树 通过调用 `print_tree()` 打印 这个函数只负责构建新树

递归找到每个叶节点 `is_leaf()` 为空则添加子树



构建一颗新树在传递参数时会很困难，因为每次只能构建一部分，不完整

如果直接处理 `sub_tree_list` 会更方便传参

```python
def sprout_leaves(t, leaves):
    for sub_tree in branches(t):
        if is_leaf(sub_tree) is False:
            # 非空子树
            return sprout_leaves(sub_tree,leaves)
        else:
            # 叶结点 添加子树 
            sub_tree = tree(label(sub_tree), leaves)
    return t
# 这样实现会有 AssertionError: branches must be trees
# emmm... 有点没思路 是因为我太久没碰递归了吗 :( 
```



参考答案

```python
def sprout_leaves(t, leaves):
    if is_leaf(t):
        # 在 branches 中建立循环 就可以规避我上面的问题
        return tree(label(t), [tree(l) for l in leaves])
    else:
        # 这里我当时面临的问题是：即使我成功挂上了 leaves_tree 在递归中怎么把新树传参传出去
        # 当时想构建新树 先把 root_label 提取出来 但是考虑后面递归会覆盖 不知道怎么传参
        # 这里也是在 branches 内部非空子树进行递归来保证最后的新树是完整的 
        return tree(label(t), [sprout_leaves(t, leaves) for t in branches(t)])
```



这两道题都是这样的.. 有实现的思路 但是在递归的具体实现过程中不清晰，导致没能实现

像是攻击我的思维盲区一样

后面在 cs61A 的复盘中 需要再好好复习



### Q5: Don't violate the abstraction barrier!

```shell
python ok -q check_abstraction --local
```



### Q6: Add trees

```shell
python ok -q add_trees --local
```

把两棵树对应位置的 label 相加 合并为一颗新树 以两棵树的并集为准

控制两棵树的递归进度相同 因为树的结构不同 不存在大小之分 可能一个又长又窄 一个又短又宽



控制两棵分离的树的递归进度比较困难 设计一个 `father_tree` 把 t1 t2 作为子树挂进去 ？

呃.... 这道题甚至可以说是一点思路都没有



```
https://www.cnblogs.com/xddxd/p/17250785.html#_label0
```

```python
def add_trees(t1, t2):
    # 非空树 也可能是递归终点的叶结点
    root_label = label(t1) + label(t2)	# !
    
    new_branch_tree = []	# 新建子树列表
    
    for b1,b2 in zip(branches(t1), branches(t2)):
        # 把都存在的 对应的位置的 branch_root 相加
        # new_branch_tree = [add_trees(b1,b2)]
        new_branch_tree += [add_trees(b1,b2)]
        
    used_index = len(new_branch_tree)	# 剩余的是无对应的结点
    new_branch_tree += branches(t1)[used_index:]
    new_branch_tree += branches(t2)[used_index:]
    
    return tree(root_label, new_branch_tree)	# !
# 背着写了一下 可以理解 但是 emm... 还是刚开始想的时候思路不清楚 
```

如果根据这个 `add_tree()` 的函数功能 我那在写 `sprout_leaves()` 的时候转换成 list 实现的思路也许可以

还是还是一样的 因为递归最后都要 `return tree()` 返回这个函数中构建的新树



## Shakespeare and Dictionaries

I don't think this will be fun at all.. :(

我好像看都没看懂 NMD 

这应该是说，构建一个二元词组的列表 比如 `This cat / This dog` `cat / dog` 作为 `This` 的后继词

那么，当用户以 `This` 开头的时候 `cat / dog` 都可以作为输出 并且这个句子是循环的 最后的 `.` 的后继是第一个单词

### Q7: Successor Tables

```shell
python ok -q build_successors_table --local
```

题目要求返回一个字典组 该字典组的一个 key 对应多个 value 

（实习学到的知识用到了，嘿嘿



### Q8: Construct the Sentence

```shell
python ok -q construct_sent --local
```

上面构造的字典中随机选词 构成句子（选到 `.` 结束）

因为是随机性的 所以检测语句只能是一对一的 不然结果可能不一样233

生成随机数的参考：[Python随机函数random使用详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/98298060) 



### Putting it all together

```
Not exactly a question to solve~
```

国内有点问题，算了 MD，手动试了，应该不是 vpn 的问题 估计是年代久远（也就3年）没人维护废弃了



# Done!

感谢最后 Fun Question! 给了我一点信心，呜呜呜呜
