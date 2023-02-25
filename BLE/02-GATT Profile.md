
GATT是Generic ATTribute Profile的缩写；GATT 定义了两个低功耗蓝牙设备间所使用的称之为**服务(Service)** 和 **特性(Characteristics)** 的数据传输方式，称之为 **属性协议(Attribute Protocol-ATT)**， 这个协议将 服务、特性和相关的数据存储在一个简单的查找表中，该表中的每个条目使用16位的UUID。

# GAP
详细介绍 GATT 之前，需要了解 GAP（Generic Access Profile），它在用来控制设备**连接和广播**。GAP 使你的设备被其他设备可见，并决定了你的设备是否可以或者怎样与合同设备进行交互。例如 Beacon 设备就只是向外广播，不支持连接，小米手环就等设备就可以与中心设备连接。

## 设备角色
GAP给设备定义了若干个角色，其中主要的两个是，外围设备(Periphral)和中心设备(Central)。

Periphral：用于提供数据的设备，例如传感器等
Central：功能相对更强大，用于连接外围设备，例如手机等。

```

```

## 广播数据
在GAP中外围设备通过两种方式向外广播数据，Adv