对于网络管理，可以在网络服务器和终端设备上的MAC层之间专门交换一组MAC命令。MAC层命令对应用或应用服务器或运行在终端设备上的应用永远不可见。

单个数据帧可以包含任何序列的MAC命令，或者捎带在**FOpts**字段中，或者当作为单独的数据帧发送时，将**FPort**字段设置为0的**FRMPayload**字段中。捎带的MAC命令始终以加密方式发送，并且必须不超过15个字节。作为**FRMPayload**发送的MAC命令始终是加密的，不得超过最大**FRMPayload**长度。

一个MAC命令由一个1字节的命令标识符（**CID**）组成，后跟一个可能为空的命令特有的字节序列。

MAC命令由接收端应答/确认，与发送保持相同的顺序。每个MAC命令的应答被顺序添加到一个缓冲区。在单个帧中接收到的所有MAC命令都必须在单个帧中应答，这意味着包含应答的缓冲区必须在单个帧中发送。如果MAC应答的缓冲区长度大于最大FOpt字段，则设备必须在端口0上发送缓冲区来作为FRMPayload。如果设备同时具有应用负载和MAC应答待发送并且两者都不适合该帧，则MAC应答应优先发送。如果缓冲区的长度大于可用的最大FRMPayload大小，则在组装帧之前，设备应将缓冲区缩短到最大FRMPayload大小。因此，最后一个MAC命令的答案可能会被截断。在所有情况下，即使包含MAC回答的缓冲区必须被剪切，也会执行完整的MAC命令列表。网络服务器不得生成一系列MAC命令，这些命令可能无法在单个上行链路中由终端设备应答。网络服务器应计算可用于应答MAC命令的最大FRMPayload大小，如下所示：

* 如果最新的上行链路ADR位是0：务必考虑对应于最低数据速率的最大有效载荷大小
* 如果最新的上行链路ADR位被设置为1：务必考虑与用于设备的最后上行链路的数据速率相对应的最大有效载荷大小

> 注意：当收到一个缩短的MAC应答时，网络服务器可能会重新发送无法应答的MAC命令

| CID | Command | 发送方 | 简要说明 |
| :---: | :---: | :---: | :---: |
| 0x01 | _**ResetInd**_ | 终端设备 | 由ABP设备用于指示重置网络并协商协议版本 |
| 0x01 | _**ResetConf**_ | 网关 | 响应ResetInd命令 |
| 0x02 | _**LinkCheckReq**_ | 终端设备 | 由终端设备用于验证其与网络的连接 |
| 0x02 | _**LinkCheckAns**_ | 网关 | 响应LinkCheckReq命令。包含接收信号强度，告知终端接收质量（链路余量） |
| 0x03 | _**LinkADRReq**_ | 网关 | 请求终端设备更改数据速率，传输功率，重传率或信道。 |
| 0x03 | _**LinkADRAns**_ | 终端设备 | 响应LinkADRReq命令 |
| 0x04 | _**DutyCycleReq**_ | 网关 | 设置设备的最大传输占空比 |
| 0x04 | _**DutyCycleAns**_ | 终端设备 | 响应DutyCycleReq命令 |
| 0x05 | _**RXParamSetupReq**_ | 网关 | 设置接收时隙参数 |
| 0x05 | _**RXParamSetupAns**_ | 终端设备 | 响应RXParamSetupReq命令 |
| 0x06 | _**DevStatusReq**_ | 网关 | 请求终端设备的状态 |
| 0x06 | _**DevStatusAns**_ | 终端设备 | 返回终端设备的状态，即其电池电量和解调余量 |
| 0x07 | _**NewChannelReq**_ | 网关 | 创建或修改无线信道的定义 |
| 0x07 | _**NewChannelAns**_ | 终端设备 | 响应NewChannelReq命令 |
| 0x08 | _**RXTimingSetupReq**_ | 网关 | 设置接收时隙的时间 |
| 0x08 | _**RXTimingSetupAns**_ | 终端设备 | 响应RXTimingSetupReq命令 |
| 0x09 | _**TxParamSetupReq**_ | 网关 | 根据当地法规，由网络服务器用于设置终端设备的最大允许驻留时间和最大EIRP |
| 0x09 | _**TxParamSetupAns**_ | 终端设备 | 响应TxParamSetupReq命令 |
| 0x0A | _**DlChannelReq**_ | 网关 | 通过从上行链路频率改变下行链路频率来修改下行RX1无线信道的定义（即创建非对称信道） |
| 0x0A | _**DlChannelAns**_ | 终端设备 | 响应DlChannelReq命令 |
| 0x0B | _**RekeyInd**_ | 终端设备 | 用于OTA设备通知安全上下文更新（更新密钥） |
| 0x0B | _**RekeyConf**_ | 网关 | 响应RekeyInd命令 |
| 0x0C | _**ADRParamSetupReq**_ | 网关 | 网络服务器用于设置终端设备的ADR\_ACK\_LIMT和ADR\_ACK\_DELAY参数 |
| 0x0C | _**ADRParamSetupAns**_ | 终端设备 | 响应ADRParamSetupReq命令 |
| 0x0D | _**DeviceTimeReq**_ | 终端设备 | 由终端设备用于请求当前日期和时间 |
| 0x0D | _**DeviceTimeAns**_ | 网关 | 由网络发送，响应DeviceTimeReq请求 |
| 0x0E | _**ForceRejoinReq**_ | 网关 | 由网络发送，要求设备立即重新加入，可选的定期重试 |
| 0x0F | _**RejoinParamSetupReq**_ | 网关 | 由网络用于设置定期设备重新加入消息 |
| 0x0F | _**RejoinParamSetupAns**_ | 终端 | 响应RejoinParamSetupReq命令 |
| 0x80 to 0xFF | Proprietary | 双向 | 专用于网络命令扩展 |

> 注意：一般来说，终端设备只会回复一次接收到的任何Mac命令。如果响应丢失，网络必须再次发送命令。当收到一个不包含响应的新上行链路时，网络决定必须重新发送该命令。只有_**RxParamSetupReq**_，_**RxTimingSetupReq**_和_**DlChannelReq**_在它们的相关部分中描述了不同的确认机制，因为它们会影响下行链路参数。
>
> 注意：当终端设备发起MAC命令时，网络会尽力在请求后的RX1 / RX2窗口中立即发送确认/应答。如果没有在该时隙中接收到应答，则终端设备可以自由地实施任何其需要的重试机制。
>
> 注意：MAC命令的长度没有明确给出，必须由MAC实现隐含知道。因此未知的MAC命令不能被跳过，并且第一个未知的MAC命令终止MAC命令序列的处理。因此建议根据MAC命令首次出现的LoRaWAN规范版本来排序MAC命令。通过这种方式，所有的LoRaWAN规范版本之前的所有MAC命令都可以被处理，即使指定的当前MAC命令是在更早的LoRaWAN规范中实现。

# 5.1 重置指示命令（ResetInd，ResetConf）

此MAC命令仅适用于兼容LoRaWAN1.1网络服务器上激活的ABP设备。LoRaWAN1.0服务器不实现此MAC命令。

OTA设备不能执行这个命令。网络服务器应忽略来自OTA设备的**ResetInd**命令。

使用**ResetInd**命令，ABP终端设备向网络表示它已被重新初始化并且已经切换回其默认的MAC和无线参数（即除了三个帧计数器之外，这个参数在最早在制造时就已编程到设备中）。必须将**ResetInd**命令添加到所有上行链路的FOpt字段中，直到收到**ResetConf**。

该命令不会向网络服务器发信号通知下行链路帧计数器已被重置。帧计数器（上行和下行）不得在ABP设备中重置。

> 注意：此命令适用于某一时刻可能会中断电源的ABP设备（例如电池更换）。设备可能丢失存储在RAM中的MAC层上下文（除了必须存储在NVM中的帧计数器）。在这种情况下，设备需要一种方法将该情景丢失传达给网络服务器。在未来版本的LoRaWAN协议中，该命令也可用于协商设备和网络服务器之间的某些协议选项。

**ResetInd**命令包含终端设备支持的LoRaWAN版本的次要版本。

| Size（bytes） | 1 |
| :---: | :---: |
| ResetInd Payload | Dev LoRaWAN version |

| Size（bytes） | 7:4 | 3:0 |
| :---: | :---: | :---: |
| Dev LoRaWAN version | RFU | Minor=1 |

次要字段表示终端设备支持的LoRaWAN版本的次要版本。

| Minor version | Minor |
| :---: | :---: |
| RFU | 0 |
| 1（LoRaWAN x.1） | 1 |
| RFU | 2:15 |

当网络服务器收到**ResetInd**时，它会用**ResetConf**命令作出响应 。

ResetConf命令包含一个单字节有效载荷，由网络服务器所支持的LoRaWAN版本编码，使用与“Dev LoRaWAN version”相同的格式。

| Size（bytes） | 1 |
| :---: | :---: |
| ResetConf Payload | Serv LoRaWAN version |

**ResetConf**携带的服务器版本必须与设备版本相同。任何其他值都是无效的。

如果服务器版本无效，则设备应丢弃**ResetConf**命令并在下一个上行链路帧中重新发送**ResetInd**。

# 5.2 链路检查命令（LinkCheckReq，LinkCheckAns）

使用**LinkCheckReq**命令，终端设备可以验证其与网络的连接。该命令没有有效载荷。

当网络服务器通过一个或多个网关接收到**LinkCheckReq**时，它会使用**LinkCheckAns**命令进行响应。

| Size（bytes） | 1 | 1 |
| :---: | :---: | :---: |
| LinkCheckAns Payload | Margin | GwCnt |

解调余量（**Margin**）是一个8位无符号整数，范围为0..254，表示上次成功接收**LinkCheckReq**命令的链路余量（以dB为单位）。值“0”意味着帧在解调层被接收（0dB或无余量），而值“20”例如意味着帧到达网关比解调层多20dB。值“255”被保留。

网关计数（**GwCnt**）是成功接收最后一个**LinkCheckReq**命令的网关数值。

# 5.3 链路ADR命令（LinkADRReq，LinkADRAns）

使用**LinkADRReq**命令，网络服务器请求终端设备执行速率适配。

| Size（bytes） | 1 | 2 | 1 |
| :---: | :---: | :---: | :---: |
| LinkADRReq Payload | DataRate\_TXPower | ChMask | Redundancy |

| Bits | \[7:4\] | \[3:0\] |
| :---: | :---: | :---: |
| DataRate\_TXPower | DataRate | TXPower |

请求数据速率（**DataRate**）和TX输出功率（**TXPower**）是区域特定的，并按照\[PHY\]中的指示进行编码。命令中指示的TX输出功率将被视为设备可能运行的最大发射功率。终端设备将指定一个比当前能使用的更高发送功率来确认一个命令的成功，在这种情况下，必须尽可能以其最大功率运行。DataRate或TXPower的值为0xF（十进制为15）表示设备必须忽略该字段，并保留当前参数值。信道掩码（**ChMask**）对可用于上行链路接入的信道进行如下编码，对应于LSB的比特0：

| Bit\# | Usable channels |
| :---: | :---: |
| 0 | Channel 1 |
| 1 | Channel 2 |
| .. | .. |
| 15 | Channel 16 |

**ChMask**字段中的位置为1意味着如果该信道允许终端设备使用当前的数据速率，则相应的信道可用于上行链路传输。设置为0的位表示应该避开相应的信道。

| Bits | 7 | \[6:4\] | \[3:0\] |
| :---: | :---: | :---: | :---: |
| Redundancy bits | RFU | ChMaskCntl | NbTrans |

在冗余位中，**NbTrans**字段是每个上行链路消息的传输数量。这适用于“已确认”和“未确认”的上行链路帧。对应于每帧的单次传输，默认值是1。有效范围是\[1:15\]。如果收到**NbTrans** == 0，终端设备应保持当前的NbTrans值不变。

信道掩码控制（**ChMaskCntl**）字段控制先前定义的ChMask位掩码的说明。它控制ChMask应用的16个通道的块。它也可以用于全局打开或关闭所有使用特定调制的信道。该字段的用法是区域特定的，并在\[PHY\]中定义。

网络服务器可以在单个下行链路消息内包括多个连续的LinkADRReq命令。为了配置终端设备信道掩码，终端设备必须按照下行消息中的顺序来处理所有连续的LinkADRReq消息，并作为单个原子块命令。网络服务器不得在下行消息中包含多个这样的原子块命令。终端设备必须发送一个LinkADRAns命令来接受或拒绝整个ADR原子命令块。如果下行消息携带多个ADR原子命令块，则终端设备只处理第一个命令块，并响应所有其他ADR命令块发送NAck（所有状态位均设置为0的LinkADRAns命令）。设备必须只处理来自相邻ADR命令块中的最后一个LinkADRReq命令的DataRate，TXPower和NbTrans，因为这些设置控制着这些值的终端设备全局状态。按顺序处理连续的ADR命令块中的所有信道掩码控制之后，响应的信道掩码ACK位必须反映对最后通道的接受/拒绝规划。

信道频率是区域特定的，它们在\[PHY\]定义的。终端设备通过**LinkADRAns**命令应答**LinkADRReq**。

| Size（bytes） | 1 |
| :---: | :---: |
| LinkADRAns Payload | Status |

| Bits | \[7:3\] | 2 | 1 | 0 |
| :---: | :---: | :---: | :---: | :---: |
| Status bits | RFU | Power ACK | Data rate ACK | Channel mask ACK |

**LinkADRAns**状态位具有以下含义：

|  | Bit = 0 | Bit = 1 |
| :---: | :--- | :--- |
| Channel mask ACK | 发送的信道掩码启用尚未定义的信道或者信道掩码要求禁用所有信道。该命令被丢弃，终端设备的状态没有改变。 | 发送的信道掩码已成功解释。所有当前定义的信道状态都是根据掩码设置的。 |
| Data rate ACK | 所请求的数据速率对于终端设备是未知的，或者不可能由给定的信道掩码（不被任何启用的信道支持）提供。该命令被丢弃，终端设备状态没有改变。 | 数据速率已成功设置或请求的DataRate字段被设置为15，这意味着它被忽略 |
| Power ACK | 该设备无法以等于或低于要求的功率等级运行。该命令被丢弃，终端设备状态没有改变。 | 该设备能够以等于或低于请求的功率级别运行，或者请求的TXPower字段被设置为15，这意味着它应该被忽略 |

如果这三者中的任何一位等于0，则该命令不成功并且节点保持先前的状态。

# 5.4 终端设备发送占空比（DutyCycleReq，DutyCycleAns）

网络协调器使用**DutyCycleReq**命令来限制终端设备的最大聚合传输占空比。聚合传输占空比对应于所有子带上的传输占空比。

| Size（bytes） | 1 |
| :---: | :---: |
| DutyCycleReq Payload | DutyCyclePL |

| Bits | 7:4 | 3:0 |
| :---: | :---: | :---: |
| DutyCyclePL | RFU | MaxDCycle |

允许的最大终端设备传输占空比为：

_aggrated duty cycle_ = 1/2^MaxDCycle

**MaxDutyCycle**的有效范围是\[0：15\]。0值对应于“没有占空比限制”，区域法规规定的值除外。

终端设备通过**DutyCycleAns**命令来应答**DutyCycleReq**。**DutyCycleAns** MAC回复不包含任何有效负载。

# 5.5 接收窗口参数（RXParamSetupReq，RXParamSetupAns）

**RXParamSetupReq**命令允许更改每个上行链路之后第二个接收窗口（RX2）的频率和数据速率。该命令还允许对上行链路和RX1时隙下行链路数据速率之间的偏移进行设置。

| Size（bytes） | 1 | 3 |
| :---: | :---: | :---: |
| RXParamSetupReq Payload | DLsettings | Frequency |

| Bits | 7 | 6:4 | 3:0 |
| :---: | :---: | :---: | :---: |
| DLsettings | RFU | RX1DRoffset | RX2DataRate |

RX1DRoffset字段设置上行数据速率与下行数据速率之间的偏移量，用于与在第一个接收时隙（RX1）上的终端设备进行通信。默认情况下，该偏移量为0。该偏移量用于考虑某些地区基站的最大功率密度限制，并平衡上行链路和下行链路无线链路余量。

数据速率（RX2**DataRate**）字段按照与**LinkADRReq**命令相同的约定（例如0表示DR0 / 125kHz），使用第二个接收窗口定义下行链路的数据速率。频率（**Frequency**）字段对应于用于第二个接收窗口的信道频率，即频率按照**NewChannelReq**命令中定义的约定进行编码。

**RXParamSetupAns**命令由终端设备用来确认接收**RXParamSetupReq**命令。必须将**RXParamSetupAns**命令添加到所有上行链路的FOpt字段中，直到终端设备收到A类下行链路。这保证即使在存在上行链路包丢失的情况下，网络也始终知道终端设备使用的下行链路参数。

有效载荷包含单个状态字节。

| Size（bytes） | 1 |
| :---: | :---: |
| RXParamSetupAns Payload | Status |

状态（**Status**）位具有以下含义。

| Bits | 7:3 | 2 | 1 | 0 |
| :---: | :---: | :---: | :---: | :---: |
| Status bits | RFU | RX1DRoffset ACK | RX2 Data rate ACK | Channel ACK |

|  | Bit = 0 | Bit = 1 |
| :--- | :--- | :--- |
| Channel ACK | 请求的频率不被终端设备使用。 | RX2时隙信道已成功设置 |
| RX2DataRate ACK | 请求的数据速率对于终端设备是未知的。 | RX2时隙数据速率已成功设置 |
| RX1DRoffset ACK | RX1时隙的上行/下行数据速率偏移量不在允许范围内 | RX1DRoffset已成功设置 |

如果3位中的任何一位等于0，则该命令不成功，并且必须保留先前的参数。

# 5.6 终端设备状态（DevStatusReq，DevStatusAns）

通过**DevStatusReq**命令，网络服务器可以向终端设备请求状态信息。该命令没有有效载荷。如果终端设备收到**DevStatusReq**，它必须用**DevStatusAns**命令响应。

| Size（bytes） | 1 | 1 |
| :---: | :---: | :---: |
| DevStatusAns Payload | Battery | Margin |

报告的电池电量（**Battery**）编码如下：

| Battery | Description |
| :---: | :---: |
| 0 | 终端设备连接到外部电源。 |
| 1..254 | 电池电量，最小值为1，最大值为254。 |
| 255 | 终端设备无法测量电池电量。 |

余量（**Margin**）是以dB为单位的解调信噪比，最近成功接收的**DevStatusReq**命令会取整为最接近的整数值。它是一个6位的有符号整数，最小值为-32，最大值为31。

| Bits | 7:6 | 5:0 |
| :---: | :---: | :---: |
| Status | RFU | Margin |

# 5.7 信道的创建/修改（NewChannelReq，NewChannelAns，DlChannelReq，DlChannelAns）

设备运行在定义了固定信道规划的地区不应执行这些MAC命令。这些命令不应该被设备应答。请参阅\[PHY\]了解适用的区域。

**NewChannelReq**命令可用于修改现有双向信道的参数或创建新信道。该命令设置新信道的中心频率和该信道上可用的上行数据速率范围：

| Size（bytes） | 1 | 3 | 1 |
| :---: | :---: | :---: | :---: |
| NewChannelReq Payload | ChIndex | Freq | DrRange |

信道索引（**ChIndex**）是正在创建或修改信道的索引。根据所使用的区域和频段，在某些区域（\[PHY\]），LoRaWAN规范强加了默认信道，这些信道必须是所有设备共有的，并且不能由**NewChannelReq**命令修改。如果默认信道数为_N_，信道从0到_N_-1，**ChIndex**的可接受范围是_N_到15.设备必须能够处理至少16个不同的信道定义。在某些地区，设备可能必须存储16个以上的信道定义。

频率（**Freq**）字段是24位无符号整数。以Hz为单位的实际信道频率为100 x **Freq**，其中代表100 MHz频率以下的值保留供将来使用。这允许以100Hz的步长设置100 MHz至1.67 GHz之间任何信道的频率。**Freq**值为0将禁用信道。终端设备必须检查频率是否被无线硬件实际允许，否则返回错误。

数据速率范围（**DrRange**）字段指定该信道允许的上行数据速率范围。该字段分为两个4位索引：

| Bits | 7:4 | 3:0 |
| :---: | :---: | :---: |
| DrRange | MaxDR | MinDR |

遵循5.3节中定义的约定，最小数据速率（**MinDR**）子字段指定该信道允许的最低上行数据速率。例如，使用欧洲区域参数，0表示DR0 / 125 kHz。类似地，最大数据速率（**MaxDR**）指定最高上行数据速率。例如，DrRange = 0x77意味着通道只允许50 kbps的GFSK，DrRange = 0x50则表示支持DR0 / 125 kHz到DR5 / 125 kHz。

新定义或修改的信道被启用便可立即用于通信。RX1下行链路频率被设置为等于上行链路频率。

终端设备通过发送**NewChannelAns**命令来确认接收到**NewChannelReq**。此消息的有效内容包含以下信息：

| Size（bytes） | 1 |
| :---: | :---: |
| NewChannelAns Payload | status |

状态（**Status**）位具有以下含义：

| Bits | 7:2 | 1 | 0 |
| :---: | :---: | :---: | :---: |
| Status | RFU | Data rate range ok | Channel frequency ok |

|  | Bit = 0 | Bit = 1 |
| :--- | :--- | :--- |
| Data rate range ok | 指定的数据速率范围超出了当前为此终端设备定义的数据速率范围 | 数据速率范围与终端设备的可能性兼容 |
| Channel frequency ok | 设备无法使用该频率 | 设备能够使用这个频率。 |

如果这两个比特中的任何一个等于0，则该命令不成功并且新的信道尚未被创建。

**DlChannelReq**命令允许网络将不同的下行链路频率与RX1时隙相关联。该命令适用于支持**NewChannelReq**命令的所有物理层规范（例如欧盟和中国的物理层，但不适用于美国或澳大利亚）。

该命令设置用于下行链路RX1时隙的中心频率，如下所示：

| Size（bytes） | 1 | 3 |
| :---: | :---: | :---: |
| DIChannelReq Payload | ChIndex | Freq |

信道索引（**ChIndex**）是下行链路频率被修改的信道索引。

频率（**Freq**）字段是24位无符号整数。以Hz为单位的实际下行链路频率为100 x **Freq**，其中代表100 MHz以下频率的值保留供将来使用。终端设备必须检查频率是否被无线硬件实际允许，否则返回错误。

终端设备通过发回一个**DlChannelAns**命令来确认接收到**DlChannelReq**。**DlChannelAns**命令应该被添加到所有上行链路的FOpt字段中，直到终端设备收到下行包。这保证即使在存在上行链路包丢失的情况下，网络也始终知道终端设备使用的下行链路频率。

此消息的有效内容包含以下信息：

| Size（bytes） | 1 |
| :---: | :---: |
| DIChannelAns Payload | Status |

状态（**Status**）位具有以下含义：

| Bits | 7:2 | 1 | 0 |
| :---: | :---: | :---: | :---: |
| Status | RFU | 上行频率存在 | 信道频率正常 |

|  | Bit = 0 | Bit = 1 |
| :--- | :--- | :--- |
| Channel frequency ok | 设备无法使用该频率 | 设备能够使用这个频率。 |
| Uplink frequency exists | 没有为该信道定义上行链路频率，只能为已经具有有效上行链路频率的信道设置下行链路频率 | 信道的上行频率是有效的 |

# 5.8 设置TX和RX之间的延迟（RXTimingSetupReq，RXTimingSetupAns）

**RXTimingSetupReq**命令允许配置TX上行链路结束和第一个接收时隙打开之间的延迟。第二个接收时隙在第一个接收时隙后一秒打开。

| Size（bytes） | 1 |
| :---: | :---: |
| RXTimingSetupReq Payload | 设置 |

延迟（**Delay**）字段指定延迟。该字段分为两个4位索引：

| Bits | 7:4 | 3:0 |
| :---: | :---: | :---: |
| Settings | RFU | Del |

延迟以秒表示。**Del **0被映射到1秒。

| Del | Delay \[s\] |
| :---: | :---: |
| 0 | 1 |
| 1 | 1 |
| 2 | 2 |
| 3 | 3 |
| .. | .. |
| 15 | 15 |

终端设备使用**RXTimingSetupAns**来应答**RXTimingSetupReq**，而无有效载荷。

**RXTimingSetupAns**命令应添加到所有上行链路的FOpt字段中，直到终端设备收到A类下行链路。这保证即使存在上行链路包丢失，网络总是知道终端设备使用的下行链路参数。

# 5.9 终端设备传输参数（TxParamSetupReq，TxParamSetupAns）

该MAC命令需要执行以符合规定在某些监管区域。请参考\[PHY\]。

**TxParamSetupReq**命令可以用于向终端设备通知最大允许驻留时间，即空中包的最大连续传输时间以及允许的最大终端设备有效全向辐射功率（EIRP）。

| Size（bytes） | 1 |
| :---: | :---: |
| TxParamSetupReq payload | EIRP\_DwellTime |

EIRP\_DwellTime字段的结构如下所述：

| Bits | 7:6 | 5 | 4 | 3:0 |
| :---: | :---: | :---: | :---: | :---: |
| MaxDwellTime | RFU | DownlinkDwellTime | UplinkDwellTime | MaxEIRP |

按照下表，使用**TxParamSetupReq**命令的位\[0 ... 3\]来编码最大EIRP值。本表中EIRP值的选择方式涵盖了不同区域法规所规定的各种最大EIRP限值。

| Coded Value | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Max EIRP（dBm） | 8 | 10 | 12 | 13 | 14 | 16 | 18 | 20 | 21 | 24 | 26 | 27 | 29 | 30 | 33 | 36 |

最大EIRP对应于设备的无线发射功率的上限。该设备不需要以该功率进行传输，但不得发射比指定的EIRP更高。

4和5位分别定义了最大上行链路和下行链路驻留时间，按照下表进行编码：

| Coded Value | Dwell Time |
| :---: | :---: |
| 0 | 不限制 |
| 1 | 400 ms |

当这个MAC命令被实现时（特定区域），终端设备通过发送**TxParamSetupAns**命令来确认**TxParamSetupReq**命令。**TxParamSetupAns**命令不包含任何有效负载。

当这个MAC命令在不需要的地区使用时，设备不处理它，并且不应该发送确认。

# 5.10 重新设置命令（RekeyInd，RekeyConf）

此MAC命令仅适用于在兼容LoRaWAN1.1网络服务器上激活的OTA设备。LoRaWAN1.0服务器不实现此MAC命令。

ABP设备不能执行这个命令。网络服务器应忽略来自ABP设备的_**RekeyInd**_命令。

对于OTA设备，_**RekeyInd**_ MAC命令用于确认安全密钥更新，以及在未来的LoRaWAN（&gt; 1.1）版本中协商在终端设备和网络服务器之间运行的次要LoRaWAN协议版本。该命令不会通知重置MAC和无线电参数（见6.2.3）。

_**RekeyInd**_命令包含终端设备支持的LoRaWAN版本的次要版本。

| Size（bytes） | 1 |
| :---: | :---: |
| RekeyInd Payload | Dev LoRaWAN version |

| Size（bytes） | 7:4 | 3:0 |
| :---: | :---: | :---: |
| Dev LoRaWAN version | RFU | Minor=1 |

这个次要字段表示终端设备支持的LoRaWAN版本的次要版本。

| Minor version | Minor |
| :---: | :---: |
| RFU | 0 |
| 1（LoRaWAN x.1） | 1 |
| RFU | 2:15 |

OTA设备应在成功处理Join-Accept（已派生新的会话密钥）之后，在所有确认和未确认的上行链路帧中发送_**RekeyInd**_，直到收到_**RekeyConf**_。如果设备在第一个ADR\_ACK\_LIMIT上行链路内没有收到RekeyConf，它将恢复到加入状态。这样的设备在之后的任何时间里发送的_**RekeyInd**_命令都应被网络服务器丢弃。在发送**Join-accept**之后和携带_**RekeyInd**_命令的第一个上行链路帧之前，网络服务器应丢弃接收到的使用新安全上下文保护的所有上行链路帧。

当网络服务器接收到一个_**RekeyInd**_时，它会用_**RekeyConf**_命令作出响应。

RekeyConf命令包含一个单字节有效载荷，该有效载荷使用与“dev LoRaWAN version”相同的格式，由网络服务器支持的LoRaWAN版本编码。

| Size（bytes） | 1 |
| :---: | :---: |
| RekeyConf Payload | Serv LoRaWAN version |

服务器版本必须大于0（不允许为0），且小于或等于（&lt;=）设备的LoRaWAN版本。因此，对于LoRaWAN1.1设备，唯一有效值为1。如果服务器版本无效，设备应丢弃_**RekeyConf**_命令并在下一个上行链路帧中重新传输_**RekeyInd**_。

# 5.11 ADR参数（ADRParamSetupReq，ADRParamSetupAns）

_**ADRParamSetupReq**_命令允许更改定义ADR回退算法的ADR\_ACK\_LIMIT和ADR\_ACK\_DELAY参数。ADRParamSetupReq命令具有单字节有效负载。

| Size（bytes） | 1 |
| :---: | :---: |
| ADRParamSetupReq Payload | ADRparam |

| Bits | 7:4 | 3:0 |
| :---: | :---: | :---: |
| ADRparam | Limit\_exp | Delay\_exp |

Limit\_exp字段设置ADR\_ACK\_LIMIT参数值：

ADR\_ACK\_LIMIT = 2^Limit\_exp

Limit\_exp有效范围是0到15，对应于ADR\_ACK\_LIMIT的范围1到32768

Delay\_exp字段设置ADR\_ACK\_DELAY参数值。

ADR\_ACK\_ DELAY = 2^Delay\_exp

Delay\_exp有效范围是0到15，对应于ADR\_ACK\_ DELAY的范围1到32768

_**ADRParamSetupAns**_命令由终端设备用来确认接收_**ADRParamSetupReq**_命令。_**ADRParamSetupAns**_命令没有有效负载字段。

# 5.12 DeviceTime命令（DeviceTimeReq，DeviceTimeAns）

此MAC命令仅可用于在兼容LoRaWAN1.1的网络服务器上激活的设备。LoRaWAN1.0服务器不实现此MAC命令。

使用_**DeviceTimeReq**_命令，终端设备可以向网络请求当前的网络日期和时间。该请求没有有效负载。

使用_**DeviceTimeAns**_命令，网络服务器将网络日期和时间提供给终端设备。所提供的时间是在上行链路传输结束时捕捉的网络时间。该命令有一个5字节的有效负载，定义如下：

| Size（byte） | 4 | 1 |
| :---: | :---: | :---: |
| DeviceTimeAns Payload | 32位无符号整数：从时代开始的秒数\* | 8位无符号整数：以1/2^8秒为单位的小数秒 |

网络提供的时间必须具有+/- 100mSec的最差情况精度。

（\*）GPS时代（即1980年6月1日星期日午夜）被用作原点。“秒”字段是从原点开始经过的秒数。该字段每秒单调递增1。要将此字段转换为UTC时间，必须考虑闰秒。

> 例如：2016年2月12日星期五14:24:31 UTC对应GPS自GPS时代以来的1139322288秒。截至2017年6月，GPS时间比UTC时间提前17秒。

# 5.13 强制重新加入命令（ForceRejoinReq）

通过强制重新加入命令，网络要求设备立即发送重新加入请求类型0或类型2的消息，并且具有可编程的重试次数，周期性和数据速率。该RejoinReq上行链路可以被网络用来立即重新设置设备或发起切换漫游过程。

该命令有两个字节的有效负载。

| Bits | 15:14 | 13:11 | 10:8 | 7 | 6:4 | 3:0 |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| ForceRejoinReq bits | RFU | Period | Max\_Retries | RFU | RejoinType | DR |

参数编码如下：

周期：重传之间的延迟应等于32秒 x 2^Period+ Rand32，其中Rand32是\[0:32\]范围内的伪随机数。

Max\_Retries：设备将重试这个重新加入请求的总次数。

* 0：重新加入只发送一次（不重试）
* 1：重新加入总共必须发送2次（1 + 1重试）
* ...
* 7：重新加入必须发送8次（1 + 7次重试）

RejoinType：该字段指定应由设备发送的重新加入请求的类型。

* 0或1：应发送重新加入请求类型0
* 2：应发送重新申请类型2
* 3至7：RFU

DR：重新加入请求帧应使用数据速率DR发送。实际物理调制数据速率与DR值之间的对应关系遵循与_**LinkADRReq**_命令相同的约定，并且针对\[PHY\]中的每个区域定义

该命令没有应答，因为设备收到该命令时务必发送一个Rejoin-Request。RejoinReq消息的第一次传输应在接收到命令后立即完成（但网络可能不会收到）。

如果设备在达到传输重试次数之前收到一个新的**ForceRejoinReq**命令，设备应该用新参数继续传输RejoinReq。

# 5.14 RejoinParamSetupReq（RejoinParamSetupAns）

通过RejoinParamSetupReq命令，周期性地发送RejoinReq类型0消息，该消息具有可编程周期性，被定义为上行链路的时间或数量。

时间和数量被推荐来处理可能没有时间测量能力的设备。指定的周期性设置两个RejoinReq传输之间的上行链路的最大时间或数量。设备可以更频繁地发送RejoinReq。

该命令具有单字节有效负载。

| Bits | 7:4 | 3:0 |
| :---: | :---: | :---: |
| RejoinParamSetupReq bits | MaxTimeN | MaxCountN |

参数定义如下：

MaxCountN = C = 0至15。设备必须至少在每2^\(C + 4\)个上行消息中发送一个重加请求类型0的消息。

MaxTimeN = T = 0至15；设备必须至少在每2^\(T + 10\)秒内发送一次重加入请求类型0的消息。

* T = 0大约相当于17分钟
* T = 15约为1年

每当满足2个条件（帧计数或时间）之一时，都会发送RejoinReq数据包。

设备必须执行上行链路计数周期。基于时间的周期性是可选的。无法实现时间限制的设备必须在应答中发出信号。应答有单字节有效载荷。

| Bits | Bits 7:1 | Bit 0 |
| :---: | :---: | :---: |
| Status bits | RFU | TimeOK |

如果Bit 0 = 1，则设备已接受时间和计数限制，否则它只接受计数限制。

> 注意：对于消息速率非常低且无时间测量能力的设备，LoRaWAN中未规定达到最佳计数限制的机制。



