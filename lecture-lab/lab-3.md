# Lab 3 Recursion and Tree Recursion

### Q1: Recursion

```python
python ok -q recursion-wwpd -u --local
```

#### case1

```python
>>> def f(a, b):
...     if a > b:
...         return f(a - 3, 2 * b)
...     elif a < b:
...         return f(b // 2, a)
...     else:
...         return b
>>> f(2, 2)
# correct 2
>>> f(7, 4)
# correct 4
>>> f(2, 28)
# correct 8
>>> f(-1, -3)
# correct Infinite
```



### Q2: Journey to the Center of the Earth

```
python ok -q sr-wwpd -u --local
```

#### case1

```python
>>> def crust():
...   print("70km")
...   def mantle():
...       print("2900km")
...       def core():
...           print("5300km")
...           return mantle()
...       return core
...   return mantle
>>> drill = crust
>>> drill = drill()
# correct 70km
>>> drill = drill()
# correct 2900km
>>> drill = drill()
(line 1)5300km
(line 2)2900km
>>> drill()
(line 1)5300km
(line 2)2900km
(line 3)Function
```



### Q3: Pascal's Triangle

```python
python ok -q pascal --local
```

```python
# triangle position count from row_0 column_0
row_0    	1
            1 1
            1 2 1
            1 3 3 1
            1 4 6 4 1
current_pos = [row - 1][column -1] + [row - 1][column]
# keep going up
pos[1][0] = 1
pos[1][1] = 1	# 实际上作为递归终点是 row_1
```

自己调用自己 当前求的行，是根据上一行的结果来求得的

法一：递归迭代

```python
def pascal(row, column):
    """Returns a number corresponding to the value at that location
    in Pascal's Triangle.
    >>> pascal(0, 0)
    1
    >>> pascal(0, 5)	# Empty entry; outside of Pascal's Triangle
    0
    >>> pascal(3, 2)	# Row 4 (1 3 3 1), 3rd entry
    3
    """
    "*** YOUR CODE HERE ***"
    if (row < column):
        return 0
    if(row == 0 and column == 0):
        return 1
    # 递归终点 在后面 [column - 1] = -1 的时候 怎么处理 
    if(row == 1 and column == 0):
        return 1
    if(row == 1 and column == 1):
        return 1
    
    def Triangle():
        res = pascal(row - 1 , column - 1) + pascal(row - 1 , column)
        return res
    return Triangle()
```

```python
# TMD 给我 NB 坏了，我都没想着能成功，艹
# 赶紧纪念一下 然后看看答案怎么写的 笑	突然觉得自己有点厉害hhhhhh
# 赶紧保存一下
```



法二：写一个循环，直接取值

理论上肯定可以，但是 `python` 的二维数组我没咋搞懂，So...



### Q4: Repeated, repeated

```python
python ok -q repeated --local
```

```python
# my code here
def repeated(f, n):
    def count_repeat(x):
        if(n == 0):
            return x
        else:
            return repeated(f,n - 1)(f(x))
    return count_repeat
# 上次在 hw 02 其实是看了答案的 因为不知道怎么接收子函数的参数 现在明白了
# 而且在 return repeated(f,n - 1)(f(x)) 也可以想到套娃使用函数的方式
```



### Q5: Num eights

```python
python ok -q num_eights --local
```

```python
# my code here
# 这个没想出来 主要还是思维上有局限性 因为要求不能有任何赋值语句
# 但是出于思维惯性 总是想着用一个变量来储存结果 实际上可以在递归过程直接传递数字
def num_eights(n):
    # 统计 8 出现的次数 
    # 如果不能赋值 那么怎么表示统计次数 
    def single_num():
        if(x == 0):
            return count
        if(x % 10 == 8):
            # 次数计数 
            count += 1

        return num_eights(x / 10)()
    	# 这里没有意识到 python 直接用除法会转为 Float 
    return single_num
```

```python
# answer
def num_eights(n):
    if n % 10 == 8:
        return 1 + num_eights(n // 10)
    elif n < 10:
        # 不需要等到 n = 0 可以减少一次递归 
        return 0
    else:
        return num_eights(n // 10)
```



### Q6: Ping-pong

```python
python ok -q pingpong --local
```

```python
# my code here
# 在网上找了一圈没找到反向的思路 这个问题估计是无法避免的..
def pingpong(n):
    """Return the nth element of the ping-pong sequence.

    >>> pingpong(8)     # 6
    8
    >>> pingpong(10)    # 8
    6
    # 遇到的数字里包含 8 或 8 可以作为因数的时候 就改变方向 
    # 可以调用刚刚实现的 num_eights() 
    # 和刚刚的函数不同的是 num_eights() 从末尾开始判断 但是 pinpang 要从开头开始
    # 和题目要求不同 反着去递归的时候 要反向运算 
    # 从 1 开始 默认up 

    if(n == 1):
        return 1
    
    if((contain_eight(n)) or (num_eights(n) > 0)):
        return pingpong(n - 1) - 1
    else:
        return pingpong(n - 1) + 1
    
    # 这样编写会有一个问题 就是反向方向改变和原方式差了一步 导致结果会差 2 
```

```python
# answer
def pingpong(n):
    # answer
    # 从底层开始找数 
    def ping(index, value, upordown):	# 虽然叫 ping 但其实是主函数 
        if index == n:
            return value
        if upordown:
            return pong(index + 1, value + 1, upordown)
        else:
            return pong(index + 1, value - 1, upordown)

    def pong(index, value, upordown):	# 方向改变函数 
        if index % 8 == 0 or num_eights(index):
            return ping(index, value, not upordown)
        return ping(index, value, upordown)

    return ping(1, 1, True)	# 给出默认方向 
# 看着答案是很好理解的 但是没有想到.. emm.. 
# 就是拆成两个方向 有点像 both_path() 
```



# Done!

这个 `lab 03` 完成的比我想象快，可能是因为看了两道题的答案...

还是改变不了废物本废的事实 :)