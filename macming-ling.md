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
| 0x09 | _**TxParamSetupReq**_ | 网关 | 根据当地法规，由网络服务器用于设置终端设备的最大允许停留时间和最大EIRP |
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

















