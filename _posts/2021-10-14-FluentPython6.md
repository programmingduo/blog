---
layout: post
title: "流畅的python——使用一等函数实现设计模式"
description: ""
categories: [python]
tags: []
redirect_from:
  - /2021/10/14/
---

* Karmdown table of content
{:toc .toc}

工程项目写多了之后慢慢感受到设计模式的重要性了。而所用的语言决定了哪些设计模式可用。

# 策略模式与命令模式

这本书里面仅仅描述了策略模式和命令模式，感觉在实际应用场景中可以比较轻松实现。不想赘述。简单写一下命令模式的样例吧。

~~~python
class MacroCommand:

	def __init__(self, commands):
		self.commands = list(commands)

	def __call__(self):
		for command in self.commands:
			command()
~~~

# 工厂模式

此外，在做深度学习试验过程中，本人利用工厂模式，可以根据配置文件返回需要的模型。直接放代码

~~~python
# 三层全连接
class FeatureModel2(nn.Module):
    def __init__(self, paras):
    	pass

    def forward(self, x):
    	pass

class BertEncoder(nn.Module):
    def __init__(self, paras):
    	pass

    def forward(self, x):
    	pass


class ModelFactory(object):
    """
    简单工厂，根据参数返回初始化之后的model
    """

    @staticmethod
    def get_model(paras):
        line = paras['name'] + "(paras)"
        return eval(line)

# 使用方法
model = ModelFactory.get_model(paras=paras['model'])
~~~~