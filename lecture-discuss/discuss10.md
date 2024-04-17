# Scheme, Scheme Lists

## Lambdas and Defining Functions

### 4.1

```scheme
(define (factorial x)
    (if (= x 1)
        1
        (* x (factorial (- x 1)))
    )
)

(factorial 10)
```



### 4.2

```scheme
(define (fib n)
    (if (= n 0)
        0
        (if (= n 1)
            1
            (+ (fib (- n 2)) (fib (- n 1)))
        )
    )
)

(fib 10)
```



## Pairs and Lists

### 5.1

这个试了试没写出来，看了看 solution

```scheme
(define (my-append a b)
    (if (null? a)
        b
        (cons (car a) (my-append (cdr a) b))
    )
)

(my-append '(1 2 3) '(2 3 4))
```

这边是我的思路有问题，我一开始想着把 b 遍历，和 a 整合成新的列表，后来发现不行，又在想把 a 和 b 都遍历一遍（理论上应该行吧，但是我也没搞出来

看了解决，发现是遍历 a 而不是遍历 b，我真是呆逼	整出一个新列表之后把 b 写入就好了



### 5.2

在列表的特定位置插入特定元素

```scheme
(define (insert element lst index)
    (if (null? lst)
        nil
        (if (= index 0)
            (cons element (insert element lst (- index 1)))
            (cons (car lst) (insert element (cdr lst) (- index 1)))
        )
    )
)

(define a '(1 2 3 4 5))
(insert 7 a 2)
> (1 2 7 3 4 5)
```



### 5.3

把每个元素在后面的位置重复一遍，应该会用到上面实现的 `insert` 

这个没整出来，因为只想着怎么去利用刚刚写的 `insert` 实际上用不了啊，我写着写着也发现了，`duplicate` 是对元素的拷贝，而 `insert` 返回的是列表，我想着可以作为参数调用，但是简单分析一下就能发现，这个递归无法结束

```scheme
(define index 0)
(define (duplicate lst)
    (if (null? lst)
        nil
        (duplicate (insert (car lst) (+ 1 index)))
    )
)
; Error
```



```scheme
(define (duplicate lst)
    (if (null? lst)
        nil
        ; 很巧妙啊这里 通过再构造一个 list 实现三个参数
        (cons (car lst) (cons (car lst) (duplicate (cdr lst))))
    )    
)

(define a '(1 2 3 4 5))

(duplicate a)

; 还是对 scheme 太不熟悉了 这种题都没想出来2333
```



# Done!

还行，这门新语言真的还得再熟悉，不过这个 discuss 本身难度不高