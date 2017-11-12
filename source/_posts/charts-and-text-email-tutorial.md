---
title: 生成Charts图片，并发送Charts图片邮件
date: 2017-10-16 22:33:37
tags: Python
---

CTEmail(Charts and Text Eamil)是一个发送带有图片的邮件的小工具，这个图片是邮件内容中显示，不是添加在附件中，这个脚本实现的比较简单粗暴，不管长相丑陋，只要能解决实际问题就行。

## 为什么有CTEmail?

* 没有一个不懒的程序员，做啥都想着写个脚本跑一下，跑个脚本抢月饼，跑个脚本...能用脚本的干嘛不用脚本。每天的数据报表需要一个邮件脚本发送。
* 对接了各大厂商，每天每周每月都会往来邮件。报表用图表的形式更简单直观的反馈数据，为什么我们不在邮件中使用图表。
* 各大厂商的邮件中图表都是小姐姐手动制作，手动发出%>_<%，为什么不跑一个脚本。
* 如果解决上面的问题，是不是解放了小手。1) 用数据生成图表，2) 将图表拼接到邮件中发出。

这个小工具的初衷，解决实际问题还是很重要的。

## 哪些人需要这个工具？

* 产品同学，图表是展示数据的最佳实践！
* 运营同学，图表是展示报表的最佳实践！
* 技术同学，为了继续懒下去！
* ...零编程基础的同学都能使用【只要会科学上网就能解决一切】

说这些都是没用，那就跟来做一下吧。
项目托管在github上，地址：[dyike/CTEmail](https://github.com/dyike/CTEmail)。记得来star哟！！！

## 怎么使用？

首先，稍微知道怎么操作Python，像我这种不会写代码都能操作，你一定也可以。其次是到[Plotly](https://plot.ly/python/getting-started/)——这是一个可视化数据的工具有点类似于HighCharts，不过支持多种语言，很强大了。先熟悉一下， 然后注册一个账号，后面会用到。本文着重数据生成图表图片。邮件服务配置查看`README`

### 1st Step：
将项目clone下来，熟悉项目的结构，里面不到两百行代码，简单粗暴。

```bash
git clone git@github.com:dyike/CTEmail.git
```

在`send.py`文件中配置自己的邮箱账号，密码，邮件标题，邮件模板路径和发送到的邮箱。

邮件模板的默认路径是`./content/`，会自动读取该路径下的html文件。

```python
from ctemail import CTEmail
e = CTEmail('Your email acount', 'Your password')
# " ./content/ 邮件文件的路径 "
e.send_email('Test Email Title', './content/', ['i@ityike.com'])
```

默认是配置QQ的STMP发送服务（stmp.qq.com）,端口是25。你也可以配置163.gmail等等，在初始化CTEmail()配置相应的配置即可。


### 2nd Step:

项目中提供了一个默认的模板，你可以根据你的实际需求定制的模板，`content`文件夹下面还有图片资源，我们生成的图表的图片资源也是在该文件夹下面。

需要注意的是，html文件中，将img标签用<EMAIL_IMG>给包了一层，这样只是为了能够方便Python解析，替换。【ps：这里可以了解一下Python发送图片邮件的实现，将图片cid替换进来】[参考廖雪峰老师的教程](https://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386832745198026a685614e7462fb57dbf733cc9f3ad000) 这里你可以不Care这些事。

模板中需要注意的一点：也是非常重要的一点就是：html中多个<EMAIL_IMG>标签需要换行。不换行就无法正确解析。
```
    <a><EMAIL_IMG><img src="image1.png"></EMAIL_IMG></a>
    <a><EMAIL_IMG><img src="image2.png"></EMAIL_IMG></a>
```

### 3th Step:

快要结束了，本文的却重点来了，不要慌，也很简单。就是使用Plotly，关于使用离线（本地）模式还是在线模式，看自己实际需求。我这里说在线的。因为我用的是定时脚本，我只能调用在线的API生成图片保存到本地`content`文件夹下面。

* 安装Plotly
```bash
pip install plotly 
```

* `get_img.py`文件,文件名可以重命名，里面需要的配置你的认证信息credentials，信息在`https://plot.ly/settings/api`中查看。有两种方式：第一种如下
设置username和api_key。

```python
import plotly 
plotly.tools.set_credentials_file(username='DemoAccount', api_key='lr1c37zw81')
```

或者在安装完成后，在`~/.plotly/.credentials`文件中配置你的账号信息。
看到的信息大致如下：修改对应的即可。
```json
{
    "username": "DemoAccount",
    "stream_ids": ["ylosqsyet5", "h2ct8btk1s", "oxz4fm883b"],
    "api_key": "lr1c37zw81"
}
```

* 继续`get_img.py`文件
```python
import plotly.plotly as py
import plotly.graph_objs as go

py.sign_in('Your account', 'API Token') # 注意：这里是plotly网站的用户名和密码

trace = go.Bar(x=[2, 4, 6], y= [10, 12, 15])
data = [trace]
layout = go.Layout(title='A Simple Plot', width=800, height=640)
fig = go.Figure(data=data, layout=layout)
# 保存图片文件的路径
py.image.save_as(fig, filename='./content/image1.png')

# 拼接模板文件
template = '<!DOCTYPE html><html><head><meta charset="UTF-8"></head><body>' + "\n" + '<a><EMAIL_IMG><img src="image1.png"></EMAIL_IMG></a>' + "\n" + '<a><EMAIL_IMG><img src="image2.png"></EMAIL_IMG></a>' + "\n" + '</body></html>'

print template
```

注意上面拼接模板文件内容的时候使用了换行符`"\n"`,为什么这样使用，一简单粗暴，二为了引起重视【这里有坑】。

* 执行上面的脚本文件`python get_img.py > ./content/index.html` 这样就可以将模板文件写入到`content`目录下的`index.html`

* 执行`python send.py`邮件就可以发送邮件，将上面的几个命令写入到shell脚本中,更新方便快捷。

* 其他图表的生成也可以参考官方文档的介绍。

### 4th Enjoy it!!!

放在最后的不是不重要，解决实际问题才是更重要，欢迎来[CTEmail](https://github.com/dyike/CTEmail)Star！！！
