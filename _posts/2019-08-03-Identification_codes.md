---
layout: post
title: Tensorflow + CNN 验证码识别尝鲜
date: 2019-08-03
tags: Tensorflow   
---
写爬虫难免与验证码打交道，虽然现在有很多打码平台交钱就能稳定使用，但是还是觉得自己动手实现更爽。
尤其是那种很简单的验证码，还使用打码平台（就像是计算1+1还要靠计算器），太不值当了。

几个月前，在网上东翻翻西找找，了解到了验证码识别的启蒙原理：
- 验证码切割为单字符，降噪（减少计算量）
- 找出字符的特征，记录下来
- 重复N次后生成一个模型（使用的`libsvm`）
- 利用这个模型反向操作实现识别


然后自己试了试：

- 部分切好的验证码字符，还没有输入正确的名字

![部分切好的验证码字符，还没有输入正确的名字](/images/posts/Identification_codes/pic_cut.png)

- 训练好的模型长这样

![训练好的模型长这样](/images/posts/Identification_codes/model.png)


虽然代码不咋地，但是成功率还是很高的！

不过这个验证码比较简单，连干扰线都没几根，字符位置也几乎没偏差，仅仅是入门级验证码。

对比人家打码平台可是来啥图识别啥图，哎，差得远。

后来在网上找到了一片文章，使用的是`Tensorflow` + `CNN`实现验证码识别，恩，感觉挺高大上的，毕竟都看不咋懂。
那位老哥测试用的4位纯数字，跑了5个小时后验证码识别成功率达到99%。
但是通常碰到的验证码都是数字加大小写字母，所以就直接Copy了一份代码下来试试水。

**验证码生成模块**：
```python
# -*- coding: utf-8 -*-
"""
@AUTH       Zau
@ADD TIME   2019/07/30
"""
import random
import numpy as np
from PIL import Image
from captcha.image import ImageCaptcha


# np.set_printoptions(threshold=np.inf)
NUMBER = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9']
LOW_CASE = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u',
            'v', 'w', 'x', 'y', 'z']
UP_CASE = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U',
           'V', 'W', 'X', 'Y', 'Z']

CAPTCHA_LIST = NUMBER + LOW_CASE + UP_CASE
CAPTCHA_LEN = 4         # 验证码长度
CAPTCHA_HEIGHT = 60     # 验证码高度
CAPTCHA_WIDTH = 160     # 验证码宽度


def random_captcha_text(captcha_size=CAPTCHA_LEN):
    """
    随机生成定长字符串
    :param captcha_size: 字符串长度
    :return: 字符串
    """
    captcha_text = [random.choice(CAPTCHA_LIST) for _ in range(captcha_size)]
    return ''.join(captcha_text)


def gen_captcha_text_and_image(width=CAPTCHA_WIDTH, height=CAPTCHA_HEIGHT, save=False):
    """
    生成随机验证码
    :param width: 验证码图片宽度
    :param height: 验证码图片高度
    :param save: 是否保存（None）
    :return: 验证码字符串，验证码图像np数组
    """
    image = ImageCaptcha(width=width, height=height)
    # 验证码文本
    captcha_text = random_captcha_text()
    captcha = image.generate(captcha_text)
    # 保存
    if save:
        image.write(captcha_text, './img/' + captcha_text + '.jpg')
    captcha_image = Image.open(captcha)
    # captcha_image.show()
    # 转化为np数组
    captcha_image = np.array(captcha_image)
    return captcha_text, captcha_image


if __name__ == '__main__':
    t, im = gen_captcha_text_and_image(save=False)
    print(t, im.shape)      # (60, 160, 3)
    print(im)
```

**工具模块**：
```python
# -*- coding: utf-8 -*-
"""
@AUTH       Zau
@ADD TIME   2019/07/30
"""
import numpy as np
from captcha_gen import CAPTCHA_LIST, CAPTCHA_LEN, CAPTCHA_HEIGHT, CAPTCHA_WIDTH
from captcha_gen import gen_captcha_text_and_image


np.set_printoptions(threshold=np.inf)


def convert2gray(img):
    """
    图片转为黑白，3维转1维
    :param img: np
    :return:  灰度图的np
    """
    if len(img.shape) > 2:
        img = np.mean(img, -1)
    return img


def text2vec(text, captcha_len=CAPTCHA_LEN, captcha_list=CAPTCHA_LIST):
    """
    验证码文本转为向量
    :param text:
    :param captcha_len:
    :param captcha_list:
    :return: vector 文本对应的向量形式
    """
    text_len = len(text)    # 欲生成验证码的字符长度
    if text_len > captcha_len:
        raise ValueError('验证码最长4个字符')
    vector = np.zeros(captcha_len * len(captcha_list))      # 生成一个一维向量 验证码长度*字符列表长度
    for i in range(text_len):
        vector[captcha_list.index(text[i])+i*len(captcha_list)] = 1     # 找到字符对应在字符列表中的下标值+字符列表长度*i 的 一维向量 赋值为 1
    return vector


def vec2text(vec, captcha_list=CAPTCHA_LIST, captcha_len=CAPTCHA_LEN):
    """
    验证码向量转为文本
    :param vec:
    :param captcha_list:
    :param captcha_len:
    :return: 向量的字符串形式
    """
    vec_idx = vec
    text_list = [captcha_list[int(v)] for v in vec_idx]
    return ''.join(text_list)


def wrap_gen_captcha_text_and_image(shape=(60, 160, 3)):
    """
    返回特定shape图片
    :param shape:
    :return:
    """
    while True:
        t, im = gen_captcha_text_and_image()
        if im.shape == shape:
            return t, im


def get_next_batch(batch_count=60, width=CAPTCHA_WIDTH, height=CAPTCHA_HEIGHT):
    """
    获取训练图片组
    :param batch_count: default 60
    :param width: 验证码宽度
    :param height: 验证码高度
    :return: batch_x, batch_yc
    """
    batch_x = np.zeros([batch_count, width * height])
    batch_y = np.zeros([batch_count, CAPTCHA_LEN * len(CAPTCHA_LIST)])
    for i in range(batch_count):    # 生成对应的训练集
        text, image = wrap_gen_captcha_text_and_image()     # 字符 图片
        image = convert2gray(image)     # 转灰度numpy
        # 将图片数组一维化 同时将文本也对应在两个二维组的同一行
        batch_x[i, :] = image.flatten() / 255
        batch_y[i, :] = text2vec(text)  # 验证码文本的向量形式
    # 返回该训练批次
    return batch_x, batch_y


if __name__ == '__main__':
    x, y = get_next_batch(batch_count=1)    # 默认为1用于测试集
    print(x)
    print()
    print(y)

```

**训练模块**：
```python
# -*- coding: utf-8 -*-
"""
@AUTH       Zau
@ADD TIME   2019/07/30
"""
# name: model_train.py

import tensorflow as tf
from datetime import datetime
from util import get_next_batch
from captcha_gen import CAPTCHA_HEIGHT, CAPTCHA_WIDTH, CAPTCHA_LEN, CAPTCHA_LIST


def weight_variable(shape, w_alpha=0.01):
    """
    初始化权值
    :param shape:
    :param w_alpha:
    :return:
   """
    initial = w_alpha * tf.random_normal(shape)
    return tf.Variable(initial)


def bias_variable(shape, b_alpha=0.1):
    """
    初始化偏置项
    :param shape:
    :param b_alpha:
    :return:
    """
    initial = b_alpha * tf.random_normal(shape)
    return tf.Variable(initial)


def conv2d(x, w):
    """
    卷基层 ：局部变量线性组合，步长为1，模式‘SAME’代表卷积后图片尺寸不变，即零边距
    :param x:
    :param w:
    :return:
    """
    return tf.nn.conv2d(x, w, strides=[1, 1, 1, 1], padding='SAME')


def max_pool_2x2(x):
    """
    池化层：max pooling,取出区域内最大值为代表特征， 2x2 的pool，图片尺寸变为1/2
    :param x:
    :return:
    """
    return tf.nn.max_pool(x, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding='SAME')


def cnn_graph(x, keep_prob, size, captcha_list=CAPTCHA_LIST, captcha_len=CAPTCHA_LEN):
    """
    三层卷积神经网络
    :param x:   训练集 image x
    :param keep_prob:   神经元利用率
    :param size:        大小 (高,宽)
    :param captcha_list:
    :param captcha_len:
    :return: y_conv
    """
    # 需要将图片reshape为4维向量
    image_height, image_width = size
    x_image = tf.reshape(x, shape=[-1, image_height, image_width, 1])

    # 第一层
    # filter定义为3x3x1， 输出32个特征, 即32个filter
    w_conv1 = weight_variable([3, 3, 1, 32])    # 3*3的采样窗口，32个（通道）卷积核从1个平面抽取特征得到32个特征平面
    b_conv1 = bias_variable([32])
    h_conv1 = tf.nn.relu(conv2d(x_image, w_conv1) + b_conv1)    # rulu激活函数
    h_pool1 = max_pool_2x2(h_conv1)     # 池化
    h_drop1 = tf.nn.dropout(h_pool1, keep_prob)      # dropout防止过拟合

    # 第二层
    w_conv2 = weight_variable([3, 3, 32, 64])
    b_conv2 = bias_variable([64])
    h_conv2 = tf.nn.relu(conv2d(h_drop1, w_conv2) + b_conv2)
    h_pool2 = max_pool_2x2(h_conv2)
    h_drop2 = tf.nn.dropout(h_pool2, keep_prob)

    # 第三层
    w_conv3 = weight_variable([3, 3, 64, 64])
    b_conv3 = bias_variable([64])
    h_conv3 = tf.nn.relu(conv2d(h_drop2, w_conv3) + b_conv3)
    h_pool3 = max_pool_2x2(h_conv3)
    h_drop3 = tf.nn.dropout(h_pool3, keep_prob)

    """
    原始：60*160图片 第一次卷积后 60*160 第一池化后 30*80
    第二次卷积后 30*80 ，第二次池化后 15*40
    第三次卷积后 15*40 ，第三次池化后 7.5*20 = > 向下取整 7*20
    经过上面操作后得到7*20的平面
    """

    # 全连接层
    image_height = int(h_drop3.shape[1])
    image_width = int(h_drop3.shape[2])
    w_fc = weight_variable([image_height*image_width*64, 1024])     # 上一层有64个神经元 全连接层有1024个神经元
    b_fc = bias_variable([1024])
    h_drop3_re = tf.reshape(h_drop3, [-1, image_height*image_width*64])
    h_fc = tf.nn.relu(tf.matmul(h_drop3_re, w_fc) + b_fc)
    h_drop_fc = tf.nn.dropout(h_fc, keep_prob)

    # 输出层
    w_out = weight_variable([1024, len(captcha_list)*captcha_len])
    b_out = bias_variable([len(captcha_list)*captcha_len])
    y_conv = tf.matmul(h_drop_fc, w_out) + b_out
    return y_conv


def optimize_graph(y, y_conv):
    """
    优化计算图
    :param y: 正确值
    :param y_conv:  预测值
    :return: optimizer
    """
    # 交叉熵代价函数计算loss 注意logits输入是在函数内部进行sigmod操作
    # sigmoid_cross适用于每个类别相互独立但不互斥，如图中可以有字母和数字
    # softmax_cross适用于每个类别独立且排斥的情况，如数字和字母不可以同时出现
    loss = tf.reduce_mean(tf.nn.sigmoid_cross_entropy_with_logits(labels=y, logits=y_conv))
    # 最小化loss优化 AdaminOptimizer优化
    optimizer = tf.train.AdamOptimizer(1e-3).minimize(loss)
    return optimizer


def accuracy_graph(y, y_conv, width=len(CAPTCHA_LIST), height=CAPTCHA_LEN):
    """
    偏差计算图，正确值和预测值，计算准确度
    :param y: 正确值 标签
    :param y_conv:  预测值
    :param width:   验证码预备字符列表长度
    :param height:  验证码的大小，默认为4
    :return:    正确率
    """
    # 这里区分了大小写 实际上验证码一般不区分大小写,有四个值，不同于手写体识别
    # 预测值
    predict = tf.reshape(y_conv, [-1, height, width])   #
    max_predict_idx = tf.argmax(predict, 2)
    # 标签
    label = tf.reshape(y, [-1, height, width])
    max_label_idx = tf.argmax(label, 2)
    correct_p = tf.equal(max_predict_idx, max_label_idx)    # 判断是否相等
    accuracy = tf.reduce_mean(tf.cast(correct_p, tf.float32))
    return accuracy


def train(height=CAPTCHA_HEIGHT, width=CAPTCHA_WIDTH, y_size=len(CAPTCHA_LIST)*CAPTCHA_LEN):
    """
    cnn训练
    :param height: 验证码高度
    :param width:   验证码宽度
    :param y_size:  验证码预备字符列表长度*验证码长度（默认为4）
    :return:
    """
    # cnn在图像大小是2的倍数时性能最高, 如果图像大小不是2的倍数，可以在图像边缘补无用像素
    # 在图像上补2行，下补3行，左补2行，右补2行
    # np.pad(image,((2,3),(2,2)), 'constant', constant_values=(255,))

    acc_rate = 0.95     # 预设模型准确率标准
    # 按照图片大小申请占位符
    x = tf.placeholder(tf.float32, [None, height * width])
    y = tf.placeholder(tf.float32, [None, y_size])
    # 防止过拟合 训练时启用 测试时不启用 神经元使用率
    keep_prob = tf.placeholder(tf.float32)
    # cnn模型
    y_conv = cnn_graph(x, keep_prob, (height, width))
    # 优化
    optimizer = optimize_graph(y, y_conv)
    # 计算准确率
    accuracy = accuracy_graph(y, y_conv)
    # 启动会话.开始训练
    saver = tf.train.Saver()
    sess = tf.Session()
    sess.run(tf.global_variables_initializer())     # 初始化
    step = 0    # 步数
    while True:
        batch_x, batch_y = get_next_batch(64)
        sess.run(optimizer, feed_dict={x: batch_x, y: batch_y, keep_prob: 0.75})
        # 每训练一百次测试一次
        if not step % 100:
            batch_x_test, batch_y_test = get_next_batch(100)
            acc = sess.run(accuracy, feed_dict={x: batch_x_test, y: batch_y_test, keep_prob: 1.0})
            print(datetime.now().strftime('%c'), ' step:', step, ' accuracy:', acc)
            # 准确率满足要求，保存模型
            if acc > acc_rate:
                model_path = "./model/captcha.model"
                saver.save(sess, model_path, global_step=step)
                acc_rate += 0.01
                if acc_rate > 0.99:     # 准确率达到99%则退出
                    break
        step += 1
    sess.close()


if __name__ == '__main__':
    train()
```

**测试模块**：
```python
# -*- coding: utf-8 -*-
"""
@AUTH       Zau
@ADD TIME   2019/07/30
"""
# name: model_test.py

import tensorflow as tf
from model_train import cnn_graph
from captcha_gen import gen_captcha_text_and_image
from util import vec2text, convert2gray
from util import CAPTCHA_LIST, CAPTCHA_WIDTH, CAPTCHA_HEIGHT, CAPTCHA_LEN
from PIL import Image


def captcha2text(image_list, height=CAPTCHA_HEIGHT, width=CAPTCHA_WIDTH):
    """
    验证码图片转化为文本
    :param image_list:
    :param height:
    :param width:
    :return:
    """
    x = tf.placeholder(tf.float32, [None, height * width])
    keep_prob = tf.placeholder(tf.float32)
    y_conv = cnn_graph(x, keep_prob, (height, width))
    saver = tf.train.Saver()
    with tf.Session() as sess:
        saver.restore(sess, tf.train.latest_checkpoint('model/'))
        predict = tf.argmax(tf.reshape(y_conv, [-1, CAPTCHA_LEN, len(CAPTCHA_LIST)]), 2)
        vector_list = sess.run(predict, feed_dict={x: image_list, keep_prob: 1})
        vector_list = vector_list.tolist()
        text_list = [vec2text(vector) for vector in vector_list]
        return text_list


if __name__ == '__main__':
    text, image = gen_captcha_text_and_image()
    img = Image.fromarray(image)
    image = convert2gray(image)
    image = image.flatten() / 255
    pre_text = captcha2text([image])
    print("验证码正确值:", text, ' 模型预测值:', pre_text)
    img.show()
```


结果如图（y轴是成功率，x轴代表训练了多少轮）：

![y轴是成功率，x轴代表训练了多少轮](/images/posts/Identification_codes/trend_chart.png)

程序用的CPU跑的，大概每100轮训练耗时1分零几秒，一共跑了两天多。

大概跑了10000轮（不到两个小时）的时候成功率就稳定在70左右了。

20000轮（三个半小时）的时候成功率稳定在80左右。

第一次保存模型（测试成功率95以上）在第204800轮（十二个多小时），此时成功率大部分在90±3左右。

没等到第二次（第二次要求在上一次的基础上成功率提高1个百分点）。。。

成功率最高的也就95.5%，期间还有爆冷跌到50%出头，“复杂”验证码识别率99%还是相当难。

![识别这种验证码也是难为机器了](/images/posts/Identification_codes/failed.png)

识别这种验证码也是难为机器了。

不过总的来说还是尝到了甜头，毕竟不用切割图片什么的了，唯一要求的就是图片规格，这个好办。
这样一来，只要训练集够大，再适当调整一些参数，应该就可以实现4位数字字母验证码“通杀”！（想得倒是简单，代码都还没弄明白呢:)）

---
最后：

[零基础入门深度学习(1) - 感知器](https://www.zybuluo.com/hanbingtao/note/433855)

[零基础入门深度学习(2) - 线性单元和梯度下降](https://www.zybuluo.com/hanbingtao/note/448086)

[零基础入门深度学习(3) - 神经网络和反向传播算法](https://www.zybuluo.com/hanbingtao/note/476663)

[零基础入门深度学习(4) - 卷积神经网络](https://www.zybuluo.com/hanbingtao/note/485480)

[零基础入门深度学习(5) - 循环神经网络](https://zybuluo.com/hanbingtao/note/541458)

[零基础入门深度学习(6) - 长短时记忆网络(LSTM)](https://zybuluo.com/hanbingtao/note/581764)

[零基础入门深度学习(7) - 递归神经网络](https://zybuluo.com/hanbingtao/note/626300)

