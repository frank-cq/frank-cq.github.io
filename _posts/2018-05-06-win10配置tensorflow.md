---
layout: post
title: Win10 配置 tensorflow
categories: []
tags: [Code]
comments: true
---


进入2018年了，谷歌的 tensorflow 如火如荼，更新一把

>系统：Win10 64-bit
>显卡：GTX 960m
>Python：3.6

##安装 Python 环境
用的 Anaconda，版本 4.5.2，python 3.6。

##安装 tensorflow 
根据 [官网说明](https://www.tensorflow.org/install/install_windows?hl=zh-cn) 安装 tensorflow，可选仅支持 cpu 模式的，这里选的是支持 gpu 的。
```
pip install --ignore-installed --upgrade tensorflow-gpu
```

##安装 cuda
下载 [cuda-9.0](https://developer.nvidia.com/cuda-90-download-archive?target_os=Windows&target_arch=x86_64&target_version=10&target_type=exelocal)，本体加两个补丁，下到本地后直接执行就行。

##安装cuDNN
到这里需要注册个会员，下载 [cuDNN](https://developer.nvidia.com/rdp/cudnn-download)，选择 win10 下适配 cuda-9.0的版本。将解压后 cuda 文件夹里的内容扔到安装的 CUDA 文件夹下。

##跑个例子
```py
import tensorflow as tf
a = tf.constant([1.0,2.0],name="a")
b = tf.constant([2.0,1.0],name="b")
result = a+b
sess = tf.Session()
sess.run(result)
```

逐行执行，到 `sess = tf.Session()`这里时，会得到 gpu 信息，tensorflow 只支持 [Nvidia 计算能力](https://developer.nvidia.com/cuda-gpus#collapse4) 在3以上的显卡，泰坦土豪请忽略。
```
2018-05-05 19:00:36.167065: I T:\src\github\tensorflow\tensorflow\core\platform\cpu_feature_guard.cc:140] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2
2018-05-05 19:00:36.899185: I T:\src\github\tensorflow\tensorflow\core\common_runtime\gpu\gpu_device.cc:1356] Found device 0 with properties:
name: GeForce GTX 960M major: 5 minor: 0 memoryClockRate(GHz): 1.176
pciBusID: 0000:01:00.0
totalMemory: 2.00GiB freeMemory: 1.65GiB
2018-05-05 19:00:36.910843: I T:\src\github\tensorflow\tensorflow\core\common_runtime\gpu\gpu_device.cc:1435] Adding visible gpu devices: 0
2018-05-05 19:03:09.977647: I T:\src\github\tensorflow\tensorflow\core\common_runtime\gpu\gpu_device.cc:923] Device interconnect StreamExecutor with strength 1 edge matrix:
2018-05-05 19:03:09.987064: I T:\src\github\tensorflow\tensorflow\core\common_runtime\gpu\gpu_device.cc:929]      0
2018-05-05 19:03:09.990290: I T:\src\github\tensorflow\tensorflow\core\common_runtime\gpu\gpu_device.cc:942] 0:   N
2018-05-05 19:03:10.018493: I T:\src\github\tensorflow\tensorflow\core\common_runtime\gpu\gpu_device.cc:1053] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 1417 MB memory) -> physical GPU (device: 0, name: GeForce GTX 960M, pci bus id: 0000:01:00.0, compute capability: 5.0)
```

##安装过程的问题
####P1

```
You are using pip version 9.x.x, however version 10.x.x is available. You should consider upgrading
```
 解决：上 python pip 官网，下10版本的 tar.gz 文件，解压后执行
 
```
 python <解压路径下的setup.py> install
```

####P2
如果装了 tensorflow 不支持的 cuda 版本，会遇到类似下面的问题
```
ImportError: Could not find 'cudart64_90.dll'. TensorFlow requires that this DLL be installed in a directory that is named in your %PATH% environment variable. 
```

解决：查看 cuda 安装路径下有 cudart64_91.dll，因为当前 tensorflow 版本不支持 cuda-9.1，所以装回 cuda-9.0 就OK了。