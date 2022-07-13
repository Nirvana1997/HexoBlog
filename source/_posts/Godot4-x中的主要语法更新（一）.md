---
title: Godot4.x中的主要语法更新(一)
category:
  - godot
tags:
  - godot
  - 游戏引擎
  - 学习笔记
date: 2022-07-06 13:33:42
---

unreal engine用了一阵下来，感觉这个引擎做2d游戏确实有点太重量级了，有很多用不到的功能，甚至很多不需要的功能我都不知道怎么关闭，所以前一阵子转了Godot，使用的版本是3.4.4-stable，整体用下来，就2d游戏开发而言，体验还是很好的。不过这两天看到Godot4正在测试，正好有一些我需要的特性，因此果断下载了4.0-alpha版，3到4的脚本语法上还是有不少更新的，所以慢慢把遇到的一些总结一哈。
<!-- more -->

## 1.支持函数变量

Godot3的gd脚本语法中，不支持使用变量来保存函数，因此很多注册和绑定函数的地方都是使用字符串去函数池中匹配的，这种方式效率肯定比直接绑定函数略低，写起来也比较麻烦：

```gdscript
# Godot3:
PlayerAttr.connect("player_change_hp", self, "on_hp_change") # 注册信号player_change_hp触发self的函数on_hp_change
```

而现在有了函数变量后，只需要这么写：

```gdscript
# Godot4:
PlayerAttr.connect("player_change_hp", self.on_hp_change)
```

调用的一个函数变量对应的函数可以这么写：

```gdscript
    var my_lambda = func(x):
        print(x)
    my_lambda.call("hello")
```

## 2.setget格式变化

Godot3中重载setget函数的写法是这样的：

```gdscript
# Godot3:
var a = 0 setget set_a,get_a

func set_a(new_value):
	...
	
func get_a():
	...
```

4中形式改了下：

```gdscript
# Godot4:
# 可以直接注册函数
var a = 0 :
	set = set_a,get = get_a
# 也可以直接使用匿名函数的形式直接定义
var b = 0 :
	set（new_value):
		...
	get:
		...
```

## 3.引入注解

Godot4的gdscript引入了注解的方式，很多以前看起来比较怪的关键词，可以通过类似Java注解的方式让他们看起来合理一点，比如原来加在var前面的`onready`、`export`，比如原来加在开头比较突兀的`tool`等，现在都加了@作为注解使用。（当你项目3转4有关键词找不到报错时，就试试加个@）

## 4.加入super关键字

原本Godot3中要调用父类方法是通过`.`的形式:

```gdscript
# base.gd
func base_func():
	...

# drived.gb
func drived_func():
	.base_func()
	...
```

Godot4中引入`super`关键词后，可以直接使用super调用，更清晰一些：

```gdscript
# base.gd
func base_func():
	...

# drived.gb
func drived_func():
	super.base_func()
	...
```

然后在函数中也可以像调用父类同名函数来覆盖：

```gdscript
# base.gd
func _ready():
	...

# drived.gb
func _ready():
	super()
	...
```

有个注意事项，那就是++Godot3中，派生类的同名函数是会默认先调用一次基类的函数的，而Godot4中当你重载父类同名函数后，只有显示调用`super`才会调用父类函数++，这点修改使得gdscript更符合我们所熟悉的其他面向对象语言了。

## 5.总结

Godot4更新了非常多特性，不过有些如有类型的数组、变量初始值可以调用静态函数等，不过我暂时没用到，也感觉不太常用，等用到了再好好总结。除此之外编辑器也改了不少，比如场景内原来的按住右键拖动变为了按住中键拖动，一开始没找到方式的时候想查也很难找，后来是自己试出来的。目前Godot可能还是有很多不方便的地方吧，希望之后功能越来越好，资料能越来越多~

### 参考资料

* [Godot Engine：GDScript 4.X中语法的变化](https://blog.csdn.net/ttm2d/article/details/107818889#4X_GDScript_3)

* [GDScript progress report: Feature-complete for 4.0](https://godotengine.org/article/gdscript-progress-report-feature-complete-40)
