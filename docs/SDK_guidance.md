# SDK使用说明

**本文档描述了开放SDK的结构及集成方法，通过本说明，可以了解开放SDK的使用集成方法，可以使用模拟的demo和sample程序了解SDK的集成及使用方法、流程等。</br>
demo既可用作测试SDK功能的主要工具，也可以用作设备模拟器，辅助平台侧的对接调试。sample每个程序只调用SDK其中的一种或几种功能，主要作用是作为参考程序指导设备厂商调用SDK的相关API。**

**名词解释**

- OVS：open video system，开放视频云，一种基于物联网云计算技术的视频云系统；
  与传统NVR，DVR类似，OVS可以接入和管理各种带视频功能设备，支持直播、录像、点播、P2P实时通信等功能，
  但是与传统NVR，DVR的不同在于用户可以在互联网上观看视频，
  并且因为应用了云计算技术，OVS拥有近于无限的接入能力以及海量的并发直播点播请求；OVS主要由OVC与OVD两部分组成。

- OVC：open video cloud，运行在云端的开放视频云平台；
  提供API接口以及管理网页，用户可以随时通过互联网，以网页、APP、公众号、小程序等方式接入OVC，对其下属的摄像头进行管理，并进行直播点播。

- OVD：open video device, 开放视频设备，安装在客户现场的，具有与OVC通信能力的视频设备，如摄像头，门铃，音箱等，OVD通过“开放协议”接入OVC，并接受OVC的管理。

- Channel：OVD上的视频通道；
  一般一路摄像头为一条channel。如果OVD为摄像头，则只有一条通道，且通道号必须为0；
  如果IVT为NVR/DVR，则会有多条通道，每条通道代表一个NVR管理下的摄像头。



## 一、 SDK在系统中的位置

![SDK在系统中的位置](SDKsysarch.jpg)



### SDK目录架构


![SDK目录结构](sdkcontent.png)


	



## 二、接口使用流程说明

### 1 SDK接入的调用流程图（请关注流程图后的注意事项）


   ![SDK流程图](detailedflowchart.jpg)
### 2 接入流程注意事项
   
- 建议设备按照流程图的步骤1、2、3、4、5进行接口调用，其中步骤4各子流程可以并行执行。
- 设备上电初始化后，若还未配置网络，则调用SDK提供的 *声波配网* 或者 *二维码配网* 方法进行配网；设备有自由配网方式，可以忽略SDK的方法。
- 若设备需要配网，且在设备在配网成功后，根据获取到的网络配置信息，调用SDK的方法OVD_DeviceBindInfo上报BIND ID。
- 若前期已经配网成功，建议在网络连接成功后，再调用步骤3，否则SDK连接服务器失败。
- 建议**保持步骤3的顺序**，因为启动服务后，服务器端可能会立即获取音视频参数，但设备还未上报音视频参数，导致服务器端异常重试，影响体验。
- 音视频准备推送、音视频停止推送可以在SDK运行过程中，根据OVD情况随时调用。但是在步骤3和步骤5建议保持流程图中的推荐调用方式。

   
## 三、SDK接入集成方法

	1. 拷贝include头文件到工程目录下，并在代码中引入OVD_OpenApi.h头文件
	2. 拷贝相应文件到工程目录下，例如lib\x86_64里的.a文件。若设备无相应库，则需将库一起拷贝。
	3. 在Makefile中链接库中增加 "-l:libovdsdk.a -l:libcjson.a -l:libmbedtls.a -l:libmbedcrypto.a -l:libmbedx509.a -lpthread -lstdc++ -lm -lrt" 



## 四、demo使用流程

### 1 编译链接

	a. x86_64 linux平台
		1.我们提供的sdk内部有提供x86环境编译出的静态库，供厂商在x86 模拟运行使用。 我们默认编译服务器配置为：gcc版本4.8.5 c++库为libstdc++.so.6.0.19 , 厂商二次编译demo时，gcc和c++库不能低于此配置要求。
		2.厂商在主目录上执行make demo
		3.生成的执行程序在demo/build/x86_64/bin 目录下
	
	b. 标准linux平台
		1、集成商首先在SDK根目录的config文件夹下，在XXX.config文件中对TOOLCHAIN_BIN_PATH / CROSS_COMPILE / CROSS_SYSROOT / CUSTOM_CFLAGS进行修改，适配编译环境(XXX 为设备型号标识)
		2、在主目录上执行TARGET_BUILD=XXX make demo
		3、生成的可执行文件在demo/build/XXX/bin目录下
	c. liteOS 平台
		1.集成商首先在SDK根目录的config文件夹下，在XXX.config文件中对TOOLCHAIN_BIN_PATH / CROSS_COMPILE/ 
		LITEOSSDKDIR / CUSTOM_CFLAGS进行修改，适配编译环境（XXX 为设备型号标识）
		2、在主目录上执行TARGET_BUILD=XXX make demo
		3.生成的静态库在demo/build/XXX/bin目录下

### 2 运行参数配置

1、若需要修改模拟的推送音视频源、模拟的告警图片、声波识别/二维码识别源等，则在DeviceConf.ini中修改所需资源文件的路径及文件名信息。</br>

2、在DeviceConf.ini文件中，根据情况修改设备参数信息。</br>
	
	主要配置项有： 
	devId:                               设备id
	OVDPassword:               信令服务器校验密码
	Enableserviceschedule:　     是否开启服务调度
	Servicescheduleurl:   设备服务调度的服务器url
	Passdomain:           本地设置的信令服务器域名
	passPort:             本地设置的信令服务器端口
	Bindid:若设置为0，则模拟器上报配网流程识别的bindid，
	若非0, 则模拟器固定上报该值，跳过配网流程的识别数据结果

### 3 demo执行

	1、 在demo/build/XXX/bin目录下执行 "./demo ${PATH}/DeviceConf.ini"，然后按照提示进行输入，引导程序继续执行
	2、 首先会询问设备本身参数是否在配置文件DeviceConf.ini准备好，若需要修改参数，请先修改完毕
	3、 准备好设备参数后，输入y，设备进行启动，初始化SDK
	4、 看到如下引导命令，即SDK启动成功
	5、 根据如下命令引导demo执行，
		退出               CMD: quit 
		声波配网           CMD: wave_net
		二维码配网         CMD: pic_net
		音视频准备好       CMD: av_ready    设备sdk识别视频格式
		系统服务准备好     CMD: service_ok    开启连接服务
		触发告警           CMD: trigger_alarm    
		告警结束           CMD: alarm_end
		音视频参数修改      CMD: av_mod
		推送音视频内容      CMD: push_av      设备开启往sdk推流
		推送音视频结束      CMD: push_end      设备停止往sdk推流
		电量信息（0-100）   CMD: battery       电量上报
		设备进入休眠        CMD: sleep
		设备从休眠状态唤醒  CMD：wakeup
	- demo执行过程中会有其他日志输出，若需要查看引导命令，请输入任何一个字符，回车即可


【注】**若需要服务器下发命令，请连接相关服务器端配合执行**</br>


## 五、sample使用流程

### 1 编译链接
	a.标准linux平台
		1、集成商首先在SDK根目录的config文件夹下，在XXX.config文件中对TOOLCHAIN_BIN_PATH / CROSS_COMPILE / CROSS_SYSROOT / CUSTOM_CFLAGS进行修改，适配编译环境(XXX 为设备型号标识)
		2、在主目录上执行TARGET_BUILD=XXX make samples
		3、生成的可执行文件在samples/build/XXX/bin目录下
	b.liteOS平台
		1、集成商首先在SDK根目录的config文件夹下，在XXX.config文件中对TOOLCHAIN_BIN_PATH / CROSS_COMPILE / LITEOSSDKDIR / CUSTOM_CFLAGS进行修改，适配编译环境(XXX 为设备型号标识)
		2、在主目录上执行TARGET_BUILD=XXX make samples
		3、生成的可执行文件在demo/build/XXX/bin目录下

### 2 sample各个子模块执行

	- sample_init： 演示sdk初始化动作， 执行命令为./sample_init
	- sample_pic_net： 演示sdk二维码识别配网动作,执行命令为./sample_pic_net filepath[二维码图片路径]
	- sample_pic_net： 演示sdk声波扫描过程， 执行命令为./sample_wave_net filepath[声波文件路径]
	- sample_service_ok： 演示sdk连接云服务器的过程，执行命令为./sample_service_ok
			预期结果为： 云服务器有收到心跳报文
	- sample_configuration:  演示sdk 连接信令服务器，并与信令服务器进行交互（上传和下载设备配置，重启设备），执行命令为./sample_configuration
			预期结果为： sdk可跟云服务器进行正常信令交互
	- sample_alarm:  此sample演示sdk 告警流程, 执行命令为./sample_alarm
			预期结果为：通过信令服务器触发 正常走完告警流程
	- sample_av: 此sample演示sdk 视频推流流程，执行命令为./sample_av
			预期结果为 通过信令服务器触发，正常走完推流流程

## 六、厂商Q/A

##### 格式要求相关
> Q: 配网数据格式要求</br>
A：格式范例:{“ssid":"zhejiang@yan","pwd":"1234@s2#","id":"87845"}


> Q: 二维码图片格式要求</br>
A：具体图片格式要求可以参看demo实现，建议24位bmp图片，其中OVD_QRInit(OVD_DEMOE_RecognizerFinish,1); 1代表是否翻转

> Q: 报警图片格式要求</br>
A：bmp和jpg都可以

> Q: 音视频格式要求</br>
A：当前sdk视频只支持H264,aac,如果格式不对，sdk会输出错误日志</br>
 //H264格式 每个NAL前有一个起始码 0x00 00 01（或者0x00 00 00 01）</br>
 //AAC格式，syncword：帧同步标识一个帧的开始，固定为0xFFF

 > Q: 厂商sd卡录像文件格式要求</br>
A：建议厂商用分钟秒数__序号.mp4格式，实际设备sdk对厂商文件名没有要求，p2p的指令是下发时间段，然后厂商反馈文件信息

 > Q: 安卓日志保存地址</br>
A：手机本地日志在/com.cmri.hgcc.jty/videodemo/手机号/log/log.txt

##### 状态判断相关

 > Q: 配网识别结果查看方法</br>
A：通过注册函数OVD_DEMOE_RecognizerFinish回调

 > Q: 设备上线判断方法</br>
A：通过日志中看到json报文，说明设备跟信令服务器已正常通信；函数可通过

> Q: 直播界面低功耗设备是否休眠判断</br>
A：直播等情况，信令服务器会下发OVD_HIBERNATE_OVC_NOTIFY 指令来回调上层，但p2p相关功能不走信令服务器，所以p2p功能会单独字段回调上层

> Q: 强制I帧判断</br>
A：在开始推流时会触发强制I帧的回调