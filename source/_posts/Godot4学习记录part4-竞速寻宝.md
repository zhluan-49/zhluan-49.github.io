---
title: "Godot学习-竞速寻宝"
date: 2025-03-16
categories: 工具
tags: Godot学习
excerpt: "竞速寻宝"

thumbnail: https://blog.zhluanlog.xyz/images/simage/Godot1.jpg

---


我们这次要使用之前所学的知识做个简单的小游戏，要实现的功能有**1.地图制作 2.随机生成物品 3.角色基础功能 4.开始结束判定 5.碰撞检测动画等

## 1.地图制作

首先我们新建个2d场景，添加个tilemap节点，然后在右侧属性栏新建tileset，随后拖入一张图块集
   ![](/images/blog/jingsu_1.jpg)

完成后，我们需要在tileset内，新建几个地形以用于我们不同地形的绘制，我们就简单绘制三个：land/grass/sea，点击tileset中的地形绘制，选择land/grass/sea开始绘制，值得一提的是我们要将terrain sets中的mode改为match corners
   ![](/images/blog/jingsu_2.jpg)
   ![](/images/blog/jingsu_3.jpg)
   ![](/images/blog/jingsu_4.jpg)
   ![](/images/blog/jingsu_5.jpg)
   ![](/images/blog/jingsu_6.jpg)

我们可以选择tilemap中地形画grass与sea地形
   ![](/images/blog/jingsu_7.jpg)

接下来是land，这里需要注意下，我想要将land放入grass下方，如何改变层级，这里就需要用到Layers的功能，新建一个layers，将z index设为-1，同时在画land时，将图层改为我们新建的这个layer2，这时的layer2就可以放置到layer1下了，由于grass与sea都放置在layer1的图层内，因此就可以将land放置于grassyusea下。
   ![](/images/blog/jingsu_8.jpg)
   ![](/images/blog/jingsu_9.jpg)

画完land，再删除部分grass，就可以得到小路
   ![](/images/blog/jingsu_10.jpg)

## 2.随机生成物品
我希望在这个池塘内可以生成石头，但他们的位置随机，如何通过代码实现呢，首先引入三张石头图片，然后我们新建三个场景，通过sprite将石头精灵化。
   ![](/images/blog/jingsu_11.jpg)

新建个tilemap用来放置石头，还是新建tileset放置图片，再建个terrains，这里我们仅需要一个区域显示就可以，因此画terrain就随便选个色块即可，画个新地形在池塘范围内，做好这些，接下来就可以写代码来控制了。首先在我们地图这个父节点新建脚本。
   ![](/images/blog/jingsu_12.jpg)
   ![](/images/blog/jingsu_13.jpg)

首先需要思考**①获取随机石头 ②将石头放置于mask地图上 ③随机位置 ④隐藏mask


### ①获取随机石头

首先我们需要建个石头数组，再建个随机数来获取数组内石头并实例化即可。
   ![](/images/blog/jingsu_14.jpg)

### ②将石头放置于mask地图上

首先先使用for循环遍历mask所有使用的单元格以用于依次放入，然后通过add_child将石头放入节点下
   ![](/images/blog/jingsu_15.jpg)

### ③随机位置
如何随机位置？首先石头大小肯定不能超过单元格大小，并且位置也不能超过单元格位置，通过下图可以更直观观察到tile_size - rock_size就是我们可以随机的区域，只要我们随机数在0-1之间，就可以实现在单元格内随机位置的功能
   ![](/images/blog/jingsu_16.jpg)
   ![](/images/blog/jingsu_17.jpg)
   ![](/images/blog/jingsu_18.jpg)

最终效果如上图

再讲下物理层，它可以批量生产同样碰撞形状，如果你有很多的树，那么添加physics layers，并在tileset中的物理层进行处理
   ![](/images/blog/jingsu_19.jpg)
   ![](/images/blog/jingsu_20.jpg)

选中树，绘制碰撞区域即可附给所有使用该图块的物体，想用这个碰撞形状给其他图块，仅需点击给其他图块即可（将多个图块合并为一个图块：仅需在tilset中删除除第一个以外的图块，再将第一个图块拉大即可）
   ![](/images/blog/jingsu_21.jpg)
   ![](/images/blog/jingsu_22.jpg)
   ![](/images/blog/jingsu_23.jpg)

## 3.角色基础功能

新建场景CharacterBody2D，子节点sprite2d，将这张图导入，可以看到这是一张序列帧图片，因此我们需要在sprite2d的属性内调整animation，来让他处于默认状态
   ![](/images/blog/jingsu_24.jpg)
   ![](/images/blog/jingsu_25.jpg)

搞定之后，我们需要加上动画，可以看到我们的序列帧是有下、右下、右、右上、上五种动画方向状态的，还缺少另外三个需要我们通过filp_h翻转来实现，首先通过判断方向来实现不同角度，我认为有两种方法可以实现，一种是通过建动画状态机中不同角度的动画以及动画树来实现，也可以通过简单的代码来实现，那我们先从简单的开始。
首先建个动画的字典，用于到时候判断方向调用
   ![](/images/blog/jingsu_26.jpg)

首先要将方向给整数化round()函数，其次字典内没有负数，所以我们要将人物左移动的负值给绝对值abs()函数，那这样不是按左键，但动画是右边吗？这里我们还需要一个sign()函数，它可以将负值返回-1，正值返回1，因此就可以用来判断flip_h是true还是false了，一个带转向动画的角色功能就完成了
   ![](/images/blog/jingsu_27.gif)

接下来再加点待机动画增加些真实感，新建个animationplayer，新建个动画，第0帧将初始位置设为关键帧，随后在0.5时，高度设为50，再设为关键帧，最后设置为循环，于此同理，我们将阴影的大小同样做动画，来随着人物上升而缩小，一个简单的动画就完成了。
   ![](/images/blog/jingsu_28.jpg)
   ![](/images/blog/jingsu_29.gif)

## 4.开始结束判定

我们要通过一些ui及代码控制玩家何时开始游戏，以及如何胜利或失败


### ①开始

我们想要屏幕中间有个321的倒计时，右上角有个游戏倒计时
那么首先需要新建个CanvasLayer画布，再建个label用于倒计时，我们需要将这两个参数设为center来让文本居中，然后再layout中将anchors preset改为整个矩形，再修改下font sizes调整
   ![](/images/blog/jingsu_30.jpg)
   ![](/images/blog/jingsu_31.jpg)

我们需要个倒计时动画，因此再新建个animationplayer，我们需要改变文本的text与scale，因此将这两个创建关键帧动画
   ![](/images/blog/jingsu_32.jpg)

可以看到再0，1，2秒时 文本的变化应该为3，2，1。scale的变化应该都为1最大，在前一帧0.9,1.9,2.9时为0来达到缩放效果，但是这是播放会发现动画是以左上角为锚点缩放的，因此我们还需要将锚点改为中间值
   ![](/images/blog/jingsu_33.jpg)
   ![](/images/blog/jingsu_34.gif)

一个简单的开始动画就完成了，但在倒计时玩家应该不能动才对，因此我们还需要在动画内增加函数
首先创建脚本，现在_ready()函数内添加角色不能动的脚本，再新建start()函数来让角色可以动，随后在动画结束时调用这个可以动的函数即可
   ![](/images/blog/jingsu_35.jpg)

在动画添加轨道内选call method track，随后在3秒位置右键insert key，选择我们的父节点，选择start()函数，这样就实现了动画结束角色才可移动。
   ![](/images/blog/jingsu_36.jpg)
   ![](/images/blog/jingsu_37.jpg)

### ②结束

我们需要一个计时器来倒计时，当倒计时结束玩家未找到npc，就判定失败，结束前找到npc就判定胜利。
新建个label用于右上角放置时间，一个timer用于倒计时
```
extends Node2D

@onready var player = $player
@onready var timer = $Timer
@onready var timelabel = $CanvasLayer/timelabel


# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	player.set_physics_process(false)
	timelabel.text = get_time(timer.wait_time)
	set_process(false)


func start():
	player.set_physics_process(true)
	timer.start()
	set_process(true)

func get_time(time):
	return str(time).pad_decimals(2).pad_zeros(2)

func _process(delta: float) -> void:
	timelabel.text = get_time(timer.time_left)
```
我们使用get_time()函数来设定倒计时格式（小数点前、后2位），再通过_process函数来实时改变文本，最后通过start()函数来启动倒计时
   ![](/images/blog/jingsu_38.gif)

这里我们先写下，当游戏完成时（胜利或失败）计时与玩家都会暂停
   ![](/images/blog/jingsu_39.jpg)

新建个area用于检测玩家进入，需要注意的是area需设置为只能与玩家进行交互，因此将collision中mask设置为2，player中的layer设置为2，这样area就只能检测到player的collision是否进入了。
   ![](/images/blog/jingsu_40.jpg)
   ![](/images/blog/jingsu_41.jpg)

随后新建一个场景用于放置胜利失败后的展示ui
   ![](/images/blog/jingsu_42.jpg)

同时，建好胜利失败不同texture及text的字典及函数以供调用。
```
extends PanelContainer

@onready var texturerect = $HBoxContainer/TextureRect
@onready var label = $HBoxContainer/Label
@onready var button = $HBoxContainer/Button

var dialogue :={
	0:{
		"text" : "You Lost",
		"expression" : preload("res://text/fail.png") 
	},
	1:{
		"text" : "You Win",
		"expression" : preload("res://text/success.png")
	}
}

func show_line(number):
	texturerect.texture = dialogue[number]["expression"]
	label.text = dialogue[number]["text"]
	button.connect("pressed",Callable(get_tree(),"quit"))
```
将我们做好的ui实例化到主场景内，在主场景脚本中，当结束游戏时，ui的visible为ture，默认为false。并且调用ui中的函数show_line并赋值参数0，判断剩余时间>0时，则参数为1
```
extends Node2D

@onready var player = $player
@onready var timer = $Timer
@onready var timelabel = $CanvasLayer/timelabel
@onready var paneltext = $CanvasLayer/paneltext
@onready var area = $Area2D

func _ready() -> void:
	player.set_physics_process(false)
	timelabel.text = get_time(timer.wait_time)
	set_process(false)
	area.connect("body_entered",Callable(self,"enter_area"))
	timer.connect("timeout",Callable(self,"finish_game"))


func start():
	player.set_physics_process(true)
	timer.start()
	set_process(true)

func get_time(time):
	return str(time).pad_decimals(2).pad_zeros(2)

func finish_game():
	player.set_physics_process(false)
	set_process(false)
	paneltext.visible = true
	paneltext.show_line(0)
	var play_win = timer.time_left > 0.0
	if play_win:
		paneltext.show_line(1)
	timer.stop()

func enter_area(body):
	finish_game()

func _process(delta: float) -> void:
	timelabel.text = get_time(timer.time_left)
```
   ![](/images/blog/jingsu_43.gif)
这样开始及胜利失败的功能就完成了。最后添加下多边形碰撞体来放置玩家去到其他区域
   ![](/images/blog/jingsu_44.jpg)

基础的功能就完成了

## 5.特殊机制

我们想让关卡更有点趣味性，当玩家踏入特定区域时，会有机关启动
我们新建个场景用于储放障碍
   ![](/images/blog/jingsu_45.jpg)

area2d用于检测玩家进入触发，我希望玩家进入后，基础触发会推开玩家，因此再新建个animationplayer来制作推得动画
   ![](/images/blog/jingsu_46.jpg)
```
extends Node2D

@onready var animationplayer = $AnimationPlayer
@onready var area2d = $Area2D


func _ready():
	area2d.connect("body_entered",Callable(self,"_on_Area2D_body_entered"))
	
func _on_Area2D_body_entered(body):
	animationplayer.play("pull")
```
简单写下玩家进入区域后会播放动画的脚本。
   ![](/images/blog/jingsu_47.gif)
一个小陷阱就完成了。