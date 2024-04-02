## Water and Might

在最后阶段，需要给这个游戏补上最后一块拼图，引入一种新地形和新蚂蚁，包括最重要的蚂蚁：蚁后



### Problem 11

```shell
python ok -q 11 -u --local
```

引入水面地形，只有一种蚂蚁可以被部署在水面上，为了判断当前的蚂蚁是否是水面安全型，需要为 Insect 类添加一个新的变量 `is_watersafe` 默认为 `False` ，因为蜜蜂能飞，所以默认为 `True` 

保持 `abstract barrier` ，首先无论地形，都先放置蚂蚁，如果不是水面安全型的，判定死亡

```shell
python ok -q 11 --local
```



### Problem 12

```shell
python ok -q 12 -u --local
```

目前没有蚂蚁可以利用水面地形，所以现在实现一个能利用的蚂蚁类（scuba-水肺）。

```shell
python ok -q 12 --local
```

注意 action 在 Insect 中只是个未实现的抽象函数，所以该类的行为基于 `ThrowAnt` 



### Problem 13

```shell
python ok -q 13 -u --local
```

第二天好像有点看懂了，首先这个 皇后蚁 继承自 `ScubaThrower` 所以有基础的攻击行为，这个就是它的 action，在此基础之上，这个 皇后蚁 会给身后还没有伤害翻倍的蚂蚁加上伤害翻倍的 buff，但是只能存在一个 真-皇后蚁，假的皇后蚁也能放上去，只是无法造成伤害，并且在该无效行动后立刻死亡，如果皇后蚁死了，那么同样视为游戏结束，蜜蜂获胜。

```shell
python ok -q 13 --local
```

##### Q：如何分辨真假皇后蚁

第一个皇后蚁在初始化之后，把标志置为 `False` ，在这之后生成的皇后蚁根据该标志得知自己是假的

```shell
python ok -q 13 --suite 3 --case 1 --local
```

我目前是在 \__init__ 函数中调用 `ScubaThrower.init()` 之后，直接修改类中 `is_true_queen` 属性

```shell
>>> queen = ants.QueenAnt()
DEBUG: self.is_true_queen True
>>> impostor = ants.QueenAnt()
DEBUG: self.is_true_queen False
```

能看到不同，但是在后续的执行中，仍然把 `True` 判定为 `False` ，不知道为什么

*****

我去看了看答案，这个思路还是不一样的，答案其实是用一个独属于实例的变量 `self.impostor` 标记该皇后蚁是否是假皇后蚁

真的是学到了，这里，很巧妙



##### Q：如何对身后未加过 buff 的蚂蚁加 buff ，注意有些格子里面有两只蚂蚁，如何标记该蚂蚁是否 buffed

在父类 `Ant` 中添加一个属性 `buffed` 标记是否 `buffed` ，遍历所有 `place` 中的蚂蚁，进行翻倍

```shell
python ok -q 13 --suite 3 --case 4 --local
```

我现在在一个 `while-loop` 里面写了 4 个 `if-else` 还没有通过测试，yue

```shell
python ok -q 13 --suite 4 --case 4 --local
```

重新放入花盆蚁中新蚂蚁的攻击力没有被翻倍，我还没改对，我要被恶心死了

重写了一遍逻辑终于过了，我也不管了，反正过了



##### Q：真皇后蚁无法在存活情况下移动位置，假皇后蚁可以

```shell
python ok -q 13 --suite 3 --case 2 --local
```

```python
def remove_from(self, place):
    if self.impostor is True:
        # Insect.remove_from(self, place)			fail
        # ScubaThrower.remove_from(self, place)		pass
        super().remove_from(place)
    else:
        pass
```

为什么被注释掉的写法就不行，调错方法了，应该调用 `Ant` 应该也行，调用 `Insect` 就不行



### Extra Credit

```shell
python ok -q EC -u --local
```

给蜜蜂施加 debuff 但是没有伤害的辅助蚂蚁，debuff 只能持续固定的回合数，每次只能施加给一个蜜蜂

- SlowThrower：往蜜蜂身上扔糖浆，施加减速 debuff 三个回合
- ScaryThrower：恐吓一个最近的蜜蜂，使对方调转方向（迷幻菇），如果对方碰巧在蜂巢，这个时候应该停止行动，效果持续两个回合，同一个蜜蜂无法被恐吓两次

实现内容：

- make_slow：只有在偶数回合才会生效
- make_scare：每轮都可以生效
- apply_status：一个蜜蜂可能有多种状态，每个状态都有序的改变行为，在应用所有的状态之后，会把自己从列表移除，函数的功能是把 `make_slow` 和 `make_scare` 两个状态添加到末尾



```shell
python ok -q EC --local
```

说实话这个我也没太想到要怎么做...好烦啊，好想看答案😶‍🌫️，现在处于无从下手的状态



##### Q：在示例中能看到游戏模拟的推进是通过调用 action 进行，如何选取攻击蜜蜂的

```python
# scare.action(gamestate)
# self.throw_at(self.nearest_bee(gamestate.beehive))
```

在两个辅助蚁的父类 `ThrowerAnt` 中有 `action()` 在里面调用了 `throw_at(nearest_bee())` 的成员函数，确定目标，所以这个 `target` 本质是 `ThrowerAnt.nearest_bee()` 的返回值，获取到当前的攻击目标

其中在 `SlowThrower` 中重写了 `throw_at` 方法，所以先执行 `ThrowerAnt.action()` 中调用的 `throw_at()` 函数时，会转到重写的 `SlowThrower.throw_at()` 方法，在 Debug 中也能看到 First，Second 顺序



##### Q：如何实现方向改变

在 `Bee.action()` 进行修改，这里只针对 `Scary buff` 进行了修改，因为 `Slow buff` 只是暂停了当前蜜蜂的行动，等效于直接 `pass` 不需要任何改动，蜜蜂的攻击行为由 `Bee.sting()` 实现，与 `Bee.action` 无关

蜜蜂的行为是每个游戏回合都要调用的，buff 的信息在蚂蚁上，所以这里提供了一个全局函数 `apply_status` 在蚂蚁和蜜蜂之间传递信息，控制蜜蜂的行为，比如 `nonlocal length` 

只能更新蜜蜂的行为函数，具体的执行由 `gamestate` 调用



##### Q：make_slow 状态只有在偶数回合判定 这是怎么做到的

在子函数中直接接收了 `gamestate` 这个参数，我还是对这个调用过程不熟悉，为什么这里能直接接收这个参数，虽然说我也这么写来着，但是不确定，我猜这个和 apply_status 有关

我没有注意到 `Bee.action()` 也有要实现的代码，我还在想这个 `action` 是个什么东西，无语

配合一起看就能理解了，由 `make_slow` 进行预处理，然后再转到蜜蜂对象，根据当前的属性表行动



##### Q：各种 buff 的应用，因为各种 buff 可以叠加

替换蜜蜂的行为函数，非常非常巧妙，指望我写出来不如指望我蹦极



##### Q：同一个蚂蚁不能被恐吓两次

答案是把 is_scared_before 写在 bee 的初始化里，让它成为了一个实例变量，因为每个蚂蚁的恐吓情况是独立的，注意，这里我直接写在类变量中，是错误的



```shell
# scare_buff
>>> from ants import *
>>> beehive, layout = Hive(AssaultPlan()), dry_layout
>>> dimensions = (1, 9)
>>> gamestate = GameState(None, beehive, ant_types(), layout, dimensions)
>>> # Testing Scare
>>> scary = ScaryThrower()
>>> bee = Bee(3)
>>> gamestate.places["tunnel_0_0"].add_insect(scary)
>>> gamestate.places["tunnel_0_4"].add_insect(bee)
>>> scary.action(gamestate)
>>> bee.action(gamestate)

>>> bee.place.name # ScaryThrower should scare for two turns
? 'tunnel_0_5'
-- OK! --

>>> bee.action(gamestate)
>>> bee.place.name # ScaryThrower should scare for two turns
? 'tunnel_0_6'
-- OK! --

>>> bee.action(gamestate)
>>> bee.place.name
? 'tunnel_0_5'
-- OK! --
```



```shell
# slow_buff
>>> from ants import *
>>> beehive, layout = Hive(AssaultPlan()), dry_layout
>>> dimensions = (1, 9)
>>> gamestate = GameState(None, beehive, ant_types(), layout, dimensions)
>>> # Testing Slow
>>> slow = SlowThrower()
>>> bee = Bee(3)
>>> gamestate.places["tunnel_0_0"].add_insect(slow)
>>> gamestate.places["tunnel_0_4"].add_insect(bee)
>>> slow.action(gamestate)
>>> gamestate.time = 1
>>> bee.action(gamestate)

>>> bee.place.name # SlowThrower should cause slowness on odd turns
? 'tunnel_0_4'
-- OK! --

>>> gamestate.time += 1
>>> bee.action(gamestate)

>>> bee.place.name # SlowThrower should cause slowness on odd turns
? 'tunnel_0_3'
-- OK! --

>>> for _ in range(3):
...    gamestate.time += 1
...    bee.action(gamestate)
>>> bee.place.name
? 'tunnel_0_1'
-- OK! --
```



### Optional Problem

```shell
python ok -q OPTIONAL -u --local
```

看了 EC 我觉得我这个也大概率没戏2333

黑白通吃，激光攻击，伤害衰减，自爆战士



```shell
python ok -q OPTIONAL --local
```

欸嘿，没想到做出来了，这个比上一个问题简单多了2333



# Phase4 Pass!

好耶，终于进入 Chap3









