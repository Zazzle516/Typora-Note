# Homework 08

## Streams

### Q1: Run-Length Encoding

```shell
python ok -q rle --local
```

介绍了一个非常简单的数据压缩方法，把多个重复的数值以 `(target, number)` 的形式存储起来，处理流数据

```
1, 1, 1, 1, 1, 6, 6, 6, 6, 2, 5, 5, 5
(1 5), (6 4), (2 1), (5 3)
```

不需要考虑无限的情况，题目说明最后总会以 `nil` 结束

肯定需要定义一个内部的辅助函数，判断当前读入的数据和上一次接收的数据是否相同

```scheme
(define (rle s)
  (define (helper last-elem time output-stream)
    (if (eq? (car s) last-elem)
        (helper last-elem (+ 1 time) output-stream)
        (helper (car s) 0 (cons-stream (cons last-elem (cons time nil)) output-stream))
    )
  )
  (helper (car s) 0 nil)
)
; 我的问题是没有处理流的输出和输入
; 可能是因为思考顺序的原因 只处理了流的执行
; 对比以往的函数 输入就是参数很简单 但是在流中需要通过递归的方式构建 我往往忽略掉这点
; 很多时候流的输出就是输入(还是要多见一些流的问题
```



```scheme
(define (helper curr num s)
    ; 构造流的执行部分
    ; 直到成功压缩一个数据返回结果 否则会持续递归
    (if (and (not (null? s)) (= curr (car s)))
        (helper curr (+ num 1) (cdr-stream s))  ; 递归累计数量
        (list curr num) ; 输出当前结果
    )
)

(define (next s curr)
    ; 构造流的入口 因为执行部分已经完成相同数据的压缩了 所以这里要把处理过的数据扔掉
    (if (null? s)
        nil
        (if (= (car s) curr)
            (next (cdr-stream s) curr)
            s	; 返回未处理的数据作为输入
        )
)
)

(define (rle s)
  'YOUR-CODE-HERE
  (if (null? s) nil
    (cons-stream (helper (car s) 0 s) (rle (next s (car s))))
  )
)
```

感觉流计算确实是很奇妙的执行，虽然我到现在看的例子都非常简单，但是网上没找到特别多的教程（可能确实是很少人去关注的领域



### Q2: Group By Nondecreasing

```shell
python ok -q group-by-nondecreasing --local
```

把流中升序的数据进行分组输出，和上面的题目很像欸

```
1 2 3 4 1 2 3 4 1 1 1 2 1 1 0 4 3 2 1 ...
(1 2 3 4) (1 2 3 4) (1 1 1 2) (1 1) (0 4) (3) (2) (1) ...
```

但是和上面的题不同的是，这里要对无限的输入进行支持，但是可以假设每个非降序的小组是有限的

题目提示要在 `cons-stream` 中完成所有的递归调用，否则无法支持无限情况

我仿着上一道题写为什么还不行呜呜呜呜呜

```scheme
(define (group-process curr input-stream group-ans)
    (if (null? input-stream)
        group-ans
        (if (> curr (car input-stream))
            group-ans
            ; 对比一下这里有问题 cdr -> cdr-stream
            ; 然后列表的构造可能也不太对 (car input-stream) 需要 list 套一层
            (group-process (car input-stream) (cdr input-stream) (append group-ans (car input-stream)))
        )
    )
)


(define (input-rest input curr)
    (if (null? input)
        nil
        (if (> curr (car input))
            input
            (input-rest (cdr input) (car input))
        )
    )
)


(define (group-by-nondecreasing s)
    (if (null? s)
        nil
        ; 初始 group-ans 不能是 nil
        ; 哪怕是完全降序的数列 也是单个元素作为 group-ans 输出
        (cons-stream (group-process (car s) s nil) (group-by-nondecreasing (input-rest s (car s))))
    )
)

; 之所以会出现 SchemeError: argument 0 of car has wrong type (Promise) 报错
; 没有对无限流用 cdr-stream 处理
; 是这道题相对于上面的区别
```



```scheme
(define (group-process curr input-stream group-ans)
    (if (null? input-stream)
        group-ans
        (if (> curr (car input-stream))
            group-ans
            (group-process (car input-stream) (cdr-stream input-stream) (append group-ans (list (car input-stream))))
        )
    )
)


(define (input-rest input curr)
    (if (null? input)
        nil
        (if (> curr (car input))
            input
            (input-rest (cdr-stream input) (car input))
        )
    )
)


(define (group-by-nondecreasing s)
    (if (null? s)
        nil
        (cons-stream (group-process (car s) (cdr-stream s) (list (car s))) (group-by-nondecreasing (input-rest s (car s))))
    )
)
```



## SQL

#### Dog Data

介绍数据集，包括狗狗父母的名字，狗狗自己的名字，狗狗的特点，注意一只狗狗可能是多个狗狗的父母

```
python sqlite_shell.py --init hw08.sql
```



### Q3: Size of Dogs

```shell
python ok -q size_of_dogs --local
```

选取符合贵宾犬标准的狗狗的姓名和尺寸

这个问题是根据 `dogs table` 的 `height range` 映射 `size` ，涉及到两个表之间的字段判断

在网上搜了一圈大部分是 `CASE...WHEN` 但是我觉得也许有更好的？因为涉及到范围的选取，找到了

```sqlite
-- 虽然说是过了 但是感觉不太对2333(+1 也太丑陋了)
CREATE TABLE size_of_dogs AS
  SELECT name, size
  FROM dogs, sizes WHERE height BETWEEN (min+1) AND max ;
```

看了答案也差不多，无所谓了



### Q4: By Parent Height

```shell
python ok -q by_parent_height --local
```

根据父母的身高从高到低排序，返回一个存在父母的狗狗的名称名单

```sqlite
CREATE TABLE parent_names AS
  -- only parent_name
  SELECT parent FROM dogs, parents WHERE dogs.name = parents.child;

CREATE TABLE parent_name_and_height AS
  -- parent_name parent_height
  -- 注意 DISTINCT 不然会得到 3 分相同的结果
  SELECT DISTINCT parent, height FROM parent_names, dogs WHERE dogs.name = parent_names.parent ORDER BY height DESC;

-- All dogs with parents ordered by decreasing height of their parent
CREATE TABLE by_parent_height AS
  -- child_name
  SELECT child FROM parent_name_and_height, parents
  WHERE parents.parent = parent_name_and_height.parent;
```

我这个实现是比较复杂的... 去看了看答案怎么写

```sqlite
CREATE TABLE by_parent_height AS
  SELECT ch.name from dogs as ch, dogs as pa, parents 
  WHERE ch.name = child AND pa.name = parent ORDER BY pa.height desc;
```

有点想吐血（不知道哪根筋搭错了



### Q5: Sentences

```shell
python ok -q sentences --local
```

有相同尺寸的兄弟姐妹，返回有相同尺寸的兄弟姐妹的名字，名字本身要根据字母表排序

不同行之间根据第一个名字的字母顺序进行排序，结果要输出字符串

题目提示创建一个辅助表 `siblings table` ，包含各种兄弟姐妹的组合，后续方便和尺寸信息结合起来



Q：在完成辅助函数的时候，怎么让拥有相同父母的狗狗名字出现在同一行

这个属性排序的问题，只能说是利用数据特点做到了，但是估计答案不是这么写的

```sqlite
CREATE TABLE siblings AS
  -- child-name share the same parent
  SELECT DISTINCT parent_B.child, parent_A.child 
  FROM parents AS parent_A, parents AS parent_B 
  WHERE parent_A.parent = parent_B.parent AND parent_A.child > parent_B.child
  ORDER BY parent_B.child ASC;
  
abraham|delano
abraham|grover
barack |clinton
delano |grover

CREATE TABLE sentences AS
  SELECT parent_B.child || "and" || parent_A.child "are" || size || "siblings"
  FROM siblings, size_of_dogs;
```

呃，有点想不出来呢... （但是 SQL 不至于想不出来吧

好像只能想到这里了，嗯.... 嗯，就酱(///￣皿￣)○～



看了看答案，当时在从 `siblings` 进行选取操作的时候不知道怎么才能知道选的列的名字，可以提前在 `siblings` 中通过 `AS` 定义列名

```sqlite
CREATE TABLE siblings AS
  SELECT a.child AS sib1, b.child AS sib2 FROM parents AS a, parents AS b 
  WHERE a.parent = b.parent AND a.child < b.child;
-- 这个其实写出来了
```



重点还是后面的排序和对体型的判断

我当时的卡壳是，在我要先把名字挑出来，然后再把该名字对应的体型找到，这两步其实没什么问题，问题是在要对体型查找的结果进行比较，判断相等的才能输出，不知道怎么用 `SQL` 语句进行表达

```sqlite
CREATE TABLE sentences AS
  SELECT sib1 || " and " || sib2 || " are " || s1.size || " siblings"
  -- 实际上是 3 个表(没想到这个..) 通过两个 size_of_dogs 表进行比较
  FROM siblings, size_of_dogs as s1, size_of_dogs as s2
  -- 通过对 name 和 sib1/sib2 的判断 s1/s2 已经暂存了体型结果
  WHERE s1.name = sib1 AND s2.name = sib2 AND s1.size = s2.size;
```

说起来这里的答案也没有考虑排序的事情（是 SQL 自动实现了吗



### Q6: Stacks

```shell
python ok -q stacks --local
```

返回一个含有两列的表，列出 4 个狗狗加起来超过 170cm 的排列组合和累计高度，升序输出

```
abraham, delano, clinton, barack	|171
grover, delano, clinton, barack		|173
herbert, delano, clinton, barack	|176
fillmore, delano, clinton, barack	|177
eisenhower, delano, clinton, barack	|180
```

题目提醒说不会有低于 4 只狗狗然后身高超过 170cm 的情况，也没有狗狗有相同的身高

题目也给出了辅助函数 `stacks_helper` （后面的括号是在定义列名吧）

- dogs：用于累加身高的狗狗名字列表
- stack_height：累加的总高度
- last_height：添加到狗塔的最后一只狗的高度
- n：目前狗塔中狗的数量



实现 `stacks_helper` 题目给出了一些步骤：

- 使用 `INSERT INTO` 在 `stacks_helper` 添加一只狗狗

  ```sqlite
  INSERT INTO t2 SELECT [expression] FROM t1 ...;
  ```

  ```sqlite
  CREATE TABLE t1 AS
  	-- (a b) => (1 2)
  	SELECT 1 as a, 2 as b;
  
  -- (c d)
  CREATE TABLE t2(c, d);
  
  INSERT INTO t2 SELECT a, b FROM t1;
  SELECT * FROM t2;
  -- 1|2
  ```

- 通过该方式逐渐增加狗狗数量



呃，额（不，我根本写不出 SQL....	不许写 SQL.jpg	

答案的测试结果也过不了，我看不出答案想干什么... 这个问题可能要先搁置在这里(‾◡◝)



# Done！

最后一个任务点！！！！！！虽然说是带着一点点遗憾完结了	（我从来没有觉得写 SQL 开心过_〆(´Д｀ )

开开心心去写总结，欸嘿嘿
