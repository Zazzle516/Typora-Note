# Higher Order Function

### 1.4 "Hi"

```python
y = "y"
h = y

def y(y):
    h = "h"     # 为什么这里的修改可以执行 
    if y == h:
        return y + "i"
    y = lambda y: y(h)
    return lambda h: y(h)
    
y = y(y)(y)
print(y)
```



```python
# 在局部函数帧中修改全局变量
# 使用同一个名字 会在局部函数帧中创造一个副本 然后该副本拥有新值
# 不过 注意不能在赋值副本前访问全局函数的变量
Error: local var x referenced before assignment
# 直接去修改是可以执行的 

x = "x"
h = x
# 只要不去提前访问全局变量 就可以在局部函数的帧中声明一个全局变量的副本 
def fun():
    # print(h)	# Error
    h = "not x"
    print(h)

y = fun()
# not x
```



### 1.5

```python
def keep_ints(cond, n):
    """Print out all integers 1..i..n where cond(i) is True

    >>> def is_even(x):
    ...     # Even numbers have remainder 0 when divided by 2
    ...     return x % 2 == 0
    >>> keep_ints(is_even, 5)
    2
    4

    """
    # my code here
    def get_cond_res(x):
        return cond(x)
    
    for i in range(1, n + 1):
        if(get_cond_res(i)):
            print(i)
        else:
            continue
if __name__ == '__main__':
    doctest.testmod()
```

