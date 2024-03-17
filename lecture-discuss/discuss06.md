# Nonlocal, Midterm Review

## Nonlocal

### 1.2

```python
def memory(n):
    def apply_function(f):
        nonlocal n
        n = f(n)
        return n
    return apply_function

f = memory(10)
print(f(lambda x: x * 2))
print(f(lambda x: x - 7))
print(f(lambda x: x > 5))
```



## Midterm Review

### 2.2

我想不到，我想不到啊啊啊啊啊

找到答案了，但是只找到了这一道题的

```python
def nonlocalist():
    get = lambda x: "Index out of range"    # 为什么会接收一个参数呢
    def prepend(value):
        nonlocal get	# 看到这步 小脑萎缩了 做不出来是有原因的
        f = get
        def get(i):
            if i == 0:
                return value
            return f(i - 1)
    return prepend,lambda x: get(x)

prepend, get = nonlocalist()
prepend(2)
prepend(3)
prepend(4)

print(get(0))   # 4
print(get(1))   # 3
print(get(2))   # 2

prepend(8)
print(get(2))   # 2
```

超出了我的理解能力....我不知道



### 2.3

我觉得是匿名函数递归调用的问题，我先去查查，我记得类似的问题之前出过，搜过的 `Y-combinator` 问题

通过改写为 HOF 传参的方式来实现匿名函数的递归 https://zhuanlan.zhihu.com/p/661364887 

从显式的函数递归视角来看

```python
def F(n):
    return 1 if n == 0 else n * F(n - 1)
# 从实现的角度看 把该函数地址以形参的放入函数内部 到执行的时候 函数会自动调用

# 最重要的转换!
def F(f, n):
    # f: F 的形参
    return 1 if n == 0 else n * f(f, n - 1)

F(F, 4)	=> 24

# 通过 HOF 的方式来简化 F(f, n) 中的参数 f
def Higher_Func(f):
    return lambda n: f(f,n) # 返回了 F 函数本身
F_ = Higher_Func(F) # F_ = [return 1 if n == 0 else n * f(f, n - 1)]
print(F_(4))    # 24
```

#### 匿名化改造

```python
def F(f, n):
    return 1 if n == 0 else n * f(f, n - 1)

=> lambda f,n: 1 if n == 0 else n*f(f,n-1)	# lambda_1

def Higher_Func(f):
    return lambda n: f(f, n)

=> lambda f: lambda n: f(f, n)	# lambda_2

# 因为 lambda_2 执行的就是 lambda_1 本身 所以用 lambda_1 对 f(f, n) 进行替换
=> lambda f: lambda n: 1 if n == 0 else n *f(f,n-1)	# lambda_x
# 但是这么写肯定是不对的 因为参数是分层调用的 而 f(f, n-1) 在同一层调用的根据传参顺序拆开
=> lambda f: lambda n: 1 if n == 0 else n * f(f)(n - 1)	# lambda_3

# eg. Higher_Func(F)(4)

# 现在的问题变成如何调用 lambda_3	向形参 f 传递的实参应该是匿名函数本身
lambda f: f(f)	# 在运行的时候接受一个函数体然后自我调用
# 额外的问题 这个写法是解释型语言独有的吗

# 把 lambda_3 函数体作为参数传递给形参 f 
(lambda f: f(f))(lambda f: lambda n: 1 if n == 0 else n*f(f)(n-1))
```

#### 核心：传递的是函数体本身

```python
F = [lambda f: lambda n: 1 if n == 0 else f(f)(n - 1)]
		    func_body = [1 if n == 0 else f(f)(n - 1)]
lambda f: f(f)	# 规定了 F 自己调用自己的递归方式
```

虽然这道题我还是不知道怎么做，但是这个匿名函数的递归调用我好像懂了.... 虽然现在脑子有点疼×





### 2.4

我连规则都没看太明白，我不明白为什么最后的 `f = f(square)` 会有 `None` 的输出



### 2.5



### 2.6



