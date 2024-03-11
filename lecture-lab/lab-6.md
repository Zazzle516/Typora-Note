# Lab 6 Nonlocal, Mutability

## Nonlocal

如果想在子函数内部修改父函数内部的变量，需要声明该变量未 `Nonlocal` 变量。

因为在直接调用子函数的时候，虽然仍在父函数的帧内，但是变量不在子函数的变量帧内，所以会有经典错误。

```
UnboundLocalError: local variable 'X' referenced before assignment
```



### Q1: Make Adder Increasing

```shell
python ok -q make_adder_inc --local
```



### Q2: Next Fibonacci

```shell
python ok -q make_fib --local
```



## Mutability

### Q3: List-Mutation

```shell
python ok -q list-mutation -u --local
```

```python
>>> lst = [5, 6, 7, 8]
>>> lst.append(6)

# corrent: Nothing
```



### Q4: Insert Items

```shell
python ok -q insert_items --local
```



# Done!

这节的 lab 蛮简单的
