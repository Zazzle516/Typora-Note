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

