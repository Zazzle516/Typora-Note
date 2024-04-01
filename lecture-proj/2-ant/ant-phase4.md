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

有点没看懂。









