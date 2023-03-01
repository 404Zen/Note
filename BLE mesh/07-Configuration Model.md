Configuration model 属于 mesh 中的基础模型之一，其中包含了 Server 和 Client 两种类型，这两个模型，在节点中都是强制要求拥有的模型。

# Configuration Server Model
- 该模型是一个根模型，只存在于首要元素中并且每个节点中有且仅有一个。
- 该模型主要用于表示设备加入mesh网络之后的mesh网络配置；mesh网络配置存放了节点的配置内容，如是否支持中继，是否支持朋友特性等等以及配置相对应的内容。
- 如果应用层想要获取或配置该模型的内容，则需要通过 **设备密钥(device key)** 才能得到正确的数据内容。
- 该模型是一个SIG定义的标准模型，其 SIG Model ID 为 0x0000
![[Pasted image 20230218132016.png]]

<table class="tg" style="undefined;table-layout: fixed; width: 714px">
<colgroup>
<col style="width: 277px">
<col style="width: 92px">
<col style="width: 139px">
<col style="width: 115px">
<col style="width: 91px">
</colgroup>
  <tr>
    <th class="tg-0pky" colspan="2">Configuration Server States</th>
    <th class="tg-0pky" colspan="3">Bound States</th>
  </tr>
  <tr>
    <td class="tg-btxf">State</td>
    <td class="tg-btxf">Instance</td>
    <td class="tg-btxf">Model</td>
    <td class="tg-btxf">State</td>
    <td class="tg-btxf">Instance</td>
  </tr>
  <tr>
    <td class="tg-0pky">Secure Network Beacon</td>
    <td class="tg-0pky">Primary</td>
    <td class="tg-0pky">-</td>
    <td class="tg-0pky">-</td>
    <td class="tg-0pky">-</td>
  </tr>
  <tr>
    <td class="tg-btxf">Composition Data</td>
    <td class="tg-btxf">Primary</td>
    <td class="tg-btxf">-</td>
    <td class="tg-btxf">-</td>
    <td class="tg-btxf">-</td>
  </tr>
  <tr>
    <td class="tg-0pky">Default TTL</td>
    <td class="tg-0pky">Primary</td>
    <td class="tg-0pky">-</td>
    <td class="tg-0pky">-</td>
    <td class="tg-0pky">-</td>
  </tr>
  <tr>
    <td class="tg-btxf">GATT Proxy</td>
    <td class="tg-btxf">Primary</td>
    <td class="tg-btxf">Configuration Server</td>
    <td class="tg-btxf">Node Identity</td>
    <td class="tg-btxf">Primary</td>
  </tr>
  <tr>
    <td class="tg-0pky">Friend</td>
    <td class="tg-0pky">Primary</td>
    <td class="tg-0pky">-</td>
    <td class="tg-0pky">-</td>
    <td class="tg-0pky">-</td>
  </tr>
  <tr>
    <td class="tg-btxf">Relay</td>
    <td class="tg-btxf">Primary</td>
    <td class="tg-btxf">-</td>
    <td class="tg-btxf">-</td>
    <td class="tg-btxf">-</td>
  </tr>
  <tr>
    <td class="tg-0lax">Model Publication</td>
    <td class="tg-0lax">Primary</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
  </tr>
  <tr>
    <td class="tg-buh4">Subscription List</td>
    <td class="tg-buh4">Primary</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
  </tr>
  <tr>
    <td class="tg-0lax">NetKey List</td>
    <td class="tg-0lax">Primary</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
  </tr>
  <tr>
    <td class="tg-buh4">AppKey List</td>
    <td class="tg-buh4">Primary</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
  </tr>
  <tr>
    <td class="tg-0lax">Model to AppKey List</td>
    <td class="tg-0lax">Primary</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
  </tr>
  <tr>
    <td class="tg-buh4">Node Identity</td>
    <td class="tg-buh4">Primary</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
  </tr>
  <tr>
    <td class="tg-0lax">Key Refresh Phase</td>
    <td class="tg-0lax">Primary</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
  </tr>
  <tr>
    <td class="tg-buh4">Heartbeat Publish</td>
    <td class="tg-buh4">Primary</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
  </tr>
  <tr>
    <td class="tg-0lax">Heartbeat Subscription</td>
    <td class="tg-0lax">Primary</td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
  </tr>
  <tr>
    <td class="tg-buh4">Network Transmit</td>
    <td class="tg-0lax">Primary</td>
    <td class="tg-buh4"></td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
    <td class="tg-buh4">-</td>
  </tr>
  <tr>
    <td class="tg-0lax">Relay Retransmit</td>
    <td class="tg-0lax">Primary</td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
    <td class="tg-0lax">-</td>
  </tr>
</table>

从上表可知，洋洋洒洒共17项内容；可能有读者会发现有一个“腰间突出”的，即**GATT Proxy**这项内容；对于这个不知道大家还有没有印象：

-   mesh provisioning server
    当设备还没有入网前，mesh gatt server就叫mesh provisioning server；其主要是让provisioning client配置provisioning server，让其加入mesh网络
    
-   mesh proxy server
    当设备已经入网了，则为mesh proxy server；其主要作用是接收或者发送proxy pdus从或者到client
    
而这个**GATT Proxy**与**Node Identity**的关系就是，如果proxy feature被支持且mesh proxy server被公开。在入网完成之后，就会将在广播包中携带有**Node Identity**这个域值。

既然**Configuration Server Model**存放的是节点的配置内容，那用户应该如何去获取这个内容呢？具体如下所示：
<table class="tg" style="undefined;table-layout: fixed; width: 918px">
<colgroup>
<col style="width: 92px">
<col style="width: 108px">
<col style="width: 212px">
<col style="width: 282px">
<col style="width: 114px">
<col style="width: 110px">
</colgroup>
  <tr>
    <th class="tg-0pky">Element</th>
    <th class="tg-0pky">SIG Model ID</th>
    <th class="tg-0pky">States</th>
    <th class="tg-0pky">Messages</th>
    <th class="tg-0pky">Rx</th>
    <th class="tg-0pky">Tx</th>
  </tr>
  <tr>
    <td class="tg-kyy7" rowspan="69">Primary</td>
    <td class="tg-kyy7" rowspan="69">0x0000<br></td>
    <td class="tg-kyy7" rowspan="3">Secure Network Beacon<br></td>
    <td class="tg-btxf">Config Beacon Get</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Beacon Set<br></td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Beacon Status</td>
    <td class="tg-btxf">-</td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="2">Composition Data</td>
    <td class="tg-0pky">Config Composition Data Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Composition Data Status</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="3">Default TTL</td>
    <td class="tg-0pky">Config Default TTL Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Default TTL Set</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Default TTL Status</td>
    <td class="tg-0pky">-</td>
    <td class="tg-0pky">M</td>
  </tr>
  <tr>
    <td class="tg-kyy7" rowspan="3">GATT Proxy</td>
    <td class="tg-btxf">Config GATT Proxy Get</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config GATT Proxy Set</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config GATT Proxy Status</td>
    <td class="tg-btxf">-</td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="3">Friend</td>
    <td class="tg-0pky">Config Friend Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Friend Set</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Friend Status</td>
    <td class="tg-0pky">-</td>
    <td class="tg-0pky">M</td>
  </tr>
  <tr>
    <td class="tg-kyy7" rowspan="3">Relay and Relay Retransmit</td>
    <td class="tg-btxf">Config Relay Get</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Relay Set</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Relay Status</td>
    <td class="tg-btxf">-</td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="4">Model Publication</td>
    <td class="tg-0pky">Config Model Publication Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Model Publication Set</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Model Publication Virtual Address Set</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Model Publication Status</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="12">Subscription List</td>
    <td class="tg-0pky">Config Model Subscription Add</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Model Subscription Virtual Address Add</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Model Subscription Delete</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Model Subscription Virtual <br>Address Delete</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Model Subscription Virtual <br>Address Overwrite</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Model Subscription <br>Overwrite</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Model Subscription Delete<br>All</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Model Subscription Status</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-0pky">Config SIG Model Subscription Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config SIG Model Subscription List</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Vendor Model Subscription Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Vendor Model Subscription List</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="6">NetKey List</td>
    <td class="tg-0pky">Config NetKey Add</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config NetKey Update</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config NetKey Delete</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config NetKey Status</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-0pky">Config NetKey Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config NetKey List</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="6">AppKey List</td>
    <td class="tg-0pky">Config AppKey Add</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config AppKey Update</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config AppKey Delete</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config AppKey Status</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-0pky">Config AppKey Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config AppKey List</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="7">Model to AppKey List</td>
    <td class="tg-0pky">Config Model App Bind</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Model App Unbind</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Model App Status</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky">M</td>
  </tr>
  <tr>
    <td class="tg-btxf">Config SIG Model App Get</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config SIG Model App List</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky">M</td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Vendor Model App Get</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Vendor Model App List</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky">M</td>
  </tr>
  <tr>
    <td class="tg-kyy7" rowspan="3">Node Identity</td>
    <td class="tg-btxf">Config Node Identity Get</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Node Identity Set</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Node Identity Status</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="2">N/A</td>
    <td class="tg-0pky">Config Node Reset</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Node Reset Status</td>
    <td class="tg-btxf"></td>
    <td class="tg-btxf">M</td>
  </tr>
  <tr>
    <td class="tg-9wq8" rowspan="3">Key Refresh Phase</td>
    <td class="tg-0pky">Config Key Refresh Phase Get</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-btxf">Config Key Refresh Phase Set</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Key Refresh Phase Status</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky">M</td>
  </tr>
  <tr>
    <td class="tg-kyy7" rowspan="3">Heartbeat Publication</td>
    <td class="tg-btxf">Config Heartbeat Publication Get</td>
    <td class="tg-btxf">M</td>
    <td class="tg-btxf"></td>
  </tr>
  <tr>
    <td class="tg-0pky">Config Heartbeat Publication Set</td>
    <td class="tg-0pky">M</td>
    <td class="tg-0pky"></td>
  </tr>
  <tr>
    <td class="tg-buh4">Config Heartbeat Publication Status</td>
    <td class="tg-buh4"></td>
    <td class="tg-buh4">M</td>
  </tr>
  <tr>
    <td class="tg-nrix" rowspan="3">Heartbeat Subscription</td>
    <td class="tg-0lax">Config Heartbeat Subscription Get</td>
    <td class="tg-0lax">M</td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-buh4">Config Heartbeat Subscription Set</td>
    <td class="tg-buh4">M</td>
    <td class="tg-buh4"></td>
  </tr>
  <tr>
    <td class="tg-0lax">Config Heartbeat Subscription Status</td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">M</td>
  </tr>
  <tr>
    <td class="tg-57iy" rowspan="3">Network Transmit</td>
    <td class="tg-buh4">Config Network Transmit Get</td>
    <td class="tg-buh4">M</td>
    <td class="tg-buh4"></td>
  </tr>
  <tr>
    <td class="tg-0lax">Config Network Transmit Set</td>
    <td class="tg-0lax">M</td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-buh4">Config Network Transmit Status</td>
    <td class="tg-buh4"></td>
    <td class="tg-buh4">M</td>
  </tr>
</table>


从上表可以看到，一个State对应多个Message；那么，State与Message的关系就类似于 **“手机现在都支持BLE，那么就有打开BLE、关闭BLE以及查看当前BLE的状态的动作”**；其中，手机支持BLE就等同于State，而那些动作就是Message。所以，如果你想要获取或者配置相关的内容，就可以通过Message去实现。对于，**RX**和**TX**是相对于节点本身为参考体。这里举个例子 **“就Network Transmit状态而言，Config Network Transmit Get/Set表示该节点可以接收这些Messages；而节点在接收到这些消息之后，就会返回Config Network Transmit Status即发送回给源地址的设备”**；那么，紧接着我们继续解析每一个State以及其对应的Message的作用。


## Secure Network Beacon
该状态用于表示节点是否在周期性广播Secure Network Beacon，这个比较简单好理解；其具体的状态如下表所示：
| Values    | Description                                          |
| --------- | ---------------------------------------------------- |
| 0x00      | The node is not broadcasting a Secure Network beacon |
| 0x01      | The node is    broadcasting a Secure Network beacon  |
| 0x02–0xFF | Prohibited                                           |

## Composition Data
这个状态包含了节点的信息，如其元素以及模型的信息。这些信息可以由多个信息页组成；但是目前SIG只规定了Page0的内容，其他信息页内容是可选的。那些，SIG规定的Composition Data Page0有哪些内容呢？我想信很多一直看我们[PB-GATT入网过程](05-PB_GATT入网过程.md)教程的读者应该对这些内容比较熟悉。没错！这些就是节点的元素、模型以及其所支持的特性等信息，入网成功之后Provisioner向节点获取得到的内容；
![[Pasted image 20230218133520.png]]


| Field                 | Size(octets) | Notes                                                                                                                 |
| --------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------- |
| [CID](#CID)           | 2            | Contains a 16-bit company identifier assigned by the Bluetooth SIG                                                    |
| [PID](#PID)           | 2            | Contains a 16-bit vendor-assigned product identifierVID2 Contains a 16-bit vendor-assigned product version identifier |
| [VID](#VID)           | 2            | Contains a 16-bit vendor-assigned product version identifier                                                          |
| [CRPL](#CRPL)         | 2            | Contains a 16-bit value representing the minimum number of replay protection list entries in a device                 |
| [Features](#Features) | 2            | Contains a bit field indicating the device features                                                                   |
| [Elements](#Elements) | variable     | Contains a sequence of element descriptions                                                                           |

- CID: Company ID
- PID: Product ID
- VID: Version ID
- CRPL: 这个域值表明用于重播攻击保护 **(replay attack protection)** 的Relay cache List的大小，即最多可以存放多少个relay cache
- Features: 表示节点支持哪些特性， Relay, Friend, Proxy, Low Power
- Elements
    该域表示当前这个元素具备有多少个SIG Model和Vendor Model、具体是哪几个模型；该域有如下几个字段：
    -   Loc
        该域为位置描述符的意思，主要起到加固上下文描述的作用；如 **“我们有一个采用了BLE Mesh技术的排插，并且该排插有3个插座分别对应着三个Element，那么这个时候该域就可以配置为是哪个插座，例如：1、2、3”** 具体可以为哪些值，请参考SIG的[GATT Namespace Descriptors](https://www.bluetooth.com/specifications/assigned-numbers/gatt-namespace-descriptors/)
        
    -   NumS
        这个域就相对比较好理解了，S = SIG Model；表示该元素下有多少个SIG Model
        
    -   NumV
        同理，V = Vendor；表示该元素下有多少个Vendor Model
        
    -   SIG Models/Vendor Models
        这个就更加好理解了，表明当前这个元素下SIG/Vendor Models的ID号是什么


## Default TTL
表示当前节点发送消息时的默认TTL的值。

## GATT Proxy/Freind/Relay & Relay Retransmit
表示当前节点是否支持某一特性; 如果支持，可以设置该特性的开启与关闭。


## Model Publication
该状态覆盖了Model发送消息的方方面面。
- Publish Address
	模型发布时的目标地址。
- Publish Period
	模型发布Status消息时的时间间隔。
- Publish AppKey Index
	表示模型发送消息时所用到的AppKey的索引。
- Publish Frinedship Credential Flag
	用于控制模型发布消息时使用哪种安全素材。
- Publish TTL
	定义了模型发布Status消息时的TTL值。
- Publish Retransmission Count
- Publish Retransmit Interval Steps



## Subscription List
订阅列表，订阅列表是组地址或者虚拟地址的集合。

## NetKey List
该状态表示网络密钥的索引列表，每个入口最多只能容纳两个NetKey，一个是旧的密钥，另外一个是新的密钥。
小编一直都在强调，由于NetKey的长度一般都很长，将其塞入网络消息中不太实现；然而，使用一个索引值就可以找到相对应的NetKey的方式无疑是最佳的。一些消息可能有一个或者多个索引值，比如说：**节点使用“Config NetKey Add”添加了多个NetKey，现在通过“Config NetKey Get”获取当前的NetKey索引值，这个时候“Config NetKey List”就会携带多个NetKey索引值了**。然而，每个索引值的长度均为12Bits，不是8的倍数；这个时候就需要根据索引值个数的奇偶来重新组成这个索引值了，具体的实现方式如下所示：
![[Pasted image 20230218135906.png]]

## AppKey List

该State跟上述的[NetKey List](#NetKey-List)的描述基本上是一致的，只不过这个是AppKey的索引值列表；同样，每个AppKey列表的条目都只包括一个索引值以及最大两个AppKey值，其中一个是旧的AppKey，另外一个是新的AppKey。其中AppKey Index与NetKey Index是一样的，它们都是12Bits。

## Model to AppKey List
该状态用于阐述模型与AppKey的关系，也就是说模型绑定了哪些AppKey，怎么解绑，怎么获取绑定的内容等等。


## Node Identity
该状态的作用是配置是否使用Node Identity来广播 **（可连接的非定向广播）**，其主要目的是当proxy node的Proxy Service在广播包中被暴露出来，可以根据proxy节点的首要元素的单播地址和该节点所属子网的网络密钥，来快速锁定想要连接的哪个proxy节点，尤其是周边有大量的节点的时候.


## Config Node Reset
该消息就是将当前节点从Mesh网络中移除出去，其应答的消息为[Config Node Reset Status](#Config-Node-Reset-Status)。该消息没有携带任何参数，所以除了Opcode也就没什么其他的帧格式了。复位之后，Provisioner可以重复使用该节点的所有单播地址，但是还要重复使用Sequence Numbers则需要等IV Index更新之后 **（比删除前的IV Index值大就行）** 才能使用这些单播地址。

## Key Refresh Phase
该状态用于节点的NetKey列表中的NetKey以及其相对应绑定的AppKey的刷新


## Heartbeat Publication
该状态跟上述的[Model Publication](#Model-Publication)有些类似，只是该状态用于控制心跳消息周期地发送以及发送相关内容的配置；


## Network Transmit
该状态与上述的[Relay Retransmit](#relay_retransmit)极其相似，只不过该状态用于配置节点发送NetWork PDU时的次数以及相对应的时间间隔



# Configuration Client Model
该模型跟[Configuration Server Model](#Configuration-Server-Model)是一样的：
-   首先，该模型是一个根模型，只存在于首要元素中并且每个节点中有且仅有一个
-   该模型主要用于控制或者监控节点的配置
-   如果应用层想要获取或者配置对端节点的配置内容时，则需要通过**设备秘钥**才能得到正确的数据内容
-   该模型是SIG的一个标准模型，其SIG Model ID为0x0001
至于，该模型对应的状态和消息；小编在[Configuration Server Model](#Configuration-Server-Model)中已经详述，无非就是Client扮演的是配置Server的角色，而Server扮演的是接收Client的配置命令的角色；同样的消息在client端则是发往server，在server端则变成接收client的设置命令；基本上拥有配置Client模型的都是Provisioner，而Server模型的则是节点。它们两者的数据交互拓扑图如下所示：

![[Pasted image 20230218140558.png]]


**注意：同时支持Client模型和Server模型的叫控制端模型**


![[Pasted image 20230218140623.png]]