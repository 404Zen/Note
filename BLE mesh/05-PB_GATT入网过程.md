
# 入网流程
1. 发送beacon信号
2. 邀请
3. 交换公共密钥
4. 认证
5. 启动配置数据分发

![[Pasted image 20230217164957.png]]

![[Pasted image 20230217165058.png]]
从上图可知，整个入网的过程还是比较复杂了，做了这么多交互就是为了最终的**provisioning data**中的值。在new device向provisioner发送provisioning complete PDU之后，new device华丽地转身为Node。

# Mesh Provisioning Service
这个服务主要有两个特征值：
-   mesh provisioning data in **(Write Without Response)**
    该特征值主要用于provisioning client向provisioning server发送provisioning pdu
    
-   mesh provisioning data out **(Notify)**
    该特征值主要用于provisioning server向provisioning client发送provisioning pdu
    
![[Pasted image 20230217170853.png]]
从上图的描述，provisioning pdus的交互都是在provisioning service中进行的。mesh provisioning data in与mesh provisioning data out中的进和出的参照物是设备本身；该服务的作用仅用于unprovisioned device与provisioner用于入网数据交互时使用。


# Mesh Proxy Service
**Mesh Proxy Service**，该服务主要就两个特征值
-   mesh proxy data in **(Write Without Response)**
    该特征值主要用于provisioner向node发送proxy pdus **(不包括provisioning pdu)**
    
-   mesh proxy data out **(Notify)**
    该特征值主要用于node发送proxy pdus **(不包括provisioning pdu)** 至provisioner


为了让上述的语句更加易于理解，我们可以查看下图中所述的mesh proxy service的工作原理：
![[Pasted image 20230217171945.png]]
从上面我们可以很清楚地了解到，**Mesh Network PDU**都可以通过 **mesh proxy data in**这个特征值传送相应的PDU至node。当然，也可以通过**mesh proxy data out**这个特征值传输相应的PDU至**provisioner**；那么，这两个特征值除了上面提及的PDU之外，它还能传输哪些类型的PDU呢？显然，小编这么说那么就肯定是不止上述的两种类型PDU啦，对吧😄。其实，该mesh proxy service是可以传输如下几种类型的PDU：

-   Network PDU
-   Mesh Beacon
-   Proxy Configuration

其中**Mesh Beacon**是Node-->provisioner发送**mesh secure network beacon**，而**Proxy Configuration**则是用于Proxy Client与Proxy Server之间交互Proxy配置信息，主要用于将目标地址新增至白名单或者黑名单,或从白名单和黑名单中移除，起到过滤的作用；显然，该服务的作用是用于入网之后，以上类型PDU的数据交互。

## Proxy PDU
Proxy客户端通过Proxy PDU与Proxy服务端进行数据交互；如上所述，Proxy PDUs可以包含**Network PDUs**、**mesh beacons**、**proxy configuration messages**或者**Provisioning PDUs**。同时，单一的Proxy PDU可以包括一帧完整的数据，也可以是分包的数据；**还有一点要注意！！！，Proxy PDU的长度大小是根据ATT_MTU来决定的**。

也就是说， Proxy PDU可能包含其他设备的 **数据，beacon, 或者用于配置**的消息。

仅支持ADV承载的节点需要通过Proxy节点进行配网(Prosionner为智能设备的情况)?。


## Provisioning PDU
该PDU的主要用于provisioner与new device的交互。帧格式如下
![[Pasted image 20230217173022.png]]
上图包含了GATT承载方式入网所用到的所有帧类型。
- Padding
	固定为0b00
- Type
| Type      | Name                        |
| --------- | --------------------------- |
| 0x00      | Provisioning Invite         |
| 0x01      | Provisioning Capabilities   |
| 0x02      | Provisioning Start          |
| 0x03      | Provisioning Public Key     |
| 0x04      | Provisioning Input Complete |
| 0x05      | Provisioning Confirmation   |
| 0x06      | Provisioning Random         |
| 0x07      | Provisioning Data           |
| 0x08      | Provisioning Complete       |
| 0x09      | Provisioning Failed         |
| 0x0A-0xFF | RFU                         |

- Parameters
	根据Type填充对应的参数


## 发送Beacon信号

不管是入网了还是没有入网，设备均会向外发送Beacon信号。唯一的不同就是：
-   入网之后的设备发送出来的是Secure Network Beacon
-   未入网的设备则发送出来的是Unprovisioned Device Beacon

## Invitation(邀请)
这个动作是provisioner主动发起的，当其发现对端设备是unprovisioned device时，便会向未入网的设备发出邀请；而其包含了两个步骤：
![[Pasted image 20230217173705.png]]
### Provisioning Invite
首先，provisioner通过**mesh proxy data in**特征值以Write Command的形式写入**Provisioning Invite PDU**，该类型的帧内容仅包括**attention time**，就是给予new device一个时间值，然后做出任何可以引起周边事物注意的动作时长为**attention time**秒，

### Provisioning Capabilities
该PDU是new device发给provisioner的，内容会比较复杂；主要是用于告诉provisioner我new device具备哪些能力，这些能力也就影响着后续的一序列动作，如公钥的交互、认证等等。(OOB:带外数据)
| Field               | Size(octets) | Notes                      |
| ------------------- | ------------ | -------------------------- |
| Numbers of elements | 1            | 设备支持的元素的个数       |
| Algorithms          | 2            | 支持的算法和其他功能等 (目前仅支持一种算法)    |
| Public Key Type     | 1            | 支持的公钥的类型           |
| Static OOB Type     | 1            | 支持的带外数据类型         |
| Output OOB Size     | 1            | 输出带外数据的最大size     |
| Output OOB Action   | 2            | 支持的输出带外数据的动作？ |
| Input OOB Size      | 1            | 输入的带外数据的最大size   |
| Input OOB Action    | 2            | 支持的输入带外数据的动作？ |



## Exchanging Public keys(交换公钥)
交换公钥， 经过邀请的步骤之后，provisioner已经知道new device具备哪些能力了，那么provisioner就会根据new device响应的能力进行相对应类型的公钥交换。还有一点，需要注事的是：如果响应的**Provisioning Capabilities PDU**中的加密算法provisioner不支持，则provisioner会选择它所支持的安全性最高的加密算法。

### Provisioning start
由上述的交换公钥可以很清楚地了解到，该PDU由provisioner根据Provisioning Capabilities PDU中的选项，发送相应的内容给new device，告诉它接下来要采用哪种方式进行公钥交换以及采用何种认证方式。同时，new device收到该PDU之后，它会将**Attention Timer**设置为0。

如果不采用OOB方式的Public key, 那么公钥酱油provisioner和device各自产生并相互交换。

如果采用OOB方式的Public key, 那么provisioner通过OOB方式从new device获取到公钥，然后provisioner自己产生一个公钥传送给new device。

不管是provisioner还是new device都要检验公钥的有效性，如果有效，则使用公钥通过下面的公式计算得出**ECDHSecret**：

> ECDHSecret = P-256(private key, peer public key)

否则此次配置入网失败。

这一步，是Provisioner告诉new device,我根据你的**Provisioning Capabilities PDU**反馈，决定进行什么类型方式的认证。


### Provisioning Public Key

此时，Provisioner与new device两者相互交换要在ECDH计算所用到的公钥，此PDU格式如下所示：
| Field        | Size | Notes                                       |
| ------------ | ---- | ------------------------------------------- |
| Public key x | 32   | The x component of the FIPS P-256 algorithm |
| Public key y | 32   | The y component of the FIPS P-256 algorithm | 


## Authentication(认证)
### Provisioning Input Complete
当new device完成认证数字的输入之后，会向procisioner发送该PDU，该PDU不携带任何参数。

### Provisioning Confirmation
Provisioner与new device会各自将目前为止所有已经交互过的PDU **（包括发送的和接收到的，如Provisioning_Invite_PDU、Provisioning_Capabilities_PDU、Provisioning_Start_PDU、Provisioning_Public_key_PDU，但是要注意的是仅仅是PDU不包括帧头的类型，如Invite、Capabilities等等）**,包括OOB认证值以及还未发送出来的随机数，将它们加密之后的哈希值发送对端设备，以便进行下一步的确认，但是该PDU名虽说是confirmation，实质其还不能完全地确认。它们仍然需要知道对方的随机数方可确认发送的Confirmation是否匹配。该PDU的帧格式比较简单，如下所示：
| Field        | Size | Notes                                                             |
| ------------ | ---- | ----------------------------------------------------------------- |
| Confirmation | 16   | The value exchanged so far including the OOB Authentication value | 

### Provisioning Random
交换随机数，只有当Provisioner与new device双方收到该PDU才能校验确认值。同样，该PDU也是很简单：
| Field  | Size | Notes                               |
| ------ | ---- | ----------------------------------- |
| Random | 16   | The final input to the confirmation |

到此，认证工作就已经完成了，provisioner将会给new device发放配置数据。

## Distribution of provisioning data(配置数据分发)

### Provisioning Data
通过认证之后，provisioner会将provisioning data发给new device,但是所有的数据都是通过**会话秘钥**加密的，而会话秘钥又派生于我们上述所说的**ECDHSecret**。
| Field                       | Size | Notes                                                                                                                                                                 |
| --------------------------- | ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Encrypted Provisioning data | 25   | An encrypted and authenticated network key, NetKey Index, Key Refresh Flag, IV Update Flag, current value of the IV Index, and unicast address of the primary element |
| Provisioning Data MIC       | 8    | PDU Integrity Check value                                                                                                                                             | 

**Encrypted Provisioning data**
| Field           | Size | Notes                                  |
| --------------- | ---- | -------------------------------------- |
| Network key     | 16   | 网络密钥                               |
| Key Index       | 2    | Index of the Network key               |
| Flags           | 1    | Flags bitmask                          |
| IV Index        | 4    | Current value of IV Index              |
| Unicast Address | 2    | Unicast address of the primary element | 

-   Network Key
    这个毫无疑问就是我们的网络秘钥了；
    
-   Key Index
    这个又是什么玩意呢？由于网络密钥的长度是16字节，如果直接在SIG Mesh网络中传输16字节的网络密钥，有点不太现实。因为，原本就能占用的帧数据的位置就不够，所以引入了网络密钥索引这个概念，当我们要将网络密钥传输给其他节点，只需要要传网络密钥索引即可。通过个索引我们可以在网络密钥列表获取相对应的网络密钥；更多的细节，读者可以参考**SIG Mesh Spec的4.3.1.1章节**；
    
-   Flags
    该字段会比较好理解，无非就是IV Update和Key Refresh相关：
| Bits | Definition                                               |
| ---- | -------------------------------------------------------- |
| 0    | Key Refresh Flag (0: False 1: True)                      |
| 1    | IV Update Flag (0: Normal operation 1: IV Update active) |
| 2–7  | Reserved for Future Use                                  | 
    
-   IV Index
    这个也同样好理解，表示当前的IV索引值；其具备如下两个功能：
    1.  用于应用层和网络层的身份验证和加密
    2.  当**Sequence Number**快要溢出时，发起**IV Update**程序，让**IV Index**值增加，从而防止**Sequence Number**被重复使用
    
-   Unicast Address
    该内容表示provisioner分配给当前设备首要元素的单播地址；

### Provisioning Complete、Failed
这两个概念就更好理解了，如果成功接收到并处理provisioning data,那些new device就会向provisioner发送provisioning complete PDU，否则发送provisioning failed PDU。需要大家注意的是前者是没有携带任何的参数，而后者是携带有具体的错误码 **(1个字节的长度)**，也就是说我在哪个环节出现问题了.










