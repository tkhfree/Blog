---
title: tensorflow一些函数
tags:
  - Tensorflow
comments: true
toc: true
mathjax: true
abbrlink: 88bb568c
date: 2019-09-18 00:21:52
urlname:
categories:
thumbnail:
---
# 相关函数理解
## tf.constant
生成常量张量

    constant(
        value,
        dtype=None,
        shape=None,
        name='Const',
        verify_shape=False
    )

**value 必选 常量数值或者list** 输出张量的值
**dtype 可选 dtype** 输出张量元素类型
**shape 可选 1维整形张量或者array** 输出张量的维度
**name 可选 string** 张量名称
**verify_shape 可选 boolean**
检测shape是否和value一致，若为false，不一致时，会用最后一个元素补全shape



----------
## tf.placeholder
占位符

    placeholder(
        dtype,
        shape=None,
        name=None
    )
dtype 占位符类型
shape 占位符维度

----------

## tf.nn.bias_add
将偏差项 bias 加到 value 上面，可以看做是 tf.add 的一个特例，其中 bias 必须是一维的，并且维度和 value 的最后一维相同，数据类型必须和 value 相同。

    bias_add(
        value,
        bias,
        data_format=None,
        name=None
    )
**value 必选 张量** 	数据类型为 float, double, int64, int32, uint8, int16, int8, complex64, or complex128
**bias 必选 1维张量**	维度必须和value最后一维维度相等
**data_format 可选 string** 	数据格式，支持 ' NHWC ' 和 ' NCHW '
**name 可选 string** 运算名称

    import tensorflow as tf
    import numpy as np
    
    a = tf.constant([[1.0, 2.0],[1.0, 2.0],[1.0, 2.0]])
    b = tf.constant([2.0,1.0])
    c = tf.constant([1.0])
    sess = tf.Session()
    print (sess.run(tf.nn.bias_add(a, b)))
    #print (sess.run(tf.nn.bias_add(a,c))) error
    print ("##################################")
    print (sess.run(tf.add(a, b)))
    print ("##################################")
    print (sess.run(tf.add(a, c)))
    
    运算结果：
    [[3. 3.]
     [3. 3.]
     [3. 3.]]
    ##################################
    [[3. 3.]
     [3. 3.]
     [3. 3.]]
    ##################################
    [[2. 3.]
     [2. 3.]
     [2. 3.]]

----------
##tf.reduce_mean
计算张量 input_tensor 平均值

    reduce_mean(
        input_tensor,
        axis=None,
        keep_dims=False,
        name=None,
        reduction_indices=None
    )
**input_tensor 必选 张量** 输入带求平均值的张量
**axis 可选 	None、0、1** None：全局求平均值；0：求每一列平均值；1：求每一行平均值
**keep_dims 可选 boolean** 保留原来的维度
**name 可选 string** 运算名字
**reduction_indices 可选 none** 和axis等价，弃用、

----------

## tf.squared_difference
计算张量 x、y 对应元素差平方

    squared_difference(
        x,
        y,
        name=None
    )

----------

## tf.square
计算张量对应元素平方

    square(
        x,
        name=None
    )

----------

## tf.nn.conv2d 卷积核

    conv2d(
        input,
        filter,
        strides,
        padding,
        use_cudnn_on_gpu=True,
        data_format='NHWC',
        name=None
    )

**input 必选 tensor** 是一个四维的tensor，即[batch, in_height, in_width, in_channels]（若input是图像，[训练时一个batch的图片数量，图片高度，图片宽度，图像通道数]）


**filter 必选 tensor** 是一个四维的tensor，即[filter_height, filter_width, in_channels, out_channels]（若input是图像，[卷积核的高度，宽度，图像通道数，卷积核的个数]）


**strides 必选 列表** 长度为4的list，卷积时候在input上每一维的步长，一般strides[0]= striders[3]=1(strides[0]指向输入batch数量，strides[3]映射通道数，都为1就是指一幅一幅和一个通道一个通道卷积)

**padding 必选 string**
只能为“VALID”,"SAME"之一，决定了卷积方式，"VALID"：丢弃/"SAME"：补全。

use_cudnn_on_gpu=True,
data_format='NHWC',
name=None
都是可选的，分别是使用GPU加速，数据格式，运算名称


    import tensorflow as tf
    a = tf.constant([1,1,1,0,0,0,1,1,1,0,0,0,1,1,1,0,0,1,1,0,0,1,1,0,0],dtype = tf.float32, shape = [1, 5, 5, 1])
    b = tf.constant([1,0,1,0,1,0,1,0,1],dtype = tf.float32, shape = [1, 3, 3, 1])
    c = tf.nn.conv2d(a,b,strides=[1, 2, 2, 1],padding='VALID')
    d = tf.nn.conv2d(a,b,strides=[1, 2, 2, 1],padding='SAME')
    with tf.Session() as sess:
        print ("c shape:")
        print (c.shape)
        print ("c value:")
        print (sess.run(c))
        print ("d shape:")
        print (d.shape)
        print ("d value:")
        print (sess.run(d))
    
    cd /home/ubuntu;
    python conv2d.py
    
    c shape:
    (1, 2, 2, 1)
    c value:
    [[[[4.]
       [4.]]
    
      [[2.]
       [4.]]]]
    d shape:
    (1, 3, 3, 1)
    d value:
    [[[[2.]
       [3.]
       [1.]]
    
      [[1.]
       [4.]
       [3.]]
    
      [[0.]
       [2.]
       [1.]]]]

----------
## tf.nn.relu 激活函数

    relu(
        features,
        name=None
    )
**feature 必选 tensor** 以下类型float32，float64，int32，int64，unit8，int16，int8，uint16，half
**name 可选 string** 运算名称

    import tensorflow as tf
    
    a = tf.constant([1,-3,0,6,-2,0,3])
    b = tf.nn.relu(a)
    
    with Session() as sess:
        print(sess.run(b))
        
    执行结果：
    [1,0,0,6,0,0,3]

----------

## tf.nn.max_pool 池化层

    max_pool(
        value,
        ksize,
        strides,
        padding,
        data_format='NHWC',
        name=None
    )
**value 必选 tensor** 4维的张量，即[batch，height，width，channels]，数据类型为tf.float32
**ksize 必选 列表** 池化窗口的大小，长度为4的list，一般是[1,height,width,1](类似于conv2d函数的strides参数)，第一个维度和最后一个维度是batch和channels的池化窗口大小
**strides 必选 列表** 池化窗口在每一维的步长，一般strides[0]=strides[3]=1
**padding 必选 string**

只能为"VALID","SAME"中的一个，这个值决定了不同的池化方式。VALID是丢弃，SAME是补全。

**data_format 可选 string** 默认"NHWC"

**name 可选 string** 运算名称


    import tensorflow as tf
    
    a = tf.constant([[1,2,3,4,5],[6,7,8,9,10],[11,12,13,14,15],[16,17,18,19,20],[21,22,23,24,25]],dtype=tf.float32,shape=[1,5,5,1])
    b = tf.nn.max_pool(a,ksize=[1, 2, 2, 1],strides=[1, 2, 2, 1],padding='VALID')
    c = tf.nn.max_pool(a,ksize=[1, 2, 2, 1],strides=[1, 2, 2, 1],padding='SAME')
    with tf.Session() as sess:
        print ("b shape:")
        print (b.shape)
        print ("b value:")
        print (sess.run(b))
        print ("c shape:")
        print (c.shape)
        print ("c value:")
        print (sess.run(c))
        
        运算结果：
    b shape:
    (1, 2, 2, 1)
    b value:
    [[[[ 7.]
       [ 9.]]
    
      [[17.]
       [19.]]]]
    c shape:
    (1, 3, 3, 1)
    c value:
    [[[[ 7.]
       [ 9.]
       [10.]]
    
      [[17.]
       [19.]
       [20.]]
    
      [[22.]
       [24.]
       [25.]]]]
       池化窗口偶数，value奇数尺寸，valid直接舍弃最后一列，same最后补0.
       ksize与最后计算的尺寸没有关系，只是池化的范围

----------
## tf.nn.dropout
防止过拟合，对于神经网络单元，按照一定的概率将其暂时随机的从网络中丢弃。对于随机梯度下降来说，由于是随机丢弃，所以每一个mini-batch都在训练不同的网络。

    dropout(
        x,
        keep_prob,
        noise_shape=None,
        seed=None,
        name=None
    )
**x 必选 tensor** 输出元素是x中的元素，数值变为1/keep_prob，否则为0
**keep_prob 必选 scalar tensor** dropout的概率，一般是占位符。设置神经元被选中的概率,在初始化时keep_prob是一个占位符,  keep_prob = tf.placeholder(tf.float32) 。tensorflow在run时设置keep_prob具体的值，例如keep_prob: 0.5
**nosie_shape 可选 tensor** 默认情况下，每个元素是否dropout是相互独立。如果制定noise_shape，若noise_shape[i]=shape(x)[i],该维度的元素是否dropout是相互独立。若noise_shape[i] != shape(x)[i] 该维度元素是否 dropout 不相互独立，要么一起 dropout 要么一起保留。看shape和value_shape行列数是否一致，行一致，则行向量要么为0，要么都dropout；列同理。
**seed 可选 数值** 如果指定该值，每次dropout结果相同
**name 可选 string** 运算名称

    import tensorflow as tf
    
    a = tf.constant([1,2,3,4,5,6],shape=[2,3],dtype=tf.float32)
    b = tf.placeholder(tf.float32)
    c = tf.nn.dropout(a,b,[2,1],1)
    with tf.Session() as sess:
        sess.run(tf.global_variables_initializer())
        print (sess.run(c,feed_dict={b:0.75}))
        
    运算结果：
    [[ 0.          0.          0.        ]
     [ 5.33333349  6.66666651  8.        ]]

----------

## tf.nn.sigmoid_cross_entropy_with_logits 
这个函数的作用是计算经sigmoid 函数激活之后的交叉熵 /labels和logits之间的交叉熵（cross entropy）
/先对 logits 通过 sigmoid 计算，再计算交叉熵

    sigmoid_cross_entropy_with_logits(
        _sentinel=None,
        labels=None,
        logits=None,
        name=None
    )
为了描述简洁，我们规定 x = logits，z = targets，那么 Logistic 损失值为：

    max(x,0)−x∗z+log(1+exp(−abs(x)))
log以e为底的对数，exp是以e为底的指数函数
logits 和 targets 必须有相同的数据类型和数据维度。
它适用于每个类别相互独立但互不排斥的情况,在一张图片中，同时包含多个分类目标（大象和狗），那么就可以使用这个函数。
 **_sentinel 可选 none** 一般情况下没有使用的参数
**labels 可选 tensor** type，shape与logits相同。分类标签，所不同的是，这个label是分类的概率，比如说[0.2,0.3,0.5]，labels的每一行必须是一个概率分布。
**logits 可选 tensor** type是float32或者float64
**name 可选 string** 运算名称

    import tensorflow as tf
    x = tf.constant([1,2,3,4,5,6,7],dtype=tf.float64)
    y = tf.constant([1,1,1,0,0,1,0],dtype=tf.float64)
    loss = tf.nn.sigmoid_cross_entropy_with_logits(labels = y,logits = x)
    with tf.Session() as sess:
        print (sess.run(loss))
        
    运算结果：
     [3.13261688e-01 1.26928011e-01 4.85873516e-02 4.01814993e+00
     5.00671535e+00 2.47568514e-03 7.00091147e+00]

关于交叉熵的理解，参考[交叉熵为何能做损失函数](https://blog.csdn.net/wenzishou/article/details/77618992)

----------
## tf.truncated_normal
产生截断正态分布随机数，取值范围为 [ mean - 2 * stddev, mean + 2 * stddev ]

    truncated_normal(
        shape,
        mean=0.0,
        stddev=1.0,
        dtype=tf.float32,
        seed=None,
        name=None
    )
**shape 必选 1维整形张量或array** 输出张量的维度
**mean 可选 0维张量或数值** 均值
**stddev 可选 0维张量或数值** 标准差
**dtype 可选 dtype** 输出类型
**seed 可选 数值** 随机种子，若seed赋值，每次产生相同随机数
**name 可选 string** 运算名称

    import tensorflow as tf
    initial = tf.truncated_normal(shape=[3,3], mean=0, stddev=1)
    print(tf.Session().run(initial))
    
    输出结果：
    产生一个取值范围 [ -2, 2 ] 的 3 * 3 矩阵

----------
# 相关类理解
## tf.Variable
维护图在执行过程中的状态信息，例如神经网络权重值的变化。

    __init__(
        initial_value=None,
        trainable=True,
        collections=None,
        validate_shape=True,
        caching_device=None,
        name=None,
        variable_def=None,
        dtype=None,
        expected_shape=None,
        import_scope=None
    )
**initial_value 张量** Variable 类的初始值，这个变量必须指定 shape 信息，否则后面 validate_shape 需设为 False
**trainable boolean** 是否把变量添加到 collection GraphKeys.TRAINABLE_VARIABLES 中（collection 是一种全局存储，不受变量名生存空间影响，一处保存，到处可取）
**collections Graph collections** 全局存储，默认是 GraphKeys.GLOBAL_VARIABLES
**validate_shape	Boolean**	是否允许被未知维度的 initial_value 初始化
**caching_device	string**	指明哪个 device 用来缓存变量
**name	string**	变量名
**dtype	dtype**	如果被设置，初始化的值就会按照这个类型初始化
**expected_shape	TensorShape**	要是设置了，那么初始的值会是这种维度

    import tensorflow as tf
    initial = tf.truncated_normal(shape=[3,3],mean=0,stddev=1)
    W=tf.Variable(initial)
    list = [[1.,1.],[2.,2.]]
    X = tf.Variable(list,dtype=tf.float32)
    init_op = tf.global_variables_initializer()
    with tf.Session() as sess:
        sess.run(init_op)
        print ("##################(1)################")
        print (sess.run(W))
        print ("##################(2)################")
        print (sess.run(W[:2,:2]))
        op = W[:2,:2].assign(22.*tf.ones((2,2)))
        print ("###################(3)###############")
        print (sess.run(op))
        print ("###################(4)###############")
        print (W.eval(sess)) #computes and returns the value of this variable类似tf.Session.run(W)
        print ("####################(5)##############")
        print (W.eval())  #Usage with the default session
        print ("#####################(6)#############")
        print (W.dtype)
        print (sess.run(W.initial_value))
        print (sess.run(W.op))
        print (W.shape)
        print ("###################(7)###############")
        print (sess.run(X))

----------
