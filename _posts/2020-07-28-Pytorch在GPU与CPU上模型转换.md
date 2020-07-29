---
layout: post
title: 'Pytorch在GPU与CPU上模型转换'
date: 2020-07-28
author: hughie
cover: '/assets/img/pytorch.png'
tags: AI Pytorch
---

# 前言

pytorch在不同设备上训练的模型会有所差异，在科研过程中经常有在不同设备上运行的情况，本文是对这些情况的记录备忘。

  

各种情况如下：

#### 1、直接在GPU上加载：

**（多卡->多卡   单卡->单卡）**

不论是多卡到多卡还是单卡到单卡没得说，正常加载即可。

```python
pretrain = torch.load(pretrain_path)
model.load_state_dict(pretrain)
```
**（多卡->单卡 ）**

多卡训练的模型加载到单卡上，可以利用单卡的DataParallel模式先加载好model，再将模型参数加载到model中，此时虽然只有一张卡，但属于多卡模式。

> 此时model还是多卡模式，与单卡模式还是有所区别的

```python
 pretrain = torch.load(pretrain_path)
 model = nn.DataParallel(model)
 model.load_state_dict(pretrain)
```

**（单卡->多卡）**

单卡模型参数可以先加载到model中，再使用DataParallel多卡模式。

```python
 pretrain = torch.load(pretrain_path)
 model.load_state_dict(pretrain)
 model = nn.DataParallel(model)
```



#### 2、将GPU模型加载在CPU上：

**(单卡->CPU)**

单卡GPU模型加载到CPU上有两种方式，第一种是强制所有GPU张量在CPU中

```python
pretrain = torch.load(pretrain_path, map_location=lambda storage, loc: storage)
```

```python
pretrain = torch.load(pretrain_path, map_location='cpu')
```

**（多卡->CPU）**

多卡模式的模型在用上述方法加载会出现这个错误：`KeyError: 'unexpected key "module.conv1.weight" in state_dict'`

这是因为多卡模式`nn.DataParallel`，该模式将模型存储在模型中的`module`中，而单卡的是没有这个key的，所以要用增加下面代码新建一个没有module的字典。

```python
from collections import OrderedDict
new_state_dict = OrderedDict()
for k, v in pretrain.items():
	name = k[7:]# remove `module.`
	new_state_dict[name] = v
model.load_state_dict(new_state_dict)
```



**其他解决方法**

在保存的时候直接model.module.state_dict()保存，就可以在任意设备加载，不考虑module的问题

  

#### 3、CPU加载到GPU

device_id是GPU的设备号

```python
pretrain = torch.load(pretrain_path, map_location=lambda storage, loc: storage.cuda(device_id))
```



# 最后

要特别注意tensor与model要在同一个设备上才能在一起使用，使用并行模式DataParallel会将model与tensor拆分，如果中途自定义了tensor数据，要加载好像会出问题，我目前还未解决，只能避免遇到这种情况:cry::pensive:，有人看到这篇文章知道的话请指导我一下哦。



### 声明

本文仅个人学习总结，如有错误，请指正。