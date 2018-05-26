A类规范中描述的所有命令都应在B类设备中实现。B类规范添加了以下MAC命令。

| CID | Command | 发送方 | 简要说明 |
| :---: | :---: | :---: | :---: |
| 0x10 | _**PingSlotInfoReq**_ | 终端设备 | 由终端设备用来将ping单播时隙的周期性传达给网络服务器 |
| 0x10 | _**PingSlotInfoAns**_ | 网关 | 网络使用它来确认PingInfoSlotReq命令 |
| 0x11 | _**PingSlotChannelReq**_ | 网关 | 网络服务器用于设置终端设备的单播ping信道 |
| 0x11 | _**PingSlotChannelAns**_ | 终端设备 | 由终端设备用来确认_**PingSlotChannelReq**_命令 |
| 0x12 | _**BeaconTimingReq**_ | 终端设备 | 过期 |
| 0x12 | _**BeaconTimingAns**_ | 网关 | 过期 |
| 0x13 | _**BeaconFreqReq**_ | 网关 | 网络服务器用于修改终端设备期望接收信标广播的频率的命令 |
| 0x13 | _**BeaconFreqAns**_ | 终端设备 | 由终端设备用来确认BeaconFreqReq命令 |

# 14.1 PingSlotInfoReq

通过_**PingSlotInfoReq**_命令，终端设备将通知服务器其单播ping时隙的周期性。此命令只能用于通知服务器单播ping时隙的周期性。组播时隙完全由应用定义，不应使用此命令。

| Size（bytes） | 1 |
| :---: | :---: |
| PingSlotInfoReq Payload | PingSlotParam |

| Bit\# | 7:3 | \[2:0\] |
| :---: | :---: | :---: |
| PingSlotParam | RFU | Periodicity |

**Periodicity**子字段是一个无符号的3位整数，用于编码终端设备当前使用的ping时隙期，使用以下公式。

_pingNb_ = 2^\(_7-Periodicity_\) _and pingPeriod_ = 2^\(_5+Periodicity_\)

实际的ping时隙周期将以秒为单位等于0.96×2^_Periodicity_

* **Periodicity **= 0意味着终端设备在beacon\_window间隔期间大约每秒钟都会打开一个ping时隙
* **Periodicity **= 7，每128秒是LoRaWAN Class B规范支持的最大ping周期。

为了改变ping时隙的周期性，设备应首先恢复到A类，通过_**PingSlotInfoReq**_命令发送新的周期性，并通过_**PingSlotInfoAns**_从服务器获得确认。然后可以用新的周期性切换回到B类。

该命令可以与**FHDRFOpt**字段中的任何其他MAC命令级联，如A类规范帧格式中所述。

# 14.2 BeaconFreqReq

该命令由服务器发送到终端设备，以修改终端设备期望信标的频率。

| Octets | 3 |
| :---: | :---: |
| BeaconFreqReq payload | Frequency |

频率编码与A类中定义的_**NewChannelReq**_ MAC命令相同。

**Frequency**是24位无符号整数。以Hz为单位的实际信标频道频率为100 x frequ。这允许通过100Hz步长在100MHz到1.67GHz之间的任何地方定义信标信道。终端设备必须检查频率是否被无线电硬件实际允许，否则返回错误。

即使默认行为指定了跳频信标（即US ISM频段），有效的非零频率也会强制设备在固定频率信道上收听信标。

值为0将指示终端设备使用“信标物理层”部分中定义的默认信标频率规划。在适当的情况下，设备恢复跳频信标搜索。

在接收到该命令后，终端设备用_**BeaconFreqAns**_消息来回答。此消息的MAC有效负载包含以下信息：

| Size（bytes） | 1 |
| :---: | :---: |
| BeaconFreqAns payload | Status |

**状态**位具有以下含义：

| Bits | 7:1 | 0 |
| :---: | :---: | :---: |
| Status | RFU | Beacon frequency ok |

|  | Bit = 0 | Bit = 1 |
| :--- | :--- | :--- |
| Beacon frequency ok | 设备不能使用这个频率，保持之前的信标频率 | 信标频率已更改 |

# 14.3 PingSlotChannelReq

该命令由服务器发送到终端设备，以修改终端设备期望下行链路ping的频率和/或数据速率。

该命令**只能在A类接收窗口**（在上行链路之后）**发送**。该命令不得在B类ping时隙中发送。如果设备在B类ping时隙内收到它，MAC命令将不被处理。

| Octets | 3 | 1 |
| :---: | :---: | :---: |
| PingSlotChannelReq Payload | Frequency | DR |

频率编码与A类中定义的_**NewChannelReq**_ MAC命令相同。

**Frequency**是24位无符号整数。以Hz为单位的实际ping信道频率为100 x frequ。这允许通过100Hz步长在100MHz至1.67GHz之间的任何位置定义ping信道。终端设备必须检查频率是否被无线电硬件实际允许，否则返回错误。

值为0指示终端设备使用默认频率规划。

DR字节包含以下字段：

| Bits | 7:4 | 3:0 |
| :---: | :---: | :---: |
| DR | RFU | data rate |

“数据速率”子字段是用于ping时隙下行链路的数据速率的索引。索引和物理数据速率之间的关系在\[PHY\]中为每个区域定义。

在接收到该命令后，终端设备使用_**PingSlotFreqAns**_消息进行应答。此消息的MAC有效负载包含以下信息：

| Size（bytes） | 1 |
| :---: | :---: |
| PingSlotFreqAns Payload | Status |

**状态**位具有以下含义：

| Bits | 7:2 | 1 | 0 |
| :---: | :---: | :---: | :---: |
| Status | RFU | Data rate ok | Channel frequency ok |

|  | Bit = 0 | Bit = 1 |
| :--- | :--- | :--- |
| Data rate ok | 没有为这个终端设备定义指定的数据速率，保持以前的数据速率 | 数据速率与终端设备的可能兼容 |
| Channel frequency ok | 设备无法接收此频率 | 该频率可以由终端设备使用 |

如果这两个比特中的任何一个等于0，则该命令不成功，并且ping时隙参数未被修改。

# 14.4 BeaconTimingReq和BeaconTimingAns

这些MAC命令在LoRaWAN1.1版本中已弃用。该设备可以使用DeviceTimeReq＆Ans命令作为替代。











