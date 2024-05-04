# Scheme

公开的 scheme 解释器，在本机环境上运行		（用 C 实现

```shell
chicken --version	# Compiler Interpreter
```



仅仅是为了这门课的学习的话，是 Proj4 的任务（是一个先有鸡还是先有蛋的问题

课程本身也提供了一个小的 `scheme interpreter` 

```shell
# D:\cs-61A\lecture-lab\lab10	=> cmd
python scheme
python scheme -i <file.scm>
```



基础形式就是 `(operator operandA operandB)` 等效于 `operator(operandA, operandB)` 注意位置关系



## Basic Forms 

从结构上看，所有的表达式都是前缀表达式，都是通过 `operator` 或者 `key word` 去执行逻辑

控制语句，被 `Special Forms` 声明的 operands 不一定会被处理（比如控制分支的情况）

- 赋值 Binding Symbols：`(define <symbol> expression)` 

- 定义函数 New Procedures：`(define (<symbol> <formal parameters>) <body>)`
  - symbol：function name

- 所有的函数在 scheme 内部都是 lambda 表达式

scheme 的注释是用分号开始，并且注释不能出现中文，否则会注释失败



## Lambda Expression

在 scheme 内部，所有函数都是 `lambda expression`，也是一等公民，可以作为返回值也可以作为参数

```scheme
(define (plus4 x) (+ x 4))

(define plus4 (lambda (x) (+ x 4)) )

(define (lambda (x) (+ x 4)))
```



## Special Forms

### Branch Control

```python
# python
if x > 10:
	print(big)
elif x > 5:
    prinr(medium)
else:
    print(small)
```



```scheme
;scheme
(cond ( (> x 10) (print 'big') )
      ( (> x 5) (print 'medium'))
      (else (print 'small'))
)

; example
(define (fib x)
  ; 常数的返回值不需要添加 '()'
  ; 否则会认为是函数
    (cond ((= x 0) 0)
        ((= x 1) 1)
        (else (+ (fib (- x 2)) (fib (- x 1))))
    )
)
```



### Function Call

传参可以传多个

```scheme
; 通过 (sym1 sym2 ... symn . symr)	用 symr 来接收多余参数
(define (sum . numbers)
  (apply + numbers)
)

(display (sum 1 2 3 4))
```





### Begin expression

把写在 begin 后面的每个 expression 都进行一遍求值，并把最后一个 expression 作为返回结果



### Return

在这个解释器里面，返回值和打印指是在一起显示的

```scheme
> (begin (print 1) (print 2) 3 4)
; 打印 1 2 然后 3 4 作为 Primitive Expression do nothing
; 返回 4 作为 return value
1
2
4
```



### Let Expression

局域变量，进行赋值的操作，进行一次 `value expression` 后该绑定关系消失

```scheme
(define c (let ((a 3)
                (b (+ 2 2)))
            
            (sqrt (+ (* a a) (* b b)))))

(let ((a 3) (b 4)) sum-square(a b))
; let para return_expr (后面的 return_expr 也可以通过 begin 写很多 expression
```



### Scheme Lists

```scheme
; 注意最内层的 cons 不需要括号
; nil: empty list
(define a (cons 1 (cons 2 (cons 3 nil))))

; (car) return the first elem of the list
; car: content of the address register
car(a)

; (cdr) return the rest elems of the list
; cdr: content of the decrement register
cdr(a)

; 可以嵌套式构造 list
; any elem of a list can itself be a list
```



### Symbolic Programming

scheme 很特殊的一点：把 names 也作为 data 处理（实习的项目也是根据名称去找大的 

现在的主流语言，往往都是把声明的变量存放在缓存里，然后记住它的地址，而不是它的变量名称（不过我

这会引出一个问题：可以通过 symbol 找到 value，但是 symbol 本身要如何找到呢



#### Special Form: Quotation

Quotation is used to refer to symbols directly in Lisp

在 scheme 的设计概念里面：程序本身就是数据

```scheme
> (define a 2)
> (define b 3)

; buildin function: 根据右侧的参数构造 list
> (list a b)
(a b)

; \': (quota xxx)
; indicate that the expression itself is the value
> (list 'a 'b)
(2 3)
```



### Programs as Data

从上面的语法可以看到，scheme 从设计层面就把程序本身作为数据进行处理，程序语言本身也是一种 list

```scheme
; https://code.cs61a.org/
> (draw '(define a (cons 1 (cons 2 (cons 3 nil)))))
```



#### A Scheme Expression is a Scheme List

呃，写一个可以写程序的程序

```scheme
> (list 'quotient 10 2)		; 有点像 __repr__
quotient 10 2

> (eval (list 'quotient 10 2))
5
```



### Quasiquotation

实现只对一部分内容 quotation，相对于 quotation 支持通过 `,` 取消 quotation，在普通的 quotation 执行中，只能被识别为 `unquote` 

```scheme
> (define b 2)

; Quote
'(a ,(+ b 1))	=> (a (unquote (+ b 1)))

; Quasiquotation
`(a ,(+ b 1))	=> (a 3)
```

可以用来给高阶函数传值，生成新的可执行代码

```scheme
; 第一层 make-add-procedure 只计算出 n
; 内层的匿名函数会保留自己的形参
(define (make-add-procedure n) `(lambda (d) (+ d, n)))

(make-add-procedure 2) => (lambda (d) (+ d 2))
```



### Loop Statement

scheme 中没有定义额外的循环语句，基本都是通过递归实现

```scheme
; 计算在 [2, 9] 范围内偶数的平方和
; 从小到大进行的递归
(begin
    ;定义函数 f(x, total)
 	(define (f x total)
      (if (< x 10)
        (f (+ x 2) (+ total (* x x)))
        total))

    ; 定义 f 的初始值
    (f 2 0))
```



#### Higher Order Function

参考上面写好的递归结构，抽象出实现所需要的各个部分，通过 HOF 的方式传入

`sum-while` 函数名称

`starting-x` 循环的开始位置

`while-condition` 判断是否继续循环	语句列表

`add-to-total` 对要加入 total 的当前变量的值，可能涉及一些运算处理

`update-x` 循环前进的步伐

```scheme
(define (sum-while starting-x while-condition add-to-total update-x)
  	; 必须通过 begin 调用 否则直接使用 define 的话作为被引量不会被识别
    `(begin
        (define (f x total)
          ; "," 表示 while-condition 在调用外层函数时这里要进行替换
            (if ,while-condition
                ; update-x	add-to-total 同理
                (f ,update-x (+ total ,add-to-total))
                total
            )
        )
      	; 内部函数初始化
        (f ,starting-x 0)
    )
)

> (sum-while 2 '(< x 10) '(* x x) '(+ x 2))
(begin (define (f x total) (if (< x 10) (f (+ x 2) (+ total (* x x))) total)) (f 2 0))

> (eval (sum-while 2 '(< x 10) '(* x x) '(+ x 2)))
120
```



Scheme 相关编程汇总

```scheme
(define (unique s)
  'YOUR-CODE-HERE
  (define (find-item item sublst)
    (if (null? sublst)
        #f
        (if (eq? item (car sublst))
            #t
            (find-item item (cdr sublst)))))
  (define (repeated x blst)
    (if (find-item x blst)
        blst
        (cons x blst)))
  (define (creat-new-list s alst)
    (if (null? s)
        alst
        (creat-new-list (cdr s) (repeated (car s) alst))))
  (define lst '())
  (creat-new-list s lst))

; (define (fact n)
;     (if (= n 0)
;         1
;         (* n (fact (- n 1)))
;     )
; )
(define (fact n)
  ; 实际上只用到了子函数 fact-tail 外层函数只负责保存变量
  (define (fact-tail n result)
    (if (= n 0)
        result
        (fact-tail (- n 1) (* result n))))
  (fact-tail n 1))

(define (reverse lst)
    (if (null? lst)
        '()
        (cons (reverse (cdr lst))  (cons (car lst) '()))
    )
)


(define (reversed lst)
  (define (iter result remaining)
    (if (null? remaining)
        result
        (iter (cons (car remaining) result) (cdr remaining))))
  
  (iter '() lst))

(define (inserted n lst)
    (define (copy-lst n result lst)
        (if (null? lst)
            result
            (if (> n (car lst))
                (if (< n (car (cdr lst)))
                    (copy-lst n (cons result (cons (car lst) (cons n nil))) (cdr lst))
                    (copy-lst n (cons result (cons (car lst) nil)) (cdr lst))
                )
                (copy-lst n (cons result (cons (car lst) nil)) (cdr lst))
            )
        )
    )
    (copy-lst n () lst)
)

; no tail recursion
(define (insert elem lst)
    (if (null? lst)
        (list elem)
        (if (< elem (car lst))
            (cons elem lst)
            (cons (car lst) (insert elem (cdr lst)))
        )
    )
)



(define (accumulate-tail combiner start n term)
  'YOUR-CODE-HERE
  (define (inner-accu combiner start n term result)
    (if (= n 1)
      result
      (inner-accu combiner (+ start 1) (- n 1) term (combiner result (term start)))
    )
  )
  (inner-accu combiner '1 n term start)
)

(define (identity x) x)

(accumulate-tail * 1 5 identity)
```



# Logo

看到书里面提到了 `Lexical Scoping` 和 `Dynamic Scoping` 这两个概念，有一些资料说的很好

[Static and Dynamic Scoping - GeeksforGeeks](https://www.geeksforgeeks.org/static-and-dynamic-scoping/)

[Lexical and Dynamic Scope (northeastern.edu)](https://prl.khoury.northeastern.edu/blog/2019/09/05/lexical-and-dynamic-scope/) 

```
func f(x)
	x = 5
	
func A(x = 5)
		extend current env
```



```logo
? to ifelse2 :predicate :True :False
> output run run word ": :predicate
> end
```

