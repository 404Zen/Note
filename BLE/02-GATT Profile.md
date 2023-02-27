
GATT是Generic ATTribute Profile的缩写；GATT 定义了两个低功耗蓝牙设备间所使用的称之为**服务(Service)** 和 **特性(Characteristics)** 的数据传输方式，称之为 **属性协议(Attribute Protocol-ATT)**， 这个协议将 服务、特性和相关的数据存储在一个简单的查找表中，该表中的每个条目使用16位的UUID。

# GAP
 GAP（Generic Access Profile）用来控制设备**连接和广播**。GAP 使你的设备被其他设备可见，并决定了你的设备是否可以或者怎样与合同设备进行交互。例如 Beacon 设备就只是向外广播，不支持连接，小米手环就等设备就可以与中心设备连接。

## 设备角色
GAP给设备定义了若干个角色，其中主要的两个是，外围设备(Periphral)和中心设备(Central)。

Periphral：用于提供数据的设备，例如传感器等
Central：功能相对更强大，用于连接外围设备，例如手机等。

在CH58x的Periphral例程中，初始化时调用了这个函数`GAPRole_PeripheralInit();`将设备初始化为Periphral角色。

## 广播数据
在GAP中外围设备通过两种方式向外广播数据，Advertising Data Payload（广播数据）和 Scan Response Data Payload（扫描回复），每种数据最长可以包含 31 byte。Periphral设备需要不断地向外广播，让Central设备知道他的存在。Scan Response则是可选的，因为Scan Response一般携带的是额外信息。
![[Pasted image 20230225184035.png]]
广播的流程大致如上，Periphral设备以一个时间间隔发送广播数据，当收到扫描请求的时候，则发送扫描回复。这个事件间隔在初始化的时候进行设定，时间间隔越长，设备功耗越低，但也更难被中心设备发现。

CH58x通过以下函数设置广播时间间隔
```
uint16_t advInt = DEFAULT_ADVERTISING_INTERVAL; //默认为50ms

// Set advertising interval
GAP_SetParamValue(TGAP_DISC_ADV_INT_MIN, advInt);
GAP_SetParamValue(TGAP_DISC_ADV_INT_MAX, advInt);
```

## 广播的网络拓扑结构
大部分情况下，外设通过广播自己来让中心设备发现自己，并建立 GATT 连接，从而进行更多的数据交换。也有些情况是不需要连接的，只要外设广播自己的数据即可。用这种方式主要目的是让外围设备，把自己的信息发送给多个中心设备。因为基于 GATT 连接的方式的，只能是一个外设连接一个中心设备。 使用广播这种方式最典型的应用就是苹果的 iBeacon。广播工作模式下的网络拓扑图如下:
![[Pasted image 20230225184547.png]]


# GATT
GATT 的全名是 Generic Attribute Profile（通用属性协议，基本属性协议），它定义了 两个 BLE 设备互相传输数据进行通信的方法，该方法用了两个概念， Service 和 Characteristic 。GATT 使用 ATT（Attribute Protocol）协议，ATT 协议把 Service, Characteristic以及相关的数据存储在一个简单的有索引的表中，这个索引表使用 16 位的 ID 作为表中每一项条目的索引。

一旦两个设备建立起了连接，GATT 就开始起作用了，这也意味着，你必需已经完成了前面的 GAP 协议管理的广播过程。  

GATT 连接需要特别注意的是：GATT 连接是独占的。也就是一个 BLE 外设同时只能被一个中心设备连接。一旦外设被连接，它就会马上停止广播，这样它就对其他设备不可见了。当设备断开，它又开始广播。

中心设备和外设需要双向通信的话，唯一的方式就是建立 GATT 连接。

## GATT网络连接拓扑
下图展示了 GTT 连接网络拓扑结构。这里很清楚的显示，一个外设只能连接一个中心设备，而一个中心设备可以连接多个外设。
![[Pasted image 20230225184826.png]]
一旦建立起了连接，通信就是双向的了，对比前面的 GAP 广播的网络拓扑，GAP 通信是单向的。如果你要让两个设备外设能通信，就只能通过中心设备中转。

## GATT通信事务
GATT 通信的双方是 C/S 关系。外设作为 GATT 服务端（Server），它维持了 ATT 的查找表以及 service 和 characteristic 的定义。中心设备是 GATT 客户端（Client），它向 Server 发起请求。需要注意的是，所有的通信事件，都是由客户端（也叫主设备，Master）发起，并且接收服务端（也叫从设备，Slave）的响应。

一旦连接建立，外设将会给中心设备建议一个连接间隔（Connection Interval）,这样，中心设备就会在每个连接间隔尝试去重新连接，检查是否有新的数据。但是，这个连接间隔只是一个建议，你的中心设备可能并不会严格按照这个间隔来执行，例如你的中心设备正在忙于连接其他的外设，或者中心设备资源太忙。

下图展示一个外设（GATT 服务端）和中心设备（GATT 客户端）之间的数据交换流程，可以看到的是，每次都是主设备发起请求：
![[Pasted image 20230225185018.png]]

## GATT结构
GATT 事务是建立在嵌套的Profiles, Services 和 Characteristics之上的的，如下图所示：![[Pasted image 20230225185050.png]]
Profile Profile 并不是实际存在于 BLE 外设上的，它只是一个被 Bluetooth SIG 或者外设设计者预先定义的 Service 的集合。例如心率Profile（Heart Rate Profile）就是结合了 Heart Rate Service 和 Device Information Service。所有官方通过 GATT Profile 的列表可以从这里找到。

Service Service 是把数据分成一个个的独立逻辑项，它包含一个或者多个 Characteristic。每个 Service 有一个 UUID 唯一标识。 UUID 有 16 bit 的，或者 128 bit 的。16 bit 的 UUID 是官方通过认证的，需要花钱购买，128 bit 是自定义的，这个就可以自己随便设置。

Characteristic 在 GATT 事务中的最低界别的是 Characteristic，Characteristic 是最小的逻辑数据单元，当然它可能包含一个组关联的数据，例如加速度计的 X/Y/Z 三轴值。

与 Service 类似，每个 Characteristic 用 16 bit 或者 128 bit 的 UUID 唯一标识。你可以免费使用 Bluetooth SIG 官方定义的标准 Characteristic，使用官方定义的，可以确保 BLE 的软件和硬件能相互理解。当然，你可以自定义 Characteristic，这样的话，就只有你自己的软件和外设能够相互理解。

实际上，和 BLE 外设打交道，主要是通过 Characteristic。你可以从 Characteristic 读取数据，也可以往 Characteristic 写数据。这样就实现了双向的通信。所以你可以自己实现一个类似串口（UART）的 Sevice，这个 Service 中包含两个 Characteristic，一个被配置只读的通道（RX），另一个配置为只写的通道（TX）。


## ATT protocol
ATT（Attribute） protocol为所有基于LE link的应用提供了一个底层的框架。它定义了server与client，定义了属性以及client如何获取server端的一系列属性。Generic Attribute Profile作为一个通用的基于ATT的profile，为上层应用提供了一个基本的服务框架（service framework），使得所有基于LE的应用都可以将自身的功能映射到这个框架中来。


相比ATT protocol，GATT Profile定义了以下更为具体的概念：
-   一组通用的Attribute Type，如Primary Service、Characteristic；
-   以上通用Attribute Type的Attribute Value的格式，不是所有的Attribute Value都只有一个UUID；
-   如何使用ATT PDU对这组通用的Attribute进行读写、查询、配置；
-   上层应用要如何基于GATT定义自己的Service

### Attribute type
**参考 《Core Spec V5.3》 , Vol 3: Host -> Part G: Generic Attribute Profile (GATT) -> 3 Service interoperability requirements**

GATT定义了以下通用的Attribute Type：
| Attribute type                      | UUID   | Description                                                                                                                                                                                  |
| ----------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Primary Service                     | 0x2800 | 基础服务，通常包含 Include 以及 Characteristic 来描述他的具体特性                                                                                                                            |
| Secondary Service                   | 0x2801 | 二级服务                                                                                                                                                                                     |
| Include                             | 0x2802 | 通常被包含在一个Primary Service中，表示当前的Service用到了这个Include Service 的 Attribute handle, End Group Handle. 如果这个Include Service的UUID是SIG定义的16bit UUID,则还包含这个UUID的值 |
| Characteristic                      | 0x2803 | 服务的特性，包含Property和Value,以及0或多个Descriptor(↓↓↓下面都是Descriptor↓↓↓), Descriptor是用来描述Characteristic的                                                                        |
| Characteristic Extended Properties  | 0x2900 | 扩展属性Descriptor, Characteristic本身只有一个字节来描述它的property，更多属性将在这个Descriptor中说明                                                                                       |
| Characteristic User Descriptor      | 0x2901 | 对某个Characteristic的描述，是一个UTF-8格式的字符串                                                                                                                                          |
| Client Characteristic Configuration | 0x2902 | GATT Client可以通过这个Descriptor, 来使能对应Characteristic的Notifiction或者Indication, 不同的GATT Client有各自独立的Client Characteristic Configuration                                     |
| Server Characteristic Configuration | 0x2903 | 也是一个GATT Client可以配置的Descriptor,可以使能Server的Boardcast特性, 即将该Descriptor所属的Characterstic的Value放在LE广播包中进行传输。和CCC不一样的是，所有的Client共享同一个SCC          |
| Characterstic Format                | 0x2904 | 描述Characterstic Value的格式，比如一个温度计的度数所呈现的格式                                                                                                                              |
| Characterstic Aggregate Format      | 0x2905 | 一组Characterstic Format(Descriptor)的handle的集合, 每个handle指向一个Characterstic Format                                                                                                   | 

以上，Primary Service、Secondary Service 和Characteristic属于ATT protocol中定义的“group of attributes”。Service由其声明（declaration）、Include 和 Characteristic 组成一个group。Characteristic则由其声明、Value以及隶属于它的Descriptors组成。Primary Service、Secondary Service 都可以通过“Read By Group Type Request”来查询它们的起止handle，而Characteristic则需要多个procedure的组合来查询它的所有信息。

有了以上的几个“group of attributes”的概念，一个完整的GATT Profile的层级视图就可以用下面的图大致勾画出来了：
![[Pasted image 20230225192042.png]]
注意到在上面的图中，Include、Descriptors 都是用虚线框表示的。因此，一个最简单的GATT层级图，将只有一个Service的描述、一个Characteristic的描述以及Characteristic的Value组成。
此外，上图并没有单独列出Service的声明，而实际上它就是Service这个group的第一个Attribute。?

**Primary Service 格式**
![[Pasted image 20230225192835.png]]
Primary Service的Attribute Type就是GATT定义的通用的Attribute Type(0x2800或者0x2801)，Attribute Value字段是这个 Service 的 UUID, Attribute Permission则是这个Attribute的权限，Service的权限仅为只读。

**Include 格式**
![[Pasted image 20230225193437.png]]
Include 声明了这个服务所包含的服务，Attribute Value中的数据包括所包含服务的属性句柄，终止句柄，服务UUID，是一个只读的Attribute.

**Characterstic 格式**
![[Pasted image 20230225193838.png]]

Characterstic 主要用于说明 Service 的主要特性，Attribute Value由以下三部分组成
- Characteristic Properties
	![[Pasted image 20230225194247.png]]
	该字段决定了如何使用 Characteristic Value 或者 Characteristic Descriptors 是否能被访问(置位之后才会包含对应的Descriptor, 然后才能访问到？)，
- Characteristic Value Handle : 保存Characteristic本身的value的handle值
- Characteristic UUID : 由上层定义。

 **Characteristic Value的Attribute格式如下：**
 ![[Pasted image 20230225195004.png]]
 这里的Attribute Type是上层定义的。这也是GATT Profile当中唯一一个Type由上层定义的Attribute。其实它就是前面Characteristic声明中Attribute Value的Characteristic UUID字段。本身就是上层定义的，因此它的Attribute Permissions也是上层来决定的。每个基于GATT的profile都需要定义自己的Characteristic及其permissions。




## CH58x Periphral GATT Service
CH58x Periphral 中 自定义了一个 Service, 如下
```
/*********************************************************************
 * Profile Attributes - Table
 */

static gattAttribute_t simpleProfileAttrTbl[] = {
    // Simple Profile Service
    {
        {ATT_BT_UUID_SIZE, primaryServiceUUID}, /* type */
        GATT_PERMIT_READ,                       /* permissions */
        0,                                      /* handle */
        (uint8_t *)&simpleProfileService        /* pValue */
    },
	/* 第一个组为Primary Service(0x2800)
	  | Attribute Types    | Attribute Value            | Attribute Permission |
	  | ------------------ | -------------------------- | -------------------- |
	  | primaryServiceUUID | simpleProfileService->uuid | GATT_PERMIT_READ     |
	*/

    // Characteristic 1 Declaration
    {
        {ATT_BT_UUID_SIZE, characterUUID},
        GATT_PERMIT_READ,
        0,
        &simpleProfileChar1Props},
        //simpleProfileChar1Props = GATT_PROP_READ | GATT_PROP_WRITE;
    /* 第二组数据声明了一个(0x2803-Attribute)一个权限为simpleProfileChar1Props的Characteristic, 
	    这是一个条目(Attribute), 下面第三组才是他的特征(Characterstic)(Attribute Value中所描述的东西)
	    | Attribute Type        | Attribute Value         | Attribute Permission |
		| --------------------- | ----------------------- | -------------------- |
		| characterUUID(0x2803) | simpleProfileChar1Props | GATT_PERMIT_READ     | 
*/

    // Characteristic Value 1
    {
        {ATT_BT_UUID_SIZE, simpleProfilechar1UUID},
        GATT_PERMIT_READ | GATT_PERMIT_WRITE,
        0,
        simpleProfileChar1},
    /* 第三组数据声明这个Characteristic的UUID,此处权限应该是只读？；同时声明了Characteristic Value Attribute Handle 
    第二个第三组数据组成一个characteristic
| Attribute Value                                                                 |
| Characteristic Properties | Characteristic Value Handle | Characteristic UUID   |  
| ------------------------- | --------------------------- | --------------------- |
| RW                        | simpleProfileChar1          | simpleProfilechar1UUID| 
    
    */

    // Characteristic 1 User Description
    {
        {ATT_BT_UUID_SIZE, charUserDescUUID},
        GATT_PERMIT_READ,
        0,
        simpleProfileChar1UserDesp},
    /* Characteristic User Descriptor(0x2901), 权限只读，指向一个UTF-8格式字符串 */

    // Characteristic 2 Declaration
    {
        {ATT_BT_UUID_SIZE, characterUUID},
        GATT_PERMIT_READ,
        0,
        &simpleProfileChar2Props},

    // Characteristic Value 2
    {
        {ATT_BT_UUID_SIZE, simpleProfilechar2UUID},
        GATT_PERMIT_READ,
        0,
        simpleProfileChar2},

    // Characteristic 2 User Description
    {
        {ATT_BT_UUID_SIZE, charUserDescUUID},
        GATT_PERMIT_READ,
        0,
        simpleProfileChar2UserDesp},

    // Characteristic 3 Declaration
    {
        {ATT_BT_UUID_SIZE, characterUUID},
        GATT_PERMIT_READ,
        0,
        &simpleProfileChar3Props},

    // Characteristic Value 3
    {
        {ATT_BT_UUID_SIZE, simpleProfilechar3UUID},
        GATT_PERMIT_WRITE,
        0,
        simpleProfileChar3},

    // Characteristic 3 User Description
    {
        {ATT_BT_UUID_SIZE, charUserDescUUID},
        GATT_PERMIT_READ,
        0,
        simpleProfileChar3UserDesp},

    // Characteristic 4 Declaration
    {
        {ATT_BT_UUID_SIZE, characterUUID},
        GATT_PERMIT_READ,
        0,
        &simpleProfileChar4Props},

    // Characteristic Value 4
    {
        {ATT_BT_UUID_SIZE, simpleProfilechar4UUID},
        0,
        0,
        simpleProfileChar4},

    // Characteristic 4 configuration
    {
        {ATT_BT_UUID_SIZE, clientCharCfgUUID},
        GATT_PERMIT_READ | GATT_PERMIT_WRITE,
        0,
        (uint8_t *)simpleProfileChar4Config},

    // Characteristic 4 User Description
    {
        {ATT_BT_UUID_SIZE, charUserDescUUID},
        GATT_PERMIT_READ,
        0,
        simpleProfileChar4UserDesp},

    // Characteristic 5 Declaration
    {
        {ATT_BT_UUID_SIZE, characterUUID},
        GATT_PERMIT_READ,
        0,
        &simpleProfileChar5Props},

    // Characteristic Value 5
    {
        {ATT_BT_UUID_SIZE, simpleProfilechar5UUID},
        GATT_PERMIT_AUTHEN_READ,
        0,
        simpleProfileChar5},

    // Characteristic 5 User Description
    {
        {ATT_BT_UUID_SIZE, charUserDescUUID},
        GATT_PERMIT_READ,
        0,
        simpleProfileChar5UserDesp},
};
```


### 抓包数据
![[Pasted image 20230226112039.png]]

从抓包数据看，在两个设备连接的时候，Central 会先请求设备的Primary Service，然后再请求 Characteristic。

![[Pasted image 20230226112536.png]]
Group Type Response 数据中包含了这个 Service 的所包含的所有条目的 Handle ，

![[Pasted image 20230226112945.png]]
(Attribute Type Response)
Characteristic 数据中则包含了对应的 Attribute Value( Characteristic Properties, Characteristic Value Handle, Characteristic UUID).


### 代码分析
以 Periphral 例程中的 0xFFE0 Service 进行分析。
...