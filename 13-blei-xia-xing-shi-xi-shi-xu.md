# 13.1定义

要在B类中成功运行，终端设备必须在相对于基础设施信标的精确时刻打开接收插槽。本节定义了所需的时间。

两个连续信标开始之间的间隔称为信标周期。信标帧传输与BEACON\_RESERVED时间间隔的开始对齐。每个信标之前都有一个保护时间间隔，在该时间间隔内不能放置ping时隙。保护间隔的长度对应于最长允许帧的空中时间。这是为了确保在保护时间之前的ping时隙期间发起的下行链路总是有时间完成而不与信标传输冲突。因此，ping时隙的可用时间间隔从信标保留时间间隔的结束延伸到下一个信标保护间隔的开始。

![](/assets/Figure 53: Beacon timing.png)

| Beacon\_period | 128s |
| :--- | :--- |
| Beacon\_reserved | 2.120 s |
| Beacon\_guard | 3.000 s |
| Beacon-window | 122.880 s |

空中的信标帧时间实际上比信标保留时间间隔短得多，以允许将来添加网络管理广播帧。

信标窗口时间间隔分为2^12 = 4096个30ms的ping时隙，每个时隙编号从0到4095。

使用时隙编号N的终端设备必须在信标启动后的_Ton_秒钟准确开启其接收器，其中：

_Ton_ = _beacon\_reserved_ + _N_ \* 30 ms

N被称为_时隙索引_。

最新的ping时隙从信标开始后的_beacon\_reserved_ + 4095 \* 30 ms = 124 970 ms开始或在下一个beacon开始之前3030 ms开始。

# 13.2 时隙随机化

为了避免系统冲突或串听问题，时隙索引随机化，并在每个信标周期进行更改。

使用以下参数：

| DevAddr | 设备32位网络单播或组播地址 |
| :--- | :--- |
| _pingNb_ | 每个信标周期的ping时隙数。这必须是2整数的幂：pingNb = 2^k，其中0 &lt;= k &lt;= 7 |
| _pingPeriod_ | 设备接收器唤醒周期以时隙数表示：_pingPeriod_ = 2^12 / _pingNb_ |
| _pingOffset_ | 在每个信标周期开始时计算随机偏移量。值的范围可以从0到（pingPeriod-1） |
| _beaconTime_ | 在前一个信标帧的字段**BCNPayload**.Time中携带的时间 |
| _slotLen_ | 单元ping时隙的长度= 30 ms |

在每个信标周期，终端设备和服务器计算一个新的伪随机偏移量以对齐接收时隙。使用全零的固定密钥进行AES加密以随机化：

_Key_ = 16 x 0x00

_Rand_ = aes128\_encrypt\(Key, beaconTime \| DevAddr \| pad16\)

_pingOffset_ = \(_Rand_\[0\] + _Rand_\[1\]x 256\) modulo _pingPeriod_

用于此信标时段的时隙将为：

_pingOffset_ + _N_ x _pingPeriod_ with _N_=\[0:_pingNb_-1\]

节点因此打开接收时隙，开始于：

| First slot | Beacon\_reserved + pingOffset x slotLen |
| :--- | :--- |
| slot 2 | Beacon\_reserved + \(pingOffset + pingPeriod\) x slotLen |
| slot 3 | Beacon\_reserved + \(pingOffset + 2 x pingPeriod\) x slotLen |
| ... | ... |

如果终端设备同时服务于单播和一个或多个组播时隙，则在新的信标周期开始时多次执行该计算。一次为单播地址（节点网络地址），一次为每个组播组地址。

在组播ping时隙和单播ping时隙发生冲突且不能由终端设备接收器提供服务的情况下，终端设备应该优先监听组播时隙。如果在组播接收时隙之间存在冲突，则可以使用先前组播帧的FPending位来设置首选项。

随机化方案防止单播和组播时隙之间的系统冲突。如果在信标周期内发生冲突，则在下一个信标周期内不太可能再次发生。























