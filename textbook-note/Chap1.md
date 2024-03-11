# Abstract Function

```python
f = abs
g = min
x = [1,2,-1,-2]
res1 = f(g(x))
print("res1:",res1)

# res2 等效于一个函数指针
fun2 = lambda f,g: lambda x: print("fun2:",f(g(x)))
res2 = fun2(f,g)
print("res2:",res2)
print("res2_x:",res2(x))

fun4 = lambda f,g: lambda x: f(g(x))
res4 = fun4(f,g)
print("res4:",res4)
print("res4_x:",res4(x))

# 显示传参出错 传递顺序不能改变 只能由外往里面传递 
# fun5 = lambda f,g: lambda x: f(g(x))
# res5 = fun5(x)
# print("res5:",res5)
# print("res5_x:",res5(f,g))

fun3 = lambda f,g: print("fun3:",f(g(1,2,-1,-2)))
res3 = fun3(f,g)
print("res3:",res3)
```

```python
D:\Python3.9.2\python.exe D:\cs-61A\test_code\main.py 
res1: 2
res2: <function <lambda>.<locals>.<lambda> at 0x000001B01020B550>
fun2: 2
res2_x: None
res4: <function <lambda>.<locals>.<lambda> at 0x000001B01020B670>
res4_x: 2
fun3: 2
res3: None

Process finished with exit code 0
```

