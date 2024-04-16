# Lab 10: Scheme, Scheme Lists

#### Scheme

主要是一些语法和程序编写原理，在 `./textbook-note/Chap3.md` 中详细介绍

在线编译 [61A Code (cs61a.org)](https://code.cs61a.org/) 

本地编译 `python scheme` 



### Q1: WWSD: Lists

```shell
python ok -q wwsd_lists -u --local
```

```scheme
scm> (cons 1 '(list 2 3))
(1 list 2 3)	; 内部显示不包含括号（）

scm> '(cons 4 (cons (cons 6 8) ()))
(cons 4 (cons (cons 6 8) ()))
```



### Q2: Over or Under

```shell
python ok -q over_or_under -u --local
```

```shell
python ok -q over_or_under --local
```



### Q3: Filter Lst

```shell
python ok -q filter_lst -u --local
```

实现类似于 `filter()` 的函数，题目给出的就是 HOF 的实现方式

因为涉及到遍历列表中的所有元素，应该是用递归实现的，从后往前，把新的元素通过 `car` 传入新的列表中

```python
def filter_lst(fn, lst):
    result_lst = []
    if len(lst) == 0:
        return
    if fn(lst[0]):
        result_lst.append(lst[0])
    result_lst.append(filter_lst(fn, lst[1:]))
    return result_lst
```

python 还是很简单的，scheme 就.... 不是很会啊，还是语法不熟悉，看一看

```scheme
(define (filter-lst fn lst)
  ; 判断是否为空 空-结束递归  非空-继续
	(if (null? lst) nil
    ; 判断 fn(lst) is True
		(if (fn (car lst))
      ; 利用 cons 构造 return-list
			(cons (car lst) (filter-lst fn (cdr lst)))  ; 满足
			(filter-lst fn (cdr lst))   ; 不满足
		)
	)
)
```

scheme 写递归好简洁啊... 👾（果然菜的只是我而已，只能说递归思路确实和 python 一样



看完了之后自己写了一遍，判断为空用 `null`，还有获取元素的 `(car lst)` 和 `(cdr lst)` 括号是写在外面的

```shell
python ok -q filter_lst --local
```



### Q4: Make Adder

```shell
python ok -q make_adder -u --local
```

同样是 HOF 的结构，要求返回一个函数，子函数有一个输入参数的要求，返回两个参数的和

scheme 默认返回最后一个括号的内容

```python
def make_adder(num):
    def sub_func(inc):
        return num + inc
    return sub_func
```



Q：如何返回一个子函数

写不加括号的函数名返回

```scheme
(define (make-adder num)
  'YOUR-CODE-HERE
  (define (adder inc)
    (+ num inc)
  )
  adder
)
```

```shell
python ok -q make_adder --local
```



### Q5: Make a List

```shell
python ok -q make_structure -u --local
```

~~构造一个只能有两个元素的列表~~ 

```shell
scm> (define c (list 3 b))
c

scm> c
(3 (2 1))

scm> (car c)
3

scm> (cdr c)
((2 1))

scm> (cdr (car (cdr c)))
(1)

scm> 'How would scheme display the list of that graph'
((1) 2 (3 4) 5)
```



![](./../../../cs-61A/lecture-lab/lab10/pics/list2.png)

我最开始没有理解这个 list 是如何构造的，所以最后一个问答我去看了答案

它这个意思应该是说，上面那行是主列表，但凡是通过指针指向的都是嵌套进去的，我一开始错误的理解为是一个只能包含两个元素的列表

运行的时候把 `'YOUR-CODE-HERE` 删掉，否则无法通过测试

```shell
python ok -q make_structure --local
```



### Q6: Compose

```shell
python ok -q composed -u --local
```

传入两个函数，返回一个子函数，子函数接收一个参数，返回 `f(g(x))` 

```python
def composed(f, g):
    def sub(x):
        return f(g(x))
    return sub
```

```shell
python ok -q composed --local
```



### Q7: Remove

```shell
python ok -q remove -u --local
```

从给给定的 lst 中移除特定元素，返回新的 lst，可以使用前面完成的 `filter-lst` 给定的列表不会有除了数字以外的元素，并且不会嵌套

```shell
python ok -q remove --local
```



### Q8: No Repeats

```shell
python ok -q no_repeats -u --local
```

求出列表中非重复元素，按照最开始出现的顺序输出，判断相等 `=`，判断不等 `not =` 

Python 本身是可以通过一个二重循环实现，但是 scheme 这里只能用递归

```python
def no_repeats(s):
    if s is None:
        return
    return no_repeats(filter(s[0], s[1:]))
```

我一开始想着怎么利用 `filter-lst` 函数简化这个问题的调用，但是越想越复杂，看了看答案，果然没用

（没想起用上面写的 remove 我真是小丑🤡

```scheme
(define (no-repeats s)
	(if (null? s) nil	; 注意空列表返回的是 nil
        ; 删掉重复元素 构造新列表
        ; 这里的技巧和之前在练习题里看到的一样 在列表内部递归 实现顺序的输出
		(cons (car s) (no-repeats (remove (car s) (cdr s)) ))
	)
)
```



```shell
python ok -q no_repeats --local
```



### Q9: Substitute

```shell
python ok -q substitute -u --local
```

把列表中的特定元素值替换为新值，传入的列表会有子列表嵌套的情况，提示用 `pair?` 判断嵌套情况

```scheme
(define (substitute s old new)
  'YOUR-CODE-HERE
  (if (null? s)
    nil
    (if (pair? (car s))
      (cons (substitute (car s) old new) nil)	; 这里出的问题
      (if (eq? old (car s))
        (cons new (substitute (cdr s) old new))
        (cons (car s) (substitute (cdr s) old new))
      )
    )
  )
)

scm> (substitute '(c a b) 'b 'l)
(c a l)
scm> (substitute '(f e a r s) 'f 'b)
(b e a r s)
scm> (substitute '(g (o) o (o)) 'o 'r)
(g (r))

# Error: expected
#     (g (r) r (r))
# but got
#     (g (r))

; 问题在没有保证原列表的结构 没有对 (cdr s) 进行处理 加上就好了
```



```shell
python ok -q substitute --local
```



### Q10: Sub All

```shell
python ok -q sub_all -u --local
```

接收两个长度相等的字符串，提示会用到 `substitute()` 函数，balabala，我没看懂这个题目在说啥

```scheme
scm> (load-all ".")
scm> (sub-all '(go ((bears))) '(go bears) '(big game))
(big ((game)))
```

从这个提示看，应该是说，在 `s` 中涉及到的词，如果 `old` 中有相同的值，那么用相应位置的 `new` 进行替换

```scheme
(define (sub-all s olds news)
  'YOUR-CODE-HERE
  (if (null? s)
    nil
    (cons (substitute (car s) (car olds) (car news)) (sub-all (cdr s) (cdr olds) (cdr news)))
  )
)

; 显示语法不对 问题就是 scheme 我不太会 debug 所以出了点问题没思路的话就只能去看答案...
```



```scheme
(define (sub-all s olds news)
  'YOUR-CODE-HERE
  (if (null? s)
    nil
    (if (null? olds)
      s
      (sub-all (substitute s (car olds) (car news)) (cdr olds) (cdr news))
    )
  )
)
; 对比答案 思路上是差不多的 忘了 substitute 在内部已经构造列表了 不用在 sub-all 再构建了
```



```shell
python ok -q sub_all --local
```



# Done!

还行，初接触 scheme，括号太多啦，语法还不是很熟悉
