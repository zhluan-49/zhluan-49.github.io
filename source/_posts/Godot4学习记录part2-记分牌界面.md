---
title: "Godot学习-记分牌界面"
date: 2025-01-20
categories: 工具
tags: Godot学习
excerpt: "记分牌界面"

thumbnail: https://blog.zhluanlog.xyz/images/simage/Godot.jpg

---


实现一个可以输入并展示的用户界面

首先拆分下界面，1.输入界面 2.展示界面，需要在输入界面输入内容并储存，再放置到展示界面中

## 一、输入界面

首先新建个用户界面作为我们的主界面，那么这个界面我们需要放什么呢，两个输入框、一个按键，还有个容器可以帮助我们将控件呈列或行（V、H）摆放，因此我们添加这四个节点：HBoxContainer、LineEdit\*2、Button（自己再命名下让自己知道是什么）
   ![](/images/blog/jifen_1.jpg)
   ![](/images/blog/jifen_2.jpg)

在这里可以通过调整锚点来改变它的布局，HBoxContainer同理
   ![](/images/blog/jifen_3.jpg)

LineEdit在这里调整布局
   ![](/images/blog/jifen_4.jpg)

按照自己的需要调整后，可以填写LineEdit为空时，默认显示的文字，
   ![](/images/blog/jifen_5.jpg)

运行出来应该是这个效果，输入界面的可以先放在一边了，接下来做输出界面
   ![](/images/blog/jifen_6.jpg)

## 二、展示界面
简单用原型图画下我们想要的界面，画出来就会知道自己要哪些控件了，首先是**背景界面、标题、内容显示区域、按键**，我们就可以依次引入PanelContainer（提供背景）、VBoxContainer1（提供为后续控件排成列）、VBoxContainer2（为内容显示区域排成列）、Label（显示纯文字），为了排版，我们可以再加入MarginContainer（子控件可以保留边距）、HSeparator（分隔线）
   ![](/images/blog/jifen_7.jpg)
   ![](/images/blog/jifen_8.jpg)

至此我们两个界面就制作完成了，接下来就是如何让他们两个生效了。

首先我们要思考：1.在展示界面的VBoxContainer2显示我们的内容，因此我们要新建一个HBoxContainer来放置两个Label（显示名字与分数），再新建个函数，以供输入界面调用，因为涉及到用户数据的传递，所以需要形参来传递
2.将输出界面实例化到输入界面中，调用函数，将用户输入的内容赋值到实参中，再传递给输出界面的函数中，按我的理解画成下图，就是通过参数的传递来达成内容的输出。
   ![](/images/blog/jifen_9.jpg)

## 三、代码部分
### 1.输出界面的展示部分

和之前一样，新建HBoxContainer、Label\*2来展示内容，那么我需要新建函数来供调用，函数的功能是什么呢，就是将形参赋值给label的文本，这样实参传递给形参后，再赋值给label文本就可以显示出来了
在写之前，还有几个知识点：1.尽量用变量来指示节点，使代码简洁易懂 2.$是函数的别名get_node() 3.需要就绪状态才能访问节点，即_ready()函数，但是godot有个简单的@onready来替代
综上，我们先用变量来引用节点，再通过新建函数并讲参数赋值给节点的文本
   ![](/images/blog/jifen_10.jpg)

最后就是通过按钮来关闭输出界面，这里需要用到之前的信号，那么直接连接按下操作的信号就可以了
   ![](/images/blog/jifen_11.jpg)

使用hide()函数，隐藏起来即可
   ![](/images/blog/jifen_12.jpg)



### 2.输入界面

输入界面需要的功能是什么，就是**1.点击按钮后生效 2.调用函数，并将用户输入内容传到实参 3.显示展示界面 4.未输入内容时，无法显示展示界面
那么代码先建起来，我们的所有功能都是在点击按钮后生效，因此先把按钮的信号连接上，我们需要显示展示界面，因此展示界面tscn文件拖到主界面里来，设置为默认隐藏，以供我们代码将其显示
   ![](/images/blog/jifen_13.jpg)

我们需要将之前我们的输出界面拖入到输入界面来实现实例化，这样我们就可以调用输出界面的函数了，（我们也可以代码来实例化，后面讲）
   ![](/images/blog/jifen_14.jpg)

然后，还是变量来引用节点，节点的内容作为实参传入，然后通过if判断两个输入框文本有一个为空则返回（OR：任意一个为真，则为真），最后设置输入框默认文本为空，每次输完后清空
   ![](/images/blog/jifen_15.jpg)

效果如下图
   ![](/images/blog/jifen_16.gif)

## 四、代码实例化

我们可以看到，输入新内容时，他无法多次存储，这是因为我们只在展示界面建了一个HBoxContainer，那么多个怎么储存呢，建多个HBoxContainer？这样太麻烦了，我们只需将新建场景HBoxContainer，将其实例化，就可以一直使用了
首先删除掉我们之前在输出界面新建的HBoxContainer和两个label及对应的代码，新建场景HBoxContainer和两个label作为展示内容部分，代码还是和之前一样
   ![](/images/blog/jifen_17.jpg)

接下来就是输出部分了，我们需要新建一个函数，来实例化我们刚新建的展示部分，并加入到节点数内，再调用两个函数（那么就需要两个参数来传递了）
那么还是先讲下知识点：**1.preload() 预加载场景 2.instantiate() 实例化代码 3.add_child()将参数节点作为目标节点子节点**（我们需要将展示内容部分节点放入到VboxContainer底下，就像之前那样）
一通操作后，我们的代码应该长这样，通过add_lines函数调用两个set函数，当然，这个的前提是我们实例化并放入节点树，这样才能调用函数
   ![](/images/blog/jifen_18.jpg)

最后就是输出界面了，因为我们将两个set函数都放到一个函数中，因此输出界面仅需调用add_lines函数即可
   ![](/images/blog/jifen_19.jpg)
成果展示
这样我们终于实现了储存功能
   ![](/images/blog/jifen_20.gif)

## 五、遍历字典

再简单记录下字典用法，字典有一个keys列表，每一个key都指向一个value
   ![](/images/blog/jifen_21.jpg)

通过for循环来依次遍历，使用临时变量name来指代KEY for name in player_scores: 表示name在player_scores字典内遍历，再使用之前的add_lines()函数，需要两个参数，一个是名字，一个是分数，名字就是name变量，有了name这个KEY，就可以通过player_scores[name]来找到对应的value，再通过str()来将其转化为字符串
   ![](/images/blog/jifen_22.jpg)
   ![](/images/blog/jifen_23.jpg)
这样就完成了遍历字典并展示了。