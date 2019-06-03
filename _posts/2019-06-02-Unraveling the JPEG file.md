---
layout:     post
title:      Unraveling the JPEG file
subtitle:   了解一下JPEG
date:       2019-06-02
author:     YunLambert
header-img: img/post-bg-JPEG.png
catalog: true
tags:
    - Review
    - Share
---

> Unraveling the JPEG file

首先这篇文章其实还有一部分没有写完，之所以提前放出来是因为近期要毕设答辩，并且剩下的一部分实际上是实际写一个JPEG解码器部分，如果没有意外的话想把这部分放到近期在写的一个个人项目中，然后会在这个项目中对实现JPEG解码器进行详细介绍。如果想直接看时间部分的话可以前往MROS的Github上，他最近用了rust语言书写了一个教程，本文后半段参考了教程。下面进入正文：

JPEG文件在当下数字化生活中是无处不在的，但是在熟悉的JPEG面纱背后，隐藏着一些算法，它们去除了人类眼中无法察觉到的细节。这产生了最高的视觉质量与最小的文件大小。让我们来看看这一算法。

## 引言

我们发送图片给朋友的时候，并不需要担心哪个设备或者浏览器或者操作系统的兼容性，我们只是把图片"发出去"。但是就像网络传输一样，只是传输却没有其他的措施会造成兼容性极差，所以需要一种协议或者一种标准。于是一组人在1992年创建了JPEG，一种数字图像压缩的标准。任何曾经使用过互联网的人都可能看到过JPEG编码的图像。它是迄今为止最普遍的编码、发送和存储图像的方式。从网页到电子邮件到社交媒体，JPEG每天被使用数十亿次-几乎每次我们在网上查看或发送图片。没有JPEG，网络就会少一点色彩并且慢很多。

所以这篇文章里将介绍一下关于如何将存储在计算机上的压缩数据转换为显示在屏幕上的图像。当然此文首先会对[该文](https://parametric.press/issue-01/unraveling-the-jpeg/)进行部分翻译、然后结合MROS大神写的一个[JPEG教程](https://github.com/MROS/jpeg_tutorial)进行讲解，当然其中会加上自己的理解进行整理和验证。如果能够真正理解图像的压缩步骤，感觉对之后的图像处理这一部分的学习也是有帮助的。

## 开始

首先双手奉上女神的照片：

![songzhixiao.jpeg](https://upload-images.jianshu.io/upload_images/7154520-faa5705f61a82258.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来我们就拿这张图片作为例子来讲解，如果我们用文字编辑器打开图片就会发现是一堆乱码：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-cecef038a53ebf6c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为这是文字编辑器不能识别的格式，用文字编辑器打开图片，既让计算机感到疑惑同时也不能让我们得到正确的可视结果。因为我们的打开方式不正确，我们如果要理解JPEG图像是如何解码的，我们需要查看原始流，即查看二进制数据。可以利用专门查看hex编码的编辑器来查看：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-65edfeacb022f816.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们现在随意修改其中的一些数字，可以发现有些数字的更改是无伤大雅的，但是有一些数字一旦修改可能会损坏整个图像，或者造成图像颜色和位置发生变动。

![image.png](https://upload-images.jianshu.io/upload_images/7154520-2f551db46507553c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/7154520-b6ac99fdc25f95bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

很难通过这样的方式来解释如何从这些字节中重建图像，因为jpeg压缩实际上是由三种不同的压缩技术组成的，它们被应用在连续的层中。我们将分别查看每一层压缩，以解开我们正在看到的神秘行为。

## JPEG压缩的三层

1.色度次采样(Chrominance Subsampling)

2.离散余弦变换(Discrete Cosine Transform & Quantization)

3.扫描宽度、霍夫曼编码(Run-length,Delta&Huffman Encoding)

首先我们需要了解这种压缩的规模，请注意上面的图像是使用确切的79，819个数字表示的，大约是79千字节。如果没有压缩存储，则每个像素需要三个数字-每个红色、绿色和蓝色组件都需要一个数字。这意味着总共有917，700个数字，约917千字节。使用jpeg压缩，结果文件要小十倍以上！而jpeg被称为有损压缩技术，图像由于压缩而改变并丢失了一些细节

### Chrominance Subsampling

现在破译要简单一些。这几乎是一个简单的颜色列表，每个字节只改变一个像素，但它已经几乎是未压缩图像的两倍(对于这个较小的大小，大约是300 kb)。可以看出，这些数字并不代表标准的红色、绿色和蓝色组件，因为用0替换所有数字会使图像变成绿色(而不是黑色)。这是因为这些字节表示图像的y(亮度)、cb(相对蓝度)和cr(相对红度)。为什么不直接用RGB呢？毕竟，这就是大多数现代屏幕的工作原理。您的显示器可以通过打开每个像素的不同强度的红色、绿色和蓝色灯来显示任何颜色。白色是通过打开显示的。

![image.png](https://upload-images.jianshu.io/upload_images/7154520-be5de7332167349b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这也和人类眼睛的工作原理非常相似。我们眼中的颜色受体被称为“锥”，分为三种类型，每种类型对红色、绿色或蓝色都很敏感。我们眼中的另一种感受器-棒，只能检测到亮度的变化，但它们要敏感得多。我们的眼睛里有大约1.2亿根棒状物。

这意味着我们的眼睛在检测亮度变化方面比在检测颜色变化方面要好得多。如果我们能把颜色和亮度分开，我们就可以在没有人注意到的情况下去除一点颜色。色度次采样是以比亮度分量更低的分辨率来表示图像的颜色分量的过程。在上面的例子中，每个像素都有一个y分量，而每个由四个像素组成的离散组正好有一个CB和一个cr分量。因此，图像中包含的颜色信息只有最初的四分之一。这就是为什么从上面的编辑器中删除一个数字完全破坏了颜色。在这里，组件存储为y cbcr。删除第一个数字将使CB值被解释为y，cr被解释为CB，并创建一个波动效应，在图像中翻转所有颜色。

![image.png](https://upload-images.jianshu.io/upload_images/7154520-5d4c98aead897c15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



### Discrete Cosine Transform & Quantization

这一层压缩很大程度上是jpeg的定义特性。在将颜色转换为ycbcr之后，组件将被单独压缩，因此我们可以在本文的其余部分只关注y分量。下面是应用这个层的y组件的字节。

![image.png](https://upload-images.jianshu.io/upload_images/7154520-6d30d89269ccea8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

离散余弦变换是将图像分解成8x8块并将每个块转换成这64个系数的组合的过程。

![image.png](https://upload-images.jianshu.io/upload_images/7154520-0df46684de4abbf8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/7154520-a3bf2947abe118d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果我们并不是对`8*8`像素点而是对整个独享进行离散余弦变换，得到的就为上图。并且可以不断删除结尾处的数据，但是一旦变更开头处的数据，就会造成肉眼可见的亮度变化。原因是开头处为图像的低频变化，结尾处为高频变化。

### Run-Length, Delta & Huffman Encoding

run-length encoding是用三个bytes去指定颜色，还有一个byte去指定有多少个像素有这个颜色。

delta-encoding是记录变化量而不是原值，比如说存储值为`12 13 14 13 2`，则就可以表示为：`12 1 1 -1 -1`。每个DCT块中的第一个值称为dc值，每个dc值相对于前面的dc值进行增量编码。因此，改变第一个块的亮度将影响图像中的所有块。

## JPEG解码流程

正如前面所说的一样，JPEG是一个标准，而不是单单一个算法。包含了多种压缩算法以及编码方式，也就是说两张jpeg图片之后使用的算法可能完全不同。

如果用压缩算法进行区分的话，可以分为sequential、progressive、hierarchical和lossless，其中sequential会从上至下解码：

![sequential.gif](https://upload-images.jianshu.io/upload_images/7154520-fa250551285c2968.gif?imageMogr2/auto-orient/strip)

progressive则会在解码过程中，从模糊逐步变得清晰：

![progressive.gif](https://upload-images.jianshu.io/upload_images/7154520-f4ed69767a8a6839.gif?imageMogr2/auto-orient/strip)

除了压缩算法，还有编码方式不同：霍夫曼编码、算术编码。除了压缩算法和编码方式，还有数据的粒度，可以理解为精度。先介绍所有JPEG规范组合中最简单的一种：baseline，它使用的压缩算法为sequential，编码方式采用的为霍夫曼编码，精度为8bit。而JPEG的编码与解码过程可以由下图表示：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-aff9821cb4e81876.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中，DCT变换以及霍夫曼编码都是完全可逆的运算，如果只有这两个步骤，呢么得到的图片就是无损的。但是为了进一步提高压缩效率，baseline方法会抛弃掉一些"精细结构"，就像语音识别时会去掉音高等精细结构而只是保存采样数据一样。而从图中可以看出将颜色从RGB空间转换成YCbCr空间后降采样 以及 在DCT变换后的量化数据都是去掉这些精细结构。

## 一些概念

### 降采样

缩小图像(降采样)的目的主要有两个：1.使得图像符合显示区域的大小；2.生成对应图像的缩略图。原理是这样的：对于一幅图像尺寸为`M*N`，对其进行s倍 的下采样，即得到$(M/s)*(N/s)$ 储存的分辨率图像。如果考虑是矩阵形式的图像，就是把原始图像 $s*s$窗口内的图像变成一个像素，这个像素点就是窗口内所有像素的均值$P_k=\frac{\sum X_{i}}{S^{2}}$

### 范式霍夫曼编码

![image.png](https://upload-images.jianshu.io/upload_images/7154520-9498cd101afb5753.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

左边就是霍夫曼树，让所有节点维持高度不变，但尽量往右移动，得到了右图。由于两棵树的叶子节点高度相等，它们的压缩率是完全相等的。我们用左侧这棵树来记录编码信息，需要把整棵树的结构都记录下来；但如果使用的是右侧的树，只需要记录在高度h时有几个叶子几点，就能够完全复原出右侧的霍夫曼树，所以范式霍夫曼树能够减少存储编码信息的空间用量。这个特性可以用于加速解码。

例如用0b0代表A，0b10代表B，0b11代表C，则AABCAAA压缩为0b001011000.

### YCbCr色彩空间

RGB、YUV和YCbCr都是人为规定的色彩模型或颜色空间（有时也叫彩色系统或彩色空间）。它的用途是在某些标准下用通常可接受的方式对彩色加以描述。本质上，色彩模型是坐标系统和子空间的阐述。

![image.png](https://upload-images.jianshu.io/upload_images/7154520-9a9dd080fd071736.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中RGB是我们常见的红蓝绿，依据人眼识别的颜色定义出的空间。但是RGB的细节很难进行数字化的调整，它将色调、亮度、饱和度三个量放在一起表示。是最通用的面向硬件的彩色模型。该模型用于彩色监视器和一大类彩色视频摄像。每个像素点需要`8bits*3 = 24bits`

而YUV空间中，每一个颜色有一个亮度信号Y和两个色度信号U和V，亮度信号时强度的感觉，和色度信号断开，这样的话强度就能够在不影响颜色的情况下改变。YUV使用RGB的信息，如果只有Y信号分量而没有U、V分量，那么这样表示的图像就是黑白灰度图像。彩色电视采用YUV空间正是为了用亮度信号Y解决彩色电视机与黑白电视机的兼容问题，使黑白电视机也能接收彩色电视信号。

```
YUV和RGB的转换:
      Y = 0.299 R + 0.587 G + 0.114 B
      U = -0.1687 R - 0.3313 G + 0.5 B + 128
      V = 0.5 R - 0.4187 G - 0.0813 B + 128

      R = Y + 1.402 (V-128)
      G= Y - 0.34414 (U-128) - 0.71414 (V-128)
      B= Y + 1.772 (U-128)
```

YCbCr 其实是YUV经过缩放和偏移的翻版。其中Y与YUV 中的Y含义一致,Cb,Cr 同样都指色彩，只是在表示方法上不同而已。YCbCr其中Y是指亮度分量，Cb指蓝色色度分量，而Cr指红色色度分量。JPEG、MPEG均采用此格式。

```
 YCbCr与RGB的相互转换
　　  Y=0.299R+0.587G+0.114B
　　  Cb=0.564(B-Y)
　　  Cr=0.713(R-Y)

　    R=Y+1.402Cr
　    G=Y-0.344Cb-0.714Cr
      B=Y+1.772Cb
```

YCbCr 有许多取样格式，主要的采样格式有YCbCr 4:2:0、YCbCr 4:2:2、YCbCr 4:1:1和 YCbCr 4:4:4。其中YCbCr 4:1:1 比较常用，其含义为：每个点保存一个 8bit 的亮度值（也就是Y值），每 2x2 个点保存一个 Cr 和Cb 值，图像在肉眼中的感觉不会起太大的变化。所以，原来用 RGB(R,G,B 都是 8bit unsigned) 模型，每个点需要 8x3=24 bits（如下图第一个图）. 而仅需要 8+（8/4）+（8/4）=12bits，平均每个点占12bits。这样就把图像的数据压缩了一半。

再来具体解释一下主要的采样格式：

(1)YUV 4:4:4

YUV三个信道的抽样率相同，因此在生成的图像里，每个象素的三个分量信息完整（每个分量通常8比特），经过8比特量化之后，未经压缩的每个像素占用3个字节。
下面的四个像素为： `[Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]`
存放的码流为： Y0 U0 V0 Y1 U1 V1 Y2 U2 V2 Y3 U3 V3

(2)YUV 4:2:2

每个色差信道的抽样率是亮度信道的一半，所以水平方向的色度抽样率只是4:4:4的一半。对非压缩的8比特量化的图像来说，每个由两个水平方向相邻的像素组成的宏像素需要占用4字节内存。
下面的四个像素为：` [Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]`
存放的码流为： Y0 U0 Y1 V1 Y2 U2 Y3 V3
映射出像素点为：`[Y0 U0 V1] [Y1 U0 V1] [Y2 U2 V3] [Y3 U2 V3]`

(3)YUV 4:1:1

4:1:1的色度抽样，是在水平方向上对色度进行4:1抽样。对于低端用户和消费类产品这仍然是可以接受的。对非压缩的8比特量化的视频来说，每个由4个水平方向相邻的像素组成的宏像素需要占用6字节内存
下面的四个像素为：` [Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]`
存放的码流为： Y0 U0 Y1 Y2 V2 Y3
映射出像素点为：`[Y0 U0 V2] [Y1 U0 V2] [Y2 U0 V2] [Y3 U0 V2]`

(4)YUV 4:2:0

4:2:0并不意味着只有Y,Cb而没有Cr分量。它指得是对每行扫描线来说，只有一种色度分量以2:1的抽样率存储。相邻的扫描行存储不同的色度分量，也就是说，如果一行是4:2:0的话，下一行就是4:0:2，再下一行是4:2:0...以此类推。对每个色度分量来说，水平方向和竖直方向的抽样率都是2:1，所以可以说色度的抽样率是4:1。对非压缩的8比特量化的视频来说，每个由2x2个2行2列相邻的像素组成的宏像素需要占用6字节内存。
下面八个像素为：`[Y0 U0 V0] [Y1 U1 V1] [Y2 U2 V2] [Y3 U3 V3]
[Y5 U5 V5] [Y6 U6 V6] [Y7U7 V7] [Y8 U8 V8]`
存放的码流为：Y0 U0 Y1 Y2 U2 Y3
Y5 V5 Y6 Y7 V7 Y8
映射出的像素点为：`[Y0 U0 V5] [Y1 U0 V5] [Y2 U2 V7] [Y3 U2 V7]
[Y5 U0 V5] [Y6 U0 V5] [Y7U2 V7] [Y8 U2 V7]`


## 读取区段

JPEG压缩文件可能是由以下几个区段组成的：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-a9df73b1dcb88054.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

除此之外，我们需要在JPEG开始和结尾处加上"文件开始"和"文件结束"标记，并且采用何种算法会和降采样在同一个区段；压缩图像数据会前准一个SOS区段，告知解码器接下来该如何读取。综上可以将JPEG压缩文件表示为：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-fa6602c98b476b09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在一个JPEG文件中，一定会出现`SOF0`、`SOF1`....`SOF16`其中一个区段，如果出现的是`SOF0`就说明使用的为baseline。DQT和DHT都可能不止一个：

| 区段名称 | 标记码 | 有无数据 |
| :------: | :----: | :------: |
| SOI      | 0xFFD8 |    NO    |
| EOI      | 0xFFD9 |    NO    |
| DQT      | 0xFFDB |   YES    |
| DHT      | 0xFFC4 |   YES    |
| SOF0     | 0xFFC0 |   YES    |
| SOS      | 0xFFDA |   YES    |

如果图像信息里包含了0xff怎么办？所以在规定压缩图像时会在0xff后加上一个0x00的字节，检测时如果遇到0xff00直接跳过就行了。而又因为这里一个码有2个字节，所以在写程序是可以两个字节两个字节的读。

而在baseline中，文件开头、文件结尾、JFIF特有资讯实际上都已经确定了，所以接下来只需要确定量化表(DQT)、霍夫曼表、压缩数据、SOF0和SOS就行了，一步一步来。

### 读取DQT

一个DQT里最多可能有4个量化表，所以量化表的存放在不同程序下可能会使用不同的方式下面是一个量化表的内容：

| 代号 | 大小           | 描述                                                         |
| ---- | -------------- | ------------------------------------------------------------ |
| ①    | 1 byte         | 高 4 bits 表示每個量化值的大小，0 代表 1 byte，1 代表 2 bytes 低 4 buts 表示本量化表的 id ， id 可爲 0, 1, 2, 3 |
| ②    | 64 或 128 byte | 64 個 1 或 2 bytes 的量化值，是 1 byte 還是 2 bytes 取決於 ① 的高 4 bits |



### 读取霍夫曼编码表

| 代号 | 大小              | 描述                                                         |
| ---- | ----------------- | ------------------------------------------------------------ |
| ①    | 1 byte            | 高 4 bits 代表本表是直流還是交流， 0 是直流， 1 是交流 低 4 bits 代表是 0 號表還是 1 號表， 0 是 0 號 ， 1 是 1 號 |
| ②    | 16 bytes          | 第 n 個 byte 代表高度爲 n 時，霍夫曼樹有幾個葉子節點         |
| ③    | 叶子节点数量bytes | 代表叶子节点所对应的信源符号，叶子从低到高，从左到右         |

![image.png](https://upload-images.jianshu.io/upload_images/7154520-10b2c4043bc75088.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

红色就是直流1号，紫色部分就为形成的范式霍夫曼树的叶节点数目，叶子节点应该有2+2+5+1+5+1=16，所以绿色部分应该有16个数值代表叶子节点上对应的信源符号；如果不足16个数值，说明这个DHT区段可能包含多个霍夫曼表，在下一个霍夫曼表中继续读取。



### 读取SOF0区段

SOF0区段记录了图片宽高以及各个颜色分量的信息，有了这些数据我们才能够读取SOS段中的压缩图像数据，不过先看SOF0区段：

| 代號 | 大小    | 描述                   | 備註                                                         |
| ---- | ------- | ---------------------- | ------------------------------------------------------------ |
| ①    | 1 byte  | 精度                   | baseline 流程的精度固定爲 8 ，也就是說用 1 byte 來存取色彩空間的數值即可 |
| ②    | 2 bytes | 圖片高度               |                                                              |
| ③    | 2 bytes | 圖片寬度               |                                                              |
| ④    | 1 byte  | 有幾個顏色分量         | JFIF 規定顏色空間爲 YCbCr 因此顏色分量數量固定爲 3           |
| ⑤    | 9 bytes | 各個顏色分量的詳細資訊 | 見下一張表                                                   |

上表中的5是下表重复三次，因为需要分别描述3个颜色分量的信息：

| 代號 | 大小   | 描述        | 備註                                                         |
| ---- | ------ | ----------- | ------------------------------------------------------------ |
| ①    | 1 byte | 顏色分量 id | 1 表示 Y ， 2 表示 Cb ， 3 表示 Cr                           |
| ②    | 1 byte | 採樣率      | 高 4 bit 代表水平採樣率 ，低 4 bit 代表垂直採樣率。採樣率可以是 1, 2, 3, 4 |
| ③    | 1 byte | 量化表 id   | 最後解碼時，要選擇哪一個量化表對該顏色分量進行量化           |

### 读取SOS区段

| 代號 | 大小        | 描述                   | 備註                                   |
| ---- | ----------- | ---------------------- | -------------------------------------- |
| ①    | 1 byte      | 有幾個顏色分量         |                                        |
| ②    | 2 * 3 bytes | 顏色分量對應的霍夫曼表 | 每個顏色分量佔用 2 bytes ， 見下一張表 |
| ③    | 3 bytes     | baseline 流程用不到    | 在 baseline 流程固定爲 0x003F00        |

② 的內容爲下表重重复3 次：

| 代號 | 大小   | 描述              | 備註                                                         |
| ---- | ------ | ----------------- | ------------------------------------------------------------ |
| ①    | 1 byte | 顏色分量 id       | 1 表示 Y，2 表示 Cb，3 表示 Cr                               |
| ②    | 1 byte | 對應的霍夫曼表 id | 高 4 bit 代表直流（DC）霍夫曼表，低 4 bit 代表交流（AC）霍夫曼表 |

### 读取压缩图像数据

數據流切割來看，可以看成一個接一個的 MCU 串:

[MCU1], [MCU2], [MCU3], [MCU4], ....

而一個 MCU 又可以進一步切割成（假設 Y 水平、垂直採樣率爲 2 ，Cb, Cr 水平、垂直採樣率爲 1）不同顏色分量的串連，以 Y -> Cb -> Cr 的順序:

[Y1, Y2, Y3, Y4, Cb1, Cr1]

這裏的 Y1, Y2, .... Cr1 ，都是一個 8 * 8 的 block 了，所以之後我們只要瞭解一個 block 如何讀取，就大功告成了。

### 读取一个block

每个block都是`8*8`的，最左上角的数值就是直流变量，要使用直流霍夫曼表来解码，而与下的63个数值为交流变量。

![image.png](https://upload-images.jianshu.io/upload_images/7154520-8648b05d3ddea869.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## JPEG分块机制

因为再sequential算法中，是从上至下边收数据边进行解码的，所以说明JPEG文件必然是分块进行压缩的，因为不需要知道所有的数据就已经可以开始解码：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-e1c04457fc344335.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MCU是最小编码单元Minimum Coded Unit的缩写(通信协议上某一层上所能通过的最大数据包大小的概念为MTU)。MCU的宽高并不固定为16px也不一定是正方形，而是取决于颜色分量的采样率。

![image.png](https://upload-images.jianshu.io/upload_images/7154520-11c2471dc809d116.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

JPEG使用的为YCbCr色彩空间，采样图如上。每个颜色分量的最基本单元是block，一个block的宽、高固定为`8*8`：

```
MCU的宽 = 8 * 最高水平采样率
MCU的高 = 8 * 最高垂直采样率
```

Cb、Cr的最左上角对应到了MCU的左上4个px：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-eebb9569b6904098.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

MCU从左到右、再从上到下：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-7624bb8a588f97f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一个颜色分量内部各个block的顺序一样是从左到右、从上到下：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-e583d3e27a780f4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### MCU、颜色分量、block关系

读取图像压缩数据时，需要先读取MCU信息，在读取MCU时又需要读取block的信息。

## 解码 decode!

由于已经将JPEG文件的所有区段全部读取出来了，接下来就是简单的转换，就能够还原为原始图像数据，接下来将按照步骤来依次转换。

### 反量化

从DQT区段中已经读出了量化表，在SOF0区段中也知道了各个颜色分量所对应的量化表id，一个block和一个量化表都是`8*8`，将它们对应的位置相乘就完成这个转换了。

### 反zigzag

zigzag时，会将一个block以下图的顺序重新排列：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-ea7602c55221e2ea.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

反zigzag就是要将这个过程逆转过来进行还原

### 反向DCT变换

![image.png](https://upload-images.jianshu.io/upload_images/7154520-9d9031422e839868.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 升采样与YCbCr转RGB颜色空间

照着公式来变换即可。

最后将图像的各个MCU拼接起来就可以得到原图了。

## 一个实例

下面给出一个将JPEG文件转换成bmp文件的程序：[https://github.com/MROS/jpeg_decoder/blob/master/main.cpp](https://github.com/MROS/jpeg_decoder/blob/master/main.cpp)