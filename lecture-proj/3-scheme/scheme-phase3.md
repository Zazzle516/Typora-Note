# User-Defined Procedures

这部分主要实现用户自定义函数

#### 项目结构

LambdaProcedure Instance

- formals：形参的 Scheme List 
- body：用户自定义函数的函数体，同样是 Scheme List （这里也能看出程序即数据的特点
- env：定义用户自定义函数的环境



### Problem 7

```shell
python ok -q 07 -u --local
```

```python
from scheme import *

env = create_global_frame()
eval_all(Pair(2, nil), env)     # 2

eval_all(Pair(4, Pair(5, nil)), env)    # 5

eval_all(nil, env) # return None (meaning undefined)

# (begin (define x 3) x)    # 3

# (begin (define x (+ x 1)) 42 (define y (+ x 1)))      # y
```

实现 scheme.py 文件中的 `eval_all()` 函数，在 `do_let_form()`，`scheme_apply()` 和 `do_begin_form()` 中调用，就是计算所有子表达式的意思吧，`do_begin_form()` 的返回结果就是最后一个子表达式的运算结果

如果传递给 `eval_all()` 的是一个空列表，那么返回 `Python-None` 这样就无法被 Scheme 识别然后报错



因为是计算所有表达式并且对最后一个子表达式进行特殊处理，在我看来应该是递归的

每个子表达式调用 `scheme_eval()` 执行（例题的 `define` 会在 `scheme_eval()` 中调用 `SPECIAL FORM` 

不能改变原有的 scheme list 



```shell
python ok -q 07 --local
```

吼吼，过啦，这个也比我想的简单233



### Problem 8

```shell
python ok -q 08 -u --local
```

```python
from scheme_reader import *
from scheme import *

env = create_global_frame()

# Pair('lambda', Pair(Pair('a', Pair('b', Pair('c', nil))), Pair(Pair('+', Pair('a', Pair(Pair('*', Pair('b', Pair('c', nil))), nil))), nil)))
lambda_line = read_line("(lambda (a b c) (+ a (* b c)))")

# print(repr(lambda_line))

formals = Pair('a', Pair('b', Pair('c', nil)))
body = Pair(Pair('+', Pair('a', Pair(Pair('*', Pair('b', Pair('c', nil))), nil))), nil)
# Pair(formals, body)

# lambda_proc = do_lambda_form(lambda_line.rest, env)

# print(repr(lambda_proc.formals))
# Pair('a', Pair('b', Pair('c', nil)))

# print(repr(lambda_proc.body))
# Pair(Pair('+', Pair('a', Pair(Pair('*', Pair('b', Pair('c', nil))), nil))), nil)
```



```scheme
; scheme lambda 语法
(lambda ([param] ...) <body> ...)
```



`LambdaProcedure` 有一列形参列表，函数体，最重要的是作为 `Parent` 的 `env` 

实现 `do_lambda_form()`，构造并返回一个 `LambdaProcedure` 实例，目前是语法层面的实现，执行要等到后面



所以从题目描述来看，就是返回一个合理的 `Pair` 结构

```shell
python ok -q 08 --local
```



### Problem 9

```shell
python ok -q 09 -u --local
```

```shell
from scheme import *
from scheme_reader import *

env = create_global_frame()

# (define (f x) (* x 2))

line = read_line("((f x y) (+ x y))")
# 会把 define 拆掉 但是保留外层的 Pair 结构

expression = Pair(Pair('f', Pair('x', Pair('y', nil))), Pair(Pair('+', Pair('x', Pair('y', nil))), nil))

target = Pair('f', Pair('x', Pair('y', nil)))  # expr.first
func_name = target.first
formals = target.rest           # Pair('x', Pair('y', nil))

body = expression.rest.first    # Pair('+', Pair('x', Pair('y', nil)))

do_define_form(line, env)
```



实现一个语法糖，让用户通过更简单的语法实现 `define` 定义函数的功能

```scheme
(define f (lambda (x) (* x 2)))
; f

(define (f x) (* x 2))
; f
"改成也适配这种定义方式"
```

修改 `do_define_form()` 中的实现，确保能实现嵌套多表达式的函数体

- 使用给定的变量 `target`，`expressions` 来建立函数，确定函数名
- 创造一个 `LambdaProcedure Instance` 可以使用 Q8 中完成的 `do_lambda_form()` 功能
- 把函数名与该匿名函数实例绑定在一起



实际上这里有个问题，因为 Pair 的结构问题，不能直接向 `do_lambda_form()` 传递 `expr.rest` 必须自己构造新的 `Pair expression`，如果自己直接构造 `LambdaProcedure` 的话，无法处理多个 `body-expr` 的问题

```
((x) (+ x 2))		# do_lambda_form
((f x) (+ x 2))		# do_define_form
```



```shell
python ok -q 09 --local
```



### Problem 10

```shell
python ok -q 10 -u --local
```

```python
from scheme import *
global_frame = create_global_frame()

# formals = Pair('a', Pair('b', Pair('c', nil)))
# vals = Pair(1, Pair(2, Pair(3, nil)))

# frame = global_frame.make_child_frame(formals, vals)

# global_frame.lookup('a')    # SchemeError
# frame.lookup('a') # 1

formals = Pair('a', Pair('b', nil))
values = Pair(Pair(1, nil), Pair(Pair(2, nil), nil))

frame = global_frame.make_child_frame(formals, values)
# a=(1, nil)   b=((2, nil), nil)
```



实现 `FrameClass.make_child_frame()` 方法，用来为用户自定义函数创造新的函数帧

Two Arguments:

- formals：a scheme list of Symbols
- vals：a scheme list of Values

返回一个新的子函数帧，并且在该帧中对 `[symbols: values]` 完成绑定

- 如果当前形参与值的数量不匹配那么报错 `SchemeError` 
- 在调用函数的函数帧中建立子函数帧的实例，并且设定自己为子函数帧的父亲
- 返回新的子函数帧

由于 Pair 表达式不能进行迭代操作，所以通过递归实现

```shell
python ok -q 10 --local
```

在写的过程中，其实是有点想复杂了，以为要把 `value` 作为 `scheme_expression` 求出值后再定义到子函数帧中

实际上只要按照原结构直接传递就行（不过我的情况判断写的有点丑陋了

所以这里的传参我觉得也可以理解为 `quote`，毕竟不需要计算，直接传递



### Problem 11

```shell
python ok -q 11 -u --local
```

实现 `LambdaProcedure` 中的 `make_call_frame()` 方法，在 `scheme_apply()` 中有调用，通过使用 `make_child_frame()` 方法，应该返回一个子函数帧的实例，完成形参与值的绑定

```scheme
(define (outer x y)
	(define (inner z x)
		(+ x (* y 2) (* z 3)))
	(inner x 10))

scm> (outer 1 2)
; (outer x=1 y=2)
; 	(inner z=1 x=10)
;		(* 1 3) = 3
;		(* 2 2) = 4
;		(+ 10 4 3) = 17

(define (outer-func x y)
    (define (inner z x)
        (+ x (* y 2) (* z 3))
    )
    inner
)
; outer-func

((outer-func 1 2) 1 10)
; (outer-func 1 2) => (x=1 y=2)
; (inner 1 10) => (z=1 x=10)
; (* 1 3) = 3
; (* 2 2) = 4
; (+ 10 4 3) = 17
```

匿名函数表达式是 `lexically scoped` （静态变量作用域），注意函数帧的父子关系



键盘狂敲一通得到这样的结果(///￣皿￣)○～

```shell
>>> from scheme import *
>>> env = create_global_frame()
>>> double = do_lambda_form(read_line("((n) (* 2 n))"), env) # make double a LambdaProcedure that doubles a number
>>> f1 = double.make_call_frame(Pair(10, nil), env)
>>> f1.lookup('n')
10

>>> env.define('n', 5)
>>> add_n = do_lambda_form(read_line("((x) (+ x n))"), env)
>>> f2 = add_n.make_call_frame(Pair(5, nil), f1) # pass in a different environment as env
>>> f2.lookup('x')
5
>>> f2.lookup('n') # Hint: make sure you're using self.env not env
"Error" 10

# Error: expected
#     5
# but got
#     10
```

Q：为什么强调用 `self.env` 去调用呢，改了之后就全通过了

首先，一个自定义函数有自己的定义帧，执行帧，比如 `add_n` 在 `env` 中定义，在 `f1` 中执行

题目有提示：your new frame should be a child of the frame in **which the lambda is defined** （就是 `env` 

（其实我之前也查过 `lexically scoped` 概念，看来还是没有完全理解

理解一下就是，真正执行的 Body 位于 `env` 函数帧中，所以 Body 查找的变量自然是 `env` 中的 `n` 

至于 `f2` 定义用的 `f1` 参数，应该是为了后续 `dynamically scoped` 的功能做准备（题目也提到了



```shell
python ok -q 11 --local
```



# Phase3 Pass!

这个阶段完成情况比我想象的好，还有 3 个阶段，希望也能顺利👾
