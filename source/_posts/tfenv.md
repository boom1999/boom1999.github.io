# 1.前言：先装python还是先装anaconda #
# 2. Anaconda版本选择与下载 #
# 3. Anaconda安装 #
# 4. 更换镜像源 #
# 5. Pycharm使用 #
# 6. Pycharm虚拟环境介绍 #
# 7. CUDA与cuDNN #
# 8. TensorFlow #


----------


##版本##
### python 3.7.0 ###
### conda 4.5.11 ###
### CUDA 11.1 ###
### Cudatoolkit 11.1.0 ###
### cuDNN 11.1 v8.0.5 ###
### tensorflow_gpu-2.4.1 ###
### Pycharm Professional 2020.3.3 ###


----------


> 写在最前面
>下面列举了一些常见问题，请务必注意

 - 国外源速度慢导致连接断开（请参考下文更换pip源和conda源）
 - 版本对应问题（请勿安装过高版本的Anaconda，会导致python版本过高，Tensorflow不支持，至本文发布为止最高支持到python3.8，Tensorflow_CPU的安装较为简单，故演示了GPU-2.4.1）

笔者在开始前已经有了python3.6.8，其实可以直接跳过

> [Python Download][1]
鉴于目前网上教程采用的版本比较混乱，故此次保姆级教程采用了较新的版本并通过测试
> 保姆级教程：尽可能详细的列出具体步骤，但省略了一些作者认为比较简单的操作：如何打开命令行、命令行基本操作、如何使用Pycharm、软件的基本安装流程、环境变量的手动配置等。
> 
> **如果仍然无法解决您的问题**
> 建议1：转身立马退坑，太卷了
> 建议2：[点击这里][2]

----------
## 1.前言：先装python还是先装anaconda ##

> **装anaconda，就不需要单独装python了**

> anaconda 是一个python的发行版，包括了python和很多常见的软件库,和一个包管理器conda，集成了很多关于python科学计算的第三方库，主要是安装方便，而python是一个编译器，如果不使用anaconda，那么安装起来会比较痛苦，各个库之间的依赖性就很难连接的很好。


## 2.Anaconda版本选择与下载 ##

###方法一###  
> **官网下载** *不推荐，不会开飞机的话速度慢并且容易断*
> [Anaconda官网][3]
![Anaconda官网][4]

###方法二###  
> **国内镜像源** *可以用清华源、中科大源......*
> [Anaconda清华源][5]
![Anaconda清华源][6]

###版本选择###  
 ![请输入图片描述][7]

## 3.Anaconda安装 ##

> ***比较容易***，next和skip，包括自动添加到环境变量等操作，建议直接装到C盘可以避免一些后续问题，笔者这里选择装到了E://Anaconda3 ***等待安装完毕即可***
> 若没有勾选“Add Anaconda to the system Path environment variable”，需要手动添加环境变量，比较简单，此处勾选了所以忽略

> 验证是否安装完成 conda info

![验证是否安装完成 conda info][8]

> Anaconda Navigator是否可用

![Anaconda Navigator是否可用][9]

至此，Python 3.7.0 + Anaconda3-5.3.1安装完成
-------------------------------------


----------


## 4.更换镜像源 ##

> 可以在Anaconda Navigator中的channel直接更换
> 也可以在Anaconda Prompt中更换镜像源

    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
    conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
    conda config --set show_channel_urls yes

> 查看是否已经更换好通道

    conda config --show channels
![Change_channel][10]


## 5.Pycharm使用 ##

> Jetbrains全家桶,US $649.00/Year,当然建议使用edu邮箱获取，申请教育版邮箱认证即可（可能需要不定时验证信息），建议安装**Toolbox**方便更新以及使用

> [附上**Jetbrains**链接][11]

> Jetbrains的验证邮件容易出现在e邮的**拦截队列**中，收不到confirm email可以在拦截队列中找到

> 如果比较困难，可以尝试装中文插件 File->Settings->Plugins->Chinese (Simplified) Language Pack EAP  **当然不建议这样使用，汉化的翻译可能存在偏差**


## 6. Pycharm虚拟环境介绍 ##
> Pycharm采用虚拟的环境，配置是可以采用为项目新建Python解释器或者使用原有的环境
 1. Inherit global site-packages
    继承本地的库，如果需要重复安装第三方库，可以勾上这个
 2. Make available to all projects
    可以使别的工程项目可以使用此工程的环境
> 区别在于，创建Virtualenv Environment 或者 Conda Environment
> 若要在服务器上远程运行，配置SSH interpreter，并且在Tools->Deploment->Configuration中配置主机即可，后续有机会再详细展开......

## 7. CUDA和cuDNN ##

> 检测目前安装了哪些环境
>命令行 `conda info --envs`

> 检查目前有哪些版本的python可以安装
> 命令行`conda search --full-name python`

> 检查目前有哪些版本的Tensorflow可以安装
> 命令行`conda search --full -name tensorflow`

> 查看依赖关系
> 命令行`conda info tensorflow`
> 
> 
> **在这之前，检查Nvidia驱动支持的CUDA版本，此处安装的是Tensorflow2.4.1，需要CUDA11.0或及以上版本**
> [查看Tensorflow和CUDA对应版本][12]（版本一定要对应）
> `nvidia-smin`命令行查看CUDA Version （当前显卡驱动最高支持的CUDA版本）

> [Nvidia官网下载Cudatoolkit][13]（下载速度慢可以复制链接到迅雷下载）
> 一路安装下来即可，安装过程中已经自动添加了环境变量
> 命令行`nvcc -V`返回CUDA版本信息，如果**nvcc命令无法识别则表示安装失败**
![CUDA版本信息][14]

> [Nvidia官网下载cuDNN][15]（下载速度慢可以复制链接到迅雷下载）
>*只要把cuDNN文件复制到CUDA的对应文件夹里就可以，即所谓的插入式设计，cuDNN是CUDA的扩展计算库，不会对CUDA造成其他影响。*
> 复制后，手动将CUDA相对路径下的/bin和/lib/x64添加到环境变量CUDA_PATH
> 
> **测试CUDA**
> 命令行`cd /d C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.1\extras\demo_suite`
> 测试`deviceQuery.exe`
![CUDA_deviceQuery][16]
> 测试`bandwidthTest.exe`
![CUDA_bandwidthTest][17]
> 都返回Result = PASS   
> That's OK

## 8. TensorFlow ##
###8.1 创建环境###

    conda create -name tfenv python=3.7.0

> 由于之前已经更换了清华源，由于网络问题造成的错误率大大降低，但仍然可能被中断
> 用`conda info --envs`检查一下是否装好了tfenv环境

###8.2 激活环境###

    conda activate tfenv

> 退出环境`conda activate`

###8.3 激活后安装Tensorflow###

> 更换tfenv环境的pip源
[更换pip清华源参考方法][18]
    pip install tensorflow-gpu
> 虽然已经换源，但还是可能出现`These Packages Do Not Match The Hashes From The Requirements File.`
> 解决方案：在pip时，添加"–upgrade"参数（**实测有效**）。
> 几分钟后-->**安装成功**

###8.4 激活后测试Tensorflow###

> 输出包含GPU信息的日志

    import tensorflow as tf
    tf.test.is_gpu_available()
    tf.test.gpu_device_name()
![Prompt_test][19]

###8.5 Pycharm测试Tensorflow###

> 添加安装有Tensorflow的环境作为工程的python解释器
![Anaconda官网.png][20]

    import tensorflow as tf
    tf.compat.v1.disable_eager_execution()
    greeting = tf.constant('Hello Tensorflow!')
    sess = tf.compat.v1.Session()
    result = sess.run(greeting)
    print(result)
    sess.close()
![Pycharm中测试][21]


----------

> 至此所有安装和配置完成，**版本对应**问题非常重要，如果出现装错了版本请尝试**彻底卸载**后重装
> 为了更有针对性，本文采用了较新的版本，请务必在安装前选择**合适的版本**
> 如果提示确实dll动态链接库，请尝试手动下载，貌似采用CUDA11.1可能会遇到这种情况，并且有部分人认为11.1中有未知错误并且不推荐使用，但在安装过程中并未发现，或许之后遇到了会继续补充


  [1]: https://www.python.org/downloads/
  [2]: https://www.google.com.hk/
  [3]: https://www.anaconda.com/products/individual
  [4]: https://www.lingzhicheng.cn/usr/file/picture/Python/Anaconda.png
  [5]: https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive
  [6]: https://www.lingzhicheng.cn/usr/file/picture/Python/Anaconda_tsinghua.png
  [7]: https://www.lingzhicheng.cn/usr/file/picture/Python/Anaconda_python.png
  [8]: https://www.lingzhicheng.cn/usr/file/picture/Python/Anaconda_complete.png
  [9]: https://www.lingzhicheng.cn/usr/file/picture/Python/Anaconda_navigator.png
  [10]: https://www.lingzhicheng.cn/usr/file/picture/Python/Change_channel.png
  [11]: https://www.jetbrains.com
  [12]: https://tensorflow.google.cn/install/source_windows?hl=cn#gpu
  [13]: https://developer.nvidia.com/cuda-toolkit-archive
  [14]: https://www.lingzhicheng.cn/usr/file/picture/Python/nvcc%20-V.png
  [15]: https://developer.nvidia.com/rdp/cudnn-archive
  [16]: https://www.lingzhicheng.cn/usr/file/picture/Python/CUDA_test0.png
  [17]: https://www.lingzhicheng.cn/usr/file/picture/Python/CUDA_test1.png
  [18]: https://mirrors.tuna.tsinghua.edu.cn/help/pypi/
  [19]: https://www.lingzhicheng.cn/usr/file/picture/Python/Prompt_test.png
  [20]: https://www.lingzhicheng.cn/usr/file/picture/Python/Pycharm_test0.png
  [21]: https://www.lingzhicheng.cn/usr/file/picture/Python/Pycharm_test1.png


