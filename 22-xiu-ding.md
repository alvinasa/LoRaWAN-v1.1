# 22.1 版本1.0

* LoRaWAN1.0的批准版本

# 22.2 版本1.0.1

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

# 22.3 版本1.0.2

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

# 22.4 版本1.1

本节概述了LoRaWAN1.1和LoRaWAN1.0.2之间的主要变化。

# 22.4.1 说明

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

# 22.4.2 功能修改



























