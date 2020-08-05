# 录像切片直存服务

本文档描述了开放视频平台对外（为视频编码设备）提供的录像切片直存服务，及相应API。

## 1 名词解释

- OVC：open video cloud，运行在云端的开放视频云平台，其包含各种类型的子服务、通信协议及API接口，支持多形态开放视频设备（OVD）的接入管理，实时媒体流推送、录像切片上传及其它功能。

- OVD：open video device, 开放视频设备，安装在客户现场的，具有与OVC通信能力的视频设备，如摄像头，门铃，音箱等，OVD通过“开放协议”接入OVC，并接受OVC的管理。

- OVR：open video recording, 开放视频录像服务，是OVC包含的一项录像子服务，向OVD提供录像切片上存功能。

- 录像直存通道： 切片直存服务是以通道为单位进行管理的，任一录像分片只属于某一个特定的通道。OVS可以根据通道查询相应的分片列表

## 2 服务描述


OVR主要目标是向OVD提供录像切片（通过HTTP）直接上传功能，中间无需经由流媒体服务器进行视频转录，达到降低服务器成本，减少中间环节（瓶颈点），提高录像稳定性。
目前分片格式仅支持mpegts，其它格式的封装暂时不做支持。


### 2.1 录像分片直存流程

在正常情况下，OVD的分片上传的步骤如下：

1. OVD通过信令协议获取到对应（录像）通道的服务URL

2. OVD将录制一个视频分片

3. OVD向通道对应的URL发送create请求，获得分片上传URL

4. OVD通过（http）PUT方法，将分片上传到该存储地址

5. OVD向通道对应的URL发送save请求，确认分片上传成功

6. 跳转步骤2，继续下一个分片

### 2.2 异常处理

在某些情况下，由于网络异常等原因可能会导致分片上传失败，OVD应该尽力进行多次重试。若最终重试无效，应该向直存服务发送fail请求，表示分片上传失败。OVC将清理残留信息。

### 2.3 捎带确认

在正常流程中，由于采用了两阶段提交策略，每个分片上传需要调用直存服务接口两次（一次create,一次save），开销比较大。考虑到录像分片一般是连续上传的情况，在直存服务的create请求中，
支持同时对上一次分片进行确认，由此可减免save请求。



## 3 服务接口API

OVR服务对外提供HTTP类型api，设备只需要支持http协议，即可调用直存服务进行切片上传。每个设备在OVC上对应一个录像直存通道，每个直存通道拥有一个独立的HTTP URL。

直存通道的URL类似如下格式： 

```
  http://<OVR domain:port>/live/cseg/<通道ID>
```

其中通道ID为系统为每个OVD设备分配的通道ID，用于区分不同的设备的录像。直存服务的所有请求都是通过POST方法发送，参数都是基于x-www-form-urlencoded格式（key1=value1&key2=value2...）通过请求消息体发送，
执行结果通过响应消息体json格式返回。




### 3.1 创建分片

**HTTP请求方法：**

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded

**HTTP请求参数：**

HTTP请求参数使用x-www-form-urlencoded格式编码，包含在POST请求body中，相关的参数描述如下：

```
  op: 字符串枚举型，必填：创建分片固定为create
  content_type：字符串，必填：分片的封装格式，目前仅支持video/mp2t
  size:整数,必填:该分片数据的大小，字节为单位
  start:浮点数,必填:该分片录制的开始时间，从epoch到分片的开始录制时间，单位秒
  duration:浮点数,必填:该分片的时间长度，单位秒
  discontinue:整数，必填：连续分片标记，0表示该分片与上一分片时间戳连续；1表示该分片与上一分片非时间戳连续。
```

最终在HTTP请求body的字符串示例如下：

```
  op=create&content_type=video%2Fmp2t&size=32768&start=1574322174&duration=600&discontinue=0
``` 


**HTTP状态码：**

- 200：正常
- 400~499：用户输入错误
- 500~599：服务器异常


**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 

**HTTP响应：**

HTTP响应的消息体为JSON格式字符串，返回值描述如下：

正常情况
```
	{
	  "name": <字符串，必填：新分片的文件名称，如果为空串表示该分片不上传>
	  "uri": <字符串，必填：新分片的上传URL，如果为空串表示该分片不上传>
	}
```
HTTP响应的消息体为JSON格式字符串，返回值描述如下：


异常情况
```
	{
	  "info": <字符串，必填：错误描述>
	}
```

### 3.2 确认分片

**HTTP请求方法：**

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded


**HTTP请求参数：**

HTTP请求参数使用x-www-form-urlencoded格式编码，包含在POST请求body中，相关的参数描述如下：

```
  op: 字符串枚举型，必填：创建分片固定为save
  name：字符串，必填：需要确认的分片的文件名，应该和create请求中返回的文件名相对应
```

最终在HTTP请求body的字符串示例如下：

```
  op=save&name=0003.ts
```  

**HTTP状态码：**

- 200：正常
- 400~499：用户输入错误
- 500~599：服务器异常

**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 

**HTTP响应：**

HTTP响应的消息体为JSON格式字符串，返回值描述如下：

正常情况body:

```
无
```

HTTP响应的消息体为JSON格式字符串，返回值描述如下：


异常情况body:
```
	{
	 "info": <字符串，必填：错误描述>
	}
```


### 3.3 取消分片
**HTTP请求方法：**

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded


**HTTP请求参数：**

HTTP请求参数使用x-www-form-urlencoded格式编码，包含在POST请求body中，相关的参数描述如下：

```
  op: 字符串枚举型，必填：取消分片固定为fail
  name：字符串，必填：需要取消的分片的文件名，应该和create请求中返回的文件名相对应
```

最终在HTTP请求body的字符串示例如下：

```
  op=fail&name=0003.ts
```  

**HTTP状态码：**

- 200：正常
- 400~499：用户输入错误
- 500~599：服务器异常


**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 


**HTTP响应：**

HTTP响应的消息体为JSON格式字符串，返回值描述如下：

正常情况body:

```
无
```

HTTP响应的消息体为JSON格式字符串，返回值描述如下：


异常情况body:
```
	{
	 "info": <字符串，必填：错误原因>
	}
```


### 3.4 获取服务器当前日历时间


**HTTP请求方法：**

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded


**HTTP请求参数：**

HTTP请求参数使用x-www-form-urlencoded格式编码，包含在POST请求body中，相关的参数描述如下：

```
  op: 字符串枚举型，必填：操作类型，获取时间固定为getcurrenttime
```

最终在HTTP请求body的字符串示例如下：

```
  op=getcurrenttime
```

**HTTP状态码：**

- 200：正常
- 400~499：用户输入错误
- 500~599：服务器异常


**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 

**HTTP响应：**

HTTP响应的消息体为JSON格式字符串，返回值描述如下：

正常情况
```
	{
	  "currenttime": <浮点数，必填：从epoch到当前的秒数，单位为秒>
	}
```

HTTP响应的消息体为JSON格式字符串，返回值描述如下：

异常情况
```
	{
	  "info": <字符串，必填：错误原因>
	}
```

### 3.5 上报统计数据



**HTTP请求方法：**

OVD通过此接口，向OVR上报录像切片相关统计信息

```
  POST
```  

**HTTP请求头：**

- Content-Type：application/x-www-form-urlencoded


**HTTP请求参数：**

HTTP请求参数使用x-www-form-urlencoded格式编码，包含在POST请求body中，相关的参数描述如下：

```
  op: 字符串枚举型，必填：取消分片固定为stat
  dropped_nb：整数，选填：上一次上报完成后到当前时间内的分片丢弃数目（由于缓存队列满）。默认为0
  dropped_time: 浮点数，选填：上一次上报完成后到当前时间内的录像丢失时长（由于缓存队列满）。默认为0
```

最终在HTTP请求body的字符串示例如下：

```
  op=stat&dropped_nb=3&dropped_time=30
```  

**HTTP状态码：**

- 200：正常
- 400~499：用户输入错误
- 500~599：服务器异常


**HTTP响应头：**

- Content-Type：application/json; charset=utf-8
- Access-Control-Allow-Origin：\*
- Access-Control-Allow-Methods：POST 


**HTTP响应：**

HTTP响应的消息体为JSON格式字符串，返回值描述如下：

正常情况body:

```
无
```

HTTP响应的消息体为JSON格式字符串，返回值描述如下：


异常情况body:
```
	{
	 "info": <字符串，必填：错误原因>
	}
```

