# SDK架构说明及接入方法
## 一、SDK目录及说明
SDK文档目录包含三个主目录ovdOpenApi、demo、sample
ovdOpenApi中为设备接入sdk的头文件及链接库
              接口头文件 OVD_OpenApi.h
              链接库libovd.a
              需要的json库 cjson

demoe为提供的模拟设备，提供一个设备注册、配网、推流、告警等流程。

sample为各个接口的使用示例。

   
## 二、接入集成方法
## 1 详细的接口说明，请见OVD_OpenApi.h

## 2 集成说明
    1、拷贝ovdOpenApi的include及链接库到工程目录下（若无json库，也需要拷贝jison库）
    2、接口实现处，引用OVD_OpenApi.h
    3、Makefile中, -I 引入头文件
    4、Makefile中,  -l:libovd.a  -lpthread -lstdc++ -lm -lrt  引入lib库
    

## 三、接口使用流程说明
## 1 SDK初始化
### 接口描述
   流程图
   先后顺序
   注意事项




## 四、已知问题说明

