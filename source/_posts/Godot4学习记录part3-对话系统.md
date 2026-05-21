---
title: "Godot学习-对话系统"
date: 2025-02-10
categories: 工具
tags: Godot学习
excerpt: "对话系统"

thumbnail: https://blog.zhluanlog.xyz/images/simage/Godot.jpg

---


实现一个简单的对话系统

## 一、准备
关于剧情游戏，我要实现的功能有**1.可交互按钮 2.可切换文本及图片3.打字机效果等
知识点运用：**1.字典的运用 2.代码增加信号 3.富文本及TWEEN节点 4.代码创建节点等

## 二、场景准备
还是先原型图自己简单搭一个，根据这个建场景
   ![](/images/blog/duihua_1.jpg)
   ![](/images/blog/duihua_2.jpg)

## 三、实现按钮更新文本
首先我们来显示所有的内容，先随便搞个字典来放置对话文本
   ![](/images/blog/duihua_3.jpg)

和之前的一样，先想下，1.需要增加新按钮节点 2.我们需要两个函数来获取按钮及对话框的文本 3.需要调用函数来展示

### 1.增加新节点
这里需要用到new()函数来创建新的节点* var buttons = Button.new()* 这样我们再将buttons利用add_child()放置到我们的VBoxContainer下就可以了。

### 2.获取文本函数
文本框的很简单，直接通过@onready来获取文本框节点，再.text = 参数即可，需要注意下按钮文本，因为我们按钮文本也是需要分别出现的，所以也是以字典的形式，因此参数类型为字典，需要使用for循环来遍历，*for text in button_group*，再将参数赋值给buttons.text即可。
   ![](/images/blog/duihua_4.jpg)

### 3.展示函数
新建show_line函数，参数为id，id就相当于检索字典内顺序，再通过检索key，就可以获取我们想得到的，例如dialogue[id]["text"]就是对应id的文本，我们再调用函数，进行传参，展示文本
   ![](/images/blog/duihua_5.jpg)
最后利用_ready()函数 输出show_line(0)就可以展示第一个内容了
   ![](/images/blog/duihua_6.jpg)

### 4.更新文本
展示做好了，就可以继续如何按钮更新文本了，先随便搞点剧情，注意要在按钮字典key后加入想要跳转的文本
   ![](/images/blog/duihua_7.jpg)

我们需要通过代码获取这个你想要跳转的id，通过key可以获取，我们之前的text就是按钮字典中的key，通过button_group[text]就可以获取button_id，随后通过信号连接，当按钮点击时，调用show_line函数，参数为button_id，即可通过索引玩家点击的key到对应的id，由此实现按钮更新文本功能。但是点击会发现原先的按钮还存在，因此我们还需要删除多余的按钮，我们需要遍历button_column的子节点，来删除按钮
   ![](/images/blog/duihua_8.jpg)
   ![](/images/blog/duihua_9.gif)
### 5.打字机效果
我们想要文字呈现打字机动画该怎么办？那我们需要使用tween来实现轻量化动画，首先先在节点下创建个tween，*var tween = get_tree().create_tween()*，之后我们就可以使用tween下的函数，使用*tween_property(object: Object, property: NodePath, final_val: Variant, duration: float)*，这个方法会将 object 对象的 property 属性在初始值和最终值 final_val 之间进行补间，持续时间为 duration 秒，而tabel中的vusible_ratio可以控制显示文字占比来达成打字机效果，我们可以默认设为0，速度我们可以通过文本的长度/固定数值来实现，需要注意的一点是当vusible_ratio值通过tween_property变为1后，之后更新文本也还是1，所以需要再将值变为0，这样就实现了打字机效果。
   ![](/images/blog/duihua_10.jpg)
   ![](/images/blog/duihua_11.jpg)
   ![](/images/blog/duihua_12.gif)

### 6.更新texture
通过上面的gif，可以看到我更新了texture，实现的方法其实大同小异，就是通过索引我们字典中的KEY：expression，获得后面的字符串，再通过简单的if函数判断实现更换texture操作，当然不要忘记在show_line中调用。
   ![](/images/blog/duihua_13.jpg)

### 7.增加退出按钮
我们想实现判断按钮字典中无内容时，出现退出按钮。首先是创建退出按钮，通过Button.new()创建按钮，利用信号，当其被点击时，退出。
   ![](/images/blog/duihua_14.jpg)

之后在show_line中增加判断，当button字典中没有值时，调用增加退出按钮函数
   ![](/images/blog/duihua_15.jpg)
   ![](/images/blog/duihua_16.gif)

一个简单的对话系统就完成了。
```
extends PanelContainer
@onready var text_label = $VBoxContainer/HBoxContainer/RichTextLabel
@onready var texture_rect = $VBoxContainer/HBoxContainer/TextureRect
@onready var button_column = $VBoxContainer/VBoxContainer

var dialogue = {
	0: {
		"text": "昨天的功课复习的怎么样",
		"expression": "idle",
		"buttons":
		{
			"压根就没看": 2,
			"当然都看了": 1,
		}
	},
	1: {
		"text": "很好，那你预习内容了吗",
		"expression": "happy",
		"buttons":
		{
			"当然预习了": 3,
			"什么预习？": 2,
		}
	},
	2: {
		"text": "你忘记了",
		"expression": "sad",
		"buttons":
		{
			"不，你再问我遍": 0,
			"其实我学习了": 3,
		}
	},
	3: {
		"text": "很好!",
		"expression": "happy",
		"buttons": {}
	}
}

func _ready() -> void:
	show_line(0)

func show_line(id):
	var line_data = dialogue[id]
	set_text(line_data["text"])
	set_texture(line_data["expression"])
	for button in button_column.get_children():
		button.queue_free()
	if line_data["buttons"]:
		add_button(line_data["buttons"])
	else:
		add_quit_button()

func add_button(button_group):
	for text in button_group:
		var buttons = Button.new()
		button_column.add_child(buttons)
		buttons.text = text
		var button_id = button_group[text]
		buttons.pressed.connect(show_line.bind(button_id))

func add_quit_button():
	var quit_button = Button.new()
	quit_button.text = "Quit"
	button_column.add_child(quit_button)
	quit_button.connect("pressed", Callable(get_tree(), "quit"))

func set_text(new_text):
	text_label.bbcode_text = new_text
	var tween = get_tree().create_tween()
	var duration = text_label.text.length() / 60.0
	tween.tween_property(text_label,"visible_ratio",1.0,duration)
	text_label.visible_ratio = 0.0


func set_texture(new_texture):
	if new_texture == "idle":
		texture_rect.texture = preload("res://picture/zhangyu_idle.png")
	elif new_texture == "happy":
		texture_rect.texture = preload("res://picture/zhangyu_happy.png")
	elif new_texture == "sad":
		texture_rect.texture = preload("res://picture/zhangyu_sad.png")
```