# Lab 12: Macros, Streams

## What Would Scheme Display?

### Q1: WWSD: Macros

```shell
python ok -q wwsd-macros -u --local
```

```scheme
+	; #[+]

(define-macro (f x) (car x))    ; f

(f (+ 2 3))     ; #[+]

(f (quote (2 3 4)))     ; SchemeError

(define quote 7000) ; quote

(f (quote (2 3 4))) ; 7000

(define-macro (g x) (+ x 2))
(g (+ 2 3))		; SchemeError

(define-macro (h x) (list '+ x 2))
(h (+ 2 3))		; 7
```



### Q2: WWSD: Quasiquote

```shell
python ok -q wwsd-quasiquote -u --local
```

没什么好说的其实，主要是对 `quote` 和 `unquote` 情况的处理



## Macros

### Q3: Scheme def

```shell
python ok -q scheme-def --local
```

模仿 `python` 的 `def statement` 实现 `scheme def`

```scheme
(def f (x y) (+ x y))
; 和一般的 scheme expr 区分: 函数名写在了外面
; 其实就是把函数名修改的和 def 平级了
```

利用 scheme 本身的语法去计算 body，调用 `eval()` 函数

```scheme
(eval '(cons 1 (cons 2 nil)))
; (1 2)
```



**Q：**就算是计算，怎么提供变量环境呢

重新构造一个新的 scheme statement 交给解释器执行（就是利用 `do_define_form()` 执行并返回 `func` 



有一种很奇怪的感觉，写完解释器之后我觉得我能做出来（或许说我知道该调用解释器的哪部分，但是直接写在解释器外面的话反而搞不定呢... 2333



不需要调用 eval 因为这个时候只是定义结构，还没有传递参数，也就还没开始进行实际的运算

因为 `args` 和 `body` 本身就是有结构的，所以不需要额外进行 `cons` 操作

（又来了，挫败感，答案的思路倒是明白，但是没写出来，害

```scheme
(define-macro (def func args body)
              ; 只需要从 define 顶层来定义结构
              `(define ,(cons func args) ,body)
)
```



## Streams

### Q4: Multiples of 3

```shell
python ok -q multiples_3 --local
```

定义一个无限流 `all-three-multiples` 包含 3 的所有倍数题目提供了一个辅助函数 `map-stream`，接收一个参数函数 f 和一个流 s，并返回一个元素全部满足 f 要求的新流 `new_s` 

不太懂题目中讲的 `scheme-stream` 概念，因为我的参考资料里完全没有（我猜视频里可能有   （有的，打脸了



#### Scheme Stream

底层是通过 `delay()` 和 `force()` 函数实现的，而 `delay()` 又是服务于 `Promise` 的

（实际上在 Scheme Interpreter 中也有实现，只不过在代码框架中，没有涉及就没有关心

```scheme
(define (delay exp)
  ; 将 exp 暂时封装到一个 lambda 表达式里 而不是对 lambda 求值
  (lambda() exp)
)

(define (force delayed)
  ; 对该 lambda 表达式立刻求值
  (delayed)
)
```



#### Promise

一个 Promise 对象表示在当前创建出的时候值未知的状态，在未来的时候把正确的值交给使用者

（好像不止 scheme，JavaScript 也有实现

Promise State：

- 待定 pending：初始状态，未拒绝，未兑现
- 已兑现 fulfilled：操作成功完成
- 已拒绝 rejected：操作失败

一般的异步编程，都是通过回调函数的模式来通知你之前开启执行的函数执行完毕了，可以进行新的动作

这里以 JavaScript 为例

回调函数 `callback()` 的方式在规模较小的时候可以接收，但是在执行一系列的操作的时候会难以接受

```javascript
setTimeout(() => {
    "回调地狱"
    console.log("wait after three seconds");
    
    setTimeout(() => {
        console.log("another three sconds");
        
        setTimeout(() => {
            console.log("another");
        })
    })
}, 3000);

// 需要一层层的嵌套式写法
```



后续 JavaScript 提供了 Promise 方法，例如 `fetch()` 获取网页资源，都会立刻返回一个 `Promise` 对象

特点在于可以用一种链式结构将多个异步操作串起来

```javascript
fetch("URL").then((response) => response.json());
// 在未来的时刻将返回的数据转换为 JSON 格式

fetch("URL").then((response) => response.json())
            .then((json) => console.log(json));
// 无论是多少个操作都可以通过 then() 串联起来

// 把代码从向右增长变成了向下增长... (也没感觉到特别的优化啊..)

fetch("URL").then((response) => response.json())
            .then((json) => console.log(json))
			.catch((error) => {
    			console.error(error);
			})
			.finally(() => {
    			// 关掉加载动画
    			stopLoadingAnimation();
			});
```

> 后续 JavaScript 有更简洁的实现 Async Await（感觉和 generator 好像.. 



所以换到 scheme 中的实现就是对 `expression.first` 进行运算，而对 `expression.rest` 进行 `delay()` 处理

```python
# scheme.py
class Promise(object):
    """A promise."""
    def evaluate(self):
        # 强制 promise 进行运算
        if self.expression is not None:
            value = scheme_eval(self.expression, self.env)
            if not (value is nil or isinstance(value, Pair)):
                raise SchemeError
            self.value = value
            self.expression = None
        return self.value
    
def do_delay_form(expressions, env):
    return Promise(expressions.first, env)

def do_cons_stream_form(expressions, env):
    return Pair(scheme_eval(expressions.first, env),
                do_delay_form(expressions.rest, env))
    
# 如果不想丢掉 expr.rest 那么一定是从 do_cons_stream_form() 开始调用
```



#### Implement

Q：如何生成一个流

无限的自我递归，但是题目要求不能写新的辅助函数（子函数还是可以的吧）但是要怎么停下来呢，呃

理解有问题，流的运算状态是根据输入决定的（当输入停止的时候，流的运算自然就停止了

```scheme
(define (map-stream f s)
    (if (null? s)
    	nil
    	(cons-stream (f (car s)) (map-stream f (cdr-stream s)))))

(define all-three-multiples
  'YOUR-CODE-HERE
  (define (mul-three elem)
    (if (eq? (modulo elem 3) 0)
      #t
      #f
    )
  )

  (define (helper n)
    (map-stream mul-three (helper (+ 1 n)))
  )
  (helper 0)
)
```



很迷茫，看看答案（艹，进入了这个阶段，就没有一道题是我自己写的

答案是针对每个元素进行 + 3 的操作，移动到 3 的倍数位置，生成一个流（有一说一，虽然没写出来，但是调用自己的思路是对的，但是没有想到用 `cons-stream()` 这个函数）

但是在过测试的时候不能有除了答案的任何内容，所以我也看不到它的输入是如何进行的，又是怎么停下的



## Scheme Basics

### Q5: Compose All

```shell
python ok -q compose-all --local
```

实现类似于 `Python f(g(h(x)))` 的运算方式，也能看成是流计算的一种，这道题是在要求构建流运算的语法..

￣へ￣ 	（太看得起我了

```scheme
(define (compose-all funcs)
  (define (place-taker x)
    x
  )

  (if (null? funcs)
    place-taker
    (begin
      (compose-all (cdr funcs))
      (car funcs)
    )
  )
)

; 虽然没有全部通过 但是我居然能过一个测试我好感动 (虽然可能写的完全不对吧)

(define (halve x) (/ x 2))
(define (square x) (* x x))
(define halve-then-square (compose-all (list halve square)))
(halve-then-square 42)

; Error: expected
;     441
; but got
;     21

; 只计算了第一个函数没有计算到第二个
; 打印了输出 层次应该是正确的..	
(lambda (x) (/ x 2))
(lambda (x) (* x x))
```



好奇怪，和答案的思路也一致（是最接近做出来的一道题了啊啊啊啊啊呜呜呜呜呜

在判断 `funcs` 为空的时候，返回一个返回自己的函数，保证递归的完整性（这部分没什么问题

问题肯定出在递归的执行中，有两件事情要做

- 递归 `(cdr funcs)` 直到为空
- 保存当前的 `(car funcs)` 函数，在递归出栈的时候输出（所以有先后顺序问题

区别在我只返回了函数，答案其实是返回了一个执行体（应该可以这么认为吧

```scheme
(define (compose-all funcs)
  (if (null? funcs)
      (lambda (x) x)
      
      (lambda (x) (
        (compose-all (cdr funcs))
        ; 构造了一个执行体 (func para) 返回
        ; 对比我只返回了 func-name
        ((car funcs) x))
      )
  )
)

(define (halve x) (/ x 2))
(define (square x) (* x x))
(define halve-then-square (compose-all (list halve square)))
(halve-then-square 42)

; 这道题其实是 HOF 啊.. 之前没看出来 outter_func() =>inner_func() 外层要调用子函数
; 所以在 compose-all() 要给出一个参数的位置 (x)
; 用 Python 写一遍问题会更明显
```



```python
def compose_all(func_list, x):
    def inner_func(x):
        # print("current", func_list[0])
        return compose_all(func_list[1:], func_list[0](x)) 
    
    def null_func(x):
        return x
    
    if func_list == []:
        return null_func(x)
    else:
        return inner_func(x)

def halve(x):
    return x // 2

def square(x):
    return x * x

def add_one(x):
    return x + 1

def double(x):
    return x * 2

def halve_then_square(func):
    return func
print(halve_then_square(compose_all([halve, square], 42)))

def composed(func):
    return func
print(composed(compose_all([double, square, add_one], 1)))
```

是能改成 Python，但是参数的结构会发生变化，因为 Python 没办法直接传入执行参数



### Q6: Partial sums

```shell
python ok -q partial-sums --local
```

定义一个流的输入输出

```shell
input> a1, a2, a3, ...

output> a1, a1 + a2, a1 + a2 + a3, ...
```

输出的长度根据输入变化而变化（流的特性）

```scheme
(define (partial-sums stream)
  (define (helper current-ans input-stream)
    (+ (car-stream input-stream) current-ans)
    (helper (+ (car-stream input-stream) current-ans) (cdr-stream input-stream))
  )
  (helper 0 stream)
)
; 问题没有构造返回的 stream 上面那道流的问题我好像也是没想到这个 对流还是太陌生了23333

(define (partial-sums stream)
  (define (helper current-ans input-stream)
    (if (null? input-stream)
      nil
      (cons-stream
        (+ current-ans (car input-stream))
        (helper (+ current-ans (car input-stream)) (cdr-stream input-stream))
      )
    )
  )
  (helper 0 stream)
)
; 思路上没啥问题 主要是我只写了流的执行部分
```



# Done!

只能说是学到了很多，写出来几乎没有2333，快结束了，加油！
