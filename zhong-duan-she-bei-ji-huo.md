要加入LoRaWAN网络，每个终端设备必须进行个性化和激活。

终端设备的激活可通过两种方式实现，无论是通过空中激活（OTAA）还是通过个性化激活（ABP）

# 6.1 数据存储在终端设备中

## 6.1.1 激活之前

### 6.1.1.1 JoinEUI

JoinEUI是IEEE EUI64地址空间中的全局应用ID，它唯一标识了入网服务器，它能够协助处理Join过程和会话密钥派生。

对于OTAA设备，JoinEUI必须在Join过程执行之前存储在终端设备中。仅限ABP的终端设备，JoinEUI不是必需的。

### 6.1.1.2 DevEUI

DevEUI是IEEE EUI64地址空间中的全球终端设备ID，可唯一标识终端设备。

DevEUI是Network Server推荐的唯一设备标识符，无论使用何种激活程序，都可以识别跨网络漫游的设备。

对于OTAA设备，必须在执行Join过程之前将DevEUI存储在终端设备中。ABP设备不需要将DevEUI存储在设备本身中，但推荐这样做。

> 注意：建议DevEUI在设备标签上也可用，以便设备管理。

### 6.1.1.3 设备根密钥（AppKey＆NwkKey）

NwkKey和AppKey是特定于终端设备的AES-128根密钥，在制造过程中分配给终端设备。①当终端设备通过空中激活加入网络时，NwkKey用于派生FNwkSIntKey，SNwkSIntKey和NwkSEncKey会话密钥，AppKey用于派生AppSKey会话密钥。

> 注意：在使用v1.1网络服务器时，应用程序会话密钥仅从AppKey派生，因此NwkKey可能会交给网络运营商来管理JOIN过程，而不允许操作者窃听应用有效载荷数据。

根密钥NwkKey和AppKey在终端设备和后端的安全配置，存储和使用是解决方案整体安全性的本质。这些待实现并超出了本文的范围。但是，此解决方案的元素可能包括SE（安全元素）和HSM（硬件安全模块）。

为了确保向后兼容LoraWAN 1.0及更早版本的不支持两个根密钥的网络服务器，终端设备在加入此类网络时务必默认回到单根密钥方案。在这种情况下，只使用根NwkKey。这种情况通知终端设备通过加入接受消息的DLsetting字段的“OptNeg”位（位7）为零。在这种情况下，终端设备必须

* 按照LoRaWAN1.0规范，使用NwkKey派生AppSKey和FNwkSIntKey会话密钥。
* 根据LoRaWAN1.0规范，设置SNwkSIntKey和NwkSEncKey等于FNwkSIntKey，相同的网络会话密钥被有效地用于上行链路和下行链路MIC计算以及MAC有效载荷的加密。

NwkKey必须存储在打算使用OTAA过程的终端设备上。

对于仅限ABP的终端设备，NwkKey不是必需的。

AppKey必须存储在打算使用OTAA过程的终端设备上。

对于仅限ABP的终端设备，Appkey不是必需的。

NwkKey和AppKy都应该以防止恶意行为者提取和重用的方式存储。

①由于所有终端设备都配备了针对每个终端设备的独特应用和网络根密钥，因此从终端设备提取AppKey / NwkKey只会影响这一个终端设备。

### 6.1.1.4 JSIntKey和JSEncKey派生

对于OTA设备，两个特定的有效期密钥是从NwkKey根密钥导出的：

* JSIntKey用于MIC重新加入请求类型1消息和加入接受响应
* JSEncKey用于加密由重新加入请求触发的加入接受

JSIntKey = aes128\_encrypt\(NwkKey, 0x06 \| DevEUI \| pad16\)

JSEncKey = aes128\_encrypt\(NwkKey, 0x05 \| DevEUI \| pad16\)

## 6.1.2 激活之后

在激活之后，以下附加信息被存储在终端设备中：设备地址（**DevAddr**），网络会话密钥的三元组（**NwkSEncKey** / **SNwkSIntKey** / **FNwkSIntKey**）和应用会话密钥（**AppSKey**）。

### 6.1.2.1 终端设备地址（DevAddr）

**DevAddr**由32位组成，用于识别当前网络中的终端设备。

DevAddr由终端设备的网络服务器分配。

其格式如下：

| Bit\# | \[31..32-N\] | \[31-N..0\] |
| :---: | :---: | :---: |
| DevAddr bits | AddrPrefix | NwkAddr |

其中N是\[7:24\]范围内的整数。

LoRaWAN协议支持具有不同网络地址空间大小的各种网络地址类型。除了为测试/专用网络保留的AddrPrefix值之外，可变大小的AddrPrefix字段是从由LoRa联盟分配的网络服务器的唯一标识符**NetID**（见6.2.3）派生而来的。AddrPrefix字段允许在漫游期间发现当前管理终端设备的网络服务器。不遵守此规则的设备不能在两个网络之间漫游，因为无法找到其主网络服务器。

最低有效位（32-N）即终端设备的网络地址（NwkAddr）可由网络管理员任意分配。

以下AddrPrefix值可以被任何私人/测试网络使用，并且不会被LoRa Aliance分配。

| Private/experimental network reserved AddrPrefix |
| :---: |
| N=7 |
| AddrPrefix = 7'b0000000 or AddrPrefix = 7'b0000001 |
| NwkAddr = 25bits freely allocated by the network manager |

请参阅\[BACKEND\]了解AddrPrefix字段的确切结构以及各种地址类的定义。

### 6.1.2.2 转发网络会话完整性密钥（FNwkSIntKey）

FNwkSIntKey是特定于终端设备的网络会话密钥。终端设备使用它来计算所有上行链路数据消息的MIC或MIC（消息完整性代码）的一部分，以确保4.4中规定的数据完整性。

FNwkSIntKey应该以防止恶意行为者提取和重复使用的方式进行存储。

### 6.1.2.3 提供网络会话完整性密钥（SNwkSIntKey）

**SNwkSIntKey**是特定于终端设备的网络会话密钥。终端设备使用它来验证所有下行数据消息的**MIC**（消息完整性代码），以确保数据完整性并计算上行消息一半的MIC。

> 注意：上行链路MIC计算依赖于两个密钥（FNwkSIntKey和SNwkSIntKey），以便允许漫游设置中的转发网络服务器能够仅验证MIC字段的一半

当设备连接到LoRaWAN1.0网络服务器时，同样的密钥用于4.4中规定的上行和下行MIC计算。在这种情况下，**SNwkSIntKey**的值与**FNwkSIntKey**的值相同。

**SNwkSIntKey**应该以防止恶意行为者提取和重复使用的方式进行存储。

### 6.1.2.4 网络会话加密密钥（NwkSEncKey）

NwkSEncKey是特定于终端设备的网络会话密钥。它用于加密和解密在端口0或FOpt字段中作为有效载荷传输的上行和下行MAC命令。当设备连接到LoRaWAN1.0网络服务器时，同一个密钥用于MAC有效负载加密和MIC计算。在这种情况下，**NwkSEncKey**采用与**FNwkSIntKey**相同的值。

NwkSEncKey应该以防止恶意行为者提取和重复使用的方式进行存储。

### 6.1.2.5 应用会话密钥（AppSKey）

**AppSKey**是特定于终端设备的**应用会话密钥**。它由应用服务器和终端设备用来加密和解密应用特定数据消息的有效负载字段。应用有效负载在终端设备和应用服务器之间进行端对端加密，但它们仅以逐跳方式进行完整性保护：终端设备和网络服务器之间的一跳，以及另一跳网络服务器和应用服务器。这意味着恶意网络服务器可能能够改变传输中的数据消息的内容，甚至可以通过观察应用端点对改变的数据的反应来帮助网络服务器推断关于数据的一些信息。网络服务器被认为是可信的，但希望实现端到端保密性和完整性保护的应用可以使用额外的端到端安全解决方案，这超出了本规范的范围。

**AppSKey**应该以防止恶意行为者提取和重复使用的方式进行存储。

### 6.1.2.6 会话上下文

会话上下文包含网络会话和应用会话。

网络会话包含以下状态：

* F/SNwkSIntKey

* NwkSEncKey

* FCntUp

* FCntDwn \(LW 1.0\) or NFCntDwn \(LW 1.1\)

* DevAddr

应用会话包含以下状态：

* AppSKey

* FCntUp

* FCntDown \(LW 1.0\) or AFCntDwn \(LW 1.1\)

网络会话状态由NS和终端设备维护。应用会话状态由AS和终端设备维护。

在完成OTAA或ABP过程后，NS / AS和终端设备之间建立了一个新的安全会话上下文。密钥和终端设备地址在会话期间不变（FNwkSIntKey，SNwkSIntKey，AppSKey，DevAddr）。帧计数器在会话期间交换帧流量（FCntUp，FCntDwn，NFCntDwn，AFCntDwn）时递增。

对于OTAA设备，帧计数器不能重复用于给定的密钥，因此必须在帧计数器饱和之前建立新的会话上下文。

建议在终端设备的电源循环①中保持会话状态。对于OTAA设备不这样做意味着激活过程将需要在设备的每个电源循环中执行。

①power cycling（电源循环）：将设备关闭，然后重新打开。又称为软重启、开关测试

# 6.2 空中激活

对于空中激活，终端设备必须在参与同网络服务器的数据交换之前遵循加入过程。每次丢失会话上下文信息时，终端设备都必须经历一个新的加入过程。

如上所述，加入过程需要在启动加入过程之前使用以下信息对终端设备进行个性化：DevEUI，JoinEUI，NwkKey和AppKey。

> 注意：对于空中激活，终端设备不会使用一对网络会话密钥进行个性化设置。相反，无论何时终端设备加入网络，都会派生出特定于该终端设备的网络会话密钥，以在网络级别加密和验证传输。这样便于在不同提供商的网络之间漫游终端设备。使用不同的网络会话密钥和应用会话密钥进一步允许联网的网络服务器，其中应用程序数据不能被网络提供商读取。

## 6.2.1 加入过程

从终端设备的角度来看，加入过程由**加入或重新加入 - 请求**和**加入 - 接受**交换组成。

## 6.2.2 加入请求消息

加入过程始终由终端设备通过发送加入请求消息来启动。

| Size（bytes） | 8 | 8 | 2 |
| :---: | :---: | :---: | :---: |
| Join-request | JoinEUI | DevEUI | DevNonce |

加入请求消息包含终端设备的**JoinEUI**和**DevEUI**，后跟2个字节（**DevNonce**）的随机数。

**DevNonce**是一个从0开始的计数器，当设备开始启动，并且随着每个加入请求而递增。对于给定的JoinEUI值，DevNonce值永远不能被重用。如果终端设备可以重新启动，那么DevNonce应该是持久化的（存储在非易失性内存中）。在不更改JoinEUI的情况下重置DevNonce将导致网络服务器丢弃设备的加入请求。对于每个终端设备，网络服务器会跟踪终端设备使用的最后一个DevNonce值，并且如果DevNonce未递增，则忽略加入请求。

> 注意：该机制通过发送先前记录的加入请求消息来防止重放攻击，意图将相应的终端设备从网络断开。只要网络服务器处理加入请求并生成加入接受帧，它就应该保留旧的安全上下文（密钥和计数器，如果有的话）和新的安全上下文，直到它接收到包含_**RekeyInd**_命令的第一个成功上行链路帧使用新的上下文，之后可以安全地删除旧的上下文。

加入请求消息的消息完整性代码（**MIC**）值（见第4章MAC消息描述）计算如下：①

_cmac_ = aes128\_cmac\(NwkKey, MHDR \| JoinEUI \| DevEUI \| DevNonce\)

MIC = _cmac_\[0..3\]

加入请求消息未加密。加入请求消息可以使用任何数据速率并且在指定的加入信道之间遵循随机跳频序列来传送。建议使用多种数据速率。**加入请求**传输之间的间隔应遵守第7章中描述的条件。对于加入请求的每次传输，终端设备应递增DevNonce值。

## 6.2.3 加入接受消息

如果终端设备被允许加入网络，网络服务器将使用**加入接受**消息来响应**加入**或**重新加入**请求消息。加入接受消息像普通下行链路一样发送，但使用延迟JOIN\_ACCEPT\_DELAY1或JOIN\_ACCEPT\_DELAY2（分别代替RECEIVE\_DELAY1和RECEIVE\_DELAY2）。这两个接收窗口使用的信道频率和数据速率与\[PHY\]的“接收窗口”部分中描述的RX1和RX2接收窗口使用的窗口相同。

如果加入请求未被接受，则没有响应给终端设备。

加入接受消息包含3个字节的服务器随机数（**JoinNonce**），网络标识符（**NetID**），终端设备地址（**DevAddr**），提供某些下行链路参数的（**DLSettings**）字段，TX和RX之间的延迟（**RxDelay**）以及终端设备加入的网络的可选网络参数列表（**CFList**）。可选的CFList字段是区域特定的，并在\[PHY\]中定义。

| Size（bytes） | 3 | 3 | 4 | 1 | 1 | （16）Optional |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Join-accept | JoinNonce | Home\_NetID | DevAddr | DLSettings | RxDelay | CFList |

**JoinNonce**是设备特定的计数器值（从不重复自身），由加入服务器提供并由终端设备用来派生会话密钥**FNwkSIntKey，SNwkSIntKey，NwkSEncKey**和**AppSKey**。JoinNonce会随着每个加入接受消息而递增。

设备跟踪上次成功处理的Join accept中使用的JoinNonce值（对应于上次成功的密钥派生）。只有当MIC字段正确且JoinNonce严格大于记录的字段时，设备才能接收加入接受。在这种情况下，新的JoinNonce值会替换先前存储的值。

如果设备易受电源循环影响，JoinNonce应持久保存（存储在非易失性内存中）。

LoRa联盟为所有网络分配一个24位唯一网络标识符（**NetID**），但以下的**NetID**值除外，这些值保留为不受管理的测试/专用网络。

有2^15个私有/测试网络保留NetID值如下构建：

| Nb bits | 3 | 14 | 7 |
| :--- | :--- | :--- | :--- |
|  | 3'b000 | XXXXXXXXXXXXXX 任意14位值 | 7'b0000000或7'b0000001 |

加入接受帧的**home\_NetID**字段对应于设备主网络的**NetId**。

在漫游场景中，分配devAddr和本地网络的网络可能会有所不同。要获得更高精度，请参阅\[BACKEND\]。

**DLsettings**字段包含下行链路配置：

| Bits | 7 | 6:4 | 3:0 |
| :---: | :---: | :---: | :---: |
| DLSettings | OptNeg | RX1DRoffset | RX2 Data rate |

OptNeg位指示网络服务器是否实现LoRaWAN1.0协议版本（未设置）或1.1和更高版本（设置）。当OptNeg位设置时

* 协议版本在终端设备和网络服务器之间进一步协商（1.1或更高版本）通过_RekeyInd_ / _RekeyConf_ MAC命令交换。
* 设备从**NwkKey**派生**FNwkSIntKey**＆**SNwkSIntKey**＆**NwkSEncKey**
* 设备从**AppKey**派生**AppSKey**

当OptNeg位未设置时

* 设备恢复到LoRaWAN1.0，不能协商任何选项
* _**RekeyInd**_命令不由设备发送的
* 设备从**NwkKey**派生**FNwkSIntKey**＆**AppSKey**
* 设备将**SNwkSIntKey**＆**NwkSEncKey**设置为等于**FNwkSIntKey**

这4个会话密钥**FNwkSIntKey**，**SNwkSIntKey**，**NwkSEncKey**和**AppSKey**派生如下：

如果OptNeg未设置，则会话密钥从NwkKey派生，如下所示：

AppSKey = aes128\_encrypt\(NwkKey, 0x02 \| JoinNonce \| NetID \| DevNonce \| pad16①\)

FNwkSIntKey = aes128\_encrypt\(NwkKey, 0x01 \| JoinNonce \| NetID \| DevNonce \| pad16\)

SNwkSIntKey = NwkSEncKey = FNwkSIntKey.

加入接受消息的MIC值计算如下：②

_cmac_ = aes128\_cmac\(**NwkKey**, MHDR \| JoinNonce \| NetID \| DevAddr \| DLSettings \| RxDelay \| CFList \)

MIC = _cmac_\[0..3\]

否则，如果OptNeg已设置，则AppSKey将从AppKey派生，如下所示：

AppSKey = aes128\_encrypt\(AppKey, 0x02 \| JoinNonce \| JoinEUI \| DevNonce \| pad16\)

且网络会话密钥来源于NwkKey：

FNwkSIntKey = aes128\_encrypt\(NwkKey, 0x01 \| JoinNonce \| JoinEUI \| DevNonce \| pad16 \)

SNwkSIntKey = aes128\_encrypt\(NwkKey, 0x03 \| JoinNonce \| JoinEUI \| DevNonce \| pad16\)

NwkSEncKey = aes128\_encrypt\(NwkKey, 0x04 \| JoinNonce \| JoinEUI \| DevNonce \| pad16\)

在这种情况下，MIC值计算如下：③

_cmac_ = aes128\_cmac\(**JSIntKey**, JoinReqType \| JoinEUI \| DevNonce \| MHDR \| JoinNonce \| NetID \| DevAddr \| DLSettings \| RxDelay \| CFList \)

MIC = _cmac_\[0..3\]

JoinReqType是一个单字节字段，用于编码触发加入接受响应的加入请求或重新加入请求类型。

| Join-request or Rejoin-request type | JoinReqType value |
| :--- | :--- |
| Join-request | 0xFF |
| Rejoin-request type 0 | 0x00 |
| Rejoin-request type 1 | 0x01 |
| Rejoin-request type 2 | 0x02 |

用于加密加入接受消息的密钥是触发它的加入或重新加入请求消息的函数。

| Triggering Join-request or Rejoin-request type | Join-accept Encryption Key |
| :--- | :--- |
| Join-request | **NwkKey** |
| Rejoin-request type 0 or 1 or 2 | **JSEncKey** |

加入接受消息按如下加密：

aes128\_decrypt\(**NwkKey** or **JSEncKey**, JoinNonce \| NetID \| DevAddr \| DLSettings \| RxDelay \| CFList \| MIC\).

①pad16函数附加零字节，以便数据的长度为16的倍数

②\[RFC4493\]

③\[RFC4493\]

消息长度为16或32个字节。

> 注意：ECB模式下的AES解密操作用于加密加入接受消息，以便终端设备可以使用AES加密操作解密消息。这样一个终端设备只需要实现AES加密而不是AES解密。
>
> 注意：建立这四个会话密钥允许联网的网络服务器基础结构，网络运营商不能窃听应用数据。应用提供商向网络运营商承诺，它将收取终端设备发生的任何流量的费用，并保留对用于保护其应用数据的AppSKey的完全控制权。
>
> 注意：设备的协议版本（1.0或1.1）在后端侧与DevEUI和设备的NWKKY和可能的AppKy同时注册。

RX1DRoffset字段设置上行数据速率与用于在第一个接收时隙（RX1）上与终端设备通信的下行数据速率之间的偏移量。默认情况下，此偏移量为0。偏移量用于考虑某些地区基站的最大功率密度限制，并平衡上行链路和下行链路无线链路余量。

上行链路和下行链路数据速率之间的实际关系是区域特定的，并在\[PHY\]中定义

延迟**RxDelay**遵循与_**RXTimingSetupReq**_命令中的延迟字段相同的约定。

如果在以下传输之后接收到加入接受消息：

* 加入请求或重新加入请求类型为0或1，如果CFlist字段不存在，则设备应该恢复到默认的信道定义。如果CFlist存在，它将覆盖当前定义的所有信道。由加入接受消息传输的MAC层参数（RXdelay1，RX2数据速率和RX1 DR Offset除外）应全部重置为其默认值。
* 重加入请求类型为2，如果CFlist字段不存在，设备应保持其当前信道定义不变。如果CFlist存在，它将覆盖当前定义的所有通道。所有其他MAC参数（复位的帧计数器除外）保持不变。

在成功处理加入接受消息后的所有情况下，设备都应该发送_**RekeyInd**_ MAC命令，直到它收到_**RekeyConf**_命令（见5.9）。_**RekeyInd**_上行链路命令的接收被网络服务器用作切换到新的安全上下文的信号。

## 6.2.4 重新加入请求消息

一旦激活，设备可以定期地在其正常应用流量之上发送重新加入请求消息。此重新加入请求消息定期为后端提供初始化终端设备新会话上下文的机会。为此，网络使用加入接受消息进行回复。这可以用于在两个网络之间交换设备，或者在给定网络上重新设置和/或更改设备的devAddr。

网络服务器还可以使用重新加入请求RX1 / RX2窗口来发送正常的已确认或未确认的下行链路帧，可选地携带MAC命令。在设备和网络服务器之间存在MAC层状态不同步的情况下，这种可能性对于重置设备的接收参数很有用。

示例：此机制可能用于更改设备的RX2窗口数据速率和RX1窗口数据速率偏移量，使用当前下行链路配置在下行链路中不再可达。

重新加入过程始终由终端设备通过发送重新加入请求消息来启动。

> 注意：任何时候，网络后端处理重新加入请求（类型为0，1或2）并生成加入接受消息时，它应保持旧的安全上下文（密钥和计数器，如果有的话）和新的，直到它收到第一个成功的上行链路帧，该帧使用新的上下文，之后可以安全地丢弃旧的上下文。在所有情况下，网络后端对重新加入请求消息的处理与标准加入请求消息的处理类似，因为网络服务器最初处理消息决定是否应将其转发到加入服务器以创建作为回应的加入接受消息。

有三种类型的重新加入请求消息可以由终端设备传输，并且对应于三个不同的目的。重新加入请求消息的第一个字节称为重新加入类型，用于对重新加入请求的类型进行编码。下表介绍了每个重新加入请求消息类型的用途。

| RejoinReqType | Content & Purpose |
| :---: | :--- |
| 0 | 包含NetID + DevEUI。用于重置包括所有无线参数（devAddr，会话密钥，帧计数器，无线电参数等）的设备上下文。此消息只能由接收网络服务器路由至设备的本地网络服务器，而不能路由至设备的入网服务器。此消息的MIC只能由服务或本地网络服务器验证。 |
| 1 | 包含JoinEUI + DevEUI。完全等同于初始加入请求消息，但可以在正常应用流量之上传输而不断开设备。只能由接收网络服务器路由到设备的入网服务器。用于恢复丢失的会话上下文（例如，网络服务器已丢失会话密钥，并且无法将设备关联到入网服务器）。只有入网服务器能够检查此消息的MIC。 |
| 2 | 包含NetID + DevEUI。用于重新设置设备或更改其DevAddr（DevAddr，会话密钥，帧计数器）。无线参数保持不变。此消息只能通过访问网络路由到设备的本地网络服务器，而不能路由到设备的入网服务器。此消息的MIC只能由服务或本地网络服务器验证。 |

### 6.2.4.1 重新加入请求类型为0或2的消息

| Size（bytes） | 1 | 3 | 8 | 2 |
| :---: | :---: | :---: | :---: | :---: |
| Rejoin-request | Rejoin Type = 0 or 2 | NetID | DevEUI | RJcount0 |

重新加入请求类型为0或2的消息包含**NetID**（设备的本地网络的标识符）和终端设备的**DevEUI**，后面跟着一个16位计数器（**RJcount0**）。

**RJcount0**是一个递增的计数器，每传输一次类型为0或2重新加入帧。每次加入接受被终端设备成功处理时，RJcount0被初始化为0。对于每个终端设备，网络服务器务必跟踪终端设备使用的最后一个RJcount0值（称为RJcount0\_last）。如果（Rjcount0 &lt;= RJcount0\_last）它忽略Rejoin请求。

RJcount0永不环绕。如果RJcount0达到2^16-1，设备应停止传送重新加入请求类型为0或2的帧。设备可以返回到加入状态。

> 注意：该机制通过发送先前记录的重新加入请求消息来防止重放攻击

重新加入请求消息的消息完整性代码（**MIC**）值（请参阅第4章MAC消息描述）计算如下：①

_cmac_ = aes128\_cmac\(SNwkSIntKey, MHDR \| Rejoin Type \| NetID \| DevEUI \| RJcount0\)

MIC = _cmac_\[0..3\]

重新加入请求消息未加密。

设备的**Rejoin-Req**类型为0或2的传输占空比应始终**&lt;0.1％**

> 注意：根据设备的使用情况，重新加入请求类型0的消息被发送从每小时一次到每隔几天一次。该消息也可以在ForceRejoinReq MAC命令后发送。该消息可用于在漫游情况下将移动设备重新连接到访问网络。它也可以用来重新设置或更改静态设备的devAddr。期望在网络之间漫游的移动设备应该比静态设备更频繁地发送该消息。
>
> 注意：重新加入请求类型2消息仅用于启用终端设备的密钥更新。此消息只能在ForceRejoinReq MAC命令后发送。

①\[RFC4493\]

### 6.2.4.2 重新加入请求类型为1的消息

类似于加入请求，重新加入请求类型为1的消息包含终端设备的JoinEUI和DevEUI。因此，重新加入请求类型为1的消息可以由任何接收它的网络服务器路由到终端设备的加入服务器。在网络服务器完全丢失状态的情况下，重新加入请求类型为1可用于恢复与终端设备的连接。建议每月至少发送一次重新加入请求类型为1的消息。

| Size（bytes） | 1 | 8 | 8 | 2 |
| :---: | :---: | :---: | :---: | :---: |
| Rejoin-request | Rejoin Type = 1 | JoinEUI | DevEUI | RJcount1 |

用于重新加入请求类型为1的RJcount1是与用于重新加入请求类型为0的RJCount0不同的计数器。

**RJcount1**是一个计数器，随着每发送一个重新加入请求类型为1的帧递增。对于每个终端设备，加入服务器会跟踪终端设备使用的最后一个**RJcount1**值（称为RJcount1\_last）。如果（Rjcount1 &lt;= RJcount1\_last），它将忽略Rejoin请求。

对于给定的JoinEUI，RJcount1不会环绕。重新加入请求类型为1的传输周期应该是这样的，以使得在给定的JoinEUI值的设备的生命周期内不能发生这种环绕。

> 注意：该机制通过发送先前记录的重新加入请求消息来防止重放攻击

重新加入请求类型为1的消息的消息完整性代码（**MIC**）值（见第4章MAC消息描述）计算如下：①

_cmac_ = aes128\_cmac\(JSIntKey, MHDR \| RejoinType \| JoinEUI\| DevEUI \| RJcount1\)

MIC = _cmac_\[0..3\]

重新加入请求类型为1的消息未加密。

设备的**Rejoin-Req**类型为1的传输占空比应始终**&lt;0.01％**

> 注意：重新加入请求类型为1的消息被发送从一天一次到每周一次。此消息仅用于服务器端完全丢失上下文的情况。此类事件是很可能延迟一天到一个星期来重新连接设备，也被认为是合适的

①\[RFC4493\]

### 6.2.4.3 重新加入请求传输

下表总结了每个重新加入请求类型消息传输的可能条件。

| RejoinReq type | 由终端设备自主和定期传输 | 在ForceRejoinReq MAC命令后传输 |
| :---: | :---: | :---: |
| 0 | x | x |
| 1 | x |  |
| 2 |  | x |

重新加入请求类型为0和1的消息应在随机跳频序列之后的任何定义的加入信道上传输（参见\[PHY\]）。

重新加入请求类型为2应当在随机跳频序列之后的任何当前启用的信道上传输。

在**ForceRejoinReq**命令后发送的重新加入请求类型为0或类型为2应使用MAC命令中指定的数据速率。

重新加入请求类型为0由终端设备周期性地自动发送（具有由RejoinParamSetupReq命令设置的最大周期性），重新加入请求类型为1应使用：

* 如果启用了ADR，则使用当前的数据速率和TX功率来传输应用有效负载
* 在加入信道上允许任何数据速率以及禁用ADR时的默认TX功率。在这种情况下，建议使用多种数据速率。

### 6.2.4.4 重新加入请求消息处理

对于所有3个重新加入请求类型，网络服务器可以响应：

* 如果想要修改设备的网络身份（漫游或重新设置），则为**加入接受**消息（如6.2.3中定义的那样）。在这种情况下，RJcount（0或1）替换密钥派生过程中的DevNonce
* 正常的下行链路帧可选地包含MAC命令。该下行链路应在相同的信道上发送，具有相同的数据速率和相同的延迟，与其替换的加入接受消息。

在多数情况下，在重新加入请求类型为0或1后，网络将不会响应。

## 6.2.5 密钥导出图

以下图表总结了设备连接到LoRaWAN1.0或1.1网络服务器的情况下的密钥派生方案。

**LoRaWAN1.0网络后端：**

当LoRaWAN1.1设备配置了LoRaWAN1.0.X网络后端时，所有密钥都来自**NwkKey**根密钥。该设备的**AppKey**未被使用。

![](/assets/Figure 48 : LoRaWAN1.0 key derivation scheme.png)

**LoRaWAN1.0网络后端：**

![](/assets/Figure 49 : LoRaWAN1.1 key derivation scheme.png)

# 6.3 个性化激活

个性化激活可以通过**加入请求** - **加入接受**过程直接将终端设备绑定到特定的网络。

通过个性化激活终端设备意味着**DevAddr**和四个会话密钥**FNwkSIntKey**，**SNwkSIntKey**，**NwkSEncKey**和**AppSKey**直接存储在终端设备中，而不是在加入过程中从**DevEUI**，**JoinEUI**，**AppKey**和**NwkKey**派生。终端设备一旦启动就配备了参与特定LoRa网络所需的信息。

每个设备应具有唯一的一组**F/SNwkSIntKey**，**NwkSEncKey**和**AppSKey**。妥协一个设备的密钥不得危及其他设备通信的安全。构建这些密钥的过程应该使得密钥不能从公共可用的信息（例如节点地址或终端设备的devEUI）中派生出来。

当个性化终端设备首次访问网络或重新初始化后，它应在所有上行链路消息的FOpt字段中传输ResetInd MAC命令，直到它从网络接收到一个ResetConf命令。重新初始化后，终端设备必须使用其默认配置（即设备首次连接到网络时使用的配置）。

> 注意：在CCM\*操作模式下，相同密钥的所有调用中，帧计数器值只能使用一次。因此，禁止重新初始化ABP终端设备帧计数器。ABP设备必须使用非易失性内存来存储帧计数器。
>
> ABP设备在其整个生命周期中使用相同的会话密钥（即密钥更新是不可能的），因此推荐将OTAA设备用于更高级别的安全应用。



