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

