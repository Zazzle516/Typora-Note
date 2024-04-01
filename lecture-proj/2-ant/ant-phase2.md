# Ants!

为蚂蚁增加不同类型，当完成不同类型的蚂蚁之后，需要把 `implemented` 设置为 `True` 

（如果游戏太难了，可以多增加蚂蚁类型来获胜	看好了，蜜蜂，来自三次元的攻击.jpg

实现完成后，把 `implemented` 设置为 `True` 才可以在 GUI 显示

### Problem 4

```shell
python ok -q 04 -u --local
```

实现豌豆射手的两个子类，花销更少但是攻击距离会有限制

- LongThrower：只能攻击距离自己5个格子远的蜜蜂
- ShortThrower：至多攻击自己前方3个格子内的蜜蜂

相对于最传统的豌豆射手，食物消耗 - 1，两个子类的索敌逻辑不变，只是多了最远和最近范围，实现结束注意保证 Problem3 正常工作，注意这里的实现有指定属性名 `max_range` 和 `min_range` 否则不通过

> The closest Bee in front of it within range

在 `ThrowerAnt.nearest_bee()` 中增加一个参数 `range` 然后子类去调用父类的方法，传入自己参数

通过计数的方式 place_count，在循环中找到自己的范围，现在的问题是怎么判断收到的 range 是最大或者最小，因为对应着不同的区间

```shell
python ok -q 04 --local
```

```shell
python ok -q 03 --local
```

面向测试编程之后我终于改对了😭，真无语	我是把两个子类拆开实现的，但是应该可以合并的，我再去试一试

可以的，没问题，这样就可以认为完全过了



### Problem 5

```shell
python ok -q 05 -u --local
```

当炸弹蚁受到伤害的时候（有点像土豆地雷），才会造成伤害，比如，受到了 X 点伤害，会对处于与它一个位置的所有蜜蜂造成相同的 X 点伤害

如果承伤到死亡，还会额外造成默认伤害（3），发生在被移除出位置之前

总之就是先获取当前同一个位置内的所有蜜蜂，然后造成 `amount` 伤害

```shell
>>> place = gamestate.places['tunnel_0_4']
>>> fire = FireAnt(armor=1)
>>> place.add_insect(fire)        # Add a FireAnt with 1 armor
>>> place.add_insect(Bee(3))      # Add a Bee with 3 armor
>>> place.add_insect(Bee(5))      # Add a Bee with 5 armor

>>> place.bees[0].action(gamestate)
```

```shell
python ok -q 05 --local
```



不明白为什么直接写 `Insect.reduce_armor` 不可以，=> 要调用 Ant 因为测试用例里面是通过 Ant 去修改的

```shell
Ant.__init__ = lambda self, armor: print("init") #If this errors, you are not calling the parent constructor correctly.
```

不能跨越自己的父类去找更上层

注意不能在计算伤害才获取当前格子内的蜜蜂，因为炸弹蚁死了之后就会被移除当前的 place ，无法通过该 place 再去找蜜蜂

用到数组拷贝的原因：因为当蜜蜂受到攻击死亡之后，会调用 `remove_from()` 方法，这个时候会改变可迭代对象

对蜜蜂的攻击要用已经存在的函数，注意 abstraction barrier



### Problem 6

```shell
python ok -q 06 -u --local
```

食人蚁，消耗 3 回合进行消化，从当前位置随机挑选一个蜜蜂吃掉

```shell
python ok -q 06 --local
```

这个问题比想象的简单



### Problem 7

```shell
python ok -q 07 -u --local
```

不爆点数的投锋，伤害所有经过的蜜蜂，但是不阻止它们（也无法被攻击

> When there is an Ant whose blocks_path attribute is True in the Bee's place

```shell
python ok -q 07 --local
```

改了半天的 `Bee.block()` 判断终于改对了，累死我了



# Phase2 Pass!

还行，完成了[]~(￣▽￣)~*



