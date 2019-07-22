---
layout:            post
title:             "Excel Tips"
menutitle:         "Excel Tips"
date:              2018-09-04 22:40:00 +0200
category:          Tips
cover:             /assets/img/excel-1.jpg
published:		   true
author:            mkimish
tags:              Tips on Excel
language:          EN
comments:          true
math:			   true
---

Sometimes I have to do some Excel operations. They say it is not good to show that you know Excel on your resume, but I do think sometimes it is handy :)

### Functions / 函数
#### 线性插值`TREND`
`trend([known_y], [known_x], new_x)` performs (least square) line fitting on `known_y` and `known_x` as $$y=ax+b$$ and predicts the $$y$$ value of `new_x`.

### Shortcuts and quick settings / 快捷键及快速设置
* 重新计算所有公式, recalculate all formulae: `Ctrl+Alt+Shift+F9`.
* 多列/行自动列宽/行高, automatic column width (or row height) for multiple columns (rows): `Home->Cells->Format->Click`!

### Charts / 图表
#### 制作带阴影的折线图
- 要求：给两条折线A,B中间夹的部分绘上阴影。
- 做法：
	1. [百度经验](https://jingyan.baidu.com/article/046a7b3ec26a44f9c27fa991.html)。用A,B画面积图（而不是折线图），将下方的折线（如A）对应面积图的填充颜色设为白色（或背景色），就可以得到阴影图。 **问题：**如果背景有其他图表则会覆盖住其他图表。
	2. 百度经验里也有…百度还是有点用的。 [English](http://www.illuminateandenumerate.com/2013/11/how-to-fill-shade-between-lines-in.html) 利用堆积面积图，首先计算B-A，而后用A和这个差值制作堆积面积图，而后将A代表的面积块的填充色设为透明即可。需要较新的Excel版本。

(Background image credit: <a href="https://www.youtube.com/watch?v=6_yylg0J-14" target="_blank">video</a>)