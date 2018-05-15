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



