# Lab 7 Midterm Review

[Lab09 - CS61A of UCB(2021-Fall) - MartinLwx's Blog](https://martinlwx.github.io/zh-cn/lab09-cs61a-of-ucb/)

## Recursion and Tree Recursion

### Q1: Subsequences

```shell
python ok -q subseqs --local
```

从整体上看，是想完成从一个完整列表拆分出所有不完整的子列表的，不限顺序，从递归思考。

如果用递归的话，不论如何实现，最后会有重复子列表的情况，换个方法用循环吧。

```
[]
[1] 
[1,2] [1,3] [1,2,3]
[2] [2,3] 
[3]
```

拆分为两步实现：

1. 在所有子列表的固定位置插入元素
2. 逆向应用，逐步把元素添加到子列表中

不知道，想了半天没想出来...

看了答案结果还是递归，我的思考还是有点问题（可能也是有段时间没碰然后捡起来

假设待处理的元素序列 = 第一个新元素 + 已经找到所有可能子序列的旧序列，

问题演变为：如何包含第一个新元素的子序列	解决方法 => 把新元素添加到每个子序列，和旧的子序列拼装起来

*********************

知道问题在哪了，这个递归是反向列表完成的，不会出现重复的情况，而我思考的是正向的（倒不如说根本没有深入思考递归的情况

```
[] [3]
[2] [2,3]
[1] [1,2] [1,2,3] [1,3]
```

```python
def insert_into_all(item, nested_list):
	return [[item] + sub_list for sub_list in nested_list]
def subseqs(s):
    if len(s) <= 1:
        # 定义递归出口 给出起始元素 [] 为后面的子串增长 & 空串情况
        return [[], s] if s != [] else [[]]
    else:
        temp = subseqs(s[1:])
        return insert_into_all(s[0], temp) + temp
```

真的不容易想到... 逆向思考很容易死角啊ಥ_ಥ	算是默下来了.. 我不知道有没有用



### Q2: Increasing Subsequences

```shell
python ok -q inc_subseqs --local
```

有点难啊... 什么玩意，这期中考试难度高成这个样子

生成的子串中，找到严格非降序的位置连续元素组成的子串	呃... 我想想，想不出来算了

**********

对，我又去看了答案，思路比较相反，上一题的例子是向前拼接，这一题是向后？

还是递归：把列表 S 拆分成两部分，`S[0]` 和假设已经是无降序的子序列，向前拼接

*******

高估我自己了，不懂，思考过程在下面

```python
def insert_into_all(item, nested_list):
    return [[item] + sub_list for sub_list in nested_list]

def inc_subseqs(s):
    def subseq_helper(s, prev):
        """
        子函数有 3 个出口
        1. 空串返回
        2. 非递增：很巧妙，因为默认子串是符合要求的，所以只能往后找子串
        3. 我.. 想不明白    智力差距吧可能  b 是 a 的一种前置状态   确定 b 之后根据 b 去往前找 a 
        但是为什么 a 的执行顺序在 b 之前呢

        其实 a 就是负责递归到最后两个元素的 实际是 b 在进行有效比较
        那样就还有个问题 在最后 insert_into_all 用的是 s[0] 而不是 prev

        又去看了原作者的注释，更看不懂了，乐    a-include_s[0]  b-exclude_s[0]
        如果仅从是否包含 s[0] 来看 那 prev 不会和 s[0] 重复吗

        在 b 的情况里 prev 是一直传递下去的 保持不变 结合二号出口，认为 b 目的是找到所有以 prev 开头的符合要求的子序列
        而 a 的目的就是对所谓默认符合的子序列的向后的查找 更新 b 运算的 prev
        """
        if not s:
            # 空串的情况直接返回
            return [[]]
        elif s[0] < prev:
            return subseq_helper(s[1:], prev)
        else:
            # 从左到右 针对每一个元素都进行一次非降序的遍历
            a = subseq_helper(s[1:], s[0])  # 递归入口
            b = subseq_helper(s[1:], prev)  # 第一轮递归之后 prev: 1 返回 a 执行的函数帧 好精妙... 
            return insert_into_all(s[0], a) + b
    # prev 初始化为 0 的目的是判断是否整个序列符合非降序的状态
    return subseq_helper(s, 0)

    
seqs = inc_subseqs([1, 3, 2])
print(seqs)                 # [[1, 3], [1, 2], [1], [3], [2], []]
print(sorted(seqs))         # [[], [1], [1, 2], [1, 3], [2], [3]]
```

不明白，不懂，你杀了我也没戏



## Mutable Lists

### Q3: Trade

```shell
python ok -q trade --local
```

找两个列表的最短的相同的前缀和，然后交换该前缀（两个前缀的长度可以不同

最笨的方法：为每个列表计算所有的前缀和，然后进行比较，如果有相同的，交换

快慢指针：让更小的指针往前走，如果反超了，停下，让另一个指针走（提示的应该也是这个思路

做对啦~



### Q4: Reverse

```shell
python ok -q reverse --local
```

反转列表



## Nonlocal

### Q5: Glookup

```shell
python ok -q make_glookup --local
```



## Recursion / Tree Recursion

### Q6: Number of Trees

```shell
python ok -q num_trees --local
```

根据叶子数量倒推树的结构	How many full binary trees have exactly n leaves

根据卡特兰的数学原理去写（至于推导，呃... 😢

[「算法入门笔记」卡特兰数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/97619085) 

适用于 n = 0 开始
$$
C_1 = 1,C_n = C_{n - 1}\frac{4n - 2}{n + 1}
$$
从编程角度的递归来看（对 n 的开始位置没有要求）：
$$
h(n) = h(0)*h(n -1) + h(1)*h(n-2) + \cdots + h(n - 1)*h(0)
$$

```python
def get_catalan(n):
    if n == 1 or n == 2:
        return 1
    else:
        catalan = 0
        for i in range(1, n):
            # 计算的是 h(i) * h(j)
            # 加法是在递归返回的时候把结果相加
            catalan += get_catalan(i) * get_catalan(n - i)
    return catalan
```



## Nonlocal

### Q7: Advanced Counter

```shell
python ok -q make_advanced_counter_maker --local
```



# Done!

这节练习的递归好难ಥ_ಥ... 其他的倒还好
