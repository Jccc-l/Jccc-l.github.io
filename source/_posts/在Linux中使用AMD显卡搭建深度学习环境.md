---
title: 在Linux中搭建AMD的深度学习环境
date: 2024-05-06 16:00:21
categories:
- 深度学习
tags:
- Linux
- Deep Learning
- Pytorch
- AMD
---

# AMD GPU运行Pytorch

<!-- more-->

以下命令**配置环境变量**

```zsh $HOME/.zshrc
export HSA_OVERRIDE_GFX_VERSION=10.3.0
export LD_LIBRARY_PATH=/opt/rocm/lib
```

> 1. HIP_VISIBLE_DEVICES=0 此处非必要不要改动，多显卡用户(例如核显+独显)出现RuntimeError: Torch is not able to use GPU问题的时候将0改为1；
> 2. HSA_OVERRIDE_GFX_VERSION= 这里要根据自己显卡型号修改：
>       - HSA_OVERRIDE_GFX_VERSION=9.0.6对应显卡型号： Radeon VII
>       - HSA_OVERRIDE_GFX_VERSION=10.3.0对应显卡型号：RX5000 / 6000系列
>       - HSA_OVERRIDE_GFX_VERSION=11.0.0对应显卡型号：RX7000系列

创建并激活虚拟环境

```sh
$ python -m venv venv
$ source venv/bin/activate
```

通过[Pytorch主页](pytorch.org)的方法安装好ROCm版本的PyTorch

```sh
$ pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/rocm6.0
```

```sh
$ python
Python 3.10.14 (main, Apr 28 2024, 15:11:45) [GCC 13.2.1 20240417] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.cuda.is_available()
True
```

<img src="torch_available.png" style="max-width:100%">

可以看到ROCm可用
