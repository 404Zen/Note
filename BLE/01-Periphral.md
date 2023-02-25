# 蓝牙Periphral例程


# 初始化

设置蓝牙所使用的堆的地址以及Size，发射功率，Flash读写接口，设置MAC地址，BLE Lib初始化。
`CH58X_BLEInit()`

GAP 角色初始化，初始化为从机或者主机。
`GAPRole_PeripheralInit()`

蓝牙从机初始化。
`Peripheral_Init()`

## 广播数据
```
GAPRole_SetParameter(GAPROLE_ADVERT_ENABLED, sizeof(uint8_t), &initial_advertising_enable);
GAPRole_SetParameter(GAPROLE_SCAN_RSP_DATA, sizeof(scanRspData), scanRspData);
GAPRole_SetParameter(GAPROLE_ADVERT_DATA, sizeof(advertData), advertData);
// 使能广播，设置广播数据以及扫描回复数据。
```

蓝牙广播包的最大长度是37个字节，其中设备地址占用了6个字节，只有31个字节是可用的。这31个可用的字节又按照一定的格式来组织，被分割为n个AD Structure。如下图所示： 
![[Pasted image 20230223192232.png]]
如上图所示，每个AD Structure包含又包含三部分，分别是：

Length(1字节)，AD Type(1字节)，AD Data(n字节)

其中Length = AD Type 长度 + AD Data 长度。
```
static uint8_t advertData[] = {
    // Flags; this sets the device to use limited discoverable
    // mode (advertises for 30 seconds at a time) instead of general
    // discoverable mode (advertises indefinitely)
    0x02, // length of this data
    GAP_ADTYPE_FLAGS,
    DEFAULT_DISCOVERABLE_MODE | GAP_ADTYPE_FLAGS_BREDR_NOT_SUPPORTED,
    // service UUID, to notify central devices what services are included
    // in this peripheral
    0x03,                  // length of this data
    GAP_ADTYPE_16BIT_MORE, // some of the UUID's, but not all
    LO_UINT16(SIMPLEPROFILE_SERV_UUID),
    HI_UINT16(SIMPLEPROFILE_SERV_UUID)
};

/*  第一字节声明了第一个AD Structure的数据长度，2字节 
	第二字节声明了AD Type, 即这个AD Structure的类型，此处为设备标识
	第三字节表示这是一个LE General Discoverable Mode， 且不支持BREDR经典蓝牙

	第四个字节开始是第二个AD Structure, 这个AD Structure长度为3个字节
	第五个字节表示第二个AD Structure的数据内容，接下来的数据为部分16bit的Service UUID
	后面两个字节共同组成了一个Service UUID
	 */
```

```
// GAP - SCAN RSP data (max size = 31 bytes)
static uint8_t scanRspData[] = {
    // complete name
    0x12, // length of this data
    GAP_ADTYPE_LOCAL_NAME_COMPLETE,
    'S','i','m','p','l','e',' ','P','e','r','i','p','h','e','r','a','l',
    
    // connection interval range
    0x05, // length of this data
    GAP_ADTYPE_SLAVE_CONN_INTERVAL_RANGE,
    LO_UINT16(DEFAULT_DESIRED_MIN_CONN_INTERVAL), // 100ms
    HI_UINT16(DEFAULT_DESIRED_MIN_CONN_INTERVAL),
    LO_UINT16(DEFAULT_DESIRED_MAX_CONN_INTERVAL), // 1s
    HI_UINT16(DEFAULT_DESIRED_MAX_CONN_INTERVAL),

    // Tx power level
    0x02, // length of this data
    GAP_ADTYPE_POWER_LEVEL,
    0 // 0dBm
};
/* Scan response data的数据结构和定义和Advertise data的一样
*/
```
**AD Type**
| AD Type | 含义                                                                       |
| ------- | -------------------------------------------------------------------------- |
| 0x01    | 设备标识(Flags)                                                            |
| 0x02    | Service: More 16-bit UUIDs available                                       |
| 0x03    | Service: Complete list of 16-bit UUIDs                                     |
| 0x04    | Service: More 32-bit UUIDs available                                       |
| 0x05    | Service: Complete list of 32-bit UUIDs                                     |
| 0x06    | Service: More 128-bit UUIDs available                                      |
| 0x07    | Service: Complete list of 128-bit UUIDs                                    |
| 0x08    | 缩略的设备名称                                                             |
| 0x09    | 完整的设备名称                                                             |
| 0x0A    | 发射功率(-127~127dBm)                                                      |
| 0x19    | 设备外观                                                                   |
| 0xFF    | 厂商自定义数据，前面2字节为Company Identifier Code，然后跟着厂商定义的数据 | 

**广播数据与扫描响应的关系**
广播是蓝牙从机设备 主动 发出的数据。
而扫描响应是， 当蓝牙主机收到从机的广播数据后，如果想要进一步了解该从机设备的信息，可以向从机设备发送扫描请求，从机收到扫描请求后，向对应的主机回复扫描响应。 
![[Pasted image 20230223192558.png]]

扫描响应的数据格式和蓝牙广播的数据格式完全一样，其作用也基本一样，那为什么还要设置这么一个扫描响应数据呢？

我们前文中讲到，蓝牙的广播数据最多是31个字节，如果广播数据太多，这31个字节装不下时，我们就可以将一部分不太重要的数据放到扫描响应数据里面，来分担广播数据的工作。

### 广播数据测试
我们注释掉广播数据中的第二个AD Structure, 然后再去扫描设备，可以发现少了UUID这一项。


## 连接间隔设置
```
GAPRole_SetParameter(GAPROLE_MIN_CONN_INTERVAL, sizeof(uint16_t), &desired_min_interval);
GAPRole_SetParameter(GAPROLE_MAX_CONN_INTERVAL, sizeof(uint16_t), &desired_max_interval);
```

这个间隔是指**成功连接后**的周期性通讯时间，主机会根据使用情况在这个取值范围内选择合适的间隔时间，这个具体值是不可控的，所以需要划定一个范围使得通讯响应在自己的可控范围之内。这个具体值会影响到下一次通讯数据包的响应时间，需要根据自己的情况来调整这个范围达到 既省电又匹配程序响应速度 的目的。

比如：  
大数据传递时：通讯数据包是连续传递的，主机会选择min值来进行通讯。  
无数据传递时：通讯是空闲状态，主机会选择max值来定期询问从机状态，以保持连接不中断。（在空闲时，由于使用max的值作为通讯周期，会影响到程序的下一个命令的发送时间）


## GAP GATT Service
```
// Set the GAP Characteristics
GGS_SetParameter(GGS_DEVICE_NAME_ATT, GAP_DEVICE_NAME_LEN, attDeviceName);
```
GGS服务包含设备和访问信息，例如设备名称，Appearance，外围首选连接参数。GGS的目的是在设备发现和连接启动过程中进行辅助。

	Windows测试：在蓝牙建立连接之前，设备名称为广播数据中设置的蓝牙名称，连接之后，为GAP GATT Service中设置的蓝牙名称，iOS在nRF Connect中完成连接后，转到系统设置中可以看到设备的蓝牙名称为GAP GATT Service中设置的蓝牙名称。


## 设置广播间隔
```
// Set advertising interval
GAP_SetParamValue(TGAP_DISC_ADV_INT_MIN, advInt);
GAP_SetParamValue(TGAP_DISC_ADV_INT_MAX, advInt);
```


## 使能扫描回复通知
```
// Enable scan req notify
GAP_SetParamValue(TGAP_ADV_SCAN_REQ_NOTIFY, ENABLE);
```
当收到主机的扫描请求时，从机需要回复扫描相应。
使能了扫描请求的通知之后，可以触发`Peripheral_ProcessGAPMsg()`的 `GAP_SCAN_REQUEST_EVENT` 事件，用户可以在此处执行一些动作。

## 配对与绑定
```
// Setup the GAP Bond Manager
{
	uint32_t passkey = 0; // passkey "000000"
	uint8_t  pairMode = GAPBOND_PAIRING_MODE_WAIT_FOR_REQ;
	uint8_t  mitm = TRUE;
	uint8_t  bonding = TRUE;
	uint8_t  ioCap = GAPBOND_IO_CAP_DISPLAY_ONLY;
	GAPBondMgr_SetParameter(GAPBOND_PERI_DEFAULT_PASSCODE, sizeof(uint32_t), &passkey);
	GAPBondMgr_SetParameter(GAPBOND_PERI_PAIRING_MODE, sizeof(uint8_t), &pairMode);
	GAPBondMgr_SetParameter(GAPBOND_PERI_MITM_PROTECTION, sizeof(uint8_t), &mitm);
	GAPBondMgr_SetParameter(GAPBOND_PERI_IO_CAPABILITIES, sizeof(uint8_t), &ioCap);
	GAPBondMgr_SetParameter(GAPBOND_PERI_BONDING_ENABLED, sizeof(uint8_t), &bonding);
}
```
	passkey是配对时所采用的静态密码。
	pairMode是默认发起配对请求或从属安全请求或禁止配对(CH58x此处疑似存在bug，并不能禁止配对)
	mitm是设置链路是否需要MITM(Mid in the middle,中间人)保护,设置为1需要输入密码，设置为0则不需要。
	bonding是绑定标志位，可以设置为绑定或不绑定。
	ioCap则是设置输入和输出能力(能展示passkey或输入passkey)。

Paring（配对）和Bonding（绑定）是实现蓝牙射频通信安全的一种机制,它实现的是蓝牙链路层的安全，对应用来说完全透明，也就是说，不管有没有Paring/Bonding，你发送或接收应用数据的方式是一样的，不会因为加了Paring/Bonding应用数据传输需要做某些特殊处理。
**概念：**
1、配对特征交换得到临时密钥TK  
2、身份确认以及短期密钥STK的产生  
3、传输特定密钥

**加密、配对和绑定区别：**
加密：确保数据的机密性，机密性即指第三方“监听”或者“攻击者”由于没有加密链路的共享秘密，因此无法拦截、破译或者读取消息的原始内容。注：数据包的报头和长度字段不会被加密，好处是接收到包后可以直接分析报头判断SN的和MESN标志。

配对：配对是找到并确定需要和自己通信的设备，也就是身份确定，接着是安全密钥共享，而这一过程仅仅是由启动加密到得到短期秘钥(STK)为止。其包括配对能力交换、设备认证、密钥（固定为128bit）生成、连接加密以及机密信息分发等过程，配对的目的有三个：加密连接、认证设备、以及生成密钥；

绑定：配对过程中会生成一个长期密钥（LTK，long-term Key），如果配对双方把这个LTK存储起来放在Flash中，（有的时候是长期密钥、身份解析密钥、连接签名解析密钥这三个密钥的某一个或者组合进行交换，然后将交换的莫要存储到数据库中）。那么这两个设备再次重连的时候，就可以跳过配对流程，而直接使用LTK对蓝牙连接进行加密，设备的这种状态称为绑定（bonding）。

明确为何需要进行配对：加密认证的整个过程几乎都是围绕怎么将两个设备使用到的秘钥能够安全的共享，也就是当一方把密码告诉另一方时，始终要提防第三方也可能听得到这个密钥， 把需要共享的密钥安全的送到正确的设备才是难点，这就引入了配对的复杂过程。

这里也需要明确一个概念，配对绑定只有在两个设备之间第一次配对时才会发生，后续的连接由于第一次的配对绑定已有“Bonding”过程（即存储），如果存储的数据库没有被人为的清空，后续的连接不要配对。并不是所有的通信都需要加密进行数据保护，因此：建立连接之前不一定需要配对和绑定，可以直接建立连接。

配对流程图
![[Pasted image 20230223201831.png]]
阶段1：配对特征交换得到临时密钥(TK)值 （配对请求、配对响应）；
阶段2：身份确认以及短期密钥(STK)生成（通过安全管理协议(SMP)配对），确定自己正在和一个真正想要同行的设备通信，而非第三方，即确定对方身份；
阶段3：传输特定密钥 （密钥分配）。绑定所需存储到安全数据库的数据也是此阶段发送的。

**上述三阶段总结：**
1、配对认证：主从机一方提供密码，一方输入密码，如果双方密码一致，那么此密码将作为TK（临时密码）；
2、加密链路：利用得到的TK（临时密码）等信息计算出STK（短期密码）用来做加密认证；
3、绑定：加密认证通过后，利用STK等信息生成LTK（长期密码），把LTK保存下来，用于下次连接时做加密认证，不需要再次配对就可以加密链路，这就是绑定了；

绑定后通讯过程 : 每次连接时，从机会向主机发送安全请求，如果主从机相互绑定过，主机不会发送配对请求，主机直接利用绑定时保存的LTK发送加密请求,从机也会利用绑定时保存的LTK来做加密回复，三次握手成功后（加密成功，三次握手通讯由底层完成，用户不可见），从机回复主机加密状态success。

**上述过程可以这样理解：**

建立连接是使用的**静态密码**，解除绑定/配对是使用的**动态密码**，因为我们删除静态密码是没有意义的，这是一个已知的。回连也是需要密码的，使用的是动态密码，这是为了防止窃听，起保护作用，但是一般为了防止泄露信息用户也可以自行在应用层去实现相同功能，两者从功能和安全性上没有本质区别。Paring/Bonding是将上述的工程标准化，使用户可以无感的使用安全的蓝牙通信。

配对和绑定区别:

1)连接：通讯的基础，通讯数据为明文；
2)配对：配对仅仅是为了在连接的基础上加密(通讯数据经过加密为密文)，提高蓝牙链路传输的安全性。不配对也能连接进行通信。
3)绑定：绑定是配对发起时的一个可选配置。把配对信息记录下来， 下次不用配对自动进入加密的连接；所以没在bonding列表里的设备不影响连接，照连不误。

**这里涉及到一个概念，为什么我们日常使用的蓝牙产品很少要求绑定呢？**
因为我们在使用绑定时，大部分是由应用层去管理的，如果需要绑定输入密码，首先会被系统层获取到，这就已经不安全了；其次，蓝牙的加密本身是不安全的，第一次连接有概率被监听到（后续的回连被监听的难度较大）

## GATT属性
```
// Initialize GATT attributes
GGS_AddService(GATT_ALL_SERVICES);           // GAP
GATTServApp_AddService(GATT_ALL_SERVICES);   // GATT attributes
DevInfo_AddService();                        // Device Information Service
SimpleProfile_AddService(GATT_ALL_SERVICES); // Simple GATT Profile
```
	前两个函数分别用于添加基础的GAP和GATT Service ?
	第三行的函数用于添加设备信息(0x180A)这个Service,
	第四行添加了一个自定义Service.(0xFFF0).

```
// Setup the SimpleProfile Characteristic Values
{
	uint8_t charValue1[SIMPLEPROFILE_CHAR1_LEN] = {1};
	uint8_t charValue2[SIMPLEPROFILE_CHAR2_LEN] = {2};
	uint8_t charValue3[SIMPLEPROFILE_CHAR3_LEN] = {3};
	uint8_t charValue4[SIMPLEPROFILE_CHAR4_LEN] = {4};
	uint8_t charValue5[SIMPLEPROFILE_CHAR5_LEN] = {1, 2, 3, 4, 5};

	SimpleProfile_SetParameter(SIMPLEPROFILE_CHAR1, SIMPLEPROFILE_CHAR1_LEN, charValue1);
	SimpleProfile_SetParameter(SIMPLEPROFILE_CHAR2, SIMPLEPROFILE_CHAR2_LEN, charValue2);
	SimpleProfile_SetParameter(SIMPLEPROFILE_CHAR3, SIMPLEPROFILE_CHAR3_LEN, charValue3);
	SimpleProfile_SetParameter(SIMPLEPROFILE_CHAR4, SIMPLEPROFILE_CHAR4_LEN, charValue4);
	SimpleProfile_SetParameter(SIMPLEPROFILE_CHAR5, SIMPLEPROFILE_CHAR5_LEN, charValue5);
}
```
	设置自定义Service的初始值。


```
// Init Connection Item
peripheralInitConnItem(&peripheralConnList);

// Register callback with SimpleGATTprofile
SimpleProfile_RegisterAppCBs(&Peripheral_SimpleProfileCBs);

// Register receive scan request callback
GAPRole_BroadcasterSetCB(&Broadcaster_BroadcasterCBs);

// Setup a delayed profile startup
tmos_set_event(Peripheral_TaskID, SBP_START_DEVICE_EVT);
```
	设置连接参数？
	设置自定义Service的回调函数
	设置GAP的回调函数
	启动任务(用于GAP Role的启动)。



# 参考资料
 https://www.bilibili.com/read/cv15007387?spm_id_from=333.999.0.0 