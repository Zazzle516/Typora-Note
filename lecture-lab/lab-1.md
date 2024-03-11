# Lab 1 Variables & Functions

**What would python display** 

### Q1: Control

没有写在 `lab01.zip` 中，是在 `shell` 以命令行的方式进行回答的

```python
python ok -q control -u --local
```

```python
>>> def how_big(x):
...     if x > 10:
...         print('huge')
...     elif x > 5:
...         return 'big'
			# return "big"
...     elif x > 0:
...         print('small')
...     else:
...         print("nothin'")
>>> how_big(7)
# correct 'big' 外面需要单引号 

>>> how_big(12)
# correct huge 不需要单引号

>>> how_big(1)
# correct small

>>> how_big(-1)
# correct nothin'
```

```python
# 注意 print('') print("")	对两种符号均不处理
# 直接 return str	是不包括单引号和双引号的
```

```python
>>> positive = -9
>>> negative = -12
>>> while negative: # If this loops forever, just type Infinite Loop
...    if positive:
...        print(negative)
...    positive += 3
...    negative += 3
(line 1)? negative
-- Not quite. Try again! --

(line 1)?-12
```



### Q2: Veritasiness

```python
python ok -q short-circuit -u --local
```

```python
>>> True and 13
13
>>> not 10
False
>>> True and 0
0
>>>  1 and 3 and 6 and 10 and 15
15
>>> 0 or False or 2 or 1 / 0
2
```



### Q3: Debugging Quiz

```python
python ok -q debugging-quiz -u --local
```

[Debugging | CS 61A Summer 2020 (berkeley.edu)](https://inst.eecs.berkeley.edu/~cs61a/su20/articles/debugging.html)

> Traceback back
>
> prints out the chain of function calls that led up to the error, with the most recent function call at the bottom

```python
# first line
File "<file name>", line <number>, in <function>

# second line
<error type>: <error message>
# displays the actual line of code that makes the next function call
```

```python
Q: In the following traceback, what is the most recent function call?
Traceback (most recent call last):
    File "temp.py", line 10, in <module>
      f("hi")
    File "temp.py", line 2, in f
      return g(x + x, x)
    File "temp.py", line 5, in g
      return h(x + y * 5)
    File "temp.py", line 8, in h
      return x + 0
  TypeError: must be str, not int
Choose the number of the correct choice:
0) g(x + x, x)
1) h(x + y * 5)
2) f("hi")

# 追溯到的最近的错误位置 也就是错误信息的最底层显示 
module
	f("hi")
    	g(x + x ,x)
        	h(x + y * 5)	# The most recent function call
            	x + 0
```

```python
Q: In the following traceback, what is the cause of this error?
Traceback (most recent call last):
    File "temp.py", line 10, in <module>
      f("hi")
    File "temp.py", line 2, in f
      return g(x + x, x)
    File "temp.py", line 5, in g
      return h(x + y * 5)
    File "temp.py", line 8, in h
      return x + 0
  TypeError: must be str, not int
Choose the number of the correct choice:
0) the code looped infinitely
1) there was a missing return statement
2) the code attempted to add a string to an integer

# 2 错误类型 要求 str 而不是 int
```

#### Use Tests

###### Error type

```python
SyntaxError
	only raised during runtime, '^'tell you where the error is
    
IndentationError
	improper indentation 缩进错误
    # 如果一开始用 tab 就一直用 tab cs61A 最好用 space 
    
TypeError
	eg. string + int
    eg. >>>var = 3 >>>var(3)	# 把变量作为函数调用 

NameError
	# 未定义变量名
    
IndexError
	# 大部分出现在数组中 越界情况
```

###### Common mistakes

```
Spelling

= vs. ==

Infinite Loops
```



##### Quick Debugging

Two ways to write my own tests

1. write additional doctests
2. write testing functions

针对第一种方式，参考已经存在的 `doctest` 即可

第二种方式，参考 `project1:take_turn_test` 即可（现在还没有做到，后续补上）

```
Write some tests before write code
Write more tests after write code
Test edge cases
```



```python
# 如何及时的定位错误(或者在某种程度上忽略错误让程序先运行下去)
# print statement
print('DEGUG: result is',result)

# prefixing debug statements with the specific string "DEBUG: " allows them to be ignored by the ok autograder used by cs61a
```

##### Long term Debugging

After figuring out the error, you would remove all the `print` statements

但是如果想周期性的去测试的话，可能会在文件内部留一些测试点，但是并不是每次测试都需要这些测试点，如果想忽略它们，可以使用一个全局的 `debug variable` 进行处理

```python
debug = True
# debug = False

def foo(n):
i = 0
while i < n:
    i += func(i)
    if debug:
        print('DEBUG: i is', i)		# if False continue
```



##### Interactive Debugging

```python
# a terminal can directly run functions and inspect their outputs
python -i file.py
```

```python
# with ok autograder enables you to jump into the middle of a failing test case
python ok -q <question name> -i
```

##### Python Tuor Debugging

```python
# with a given piece of python code is to create an environment diagram
PythonTutor
# creates environment diagrams automatically

# can also work with the ok autograder
python ok -q <question name> --trace
```

##### Using assert statement

remember that `assert` != debug

One major benefit of assert statements is that they are **more than a debugging tool** , you can **leave them in code permanantly** 

#### Shell quiz

```python
Q: How do you write a doctest asserting that square(2) == 4?
Choose the number of the correct choice:
0) def square(x):
       '''
       square(2)
       4
       '''
       return x * x
1) def square(x):
       '''
       input: 2
       output: 4
       '''
       return x * x
2) def square(x):		# correct
       '''
       >>> square(2)
       4
       '''
       return x * x
3) def square(x):
       '''
       doctest: (2, 4)
       '''
       return x * x
```

```python
Q: When should you use print statements?
Choose the number of the correct choice:
0) To investigate the values of variables at certain points in your code
1) For permanant debugging so you can have long term confidence in your code
2) To ensure that certain conditions are true at certain points in your code

# correct 0
```

```python
Q: How do you prevent the ok autograder from interpreting print statements as output?
Choose the number of the correct choice:
0) You don't need to do anything, ok only looks at returned values, not printed values
1) Print with # at the front of the outputted line
2) Print with 'DEBUG:' at the front of the outputted line

# correct 2
```

```python
Q: What is the best way to open an interactive terminal to investigate a failing test for question sum_digits in assignment lab01?
Choose the number of the correct choice:
0) python3 ok -q sum_digits --trace
1) python3 -i lab01.py
2) python3 ok -q sum_digits -i
3) python3 ok -q sum_digits

# correct 2
```

```python
Q: What is the best way to look at an environment diagram to investigate a failing test for question sum_digits in assignment lab01?
Choose the number of the correct choice:
0) python3 ok -q sum_digits
1) python3 ok -q sum_digits --trace
2) python3 ok -q sum_digits -i
3) python3 -i lab01.py

# correct 1
```

```python
# 选择不正确的
Q: Which of the following is NOT true?
0) It is generally bad practice to release code with debugging print statements left in
1) Debugging is not a substitute for testing
2) It is generally good practice to release code with assertion statements left in
3) Code that returns a wrong answer instead of crashing is generally better as it at least works fine
4) Testing is very important to ensure robust code

# correct 3
# 只要不是目标值，无论有没有跑起来都是一样垃圾，好吧，好残忍
```

```python
Q: You get a SyntaxError. What is most likely to have happened?
Choose the number of the correct choice:
0) You typed a variable name incorrectly	# NameError
1) You forgot a return statement	# Wrong
2) Your indentation mixed tabs and spaces	# IndentationError
3) You had an unmatched parenthesis		# SyntaxError
```

```python
Q: You get a IndentationError. What is most likely to have happened?
Choose the number of the correct choice:
0) You had an unmatched parenthesis
1) You typed a variable name incorrectly
2) Your indentation mixed tabs and spaces	# correct
3) You forgot a return statement
```

```python
Q: You get a TypeError: blah blah blah NoneType blah blah blah. What is most likely to have happened?
Choose the number of the correct choice:
0) You typed a variable name incorrectly
1) Your indentation mixed tabs and spaces
2) You had an unmatched parenthesis
3) You forgot a return statement	# correct
```

```
Q: You get a NameError. What is most likely to have happened?
Choose the number of the correct choice:
0) You had an unmatched parenthesis
1) You typed a variable name incorrectly
2) You forgot a return statement
3) Your indentation mixed tabs and spaces
```



### Q4: Falling Factorial

```
python ok -q falling --local
```



### Q5: Sum Digits

```
python ok -q sum_digits --local
```



### Q6: What If?

```python
python ok -q if-statements -u --local
```

```python
>>> def ab(c, d):
...     if c > 5:
...         print(c)	# 运行一次后直接跳出
...     elif c > 7:
...         print(d)
...     print('foo')
>>> ab(10, 20)
(line 1)? 10
(line 2)? foo
```

```python
>>> def bake(cake, make):
...    if cake == 0:
...        cake = cake + 1
...        print(cake)
...    if cake == 1:
...        print(make)
...    else:
...        return cake
...    return make
>>> bake(1, "mashed potatoes")
(line 1)? mashed potatoes
(line 2)? "mashed potatoes"

# 返回值是参数 是 str 类型，包含外面的双引号
```



### Q7: Double Eights

```python
python ok -q double_eights --local
```

```python
# 要求有 2 个连续的 8 而不是总数
def double_eights(n):
    if(n < 10):
        return False
    list = []
    while(n > 0):
        list.append(n % 10)
        n = n // 10
        
    for i in range(len(list)):
        if(list[i] == 8):
            if( (i + 1) >= len(list)):
                return False
            if(list[i+1] == 8):
                return True
            i += 2
    return False

# 有点丑陋说实话 最后的 False 不能去掉 在数列没有一个 8 的时候，要通过这里返回
```



# Done!
