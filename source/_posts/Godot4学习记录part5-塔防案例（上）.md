---
title: "Godot学习-塔防案例上"
date: 2025-03-19
categories: 工具
tags: Godot学习
excerpt: "塔防案例"

thumbnail: https://blog.zhluanlog.xyz/images/simage/Godot1.jpg

---


这次我们要实现一个塔防游戏，涉及的功能点有**1.炮台自动识别攻击 2.导弹追踪 3.ai自动生成敌人并寻路 4.炮台多种功能等

开始前，我们要准备些素材，首先就是炮台、导弹、敌人对应的贴图，以及发射爆炸的粒子系统（这里我照着教程拼了个）

## 1.炮台发射功能

首先要实现的是炮台的基础功能，发射导弹。那么我们先完成炮台部分：新建一个area2d及collisionshape2d用于检测敌人是否进入，一个sprite用于放贴图，一个marker2d用于定位枪口位置，最后一个timer用于发射间隔时间。调整collisionshape2d大小及marker2d位置
   ![](/images/blog/tafang1_1.jpg)
   ![](/images/blog/tafang1_2.jpg)

接下来我们再完成炮弹部分，也是个area2d及collisionshape2d用于检测是否碰到敌人，一个sprite用于放贴图，一个GPUParticles2d用于炮弹飞行的粒子拖尾效果，最后一个timer用于飞行时长。
   ![](/images/blog/tafang1_3.jpg)

然后我们新建脚本用于炮台发射的基础功能，首先新建个发射的函数，通过实例化炮弹，再通过add_child放到场景，这时需要通过market的全局位置（global_transform：它将position、rotation和scale集中在一起）来赋值给炮弹，我们再通过信号连接，每当炮台的timer结束，就会触发发射函数即可实现一直发射。
```
@onready var market = $Sprite2D/Marker2D
@onready var timer = $Timer

func _ready() -> void:
	emission_rocket()
	timer.connect("timeout",Callable(self,"emission_rocket"))

func emission_rocket():
	var rocket = preload("res://rocket/Rocket.tscn").instantiate()
	add_child(rocket)
	rocket.global_transform = market.global_transform
```
我们再给炮弹一个初始速度值，这样就可以发射一枚炮弹，再给炮弹的增加个爆炸的函数（需注意，①爆炸的实例化添加子节点要在整个场景树下，因为炮弹在场景是不存在的是靠炮台发射出来的，如果仅用add_child()则不会显示出来 ②爆炸的全局位置要在炮弹的全局位置），在炮弹的timer结束后触发，就不会一直飞行。
```
@onready var timer = $Time

var speed = 500

func _ready() -> void:
	timer.connect("timeout",Callable(self,"explode"))

func _process(delta: float) -> void:
	position += transform.x * speed * delta

func explode():
	var explosion = preload("res://explosion/explosion.tscn").instantiate()
	get_tree().root.add_child(explosion)
	explosion.global_transform = global_transform
	queue_free()
```
这样基础的发射功能就完成了
   ![](/images/blog/tafang1_4.gif)

## 2.设置敌人基础功能及炮台自动转向
这里我们简单做个敌人，首先新建个area2d用于检测鼠标事件与被检测（**①这里我们需要将敌人的collision的layer与炮台及炮弹的mask都改为2，以便与其他的区别，否则会将所有的area2d区域全部检测到 ②我们想用拖拽先用来测试**），一个sprite2d用于放贴图，一个ProgressBar用于血条。progressbar设置theme overrides中的styles用于改变底框及覆盖层的颜色，注意到它的range中有个value，这个我们填做100（设置血量为100），新建敌人的代码。
这里通过判断点击事件是否为点击及拖拽，通过将物品的位置移动到鼠标所在位置，并且为物品的点击添加偏移，点击的地方就作为移动原点。

再通过新建个造成伤害函数，来增加ProgressBar随生命值而改变的功能，这里的amount我们作为参数，造成伤害函数要在射击函数中通过信号来调用并传参，来实现射中后生命值减少直至消失的功能。
```
@onready var progress_bar: ProgressBar = $ProgressBar

var isDrag = false
var offset = Vector2.ZERO
var health = 100

func _process(delta):
	if isDrag:
		self.position = get_global_mouse_position() + offset

func _input_event(viewport, event, shape_idx):
	if event is InputEventMouseButton and event.button_index == MOUSE_BUTTON_LEFT:
		if event.is_pressed():
			offset = self.position - get_global_mouse_position()
			isDrag = true
		else:
				isDrag = false

func take_damage(amount):
	health -= amount
	progress_bar.value = health
	if health<= 0:
		health = 0
		queue_free()
```
因此我们还需在炮弹声明一个amount为10，然后新增函数用于判断进入的area是否有take_damage()这个函数，如果有的话，我们将amount传参给它，从而实现伤害为10，我们再通过信号，当area进入时（判断的是敌人是否与炮弹碰撞），调用此函数即可。
```
var amount = 10

func _ready() -> void:
	#之前写过的
	connect("area_entered",Callable(self,"_area_entered"))

func _area_entered(area):
	if area.has_method("take_damage"):
		area.take_damage(amount)
	explode()
```
   ![](/images/blog/tafang1_5.gif)

这样可以看到敌人会血量减少直至消失
接下来我们要实现的是如何能够让炮台识别敌人进入并获取方向，从而转向炮台。
首先如果我们需要做到自动转向，肯定需要知道敌人的所在的方向，而知道敌人的方向，肯定先要判断敌人是否进入区域，因此我们新建个area2d类型的变量var target: Area2D = null，将他设置为空，他就代表着敌人，当敌人进入时，将敌人赋值给target即可，这样我们就完成了识别敌人进入了。
```
var target: Area2D = null

func _ready() -> void:
	#之前写过的
	connect("area_entered",Callable(self,"_area_entered"))
	connect("area_exited",Callable(self,"_area_exited"))

func _area_entered(area):
	target = area

func _area_exited(area):
	target = null
```
那么识别之后，我们就要开始让他转向了，首先我们想让他每帧都处理转向操作，因此是_physics_process(delta: float)函数，然后我们再声明个敌人角度变量，我们如何获取角度以使炮台能够直接运用呢，介绍个angle_to_point()函数，（他的功能是返回连接两点的直线与X轴之间的夹角），看下官方示意图，以A.angle_to_point(B)举例的话，就是A点到B的直线与X轴角度，也就是绿色扇形的角度。
   ![](/images/blog/tafang1_6.jpg)

因此我们可以使用这个函数来获取原点到target全局位置的角度，再给到我们的炮台即可。
```
func _physics_process(delta: float) -> void:
	var target_angle = PI / 2
	if target:
		target_angle = self.global_position.angle_to_point(target.global_position)
	sprite.rotation = target_angle
```
但是这样炮台转向是瞬间完成，不是很自然，因此我们还需要个lerp_angle()来进行两个角度的插值，让他可以逐渐旋转。
```
var roation_factor = 0.1

func _physics_process(delta: float) -> void:
	var target_angle = PI / 2
	if target:
		target_angle = self.global_position.angle_to_point(target.global_position)
	sprite.rotation = lerp_angle(sprite.rotation,target_angle,roation_factor)
```
   ![](/images/blog/tafang1_7.gif)
这样一个带有自动索敌并发射的炮台就完成了。但是可以看到我们的炮弹很不智能，敌人移动快的话，完全打不到。我们接下来再优化一下，让炮弹也可以自动索敌并跟踪。

## 3.炮弹索敌
我们先在我们的导弹场景右键新建个继承场景，这样我们就可以相当于实例化炮弹，将其重命名为HomingRocket，新建脚本给他。
   ![](/images/blog/tafang1_8.jpg)

这里引入个知识点，就是我们平常写脚本，前面都会带个extends XX，例如extends Area2D，那是因为Area2D是 Godot 引擎内置的类型，因此直接拓展就好，如果我们想拓展我们自己的脚本，那么直接extends "rocket.gd"自己的脚本名称就好，这样的好处在于如果有大量同类型但有些许功能区别的对象，我们可以通过拓展脚本就可以快速修改或覆盖函数。
然后获取获取角度并丝滑转向，我们之前有学过，这里直接写上
```
extends "rocket.gd"

var velocity = Vector2.ZERO
var drag_factor = 0.1

func _physics_process(delta: float) -> void:
	var direction = global_position.direction_to(target.position)
	var desired_velocity = speed * direction
	var steering_velociy = desired_velocity - velocity
	velocity +=steering_velociy * drag_factor
	position +=velocity * delta
	rotation = velocity.angle()
```
然后我们再给他一个初速度
```
func set_initial_velocity():
	velocity = transform.x * speed
但是目前还有个问题，导弹的目标应该设定有效。一旦敌人可能在被导弹击中之前被另一枚导弹杀死，在这种情况下，它们将从计算机内存中被抹去，这时候就会报错。因此我们还要加个判断目标是否有效，如果无效，则直接爆炸并退出导弹自动跟踪函数。is_instance_valid()函数是用来判断对象是否是有效。

func _physics_process(delta: float) -> void:
	if not target or not is_instance_valid(target):
		explode()
		return
	#你之前写过的
```
那我们直接将炮台脚本中炮弹实例改为这个HomingRocket实例就可以了吗，我们运行会发现导弹在刚生成出来的时候就会爆炸，我们知道我们的explode()爆炸函数仅在三个位置调用，一个是导弹与敌人接触，一个是时间到了，一个就是无有效目标，那么很显然，是因为target为null，因此我们需要设置目标以及调用初速度函数。
我们继续我们的新建炮台的继承场景，并拓展我们的炮台脚本，覆盖我们的发射函数。
```
extends "turret.gd"

func emission_rocket():
	#if not target_list:
		#return
	var rocket = preload("res://rocket/homing_rocket.tscn").instantiate()
	add_child(rocket)
	rocket.global_transform = market.global_transform
	rocket.target = target
	rocket.set_initial_velocity()
```
   ![](/images/blog/tafang1_9.gif)

左侧是正常的炮台，右侧为追踪导弹，这里可以看到实现了追踪功能，但是一直发射怪怪的，而且有个bug就是当新对象进入范围时，旧对象就会被抹除从而不会对其攻击，我们再来优化下。

我们需要记录每个进入的对象，那么肯定就需要数组来储存了。那么先声明个数组，然后当物体进入范围时，数组增加，并将目标设为第一个，当物体离开时，将他从数组中删除并判断数组是否为空，那么当数组为空时，就在发射函数中返回，不要忘记在你的追踪炮台脚本也要加入那么当数组为空时，就在发射函数中返回，因为是覆盖函数嘛。
```
var target_list = []

func emission_rocket():
	#你之前写过的
	if not target_list:
		return

func _area_entered(area):
	target_list.append(area)
	target = target_list[0]

func _area_exited(area):
	var index = target_list.find(area)
	target_list.remove_at(index)
	if target_list:
		target = target_list[0]
	else:
		target = null
```
   ![](/images/blog/tafang1_10.gif)
这个时候就可以看到，我们实现了没有目标时静止，并且将目标存在数组中等功能。
至此我们完成了炮台的基础功能，之后我们再完善后续内容。