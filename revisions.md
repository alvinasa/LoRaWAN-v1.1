# 22 版本

## 22.1 版本1.0

* LoRaWAN1.0的批准版本

## 22.2 版本1.0.1

* 阐明了RX窗口的开始时间定义
* 在NA部分更正了DR2的最大有效载荷大小
* 更正了7.2.2中下行数据速率范围的拼写错误
* 引入了在7.2.2中使用编码率4/5的要求，以保证空中的最大时间&lt;400毫秒
* 更正了6.2.5中的加入接受MIC计算
* 阐明了NbRep字段并将其重命名为5.2中的NbTrans
* 删除了不加密MAC层中应用有效负载的可能性，删除了4.3.3.2段。如果应用需要进一步的安全性，则应用层将使用任何方法对有效负载进行加密，然后使用指定的默认LoRaWAN加密在MAC层重新加密
* 更正了FHDR字段大小错字
* 当chMaskCntl等于7.2.5中的6或7时，修正ChMask影响的信道
* 阐明描述JoinResp消息中的RX1时隙数据速率偏移的6.2.5语句
* 根据定义，DR &gt; 4将不会用于上行链路，因此删除了7.2.7中DRoffset表的后半部分
* 取消了EU868Mhz ISM频段的明确占空比限制实现（章节7.1）
* 使用RXtimingSetupAns和RXParamSetupAns sticky MAC命令来避免终端设备的隐藏状态问题。（在5.4和5.7中）
* 增加了中国470-510MHz测量频段的频率规划
* 增加了澳大利亚915-928MHz ISM频段的频率规划

## 22.3 版本1.0.2

* 提取的第7节“物理层”现在将成为一个单独的文档“LoRaWAN区域物理层定义”
* 纠正了ADR\_backoff序列描述（写入了ADR\_ACK\_LIMT而不是ADR\_ACK\_DELAY）4.3.1.1节
* 更正了第18.2节标题中的格式问题（之前的1.0.1版中的第19.2节）
* 增加了DlChannelRec MAC命令，该命令用于修改终端设备预期下行链路的频率。
* 增加了Tx ParamSetupRec MAC命令。通过该命令可以远程修改设备在特定区域的最大TX驻留时间和最大无线发射功率
* 增加了终端设备在5.2中的单个块中处理多个ADRreq命令的功能
* 阐明AppKey的定义引入了ResetInd / ResetConf MAC命令
* 为了清晰起见，7.1.3中的分开数据速率和txpower表
* 向A类添加了DeviceTimeReq/Ans MAC命令
* 将B类时间起源更改为GPS纪元，添加了BeaconTimingAns描述
* 将B类的所有信标对准到同一时段。B类信标现在对所有网络都公共。
* 分离的AppKey和NwkKey独立派生AppSKeys和NetSKeys。
* 分离NetSKeyUp和NetSKeyDnw用于漫游

## 22.4 版本1.1

本节概述了LoRaWAN1.1和LoRaWAN1.0.2之间的主要变化。

### 22.4.1 说明

* 语法的
* 规范性文字一贯使用
* ADR行为
  * 介绍了ADR命令块处理的概念
  * TXPower处理
  * 默认频道重新启用
  * ADR回退行为
* 默认的TXPower定义
* FCnt不能重复使用相同的会话密钥
* MAC命令如果存在于FOpts和Payload中则被丢弃
* 重传回退说明

### 22.4.2 功能修改

* FCnt变化
  * 所有计数器都是32位宽，不再支持16位
  * 将FCntDown分离为AFCntDown和NFCntDown
    * 从NS / AS中删除状态同步要求
  * 如果FCnt间隙大于MAX\_FCNT\_GAP，则移除丢弃帧的要求
    * 不需要32位计数器
  * 成功处理加入接受后，终端设备帧计数器将重置
  * ABP设备不得重置帧计数器
* 重传（不增加FCnt的传输）
  * 下行链路帧不会重新传输
  * 后续接收具有相同FCnt的帧将被忽略
  * 上行链路重传由NbTrans控制（这包括确认帧和未确认帧）
  * 在RX1和RX2接收窗口都过期之前，重发可能不会发生
  * B / C类设备在接收到RX1窗口中的帧后停止重新发送帧
  * A类设备在接收到RX1或RX2窗口中的帧时停​​止重新发送帧
* 密钥变化
  * 增加了一个新的根密钥（拆分密码功能）
    * NwkKey和AppKey
  * 增加了新的会话密钥
    * NwkSEncKey加密Fport = 0的有效载荷（MAC命令有效载荷）
    * AppSKey加密Fport != 0（应用有效载荷）
    * NwkSIntKey用于MIC下行链路帧
      * 对于设置了ACK位的下行链路，产生ACK的确认上行链路的AFCntUp的2个LSB被加到MIC计算中
    * SNwkSIntKey和FNwkSIntKey用于MIC上行链路帧
      * 每个用于计算2个独立的16位MIC，并将其组合为一个32位MIC
      * SNwkSIntKey部分被认为是“私有的”，不与漫游的fNs共享
      * FNwkSIntKey部分被认为是“公共的”，并可能与漫游的fNs共享
      * 私有的MIC部分现在使用TxDr，TxCh
      * 对于设置了ACK位的上行链路，产生ACK的确认下行链路的FCntDown的2个LSB被添加到专用MIC计算
    * 全部的密钥定义在后面（第6节）
  * 相关的MIC和加密改为使用新密钥
* MAC命令介绍
  * TxParamSetupReq/Ans
  * DlChannelReq/Ans
  * ResetInd/Conf
  * ADRParamSetupReq/Ans
  * DeviceTimeReq/Ans
  * ForceRejoinReq
  * RejoinParamSetupReq/Ans
  * 对于linkADRReq命令
    * 值0xF对于Dr或TXPower将被忽略
    * NbTrans的值0被忽略
* 激活
  * JoinEUI替换AppEUI（阐明）
  * EUI完全定义
  * 根密钥定义
    * NwkKey
    * AppKey
  * 添加了额外的会话密钥（拆分MIC /加密密钥）
    * SNwkSIntKeyUp和FNwkSIntKeyUp（拆分MIC上行链路）
    * NwkSIntKeyDown（MIC下行链路）
    * NwkSEncKey（加密上行/下行）
    * JSIntKey（重新加入请求和相关的加入接受）
    * JSencKey（加入接受响应重新加入请求）
  * 会话上下文定义
* OTAA
  * 加入接受MIC修改以防止重放攻击
  * 会话密钥派生定义
  * 重新加入请求消息定义（一个新的LoRaWAN消息类型\[MType\]）
    * 0 - 切换漫游协助
    * 1 - 后端状态恢复辅助
    * 2 - 重新设置会话密钥
  * 所有的随机数现在都是计数器（不再是随机数）
  * NetId阐明（与本地网络关联）
  * 在加入接受中定义的OptNeg位用于识别1.0或1.1+网络后端的操作版本
    * 1.0操作反转由1.1设备定义
* ABP
  * 描述额外会话密钥的要求
* B类
  * 网络现在控制设备的DR
  * 信标定义已移至区域文档
  * 说明
  * 弃用BeaconTimingReq / Ans（由标准MAC命令DeviceTimeReq / Ans取代）
* C类
  * 明确DL超时的要求
  * 添加C类MAC命令
    * DeviceModeInd/Conf

### 22.4.3 示例

* 在重传期间删除积极的数据速率回退示例

