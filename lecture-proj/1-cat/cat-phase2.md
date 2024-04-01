# Cat-Autocorrect

在第一阶段实现了选择语句，拼写的正确性判断和时间

在第二阶段实现自动纠错的功能 在用户敲击空格后把相似但是不正确的单词替换为正确的



## Phase2-Autocorrect

### Problem 5

```shell
python ok -q 05 -u --local
```

```shell
python ok -q 05 --local
```

会通过一个 `diff_function()` 判断该 `user_word` 和目前的所有单词的不同

选择差距最小的那一个 如果差距都大于 `limit` 那么直接返回

```shell
>>> first_diff = lambda w1, w2, limit: 1 if w1[0] != w2[0] else 0
>>> autocorrect("wrod", ["word", "rod"], first_diff, 1)
? 'word'
-- OK! --

>>> autocorrect("inside", ["idea", "inside"], first_diff, 0.5)
# 已经存在直接返回 
? 'inside'
-- OK! --
```

我是天才，反向查列表返回的第一个最小值就可以直接更新了	嘿嘿	:）



### Problem 6

```shell
python ok -q 06 -u --local
```

```shell
python ok -q 06 --local
```

如果两个单词长度一致 需要保持严格顺序

如果当前不匹配的字符数已经超过了 `limit` 那么直接返回一个大于 `limit` 的值，减小后续计算量

题目要求使用递归实现	注意递归中可能需要传递总值判断，同时也要加上单个步骤的值，不要把结果重复相加



### Problem 7

```shell
# nothing to unlock for this one
```

```shell
python ok -q 07 --local
```

通过 3 种操作判断两个单词之间的不同 `add_letter()`	`remove_letter()`	`substitute_letter()` 

通过观察题目给出的例子可以看出这个 `add_function()` 算是头插法

```python
# e.g. purng -> purring
# 比较到 purng (n,r) 插入 r => pur(r_new)ng	比较 (n,i) 插入 i => purr(i_new)ng
```

区分替换字符和添加字符的功能 判断点在下一个字符	如果相同是替换	否则是添加

```python
    if ______________: # Fill in the condition
        # BEGIN
        "*** YOUR CODE HERE ***"
        
        # END

    elif ___________: # Feel free to remove or add additional cases
        # BEGIN
        "*** YOUR CODE HERE ***"
        # END

    else:
        add_diff = ...  # Fill in these lines
        remove_diff = ... 
        substitute_diff = ... 
        # BEGIN
        "*** YOUR CODE HERE ***"
        # END
```

虽然解决了... 但是解决的有点丑陋 算了 反正 Pass 了



# Phase1 Pass!

```shell
python gui.py
```

也可以实现一个自己的 `final_diff()` 我懒 ✌️
