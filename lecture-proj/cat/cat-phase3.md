# Cat-Multiplayer

实现多人游戏功能，可以连接到课程官网然后寻找一个对手玩家

为了实现玩家对抗，以下 5 个程序会一直运行：

1. GUI 界面，处理网站显示中的词语高亮
2. 当前玩家的 `gui.py` 文件，和当前玩家 GUI 界面交流的网站服务器，通过 `cat.py` 提供服务
3. 对手玩家的 `gui.py` 文件
4. 对手玩家的 GUI 文件
5. cs61A 的多人服务器，匹配玩家并交流玩家信息

前端 GUI 和后端 `gui.py` 通信的过程：

1. 当你打字的时候，GUI界面会把你的结果发给 `gui.py` 服务器，返回相应结果，同时也会更新你在对手端的信息
2. 同时，你的 GUI 也会一直循环访问 `gui.py` 的进度，多人服务器会对两个玩家轮流访问
3. 每个玩家都会有一个 id 号



### Problem 8

```shell
python ok -q 08 -u --local
```

```shell
python ok -q 08 --local
```

每当用户敲完一个单词，就调用 `report_progress()` 函数

`typed_list` 中词的数量永远 ≤ `prompt_list` 	遇到第一个错误单词后停止判断，总数是以 `prompt_list` 为准

通过一个字典储存结果，和多人服务器进行沟通

```shell
>>> report_progress(typed, prompt, 1, print_progress)
(line 1)? ID: 1 Progress: 0.6
(line 2)? 0.6
```



### Problem 9

```shell
python ok -q 09 -u --local
```

```shell
python ok -q 09 --local
```

记录每个玩家敲完每个单词的时间，用一个二维数组记录 `times[i][j]` `i-time; j-word` 

每个玩家的时间戳是累加而且一直在增长的 `times_per_player` 记录的是真实时间 分别代表两个玩家

```shell
>>> p = [[75, 81, 84, 90, 92], [19, 29, 35, 36, 38]]
>>> game = time_per_word(p, ['collar', 'plush', 'blush', 'repute'])
>>> all_words(game)
['collar', 'plush', 'blush', 'repute']
>>> all_times(game)
[[6, 3, 6, 2], [10, 6, 1, 2]]
```



```shell
>>> p = [[0, 2, 3], [2, 4, 7]]
>>> game = time_per_word(p, ['hello', 'world'])
>>> word_at(game, 1)
? 'world'
-- OK! --

>>> all_times(game)
? [[2, 1],[2, 3]]
-- OK! --

>>> time(game, 0, 1)
? 1
-- OK! --
```

`time()` 返回玩家 `id = player_num` 下标是 `word_index` 的输入时间



### Problem 10

```shell
python ok -q 10 -u --local
```

```shell
python ok -q 10 --local
```

找出每个玩家输入的最快的字符，在多个玩家都结束输入之后才会被调用 针对每个单词而言（可能有 3 个玩家

通过 `cat.py` 给出的 `game` 数据结构的选择器访问数据

`fastest_word()` 为每个玩家返回对应的一列单词，考虑到平局的可能，把 id 号小的排在前面

```shell
>>> p0 = [2, 2, 3]
>>> p1 = [6, 1, 3]
>>> fastest_words(game(['What', 'great', 'luck'], [p0, p1]))
? [['What', 'luck'], ['great']]
-- OK! --
```



在写这道题的时候出现了很奇怪的问题

```python
# min_time_word_list [[], []] 二维列表
# 理想中: min_time_word_list[0].append(x) => [[x], []]
# 实际上: min_time_word_list[0].append(x) => [[x], [x]]
# 两个子列表同时被改变了
min_time_word_list[single_word_min_time_player].append(game[0][single_word_index])

# 但是我在 python 测试了结果又不是这样的
>>> a = [[],[]]
>>> a[0].append(1)
>>> a
[[1], []]
```

[python之append方法容易踩的坑 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/161266559) 

所以，需要把这两个子列表的地址分离吗?  嘶... 没有找到很好的分离方式 考虑过 `extend()` 但是还是一样的结果，而且是以字符的形式追加的

```python
def fastest_words(game):
    players = range(len(all_times(game)))  # An index for each player 玩家总数
    words = range(len(all_words(game)))    # An index for each word 单词总数 
    # BEGIN PROBLEM 10
    "*** YOUR CODE HERE ***"
    import copy
    player_id_list = list(players)[::-1]
    print("DEBUG: " + "reversed_player_index: " + str(player_id_list))
    # 多维列表按列的访问    两层循环实现 
    single_word_min_time = float('inf')
    single_word_min_time_player = -1
    min_time_word_list = [[]] * len(players) 
    print("DEBUG: " + "resulr_list_init: " + str(min_time_word_list))
    
    # 找到每个单词打印最短时间的玩家 id 
    for single_word_index in words:
        print("DEBUG: " + "outter_loop: " + str(single_word_index))
        for single_player_index in player_id_list:
            single_word_time = time(game, single_player_index, single_word_index)
            print("DEBUG: " + str(single_word_time))
            if single_word_time < single_word_min_time:
                single_word_min_time = single_word_time
                single_word_min_time_player = single_player_index
                print("DEBUG: " + "find player: " + str(single_player_index))
        # 离开内层循环 此时已经找到某一个单词的最小值的玩家 id 
        print("DEBUG: " + "sub_resulr_list: " + str(min_time_word_list[single_word_min_time_player]))
        min_time_word_list[single_word_min_time_player].append(word_at(game, single_word_index))
        print("DEBUG: " + "resulr_list: " + str(min_time_word_list))
    return min_time_word_list
```

发现个 bug 对 `single_word_min_time_player` 的设定应该放在外层循环内部



看看别人是怎么实现的

```python
def fastest_words(game):
    players = range(len(all_times(game)))  # An index for each player
    words = range(len(all_words(game)))    # An index for each word
	# 鬼鬼 从来没见过的新鲜写法
    # DEBUG: init: [[], []] 显示看不出区别
    fastest = [[] for _ in players]
    
    # 思路和我其实是一样的 就是通过赋值避开了我上面的问题
    for word_index in words:
        min_time = float('inf')
        player = 0
        for player_index in players:
            if time(game, player_index, word_index) < min_time:
                min_time = time(game, player_index, word_index)
                player = player_index
        fastest[player].append(word_at(game, word_index))
    return fastest
```

只能说学到了，不能通过 `[[]] * K` 的方式定义子列表，通过循环迭代可以构造独立的子列表

```python
# 可以证明
a = [[]] * 3
b = [[] for i in range(0, 3)]
print("------ a --------")
print(id(a[0])) # 2442303854528
print(id(a[1])) # 2442303854528
print("------ b --------")
print(id(b[0])) # 2442303854848
print(id(b[1])) # 2442303854016
```

学到了有意思的东西😶‍🌫️



## Extra Credit

### Problem 1

```shell
python ok -q EC1 --local
```

不太理解，我觉得这个题目也没有说的很清楚

```shell
>>> key_distance_diff("wird", "word", 4)
0.6834861261734088
>>> key_distance_diff("aird", "word", 4)
1.650081475501692
>>> key_distance_diff("bord", "word", 4)
2.464344273986475

# Error: expected
#     2
# but got
#     2.464344273986475

# 会有这样的问题 但是我找到的答案没有特别好... 
```

```shell
# 只找到 https://github.com/PKUFlyingPig/CS61A 这个版本的实现 但是我觉得他的实现也有点问题..
```



### Problem 2

```shell
python ok -q EC2 --local
```

所以这两个问题暂时先放在这里... 



# Phase3 Pass!

