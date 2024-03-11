# Homework02



[Environment Diagrams | Albert Wu](http://albertwu.org/cs61a/notes/environments#preface-a-defense)

[Python Tutor code visualizer: Visualize code in Python, JavaScript, C, C++, and Java](https://pythontutor.com/render.html#mode=display)

### Pre

```python
from operator import add, mul, sub

square = lambda x: x * x

identity = lambda x: x

triple = lambda x: 3 * x

increment = lambda x: x + 1
```



### Q1: Product

```python
python ok -q product --local
```



### Q2: Accumulates

#### Accumulate

```python
python ok -q accumulate --local
```

注意运算顺序的影响



#### Summation_using_accumulate

```python
python ok -q summation_using_accumulate --local
```



#### product_using_accumulate

```python
python ok -q product_using_accumulate --local
```

就是根据之前的加法乘法作用到 `accumulate` 就可以



### Q3: Make Repeater

```
python ok -q make_repeater --local
```



```python
def make_repeater(func, n):
    """Return the function that computes the nth application of func.

    >>> add_three = make_repeater(increment, 3)
    >>> add_three(5)
    8
    >>> make_repeater(triple, 5)(1) # 3 * 3 * 3 * 3 * 3 * 1
    243
    >>> make_repeater(square, 2)(5) # square(square(5))
    625
    """
    
    "*** YOUR CODE HERE ***"
	# Q: 我寄了，我想不出来 主要的问题在参数接收 表面上只传递了 func 和 n 两个参数 
    # A: 额外参数需要在函数内部定义子函数来体现 
	
    # 递归的写法
    def sub_func(x):
        if n == 0:
            return x
        else:
            # 里面的嵌套不能修改 n 的值
            # return make_repeater(func,n - 1)(func(x))
            # 有点复杂.. 真的想不到 
            return func(make_repeater(func,n - 1)(x))
    return sub_func
```

```python
# 就一般而言 递归的方式是 fun(){ return fun() } 这样自己调用自己的过程 伴随着一个出递归的标志

# 函数层级关系
# 给出参数的情况 可以认为是同阶的
make_repeater
	sub_func
	make_repeater(func,n)
		func
		sub_func(x)
        	func(x)
		
# 要求 func(func(...func(x))) 从结果上看 肯定是 func() 反复调用自己
# 重点在怎么控制递归次数

# func(make_repeater(func,n - 1)(x)) 中 make_repeater(func,n - 1) == sub_func
# func(      sub_func.n - 1     (x))	因为 func 只能接受一个参数 所以通过调用上层函数传递次数 n
# func(         func               )	递归
# 直到 n == 0 return x => func 转换为具体数值 上一层的 func 可以开始计算
```



### Q4: Church numerals

#### church_to_int

```python
python ok -q church_to_int --local
```

```python
# church_to_int(one) 调用过程 假设 one = successor(zero)
church_to_int(one)	# n = one
	return one(f)(0)
	return successor(zero)(f)(0)
	return lambda f: lambda x: f(zero(f)(x)) (f)(0)	# f: x + 1	x = 0 此处传递
	# 第二个 lambda 进行求值
    return lambda f: f(zero(f)(0)) (f)(0)
	return lambda f: f(0) (f)
	# 第一个 lambda 进行求值
    return lambda f: f(0) (f)
	return 1
# 可以看出在实际调用的过程中 church_to_int 返回值的 (f)(0) 没有运算意义 只有传递参数的意义
# 关于 lambda 表达式 正向传递参数 反向求值
```



#### add_church

```python
python ok -q add_church --local
```

```python
# 目标是 m(f)( n(f)(0) ) 
return m(f)(n(f)(x))
```



```python
# 分析上面的功能 church_to_int 只是具体化函数的过程 在思考时其实是可以忽略掉的
add_church( nf(zero.x) , mf(zero.x) )
nf(zero.x) add_func mf(zero.x)
```



#### mul_church

```python
python ok -q mul_church --local
```

```python
# 每调用一次 m 求值 就遍历一整套 n 可以认为是跳着加的 f 是每次 + 1 nf(x) 每次 + n
# 目标是 m( nf(x) )(0)
return m(n(f))(x)	# + n + n + .. + n (m 次)
```

```

```



#### pow_church

```python
python ok -q pow_church --local
```

```python
# 这个估计要修改 successor 的累计方式
f = lambda x: x + x
x = 1
# m * m * m * .. * m (n 次)

# -------------------------------------- #
# 我的思路有问题...不需要修改累计方式和初始状态 我的想法问题出在过度具体化
# n1f(zero.x) pow_func n2f(zero.x)
# 再去思考具体的数学过程 指数也是一个迭代累加的过程 
# m1 * n1 * k1 => (m1 + m1 +..+ m1) = A -> A + A +..A

# 想不明白
return lambda f: n(m)(f)
```



```python
# 想明白了，要从抽象层面去思考
# n(f)(x) : 把 f 函数进行 n 次
# n(m)	  : m : 将输入的函数嵌套 m 次 当它经过 n 作用于自身的时候，总层数会不停地乘上m
			# ！= m(f)(m(f( ... ))) 这样去理解仍然是乘法 传递的是 m 整个函数
    		# 其实是两层套用(三层循环)
        		# 内层调用 f + m 每次的结果独立
            	# 外层调用 m 作用 n 次在这个结果之上
            # 刚开始调用 n 然后进入 m 
# 虽然说是搞明白了.. 但是还是有点迷茫的~
```

```python
# 用循环举个例子
def pow(down,up):
    temp = 0
    sum = down
    temp_sum = 0

    is_temp_sum = True

    for i in range(0,up):   # n 3 
        temp = 0
        is_temp_sum = True
        for j in range(1,down + 1): # m 2 
            temp_sum += temp
            if(is_temp_sum == True):
                sum += temp_sum
                is_temp_sum = False
            temp = 0
            for k in range(1,down + 1): # f 2 
                temp += 1
    return sum
res = pow(4,2)
print(res)
```

<img src="D:\cs-61A\homework pic\hw02\1.jpg" style="zoom:25%;" />



```python
# my Passed code 不优雅就是了
count = church_to_int(n)
res = m
while(count > 1):
	m = mul_church(res,m)
	count -= 1
return m
```



# Done!
