# 开放平台设备休眠协议

本文档描述了开放视频平台对外（为视频编码设备）提供的休眠服务，及相应通信协议。通过本协议，设备在休眠状态下仍然能够与平台保持通信

## 1 名词解释

- OVC：Open Video Cloud，运行在云端的开放视频云平台，其包含各种类型的子服务、通信协议及API接口，支持多形态开放视频设备（OVD）的接入管理，实时媒体流推送、录像切片上传及其它功能。

- OVD：open video device, 开放视频设备，安装在客户现场的，具有与OVC通信能力的视频设备，如摄像头，门铃，音箱等，OVD通过“开放协议”接入OVC，并接受OVC的管理。

- OVH：Open Video Hibernation, 开放视频休眠服务，是OVC提供的其中一项子服务，为OVD提供休眠控制功能。

- OVH休眠协议：指开放视频休眠服务（OVH）对外提供的休眠控制协议，OVD设备可按照该协议与OVH通信，实现休眠控制功能

## 2 协议特性


OVH休眠协议的设计目标是尽可能简单，能够兼容大部分设备休眠模块的处理逻辑。协议直接基于**TCP/IP**之上，在TCP长连接上收发固定格式的报文即可实现。


### 2.1 接入流程


1. OVD在没活动的情况下自动关闭主芯片电路，并启动网络接口（卡/板）的休眠功能。

2. 网络接口（卡/板）与OVH的休眠服务IP地址/端口建立TCP长连接。

3. 网络接口（卡/板）每60秒向OVH休眠服务发送一个心跳报文。

4. 若收到OVH发出的唤醒报文，则停止网络接口（卡/板）的休眠功能，启动主芯片电路，正常接入OVC其余服务。

### 2.2 异常处理

在某些原因下，网络原因可能会被中间设备中断。OVD应该在检测到TCP长连接断开后，能够在一定的随机时间内重新与OVH建立连接，继续发送心跳报文。
TCP连接中断，OVH并不认为是OVD离线，只有在一定周期（135S）内没有收到相关心跳报文，才会对OVD做离线处理。


## 3 应用层协议数据包格式

此处数据包指的是直接基于TCP之上传输的数据包，数据包的格式为二进制格式，字节顺序为 **大端**

### 3.1 心跳上报

OVD应该按照一定的时间周期（60秒），在TCP长连接上，向OVH发送下述结构的数据包。

方向： 

```
  OVD -> OVC
```

包格式:  

```
  typedef struct
  {
      char magic[2];         // 同步字段，固定为0xcc, 0x00
      uint16_t length;       // 长度，固定为40，大端
      char macid[16];        // 设备id，16位十进制数字字符串，例如"2009550000011079"
      uint8_t cmd;           // 命令字, 固定为0x4A。
      uint8_t reserved1;     // 保留字节
      uint8_t reserved2;     // 保留字节      
      uint8_t reserved3;     // 保留字节    
      char token[16];        // 字符串令牌，最长16位字符，不足16位使用’\0'字符从后填充      
  }CmdPacket;
```


### 3.2 唤醒

当OVH需要唤醒OVD时，在TCP连接上向OVD发送下述结构的数据包。

方向:    
```
  OVC -> OVD
```  
  
包格式:   
```
  typedef struct{
      char cmd[8];         // 字符串，固定为"TOWAKEUP"
  }CmdAwake；
```
