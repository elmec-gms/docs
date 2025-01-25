# CAN通讯库(EcCanApi)使用说明
wanglinqiang  2020年4月21日 20:25:18

## 概要
EcCanApi.dll是ELMEC公司基于Vector公司的Vector XL Driver Library开发而成，主要应用在PC和ECM之间CAN通讯上，在使用本DLL时必须要有Vector公司开发的vxlapi.dll文件，与ECM之间的通讯设备必须是Vector公司出品的CAN设备。

本文档仅对DLL中的函数接口及使用上注意点加以说明，对ECM时使用代码格式请参考ECM生产厂家提供的资料，这里不做说明。

本文档仅对C/C++语言函数调用加以说明，对其它语言(如VB、C#等等)不做说明。

### 本DLL的特性

+ 最大支持4个CAN通道
+ 支持CAN和CAN-FD总线类型
+ 支持CAN ID的基本帧(11bit)和扩展帧(29bit)收发模式


### 本DLL的包含文件及说明

| Function name | Commend |
| :--- | :--- |
| EcCanApi.h | 函数调用的头文件 |
| EcCanApi.dll | 在32位APP调用时的DLL文件名 |
| EcCanApi.lib | 静态调用32位DLL时使用 |
| EcCanApi64.dll | 在64位APP调用时的DLL文件名 |
| EcCanApi64.lib | 静态调用64位DLL时使用 |


## 函数调用流程
请参考下图
![image-20200326115842980](C:\Users\wang\AppData\Roaming\Typora\typora-user-images\image-20200326115842980.png)


## 函数使用说明
本DLL包含函数及简介
| Function name | Commend |
| :--- | :--- |
| elcCanCreate() | 创建一个CAN通道接口 |
| elcCanDelete() | 删除一个CAN通道道口 |
| elcCanOpen() | 打开CAN通道并初始化 |
| elcCanClose() | 关闭CAN通道 |
| elcCanDoCmd() | ECM收发命令接口 |
| elcCanSetEcmType() | 设定ECM通讯协议类型（不建议使用） |
| elcCanDoMi() |  MI命令通讯接口（不建议使用） |
| elcCanDoPs() | PS命令通讯接口（不建议使用） |
| elcCanDoPsReset() | PsReset命令通讯接口（不建议使用） |
| elcCanDoTcc() | TCC命令通讯接口（不建议使用） |
| elcCanDoTcr() | TCR命令通讯接口（不建议使用） |
| elcCanDoSrlStart() | SRL开始命令通讯接口（不建议使用） |
| elcCanDoSrlState() | SRL状态命令通讯接口（不建议使用） |
| elcCanDoSrlStop() | SRL停止命令通讯接口（不建议使用） |


## elcCanCreate()
**函数原型**

```cpp
DWORD elcCanCreate(int ch, void* szLogDir)
```
**功能**
+ 创建一个CAN通道接口，在使用其它函数前必须先调用此函数。

**参数**
+ ch: [in]接口要使用的通道
+ szLogDir: [in]日志文件保存文件夹（如果是NULL，则日志文件夹和调用APP的文件夹相同）

**返回值**

+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏


## elcCanDelete()
**函数原型**
```cpp
DWORD elcCanDelete(int ch)
```
**功能**

+ 删除一个CAN通道道口，如果你调用了此函数，则无法调用下面的函数。

**参数**
+ ch: [in]接口要使用的通道

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏


## elcCanOpen()
**函数原型**
```cpp
DWORD elcCanOpen(int ch, void* p)
```
**功能**
+ 打开CAN通道并初始化，在ECM通讯前必须调用此函数，如果调用失败则无法进行ECM通讯。

**参数**
+ ch: [in]接口要使用的通道
+ p: [in]接口要使用的参数，详细参数结构体

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏

**参数结构体**
```cpp
#pragma pack (push, 1)

typedef struct
{
    // Device ID, please reference vxlapi.h define
    DWORD DeviceId;
    // 0=CAN, 1=CAN-FD
    DWORD CanBus;
    // 0=Standard format (11bit), 1=Extended format(29bit)
    DWORD CanIdMode;
    // PC side CAN ID address(send use)
    DWORD PcAddr;
    // ECM side CAN ID address(Receive use)
    DWORD EcmAddr;
    // Have watchdog
    DWORD HaveWatchdog;
    // Watchdog data
    BYTE WatchdogData[8];
    // watchdog inv time(ms)
    DWORD WatchDogInv;
    // Reserve memory
    DWORD Reserve[32];
}CAN_PARA;

#pragma pack (pop)
```


## elcCanClose()
**函数原型**
```cpp
DWORD elcCanClose(int ch)
```
**功能**
+ 关闭CAN通道，如果调用了此函数，则无法进行ECM通讯。

**参数**
+ ch: [in]接口要使用的通道

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏




## elcCanDoCmd()
**函数原型**
```cpp
DWORD elcCanDoCmd(int ch, BYTE* pSendData, BYTE* pRecvData)
```
**功能**
+ ECM收发命令接口，此函数是和ECM通讯的通用函数。

**参数**
+ ch: [in]接口要使用的通道
+ pSendData: [in]发送给ECM的代码，首字节必须是长度，最大支持256(0xFF)个字节。
+ pRecvData: [out]从ECM上接收到的代码，如果调用时设定为NULL，则不返回代码。
>本函数不对ECM接收到的数据进行长度检查，因此在定义pRecvData内存块时，必须按照ECM返回信息进行定义，以防止内存泄漏

<font color=#ff0000>注意：Ver1.12以后（包含1.12）的版本，当pSendData或pRecvData超过8byte时，前面2个byte为长度，使用时请检查版本号。</font>
```cpp
// 收发数据在8字节以下，没有变化
Ver1.11: 06 01 02 03 04 05 06 
Ver1.12: 06 01 02 03 04 05 06

// 收发数据在8字节以上，有变化
Ver1.11: 08 01 02 03 04 05 06 07 08 // #1byte表示长度
Ver1.12: 10 08 01 02 03 04 05 06 07 08 // #1和#2byte表示长度
Ver1.11: 15 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 10 11 12 13 14 15 // #1byte表示长度
Ver1.12: 10 15 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F 10 11 12 13 14 15 // #1和#2byte表示长度
```


**返回值**

+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏

**调用示例**
```cpp
// 返回参数定义
DWORD dwRet;	
// 定义一个要通讯的CAN通道
int nCanCh = 0; // 0表示CAN通道1

// 定义并设定参数
CAN_PARA Para;
memset(&Para, 0x00, sizeof(CAN_PARA));
// Device ID, please reference vxlapi.h define
Para.DeviceId = 55;				// 55=VN1610, 57=VN1630A
// 0=CAN, 1=CAN-FD
Para.CanBus = BUS_CAN;			// BUS_CAN的定义请参考EcCanApi.h文件
// 0=Standard format (11bit), 1=Extended format(29bit)
Para.CanIdMode = CAN_ID_STD;	// CAN_ID_STD的定义请参考EcCanApi.h文件
// PC side CAN ID address(send use)
Para.PcAddr = 0x7E0;
// ECM side CAN ID address(Receive use)
Para.EcmAddr = 0x7E8;
// Have watchdog
Para.HaveWatchdog = 0;			// 0=没有Watchdog, 其它值=有Watchdog

// 创建一个CAN通道接口
dwRet = elcCanCreate(nCanCh);
if(dwRet)
{
	// 创建一个CAN通道接口失败
	return 1;
}

// 打开一个CAN通道接口，注意：参数中的ch必须已经创建成功，如没有创建或创建不成功将返回失败
dwRet = elcCanOpen(nCanCh, (void*)&Para);
if(dwRet)
{
	// 打开一个CAN通道失败
	return 1;
}

// 做一此发送命令并返回
// 定义发送命令内存块
BYTE pSendData[8];
memset(pSendData, 0x00, 8);
pSendData[0] = 0x03; // 有效数据长度
pSendData[1] = 0x22; // 命令号(也称服务号)，0x22表示读取ECM某一项目数据
pSendData[2] = 0x11; // DID高位， DID是数据项目ID的意思
pSendData[4] = 0x01; // DID低位
// 定义接收数据内存块并初始化为0值
BYTE pRecvData[256];
memset(pRecvData, 0x00, 256);

dwRet = elcCanDoCmd(nCanCh, pSendData, pRecvData);
if(dwRet)
{
	// 做发送命令失败处理
	// to do ...
}
else
{
	// 做发送命令成功处理
	// to do ...
}
// 关闭CAN通道端口
elcCanOpen(nCanCh);
// 删除CAN通道
elcCanDelete(nCanCh);

return 0;
```

## elcCanSetEcmType()

------

**函数原型**
```cpp
DWORD elcCanSetEcmType(int ch, int EcmType)
```
**功能**
+ 设定ECM通讯类型，在使用下面函数前，必须先调用此函数

**参数**
+ ch: [in]接口要使用的通道
+ EcmType: [in]ECM类型

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏


## elcCanDoMi()
**函数原型**
```cpp
DWORD elcCanDoMi(int ch, DWORD dwDID, BYTE* pRecvData)
```
**功能**
+ MI命令通讯接口，读取ECM数据命令。

**参数**
+ ch: [in]接口要使用的通道
+ dwDID: [in]数据项目的ID
+ pRecvData: [out]从ECM上获取到的数据信息

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏



## elcCanDoPs()
**函数原型**
```cpp
DWORD elcCanDoPs(int ch, DWORD dwDID, int type, DWORD data, BYTE* pRecvData)
```
**功能**
+ PS命令通讯接口，写入数据控制ECM命令

**参数**
+ ch: [in]接口要使用的通道
+ dwDID: [in]数据项目的ID
+ type: [in]数据项目的字节数
+ data: [in]要写入ECM中的数据
+ pRecvData: [out]从ECM上获取到的数据信息

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏



## elcCanDoPsReset()
**函数原型**
```cpp
DWORD elcCanDoPsReset(int ch, DWORD dwDID, BYTE* pRecvData)
```
**功能**
+ PSReset命令通讯接口，控制ECM的某一参数返回到初始状态（相对于elcCanDoPs函数）

**参数**
+ ch: [in]接口要使用的通道
+ dwDID: [in]数据项目的ID
+ pRecvData: [out]从ECM上获取到的数据信息

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏



## elcCanDoTcc()
**函数原型**
```cpp
DWORD elcCanDoTcc(int ch, BYTE* pRecvData)
```
**功能**
+ TCC命令通讯接口，清空ECM诊断代码命令
+ pRecvData: [out]从ECM上获取到的数据信息

**参数**
+ ch: [in]接口要使用的通道

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏




## elcCanDoTcr()
**函数原型**
```cpp
DWORD elcCanDoTcr(int ch, BYTE* pRecvData)
```
**功能**
+ TCR命令通讯接口，读取ECM诊断代码命令

**参数**
+ ch: [in]接口要使用的通道
+ pRecvData: [out]从ECM上获取到的数据信息

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏




## elcCanDoSrlStart()
**函数原型**
```cpp
DWORD elcCanDoSrlStart(int ch, DWORD dwDID, BYTE* pRecvData)
```

**功能**
+ SRL开始命令通讯接口，开始ECM学习命令

**参数**
+ ch: [in]接口要使用的通道
+ dwDID: [in]数据项目的ID
+ pRecvData: [out]从ECM上获取到的数据信息

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏




## elcCanDoSrlState()
**函数原型**
```cpp
DWORD elcCanDoSrlState(int ch, DWORD dwDID, BYTE* pRecvData)
```
**功能**
+ SRL状态命令通讯接口，获取ECM学习状态

**参数**
+ ch: [in]接口要使用的通道
+ dwDID: [in]数据项目的ID
+ pRecvData: [out]从ECM上获取到的数据信息

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏




## elcCanDoSrlStop()
**函数原型**
```cpp
DWORD elcCanDoSrlStop(int ch, DWORD dwDID, BYTE* pRecvData)
```
**功能**
+ SRL停止命令通讯接口，停止ECM学习命令

**参数**
+ ch: [in]接口要使用的通道
+ dwDID: [in]数据项目的ID
+ pRecvData: [out]从ECM上获取到的数据信息

**返回值**
+ 0: 调用成功
+ 其它值：调用失败，详细错误代码一栏



## 错误代码列表

| Decimal | Hex | Commend |
| :---- | :---- | :---- |
| 0     | 0x0000 | 成功返回     |
| 255   | 0x00FF | 不知道的错误     |
| 258   | 0x0102 | 等待超时     |
| 259   | 0x0103 | 通道号不符     |
| 260   | 0x0104 | 没有创建通道接口     |
| 其它   | ---   | 请参考Vector公司的vxlapi.dll错误代码 |



**END**