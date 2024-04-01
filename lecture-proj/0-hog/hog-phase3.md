# Hog-Strategies

使用一些函数来决定扔色子的次数 感觉和游戏本体的关系已经不大了

## Phase 3 Strategies

### Problem 8

```python
# unlock
python ok -q 08 -u --local

# check
python ok -q 08 --local
```

好像是三层函数嵌套 第二层的参数要原样传给第三层

要求有变长的参数输入 new python syntax

```python
# Q1
What makes make_averaged a higher order function
#  It both takes in a function as an argument and returns a function

# Q2
dice = make_test_dice(3, 1, 5, 6)
averaged_roll_dice = make_averaged(roll_dice, 1000)
# original_function = roll_dice
# averaged_roll_dice() 3.75

# Enter a float (e.g. 1.0) instead of an integer
# 接收的这个参数 *args 
averaged_roll_dice(2, dice)
# roll_dice(2, dice)	(3,1)=> 1 (5,6)=> 11  (1 + 11)/2 = 6
# correct 6.0
```

莫名其妙的成功了.. 呃，其实我还没看懂这个题目问的是什么 然后那个 `console` 我也没懂为什么是 6.0

解释一下：`averaged_roll_dice()` 的两个参数其实是 `roll_dice()` 的两个参数，计算扔两次后的结果

 

### Problem 9

```python
# unlock
python ok -q 09 -u --local

# check
python ok -q 09 --local

# run this function on a randomized dice
python hog.py -r
```

我明白了。

这个 console 测试是这个意思：在已经知晓自己的色子扔出的点数的时候，我要扔几次才能让我的得分最大化

如果色子是个固定值，那么就是最多次 10

如果色子不是固定值，不同的次数可能会有区别，比如 `Pig out` 规则

（每次取不同的次数 比如 2 次或者 3 次，可能有的含 1 有的不含有，那么结果也会不同）

这个方法只能用在已知情况下，未知情况下用不了

之所以要用到 `roll_dice()` 是因为里面实现了对 `Pig out` 规则的判断

就是要对这个结果取 `trials_count` 次数的平均值，不知道为啥要这么干，反正结果是固定的



```python
# Q1
>>> dice = make_test_dice(3)   # dice always returns 3
>>> max_scoring_num_rolls(dice, trials_count=1000)
# correct 10
```



### Problem 10

```python
# unlock
python ok -q 10 -u --local

# check
python ok -q 10 --local
```



### Problem 11

```python
# unlock
python ok -q 11 -u --local

# check
python ok -q 11 --local
```

针对 `Swine Swap` 规则进行的优化

如果进行交换规则后变高了（不严格）那就 `return 0` 否则就投掷色子

也是在 Problem 10 的基础之上进行判断



```python
# error
>>> swap_strategy(50, 45, 8, 10)
0

# Error: expected
#     10
# but got
#     0

#  17 test cases passed before encountering first failed test case
python ok -q 11 --suite 2 --case 11 --local
```



### Problem 12

实现 “终极无敌超级策略” 结合上面的所有判断策略，以及你能实现的对抗 `always_roll(4) ` 有高胜率的策略

主要是我没啥想法.. emmm.. 我就不做了2333

```python
# check win rate
python calc.py --local
# 好像访问不了.. 
```



# Phase3 Pass!

```python
# check and submit
python ok --score

# play with a friend
python hog_gui.py
```

