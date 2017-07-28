---
title: python3 常用第三方模块
date: 2016-12-09 12:10:00
categories:
- python
tags:
- python
keywords: python, python3, 常用第三方模块
---
> 
python3 常用第三方模块
**python教程推荐** [廖雪峰 - python3教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)

<!-- more -->

## pillow
<pre><code class="language-python line-numbers">PIL：Python Imaging Library，已经是Python平台事实上的图像处理标准库了

# pip install
pip install pillow

## 操作图像
来看看最常见的图像缩放操作，只需三四行代码：
from PIL import Image
# 打开一个jpg图像文件，注意是当前路径:
im = Image.open('test.jpg')
# 获得图像尺寸:
w, h = im.size
print('Original image size: %sx%s' % (w, h))
# 缩放到50%:
im.thumbnail((w//2, h//2))
print('Resize image to: %sx%s' % (w//2, h//2))
# 把缩放后的图像用jpeg格式保存:
im.save('thumbnail.jpg', 'jpeg')

其他功能如切片、旋转、滤镜、输出文字、调色板等一应俱全。
比如，模糊效果也只需几行代码：
## 模糊滤镜
from PIL import Image, ImageFilter
# 打开一个jpg图像文件，注意是当前路径:
im = Image.open('test.jpg')
# 应用模糊滤镜:
im2 = im.filter(ImageFilter.BLUR)
im2.save('blur.jpg', 'jpeg')

PIL的ImageDraw提供了一系列绘图方法，让我们可以直接绘图。比如要生成字母验证码图片：
## 生成验证码图片
from PIL import Image, ImageDraw, ImageFont, ImageFilter
import random
# 随机字母:
def rndChar():
    return chr(random.randint(65, 90))
# 随机颜色1:
def rndColor():
    return (random.randint(64, 255), random.randint(64, 255), random.randint(64, 255))
# 随机颜色2:
def rndColor2():
    return (random.randint(32, 127), random.randint(32, 127), random.randint(32, 127))
# 240 x 60:
width = 60 * 4
height = 60
image = Image.new('RGB', (width, height), (255, 255, 255))
# 创建Font对象:
font = ImageFont.truetype('Arial.ttf', 36)
# 创建Draw对象:
draw = ImageDraw.Draw(image)
# 填充每个像素:
for x in range(width):
    for y in range(height):
        draw.point((x, y), fill=rndColor())
# 输出文字:
for t in range(4):
    draw.text((60 * t + 10, 10), rndChar(), font=font, fill=rndColor2())
# 模糊:
image = image.filter(ImageFilter.BLUR)
image.save('code.jpg', 'jpeg')

如果运行的时候报错：
IOError: cannot open resource

这是因为PIL无法定位到字体文件的位置，可以根据操作系统提供绝对路径，比如：
'/Library/Fonts/Arial.ttf'
</code></pre>

## 常用模块
<pre><code class="language-python line-numbers">## 常用模块
pip install ipython requests beautifulsoup4 pymysql mysqlclient tqdm pexpect virtualenv virtualenvwrapper django shadowsocks

## virtualenvwrapper配置：
--- ~/.bash_profile ---
if [ -f /usr/local/python3/bin/virtualenvwrapper.sh ]; then
    export WORKON_HOME=$HOME/.venv
    export VIRTUALENVWRAPPER_PYTHON=/usr/bin/py3
    . /usr/local/python3/bin/virtualenvwrapper.sh
fi

## tqdm 进度条模块
#!/bin/env python3
# -*- coding: utf-8 -*-

import time
from tqdm import tqdm

for i in tqdm(range(200)):
    time.sleep(0.1)
</code></pre>

## sqlite3模块

[python3启用sqlite3](https://www.zfl9.com/dev-tools.html#python3-sqlite3)
