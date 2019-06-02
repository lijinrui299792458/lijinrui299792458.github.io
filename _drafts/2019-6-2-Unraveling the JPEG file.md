# Unraveling the JPEG file

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

![image.png](https://upload-images.jianshu.io/upload_images/7154520-2aac68685f15b1a6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Discrete Cosine Transform & Quantization

这一层压缩很大程度上是jpeg的定义特性。在将颜色转换为ycbcr之后，组件将被单独压缩，因此我们可以在本文的其余部分只关注y分量。下面是应用这个层的y组件的字节。



## JPEG解码流程

正如前面所说的一样，JPEG是一个标准，而不是单单一个算法。包含了多种压缩算法以及编码方式，也就是说两张jpeg图片之后使用的算法可能完全不同。

如果用压缩算法进行区分的话，可以分为sequential、progressive、hierarchical和lossless，其中sequential会从上至下解码：

![sequential.gif](https://upload-images.jianshu.io/upload_images/7154520-fa250551285c2968.gif?imageMogr2/auto-orient/strip)

progressive则会在解码过程中，从模糊逐步变得清晰：

![progressive.gif](https://upload-images.jianshu.io/upload_images/7154520-f4ed69767a8a6839.gif?imageMogr2/auto-orient/strip)

除了压缩算法，还有编码方式不同：霍夫曼编码、算术编码。除了压缩算法和编码方式，还有数据的粒度，可以理解为精度。先介绍所有JPEG规范组合中最简单的一种：baseline，它使用的压缩算法为sequential，编码方式采用的为霍夫曼编码，精度为8bit。而JPEG的编码与解码过程可以由下图表示：

![image.png](https://upload-images.jianshu.io/upload_images/7154520-aff9821cb4e81876.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中，DCT变换以及霍夫曼编码都是完全可逆的运算，如果只有这两个步骤，呢么得到的图片就是无损的。但是为了进一步提高压缩侠侣，baseline方法会抛弃掉一些"精细结构"，就像语音识别时会去掉音高等精细结构而只是保存采样数据一样。而从图中可以看出将颜色从RGB空间转换成YCbCr空间后降采样 以及 在DCT变换后的量化数据都是去掉这些精细结构。

## 一些概念

### DCT变换(离散余弦变换)



### 降采样

缩小图像(降采样)的目的主要有两个：1.使得图像符合显示区域的大小；2.生成对应图像的缩略图。原理是这样的：对于一幅图像尺寸为`M*N`，对其进行s倍 的下采样，即得到$(M/s)*(N/s)$ 储存的分辨率图像。如果考虑是矩阵形式的图像，就是把原始图像 $s*s$窗口内的图像变成一个像素，这个像素点就是窗口内所有像素的均值$P_k=\frac{\sum X_{i}}{S^{2}}$

### 霍夫曼编码



### 范式霍夫曼编码

![image.png](https://upload-images.jianshu.io/upload_images/7154520-9498cd101afb5753.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

左边就是霍夫曼树，让所有节点维持高度不变，但尽量往右移动，得到了右图。由于两棵树的叶子节点高度相等，它们的压缩率是完全相等的。我们用左侧这棵树来记录编码信息，需要把整棵树的结构都记录下来；但如果使用的是右侧的树，只需要记录在高度h时有几个叶子几点，就能够完全复原出右侧的霍夫曼树，所以范式霍夫曼树能够减少存储编码信息的空间用量。这个特性可以用于加速解码。

### YCbCr色彩空间

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

