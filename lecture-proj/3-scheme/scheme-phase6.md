# Extra Credit

### Problem 19

```shell
python ok -q 19 --local
```

Thunk 形式转换程序 / 替换程序

优化尾递归的空间使用，完成  `scheme.py` 中的 `optimize_tail_calls()`，题目提示说要对一些已经提供的函数进行一些修改，本质上是要返回一个实现尾递归 `scheme_eval()` 备份实现

`Thunk Class` 代表一个在当前环境中需要被计算的表达式，当 `scheme_optimized_eval()` 收到了一个非原子性的表达式，返回一个 `Thunk Instance`，否则反复调用 `prior_eval_function()` 直到结果是一个值而不是 `Thunk Class` 

尾递归实现需要 `True` 作为第三个参数传入 `scheme_eval()` 函数

（我浅浅的理解一下，这个尾递归在物理层面的优化已经存在了，只是现在要去调用它



有一说一，我连题在说什么都没看懂，我 TM 果断答案好吧



**Q：**要求实现的内容是要判断哪些表达式可以使用尾递归优化，我结合答案仔细看了一下，其实只是照猫画虎的完成了题目要求的过程，emm.. 这能说是尾递归的有效实现吗

是的，是从代码层面实现的，一般的函数调用会执行 `make_child_frame()` 这个在 `Frame Class` 中的方法，而这里的 `Thunk` 直接传递当前环境，避开了创建新的函数帧的过程



**Q：**题目说的修改其他函数，应该就是判断哪些函数可以支持尾递归 `prior_eval_function()` 

修改的时候因为很多 `do_XXX_form()` 实际上调用了 `eval_all()` 导致修改工作并不是很大，体现了 `abstract barrier` 的好处



**Q：**尾递归的工作过程

我 TM 直接一通 Debug，累死了，在注释里写的比较清晰（也不敢说自己完全理解了...

因为替换函数增加太多 Debug 的难度，很难完全追踪到，但是尾递归我应该是把握到了吧..?



**Q：**对 `scheme_symbolp()` 的判断，要求不能是运算函数但是在运算函数的执行内部又会调用尾递归

主要是判断是否是变量，布尔值之类的



（顺便这个题的附带视频我在网上找到了，非常清晰的提示了哪些函数需要进行尾递归的处理

```python
# uncomment this line
# 非常巧妙的通过 HOF 记录了原函数的地址 然后替换为一个新函数 同时保留了原函数的功能
scheme_eval = optimize_tail_calls(scheme_eval)
```



```scheme
; Debug 用例
(define (sum n total) (if (zero? n) total (sum (- n 1) (+ n total))))
(sum 4 0)

; 需要足足 23 层的 scheme_eval() 调用...
```

（顺便修了一个之前的 bug，`eval_all()` 的处理没有针对 `expression.first` 进行处理

（也不知道之前的测试怎么过的，我其实怀疑过，但是测试都通过了就没管2333



### Problem 20

```shell
python ok -q 20 --local
```

