---
layout:     post
title:      On-site Evaluation of a Software Cellular based MEC system with Downlink Slicing Technology笔记
subtitle:   论文阅读
date:       2019-09-18
author:     YunLambert
header-img: img/post-bg-paper1.jpg
catalog: true
tags:
    - Review
    - Paper
---

> On-site Evaluation of a Software Cellular based MEC system with Downlink Slicing Technology

这是一篇2018年关于边缘计算的文章，提出了关于Multi-access Edge Computing(MEC)的一种解决方案

## Introduction

为了收集和处理物联网应用在最佳的时间和地点，低延迟和宽带宽的宝贵数据是网络基础设施的关键要求。无人驾驶汽车本地动态映射时，利用车载摄像头和图像处理识别道路上的其他车辆、行人或障碍物，需要较宽的上行带宽和较低的延迟，而MEC能做到这一点。

MEC与5g网络的结合应该通过在用户设备(UE)附近处理数据来满足低时延和宽带宽的需求。MEC提供了诸如手机网络、光纤、无线网络等等可供接入的设备，从而具有较高的响应性和容量，可以容纳来自无线或有线接入网的大量流量。

然而当前部署的4G设备硬件如果全部替换为MEC，从经济上考虑不适用，所以作者认为将MEC软件化、同时蜂窝网络系统是降低MEC部署成本的救星。作者的想法是：如果移动网络运营商使用商品硬件将蜂窝网络设备迁移到软件设备，则可以将MEC功能安装到具有蜂窝网络功能的相同基础设施中，从而节省了MEC的额外成本。

接下来要解决的就是资源隔离问题，

这篇文章定义了蜂窝网络中MEC基础设施的软件体系结构。

资源隔离

LTE协议的核心网（EPC），基站（eNB）和用户（UE） 

