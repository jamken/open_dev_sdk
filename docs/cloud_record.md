# 录像切片直存服务

本文档描述了开放视频平台对外（为视频编码设备）提供的录像切片直存服务，及相应API。

## 1 名词解释

- OVS：open video system，开放视频云，一种基于物联网云计算技术的视频云系统；
  与传统NVR，DVR类似，OVS可以接入和管理各种带视频功能设备，支持直播、录像、点播、P2P实时通信等功能，
  但是与传统NVR，DVR的不同在于用户可以在互联网上观看视频，
  并且因为应用了云计算技术，OVS拥有近于无限的接入能力以及海量的并发直播点播请求；OVS主要由OVC与OVD两部分组成。

- OVC：open video cloud，运行在云端的开放视频云平台；
  提供各种类型的协议、API接口以及管理接口，相应的开放视频设备（OVD）能够根据相应的接入协议/API接口接入到OVC中由OVC远程管理，并将实时视频流/录像推送到OVC

- OVD：open video device, 开放视频设备，安装在客户现场的，具有与OVC通信能力的视频设备，如摄像头，门铃，音箱等，OVD通过“开放协议”接入OVC，并接受OVC的管理。

- OVS RECORD：OVD与OVC之间采用的录像切片上传服务

## 2 服务描述


OVS RECORD服务的主要目标是对OVD直接提供录像分片（HTTP）上传功能，中间无需经过流媒体服务器中转录像，达到降低服务器成本，减少中间环节（瓶颈点），提高录像稳定性的目的。
目前分片格式仅支持mpegts，其它格式的封装暂时不做支持。


### 2.1 录像分片直存流程

在无错误发生情况下，OVD的分片直存步骤如下：

1. OVD通过信令协议获取到对应（录像）通道的服务URL

2. OVD将实时媒体流录制成一个分片

3. OVD向直存服务（通过接口URL）发送create请求，获得分片上传URL

4. OVD通过（http）PUT方法，将分片上传到该存储地址

5. OVD向直存服务（通过接口URL）发送save请求，确认分片上传成功

6. 跳转步骤2，继续下一个分片

### 2.2 异常处理

在某些情况下，由于网络异常等原因可能会导致分片上传失败，OVD应该尽力进行多次重试。若最终重试无效，应该向直存服务发送fail请求，表示分片上传失败。OVC将会清理云端残留信息。

### 2.3 捎带确认

在正常流程中，每个分片上传需要调用直存服务接口两次，开销比较大。考虑到录像分片一般是连续上传的情况，在直存服务的create请求中，支持同时对上一次分片进行确认，由此可减免save请求。



## 3 服务接口API

RECORD服务对外提供HTTP接口api，接入设备只需要支持http协议，即可调用该服务各项功能。每套OVD在OVC上对应一个录像直存通道，每个直存通道拥有一个独立的HTTP URL供外部调用

直存通道的URL类似如下格式： 

```
  http://<OVC domain:port>/live/cseg/<通道ID>
```

其中通道ID为系统为每个OVD设备分配的通道ID，用于区分不同的设备的录像。直存服务的所有请求都是通过POST方法发送，参数都是通过form格式（key1=value1&key2=value2...）传输，响应结果通过json格式返回。



### 3.1 创建分片

**HTTP请求方法：**

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded

**HTTP请求消息体（举例）：**

```
  op=create&content_type=video%2Fmp2t&size=%d&start=%.6f&duration=%.6f
```  

**HTTP请求参数：**

HTTP请求的消息体为form格式字符串，参数说明如下：

```
  op: 字符串枚举型，必填：创建分片固定为create
  content_type：字符串，必填：分片的封装格式，目前仅支持video/mp2t
  size:字符串,必填:该分片数据的大小
  start:字符串,必填:该分片录制的开始时间
  duration:字符串,必填:该分片的时间长度
```


**HTTP状态码：**

- 200：正常

**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 

**HTTP响应消息体（举例）：**

正常情况
```
	{
	 "code": 0,
	 "msg": "OK",
	 "data": 
	 {
		 "file_name":"20191031/record/00000000001/1542246739.ts"，
		 "uri":"https://oss.endpoint/ossbucket/20191031/record/00000000001/1542246739.ts"
	  }
	}
```
HTTP响应的消息体为JSON格式字符串，返回值描述如下：
```
file_name:字符串,下一个数据分片的文件名称
uri:字符串,下一个数据分片的上传uri地址
```
异常情况
```
	{
	 "code": 7,
	 "msg": "INVALID_PARAM"
	}
```

### 3.2 确认分片
**HTTP请求方法：**

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded

**HTTP请求消息体（举例）：**

```
  op=save&name=%s
```  

**HTTP请求参数：**

HTTP请求的消息体为form格式字符串，参数说明如下：

```
  op: 字符串枚举型，必填：确认分片固定为save
  name：字符串，必填：需要确认的分片文件名
```


**HTTP状态码：**

- 200：正常

**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 

**HTTP响应消息体（举例）：**

正常情况

```
	{
	   "code": 0,
	   "msg": "OK"
	}
```  

异常情况
```
	{
	   "code": 10401,
	   "msg": "VOD_FILE_UPLOAD_FAILED"
	}
```

### 3.3 取消分片
**HTTP请求方法：**

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded

**HTTP请求消息体（举例）：**

```
  op=fail&name=%s
```  

**HTTP请求参数：**

HTTP请求的消息体为form格式字符串，参数说明如下：

```
  op: 字符串枚举型，必填：取消分片固定为fail
  name：字符串，必填：需要取消的分片文件名
```


**HTTP状态码：**

- 200：正常

**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 

**HTTP响应消息体（举例）：**

正常情况

```
	{
	   "code": 0,
	   "msg": "OK"
	}
```  

异常情况
```
	{
	   "code": 6,
	   "msg": "OBJECT_NOT_EXIST"
	}
```


### 3.4 获取服务器当前日历时间
**HTTP请求方法：**

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded

**HTTP请求消息体（举例）：**

```
  op=getcurrenttime
```  

**HTTP请求参数：**

HTTP请求的消息体为form格式字符串，参数说明如下：

```
  op: 字符串枚举型，必填：获取服务器时间固定为getcurrenttime
```


**HTTP状态码：**

- 200：正常

**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 

**HTTP响应消息体（举例）：**

正常情况
```
	{
		"code": 0,
		"msg": "OK",
		"data": 
		{
		"currenttime":"111111111"
		}
	}
```  

异常情况
```
	{
		"code": 10004,
		"msg": "AUTH_TIME_TOO_SKEWED",
	}
```

**HTTP响应说明：**

HTTP响应的消息体为JSON格式字符串，返回值和msg描述如下：

```
code	msg							错误描述
0		OK							成功
6		OBJECT_NOT_EXIST			对象不存在
7		INVALID_PARAM				参数错误
500		SERVER_INNER_ERROR			内部服务错误
10001	AUTH_INVALID_ACCESSKEY		非法AccessKey
10002	AUTH_SIGN_ERROR				鉴权检查错误
10003	AUTH_REPEATED_REQUEST		重复的请求
10004	AUTH_TIME_TOO_SKEWED		请求时间与服务器时差过大
10005	AUTH_TIME_PARSE_EXCEPTION	鉴权时间格式转换异常
10201	LIVE_CHANNEL_NOT_EXIST		channelId不存在
10202	LIVE_PUSHURL_OUT_OF_TIME	推流地址过期
10203	LIVE_NO_RECORD				未开启转录
10204	LIVE_CHANNEL_ALREADY_EXIST	channelId已存在
10401	VOD_FILE_UPLOAD_FAILED		文件上传失败
10402	VOD_FILE_NOT_EXIST			视频文件不存在
10403	VOD_FILE_AUTH_FAILED		上传认证失败
10404	VOD_FILE_AUTH_EXPIRED		上传认证过期
```


