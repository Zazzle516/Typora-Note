# Cats

其实是自动纠错 + 拼写检查 缩写成 cats 233

如何在 `OK` 环境下 DEBUG：`print("DEBUG:",x)` 评分系统会忽略这条输出

## Phase 1: Typing

### Problem 1

```shell
python ok -q 01 -u --local
```

```shell
python ok -q 01 --local
```

在 power shell 确定实现思路的时候出了点问题，去翻了翻	这个题目要求有点怪 根据 `select()` 的返回结果去排序 根据排序出来的结果返回 `Kth selection` 最开始以为只是按照列表下标排序

注意下标的起始位置



### Problem 2

```shell
python ok -q 02 -u --local
```

```shell
python ok -q 02 --local
```

返回可以传递给 `select()` 的函数 

判断该词是否存在于 `topic` 词组中，需要忽略大小写和标点符号的影响

```shell
>>> from cats import about
>>> dogs = about(['dogs', 'hounds'])
>>> dogs('A paragraph about cats.')
? False
-- OK! --

>>> dogs('A paragraph about dogs.')
? True
-- OK! --

>>> dogs('Release the Hounds!')
? True
-- OK! --

>>> dogs('"DOGS" stands for Department Of Geophysical Science.')
? True
-- OK! --

>>> dogs('Do gs and ho unds don\'t count')
? False
-- OK! --
```



```python
# my code here
def about(topic):
    def is_contain(sentence):
        # sentence to word 
        words = sentence.split(" ")     # ['word-0','word-1'...]

        # 去除引号对单次的影响 并转变为小写进行判断 
        for word in words:
            if word[0] == '"':
                word = word.join(i for i in word if i not in '"')
            word = word.lower()
            if word in topic:
                return True
        return False
    return is_contain

>>> dogs('A paragraph about dogs.')
# Error: expected
#     True
# but got
#     False
# python ok -q 02 --suite 1 --case 1 --local

# fix
# 不止句号 . 的问题 还有问号 ? 感叹号 ! 之类的 呃...
# 通过使用正则表达式把特殊符号 r'[^\w\s]' 替换为 blank space 

>>> ab = about(['gendarme', 'tigerlike', 'countergabble', 'lollipop', 'regovern', 'acoelomate', 'walnut', 'combat', 'reattribution'])
>>> ab('tig;eRlikE fiscal nwaLnut cou(ntergab!bleg unsuccessiveness reattr<ibuTion acoeloma$te!U EgEndarme scrappily')
# Error: expected
#     True
# but got
#     False
# python3 ok -q 02 --suite 2 --case 4

# fix 
# 那看来题目的意思不是替换成 blank space 而是 None
# Pass!
```



### Problem 3 

```shell
python ok -q 03 -u --local
```

```shell
python ok -q 03 --local
```

判断用户输入单词的正确率 大小写和标点符号也要保持一致 只通过空格区分

特殊情况：即使单词敲对了 但是后面额外加了其他单词 这个单词会整体认为是错误的

```shell
>>> from cats import accuracy
>>> accuracy("a  b  c  d", "b  a  c  d")
? 50.0
-- OK! --

? 0.0
-- OK! --

>>> accuracy("Cat", "cat") # the function is case-sensitive
? 0.0
-- OK! --

>>> accuracy("a b c d", " a d ")
? 25.0
-- OK! --

>>> accuracy("abc", " ")
? 0.0
-- OK! --

>>> accuracy(" a b \tc" , "a b c") # Tabs don't count as words
? 100.0
-- OK! --

>>> accuracy("abc", "")
? 0.0
-- OK! --

>>> accuracy("", "abc")
? 0.0
-- OK! --
```



```shell
>>> accuracy("a b c d", " a d ")
50.0

# Error: expected
#     25.0
# but got
#     50.0
# python3 ok -q 03 --suite 1 --case 1

# fix
# 思路上有问题 不是顺序判断的 而是只要存在就认为得分 并且在判断存在之后不能进行二次判断 维持相对顺序
```

```python
    # BEGIN PROBLEM 3
    # if len(typed_words) > len(reference_words):
    #     return 0.0
    single_word_point = 100 / len(reference_words)  # 每个单词的分值 
    total_points = 0.0

    # for typed_word, reference_word in zip(typed_words, reference_words):
    #     if typed_word == reference_word:
    #         total_points += single_word_point

    # 从效率的角度讲：用 reference_word 去匹配 typed_word 会更快 
    # 核心是经历过比较的单词不能二次对比 列表只能往前走 

    typed_word_index = 0
    reference_word_index = 0
    for index in range(len(typed_words)):
        # python 不支持在不使用 zip 的情况下同时遍历两个列表 不过到了这步只有 len(typed_words) <= len(reference_words) 的情况
        if (typed_word_index == len(typed_words)):
            break
        else:
            if typed_words[typed_word_index] == reference_words[reference_word_index]:
                print("DEBUG:" + " match " + typed_words[typed_word_index] + " " + reference_words[reference_word_index])
                total_points += single_word_point
                typed_word_index += 1
                reference_word_index += 1
            else:
                print("DEBUG:" + " unmatch " + typed_words[typed_word_index] + " " +reference_words[reference_word_index])
                typed_word_index += 1
                if len(typed_words[typed_word_index:]) == len(reference_words[reference_word_index:]):
                    reference_word_index += 1
    # 特殊情况：len(typed_words) > len(reference_words) 认为全错 
    # if total_points == 100.0 and len(typed_words) > len(reference_words):
    #     return 0.0
    return total_points
```

说实话，我没理解他这个判断规则 我换字典结构试试

不行 测试用例是要求顺序的 只能以列表的形式进行查找 

```python
    # BEGIN PROBLEM 3
    "*** YOUR CODE HERE ***"
    single_word_points = 100 / len(typed_words)
    reference_dict = {}
    total_points = 0
    for reference_word in reference_words:
        # 手动进行初始化 
        reference_dict[reference_word] = 0
    for reference_word in reference_words:
        # 统计每个词出现的次数 
        reference_dict[reference_word] += 1
    for typed_word in typed_words:
        if typed_word in reference_dict and reference_dict[typed_word] > 0:
            reference_dict[typed_word] -= 1
            total_points += single_word_points
    return total_points
    # END PROBLEM 3
```

看了答案... 我确实有智力障碍吧估计 是我考虑的太复杂了，题目没有那么复杂的

问题出现在一个对测试用例的错误理解

```shell
>>> accuracy("a b c d", " a d ")
# 题目本身的意思是 typed_words 的有效判断区间是 [a b]
# 在 [a, b] 中有 a 符合 referene_word 那么有 50 分
# 但是我理解成 typed_words 会跳过 [b, c] 判断 [a .. d] 中有 a d 符合也是 50... 
```



### Problem 4

```shell
python ok -q 04 -u --local
```

```shell
python ok -q 04 --local
```

没啥



# Phase1 Pass!

```
python cats.py -t cats kittens --local
```

```
python gui.py
```

