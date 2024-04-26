# Scheme Interpreter

为一个 scheme 的子集开发一个解释器，所以这个项目的解释器和为 scheme 专门开发的解释器还是不一样的



#### 待实现文件

```
scheme.py	scheme_reader.py	questions.scm	test.scm
```



#### 项目结构

- scheme.py：实现 REPL 和针对 Scheme expression 的求值方法

- scheme_reader.py：针对输入进行处理	特殊的 Pair 和 nil 输入也在这里处理
  - buffer.py：实现在 scheme_reader.py 中使用的 Buffer 类
- scheme_tokens.py：针对 scheme_reader.py 得到的结果标签化
- ucb.py：针对该项目的一些功能性函数

- questions.scm：针对 Phase 3 的框架代码

- tests.scm：用 scheme 写的测试用例

- mytests.rst：可以添加自己测试用例的文件



#### Debug

```python
print("DEBUG:", x)
```



### Interpreter details

#### Symbols

Identifier：a sequence of letters

该项目的 scheme 解释器大小写不敏感，只要两个单词除了大小写完全一致，那么就认为相同



#### Turtle Graphics

该项目的额外部分（算是提高篇吧），会使用到 python turtle 的包，一般是默认安装的（我好像没有

```shell
pip install turtle
```



### Implementation overview

#### Read

把用户输入的 scheme string 解析为 Python 的表达

- 词法分析：已经在 `scheme_tokens.py` 文件中的 `tokenize_lines` 函数中实现	
  - 返回一个储存 token 的缓存 Buffer

- 语法分析：发生在 `scheme_reader.py` 文件中的 `scheme_read()` 和 `read_tail()` 函数
  - 相互递归来解析 Scheme tokens，转换为 Python 表达



#### Eval

对 scheme 表达式进行运算，主要在 `scheme.py` 文件中

- 求值 Eval：发生在 `scheme_eval()` 函数中，无论是函数调用还是特殊形式都会处理

- Procedure Apply：`scheme_apply` 调用 `BuildInProcedure` 实例中的方法 `apply` 
- User Defined Procedure：`scheme_apply` 创建一个新的函数帧，然后在新的函数帧内调用 `eval_call` 
  - 相互递归构成一个 `eval-apply` 循环



#### Print

利用 Python 的 `__str__` 进行输出



#### Loop

循环功能是在 `read_eval_print_loop` 函数中实现的



### Exceptions

在解释器的开发过程中，可能会遇到各种各样的错误，有的是解释器自身的错误，有的是用户的输入错误

针对后者通过 `raise SchemeError` 通知用户，只要在正确的地方打印错误信息即可



### Running Interpreter

```shell
python scheme.py
```



```shell
python scheme.py tests.scm
```



## Phase 1

处理用户输入，解析为 Python Value

| Input Example  | Scheme Expression Type | Our Internal Representation                                  |
| -------------- | ---------------------- | ------------------------------------------------------------ |
| `scm> 1`       | Number                 | Python's built-in `int` and `float` values                   |
| `scm> x`       | Symbols                | Python's built-in `string` values                            |
| `scm> #t`      | Booleans(`#t`, `#f`)   | Python's built-in `True`, `False` values                     |
| `scm> (+ 2 3)` | Combinations           | Instances of the `Pair` class, defined in `scheme_reader.py` |
| `scm> nil`     | `nil`                  | The `nil` object, defined in `scheme_reader.py`              |

描述中提到 Combinations 包含函数调用和特殊形式

仔细阅读 `buffer.py` 文件，因为输入的处理结果都写在这个文件中



### Problem 1

```shell
python ok -q 01 -u --local
```

实现 `scheme_read()` 和 `read_tail()` 完成解析功能



- **scheme_read ** 

从 `buffer_scr` 中移除无效的 token 比如括号什么的，返回由 Python 表示的，有层级关系的表达式

| token that read   | Action depend on that token                                  |
| ----------------- | ------------------------------------------------------------ |
| `nil`             | return the `nil` object                                      |
| `(`               | Call `read_tail` on the rest of `src` and return             |
| `'` or `\` or `,` | processed as a `quote`, `quasiquote`, or `unquote` expression |
| not a delimiter   | Build-in functions     (privided)                            |
| none of the above | raise an error                                               |



- **read_tail** 

获取列表或者元组的剩余部分，以 `Pair` 的实例，链表的形式返回

| token that read   | Action depend on that token                                  |
| ----------------- | ------------------------------------------------------------ |
| no more tokens    | missing a close parenthesis and raise an error               |
| `)`               | Remove this token from the buffer and return `nil`           |
|                   | 1. call `scheme_read()` to read next complete expression in buffer |
| none of the above | 2. call `read_tail()` to read the rest of the list           |
|                   | 3. 把结果作为 `Pair` 的实例返回                              |



在 `get_ans.py` 文件写的测试，用于完成问题

```python
from scheme_reader import *
tokens = tokenize_lines(["(+ 1 ", "(23 4)) ("])     # "(+ 1 " // "(23 4)) ("
src = Buffer(tokens)
src.current()
scheme_read(src)    # Pair(23, Pair(4, nil))        Q: 这里的括号数量是怎么进行判定的

# Q: 移除的是 complete expression 所以剩余的 ')' 没有被移除
# A: 这是在实现 scheme_read 重点注意的

# A: 所以也需要这种特殊的缓存数据结构 buffer
src.current()   # ')'

# 默认已经处理了 first_paratheses '('

# Second test
read_tail(Buffer(tokenize_lines([')'])))            # nil

read_tail(Buffer(tokenize_lines(['1 2 3)'])))       # Pair(1, Pair(2, Pair(3, nil)))

# Q: 每个元素都只能放在 Pair[0]     Pair[1] 要么是 Pair class 要么是 nil
read_tail(Buffer(tokenize_lines(['2 (3 4))'])))     # Pair(2, Pair(Pair(3, Pair(4, nil)), nil))

# the first element is the next complete expression from (1) and the second element is the rest of the combination from (2)
first_pair = Pair(3, Pair(4, nil))
second_pair = Pair(first_pair, nil)                 # 重点是这里需要的额外一层
Pair(2, second_pair)

# Pair 只能接收两个参数

read_tail(Buffer(tokenize_lines(['(1 2 3)'])))      # SyntaxError

read_line("(+ (- 2 3) 1)")
> Pair('+', Pair(Pair('-', Pair(2, Pair(3, nil))), Pair(1, nil)))
```

实现起来最主要的应该是 `read_tail()` 因为是真正返回 `Pair` 序列的



**Q：**因为 `read_tail()` 是直接处理去掉首括号的情况，而嵌套的表达式又包含首括号，该如何区分这种情况

```python
def read_tail(src):
    if src.current() is None:
        raise SyntaxError('unexpected end of file')

    elif src.current() == ')':
        src.pop_first()
        return nil

    else:
        # BEGIN PROBLEM 1
        "*** YOUR CODE HERE ***"
        src_first = src.current()
        if src_first == '(':
            return Pair(scheme_read(src), nil)
        else:
            src.pop_first()
            return Pair(src_first, read_tail(src))
        # END PROBLEM 1
```

```python
# Pair(2, Pair(Pair(3, Pair(4, nil)), nil))
# Pair(2, Pair(Pair(3, Pair(4, nil)), nil))
print(repr(read_tail(Buffer(tokenize_lines(['2 (3 4))'])))))	# check

# Pair(Pair(2, nil), nil)
# SyntaxError
print(repr(read_tail(Buffer(tokenize_lines(['(2)'])))))			# Wrong
```



**想法一**：就是在函数外面定义一个变量，然后如果是第一个元素，就修改该变量的值

我尝试了一下，设计一个 `FlagClass` 但是没有办法在读取一次后重置 `flag` 所以应该不是这种方法



**想法二**：统计左括号和右括号的数量，必须要有 1 的差值

不行，同样会面临无法重置的问题，我不知道，我想不出来呜呜呜呜呜



**想法三**：直接遍历整个列表，查看括号的匹配关系

Buffer 不是 iterable 类型 只能写递归的子函数，执行过程是符合我的想法的，但是结果展示的是 None 我不理解

**原因**：没有全部写上 `return`



**Q：**改了这个问题之后，在测试的时候又出了新的问题，判断匹配关系的时候我直接把 `src` 拿过去了，而不是复制结果，因为 python 的指针问题，这时候 src 就已经提前运行了一遍，结果为 None 了

```python
import copy
test_src = copy.deepcopy(src)

> TypeError: cannot pickle 'generator' object
# pickle: Python 的持久化模块
```

又报了个错哈，emmm.... 我快没耐心了 [python - TypeError: can't pickle generator objects - Stack Overflow](https://stackoverflow.com/questions/28963354/typeerror-cant-pickle-generator-objects) 

[一文带你搞懂Python中pickle模块 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/419362785) 

```python
# Don't use a generator expression when you want to pickle data
```

所以根据 Debug 把 src 改成 `src.current_line` ，这样辅助函数也要改成用列表处理的

```python
def parenthes_check(src):
    # False: normal     True: SyntaxError
    left_num = 0
    right_num = 0
    
    def inner_count(sub_src):
        # 不能再用 buffer 的方法而是 list 方法
        nonlocal left_num
        nonlocal right_num
        if sub_src.current() is None:
            if (right_num - left_num) != 1:
                return True
            return False
        elif sub_src.current() == '(':
            left_num += 1
            sub_src.pop_first()
            return inner_count(sub_src)
        elif sub_src.current() == ')':
            right_num += 1
            sub_src.pop_first()
            return inner_count(sub_src)
        else:
            sub_src.pop_first()
            return inner_count(sub_src)

    return inner_count(src)

# 上面只是备份
```



**Q：**在测试 `read_line('(a)')` 又出现了错误，因为 buffer_src 本身不变化，也就是 `src.current_line` 不论是否有 `src.pop_first()` 都不会变，需要根据 `src.index` 进行截断

```python
test_src = copy.deepcopy(src.current_line[src.index:])
```



**Q：**在测到 case7 的时候又出了问题，嵌套的子表达式在结束后直接 `return nil` 导致提前结束

```python
from scheme_reader import *

# Pair('+', Pair(Pair('-', Pair(2, Pair(3, nil))), Pair(1, nil)))

# 这个是我修改 read_tail().")" 的输出 显然也不对 然后才发现是 read_tail().scheme_read() 问题
# Pair('+', Pair(Pair('-', Pair(2, Pair(3, Pair(1, nil)))), nil))

# 核心问题是执行到 '1)' 提前结束了  因为直接 return nil  根据 src.index 也能判断没有读完
print(repr(read_line("(+ (- 2 3) 1)")))

# 如果在 read_tail().")" 情况再开一个 Pair 结构就会混乱
# 问题是在 "(" 的判断中 构造的 Pair 的第二个元素应不应该为 nil
```

根据结构进行了一点修改

```python
# if src_first == '(':
#	return Pair(scheme_read(src), nil)

if src_first == '(':
	return Pair(scheme_read(src), read_tail(src))
```



至此，终于通过测试... 真是艹了，我去看看答案

PS：能明显感受我的测试运行时间很长，真是抱歉了呜呜呜呜呜

```python
def scheme_read(src):
    """Read the next expression from SRC, a Buffer of tokens.

    >>> scheme_read(Buffer(tokenize_lines(['nil'])))
    nil
    >>> scheme_read(Buffer(tokenize_lines(['1'])))
    1
    >>> scheme_read(Buffer(tokenize_lines(['true'])))
    True
    >>> scheme_read(Buffer(tokenize_lines(['(+ 1 2)'])))
    Pair('+', Pair(1, Pair(2, nil)))
    """
    if src.current() is None:
        raise EOFError
    val = src.pop_first() # Get the first token
    
    if val == 'nil':
        # BEGIN PROBLEM 1
        "*** YOUR CODE HERE ***"
        return nil
        # END PROBLEM 1
    elif val == '(':
        # BEGIN PROBLEM 1
        "*** YOUR CODE HERE ***"
        return read_tail(src)
        # END PROBLEM 1
    elif val in quotes:
        # BEGIN PROBLEM 6
        "*** YOUR CODE HERE ***"
        # END PROBLEM 6
    elif val not in DELIMITERS:
        return val
    else:
        raise SyntaxError('unexpected token: {0}'.format(val))

def read_tail(src):
    """Return the remainder of a list in SRC, starting before an element or ).

    >>> read_tail(Buffer(tokenize_lines([')'])))
    nil
    >>> read_tail(Buffer(tokenize_lines(['2 3)'])))
    Pair(2, Pair(3, nil))
    """
    try:
        if src.current() is None:
            # case: no more tokens
            raise SyntaxError('unexpected end of file')
        
        elif src.current() == ')':
            # BEGIN PROBLEM 1
            "*** YOUR CODE HERE ***"
            # case: ')'
            src.pop_first()
            return nil
            # return Pair(nil, read_tail(src))
            # END PROBLEM 1
        
        else:
            # BEGIN PROBLEM 1
            "*** YOUR CODE HERE ***"
            test_src = copy.deepcopy(src.current_line[src.index:])
            if parenthes_check(test_src):
                raise SyntaxError('wrong number ()')
            src_first = src.current()
            if src_first == '(':
                return Pair(scheme_read(src), read_tail(src))
            else:
                src.pop_first()
                return Pair(src_first, read_tail(src))
            # END PROBLEM 1
    
    except EOFError:
        raise SyntaxError('unexpected end of file')

def parenthes_check(src):
    # False: normal     True: SyntaxError
    left_num = 0
    right_num = 0
    
    def inner_count(sub_src):
        nonlocal left_num
        nonlocal right_num
        if len(sub_src) == 0:
            if (right_num - left_num) < 1:
                return True
            return False
        elif sub_src[0] == '(':
            left_num += 1
            sub_src.pop(0)
            return inner_count(sub_src)
        elif sub_src[0] == ')':
            right_num += 1
            sub_src.pop(0)
            return inner_count(sub_src)
        else:
            sub_src.pop(0)
            return inner_count(sub_src)

    return inner_count(src)

# 其实改到这里就能有点意识到应该有更简单的递归...
# 因为本质上我是一个循环的思路 强行套在递归上 导致这么丑陋
```

测试实现

```shell
python ok -q 01 --local
```



独立测试功能

```shell
python scheme_reader.py --repl

exit	# exit the interpreter
```



# Phase1 Pass!

这个 phase1 比我想象的艰辛的多，虽然是完成了，但是感觉完成的很丑陋23333



我看答案了，有点破防了，呜呜呜呜呜呜

直接利用 `scheme_read()` 的异常处理就可以了，我还在想呢，我难道要自己写 `SyntaxError` 果然不需要



### Answer Think

核心是对 Buffer 这个数据结构没有深入理解，其实到最后我的实现和答案的结构也已经很像了.....

Buffer 中的数据不会消失，改变的只是 `Buffer.index` 所以只要能让 `index` 增加，就一定能结束递归

（同理，因为 Buffer 中的数据不会消失，所以不用每次对 `parenthes_number` 进行判断）



也有对 `scheme_read()` 函数功能认知不清晰的原因，一个完整的表达式不止包括 `(expr)` 也包括数字和字符

假设输入的表达式都是合乎语法的，如果出现括号不匹配的问题，一定会在最后构造错误



从 Pair 构造的结构去考虑，一个 `full expr` 的内部必然是一个个 `sub full expr` 构成的，所以每次通过 `scheme_read()` 读取当前一个完整的 `sub full expr` 再通过递归 `read_tail` 依次处理剩余的

```python
# 读取一个完整表达式    1. number	2. symbol	3. (expr)
first = scheme_read(src)

# 递归依次
rest = tail_read(src)
```

