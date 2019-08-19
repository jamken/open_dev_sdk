#接口描述
##1、 SDK初始化
###接口定义
**`int32_t OVDInit(OVD_DEVType type, OVC_NETParm netParam, OVD_CallBackFunList *pCallBackFunList)`**

###参数说明：
    [in]type:              设备类型，详细可见枚举类型RT_DevType
    [in]netParam:          云服务器地址及端口号，见OVC_NETParm结构体
    [in]pCallBackFunList:  提供给服务器端调用的回调函数，以相应服务器端的请求，见OVD_CallBackFunList。注：若未提供相关的回调函数，则相关请求被丢弃，及设备端不提供对应的功能。
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    typedef emum{
        DevType_Gun  = 1,  //枪机
        DevType_Nvr  = 2,  
        DevType_Card  = 4,  //卡片机
        DevType_Shake = 5,  //摇头机
        DevType_NewCard = 6,
        DevType_Battery = 7, 
        DevType_4G   = 8,
        DevType_NewShake= 9, //新摇头机 (16k全双工)
        DevType_NewGun = 10, //为G3S2新加 
        DevType_FaceRcgn= 11, //人脸识别Q1 
        DevType_BatSingle = 12, //电池单品 
        DevType_FamilyBall = 13, //
        DevType_LockI9M    = 14,
        DevType_PtzCamera= 15 //球机
        DevType_Other,
    }OVD_DEVType

    typedef struct _OVC_NETParm{
        char passDomain[128];    //<云服务器域名>
        unsigned int passPort;   //<云服务器端口>
    }OVC_NETParm

    typedef struct _OVD_CallBackFunList{
        int32_t (*OVDQueryRecordPage)(int32_t channel, int32_t recordType, uint32_t StartStamp,uint32_t EndStamp, uint32_t Page,uint32_t PageNum,RTFileListPerPage_3 *FilePage)

        xxx
        xxx
    }OVD_CallBackFunList


##2、开启服务（wifi连接成功后调用）
###接口定义
**`void OVDSerivceStart()`**
###参数说明：
    无
###返回值：
    无
###*附加定义及说明*
    无

##3、关闭服务（wifi断线时调用）
###接口定义
**`void OVDSerivceStop()`**
###参数说明：
    无
###返回值：
    无
###*附加定义及说明*
    无

##4、告警接口
###4.1 告警开始
###接口定义
**`int32_t OVDAlarmInfoStart(OVD_UpLoadAlarmInfo alarmInfo)`**
###参数说明：
    [in]alarmInfo:    报警信息结构体，详细可见结构体描述OVD_UpLoadAlarmInfo
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    typedef struct _OVD_UpLoadAlarmInfo
    {
	    uint32_t		startTimeStamp;	//报警开始时间戳
	    OVD_ImageInfo	ImageInfo;	//背景图信息
	    OVD_IdentifyType	AlarmType;  //报警类型
	
        //当FaceOrCry为 RT_FACE 时,以下参数才起作用
	    //OVD_FaceInfo 	FaceInfo;
	    //OVD_IdentifyInfo IdentifyInfo;
    }OVD_UpLoadAlarmInfo;

    typedef struct _OVD_ImageInfo
    {
	    unsigned char *buf;    			//数据buf
	    unsigned int  size;    			//数据长度
	    char		  ImageUrl[1024];	//目前没用到,可填空
    }OVD_ImageInfo;

    typedef enum
    {
	    OVD_FACE	=	0x00,      //脸部识别
	    OVD_CRY		=	0x01,  //哭声侦测
  	    OVD_MOTIOM	= 	0X02,  //移动侦测
    	OVD_VOICE	=	0x03,  //声音侦测	
    	OVD_PEOPLE	=	0x04,	
    	OVD_OTHER,	
    }OVD_IdentifyType;

###4.1 告警结束
###接口定义
**`int32_t OVDAlarmInfoEnd(OVD_IdentifyType alarmType,uint32_t endTimeStamp)`**
###参数说明：
    [in]alarmType:    告警类型，详细可见枚举OVD_IdentifyType
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    无

##5、音视频内容接口
###5.1 音视频内容准备传送接口
###接口定义
**`int32_t OVDAudioOrVideoStart(uint8_t channel,OVD_ContentType contentType,EncodeMode eCodeMode,uint32_t SuggestBitRate,OVDVideoDataFormat* videoinfo)`**
###参数说明：
    [in]channel:         通道号
    [in]contentType:     准备传送的内容，详见枚举值OVD_ContentType
    [in]eCodeMode:       视频清晰度，详细可见枚举类型EncodeMode
    [in]SuggestBitRate:  建议码率(可忽视)
    [in]videoinfo:       视频信息,详见结构体OVDVideoDataFormat
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    typedef enum
    {
	    OVD_Audio	=	0x00,      //音频
	    OVD_Video		=	0x01,  //视频
    }OVD_ContentType;

    typedef enum
    {
	    RT_ERMODE  =   0x00, //初始化
	    RT_HDMODE  =   0x01, //   高清
	    RT_SDMODE  =   0x02, //   标清
	    RT_LUMODE  =   0x03, //   流畅
    }EncodeMode；

	typedef struct _OVD_VideoDataFormat
	{
		unsigned int codec;				//编码方式
		unsigned int bitrate;        	//比特率, bps
		unsigned short width;			//图像宽度
		unsigned short height;			//图像高度
		unsigned char framerate;		//帧率, fps
		unsigned char colorDepth;		//should be 24 bits
		unsigned char frameInterval;   //I帧间隔
		unsigned char reserve;
	}OVDVideoDataFormat;

###5.2 音视频内容推送接口
###接口定义
**`int32_t OVDAudioOrVideoPushData(uint8_t channel,char isIFrame,void* contentData, uint32_t videoDataLen,uint32_t timestamp)`**
###参数说明：
    [in]channel:         通道号
    [in]isIFrame:        是否是I帧  0：不是 1：是
    [in]contentData:     发送数据的首字节指针
    [in]videoDataLen:    本次发送数据的长度
    [in]timestamp:       该帧时间戳(ms)
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    无

###5.3 音视频内容传送结束接口
###接口定义
**`int32_t OVDAudioOrVideoEnd(uint8_t channel)`**
###参数说明：
    [in]channel:         通道号
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    无

##6、录像回放内容推送接口
*录像内容查询、打开录像文件、录像文件控制、录像删除等功能，由回调函数定义，详见（SDK初始化）的参数定义*
###接口定义
**`int32_t OVDSendRecordVedioContent(uint8_t channel,char isIFrame,void* contentData,uint32_t videoDataLen,uint32_t timestamp)`**
###参数说明：
    [in]channel:         通道号
    [in]isIFrame:        是否是I帧  0：不是 1：是
    [in]contentData:     发送数据的首字节指针
    [in]videoDataLen:    本次发送数据的长度
    [in]timestamp:       该帧时间戳(ms)
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    无

##7、声波配置网络接口
###7.1 声波初始化
###接口定义
**`void* OVDSoundWaveInit(int sampleRate,int bitWidth)`**
###参数说明：
    [in]sampleRate:      采样率
    [in]bitWidth:        位宽(8/16bit)
###返回值：
    成功：返回声波句柄
    失败：NULL
###*附加定义及说明*
    无

###7.2 开始声波识别
###接口定义
**`int32_t OVDSoundWaveStart(void *recognizer,RecognizStart start_cbfunc,RecognizEnd end_cbfunc)`**
###参数说明：
    [in]recognizer:      声波句柄
    [in]start_cbfunc:    识别开始回调函数
    [in]end_cbfunc:      识别结束回调函数（此函数返回wifi信息）
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    无

###7.3 采集到的声波数据，写入识别器
###接口定义
**`int32_t OVDSoundWaveWriteData(void *recognizer,const void *data,unsigned long len)`**
###参数说明：
    [in]recognizer:      声波句柄
    [in]data:            声波数据的首字节指针
    [in]len:             声波数据长度
###返回值：
    成功：成功写入的数据长度
    失败：小于0的值
###*附加定义及说明*
    无

###7.4 停止声波识别
###接口定义
**`int32_t OVDSoundWaveStop(void *recognizer)`**
###参数说明：
    [in]recognizer:      声波句柄
###返回值：
    成功：0
    失败：其他值
###*附加定义及说明*
    无
