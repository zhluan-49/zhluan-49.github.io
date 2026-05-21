---
title: "Godot学习-塔防案例下"
date: 2025-03-25
categories: 工具
tags: Godot学习
excerpt: "塔防案例"

thumbnail: https://blog.zhluanlog.xyz/images/simage/Godot1.jpg

---


继续我们未完成的内容：**1.自动生成敌人并寻路 2.炮台多种功能等

## 1.自动生成及寻路功能

我们如果想要敌人按照我们的规划路径移动，首先肯定得先有个路径。那么新建个path2d，随便先画下路线，然后我们需要有个敌人生成器，新建场景-node2d-添加timer（用于计算倒计时结束出现敌人），再实例化至场景内。新建脚本。
   ![](/images/blog/tafang2_1.jpg)

在这个脚本内我们需要写什么呢，我们要实现的功能点为1.获取路径以及每个路径点的位置 2.生成敌人并给予位置目标
ok，我们先来获取路径及点位位置，利用@onready来获取path2d，并声明curve（描述 2D 空间的贝塞尔曲线，也就是path2d的形状），再声明个vector2的数组来储存各个点位的位置，这里用到的是PackedVector2Array()（专门设计用于存放vector2的数组），再通过for循环来实现遍历每个节点，get_point_position()来获取点位位置，最后再利用.to_global将其转为全局坐标。
```
extends Node2D

@onready var path_node = $"../Path2D"

var points = PackedVector2Array()

func _ready() -> void:
	var curve = path_node.curve
	for idx in curve.get_point_count():
		var point_poisition = path_node.to_global(curve.get_point_position(idx))
		points.append(point_poisition)
```
接下来就是生成敌人的函数了，新建个函数用于实现功能，首先就是实例化我们的敌人，帮将其放入场景内。但是这里的points仅在敌人生成器中有效，敌人脚本中还没有这个points，因此肯定无法实现我们的功能，因此打开我们的enemy脚本（PS：由于我们不需要拖拽进行测试了，因此将相关代码全部注释或删掉，并将类型改为CharacterBody2D，由于我们之前检测进入信号中用的是area_entered，因此也需要将area修改为body）
```
func spawn_enemy():
	var enemy = preload("res://rocket/enemy.tscn").instantiate()
	add_child(enemy)
```
首先我们需要获取到我们的敌人生成器中获取的点位位置，因此我们在这里也需要新建PackedVector2Array()，以便之后在敌人生成器脚本中调用，再将我们的enemy的全局坐标设置为获取点位的全局坐标，默认以第一个点位为目标。
```
extends CharacterBody2D

var points = PackedVector2Array()

func _ready() -> void:
	target = points[0]
	global_position = target
```
接下来就是让他移动了，我们想让敌人的位置距离第一个点位<1时，将目标设为下一个点位置，并需要利用的是（distance_to ：返回两个向量之间的距离），同时将velocity的值设为两个向量差值*速度，再move_and_slide()即可，但是两个向量的差值会随着移动而改变，这会导致我们的速度是先快后慢的，因此我们还需要用到一个normalized()来将向量缩放至单位长度，这时就可以保证我们的速度一致性。
```
func _physics_process(_delta):
	if global_position.distance_to(target)<1:
		target = points[1]
	sprite.look_at(target)
	velocity = (target - global_position) .normalized()* speed 
	move_and_slide()
```
最后在enemy脚本中的spawn_enemy()函数调用points：enemy.points = points，这样就可以实现了第一个点到第二个点的移动
   ![](/images/blog/tafang2_2.gif)

从这里看到我们实现了敌人沿着路径点移动了，接下来只要声明个路径点组的序号，并让他累加1即可，但是目前还有几个问题没有解决，第一个是敌人移动到终点不会消失，这肯定步行，我们还需要让他到终点时消失。第二个是敌人数量没有限制，我们需要作出限制。
```
func _physics_process(delta):
	if global_position.distance_to(target)<1:
		point_index += 1
		if point_index == points.size():
			queue_free()
			return
		target = points[point_index]
```
当序列到最后一个时，让它消失即可。同时在敌人生成器脚本中，增加敌人数量的变量，并每生成一个就-1，当为0时则不执行生成函数。
```
@export var enemy_amount = 10

func spawn_enemy():
	if enemy_amount == 0:
		return
	#你之前写过的
	enemy_amount -= 1
```
   ![](/images/blog/tafang2_3.gif)
这样就可以完成我们的敌人生成并寻路的功能。

## 2.其他炮台功能

仅有这两个炮台太单调了，我们再增加个可以减速功能的冰冻塔。首先还是像我们上一篇一样，将炮台新建继承场景。
再拓展我们的炮台代码，首先我希望敌人进入区域时，速度可以/2，并且颜色变为蓝色。离开区域时速度恢复且颜色恢复（这里拓展下，如果我们在拓展函数内使用同名函数增加内容，那么就是对函数的覆盖。如果我想要这个函数下的功能，那么直接复制代码可以吗？这样也是可以的，但是会有两段重复代码，如果你想要修改的话就要两个一起修改，这里最好用super()去调用我们正在覆盖的函数的父函数）
```
func _area_entered(body: PhysicsBody2D):
	super(body)
	body.speed /= 2
	body.modulate = Color.AQUA

func _area_exited(body: PhysicsBody2D):
	super(body)
	body.speed *= 2
	body.modulate = Color(1.0, 1.0, 1.0)
```
同时_ready()内也可以利用super()来调用，但是父函数有个发射函数怎么办呢，我们只需将发射函数用return覆盖即可。同时增加让他不能转动的函数。
```
func _ready() -> void:
	super()
	set_physics_process(false)

func emission_rocket():
	return
```
   ![](/images/blog/tafang2_4.gif)
最后我们就可以实现我们预期的功能了！