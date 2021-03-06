# SDK对外开放API接口说明
## 一、设备端开放SDK集成接口说明
**本文档描述了开放SDK与本地视音频设备之间的交互接口， 通过本说明，本地视音频设备可以通过SDK提供的统一接入能力接入到开放云平台，提供相应的服务及设置。更多信息请看include/OVD_OpenAPI.h**


## 二、接口详细描述
## 1 SDK初始化
### 接口描述
设备上电后调用
### 接口定义
**`int OVD_Init(OVDClientParam *clientParam, OVDLogParam *logParam, OVD_CallBackFunList *callBackFunList, char *jsonParam)`**
### 参数说明：
    [in]capinfo            设备能力集
	[in]clientParam:       云服务器地址及端口号，见OVDClientParam结构体
	[in]logParam:          输出日志配置信息，见OVDLogParam结构体
    [in]callBackFunList:   提供给服务器端调用的回调函数，以相应服务器端的请求，见OVD_CallBackFunList。   注：若未提供相关的回调函数，则相关请求被丢弃，及设备端不提供对应的功能。
    [in]jsonParam:         json格式化后的字符串，提供动态的参数配置。目前提供 1) snapshotSize: int型数值，设备截图的默认大小；2) buffDuration: int型数值，设备缓存的默认时长，单位为秒。 json格式信息参考如下：
         {
             "snapshotSize":<可选，整数： 设备默认清晰度下的截图大小。若不设置，则SDK中默认设置为500k>
             "buffDuration":<可选，整数： 设备从上电到连接到平台时，需要SDK缓存的音视频内容的时长，单位为秒。若不设置，则SDK中默认设置为0s，建议低功耗设备设置此值，一般设置为一个数据分片的大小（约10s）>
         }
### 返回值：
    成功：0
    失败：-1

### 相关结构体：
	typedef struct
	{
		char OVDDeviceID[MAX_LEN_32];           //OVD设备ID，必填
		char OVDPassword[MAX_LEN_64];           //OVD接入密码，必填
		char OVDHardWareModel[MAX_LEN_32];      //OVD的硬件型号，必填
		char OVDSystemVersion[MAX_LEN_32];      //OVD的固件版本号，必填
		//int  period;                    //device记录的SDK周期上报的周期，单位：秒，必填
		OVDNetParam netParam;           //网络信息，必填
		int enable_servicschedule;      //是否启用服务调度
		char servicescheduleurl[MAX_LEN_1024];  //服务调度的URL 
		char cafilepath[MAX_LEN_256];   //客户端ca文件路径
		int tz;                        //设备时区，默认东8区
	}OVDClientParam;

	typedef struct
	{
		OVDLogLevel logLevel;           //日志输出级别，详细见枚举值LogLevel，可选
		OVDLogSTD   logSTD;             //日志输出位置，可选，详细见枚举值LogSTD，可选
		void (*pOVDLogOutCallBack)(const char* buff);  //设备提供的日志输出回调，SDK的输出日志可以保存到device的存储文件中，可选，若未空为不支持
	}OVDLogParam;

	//日志输出基本依次增高
	typedef enum{
	    OVD_LOGLEVEL_TRACE = 0,
	    OVD_LOGLEVEL_DEBUG = 1,
	    OVD_LOGLEVEL_INFO  = 2,
	    OVD_LOGLEVEL_WARN  = 3,
	    OVD_LOGLEVEL_ERROR = 4,
	    OVD_LOGLEVEL_FATAL = 5,
	}OVDLogLevel;

	//日志输出位置
	typedef enum{
	
	    OVD_LOGSTD_NO = 0,  //不输出
	    OVD_LOGSTD_OUT = 1,  //输出到标准输出
	    OVD_LOGSTD_ERR  = 2,  //输出到标准出错
	}OVDLogSTD;
	

## 2 开启服务
### 接口描述
设备服务准备好后调用，即设备连接网络成功、音视频接口准备完毕等工作之后再调用
### 接口定义
**`int OVD_SerivceStart()`**
### 参数说明：
    无
### 返回值：
    无



## 3 关闭服务
### 接口描述
设备服务停止时后调用
### 接口定义
**`void OVD_SerivceStop()`**
### 参数说明：
    无
### 返回值：
    无


## 4 告警接口
### 4.1 告警开始
### 接口描述
设备检测到告警条件，触发告警
### 接口定义
**`int OVD_AlarmInfoStart(OVDUpLoadAlarmInfo *alarmInfo)`**
### 参数说明：
    [in]alarmInfo:    报警信息结构体，详细可见结构体描述OVDUpLoadAlarmInfo
### 返回值：
    成功：0
    失败：-1

### 相关结构体：
	typedef struct
	{
		int             channel;            //通道号
		long long    startTimeStamp;	   //报警开始时间戳
		OVDAlarmType	AlarmType;     //报警类型
		char*           desc;               //告警描述
		OVDImageInfo	ImageInfo;	   //背景图信息
	}OVDUpLoadAlarmInfo;
    
### 4.2 告警结束
### 接口描述
设备告警结束
### 接口定义
**`int OVD_AlarmInfoEnd(int channel, OVDAlarmType alarmType, long long endTimeStamp)`**
### 参数说明：
    [in]channel;        通道号
    [in]alarmType:      告警类型，详细可见枚举OVDAlarmType
	[in]endTimeStamp:   报警结束时间戳
### 返回值：
    成功：0
    失败：-1
### 相关结构体：
	typedef enum
	{
	    OVD_OUTTER  =   2,      //外部告警
	    OVD_MOTIOM	= 	3,      //移动侦测
	    OVD_CROSS	= 	4,      //拌网侦测
	    OVD_CRY		=	5,      //哭声侦测
	    OVD_FACE	=	6,      //脸部识别
	    OVD_VOICE	=	7,      //声音侦测
	    OVD_LOW_BATTERY	=	8,      //低电告警
	    OVD_LOSS_LOCK	=	9,      //撬锁告警
	    OVD_BELL	=	10,      //按铃事件
	    OVD_OTHER,
	}OVDAlarmType;

## 5 音视频内容接口
### 5.1 音视频内容准备传送接口
### 接口描述
设备准备好音频、视频或者音视频，开始准备发送
### 接口定义
**`int OVD_AVPushStart(int channel,OVDVideoDataFormat *videoinfo,OVDAudioDataFormat *audeoinfo)`**
### 参数说明：
    [in]channel:         通道号
    [in]videoinfo:       视频信息,详见结构体OVDVideoDataFormat，若无视频则为空
    [in]audeoinfo:       视频信息,详见结构体OVDAudioDataFormat，若无音频则为空
### 返回值：
    成功：0
    失败：-1
### 相关结构体：
	typedef struct
	{
	    unsigned int codec;
	    int quality;
	    unsigned int bitrate;
	    unsigned short width;
	    unsigned short height;
	    unsigned char framerate;
	    unsigned char colorDepth;
	    unsigned char frameInterval;
	    OVD_bool rsk_encrypt;
	    unsigned char reserve;
	
	}OVDVideoDataFormat;
	
	typedef struct
	{
	    unsigned int codec;
	    unsigned int samplesRate;
	    unsigned int bitrate;
	    unsigned short waveFormat;
	    unsigned short channelNumber;
	    unsigned short blockAlign;
	    unsigned short bitsPerSample;
	    int sampleperframe;
	    unsigned short frameInterval;
	    unsigned short reserve;
	}OVDAudioDataFormat;


### 5.2 音视频参数修改
### 接口描述
已经启动音视频传送后，若设备的音视频参数修改，则调用此接口通知sdk变动的参数
### 接口定义
**`int OVD_AVParamModify(int channel,OVDVideoDataFormat *videoinfo,OVDAudioDataFormat *audeoinfo)`**
### 参数说明：
    [in]channel:         通道号
    [in]videoinfo:       视频信息,详见结构体OVDVideoDataFormat，若无视频则为空，若无修改也需要携带原来参数
    [in]audeoinfo:       视频信息,详见结构体OVDAudioDataFormat，若无音频则为空，若无修改也需要携带原来参数
### 返回值：
    成功：0
    失败：-1


### 5.3 音视频内容推送接口
### 接口描述
设备向SDK推动音视频内容
### 接口定义
**`int OVD_AVPushData(int channel,OVDContentType contentType,char isIFrame,unsigned char* contentData,int dataLen, long long timestamp)`**
### 参数说明：
    [in]channel:         通道号
    [in]contentType:     准备传送的内容，详见枚举值OVDContentType 音频、视频、音视频
    [in]isIFrame:        是否是I帧  0：不是 1：是
    [in]contentData:     发送数据的首字节指针
    [in]videoDataLen:    本次发送数据的长度
    [in]timestamp:       该帧时间戳(ms)
### 返回值：
    成功：0
    失败：-1
### 相关结构体：
	typedef enum
	{
	    OVD_Video   =	0,          //视频
	    OVD_Audio	=	1,          //音频
	}OVDContentType;


### 5.4 音视频内容传送结束接口
### 接口描述
音视频内容传送结束接口
### 接口定义
**`int OVD_AVPushEnd(int channel)`**
### 参数说明：
    [in]channel:         通道号
### 返回值：
    成功：0
    失败：-1



## 6 SD卡录像内容回放
### 6.1 SD卡录像内容回放推送接口
### 接口描述
APP打开相关录像文件后，设备推送相关内容
*录像内容查询、打开录像文件、录像文件控制、录像删除等功能，由回调函数定义，详见（SDK初始化）的参数定义*
### 接口定义
**`int OVD_SendRecordAVContent(int channel,OVDContentType contentType,char isIFrame,void* contentData,int dataLen,long long timestamp)`**
### 参数说明：
    [in]channel:         通道号
    [in]contentType:     准备传送的内容，详见枚举值OVDContentType 音频、视频、音视频
    [in]isIFrame:        是否是I帧  0：不是 1：是
    [in]contentData:     发送数据的首字节指针
    [in]dataLen:         本次发送数据的长度
    [in]timestamp:       该帧时间戳(ms)
### 返回值：
    成功：0
    失败：-1

### 6.2 SD卡录像内容回放推送完成
### 接口描述
通知客户端录像文件播放完毕
### 接口定义
**`int OVD_RecordAVContentSendOver(int channel)`**
### 参数说明：
    [in]channel:         通道号
### 返回值：
    成功：0
    失败：-1



## 7 声波配置网络接口
### 7.1 声波初始化
### 接口描述
设备检测到无网络配置信息，判断若使用声波配网，则调用此接口，传入声波参数
### 接口定义
**`void* OVD_SoundWaveInit(int sampleRate,int bitWidth)`**
### 参数说明：
    [in]sampleRate:      采样率
    [in]bitWidth:        位宽(8/16bit)
### 返回值：
    成功：返回声波句柄
    失败：NULL


### 7.2 开始声波识别
### 接口描述
设备开始声波配网后，获取到音频文件，发送到SDK识别；SDK识别完后，调用回调endFunc返回配网信息
### 接口定义
**`int OVD_SoundWaveStart(void *recognizer,RecognizerStart starFunc,RecognizerFinish endFunc)`**
### 参数说明：
    [in]recognizer:  声波句柄
    [in]starFunc:    识别开始回调函数；回调函数定义为 void (*RecognizStart)(void);
    [in]endFunc:     识别结束回调函数（此函数返回wifi信息）；回调函数定义为 void (*RecognizerFinish)(struct OVDWiFiInfo info);

	typedef void (*RecognizerStart)(void);
	typedef void (*RecognizerFinish)(int type, void *info, int infoLen);
### 返回值：
    成功：0
    失败：-1


### 7.3 采集到的声波数据，写入识别器
### 接口描述
设备开始声波配网后，把声波数据传入SDK的识别器
### 接口定义
**`int OVD_SoundWaveWriteData(void *recognizer,const void *data,unsigned long len)`**
### 参数说明：
    [in]recognizer:      声波句柄
    [in]data:            声波数据的首字节指针
    [in]len:             声波数据长度
### 返回值：
    成功：成功写入的数据长度
    失败：小于0的值


### 7.4 停止声波识别
### 接口描述
识别完后，调用此接口
### 接口定义
**`int OVD_SoundWaveStop(void *recognizer)`**
### 参数说明：
    [in]recognizer:      声波句柄
### 返回值：
    成功：0
    失败：-1



## 8 二维码配置网络接口
### 8.1 二维码配网开始
### 接口描述
设备检测到无网络配置信息，判断若使用二维码配网，则调用此接口
### 接口定义
**`void *OVD_QRInit(RecognizerFinish cbfun,int reverse)`**
### 参数说明：
    [in]cbfun:      成功的回调函数
    [in]reverse:    图片是否翻转
### 返回值：
    成功：返回二维码识别句柄
    失败：NULL

### 8.2 二维码配网识别
### 接口描述
传入图片进行识别
### 接口定义
**`int OVD_QRStart(void *handle, void *raw, int width, int height, unsigned int bitCount)`**
### 参数说明：
    [in]handle:      二维码识别句柄
    [in]raw:         二维码内容
    [in]width:       宽
    [in]height:      高
    [in]bitCount:    位宽
### 返回值：
    成功：0
    失败：-1

### 8.3 二维码配网结束
### 接口描述
设备停止二维码配网
### 接口定义
**`int OVD_QRDestroy(void *handle)`**
### 参数说明：
    [in]handle:      二维码识别句柄
### 返回值：
    成功：0
    失败：-1



## 9 上报绑定信息
### 接口描述
设备上报绑定信息。
触发条件：设备不使用SDK提供的声波配网、二维码配网，由设备自身实现配网的情况下，设备在完成配网后，**必须调用此接口上报设备绑定信息**
### 接口定义
**`int OVD_DeviceBindInfo(char* bindid)`**
### 参数说明：
    [in]bindid:      绑定的信息
### 返回值：
    成功：0
    失败：-1



## 10 日志文件上传
### 接口描述
通知OVC日志文件上传完成。
### 接口定义
**`int OVD_LogDone(char* trans_id, int result, char* start, char *end, char *url);`**
### 参数说明：
    [in]trans_id:      <必填，字符串；标记此日志上传的任务ID，与LogUploadAsync保持一致>
	[in]result:      0 代表success / 1代表fail
	[in]start:      <必填，字符串；日志记录实际开始时间，格式yyyy-MM-ddTHH:mm:ss，例子：2016-12-05T02:15:32>
	[in]end:      <必填，字符串；日志记录实际结束时间，格式yyyy-MM-ddTHH:mm:ss，例子：2016-12-06T02:15:32>
	[in]url:      <必填，字符串；日志上传的URL>
### 返回值：
    成功：0
    失败：-1



## 11 设置SDK与云服务器之间的心跳周期
### 接口描述
一般不用配置
### 接口定义
**`int OVD_SetKeepaliveIntervel(int interval)`**
### 参数说明：
    [in]interval:     心跳周期，单位为秒
### 返回值：
    成功：0
    失败：-1


## 12 OVD电池电量变化通知
### 接口描述
OVD可以通过该方法向OVC上报当前的电量，一般电量百分比变化时可以发送该通知事件。设备可以决定每隔多少百分比上报一次电量，比如每隔10%上报一次
### 接口定义
**`int OVD_BatteryChange(int battery);`**
### 参数说明：
    [in]battery:     设备当前的电量百分比，整数： 0-100
### 返回值：
    成功：0
    失败：-1



## 13 回调OVD_CallBackFunList定义及说明
    

> 信息获取

##### OVC获取OVD设备信息时调用

`int (*OVD_GetOVDDeviceInfo)(OVDDeviceInfo *deviceInfo);`

	功能介绍：根据结构体OVDDeviceInfo的定义，返回相应的设备参数信息，包括设备ID、软硬件版本信息、wifi信息、ip\mac等信息
	参数说明:[out]deviceInfo 需要设备返回的信息，详细见结构体OVDDeviceInfo 
	返回值： 成功/0；失败/-1

	typedef struct
	{
	    char OVDDeviceID[MAX_LEN_32];                 //设备ID号，必填
	    char OVDHardWareModel[MAX_LEN_32];         //设备型号，必填
	    char OVDSystemVersion[MAX_LEN_32];        //设备固件版本号，必填
	    char wifi_ssid[WIFI_SSID_LEN];             //设备当前连接的wifi的ssid, 该字段空串表示未连接wifi，可选
	    int  wifi_signal;               //设备当前wifi的信号强度, 0-100, 当wifi_ssid不为空时有效，可选
	    int  upBandwidth;               //设备探测到的上行最大带宽，单位bps，不存在则表示上行带宽未知，负值表示未知，可选
	    int  downBandwidth;             //设备探测到的下行最大带宽，单位bps，不存在则表示下行带宽未知，负值表示未知，可选
	    char ipAddr[MAX_LEN_128];               //IP地址    <局域网IP，支持IPV6>，空值表示未知，可选
	    char macAddr[MAX_LEN_32]; 	            //MAC地址，空值表示未知，可选
	    int battery;
	}OVDDeviceInfo;
	    
            
	    
##### OVC获取OVD的能力信息时调用 
`int (*OVD_GetOVDConfigureInfo)(char* output_ovdconfig);`
	
	功能介绍：根据“设备配置信息定义”说明，返回相应的设备参数信息，包括时区、音视频力、告警能力等信息。没有的参数可以不携带   
	参数说明:[out]output_ovdconfig 需要返回的信息，json格式信息，见具体反馈信息如上“设备配置信息定义”说明
	返回值： 成功/0；失败/-1    
	    
    
##### OVC获取OVD预置点列表信息
`int (*OVD_GetPresetList)(int channel,int *presetList, int *count);`

	功能介绍：OVD返回前期配置的预置点信息
	参数说明:
	    [in]channel:         通道号
	    [out]presetList:     返回的预置点列表
	    [out]count:          预置点个数    
	返回值：成功/0；失败/-1

##### OVC获取OVD周期上报信息
`int (*OVD_getsimpleovdinfo)(SimpleOVDinfo *out_simpleovdinfo);`


##### OVC获取OVD上SD卡的录像文件信息列表
`int (*OVD_QueryRecordPage)(unsigned int channelmask1,unsigned int channelmask2,int recordType, unsigned long StartStamp,unsigned long EndStamp,int Page,int numInPage,OVDRecordFileListPerPage *FilePage);`

	功能介绍：OVD根据入参的配置，查询SD卡中的文件列表，并返回文件列表信息

	参数说明:
	    [in]channelmask1:   需要查询的通道mask，对应低位号的通道
	    [in]channelmask2:   需要查询的通道mask，对应高位号的通道
	    [in]recordTypemask:     文件类型（第0位：视频文件 第1位：告警文件）
	    [in]startStamp:     录像查询的起始时间戳(s)
	    [in]endStamp:       录像查询的结束时间戳(s)
	    [in]page:           查询的页码
	    [in]numInPage:      每页的条数
	    [out]fileInPage:    录像文件列表信息，详细可见结构体描述OVDRecordFileListPerPage
	    
	返回值：成功/0；失败/-1

	其他说明:
		假设查询的录像文件数有200个，numInPage=10，则最大页码Page为20。若传进的参数numInPage=10，page=2，则fileInPage应该返回第10个到第20个录像的信息;若传进的参数numInPage=10，page=21，则fileInPage返回空录像信息(录像个数为0)

	typedef struct
	{
	    int               fileCount;            //文件数量
	    OVDRecordFileInfo fileinfo[MAX_LEN_128];        //文件列表
	}OVDRecordFileListPerPage;

	typedef struct
	{
	    int channel;                    //通道号
	    char FileName[MAX_LEN_256];
	    int  FileType; 			    	//文件类型(0 视频文件， 1 告警文件)
	    unsigned long long FileStartStamp;		    //录像开始时间
	    unsigned long long FileEndStamp;			//录像接收时间
	    int  RecordDuration; 			//时长
	    int  FileSize; 					//文件大小
	}OVDRecordFileInfo;
	
	
        

##### OVC根据查询到的SD卡文件列表信息，决定查看一个文件详细信息
`int (*OVD_OpenRecordFile)(int channel,char* recordname,OVDVideoDataFormat* videoInfo,OVDAudioDataFormat* audioInfo,int* fileTotalTime);`

	功能介绍：OVD根据入参的文件名称，读取SD卡上相应文件的音视频数据参数信息及录像时长，反馈到出参

	参数说明:
	    [in]channel:          通道号
	    [in]recordname:       录像文件名称
	    [out]videoInfo:       视频信息,详细可见结构体描述OVDVideoDataFormat
	    [out]audioInfo:       音频信息,详细可见结构体描述OVDAudioDataFormat
	    [out]fileTotalTime:   该录像的时长(ms)
	    
	 返回值：成功/0；失败/-1
	    

##### OVC获取SD卡信息时调用
`int (*OVD_GetSDInfo)(OVDSDInfo *sdInfo);`

	功能介绍：OVD查询本地SD卡相关信息（OVDSDInfo定义的内容），反馈信息

	参数说明:
	    [out]sdInfo:   存储卡信息,详细可见结构体描述OVDSDInfo

	返回值：成功/0；失败/-1

	typedef struct
	{
	    int  SDExist;		    //0 not, 1 yes, 2 error
	    int	 SDTotalSize;	    //总容量(M)
	    int	 SDFreeSize;	    //空闲量
	    char EarlyFileName[MAX_LEN_24]; //当前SD卡最早一个录像文件
	}OVDSDInfo;
	   

> 远端控制
	  
##### OVC下发需要修改的参数
`int (*OVC_SetOVDConfigureInfo)(char* in_ovdconfig);`

	功能介绍：OVD根据收到的参数信息，启用相应配置的参数。没有携带的信息，设备保持原参数配置
	参数说明:
		[in]in_ovdconfig:    需要设置的信息，json格式信息，见具体反馈信息如上“设备配置信息定义”说明
	返回值：成功/0；失败/-1    

##### OVC相应的地址发生变动时，通过此接口通知OVD
`int (*OVD_SetServerInfo)(OVDNetParam *netParam);`

	功能介绍：OVD根据配置信息，断开原来的连接，根据新的服务器地址建立新的连接；其中地址为空的为未返回数据，OVD继续使用原配置
	参数说明：
		[out]netParam:     OVC反馈的服务信息，详细见结构体OVDNetParam定义，其中地址为空的为未返回数据，此时设备继续使用原配置
	返回值：成功/0；失败/-1

	typedef struct
	{
	    char passDomain[MAX_LEN_64];        //<云服务器的域名>，必填
	    int  passPort;       //<云服务器的端口>，必填
	    int  Secure_PassPort;  //<云服务器的加密端口>,必填
	    char p2p_passDomain[MAX_LEN_64];    // <P2P pass 的域名>，必填
	    int  p2p_passPort;   //<杭研p2p pass的端口>，必填
	    char turnDomain[MAX_LEN_64];        //<P2P turn的域名>，没有置为空
	    int  turnPort;       //<p2p turn 的端口>，没有置为-1
	    char hibernationDomain[MAX_LEN_64];        //<休眠服务地址域名>，没有置为空
	    int  hibernationPort;       //<休眠服务地址端口>，没有置为-1
	    int  hibernationHBInterval;       //<休眠心跳骤起>，没有置为值10s，单位为秒
	    int  maxP2PSession;       //设备支持的最多的P2P流的个数，设备固有参数，服务器不能设置
	}OVDNetParam;   
	    

##### SDK与OVC之间的连接状态发生变化时通知OVD
`void (*OVD_OVCConnectStatus)(int connectStatus);`

	功能介绍：OVD根据状态的通知，决定相应处理
	参数说明:
		[in]connectStatus:    //0:连接成功     -1:连接失败
	    
	返回值：成功/0；失败/-1
	   
	    
##### 阻止设备休眠，或者唤醒设备当前所有休眠的部件
`int (*OVD_KeepAwakenUtilExpired)(int channel, int notAllowHibernate, int expired, OVDHibernateReason reason);`

	功能介绍：OVD收到此指令后，应该在指定的过期时间内，保证设备（及指定通道）能够完全正常上电工作。收到此命令expired秒后，由设备自行决定是否进入休眠状态
	
	参数说明：
        [in]channel:           需要保持不休眠的channel
        [in]notAllowHibernate:    是否允许进入休眠状态，0：允许进入休眠，进入休眠时间根据expired参数确定；1：不允许进入休眠，此种情况下，不管expired设置为多少，设备都要保持上电状态，不休眠，直至收到下一个休眠指令
        [in]expired:           从收到命令起，到expired的时间内，保持不休眠
        [in]reason:            设置不休眠的原因，详见枚举值定义OVDHibernateReason
        
	返回值：成功/0；失败/-1

	typedef enum
	{
	    OVD_HIBERNATE_OVC_NOTIFY = 0,   //OVC信令交互，需要保持一段时间不断电
	    OVD_HIBERNATE_P2P_NAT = 1,      //P2P开始NAT穿透打洞
	    OVD_HIBERNATE_P2P_OPEN = 2,     //正在P2P通话
	    OVD_HIBERNATE_OTHER = 3
	}OVDHibernateReason;
	    

##### 恢复出厂配置
`int (*OVD_ResetConfiguration)();`

	功能介绍：设备收到该请求后，应该将所有配置恢复到出厂状态（包括wifi配置），但不能断开当前网络连接，并返回成功应答。 OVC稍后会再下发一个重启指令将设备重启，默认配置生效
	返回值：成功/0；失败/-1
	    
        	  

##### OVC根据配置及OVD当前的版本，触发OVD进行固件升级时调用
`int (*OVD_FirmwareUpgrade)(char *firmware_model, char *upgradeURL);`

	功能介绍：OVD对比要升级的固件版本号和当前自身的固件版本号，若版本号不一致，那么到相应的URL去下载升级固件

	参数说明:
	    [in]firmware_model:  要升级的固件的版本号
	    [in]upgradeURL:      升级固件的远程url，由设备主动去下载、升级
	  
	返回值：成功/0；失败/-1
	   

##### OVC下发升级版本的命令后，会周期性的查询OVD的升级状态及进度
`int (*OVD_QueryFirmwareUpgradeStatus)(OVDUpgradeStatus *upgradeStatus, int *upgradeProgress);`

	功能介绍：OVD根据当天升级状态及进度，反馈相应的值
	   
	参数说明:
	    [out]upgradeStatus:    升级状态，详细见枚举值OVDUpgradeStatus
	    [out]upgradeProgress:  升级进度，整数值 0-100
	   
	返回值：成功/0；失败/-1

	typedef enum
	{
	    OVD_STATUS_IDLE	          =	  0,  //空闲胎
	    OVD_STATUS_DOWNLOADING	  =	  1,  //安装包下载中
	    OVD_STATUS_DOWNLOADED     =   2,  //安装包下载完成
	    OVD_STATUS_WAIT_INSTALL   =   3,  //等待安装
	    OVD_STATUS_INSTALLING     =   4,  //安装中
	    OVD_STATUS_INSTALLED      =   5,  //已安装完毕
	    OVD_STATUS_FAILED         =   6,  //安装失败
	    OVD_STATUS_BUSY           =   7,  //系统忙
	}OVDUpgradeStatus;
	   
        
> APP相关

##### 当用户通过APP远程控制channel重启时，调用此接口
`int (*OVD_ReBootChannel)(int channel);`

	功能介绍：OVD根据入参，重新启动相应的channel；若OVD不支持单channel重启，那么就重启设备
	    
	参数说明:
		[in]channel:    若重启channel，则为需要重启的channel。 注：如设备不支持单独重启channel，则直接重启设备。
	    **
	返回值：成功/0；失败/-1
	
	  
##### 当用户通过APP远程控制设备重启时，调用此接口
`int (*OVD_ReBootDevice)();	`
	
	功能介绍：OVD重启设备
	返回值：成功/0；失败/-1


##### APP端控制OVD进行转动时，调用此接口
`int (*OVD_PTZCmd)(int channel,OVCPTZControlCmd ptzcmd,int speed);`

	功能介绍：OVD根据控制命令，执行动作

	参数说明:
	    [in]channel:         通道号
	    [in]ptzcmd:          控制命令,详细可见枚举类型OVCPTZControlCmd
	    [in]speed:           表示转动速度，整数，0-100，0最慢，100最快，默认100。 注：若设备不支持，则可忽略此参数值
	    
	返回值：成功/0；失败/-1

	typedef enum
	{
	    OVC_PTZ_MV_UP        = 0,   //向上
	    OVC_PTZ_MV_DOWN      = 1,   //向下
	    OVC_PTZ_MV_LEFT      = 2,   //向左
	    OVC_PTZ_MV_RIGHT     = 3,   //向右
	    OVC_PTZ_MV_UPLEFT    = 4,   //左上
	    OVC_PTZ_MV_UPRIGHT   = 5,   //右上
	    OVC_PTZ_MV_DOWNLEFT  = 6,   //左下
	    OVC_PTZ_MV_DOWNRIGHT = 7,   //右下
	    OVC_PTZ_ZOOM_IN      = 8,   //拉近
	    OVC_PTZ_ZOOM_OUT     = 9,   //拉远
	    OVC_PTZ_MV_STOP      = 10,    //停止运动
	    OVC_PTZ_GOTO_PRESET  = 11,   //跳转预置位
	    OVC_PTZ_SET_PRESET   = 12,   //设置预置位点
	    OVC_PTZ_CLEAR_PRESET = 13,   //清除预置位点
	    OVC_PTZ_MV_UP_STEP   = 14,   //单步上
	    OVC_PTZ_MV_DOWN_STEP   = 15, //单步下
	    OVC_PTZ_MV_LEFT_STEP   = 16,  //单步左
	    OVC_PTZ_MV_RIGHT_STEP  = 17,  //单步右
	    OVC_PTZ_MV_UPLEFT_STEP  = 18,  //单步左上
	    OVC_PTZ_MV_UPRIGHT_STEP  = 19,  //单步右上
	    OVC_PTZ_MV_DOWNLEFT_STEP  = 20,  //单步左下
	    OVC_PTZ_MV_DOWNRIGHT_STEP  = 21,  //单步右下
	    OVC_PTZ_ZOOM_IN_STEP       = 22,  //单步拉近
	    OVC_PTZ_ZOOM_OUT_STEP      = 23,  //单步拉远
	}OVCPTZControlCmd;
	  
  

> 音视频相关
	    
##### OVC通知OVD播放音乐时调用
`int (*OVD_SetAudioOutPlay)(int channel,char *url);`

	功能介绍：OVD去url下载音乐内容，并在channel的音箱上播放

	参数说明:
	    [in]channel:      通道号
	    [in]url:          歌曲下载的url
	   
	返回值：成功/0；失败/-1
	  
	    
##### OVC控制OVD的音乐播放
`int (*OVD_AudioOutPlayCtrl)(int channel,OVDMp3PlayCtrl ctrl);`

	功能介绍：OVD根据控制信息，控制音乐的播放 停止、暂停、继续

	参数说明:
	    [in]channel:      通道号
	    [in]ctrl:         播放控制,详细可见枚举类型OVDMp3PlayCtrl
	    
	返回值：成功/0；失败/-1

	typedef enum
	{
	    MP3_CLOSE	=	0,  	 //停止播放
	    MP3_PAUSE	=	1,  	 //暂停播放
	    MP3_RESUME	= 	2,    //恢复播放
	    MP3_OTHER,
	}OVDMp3PlayCtrl;	    


##### OVC查询OVD上当前的音乐播放情况
`int (*OVD_GetAudioOutPlayStatus)(int channel, int* status, char* out_url);`

	功能介绍：OVD根据当前的播放情况，反馈播放音乐的URL及播放状态（播放 停止、暂停、继续）；若未播放任何音乐，则URL为空
	
	参数说明:
	    [in]channel:         通道号
	    [out]status:         播放状态,见OVDAudioPlayStatus枚举值定义
	    [out]url:            正在播放的歌曲的下载url，该域不存在或者空串表示当前未播放，url最长1024字节
	   
	返回值：成功/0；失败/-1

	typedef enum
	{
	    OVD_PLAY_STOPPED		=	0,  //停止播放状态
	    OVD_PLAY_PAUSING		=	1,  //暂停状态
	    OVD_PLAY_PLAYING		=	2,  //播放中
	}OVDAudioPlayStatus;

##### 强制OVD出一个I帧
`int (*OVD_ForceIFrame)(int channel);`

	功能介绍：SDK在收到上传视频、P2P播放等命令时，为了服务器/客户端能够最短时间内播放视频，在此channel上的数据流上强制出一个I帧，并继续上传数据流
	
	参数说明:
	    [in]channel:         通道号
	   
	返回值：成功/0；失败/-1

##### OVC通知OVD截取channel上的当前图片
`int (*OVD_Snapshot)(int channel,OVDImageInfo *imageInfo, int maxImageSize);`
	
	功能介绍：OVD截取channel上的当前图片，反馈截图内容及信息。若截图大小大于maxImageSize，则返回-2并在OVDImageInfo.size中带回所需要图片大小

	参数说明:
	    [in]channel:         通道号
	    [out]ImageInfo;	   截取的图片信息
	    
	返回值：
	    成功：0
	    截图失败：-1
	    空间不足：-2,  maxImageSize空间不足，并在OVDImageInfo.size中带回所需要图片大小

	typedef struct
	{
	    char *buf;    			//数据buf
	    int  size;    			//数据长度
	    //char		  ImageUrl[1024];	//目前没用到,可填空
	}OVDImageInfo;

#### OVC通知OVD切换视频清晰度
`int (*OVD_VedioSwitchQuality)(int channel, OVDEncodeQuality quality, OVDVideoDataFormat *vedioInfo); `

	功能介绍：OVD根据配置切换到相应的视频清晰度，并返回切换后的视频参数

	参数说明:
	    [in]channel:         通道号
	    [in]quality:         要设置的清晰度，详细参考枚举值OVDEncodeQuality
	    [out]vedioInfo:      清晰度调整后的视频参数，详细参考结构体OVDVideoDataFormat
	    返回值：成功/0；失败/-1

	typedef enum
	{
	    OVD_1DMODE  =   0, //低清
	    OVD_SDMODE  =   1, //标清
	    OVD_HDMODE  =   2, //高清
	    OVD_FHDMODE  =  3, //全高清
	}OVDEncodeQuality;
	
	            	      

> 时间相关
	
##### 获取ovd时间戳，精确度毫秒 
`int (*OVD_gettime)(long long *out_time);`

	    
##### 设置ovd时间
`int (*OVD_settime)(long long input_time, int tolerance_value);`

	参数说明:
		[in]input_time:      设置的时间戳，单位为毫秒
	    [in]tolerance_value: 设置的容忍值，单位为秒,即当前系统时间戳和预设置的时间相差小于容忍值，则不需要设置系统时间
	    
##### OVC需要与设备同步时间时调用
`int (*OVD_SyncTime)(time_t time, int offset); `

	功能介绍：OVD根据传入的时间及偏差，决定是否根据下发的参数修改时间

	参数说明:
	    [in]time:             要同步的时间
	    [in]offset:           为整数，单位秒。由于网络延迟，设置此为可接受的偏差，若设备原时间与上面给定的时间的偏差在offset秒之内，则设备无需同步时间
	
	返回值：成功/0；失败/-1
	  
#### OVC获取设备时间
`int (*OVD_QueryTime)(time_t *time); `

	功能介绍：OVD查询本地时间，反馈参数

	参数说明:
	    [out]time:             设备上的时间
	  
	返回值：成功/0；失败/-1
                       
	
> SD卡相关
	    
##### 获取sd卡状态
`int(*OVD_GetDiskInfo)(int *out_state);	`
	
	参数说明：
		out_state: <必填，整数；sd卡状态，0：正常，1：未插卡，2：卡异常>,
	 
	返回值：成功/0；失败/其他值
	   
	 
##### OVD格式化SD卡
`int (*OVD_SetSDCardFormat)();`  

	功能介绍：OVC需要格式化SD卡时调用
	返回值：成功/0；失败/-1   

##### OVC根据查询到的SD卡文件列表信息，控制相应音视频文件的播放、停止、暂停等信息
`int (*OVD_RecordCotrol)(int channel,OVDCONTROLTYPE controlType,int value);`	
     
	功能介绍：OVD根据控制信息，控制文件的内容是否通过OVD_SendRecordAVContent上传文件内容

	参数说明:
	    [in]channel:         通道号
	    [in]controlType:     播放控制，详细可见枚举类型OVDCONTROLTYPE
	    [in]value:           额外值，目前只有视频拖动时会用到，代表要跳至的视频时间戳(ms)
	
	返回值：成功/0；失败/-1

	typedef enum
	{
	    OVD_CONTINUE	=	0,  //继续播放
	    OVD_PAUSE	    =	1,  //暂停
	    OVD_STOP        =   2,  //停止
	    OVD_FAST        =   3,  //快进
	    OVD_SLOW        =   4,  //慢放
	    OVD_JUMP        =   5,  //拖动  ms
	}OVDCONTROLTYPE;

> 日志相关

`int(*OVD_LogUploadAsync)(char *trans_id, char *start, char* end,char* url);`
	       
	功能介绍：OVC可以通过该命令控制OVD上传相关本地日志，供调试分析使用。OVD收到该指令后，通过异步方式将指定时间范围日志上传到相应的URL。 若指定范围的日志已经滚动删除，则使用尽力原则上传剩余的日志。若指定范围没有任何日志记录，应上传一个长度为0的日志文件。 日志文件通过PUT方法上传。
	    
	参数说明：
	      "trans_id": <必填，字符串；标记此日志上传的任务ID>
	      "start": <必填，字符串；日志记录开始时间，格式yyyy-MM-ddTHH:mm:ss，例子：2016-12-05T02:15:32>,
	      "end": <必填，字符串；日志记录结束时间，格式yyyy-MM-ddTHH:mm:ss，例子：2016-12-06T02:15:32>,
	      "url": <必填，字符串；日志上传的URL，设备通过PUT方法上传相应的日志文件>
	
	返回值：成功/0；失败/其他值
	   
    	
> 对讲相关 

##### APP端发起对讲时，调用此接口打开OVD对讲
`int (*OVD_AudioPlayStart)(int channel, OVDAudioOutDataFormat andioFormat);`

	功能介绍：OVD打开OVD对讲功能
	
	参数说明:
	    [in]channel:         通道号
        [in]andioFormat:     对讲音频数据的格式信息，见结构体定义OVDAudioOutDataFormat
	   
	返回值：成功/0；失败/-1

	typedef struct
	{
	    unsigned int codec;         //OVDAUDIOPLY_TYPE
	    unsigned int samplesRate;
	    unsigned int bitWidth;
	    unsigned int volume;
	    unsigned int priority;      //the biger ,the higher
	}OVDAudioOutDataFormat;

	typedef enum{
	    OVD_PCM          = 0,
	}OVDAUDIOPLY_TYPE;
	   
        
##### APP关闭对讲时通知OVD
`int (*OVD_AudioPlayStop)(int channel);`

	功能介绍：OVD关闭对讲功能
	     
	参数说明:
	    [in]channel:         通道号
	   
	返回值：成功/0；失败/-1
	   
        
##### OVC把对讲的音频内容，传输给OVD时调用
`int (*OVD_AudioPlayProGress)(int channel,unsigned char* buf, int size);`

	功能介绍：OVD接收数据，并在音箱播放出来
	     
	参数说明:
	    [in]channel:   通道号
	    [in]buf:      音频数据指针
	    [in]size:     音频数据大小
	    
	返回值：成功/0；失败/-1



## 14 附加定义及说明
	/*
	 *设备配置信息定义
	 *json格式
	 *
	 {
	  "channls": [
	    {
	      "channel":   <必填，可读可写，整数：通道号>
	      "video_encoding":{
	        "encoder": <必填，可读可写，字符串：视频编码器名称，目前仅支持h264>
	        "quality": <必填，可读可写，字符串；可选值为：ld、sd、hd、fhd，分别代表低清，标清，高清，全高清>
	        "fps": <可选，只读，整形：每秒帧数>
	        "bitrate": <可选，只读，整形：码流比特率>
	        "width": <可选，只读，整形：图像宽度像素>
	        "height": <可选，只读，整形：图像高度像素>
	        "gop": <可选，只读，整形：码流gop,单位帧>
	        "rsk_encrypt": <可选，客端可写，布尔型：是否对视频码流进行rsk加密，默认为false>
	      }
	      "audio_encoding":{
	        "encoder": <必填，可读可写，字符串：音频编码器名称，目前仅支持aac>
	        "sample_rate": <可选，只读，整形：采样率，即每秒钟采用数目，合法值8000/16000/32000/44100/48000>
	        "bitrate": <可选，只读，整形：码流比特率>
	        "bits_per_sample": <可选，只读，整形：位宽，即每个sample的比特数>
	        "sample_per_frame": <可选，只读，整形：每一帧中包含的sample数，AAC算法标准固定为1024>
	        "channel": <可选，只读，整形：声道数>
	      }
	      "image":{
	        "horflip":  <必填，可读可写, 布尔型：水平翻转>
	        "verflip":  <必填，可读可写, 布尔型：垂直翻转>
	      }
	
	      "alarms":{
	        "io":{           //外部报警配置，若OVD不具备该能力，该字段不存在
	          "on":  <必填，可读可写,布尔型：使能开关>
	          "sensitivity":  <必填，可读可写,整型：探测灵敏度， 0 - 100>
	        }
	        "face":{          //人脸识别配置，若OVD不具备该能力，该字段不存在
	          "on":  <必填，可读可写,布尔型：使能开关>
	          "sensitivity":  <必填，可读可写,整型：探测灵敏度， 0 - 100>
	        }
	        "cry":{           //哭声侦测配置，若OVD不具备该能力，该字段不存在
	          "on":  <必填，可读可写,布尔型：使能开关>
	          "sensitivity":  <必填，可读可写,整型：探测灵敏度， 0 - 100>
	        }
	        "voice":{         //声音侦测配置，若OVD不具备该能力，该字段不存在
	          "on":  <必填，可读可写,布尔型：使能开关>
	          "sensitivity":  <必填，可读可写,整型：探测灵敏度， 0 - 100>
	        }
	        "motion":{        //移动侦测配置，若OVD不具备该能力，该字段不存在
	          "on":  <必填，可读可写,布尔型：使能开关>
	          "sensitivity":  <必填，可读可写,整型：探测灵敏度， 0 - 100>
	        }
	        "cross":{         //拌网配置，若OVD不具备该能力，该字段不存在
	          "on":  <必填，可读可写,布尔型：使能开关>
	          "sensitivity":  <必填，可读可写,整型：探测灵敏度， 0 - 100>
	        }
	      }
	      "audio_out_volume": <可选，可读可写，整数：扬声器输出音量，0-100，若该字段不存在表示设备不支持音量调节>
	      "trace":  <可选，可读可写,布尔型：移动跟踪, 若该字段不存在，则表示设备不支持移动追踪>
	    }
	    ...
	  ],
	  "tz": <必填，可读可写，整数：时区号，例如东八区为8>
	  "auto_reboot": {   //自动维护（重启）能力，若OVD不具备该能力，则该字段不存在
	    "on": <必填，可读可写,布尔型：使能开关>
	    "cycle": <必填，可读可写,整型：自动维护（重启）的最短周期，单位秒，例如7天自动重启，可以设置为604800>
	    "start": <必填，可读可写,整型：自动维护（重启）开始时间（当天的秒数），单位秒，例如凌晨2点，即7200，设备可在start到end时间内随机选择一个时间重启>
	    "end": <必填，可读可写,整型：自动维护（重启）结束时间（当天的秒数），单位秒，例如凌晨6点，即21600，设备可在start到end时间内随机选择一个时间重启>
	  }
	 }
	 *
	 */

	/*简单的ovd信息
	"state": <必填，整数；OVD的状态，1：在线，2：升级中，3：异常>
	  "channels": [
	    {
	      "channel": <必填，整数>,
	      "state": <必填，整数；该channel的状态，0：离线，1：在线>,
	*/


	typedef enum {
	    AV_STREAM_CODEC_H264,
	    AV_STREAM_CODEC_AAC,
	}OVDAVCodec;
	
	typedef char OVD_bool; 
	typedef struct
	{
	    OVD_bool have_alarms_io;   //<布尔型：是否支持外包报警>,
	    OVD_bool have_alarms_face; //<布尔型：是否支持人脸侦测>,
	    OVD_bool have_alarms_cry; //<布尔型：是否支持哭声侦测>,
	    OVD_bool have_alarms_voice; //<布尔型：是否支持声音侦测>,
	    OVD_bool have_alarms_motion; //<布尔型：是否支持移动侦测>,
	    OVD_bool have_alarms_body;  //<布尔型：是否支持人形侦测>,
	    OVD_bool have_alarms_cross; //<布尔型：是否支持拌网侦测>,
	}OVDAlarmsInfo;
	
	typedef struct
	{
	    OVD_bool have_ptz;//<布尔型：是否支持云台控制能力>
	    OVD_bool have_battery; //<布尔型：是否电池供电>,
	
	    OVD_bool have_audio_out; //<布尔型：是否支持音乐输出>
	    OVD_bool have_voice_out; //<布尔型：是否支持对讲输出>
	
	    OVD_bool have_trace; //<布尔型：是否支持移动跟踪>
	
	    //alarms 告警上报相关能力
	    OVDAlarmsInfo ovdCapInfo_alarms; 
	}OVDCapInfo;
	
	typedef enum
	{
	    OVD_WIFI = 0,//IWiFi
	    OVD_SSID_WIFI = 1,//ssid和WIFI
	    OVD_PHONE = 2,//bindid
	    OVD_STRING = 3,
	    OVD_XX_SSID_WIFI = 7
	}OVDWifiInfoType;
	
	typedef struct
	{
	    char ssid[33];
	    int ssidLen;
	    char pwd[80];
	    int pwdLen;
	}OVDSSIDWiFiInfo;
	
	typedef struct
	{
	    char ssid[WIFI_SSID_LEN];   //wifi的ssid
	    int  ssidLen;
	    char pwd[WIFI_PWD_LEN];    //wifi密码
	    int  pwdLen;
	    char phone[16+1];
	}OVDXXSSIDWiFiInfo;
		


