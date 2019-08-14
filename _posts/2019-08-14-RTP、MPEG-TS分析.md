---
layout:     post
title:      RTP、MPEG-TS分析
subtitle:   视音频开发
date:       2019-08-14
author:     YunLambert
header-img: img/post-bg-RTP.jpg
catalog: true
tags:
    - Tips
    - Share
    - Media
---

> RTP、MPEG-TS分析

是时候祭出这张视频播放器原理图了：

![视音频结构图](https://upload-images.jianshu.io/upload_images/7154520-53740ac0e32dff18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

视频播放器播放一个互联网上的视频文件，首先从网上获取视频数据，然后对数据进行解协议，得到了封装格式数据，此封装格式内包含了音频与视频两个部分。对此封装格式进行解封装，分别提取其种的音频压缩数据和视频压缩数据，再分别进行解码获得音频与视频原始数据，原始数据进行视音频同步后进入音频驱动和视频驱动进行播放。

前面的文章里已经陆陆续续的介绍了原始数据、解码、解封装，现在剩**协议**这一部分没有提及了。

解协议的作用就是将流媒体协议的数据，解析为标准的相应封装格式数据。因为视音频在网络上传播的时候，常常采用各种流媒体协议，例如HTTP、RTMP或是MMS等等，这些协议会传输视音频数据以及一些包含播放、暂停、停止的信令数据。解协议的过程中会去掉信令数据而只保留视音频数据。当然如果只是播放本地的文件，就不会涉及到解协议这一步骤。

## 什么是RTP

RTP是一种实时传输协议，在多点传送（多播）或单点传送（单播）的网络上，提供端对端的网络传输功能，适合应用程序传输实时数据，如：音频，视频或者仿真数据。RTP没有为实时服务提供资源预留的功能，也不保证QoS（服务质量）。

常用的流媒体协议主要有HTTP渐进下载(HLS)和基于RTSP/RTP的实时流媒体协议。

有一篇[RTP协议全解析](https://blog.csdn.net/chen495810242/article/details/39207305)写的不错，可以参考一下。

### HTTP、RTSP、RTMP、HLS区别

- 共同点：

1. 都是在应用层；
2. 理论上都可以做直播和点播，但一般直播用RTSP/RTMP，点播用HTTP；视频会议用RTMP；

- 区别：

1. HTTP将搜有数据作为文件做处理，不是流媒体协议；RTMP和RTSP是流媒体协议；HTTP协议简单，流媒体协议复杂；
2. RTMP是Adobe的私有协议，未完全公司，RTSP/HTTP协议是共有协议，有专门机构做维护；
3. RTMP是flv/f4v格式流，RTSP协议一般传输的是ts/mp4格式的流，HTTP没有特定的流；
4. RTSP传输一般需要2-3个通道，命令和数据通道分离，HTTP/RTMP一般在一个TCP通道上传输命令和数据；
5. HLS主要延时比较大，RTMP/RTSP优势在于延时低；

### RTP Header解析

![rtp header](https://upload-images.jianshu.io/upload_images/7154520-eb7b06842a63e265.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

取一段码流：

```
80 e0 00 1e 00 00 d2 f0 00 00 00 00 41....
```

`0000d2f0`是Timestamp，`00000000`是SSRC，把前2字节`80 e0`换成二进制如下：`1000000011100000`，按从左到右的顺序为：10是V，0是P，0是X，0000是CC，1是M，1100000是PT，具体的请见下方：

```
P：填充标志，占1位，如果P=1，则在该报文的尾部填充一个或多个额外的八位组，它们不是有效载荷的一部分。

X：扩展标志，占1位，如果X=1，则在RTP报头后跟有一个扩展报头

CC：CSRC计数器，占4位，指示CSRC 标识符的个数

M: 标记，占1位，不同的有效载荷有不同的含义，对于视频，标记一帧的结束；对于音频，标记会话的开始。

PT: 有效荷载类型，占7位，用于说明RTP报文中有效载荷的类型，如GSM音频、JPEM图像等,在流媒体中大部分是用来区分音频流和视频流的，这样便于客户端进行解析。

序列号：占16位，用于标识发送者所发送的RTP报文的序列号，每发送一个报文，序列号增1。这个字段当下层的承载协议用UDP的时候，网络状况不好的时候可以用来检查丢包。同时出现网络抖动的情况可以用来对数据进行重新排序，序列号的初始值是随机的，同时音频包和视频包的sequence是分别记数的。

时戳(Timestamp)：占32位，必须使用90 kHz 时钟频率。时戳反映了该RTP报文的第一个八位组的采样时刻。接收者使用时戳来计算延迟和延迟抖动，并进行同步控制。

同步信源(SSRC)标识符：占32位，用于标识同步信源。该标识符是随机选择的，参加同一视频会议的两个同步信源不能有相同的SSRC。

特约信源(CSRC)标识符：每个CSRC标识符占32位，可以有0～15个。每个CSRC标识了包含在该RTP报文有效载荷中的所有特约信源。
```

### RTP荷载H.264码流

荷载格式定义三个不同的基本荷载结构，接收者可以通过RTP荷载的第一个字节后5位识别荷载结构。 

```
荷载格式定义三个不同的基本荷载结构，接收者可以通过RTP荷载的第一个字节后5位识别荷载结构。

1)单个NAL单元包：荷载中只包含一个NAL单元。NAL头类型域等于原始 NAL单元类型,即在范围1到23之间

2)聚合包：本类型用于聚合多个NAL单元到单个RTP荷载中。本包有四种版本,单时间聚合包类型A (STAP-A)，单时间聚合包类型B (STAP-B)，多时间聚合包类型(MTAP)16位位移(MTAP16), 多时间聚合包类型(MTAP)24位位移(MTAP24)。赋予STAP-A, STAP-B, MTAP16, MTAP24的NAL单元类型号分别是 24,25, 26, 27

3)分片单元：用于分片单个NAL单元到多个RTP包。现存两个版本FU-A，FU-B,用NAL单元类型 28，29标识

常用的打包时的分包规则是：如果小于MTU采用单个NAL单元包，如果大于MTU就采用FUs分片方式。

因为常用的打包方式就是单个NAL包和FU-A方式，所以我们只解析这两种。
```

## MPEG-TS格式解析

MPEG-TS一种标准数据容器格式，传输与存储音视频、节目与系统信息协议数据，应用于数字广播系统，譬如DVB,ATSC与IPTV。 

MPEG-2标准中，有两种不同的码流输出到信道，一种是节目码流（PS: Program Stream），适用于没有传输误差的场景；一种是传送流（TS：Transport Stream)，适用于有信道噪声的传输场景。节目流设计用于合理可靠的媒体，如光盘（如DVD），而传输流设计用于不太可靠的传输，即地面或卫星广播。此外，传输流可以携带多个节目。

mpeg-ts封装的文件在互联网上已经随处可见了，因为它的体系非常简单，试想有一个工具可以把一个或多个视频文件切割成一堆的小ts文件，并且生成一个内容非常简单的`m3u8`文件，然后只需要自己搭建一个标准的web server就能提供点播视频流媒体服务了。加入可以对直播数据流进行切片存储成小文件，就具备了直播流媒体服务能力。

### TS介绍

TS全称Transport Stream，是基于MPEG-2的封装格式，所以也叫MPEG-TS，通常后缀为`.ts`、`.mpeg`，在[H.264视频码流解析](http://blog.yunlambert.top/2019/08/07/H.264%E8%A7%86%E9%A2%91%E7%A0%81%E6%B5%81%E8%A7%A3%E6%9E%90/)这一篇文章里提到过TS流，只是在应用的时候泛泛而谈，将一个一个ts文件组合起来就是最终形成的视频文件了。

而由连续的固定字节的TS包组成的为一路TS比特流，包含的内容通常有：

- 一路或多路视频流
- 一路或多路音频流
- 一路或多路字幕
- PSI表格信息
- PES

首先在PSI表格信息中包括PAT(节目关联表)和PMT(节目映射)和CAT(条件接收表)和NIT(网络信息表)，PAT描述有多少路节目，每路节目的PMT表的PID是多少。

```
找个一个TS包，它的PID是0，说明它的负载内容是PAT信息，解析PAT信息，你发现节目1的PMT表的PID是0x10，接着，你在比特流中寻找一个PID为0x10的TS包，它的负载内容是节目1的PMT表信息，解析该PMT信息，你可以发现第一路流是MPEG2音频流，PID号0x21，第二路流是MPEG2视频流，PID号是0x22，第三路流是DVB字幕流，PID号是0x23，解析完毕，凡是比特流中PID号为0x22的TS包，所负载的内容为MPEG2视频流，把这些包一个一个找出来，把其中的有效码流一部分一部分的拼接起来，然后送给解码器去解码。
```

![PAT](https://upload-images.jianshu.io/upload_images/7154520-468e2f9a6261ec05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

TS流的最小数据单元是188字节，所以可以认为一个TS文件的长度一定是188的整数倍。

### TS解封装的原理

因TS是对基本码流的封装，所以TS也应该是分层结构，TS主要分为三层：TS层、PES层、ES层。ES层就是压缩编码(视频->H.264，音频->AAC)后的基本音频码流，PES在ES层的基础上加入了时间戳(PTS/DTS)等信息；TS层就是在PES层加入了定时信息(PCR)和节目专用信息(PSI)进而形成所需要的稳定MPEG-TS码流

TS流的封装过程如下：

1) 将原始音视频数据压缩之后，压缩结果组成一个基本码流（ES）。
2) 对ES（基本码流）进行打包形成PES。
3) 在PES包中加入时间戳信息(PTS/DTS)。
4) 将PES包内容分配到一系列固定长度的传输包（TS Packet）中。
5) 在传输包中加入定时信息(PCR)。
6) 在传输包中加入节目专用信息(PSI) 。
7) 连续输出传输包形成具有恒定比特率的MPEG-TS流。

反过来就是解析过程：

1) 从复用的MPEG-TS流中解析出TS包；
2) 从TS包中获取PAT及对应的PMT（PSI中的表格）；
3) 从而获取特定节目的音视频PID；
4) 通过PID筛选出特定音视频相关的TS包，并解析出PES；
5) 从PES中读取到PTS/DTS，并从PES中解析出基本码流ES；
6) 将ES交给解码器，获得压缩前的原始音视频数据

### 一个结构图

![TS结构](https://upload-images.jianshu.io/upload_images/7154520-9911c77e7b27a4e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面对这张图进行解析

### TS层的结构分析

上面已经提到了TS每个包的大小固定位188字节，其中TS header固定4个字节，adapation field为自适应区，payload为有效负载。

TS包头部字段名如下，共4个字节：

![ts包格式](https://upload-images.jianshu.io/upload_images/7154520-33f7e5f6e9528ed8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

程序如下：

```C
typedef struct Transport_Packet{
    unsigned Sync_byte                       :8 //同步标志。值固定0x47
    unsigned Transport_error_indicator       :1 //传输错误指示符。为1时代表出错
    unsigned Payload_unit_start_indicator    :1 //有效负载起始指示符。为1时代表是一帧的开始
    unsigned Transport_priority              :1 //传输优先级，为1代表优先级高于相同PID的字段
    unsigned PID                             :13 
    unsigned Transport_scrambling_control    :2 //加扰方式。00未加扰，其他值用户填写
    unsigned Adapaction_field_control        :2 //当前包是否有调整字段或者有效负载。00保留，01无调整字段有有效负载，10有调整字段无有效负载，11两者都有
    unsigned Continuity_counter              :4 //从0到15连续循环递增，代表连续的包，不一定从0开始
}
```

TS Header中主要是一些控制操作，是每个TS包的起始位置。

![TS层](https://upload-images.jianshu.io/upload_images/7154520-14ecd2a921b35bf6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### PES层的结构分析

 PES层主要分为三个部分PES header（固定6个字节），optional header（可选），和pes payload（也就是ES层了）。 

![PES](https://upload-images.jianshu.io/upload_images/7154520-8aee3c913c7ffa48.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ES层的结构分析

ES是最基本的视音频数据，在TS封装格式中视频采用H.264压缩编码，音频采用AAC压缩编码。关于这个的压缩编码在之前的文章里已经提及过了。


## 应用：通过Socket编程收取UDP包，解析其中的RTP包的信息和RTP包中的MPEG-TS Packet信息

来自雷神

```C++
#include <stdio.h>
#include <winsock2.h>
 
#pragma comment(lib, "ws2_32.lib") 
 
#pragma pack(1)
 
/*
 * [memo] FFmpeg stream Command:
 * ffmpeg -re -i sintel.ts -f mpegts udp://127.0.0.1:8880
 * ffmpeg -re -i sintel.ts -f rtp_mpegts udp://127.0.0.1:8880
 */
 
typedef struct RTP_FIXED_HEADER{
	/* byte 0 */
	unsigned char csrc_len:4;       /* expect 0 */
	unsigned char extension:1;      /* expect 1 */
	unsigned char padding:1;        /* expect 0 */
	unsigned char version:2;        /* expect 2 */
	/* byte 1 */
	unsigned char payload:7;
	unsigned char marker:1;        /* expect 1 */
	/* bytes 2, 3 */
	unsigned short seq_no;            
	/* bytes 4-7 */
	unsigned  long timestamp;        
	/* bytes 8-11 */
	unsigned long ssrc;            /* stream number is used here. */
} RTP_FIXED_HEADER;
 
typedef struct MPEGTS_FIXED_HEADER {
	unsigned sync_byte: 8; 
	unsigned transport_error_indicator: 1; 
	unsigned payload_unit_start_indicator: 1;
	unsigned transport_priority: 1; 
	unsigned PID: 13;
	unsigned scrambling_control: 2;
	unsigned adaptation_field_exist: 2;
	unsigned continuity_counter: 4;
} MPEGTS_FIXED_HEADER;
 
 
 
int simplest_udp_parser(int port)
{
	WSADATA wsaData;
	WORD sockVersion = MAKEWORD(2,2);
	int cnt=0;
 
	//FILE *myout=fopen("output_log.txt","wb+");
	FILE *myout=stdout;
 
	FILE *fp1=fopen("output_dump.ts","wb+");
 
	if(WSAStartup(sockVersion, &wsaData) != 0){
		return 0;
	}
 
	SOCKET serSocket = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP); 
	if(serSocket == INVALID_SOCKET){
		printf("socket error !");
		return 0;
	}
 
	sockaddr_in serAddr;
	serAddr.sin_family = AF_INET;
	serAddr.sin_port = htons(port);
	serAddr.sin_addr.S_un.S_addr = INADDR_ANY;
	if(bind(serSocket, (sockaddr *)&serAddr, sizeof(serAddr)) == SOCKET_ERROR){
		printf("bind error !");
		closesocket(serSocket);
		return 0;
	}
 
	sockaddr_in remoteAddr;
	int nAddrLen = sizeof(remoteAddr); 
 
	//How to parse?
	int parse_rtp=1;
	int parse_mpegts=1;
 
	printf("Listening on port %d\n",port);
 
	char recvData[10000];  
	while (1){
 
		int pktsize = recvfrom(serSocket, recvData, 10000, 0, (sockaddr *)&remoteAddr, &nAddrLen);
		if (pktsize > 0){
			//printf("Addr:%s\r\n",inet_ntoa(remoteAddr.sin_addr));
			//printf("packet size:%d\r\n",pktsize);
			//Parse RTP
			//
			if(parse_rtp!=0){
				char payload_str[10]={0};
				RTP_FIXED_HEADER rtp_header;
				int rtp_header_size=sizeof(RTP_FIXED_HEADER);
				//RTP Header
				memcpy((void *)&rtp_header,recvData,rtp_header_size);
 
				//RFC3551
				char payload=rtp_header.payload;
				switch(payload){
				case 0:
				case 1:
				case 2:
				case 3:
				case 4:
				case 5:
				case 6:
				case 7:
				case 8:
				case 9:
				case 10:
				case 11:
				case 12:
				case 13:
				case 14:
				case 15:
				case 16:
				case 17:
				case 18: sprintf(payload_str,"Audio");break;
				case 31: sprintf(payload_str,"H.261");break;
				case 32: sprintf(payload_str,"MPV");break;
				case 33: sprintf(payload_str,"MP2T");break;
				case 34: sprintf(payload_str,"H.263");break;
				case 96: sprintf(payload_str,"H.264");break;
				default:sprintf(payload_str,"other");break;
				}
 
				unsigned int timestamp=ntohl(rtp_header.timestamp);
				unsigned int seq_no=ntohs(rtp_header.seq_no);
 
				fprintf(myout,"[RTP Pkt] %5d| %5s| %10u| %5d| %5d|\n",cnt,payload_str,timestamp,seq_no,pktsize);
 
				//RTP Data
				char *rtp_data=recvData+rtp_header_size;
				int rtp_data_size=pktsize-rtp_header_size;
				fwrite(rtp_data,rtp_data_size,1,fp1);
 
				//Parse MPEGTS
				if(parse_mpegts!=0&&payload==33){
					MPEGTS_FIXED_HEADER mpegts_header;
					for(int i=0;i<rtp_data_size;i=i+188){
						if(rtp_data[i]!=0x47)
							break;
						//MPEGTS Header
						//memcpy((void *)&mpegts_header,rtp_data+i,sizeof(MPEGTS_FIXED_HEADER));
						fprintf(myout,"   [MPEGTS Pkt]\n");
					}
				}
 
			}else{
				fprintf(myout,"[UDP Pkt] %5d| %5d|\n",cnt,pktsize);
				fwrite(recvData,pktsize,1,fp1);
			}
 
			cnt++;
		}
	}
	closesocket(serSocket); 
	WSACleanup();
	fclose(fp1);
 
	return 0;
}
```

