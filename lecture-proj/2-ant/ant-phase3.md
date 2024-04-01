## More Ants!

我现在得了一种看到蚂蚁就会死的病

为蚂蚁们上防御，（骨骼装甲，小子



### Problem 8

```shell
python ok -q 08 -u --local
```

这里土豆蚁的生命值要在 \__init__ 中初始化，不能作为 `class attribute` 直接赋值

```shell
python ok -q 08 --local
```



### Problem 9

```shell
python ok -q 09 -u --local
```

花盆蚁，在游戏的过程中只有外面的花盆蚁会被暴露出来4

```shell
python ok -q 09 --local
```

> By hiding the ant from Bees and allowing it to perform its original action

```python
>>> place.add_insect(test_ant)
>>> place.add_insect(bodyguard)
>>> gamestate.remove_ant('tunnel_0_0')
>>> place.ant is test_ant
False

# Error: expected
#     True
# but got
#     False

# python ok -q 09 --suite 3 --case 5 --local
# 现在的问题是这个 remove_ant() 不知道从哪入手
```

其实是添加花盆蚁的时候出了问题，没有把内部的蚂蚁隐藏起来，直接去套用 `self.place.ant = self` 会出现递归循环，可以确定就是 `Ant.add_to()` 的问题

刚刚是脑子不清楚，赋值 `self` 😶‍🌫️，虽然我做到现在精神快崩溃了，这题总算是结束了



### Problem 10

```shell
python ok -q 10 -u --local
```

攻击就是最好的防御（确实），为上面实现的花盆蚁添加攻击属性，每轮对当前格子内的所有蜜蜂造成 1 点伤害

```shell
python ok -q 10 --local
```

因为坦克蚁里面也隐藏了其他类型的蚂蚁，所以这里的攻击是合计攻击

```python
>>> place.add_insect(tank)
>>> place.add_insect(harvester)
>>> gamestate.food = 0
>>> tank.action(gamestate)

# Error: expected
#     1
# but got
#     0

# python ok -q 10 --suite 3 --case 3 --local
# 这里只针对敌方进行了 action 而没有根据蚂蚁的特性进行 action
```



# Phase3 Pass!

好耶，三而竭了，先去休息一下，一会看点别的，P4明天再搞吧
