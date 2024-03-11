# Homework03

### Q1: Compose!

题目含义：其实就是一个栈 `function stack` 每次只针对一个参数进行运算

弹出 `operand` 后弹出 `func_addr` 进行运算，再放回去，循环直到栈为空

如果从递归的角度去思考 其实不存在这样一个栈 每次只有一个操作数而已 只有运算方式是记录的

```
python ok -q composer --local
```

```python
# To the Hint the composer() returns two functions
# 最后来看这道题还是没有搞懂 所以去找答案了
def composer(func=lambda x: x):
    def func_adder(g):
        def composed_func(x):
            # 这里的 func() 要套在外面 是我觉得最难想到的步骤 
            return func(g(x))
        # 从例子中能看出 func_addr() 同样返回 func func_addr 也就是 composer
        # 此时要显式的给出目前的复合函数 
        # 因为最早的 x = x 就在最外层 所以会一直在最外层 ? 
        return composer(composed_func)	# func(g(x))
    return func, func_adder

# 从思维习惯上将 一般都是递归到最后一步 x = x 那么这里套在外面 是表示把 g() 运算后的结果返回
# MD 看答案都看不懂 艹了 
```



### Q2: G function

```python
python ok -q g --local
```



### Q3 G_iter function

```python
python ok -q g_iter --local
```

因为是循环迭代的，每次循环内部直接给结果赋值就行



### Q4: Missing Digits

```python
python ok -q missing_digits --local
```

题目：给定了一个正整数，这个正整数的每个数字排列起来是递增的，假设从开头到结尾是连续的，需要给出中间缺失的数字的个数

```python
def missing_digits(n):
    # 因为不知道数字的大小 只能反向求 
    last_num = n % 10
    n = n // 10
    def sub_fun():
        if(n == 0 or last_num == 0):
            return 0
        if(last_num -1 > n % 10):
            return (last_num - n % 10 - 1) + missing_digits(n)
        return missing_digits(n)
    return sub_fun()
```



### Q5: Count change

```python
python ok -q count_change --local
```

给定总数，统计用硬币组合的方式

硬币单位只有 1 cent  2 cent  4 cent  8 cent  16 cent

Hint : 因为要在递归中记录数量 可以用一个 `helper function: count_change` 

感觉，有点难....

```python
# 想的还是有点差距
def count_change(total):
    # 如何拆分出一个 helper function 拆分哪部分 
    # 你明明可以选择杀了我 :)
    if(total > 16):
        return count_single(16,total)
    if(total > 8):
        return count_single(8,total)
    if(total > 4):
        return count_single(4,total)
    if(total > 2):
        return count_single(2,total)
    else:
        return 1
# 在偏小的数 OK 因为小数拆的时候没有碰到需要多次拆分的情况 
# 但是像 100 这种 需要拆成多个 也是我的结果小于正确结果的原因 我只能想到用循环表示拆分次数的方法.. 
def count_single(start_unit , total):
    # 必须包含对应的 start_unit 
    if(start_unit == 1 or total == 0):
        return 1
    if(total < start_unit):
        return count_single(start_unit // 2 ,total)
    return count_single(start_unit // 2 ,total) + count_single(start_unit,total - start_unit)
```



```python
# answer
def count_change(amount):
    import math
    def process(amount, upper):
        if amount == 0 or upper == 1:
            return 1
        if amount < 0:
            return 0
        return process(amount-upper, upper) + process(amount, upper // 2)
    return process(amount, pow(2, round(math.log2(amount))))
```



```python
# 其实分析一下我的思路很接近答案了 其实就是额外的 if 判断限制了递归树 
# 本质还是比较混乱 

# fix version-1
# 删掉 if 分支带来的递归树剪切后 答案还是不对的 但是和答案对比 递归函数本身没有问题 问题出在主函数的调用

# fix version-2
# 不能通过 total 与 start_unit 的大小比较去调用 目前我还不太清楚原因 会在 total = 100 的时候出错
# 其他情况都是可以 Pass 的

# Answer 我明白为什么了...
# 我原本的递归思路没问题 我其实是做出来了233 重点是我没有理解题意
# 我以为只有 1 2 4 8 16 固定的硬币数 但其实题目后面还有一个 etc 也就是 32 64 也可以
# 所以主函数的调用只能通过 pow 求解 其实是主函数写的不对
```



### Q6: Towers of Hanoi

```python
python ok -q move_stack --local
```

经典的汉诺塔问题：通过中间柱子，把圆盘移动位置

维持递归的方式：

1. 把其余的圆盘（n - 1）移动到 `temp` 柱子
2. 把目前最大的圆盘移动到 `end`
3. 把 （n - 1）从 `temp` 柱子移动到 `end` 柱子

经过移动可以发现，起始位置会在除了 `end` 柱子之外的两个柱子交替进行，所以改变 `start` 位置就可以了



### Q7: Anonymous factorial

```python
python ok -q make_anonymous_factorial --local
```

已知的

如果想通过匿名函数调用递归 其实是要赋给这个匿名函数一个名字的 这样才能在它的返回值中调用它自己

```python
fact = lambda n: 1 if n == 1 else mul(n, fact(sub(n, 1)))
fact(5)
# correct 120 the "fact" in the return statement
```

那么如果确实没有函数名，要怎么递归

同时题目要求 `one-line code` emm... 

```
# my thought
emm.. I don't have any thoughs. I guess that's the problem
I will just see the answer
```

首先我其实就有个问题 题目给的这个提示 `fact` 函数里面的那个 1 是干什么的

```python
# python conditional expression 
<expr1> if <conditional_expr> else <expr2>
lambda n:   # return statement 
			1 if(XXX) else YYY
    		<1 expr1> if(XXX: condi_expr) else (YYY: expr2)
        	# 虽然 expr1 写在前面 但是还是先进行 if 判断的
            # 所以这个看起来很抽象的结构就是个 if 判断
```



[一个关于Lambda函数与递归的有趣问题 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/463842005)

```python
# From the HOF view
def make_anonymous_factorial():
    # 通过内置一个函数名称实现 
    def function(x, function):
        if(x == 1):
            return 1
        else:
            return function(x - 1, function) * x
    return lambda n: function(n, function)
# 因为出现了赋值语句 所以是无法通过 ok 测试的 但是功能还是实现了 
```

```python
# 在 HOF 的基础之上 把 def function(x,function) 改为匿名表达式的形式 
def make_anonymous_factorial():
    # 不能在 else 分支写 return 无法通过编译 
    return lambda x,mul: 1 if(x == 1) else ( mul( (sub(x,1)),x) )
	# 对原本 return statement 中的返回函数进行替换 
    # 出现的两个 function 都要进行替换 
    return lambda n: lambda x,mul: 1 if(x == 1) else ( mul( (sub(x,1)),x) )(n, lambda x,mul: 1 if(x == 1) else ( mul( (sub(x,1)),x) ))
```

看评论区中有提到 `Y-combinator` 问题 在网上搜没有搜到有明显指向的结果 待查

#### Y-combinator ?



# Done!

Q1 remain thinking~
