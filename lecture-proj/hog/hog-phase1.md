# Hog-Rules

### Rules：

有两个玩家轮流扔色子，直到有一个玩家获得了累计 100 的积分

玩家可以选择扔色子的次数，每轮最多扔 10 次，这名玩家本轮的得分就是扔的色子的结果总和

如果其中一个玩家扔的色子过多会有以下规则的风险

#### Pig Out

如果有一次扔色子的结果是 1 那么当前这名玩家本轮的得分就是 1 



#### Free Bacon

本轮中一次都不扔色子的玩家，会获得 (10 - ones_digit + tens_digit) 分，`tens_digit` 是对手分数的十位数

`ones_digit` 是对手分数的个位数，如果不存在十位数，默认为 0 



#### Feral Hogs

如果本轮你**扔色子的次数**和你**之前一轮的得分**绝对值差 2 ，本轮会额外获得 3 点分数

计算上一轮点数获取的时候，不要额外考虑作用在之前那轮的 Feral Hogs 和 Swine Swap 规则

注意，每次比较的得分是上一轮的**单次得分**，和累计分值没有关系



#### Swine Swap

在把本轮得分累加到玩家总得分的时候，如果当前玩家得分的个位数和对手玩家的得分个位数的差值绝对值，等于对手玩家的十位数，那么这两个玩家的得分交换

如果是不存在的位数，比如目前只有个位数，那么十位数默认为 0 



### Start File

```
hog.py			A starter implementation of Hog
dice.py			Functions for rolling dice
hog_gui.py
ucb.py
ok
tests
gui_files
```



### Run File

```python
cd directory hog

# test
python ok --local
python ok -q [question number] -i	# question number eg.01

# Debug
print("DEBUG:", x)
```



### GU interface

```python
# run after complete play function
python hog_gui.py
```



## Phase 1 Simulator

develop a simulator for the game of hog

### Problem 0

在 `dice.py` 文件中，模拟了扔色子的过程

因为每次调用返回的结果不同，实验指导书上有说它是一个 `non-pure zero-argument functions` 

针对色子的不同的结果概率应该都是相同的 `six_sided` 

因为后面的测试函数需要特定场景，所以这里也需要创造一个确定的色子序列，传给 `make_test_dice` 函数

阅读 `dice.py` 然后完成测试，尽量不要中断，可能会出问题

```python
python ok -q 00 -u --local
```

```
# python grammer
# python assert 用于判断一个表达式 在表达式为 False 时执行
# 可以在条件不满足的情况下直接返回错误 而不用等待运行后崩溃的情况
```



```python
Q: Which of the following is the correct way to "roll" a fair, six-sided die?
Choose the number of the correct choice:
# correct six_sided()
	# return randint(1,6)
```



### Problem 1

```python
# unlock
python ok -q 01 -u --local

# check
python ok -q 01 --local
```



```python
# my code here
sum = 0
for i in range(1,num_rolls + 1):
    temp_res = dice()
    if(temp_res == 1):
        return 1
    sum += temp_res
return sum

# test
>>> counted_dice = make_test_dice(4, 1, 2, 6)
>>> roll_dice(3, counted_dice)
1
>>> roll_dice(1, counted_dice)
2	# expected 6

# error 
# 4 test cases passed before encountering first failed test case
python ok -q 01 --suite 1 --case 5 --local

# reason
# 因为我在 temp_res 判断的时候 遇到结果为 1 的就提前退出 导致下一次读取的数字位置不对
# 要保证测试的正确性 必须完成所有扔色子的操作 最后判断有没有 1 出现
```

```python
# Pass!
def roll_dice(num_rolls, dice=six_sided):
    sum = 0
    is_dice1 = False

    # 扔足够的次数 
    for i in range(1,num_rolls + 1):
        temp_res = dice()
        if(temp_res == 1):
            is_dice1 = True
        else:
            sum += temp_res

    # 结果判断 
    if(is_dice1 == True):
        return 1
    return sum
```



### Problem 2

```python
# unlock
python ok -q 02 -u --local

# check
python ok -q 02 --local

# interactively test
line_one: python -i hog.py
line_two: free_bacon
```



### Problem 3

```python
# unlock
python ok -q 03 -u --local

# check
python ok -q 03 --local
```

![](D:\cs-61A\lecture-project\hog-pic\1.png)

给出的是对手目前的分值，因为要用到 `Free Bacon` 规则进行判断

至于当前玩家只需要给出本轮的得分结果即可（反正你也不知道当前玩家的分值）



### Problem 4

```python
# unlock
python ok -q 04 -u --local

# check
python ok -q 04 --local
```

实现 `Swine Swap` 规则

```python
# error
>>> from hog import *
>>> is_swap(960, 144)
False

# Error: expected
#     True
# but got
#     False
# 8 test cases passed before encountering first failed test case
python ok -q 04 --suite 1 --case 9 --local

# fix
# 其实就是数字达到 3 位数的时候 处理 2 位数的 // 10 运算就失效了
# 再套一层 % 10 就可以
```



### Problem 5a

```python
# unlock
python ok -q 05a -u --local

# check
python ok -q 05a --local
```

实现一个完整的游戏流程，目前可以不考虑 `Feral Hogs` 规则

每轮投掷的次数目前也不用考虑，那个在 `Phase 2` 实现



#### Some Hints

- 每轮投掷只使用一次 `take_turn` 函数
- 实现除了 `Feral Hogs` 之外所有的特殊规则
- 通过 `other(player_id)` 获得对方的分数
- 可以忽视 `say statement` 那个也要在后面实现



#### Implenment

Q: 在 `Swine Swap` 规则中，和对手玩家的得分比较的时候，是对方投掷之后还是之前

在例子中，At the end of the first player's turn

那这样，岂不是只有第一个玩家可以得到这样的交换得分的机会，还是说没有轮次的概念，两个人轮流扔就完事了

再看一遍规则细则，应该是轮流那种，第二个玩家也可以被视为第一个玩家



```python
# unlock
Q1: What is a strategy in the context of this game?
# correct A function that returns the number of dice a player will roll

Q2: If strategy1 is Player 1's strategy function, score0 is
Player 0's current score, and score1 is Player 1's current
score, then which of the following demonstrates correct
usage of strategy1?
# correct strategy1(score1, score0)
```



```python
# test
while(score0 < GOAL_SCORE and score1 < GOAL_SCORE):
    score0 += take_turn(strategy0(score0,score1),score1,dice)
    if(score0 >= GOAL_SCORE or score1 >= GOAL_SCORE):
        break
    score1 += take_turn(strategy1(score1,score0),score0,dice)
    if(score0 >= GOAL_SCORE or score1 >= GOAL_SCORE):
        break
        
# error 1
# 4 test cases passed before encountering first failed test case
python ok -q 05a --suite 2 --case 2 --local

# fix
# 在 coding 的时候错误理解了全局变量 GOAL_SCORE goal
# goal 默认是 GOAL_SCORE 但是也可以人为改变 问题就出在我没有通过变量 goal 判断

# error 2
# 5 test cases passed before encountering first failed test case
python ok -q 05a --suite 2 --case 3 --local

# fix
# 最开始没有在得到结果对 swap 规则进行判断 加上就行
```



### Problem 5b

```python
# unlock
python ok -q 05b -u --local

# check
python ok -q 05b --local

# re-check
python ok -q 05a --local
```

实现 `Feral Hogs` 规则

如果本轮你投掷的次数和之前的得分相比绝对值差 2 那么额外加 3 分

重要的变量：

- 上一轮次得到的结果
- 本轮投掷的次数

所以其实不需要通过数组记录所有结果，只要保持两个变量即可

```python
# test

# Error Hint
# Failing this test means something is wrong, but you should look at other tests
# 8 test cases passed before encountering first failed test case
python3 ok -q 05b --suite 2 --case 6

# fix
# 其实我以为这个只测试 feral_hogs = True 的情况 也有 False 的情况 所以出错了
# 我的解决办法有点丑陋 套一层 if-else :)

103 test cases passed! No cases failed.
```

# Phase1 Pass!

```python
# run with GUI	port:31415
python hog_gui.py

# check your score
python ok --score
```