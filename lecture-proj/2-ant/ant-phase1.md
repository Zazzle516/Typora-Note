# Ants Vs. Bees

介绍说是个塔防游戏，参考植物大战僵尸实现，这里蚂蚁的攻击方式居然是树叶，好脆弱的攻击

**文件结构**：

- ants.py：蚂蚁大战蜜蜂的游戏逻辑，同时也是项目完成文件
- ants_gui.py：游戏界面
- graphics.py：放置二维动画（gif）的一些工具
- utils.py：一些用来进行游戏的函数接口
- ucb.py：对于这门课的一些工具函数
- assests：被 gui.py 使用的图像，文件的目录

**项目 Debug** 

```python
print("DEBUG:", x)
```

### Game

蜜蜂有点像弹跳僵尸，可以前往任意一列进行攻击，然后，可以选一个蚂蚁类型摆放到对抗路上

不同类型的蚂蚁和蜜蜂都有自己的行动逻辑

#### Game Layout

对抗路：路上有一些食物可以被扩展为部署蚂蚁军队

对抗路中的每个格子只可以放置一只蚂蚁，但是同一个地方可能会有很多蜜蜂

不同类型的蚂蚁有自己的行为，两种最基本的蚂蚁类型是 `HarvesterAnt` 负责在每轮中为领地增加食物（向日葵），`ThrowerAnt` 每轮扔出叶子（豌豆射手）

#### Game Start

这个游戏有两种运行方式，基于文本的和基于图形页面的，基于图形页面的运行方式相对于基本文本的方式增加了决策时限，文本方式主要用于 Debug

```shell
# text based
python ants_text.py

# GUI based
python ants_gui.py
```

（这个游戏居然还有难度设置... 

#### Game Info

```python
usage: ants_text.py [-h] [-d DIFFICULTY] [-w] [--food FOOD]

Play Ants vs. SomeBees

optional arguments:
  -h, --help     show this help message and exit
  -d DIFFICULTY  sets difficulty of game (test/easy/medium/hard/extra-hard)
  -w, --water    loads a full layout with water
  --food FOOD    number of food to start with when testing
```



## Phase1

### Problem 0

```shell
pyth0on ok -q 00 -u --local
```

阅读核心类的代码然后回答问题，这里的 `armor` 应该是血条的意思，`Place` 就是那一个个小格子

**Q1**：What is the significance of an insect's `armor` attribute? Does this value change? If so, how?

生命值属性，判断当前的蚂蚁或者蜜蜂是否已经死亡，如果死亡那么应该移出游戏场景，通过 `reduce_armor` 函数改变，根据攻击值 `amount` 来减少

**Q2**：What are all of the attributes of the `Insect` class?

`Class-attribute: damage` ，`Instance-attribute: armor, place` 

**Q3**：Is the `armor` attribute of the `Ant` class an instance attribute or class attribute? Why?

Instance attribute 因为是在 \__init__ 函数中声明的

**Q4**：Is the `damage` attribute of an `Ant` subclass (such as ThrowerAnt) an instance attribute or class attribute? Why?

不同的 Ant 实例在使用时可以设置 `damage` 此时是 instance attribute

**Q5**：Which class do both `Ant` and `Bee` inherit from?

Insect

**Q6**：What do instances of `Ant` and instances of `Bee` have in common?

因为继承了相同的父类，`damage`，成员函数之类的，都是共同的

**Q7**：How many insects can be in a single `Place` at any given time (before Problem 9)?

一共格子只能放一个蚂蚁但是一个格子可以放很多蜜蜂

**Q8**：What does a `Bee` do during its turn?

`Bee.action()` 朝着目标地移动，如果被蚂蚁挡住，那么攻击蚂蚁

**Q9**：When does the game end?

所有蜜蜂被消灭，有蜜蜂到达终点或者蚁后被杀



### Problem 1

```shell
python ok -q 01 -u --local
```

```shell
python ok -q 01 --local
```

目前的蚂蚁放置没有任何食物开销，需要重写 `Ant.food_cost` 属性，添加 `class attribute` 实现收获蚂蚁类型，每次增加一点食物

注意子类重写父类的类属性，直接添加类变量就可以了

```python
class HarvesterAnt(Ant):
    # OVERRIDE CLASS ATTRIBUTES HERE
    food_cost = 2
```



### Problem 2

```shell
python ok -q 02 -u --local
```

目前，`Place` 类只是追踪出口，现在同样需要追踪入口，一个 `Place` 实例只需要追踪一个入口，

这个功能在一个蚂蚁想要知道是否有一个蜜蜂在它的前面是非常重要的

但是，我们必须在创造 `Place` 之前就有入口和出口，为了解决这个问题，我们通过以下的方式来追踪入口

- new Place 通过 entrance 和 None 创建
- 如果有 exit 那么设置

因为蜜蜂是一格一格前进的，所以后一个格子的构造依赖于前一个格子，如果是从蚂蚁的角度，那就是从左到右

```shell
ant exit<- bee <- entrance
0			1			2
```

```shell
python ok -q 02 --local
```

还是挺巧妙的



### Problem 3

```shell
python ok -q 03 -u --local
```

为了让豌豆射手攻击，必须知道应该攻击哪个蜜蜂，目前的实现只允许蚂蚁攻击自己所处格子的蜜蜂，现在需要让这个蚂蚁实现远程攻击（不能攻击蜂巢里的蜜蜂）如果那个格子里有很多蜜蜂，那么随机攻击一个

结合 Problem2 实现了 Place 构造方式，在路径上寻找可攻击的蜜蜂，并且当前的蚂蚁不能攻击身后的蜜蜂

`nearest_bee` 搜索最近的蜜蜂，`action` 攻击最近的蜜蜂

要想实现的话，需要查看 Hive 类，作为判断的标准很重要~

```python
class Hive(Place)
	def __init__()
    	# 返回 bees 列表
```

如何查看自己的前方是否有蜜蜂，可以通过循环往前找下去，但是不知道终点有什么特殊表示吗

```shell
python ok -q 03 --local
```

注意在向前查找的过程中，直接修改会永久性改变实例的属性，要先赋值给一个变量，用这个变量去遍历

```shell
# 判断目前找到蜜蜂是否是蜂巢中的蜜蜂 需要 type() 进行类的判断
python ok -q 03 --suite 2 --case 2 --local
```



# Phase1 Pass!

这个时候可以用可视化界面玩一玩

```shell
python ants_gui.py --food 10
```