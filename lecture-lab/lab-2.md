# Lab 2 HOF Lambda Self Reference

### Q1: Lambda the Free

```python
python ok -q lambda -u --local
```

#### case1

```python
Q: Which of the following statements describes a difference between a def statement
and a lambda expression?
Choose the number of the correct choice:
0) A def statement can only have one line in its body.
1) A lambda expression cannot have more than two parameters.
2) A lambda expression does not automatically bind the function object that it returns to any name.
3) A lambda expression cannot return another function.

# correct 2
```



#### case2

```python
Q: How many parameters does the following lambda expression have?
lambda a, b: c + d
Choose the number of the correct choice:
0) Not enough information
1) two
2) three
3) one

# 0	这个 c 和 d 可能外面的全局变量？
# correct 1		和内部计算过程无关
```



#### case3

```python
Q: When is the return expression of a lambda expression executed?
Choose the number of the correct choice:
0) When the lambda expression is evaluated.
1) When you pass the lambda expression into another function.
2) When the function returned by the lambda expression is called.
3) When you assign the lambda expression to a name.

# correct 2
```



#### case4

```python
>>> lambda x: x  # A lambda expression with one parameter x
# wrong x
# wrong None
# correct Function
eg. <function <lambda> at 0x00000238B988F040>
```



#### case5

```python
>>> a = lambda x: x  # Assigning a lambda function to the name a
>>> a(5)
# correct 5
```



#### case6

```python
>>> (lambda: 3)()  # Using a lambda expression as an operator in a call exp
# correct 3 没有参数只有返回值
```



#### case7

```python
>>> b = lambda x: lambda: x  # Lambdas can return other lambdas!
>>> c = b(88)
>>> c
# lambda1 接收一个参数 x Lambda2 不接受参数，返回的是父函数的参数 x 
# lambda1 返回 lambda2 这个函数
# correct Function

>>> c()
# correct 88
```



#### case8

```python
>>> d = lambda f: f(4)  # They can have functions as arguments as well
>>> def square(x):
...     return x * x
>>> d(square)
# lambda 接收一个函数 返回 f(4)
# correct 16
```



#### case9

```python
>>> x = None # remember to review the rules of WWPD given above!
>>> x
>>> lambda x: x
# Tip: None 常量不会在 console 输出任何东西 所以不存在两行输出
# 如果没有参数接收 lambda 函数的时候，它就是单纯的返回函数对象
# correct Function
```



#### case10

```python
>>> #
>>> # Pay attention to the scope of variables
>>> z = 3
>>> e = lambda x: lambda y: lambda: x + y + z
>>> e(0)(1)()
# lambda1.x = 0		lambda2.y = 1		子函数不能访问到 global var 吧...
# 好吧.. 可以的 呃
# correct 4
```

> 学到了 Python 很重要的一点，子函数，无论套了几层，都可以访问 Global Frame 中的变量
>
> C++ 不允许对函数进行嵌套



#### case11

```python
>>> f = lambda z: x + z
>>> f(3)
# Error
```



#### case12

```python
>>> # Try drawing an environment diagram if you get stuck!
>>> higher_order_lambda = lambda f: lambda x: f(x)
>>> g = lambda x: x * x
>>> higher_order_lambda(2)(g) # Which argument belongs to which function call?
# 参数传递的顺序是不是有问题
# correct Error

>>> higher_order_lambda(g)(2)
# correct 4
```



#### case13

```python
>>> call_thrice = lambda f: lambda x: f(f(f(x)))
>>> call_thrice(lambda y: y + 1)(0)
# lambda1 接受一个函数名作为参数
# lambda2 接收一个参数 嵌套调用 3 次
# correct 3
```



#### case14

```python
>>> print_lambda = lambda z: print(z)
>>> print_lambda
# correct Function
```



#### case15

```python
>>> one_thousand = print_lambda(1000)
# correct 1000

>>> one_thousand
# one_thousand = {return print(1000)}
# print 返回值是 None 实际上没有任何输出
# correct Nothing (...)难绷
```



### Q2: Higher Order Functions

```
python ok -q hof-wwpd -u --local
```

#### case1

```python
>>> def even(f):
...     def odd(x):
...         if x < 0:
...             return f(-x)
...         return f(x)
...     return odd
>>> steven = lambda x: x
>>> stewart = even(steven)
>>> stewart
# correct Function

>>> stewart(61) # 传递给 odd(x:61)
# correct 61

>>> stewart(-4)
# correct 4
```



#### case2

```python
>>> def cake():
...    print('beets')
...    def pie():
...        print('sweets')
...        return 'cake'
...    return pie
>>> chocolate = cake()
# correct beets

>>> chocolate
# Tip: 在上一次的赋值中 cake() 只传递了一个参数（虽然是空）所以 chocolate 只是重定位到 pie() 
# correct Function

>>> chocolate()	# 等效为 pie() 提供了参数
(line_1)sweets
(line_2)'cake'

>>> more_chocolate, more_cake = chocolate(), cake
# more_chocolate = chocolate()
# more_cake = cake
# correct sweets

>>> more_chocolate
# correct 'cake'

>>> def snake(x, y):
...    if cake == more_cake:
...        return chocolate
...    else:
...        return x + y
>>> snake(10, 20)
# 在上面的定义过程中 more_cake 被赋值为 cake => return chocolate
# correct Function

>>> snake(10, 20)()
# 等效为 chocolate()
(line_1)sweets
(line_2)'cake'

>>> cake = 'cake'
>>> snake(10, 20)
# cake 在重新赋值后 cake != more_cake => return (x + y)
# correct 30
```



### Q3: Lambdas and Currying

匿名表达式和函数套用

```python
python ok -q lambda_curry2 --local
```



### Q4: Count van Count

```
python ok -q count_cond --local
```

```
# 这里遇到了一点小问题 不过解决了
# 在传递循环的时候 和 调用 condition 函数进行判断的时候注意范围 循环是 n+1 函数调用是 n
```



### Q5: Both Paths !

```
python ok -q both_paths --local
```

track up and down 可能调用多次需要递归

```python
# 这个没有写出来 去看了源码
TypeError: cannot unpack non-iterable function object

# Hint 说明可以可以返回多个参数 把第二个参数作为层数标记 默认为 1 
# 如果要使用这种方法 是依赖询问顺序的 第一层的返回值实际上是 index
def up():
	print("U")
def down():
    print("D")
def sub_func(f,index = 0):  # 如果不是 0 会覆盖 
    print(sofar)
    if(index == 1):
    	index -= 1
        # 2 如果已经到了最底层 判断是 up 还是 down 
        if(f == up):
        	up()
       	else:
        	down()
    if(index > 1):
        # 1 如果没有到最底层 递归 
        return sub_func(f,index - 1)

return sub_func
```

```python
# official answer
def both_paths(sofar="S"):
    print(sofar)
    def up():
        return both_paths(sofar+'U')
    def down():
        return both_paths(sofar+'D')
    # 把 up() 和 down() 这两个子函数赋值给 Global Var 
    return up, down
```



```
# 下面是结合 Environment Diagram 的过程分析
# 1 def both_paths(sofar = 'S')

# 2 up,down = both_path()
# 这里直接跳出 both_paths() 函数 进入后面的执行 调用后面的 
```

![](D:\cs-61A\lab pic\lab2\1.png)

这里返回的是储存函数地址的元组

> 联系之前在 disscuss01 里面的问题 是可以把被嵌套的子函数赋值出去的 

![](D:\cs-61A\lab pic\lab2\2.png)

学习了... 真的好厉害

```python
# 10 upup, updown = up()
```

这里会跳过外层的父函数直接找到子函数

![](D:\cs-61A\lab pic\lab2\3.png)

鬼鬼，这也太复杂了

不过递归生效的原理应该搞懂了

```python
核心在于通过 up down / upup updown 这些中间变量，记录了当前 sofar 的内容
当调用到它们时，可以直接得到经历过递归的结果
eg. upup() -> up(SU) + U => SUU
```

#### Futher Understanding

看了一遍感觉太浅.. 等做完了 cs61A 要好好复盘的地方



### Q6: Make Adder

```python
n = 9
def make_adder(n):
    return lambda k: k + n
add_ten = make_adder(n+1)
result = add_ten(n)
```

在 `result = add_ten(n)` 的时候，创造的 `local frame` 并不是 `add_ten` 而是 `lambda` 

![](D:\cs-61A\lab pic\lab2\4.png)



### Q7: Lambda the Environment Diagram

```python
# 过程没啥问题，就是调用 return a(x + a(x)) 的时候
# 对内部函数参数的求值没有额外创建一个 local frame 有点奇怪 
```



### Q8: Composite Identity Function

```python
python ok -q composite_identity --local
```

```python
# my code here
def sub_fun(x):
    res1 = compose1(f,g)(x)
    res2 = compose2(g,f)(x)
    return (res1 == res2)
return sub_fun
```

```
# answer
return lambda x: compose1(f, g)(x) == compose1(g, f)(x)
```



### Q9: I Heard You Liked Functions...

```
python ok -q cycle --local
```

和 Both_Path 不同，这里把层数显式的给出来了

那么根据我在 Disc 02 的经验，这里要通过传递函数参数的形式来表现层数

```python
# 看了答案 可能是最近看太多子函数 def function 忘记 python 算是函数编程了
```

```python
# answer
def cycle(f1,f2,f3):
	def get_num(n):
		def get_operator(x):
            loop_time = n // 3	# 整体次数 
            inter_place = n % 3	# 进入位置 
            if(loop_time):
                for i in range(0,loop_time):
                    x = f3(f2(f1(x)))	# x = 7 loop_time + inter_place
                    if(inter_place == 0):
                        return x
                    if(inter_place == 1):
                        return f1(x)
                    if(inter_place == 2):
                        return f2(f1(x))
           return get_operator
	return get_num
```



```python
def cycle(f1, f2, f3):
    def combine(n):
        def F(x):
            p = n // 3
            q = n % 3
            for i in range(p):
                x = f3(f2(f1(x)))
            if q == 0:
                return x
            elif q == 1:
                return f1(x)
            else:
                return f2(f1(x))
        return F
    return combine
```



# Done!
