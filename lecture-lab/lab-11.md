# Lab 11: Interpreters

#### Interpreters

Two languages at work：

- The language being interpreted：the PyCombinator language

- The underlying implementation language：Python

这两种语言不一样是不同的语言，本次的 lab 就是在用 python 实现一个迷你 python （人话就是没卵用	**Metacircular Evaluation** 



#### Read-Eval-Print

- Read：take an input and pass it to a Lexer and Parser	在 `read.py` 进行执行
  - Lexer 在 `tokenize()` 中实现，把 `string` 切成 `tokens` 去处理
  - Parser 在 `read_expr()` 中实现，把 `tokens` 转变为 `class Expr` 
- Eval：递归性的去调用解析函数和求值函数 `eval()` 和 `apply()` 

- Print：使用 `__str__` 进行输出

目的：Turns input into a useful data structure

返回：Input expression represented as an instance of a subclass of Expr



#### PyCombinator Interpreter

通过 python 实现一个迷你 python，包括简单的计算指令，甚至包括匿名函数调用，本项目要实现的只有在 `expr.py` 文件的 `Eval` 和 `Apply` 功能

```shell
python repl.py
# To exit the interpreter, type Ctrl-C or Ctrl-D
```



```shell
> add(3, 4)
7
> (lambda x, y: add(y, x))(3, 5)
8
> (lambda x: lambda y: mul(x, y))(3)(4)
12
> (lambda f: f(0))(lambda x: pow(2, x))
1
```



### Q1: Prologue

#### 项目结构

- repl.py：REPL logic，好贴心的提示我不需要完全理解全部代码
- reader.py：解析输入生成 `class Expr` 
- expr.py：不同的 `subclass Expr` 分别对应的不同的行为，和求值结果

```shell
# reader test
python ok -q prologue_reader -u --local

# Expr test and Value test
python ok -q prologue_expr -u --local
```



```shell
# tokenize output

add(3, 4)
['add', '(', 3, ',', 4, ')']	# 注意数字直接作为结果输出

tokenize('(lambda: 4)()')
['(', 'lambda', ':', 4, ')', '(', ')']
```



### Q2: Evaluating Names

```shell
python ok -q Name.eval --local
```

重点，the value of a name depends on the current environment

本项目中的环境是 `a dictionary that maps variable names (strings) to their values` 实现

所以 `Name.eval` 接收当前环境作为参数，根据 `var_name` 返回值



### Q3: Evaluating Call Expressions

```shell
python ok -q CallExpr.eval --local
```

实现 `Call Expression` 进行求值，每个 operator 都可能有 0 个或者多个参数

在执行过程中，接收 `CallExpr` 作为参数，重点也是对当前环境的调用，使用 `eval()` 或者 `apply()` 求值



Q：如何处理多参数

我目前是通过循环和列表处理的，但是在最后的执行有问题，我不太确定是不是我的方法有问题，但是我快没活了



Q：如何判断当前的参数是否是嵌套表达式

根据 `print("Debug",x)` 的结果显示的类型进行判断



Q：如何处理嵌套表达式

现在实现的还是最简单版本的，没考虑递归什么的，我现在是把单个 operand 取出来判断类型，如果要递归性质的话，应该把递归过程写在内部，然后调用后续的内容

注意此时取出的 `current_operand / i` 就是要递归的表达式本身，不需要进行额外处理



```python
class CallExpr(Expr):
    """A call expression represents a function call.

    The `operator` attribute is an instance of `Expr`.
    The `operands` attribute is a list of `Expr` instances.

    For example, the call expression `add(3, 4)` is parsed as

    CallExpr(Name('add'), [Literal(3), Literal(4)])

    where `operator` is Name('add') and `operands` are [Literal(3), Literal(4)].
    """
    def __init__(self, operator, operands):
        Expr.__init__(self, operator, operands)
        self.operator = operator
        self.operands = operands

    def eval(self, env):
        """
        >>> from reader import read
        >>> new_env = global_env.copy()
        >>> new_env.update({'a': Number(1), 'b': Number(2)})
        >>> add = CallExpr(Name('add'), [Literal(3), Name('a')])
        >>> add.eval(new_env)
        Number(4)
        >>> new_env['a'] = Number(5)
        >>> add.eval(new_env)
        Number(8)
        >>> read('max(b, a, 4, -1)').eval(new_env)
        Number(5)
        >>> read('add(mul(3, 4), b)').eval(new_env)
        Number(14)
        """
        "*** YOUR CODE HERE ***"
        eval_operator = self.operator.var_name
        print("DEBUG:", "eval_operator", eval_operator)

        eval_operand = []
        for i in self.operands:
            print("DEBUG:", "i type", type(i))

            if type(i) == CallExpr:
                # 需要递归式调用
                print("DEBUG:", "is PrimitiveFunction")
                eval_operand.append(CallExpr.eval(i, env))
            
            if isinstance(i, Name):
                # 变量名
                print("DEBUG:", "is Variable")
                eval_operand.append(i.eval(env))
            
            if isinstance(i, Literal):
                # 参数
                print("DEBUG:", "Number")
                eval_operand.append(i.eval(env))
                print("DEBUG:", Number(i).value)
        
        # 执行计算
        print("DEBUG:", "eval_operand last", eval_operand)
        eval_function = env[eval_operator]
        eval_result = eval_function.apply(arguments=eval_operand)
        return eval_result

    def __str__(self):
        function = str(self.operator)
        args = '(' + comma_separated(self.operands) + ')'
        if isinstance(self.operator, LambdaExpr):
            return '(' + function + ')' + args
        else:
            return function + args
```



### Q4: Applying Lambda Functions

```shell
python ok -q LambdaFunction.apply --local
```

题目分析提到，这里的匿名函数的解析执行主要包含 3 点：

- parameters：执行需要的参数

- body：匿名函数执行体
- parent：当前匿名函数定义的函数帧！参数也是在这个帧中获得，但是匿名函数的运行在下一帧中

题目提到了对匿名函数的传参要保持正确的对应关系，题目也给出了一个方法

- 制作一份当前环境的拷贝
- 在该环境中更新执行匿名函数需要的参数
- 在更新好参数的环境中执行



```python
lambda x, y: add(x, y)

LambdaExpr(['x', 'y'], CallExpr(Name('add'), [Name('x'), Name('y')]))

parameters_list = ['x', 'y']
body = CallExpr(Name('add'), [Name('x'), Name('y')])
```

虽然说是 lambda 函数，但是支持的功能也只有那些已经写进 `global_env` 的基础功能，所以具体的执行可以交给 `CallExpr.eval()` 函数



Q：如何正确匹配参数与值，尤其是减法顺序强相关的运算

```shell
# 强调形参 fomal parameters 的无关性 不能认为一定是 x y 进行传参
>>> sub_lambda = read('lambda add: sub(10, add)').eval(global_env)
>>> sub_lambda.apply([Number(8)])
Number(2)

LambdaExpr(['add'], CallExpr(Name('sub'), [Number(10), Name('add')]))
```

使用 `zip(self.parameters, arguments)` 实现



Q：实现匿名函数的递归调用

```python
read('(lambda x: lambda y: add(x, y))(3)(4)').eval(global_env)

LambdaExpr(['x'], CallExpr(
    LambdaExpr(['y'], CallExpr(Name('add'), [Name('y')]))
)
```

在这里其实能发现一个埋在 `CallExpr` 的 bug 

```python
eval_operator = self.operator.var_name

# 这行在普通函数递归调用是没有问题的 但是来到匿名函数递归 这里没有对类型进行判断
>>> read('(lambda x: lambda y: add(x, y))(3)(4)').eval(global_env)
DEBUG: operator type <class 'expr.CallExpr'>

# 这里其实是直接跑到 CallExpr 运行 我也不知道为什么
# 我分析了一下执行过程 这个问题出现在在 得到 expression 后导向了 CallExpr 实际上应该导向 LambdaExpr

# 又看了看 是在 reader.py 里出的问题 因为这里写死了导向 CallExpr
def read_call_expr(src, operator):
    ...
	operator = CallExpr(operator, operands)
```

话说是如此，我不觉得这个修改是在 `LambdaFunction.eval()` 中发生的，但是我也不知道怎么改，我去看看



好哈，我去看了看，`LambdaFunction.eval()` 本身写的没有问题，问题还是在 `CallExpr.eval()` 上

感受到了顶级的差距，核心是根据它自己的类型去调用 `eval()`，而不是我自己去判断

```python
class CallExpr(Expr):
    ...
    def eval(self, env):
        "*** YOUR CODE HERE ***"
        operator = self.operator.eval(env)
        if operator is None:
            raise NameError(f"{self.operator} is not defined")
        operands = []
        for x in self.operands:
            operands.append(x.eval(env))
        return operator.apply(operands)
```

我没想到这一点，核心是我没有意识到**这是一个抽象层**，我把 `CallExpr` 作为一个具体的函数去实现

这点一定要注意



### Q5: Handling Exceptions

这里没有给出特殊的测试或者要求，基本就是对程序健壮性的维护，比如题目里提到的未定义函数功能在上面的 `CallExpr` 也进行了 `raise Exception` 的提示



# Done!

这节的 lab 有点点坎坷，主要是实现 LambdaFunction 的时候意识到 CallExpr 除了问题，当时 CallExpr 也花了蛮长时间，但是因为通过了当时的测试，我就没往抽象层再细想，淦(‾◡◝)