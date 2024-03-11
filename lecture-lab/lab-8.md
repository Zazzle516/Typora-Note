# Lab 8 Iterator Generator OOP

## Iterators and Generators

没想到这里才正式介绍 Iterators 那我做的 homework5 是什么😭

#### Iterators

对可迭代对象使用 `__iter__()` 方法生成迭代器，迭代器使用该方法会返回它自己



#### Generators

Calling a generator function will return a generator object and will not execute the body of the function 说明了返回值类型和生成器惰性求值的特点

generator is iterators 生成器属于迭代器

就像我在 homework05 学到的 `next()` 并不是真的函数调用，不涉及参数，只是让程序继续运行

A generator function has a yield statement and returns a generator object

```python
# new knowledge
# 不同: 这里直接传 iterable 或者 iterator 都可以
	  # 这里不用指定 iterable 的位置 普通的 yield 只能返回元素
def gen_list(lst):
    yield from lst
    
>>> gen = gen_list([1, 2])
>>> next(gen)	# 1
>>> next(gen)	# 2
```



#### OOP

这个接触很久了，没什么好说的



### Q1: WWPD: Iterators

```shell
python ok -q iterators -u --local
```

```shell
# 可能的结果
[Error] [StopIteration] [Iterator]
```

对 `Iterable` 调用 `next()` 会发生 `Error` 

```python
map_iter = map(lambda x : x + 10, range(5))
next(map_iter)

# correct 10
# 注意 map() 函数是在应用函数而不是字典	Python 的字典结构是 dict()
# 从 range(5) 中枚举变量应用在匿名函数中
```

`map()` 根据提供的函数对指定序列进行映射，`map(function, iterable, ...)` 

```python
def square(x):
    return x ** 2
map(square, [1,2])
>>> [1, 4]
```



### Q2: Generators generator

```shell
python ok -q make_generators_generator --local
```

测试用例

```python
>>> def every_m_ints_to(n, m):
...     i = 0
...     while (i <= n):
...         yield i
...         i += m
...
>>> def every_3_ints_to_10():
...     for item in every_m_ints_to(10, 3):
...         yield item
...
>>> for gen in make_generators_generator(every_3_ints_to_10):
...     print("Next Generator:")
...     for item in gen:
...         print(item)
...

# expected
Next Generator:
0
Next Generator:
0
3
Next Generator:
0
3
6
Next Generator:
0
3
6
9
```



```python
# my code
def make_generators_generator(g):
    # 主函数每次都要返回一个新的生成器 更新自己的状态 每次叠加一次 next()
    while True:
        yield iter(g())

# 会发生无限循环的错误 猜测是 while True 导致的
{
    Next Generator:
    0
    3
    6
    9
}loop

# 删掉 while True 可以改掉无限循环的问题 但是解决这个问题是一定要用到循环的
# 从实现思路上考虑 应该是有两层循环	外面的 yield 对应 for item in gen ; 内部的 yield 输出

# 主函数每次都要返回一个新的生成器 更新自己的状态 每次叠加一次 next()
def make_generators_generator(g):
    while True:
        yield next(g())
        def sub():
            while True:
                yield from iter(g())
        yield sub()
```

看了答案有点震撼了，我的思路不说十分贴切吧也可以说是毫不相关

逛了一圈，大部分生成器相关的题都是需要一个 `gen_helper()` 函数的

```python
def make_generators_generator(g):
    def gen_helper(num):
        # 我一开始想过传参数 但是没有找到传参数的位置 ... 
    	gen=g()		# creat a new generator
        for i in range(num):
            yield(next(gen))	# 直接去执行最内层的函数
    count=len(list(g()))	# 通过 list() 规避惰性求值 运行 g() 得到外层循环次数 eg.count = 4
    for i in range(count):
        # 通过 gen_helper() 提供内层循环次数
        yield(gen_helper(i+1))
```

不过确实是两个循环，只是有限循环，我之所以用 `loop-true` 是因为我没想到通过 `list()` 的方式强制求出外层循环次数
