# Environment Diagram

[Python Tutor code visualizer: Visualize code in Python, JavaScript, C, C++, and Java](https://pythontutor.com/visualize.html#mode=edit)

## Test1

```
def funA(x):
	def funC(y):
		return x * 2
	return funC(x)
funB = funA
res = funB(3)
print(res)
```



## Test1-result

```python
# 1 def funA(x)
```

![](D:\cs-61A\lecture-disscus\1.png)



```python
# 2 funB = funA
```

![](D:\cs-61A\lecture-disscus\2.png)



```python
# 3 res = funB(3)
```

![](D:\cs-61A\lecture-disscus\3.png)

这里比较有意思，进行了等效替换，实际上这个参数是传给了 `funA()` 创建的 `local frame` 也是 `funA` 的局部帧



```python
# 4 def funA(x)
```

在 Diagram 上没有显示变化 只是找到了相应的 `funA()` 然后准备进入该函数



```python
# 5 def funC(y)
```

![](D:\cs-61A\lecture-disscus\5.png)

因为是 `def statement` 并不会实际执行 但是因为程序是顺序运行的，这一步会把 `function_name : funC` 加入 `funA : local frame` 中，同时也会记录该函数 `funC` 的父函数 `funA` 

注意这时就更新了 `funA` 的子函数列表

然后根据缩进跳转到下一条顺序执行语句

> When importing functions, we still create a function object in the environment diagram, bound to the name of the imported function. However, the parent and parameters of an imported function is unknown so only the function’s name is included. For example, if we imported the function add, the function object would just be add(...)
>
> 这里注释的是从包中导入函数的情况

![](D:\cs-61A\lecture-disscus\14.png)



```python
# 6 return funC(x)
```

![](D:\cs-61A\lecture-disscus\6.png)

进入 `funA` 的子函数 `funC` 构建 `local frame` 注意这里的 `parent` 并不是 `funA` 只是 `f1` 没有具体的名称，也没有 `Object` 指向，可以说现在只是保存了一个函数柄（类似于 file handle）



```python
# 7 def funC(y)
```

![](D:\cs-61A\lecture-disscus\7.png)

同样，在 `Diagram` 上没有变化，会去找 `funA : local_frame` 查看该帧内部是否有一个 `funC()` 进行执行



```python
# 8 return x * 2
```

![](D:\cs-61A\lecture-disscus\8.png)

这里的 `x` 其实不是我的本意来着233

但是也能输出我的本意说明 `parent_function : parent_para` 是可以被子函数访问的

运算的执行是在 `#5 funC()` 被定义的时候，访问 `Object` 储存的执行语句执行（猜的 

而且这里 `line_just_executed` 和 `next_line_to_exe` 指向了同一行 也就是没有更新 `next_line_to_exe` 



```
# 9 return funC(x)
```

![](D:\cs-61A\lecture-disscus\10.png)

因为 `funC()` 结束，所以 `funC : local_frame` 可以删除

`return value` 被传回父函数，也是之前学习的函数调用栈的问题，把值返回给上一个调用该函数的函数



```python
# 10 res = funB(3)
```

![](D:\cs-61A\lecture-disscus\11.png)

funA() 执行结束，也可以删除了，Object 里面的 funC() 可能要等到指向它的链接消失才会被删除吗

看来不是的，Object 中的 funC() 应该会一直存在，因为即使是子函数也是存在的，只是不能通过 Global Frame 直接索引

> Q: 那这里是如何指向的就很有意思，毕竟只能通过 A 调用 C，但是 A 这个时候没有运行，也就没有指向 funC 的能力，那 Object 中的 funC 不就孤立了吗
>
> 你总不能每次运行到 funA 的时候就创建一个 Object : funC 然后再删掉



```python
# 11 print()
```

![](D:\cs-61A\lecture-disscus\12.png)

```
??? can't understand
```



## Q: Object_funC 在结束后为什么还存在
