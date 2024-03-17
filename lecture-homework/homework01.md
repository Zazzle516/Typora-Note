# Homework 01

### Q1:Quiz

None of my business

### Q2:A Plus Abs B

```python
cd hw01
python ok -q a_plus_abs_b --local
```



```python
# my code here
if b >= 0:
	h = add(a,b) # a + b 类似的写法
...
return h(a,b)
```



```python
# 无法通过编译 会有以下报错
TypeError: 'int' object is not callable

# 变量名和函数名重复的时候发生 也就是 h 
'''
	我的写法让 h 接收了 (a + b) 返回的数据结果 然而在 return 中是以函数的方式调用的
'''
```



```python
# correct answer
# 保持 h 作为函数的意义 对 (a,b) 使用不同的函数
def a_plus_abs_b:
    if(b >= 0)
    	h = add
    else
    	h = sub
    return h(a,b)
# 看了答案才意识到 真的是很新颖的想法 不去操作数据而是操作函数 
# 感觉 C++ 的函数指针也有这样的思想在里面 
```



### Q3:Two of Three

```
python ok -q two_of_three --local
```



```
# min() 与 max() 可以接受多参数 
在选出最小的数字后的次小数字怎么选 
直接去掉重复数值不满足，有相同最小值的情况

# emm.. 虽然过了 但是觉得有点点丑陋
# 判定条件 inspect 要求只用一行 return
Error expected
	['Expr', 'Return']
but got
	['..',...]
# 只要用超过一行的语句完成就会报错... 
return add(min([x, y], [x, z], [y, z])[0]*min([x, y], [x, z], [y, z])[0],min([x, y], [x, z], [y, z])[1]*min([x, y], [x, z], [y, z])[1])
```

### Q4:Largest Factor

```python
python ok -q largest_factor --local
```

```python
# my code here
# try time 0
factors = []
index = 1
for index in range(x):
	if(x % index == 0):
		factors.append(index)

return factors[-1]
# Error
ZeroDivisionError: integer division or modulo by zero
# Analisy
# 其实是想到了除0错误 以为在前面定义 index = 1 就可以了 但是还是不行 对 python 语法不熟
```

```python
# my code here
# try time 1
factors = []
for index in range(2,x):
	if(x % index == 0):
		factors.append(index)

return factors[-1]
# 在质数的时候没有考虑到数组越界的问题 而且也没有注意审题 要从 1 开始判断
```



### Q5:If Function vs Statement

```python
python ok -q with_if_statement --local
python ok -q with_if_function --local
```

```
# python ok -q with_if_statement --local
42 42 47
```

![](D:\cs-61A\lab pic\lab0\hw01-2.png)

```
# my code here 
# 直接贴出正确的 主要是 cond() 不需要通过 print() 返回 None
# 直接进入 false 子分支就可以
```



### Q6:Hailstone

```python
python ok -q hailstone --local
```

```python
# my code here
# 这个很简单 注意输出格式 直接输出的话 因为有取余判断 x 会被转化为浮点数
print('%d' %x)
```

# Done!