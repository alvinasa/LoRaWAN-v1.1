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



