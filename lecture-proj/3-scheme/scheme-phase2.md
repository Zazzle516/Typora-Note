# Some Core Functionality

这个阶段主要是负责完成一些核心的求值运算功能

完成问答才能写代码哦

```shell
python ok -q eval_apply -u --local
```

```shell
Q: What types of expressions are represented as Pairs?
# Call expressions and special forms

Q: What expression in the body of scheme_eval finds the value of a name?
# env.lookup(expr)

Q: How do we know if a given combination is a special form?
# Check if the first element in the list is a symbol and that the symbol is in the dictionary SPECIAL_FORMS

Q: When and how do we create new Frames?
# Whenever a user-defined procedure is called; we use the make_call_frame method of LambdaProcedure

Q: What is the difference between applying builtins and applying user-defined procedures?
# 1. User-defined procedures open a new frame; builtins do not
# 2. Builtins simply execute a predefined function; user-defined procedures must evaluate additional expressions in the body

Q: What exception should be raised for the expression (1)?
# SchemeError("1 is not callable")	对比 Q1 只有调用函数的情况才会是 Pair
```



#### 项目结构

##### scheme_eval()

负责在给定环境中计算 scheme_expr 的结果，除了函数调用其他的功能已经完成

当对一个特殊表达式进行求值时，会调用 `do_?_form` 函数来单独处理



##### scheme_apply()

针对给出的参数调用 `scheme build-in function`，函数已提供



##### .apply()

`Procedure` 子类要完成的方法，并且 `make_call_frame` 会协助完成内置函数或者用户自定义函数的执行



##### Frame Class

创造一个环境帧的数据结构



##### LambdaProcedure Class

定义用户自定义函数的数据结构（联想 `scheme` 中程序本身就是数据



### Problem 2

```shell
python ok -q 02 -u --local
```

实现 `Frame Class` 中的 `lookup()` 方法，

**Frame Class ** 

Bindings：以字典的形式，表示当前帧中的变量的绑定关系，只负责变量名与变量值

Parent：当前函数帧的父函数，`Global Frame` 的父函数帧为 None



**Frame.define()** 

在 `Frame.Bindings` 中添加映射关系



**Frame.lookup()** 

接收一个 `symbol` 作为参数，返回当前环境中，出现在第一个 `Frame` 中的对应值

- Found in the current Frame, return value
- Not found, then lookup the parent frame
- No where to be found, raise `SchemeError` 



```shell
python ok -q 02 --local
```

这个倒不难，挺简单的



### Problem 3

```shell
python ok -q 03 -u --local
```

完成在 `BuildinProcedure Class` 中的 `apply()` 方法，实现对内置函数的调用

**BuildinProcedure Class** 

- fn：the Python func that implements the scheme buildin procedure
- use_env：用来表示这个内置函数是否需要当前求职环境作为最后一个参数传递的布尔值



**BuildinProcedure.apply()** 

- 把 scheme Link list 转换为 Python list，通过 `Pair.first` 和 `Pair.rest` 去构造

- 如果 `self.use_env` 为 `True`，那么把当前环境作为最后一个参数添加到 Python List 中

- 基于所有参数调用 `self.fn` 这里利用 `*args` 传递多个参数

- 如果调用函数导致 `TypeError` 那么说明传递了错误数量的参数，可以使用 `try...catch` 来捕获



```shell
python ok -q 03 --local
```

好耶，这个也简单



### Problem 4

```shell
python ok -q 04 -u --local
```

```shell
>>> expr = read_line('(yolo)')
>>> scheme_eval(expr, create_global_frame())
# SchemeError

expr = read_line('(+ (+ 2 2) (+ 1 3) (* 1 4))')
scheme_eval(expr, create_global_frame())
# 12
```

实现 `scheme_eval()` 中缺失的调用函数的功能，此处调用的都是 `Buildin Procedure` 

- 求出该被调函数所需要的操作符 operator
- 从左到右求出该被调函数所需要的所有参数
- 调用该函数然后返回该函数的计算结果

注意前两步需要递归执行，因为可能有 `add(sub(a, b), c)` 这样子表达式的嵌套

- `validate_procedure()`：判断 `operator` 是否是一个合法的 `scheme procedure` 
- `Pair.map()`：通过调用 `one-arg function` 构造一个新的 `Scheme list` 类比于 `Python.map()` 方法
- `scheme_apply()`：Q3 的实现，执行被调函数



**Q：**如何对多参数进行处理

首先，需要确定 `scheme_eval()` 返回的一定是运算后的结果，或者 `Primitive Expr` 所以可以递归

```python
# 思路大概是这样
def scheme_eval():
    if get_first is operator:
        # 递归找到该 operation 需要的参数
    	args = scheme_eval(rest)
        # 返回运算结果	也可能是外层表达式需要的参数
        return operator(args)
    else:
        # 不一定要写成循环 递归每次只处理一个元素也 ok
        args = []
        for arg in rest:
            args.append(scheme_eval(arg, env))
        # 因为在 scheme_eval() 的其他部分有提供 evaluate atom 语句
        return args
```

emm... 我感觉思路是没问题的，但是实现起来有点点问题，现在考虑的是每次递归处理一个元素，但是 `scheme_eval(rest, env)` 去调用的时候，是以 `Pair` 结构去处理而不是单个元素，无法利用 `atom` 语句

除非一次性的把后面所有的非 operator 元素都找出来，但是我觉得这样的循环思路不太对

就算这么搞，语法怎么写



**Q：**题目提示利用 `Pair.map()` 方法，怎么用的

这个 map 应该用于执行函数时要求的输入形式，和逻辑本身没什么关系，我猜

我现在想这个函数和上一个问题有关	利用 `Pair.map()` 判断当前元素是否是运算符



emmm... 怎么搞呢，想不太出来啊😶‍🌫️，就是隐隐约约能感觉到答案但是实际写不出来



**Answer Thinking** 

过去一天了还是没思路，总结一下答案的思路好了，首先是对 `operator` 的处理，这个处理就是我没想出来的

确实想到在 `Pair.map() ` 中调用 `scheme_eval()` 但是因为这个函数需要接收两个参数没有想到用匿名函数处理

之前思路上是想利用 `scheme_eval()` 递归处理原子表达式，但是语法不知道怎么写，还是没想到匿名函数的方法



为什么需要 `Pair.map()` 方法，是根据表达式的 Pair 结构决定的，即使是平级的参数在 `Expr` 中也是有层级结构的，所以必须以相同的层级结构返回

```python
def scheme_stringp(x):
    return isinstance(x, str) and x.startswith('"')

def scheme_symbolp(x):
    return isinstance(x, str) and not scheme_stringp(x)

def scheme_eval(expr, env):
    if scheme_symbolp(expr):
        # operator 一定是以 'xxx' 的方式进行存储 但是存储的不一定都是 operator
        # 也有可能是变量什么的
        # 所以这里只提供了查找	具体找到的是什么 需要额外判断
        return env.lookup(expr)
    
    elif scheme_evaluating(expr):
        # return number atom expression
        return expr
    
    first = expr.first
    rest = expr.rest
    
    if validate_procedure() and first in SPECIAL_FORMs:
        return SPECIAL_FORMS[first](rest, env)
    
    else:
    	# 1. get operator
        operator = scheme_eval(first, env)
        # 2. get operands
        # 针对 rest 中的所有元素调用 scheme_eval 递归性完成参数的获取
        # 这个函数其实就是我最开始想的循环遍历所有的参数 最后返回参数列表
        args = rest.map(lambda x: scheme_eval(x, env))
        return scheme_apply(operator, args, env)
```



```shell
python ok -q 04 --local
```



### Problem 5

```shell
python ok -q 05 -u --local
```

实现命名，`define` 主要包括 3 个功能，定义变量，定义用户自定义函数，定义匿名函数

这里只实现第一个功能，对变量进行定义，区别在于接收的参数是否是列表，在 `do_define_form` 第一部分

```
Q: What is the structure of the expressions argument to do_define_form?
Pair(A, Pair(B, nil)), where:
   A is the symbol being bound,
   B is an expression whose value should be evaluated and bound to A
   
(eval (define tau 6.28))
eval takes an expression represented as a list and evaluates it
6.28
```



```shell
python ok -q 05 --local
```

以一种很暴力很丑陋的方式解决了这个问题...	我自己都要受不了了

这里题目有提到，在实现之后考虑一下更极端的输入情况

- 在 `scheme_eval()` 执行计算之前是否要考虑运算的合法性
- 以不正确的语法输入时，是否会有合适的错误提示
- 在 `define` 求值过程中，可能嵌套处理特殊情况

这些我还真没想....



不过看了答案，也确实就这么写，只是把行数浓缩了，执行是相同的



### Problem 6

```shell
python ok -q 06 -u --local
```



```shell
Q: What is the structure of the expressions argument to do_quote_form?
# Pair(A, nil), where: A is the quoted expression
"交由 do_quote_define() 处理的 expression 本身不包含 quote"

Q: do_quote_form(Pair('hi', nil), global_frame)
# 'hi'

Q:
>>> expr = Pair(Pair('+', Pair('x', Pair(2, nil))), nil)
>>> do_quote_form(expr, global_frame)
# Pair('+', Pair('x', Pair(2, nil)))
"从这里就能看出直接按原结构返回即可"

Q:
>>> ''hello
#  (quote hello)
"不同于上面直接调用 do_quote_form 的测试 这里的输入是在 scheme 解释器中"
"(''hello) 的解析会先经过 scheme_read() 处理 (quote(quote hello)) => "

Q:
>>> (eval (cons 'car '('(4 2))))
# 4

Q: read_line(" '(a b) ")
# Pair('quote', Pair(Pair('a', Pair('b', nil)), nil))

Q: read_line(" `(,b) ")
# Pair('quasiquote', Pair(Pair(Pair('unquote', Pair('b', nil)), nil), nil))
```



实现 scheme 中的引用 quoting 功能 `do_quote_form()`，要求返回不进行求值的特殊表达式

然后，实现在 `scheme_reader.scheme_read()` 功能中的 `quote` 问题，同样涉及递归性调用

```
'<expr> => (quote <expr>)
`<expr> => (quasiquote <expr>)
,<expr> => (unquote <expr>)
```

处理的方式其实是再套了一层 Pair，`Pair('quote', Pair('bagel', nil))` 表示 `'bagel` 

```shell
python ok -q 06 --local
```

看着很难，没想到顺利解决了23333，可能是 Q4 的过程，这个答案我写的还蛮优雅欸hhhhh



# Phase2 Pass!

主要是在 Q4 感受到了严重的挫败，看了答案也发现即使有思路，那样的解答也不是我能写出来的👾

虽然说 Don't Compare 但是遇到这种情况，确实难免对自己失望，不过剩下的内容也要好好努力完成
