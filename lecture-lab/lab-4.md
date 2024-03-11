# Lab 4 Lists and Data Abstraction

## List

### Q1: List Indexing

```shell
python ok -q list-indexing -u --local
```

```python
# All pass
```



### Q2: Reverse (iteratively)

```shell
python ok -q reverse_iter --local
```

```python
def reverse_iter(lst):
    # 反转 list 内容 stack 
    length = len(lst)
    index = length - 1
    # 按已知列表的长度初始化 否则在下面调用的时候会出现 index error 
    # 不太方便 换一种思路 反向遍历 lst   back_list 采用 append() 方法添加元素 
    back_list = []
    for i in range(index):
        # index error 
        # back_list[index - i] = lst[i]
        back_list.append(lst[~i])
    # 我也不明白为什么要补 lst[0] 但是反正这样就成功了 
    back_list.append(lst[0])
    return back_list
# 我觉得正确答案应该不是这样的 23333
```



### Q3: Reverse (recursively)

```shell
python ok -q reverse_recursive --local
```

```python
# 只是思路上有感觉 但是具体如何实现并不知道
def reverse_recursive(lst):
    # 递归式调用 思路上应该是每次少传递一个元素 
    # if(len(lst) == 1):
    #     return lst[0]
    # else:
    #     rest_list = lst[1:]
    #     return reverse_recursive(rest_list)
    if not len(lst):
        return []
    # 列表结构相加 ? 语法盲区 
    # 因为要逆序输出 先取出最后一个元素 我的问题就是在这里 先取出第一个元素导致相反 
    # 再把前面的剩余元素加到 new_list
    return [lst[-1]] + reverse_recursive(lst[: -1])

```



```shell
>>> a = [1,2]
>>> b = [3,4]
>>> a + b
[1, 2, 3, 4]

>>> a = [1,2,3,4]
>>> b = a[:-1]
>>> b
[1, 2, 3]
```



## City Data Abstract

### Q4: Distance

```shell
python ok -q distance --local
```



### Q5: Closer city

```shell
python ok -q closer_city --local
```



### Q6: Don't violate the abstraction barrier!

```shell
python ok -q check_abstraction --local
```



### Q7: Add Characters

```shell
python ok -q add_chars --local
```



# Done!

这个倒是不难

