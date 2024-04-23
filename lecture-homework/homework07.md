# Homework 07

## Scheme

### Q1: Thane of Cadr

```shell
python ok -q cadr-caddr -u --local
```

```shell
python ok -q cadr-caddr --local
```



### Q2: Sign

```shell
python ok -q sign -u --local
```

使用条件 `cond` 表达式，三元

```shell
python ok -q sign --local
```



### Q3: Pow

```shell
python ok -q pow -u --local
```

递归实现乘方运算，根据奇偶有不同的执行方式，要求是 `O(log)` 级别的运算量，这个其实是书上例题了

```shell
python ok -q pow --local
```

```scheme
(define (square x) (* x x))

(define (pow x y)
  'YOUR-CODE-HERE
  (if (= 1 y)
    x
  )
  (if (even? y)
    (square (pow x (/ y 2)))
    (* x (pow x (- y 1)))
  )
)

; RecursionError: maximum recursion depth exceeded in comparison 不明白为什么...
```



明白了，被 Python 带歪了，注意分支写的位置，艹了

```scheme
(define (pow x y)
  'YOUR-CODE-HERE
  (if (= 1 y)
    x
    (if (even? y)
      (square (pow x (/ y 2)))
      (* x (pow x (- y 1)))
    )
  )
)
```



### Q4: Unique

```shell
python ok -q unique --local
```

利用内置的 `filter()` 函数删掉 `list` 中的重复元素，注意 `filter()` 里面接收的自定义函数 `<pred>` 只能接收一个参数

因为是单参数，所以必然有调用外层函数的语句

```python
def unique(s):
    def delete_same(item):
        if item in s:
            return False
        return True
    filter(delete_same, s)
```

和在 lab10 写的还不一样啊，有点想不出来呢（虽然我觉得我这种思路可以	**不可以×**

```scheme
(define (unique s)
  'YOUR-CODE-HERE

  (define (find-item item s)
    (if (null? s)
      #t
      (if (eq? item (car s))
        #f
        (find-item item (cdr s))
      )
    )
  )

  (define (delete-same item)
    (if (find-item item s)
      #t
      #f
    )
  )
  
  (filter delete-same s)
)
```



Q：筛选重复元素有个问题，如果已经重复了，按照这种思路即使实现了，元素也没办法被添加到新列表中

我能给出的一个解决想法就是定义一个新的空列表，没有出现的元素写入新列表，但是这好像和题目原本的目的相悖



又改了一通，实现是能实现，但是返回顺序有问题😭

```scheme
(define (unique s)
  'YOUR-CODE-HERE

  (define (find-item item sublst)
    (if (null? sublst)
      #f
      (if (eq? item (car sublst))
        #t
        (find-item item (cdr sublst))
      )
    )
  )

  (define (repeated x blst)
    (if (find-item x blst)
      blst
      ; (cons x blst)		; 反向的返回顺序应该是这里有问题 但是我改过来就报错.. 无语
      (append blst (list x))	; 感谢 GPT 改成这样就行了
    )
  )

  (define (creat-new-list s alst)
    (if (null? s)
      alst
      (creat-new-list (cdr s) (repeated (car s) alst))
    )
  )

  (define lst '())

  (creat-new-list s lst)
)
```



## Tail Recursion

### Q5: Replicate

```shell
python ok -q replicate --local
```

用尾递归的方式重写 `repeated`，其实就是套一层 `helper-function` 把元素改成 `lst` 

这个就能过，凭什么没有 discussion 的答案(///￣皿￣)○～



### Q6: Accumulate

```shell
python ok -q accumulate --local
```

给出一个自定义函数，计算 start 与自然数列截至 n 的累计和

```scheme
scm> (accumulate + 0 5 square)  ; 0 + (1^2 + 2^2 + 3^2 + 4^2 + 5^2)
55

scm> (accumulate + 5 5 square)  ; 5 + (1^2 + 2^2 + 3^2 + 4^2 + 5^2)
60
; combinator(term(start), term(start + 1))
```



```python
def accumulate(combinator, start, n, term):
    # 用循环实现
    result = start
    start = 0		# 注意重置 start 的值
    while n > 0:
        n -= 1
        start += 1
        result = combinator(result, term(start))	# 核心是每次迭代 result
    return result
```



```python
def accumulate(combinator, start, n, term):
    # 子函数递归实现
    def inner_accu(combinator, start, n, term):
        if n == 1:
            return term(start)
        return combinator(term(start), inner_accu(combinator, start + 1, n - 1, term))
    return combinator(start, inner_accu(combinator, 1, n, term))
```



```scheme
(define (accumulate combiner start n term)
  'YOUR-CODE-HERE
  (define (inner-accu combiner start n term)
    (if (= n 1)
      (term start)
      (combiner (term start) (inner-accu combiner (+ start 1) (- n 1) term))
    )
  )
  (combiner start (inner-accu combiner '1 n term))
)
; 终于改对了 无语
```



### Q7: Tail Recursive Accumulate

```shell
python ok -q accumulate-tail --local
```

把刚刚的实现更新为尾递归，可以默认子函数也是符合尾递归要求的，很不幸，我刚刚的实现并不是尾递归

不过就是再套一层而已，我能做到的，加油😎

```scheme
(define (accumulate combiner start n term)
  'YOUR-CODE-HERE
  (define (inner-accu combiner start n term result)
    (if (= n 0)
      (term start)
      (combiner (term start) (inner-accu combiner (+ start 1) (- n 1) term))
    )
  )
  (combiner start (inner-accu combiner '1 n term 0))
)
```



## Macros

### Q8: List Comprehensions

```shell
python ok -q list-comp --local
```

使用 scheme-macro 在列表中创造列表，类比于 Python 的 `map-function` 

```python
<map-func> for elem in list if <condition>

(list-of <map-expression> for <name> in <list> if <conditional-expression>)
```

返回一个新列表

```scheme
(list-of map-expr for var in lst if filter-expr)

(list-of (* x x) for x in '(3 4 5) if (odd? x))
> (9 25)

(list-of 'hi for x in '(1 2 3 4 5 6) if (= (modulo x 3) 0))
> (hi hi)

(list-of (car e) for e in '((10) 11 (12) 13 (14 15)) if (list? e))
> (10 12 14)
```

说实话我看着这个毫无头绪。。。	直接看答案吧，主要这方面相关的知识除了 Python 的 `eval` 就没接触过了



MD 这个答案比我想的简单的多，应该是用了很多 `scheme build-in function` 然后我不知道...

```scheme
(define-macro (list-of map-expr for var in lst if filter-expr)
  'YOUR-CODE-HERE
  `(map (lambda (,var) ,map-expr) (filter (lambda (,var) ,filter-expr) ,lst))
)

; lambda
; (lambda (<param1> <param2> ...) <body>)
; 与 function define 不同 因为不需要命名 所以匿名函数直接写参数

; map
; proc: (lambda (,var) ,map-expr)	用 "," 对 var 和 map-expr 进行求值
; lst: (filter (lambda (,var) ,filter-expr) ,lst)

; filter
; pred: (lambda (,var) ,filter-expr)
; lst: lst
```

运行过程分析：

- filter：根据 `function-body: filter-expr` 得到满足的新列表

- map：将得到的新列表应用在 `map()` 上，返回结果列表



```scheme
map
(map <proc> <lst>)

; Returns a list constructed by calling proc (a one-argument procedure) on each item in lst

filter
(filter <pred> <lst>)
; Returns a list consisting of only the elements of lst that return true when called on pred (a one-argument procedure)
```



# Done!

emmm... 其他的都还行，但是最后这个 Marcos 确实不熟，因为不熟悉所以就没有勇气是思考，看了答案觉得也没那么难

这次因为书上其实没有讲 `tail-recursion` 和 `Macros` 所以我是先看 disc 讲义再来写 homework 的，看完一部分，写一部分，disc 没有答案参考真的挺麻烦的，害

可能就是前两天找答案找的心累了（也没找到）然后看到这道题已经没有思考能力了2333
