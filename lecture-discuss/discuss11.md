# Tail Recursion and Macros

## Tail Recursion

尾递归会进行一个栈空间的优化，在一个函数的执行末尾调用另一个函数，那么调用函数的函数帧就没有任何意义了，该栈空间可以全部给被调用函数再使用

作用在递归算法的应用中就是递归的空间消耗不会再线性增长，而是保持常量级

因为函数等于是在原地反复解析

### 1.2

实现 scheme 反转

```python
def reversed(lst):
    if lst is ():
        return
    return (reversed(lst[1:]), lst[0])
```



```scheme
(define (reverse lst)
    (if (null? lst)
        '()
        (cons (reverse (cdr lst))  (cons (car lst) '()))
    )
)

; 只能得到这样的结果 ((((() 4) 3) 2) 1)
```



可以 debug 看一下过程 [61A Code (cs61a.org)](https://code.cs61a.org/) 过程还是很清晰的，往往需要一个外部的辅助函数记录数据

其实主要是我最开始作为模板的 python 程序也有问题，呈现的结果也是很多个括号的

```scheme
; GPT answer 感叹一下我真是废物呀
(define (reversed lst)
  (define (iter result remaining)
    (if (null? remaining)
        result
        (iter (cons (car remaining) result) (cdr remaining))))
  
  (iter '() lst))

(reversed '(1 2 3 4))
```



```python
def reversed(lst):
    def iter(result, remain_lst):
        if remain_lst == ():
            return result
        return iter((remain_lst[0], result), remain_lst[1:])
    return iter((), lst)

# 但是这样还是会有层级问题哈	(然后我又去问了 GPT 程序结构没问题 但是数组的处理有点问题
def reversed(lst):
    def iter(result, remain_lst):
        if remain_lst == ():
            return result
        # 不要把 result 用 "()" 包裹起来就好了
        return iter((remain_lst[0], ) + result, remain_lst[1:])
    return iter((), lst)
```

http://markmiyashita.com/cs61a/scheme/reverse_scheme_list/

```scheme
(define (reverse xs)
 (define (reverse-iter xs result)
   (if (null? xs)
     result
     (reverse-iter (cdr xs) (cons (car xs) result)))
 (reverse-iter xs nil))
```



### 1.3

题目中的 n 是元素大小本身

```python
def my_append(a, b):
    a.append(b)
    return a

def my_extend(a, b):
    a.extend(b)
    return a

def insert(n, lst):
    
    def copy_lst(n, result, lst):
        if lst == []:
            return result
        else:
            if n >= lst[0]:
                if n <= lst[1]:
                    return copy_lst(n, my_extend(result, [lst[0], n]), lst[1:])
                return copy_lst(n, my_append(result, lst[0]), lst[1:])
            else:
                return copy_lst(n, my_append(result, lst[0]), lst[1:])
            
    return copy_lst(n, [], lst)

lst = [2, 4, 5, 7]
n =6
print(insert(6, lst))
```



```scheme
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

; 至多得到 ((((() 2) 4) 5 6) 7) 这样的结果	GPT 也寄了
; 我没活了 我想不到为什么
```

Python 就是可以的，爱怎么样怎么样吧，我放弃了



## Macros in Python

Python 本身其实没有宏定义的能力，每个表达式都会作为可执行语句执行，传递返回结果而不是语句本身

本质是 Python 的求值顺序的问题，可以通过字符串 + `eval()` 的方式主动控制求值时机

```python
def list_5(expr):
    return [expr for i in range(5)]

# 1. 执行 print(10)     2. 把 (1) 的执行结果 None 作为 expr 传入
lst = list_5(print(10))     # 10


print(lst)                  # [None, None, None, None, None]
```



```python
def list_5(expr):
    return eval("[" + expr + "for i in range(5)]" )

lst = list_5("print(10)")     # 10
```



### 2.1

Q：传递参数可能有多个

对传入的字符串进行切割，把形参作为字典 Key 存储起来，也要支持 Python function 一等公民的特性



Q：怎么构建特定参数个数的子函数

或许可以试试 `*args` OK 的

```python
def make_lambda(params, body):
    para_list = params.split(", ")  # ['x', 'y', 'f']
    para_num = len(para_list)

    para_dict = {}
    for para in para_list:
        para_dict[para] = None      # 初始化变量 dict

    def run_func(*args):
        para_count = 0
        for para, arg in zip(para_list, args):
            para_dict[para] = arg
            para_count += 1

        if para_count != para_num:
            return False
        return eval(body,globals(), para_dict)
    
    return run_func
```



### 2.2

python f-string 特性 会把最终结果转换为字符串输出，这个 f-string 的特性真的很特别欸....

```python
f"{4 + 4}"
> '8'

# 会在当前环境进行解析
def magic_number():
	return 42
f"{magic_number() = }"

> 'magic_number() = 42'

# 观察当前的结果 magic_number() 被 {} 包裹了 但是还是被打印了出来 并同时进行了函数计算
# f-string 对调用函数进行的特殊处理
```

我个人觉得哈，只用 f-string 去实现输入的解析式是不太可能的，所以我猜想是 f-string 里面嵌套了 eval 函数

不过这样也确实，没什么写的意义了，未解之谜了，放弃



## Macros in Scheme

```scheme
(define-macro (twice f)
	(list 'begin f f)
)

(twice (print 'woof))

; woof
; woof
```



### 3.1

```scheme
(define-macro (or-macro expr1 expr2)
    `(let (
        (v1, expr1))
        (if v1 v1 ,expr2)
    )
)

(or-macro (= 1 0) (+ 1 2))
> 3

; Q: 道理我都懂 但是为什么需要 v1 暂存 expr1 的值

; Debug: 
; (let ((v1 (= 1 0))) (if v1 v1 (+ 1 2)))	其实是让 expr1 进行运算
; v1 False

; 我明白了 expr2 前面的 "," 目的就是让 expr2 进行运算

(define-macro (or-macro expr1 expr2)
    `(let ((v1, expr1))
        (if ,expr1 expr1 ,expr2)
    )
)

; 这样也是可运行的	这个时候 let statement 就无所谓了
```



### 3.2

保留奇数位的数字，然后计算根据第一个运算符对奇数位数字进行运算

```scheme
scm> (prune-expr (+ 10 100 1000))
1010

scm> (prune-expr (prune-expr (+ 10 100) 'garbage))
10
```

碰到这种问题，我感觉我的大脑跟宕机了一样，怎么就啥思路都没有呢

我只知道第一步应该写 list 或者反引号，然后我就不会了✌️

```scheme
(define (prune-expr expr)
    (define (prune-helper lst)
      (if (or (null? lst) (null? (cdr lst)))
        lst
        (cons (car lst) (prune-helper (cdr (cdr lst))))
      )
    )
    (cons (car expr) (prune-helper (cdr expr)))
)

(prune-expr '(+ 10 20 30))
> (+ 10 30)

; 找到了 22sp hm07 的答案 不过这个和 disc11 的题意有点点不同
```



基于这个思路我改了一通

```scheme
(define (prune-expr expr)
    (define (prune-helper lst result)
        (if (or (null? lst) (null? (cdr lst)))
            (if (null? lst)
                '0
                (car lst)
            )
            (+ (prune-helper (cdr (cdr lst)) result) (car lst))
        )
    )

    (prune-helper (cdr expr) 0)
)

(prune-expr '(+ 10))

(prune-expr '(+ 10 100 1000))

(prune-expr '(+ 10 100))

(prune-expr '(+ 10 100 1000 2000 3000))

; 能实现单纯的运算式 但是要求题目中要加上反引号
```



还有运算符 operator 的问题 我最开始是想作为参数传递的，但是也会报错，我也没办法了，直接写死是可以



接下来我是想完成对递归表达式的运算的，但是我在判断第一个表达式是否是递归时判断有问题，所以没戏

不明白，不懂，直接放弃咧（也找不到答案



我找到了！！！！！！！！

```scheme
(define (helper expr remove)
    (cond ((null? expr) nil)
		  (remove (helper (cdr expr) #f))
		  (else (cons (car expr) (helper (cdr expr) #t)))))

(define-macro (prune-expr expr)
    ; 我还在想是什么高大上的解法 结果就是用了 eval 函数 感觉自己被骗了
    ; 不过 define-macro 反引号 逗号 的使用还是要学习的
  `(eval (cons (car ',expr) (helper (cdr ',expr) #f)))
)

; 注意到在递归测试中 内层的 operator 并没有给出
(prune-expr (prune-expr (+ 10 100) 'garbage))
```



### 3.3

自定义一个新的语法执行结构，如果 condition 为真，那么计算下面的所有表达式并且返回最后一个表达式的值作为结果，否则 condition 整体返回 okay

```scheme
scm> (when (= 1 0) ((/ 1 0) 'error))
okay

scm> (when (= 1 1) ((print 6) (print 1) 'a))
6
1
a
```



我摆烂了，我直接躺平看答案

```scheme
(define-macro (when condition exprs)
    (list 'if condition 
          (cons 'begin exprs)
          (quote (quote okay))
    )
)


(when (= 1 0) ((/ 1 0) 'error))
 
(when (= 1 1) ((print 6) (print 1) 'a))

; 很奇怪 我不理解为什么 okay 这里需要两个单引号
; 在 Debug 过程中可以看到 (''okay) 其实是引用了两次了 因为最初的 list 解析就需要一次 后续输出又是一次

(define-macro (when condition exprs)
    (list 'if condition 
          (cons 'begin exprs)
          '(display 'okay)
    )
)

(when (= 1 1) ((print 6) (print 1) 'a))
```

行，至少把最后一题搞懂了



# Done?

我就这么认为了，没办法，我也找不到答案，我能怎么办😭

