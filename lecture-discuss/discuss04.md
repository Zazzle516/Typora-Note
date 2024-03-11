# Recursion

### 2.2 Step

```python
def count_k(n, k):
    # 我没太懂.. 
    
    if n == 0 or n == 1:
        return 1
    elif k == 1:
        return 1
    elif k >= n: 
        return 2 * count_k(n - 1, n)
    else:
        return 2 * count_k(n - 1, k) - count_k(n - 1 - k, k)
```

```
https://www.cnblogs.com/evenleee/p/8474465.html
```

```
https://www.soinside.com/question/6nVeLPoJ8pw7S89DaPzNoY
```



## List

### List Slicing

获取列表数据的一部分， new_list = old_list[starting index, ending index]

`list[m:n:k]` 注意 step/k 在作用的时候，step = 2 是跳过 1 个元素，index += 2 

left out 就是缺省 Python 会以默认状态处理这条语句的缺省部分



### List Comprehensions

对列表的元素应用一个函数后的结果作为一个新列表输出

```python
# list comprehension => a compact way 简洁的方式 
# 通过固定的表达方式计算得到一个 list 
[<map exp> for <name> in <iter exp> if <filter exp>]

[x * x - 3 for x in [1,2,3,4,5] if x % 2 == 1]	# note that the <if> is optional

# to Expression 
list = []
for x in [1,2,3,4,5]:
	if x % 2 == 1:
        list + (x * x - 3)
```

![](D:\cs-61A\lecture-disscus\15.png)

判断元素有没有存储在列表中

只能判断到一级，内部嵌套的列表是无法检测到的

```python
a = [1,[2,3]]
2 in a
False
```

这里面讨论的例题挺难的... 没有一个想出来的



### Hole Number

