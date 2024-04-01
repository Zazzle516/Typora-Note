# Hog-Commentary

实现跟踪游戏流程的功能

commentary function

takes two arguments. Player 0's current score and Player 1's current score

print out commentary based on either or both current scores and any other information in its parent environment

a commentary function always returns another commentary function to be called on the next turn 总是返回自己 是个递归调用的样子，但是为什么要搞成递归的方式呢

当只有一个 side effect 的时候这个语句会被打印

## Phase 2 Commentary



```python
# eg say_scores()
# 返回自己并打印当前的分
# 因为是递归调用 肯定不能直接使用 需要外面的高层级函数进行控制
```



```python
# eg announce_lead_changes()
# 1. 只有在领先关系改变的时候才会更新
# 2. 通过 leader 和 last_leader 判断是否发生了更新
# 3. 通过 HOF 控制无限循环
```



```
# eg both()
# 1. 综合上面的两种方式 打印目前的得分状况和领先情况
```



### Problem 6

```python
# unlock
python ok -q 06 -u --local

# check
python ok -q 06 --local
```

更新 `play_function` 完成上面的功能，直接调用传参的函数 `say()` 表示

注意结果应该是所有规则作用完之后的

```python
# Q1
from hog import play, always_roll
from dice import make_test_dice
def echo(s0, s1):
    print(s0, s1)
    return echo
s0, s1 = play(always_roll(1), always_roll(1), dice=make_test_dice(3), goal=5, say=echo)
(line 1) 3 0
(line 2) 3 3
(line 3) 9 3
```

```python
# Q2
from hog import play, always_roll, both, announce_lead_changes, say_scores
from dice import make_test_dice
def echo_0(s0, s1):
    print('*', s0)
    return echo_0
def echo_1(s0, s1):
    print('**', s1)
    return echo_1
s0, s1 = play(always_roll(1), always_roll(1), dice=make_test_dice(2), goal=3, say=both(echo_0, echo_1))
# 应该是后续实际测试中的形式
(line 1) * 2
(line 2) ** 0
(line 3) * 2
(line 4) ** 2
(line 5) * 4
(line 6) ** 2
```



```python
# error
>>> def count(n):
...     def say(s0, s1):
...         print(n, s0)
...         return count(n + 1)
...     return say
s0, s1 = play(always_roll(1),always_roll(1),dice=make_test_dice(3),goal=10,say=count(1))

# Error: expected
#     1 3
#     2 3
#     3 9
#     4 9
#     5 15
# but got
#     1 3
#     1 3
#     1 9
#     1 9
#     1 15

# 2 test cases passed before encountering first failed test case
python ok -q 06 --suite 2 --case 2 --local

# fix
# 插入位置是对的 因为 score 结果是正确的 但是序号没有得到更新
# 给出的函数无一例外都是在子函数中接收 s0 s1 
# 是因为每次直接调用了 say function 应该让它自己调用自己 n 才会更新

say = say(score0, score1)
```



### Problem 7

```python
# unlock
python ok -q 07 -u --local

# check
python ok -q 07 --local
```

每次只保留了当前玩家的历史最高得分，如果有更高的，刷新并调用子函数

通过 `who` 来判断玩家 `id` 

```python
# Q1
When does the commentary function returned by announce_highest print something out
# When the current player, given by the parameter `who`,earns their biggest point increase yet in the game.

# Q2
What does the parameter last_score represent
# The current player's score before this turn
```



```python
# error

>>> f0 = announce_highest(1) # Only announce Player 1 score gains
>>> f1 = f0(12, 0)
>>> f2 = f1(12, 10)
10  point(s)! That's the biggest gain yet for Player  1

# Error: expected
#     10 point(s)! That's the biggest gain yet for Player 1
# but got
#     10  point(s)! That's the biggest gain yet for Player  1

# 3 test cases passed before encountering first failed test case
python ok -q 07 --suite 2 --case 1 --local

# fix
# 空格.. 我真服了

# error
>>> f0 = announce_highest(1) # Only announce Player 1 score gains
>>> f1 = f0(12, 0)
>>> f2 = f1(12, 10)
10 point(s)! That's the biggest gain yet for Player 1
>>> f3 = f2(20, 10)
>>> f4 = f3(22, 20)
10 point(s)! That's the biggest gain yet for Player 1
# Error: expected

# but got
#     10 point(s)! That's the biggest gain yet for Player 1

# fix
# 猜测是 max_score_gap 根本没有更新 一直是默认 0

# error
>>> s0, s1 = play(always_roll(1), always_roll(1), dice=make_test_dice(5, 3, 5), goal=10, say=announce_both)
3 point(s)! That's the biggest gain yet for Player 0
3 point(s)! That's the biggest gain yet for Player 1

# Error: expected
#     5 point(s)! That's the biggest gain yet for Player 0
#     3 point(s)! That's the biggest gain yet for Player 1
# but got
#     3 point(s)! That's the biggest gain yet for Player 0
#     3 point(s)! That's the biggest gain yet for Player 1

# 4 test cases passed before encountering first failed test case
python ok -q 07 --suite 2 --case 2 --local

# fix
# 应该是没有对玩家 id 相同的保持一个运行环境 知道是这里的问题但是改起来会有点麻烦...
# 不知道是在 play() 哪里调用的 应该和 Problem6 差不多的位置
# emm.. 没想出来 不知道怎么在一个函数里面创造两个用户的运行环境

# 换个思路 一开始就给出两个玩家 id 然后在函数内部选择不同的变量
# 嘶.. 修改了原函数的传参要求 增加了两个 通过了.. 去看看答案

# answer
def announce_highest(who,last_score = 0 , runnng_high = 0):
    assert who == 0 or who == 1,'The two argument'
    def track_high(score0 , score1 , highest = running_high):
        if who == 0:
            this_score = score0
        else:
            this_score = score1
        gain = this_score - last_score
        if gain > highest:
            print(gain,"point(s)! That's the biggest gain yet for player",who)
            highest = gain
            return announce_highest(who, this_score, highest)
        return track_high
# 我个人觉得呢 是因为 last_score 这里 我不认为直接调用可以调用到正确的值 但是从答案来看是可以的
# emm... 其实还是父函数的问题 在 return announce_highest 中 从调用开始就切开了两个环境
# 也就是 who == 0 只能反复调用 who == 0 的环境
# 我最开始也是这么想的 但是出现了数值覆盖的现象 1. para1.not_impotant 没有参与运算
# 所以实际上不是数值更新 而是压根没有计算 who == 0 的情况
```





# Phase2 Pass!

```python
# see change in GUI
python hog_gui.py
```

