# 18 C类MAC命令

Class A规范中描述的所有命令都应在C类设备中实现。C类规范添加了以下MAC命令。

| CID | Command | 发送方 | 简要说明 |
| :---: | :---: | :---: | :---: |
| 0x20 | DeviceModeInd | 终端设备 | 由终端设备用于指示其当前的操作模式（A类或C类） |
| 0x20 | DeviceModeConf | 网关 | 由网络用于确认_**DeviceModeInd**_命令 |

## 18.1 设备模式（DeviceModeInd，DeviceModeConf）

使用_**DeviceModeInd**_命令，终端设备向网络表明它希望在A类或C类中运行。该命令具有一个字节的有效载荷，其定义如下：

| Size（bytes） | 1 |
| :---: | :---: |
| DeviceModeInd Payload | Class |

为上述命令定义的类为：

| Class | Value |
| :---: | :---: |
| Class A | 0x00 |
| RFU | 0x01 |
| Class C | 0x02 |

当网络服务器接收到_**DeviceModeInd**_命令时，它会用_**DeviceModeConf**_命令作出响应。设备应在所有上行链路中包含_**DeviceModeInd**_命令，直到接收到_**DeviceModeConf**_命令。

第一个_**DeviceModeInd**_命令发送后，设备应该立即切换模式。

> 注意：当从A类过渡到C类时，建议电池供电的设备在应用层实现超时机制，以确保在没有与网络连接的情况下，它不会无限期地停留在C类模式。

_**DeviceModeConf**_命令有一个1字节的有效载荷。

| Size（bytes） | 1 |
| :---: | :---: |
| DeviceModeConf Payload | Class |

定义的类参数如同**DeviceModeInd**命令

