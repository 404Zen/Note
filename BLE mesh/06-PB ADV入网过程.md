# 与GATT承载方式入网的差异
1. PB-ADV使用 **Generic Provisioning PDUs** 配置入网，而PB-GATT通过 **Proxy PDUs** 入网，前者在广播信道进行通讯，后者则通过 Proxy Service 进行数据收发，在数据通道进行通讯。
2. PB-GATT是基于链接的模型进行一序列的入网操作，而PB-ADV则基于会话模型进行相应的入网操作；前者传输配置数据的最大PDU是由ATT的MTU Size决定，而后者的MTU Size最大就只有24字节，且使用连接建立程序 **(Link Establishment procedure)** 进行会话的建立。

PB-ADV的帧格式
![[assert/Pasted image 20230217191428.png]]
PB-ADV PDU主要由**Link ID**，**Transaction Number**，**Generic Provisioning PDU**三个部分组成；
- Link ID
	该字段所表示的含义比较好理解，其起到的就是一个标识符的作用；主要用于在配置过程辨别同一个LINK ID下，Provisioner与unprovisioned device的数据交互；不同LINK ID的packets会被彼此忽略，因为一个会话的生命周期时有且仅有一个Link ID,不同的会话其LINK ID是不一样的。**注意，LINK ID是由Provisioner随机产生的**。
	
- Transaction Number
	该字段主要用于辨别每个单独的**Generic Provisioning PDU**；即从0x00开始，该值在每个新的**Generic Provisioning PDU**上累加 **(对于Provisioning Bearer Control PDU，其Transaction Number一直都是0x00)**，如果累加到最大值时，则又从0x00重新开始；对于Provisioner和Unprovisioned device它们两者的最大值是不一样的：
	-   Provisioner
	    最大只能达到0x7F;
	-   Unprovisioned Device
	    最大可能达到0xFF;
	
	但是，也有两种情况是不需要累加的：
	1.  如果一个**Provisioning PDU**放不下去时，所有的分包均是同样的**Transaction Number**的，不需要累加；
	2.  如果**Provisioning PDU**需要被重传，那么也是不需要累加的，还是使用旧的**Transaction Number**;
	
- Generic Provisioning PDU
	....


# 入网流程
与PB-GATT相比之下，多了**Generic Mesh Provisioning Link Open、Generic Mesh Provisioning Link Ack、Generic Mesh Provisioning Link Close**，那么这三者之间有什么关联呢？我们在上述的内容也有提到这一点，PB-ADV是采用会话的模型进行配置数据的收发，那么会话模型就需要这三个PDU来建立，具体如下图所示：
![[assert/Pasted image 20230217191812.png]]

## Generic Mesh Provisioning Link Open
Link Open消息用于在Provisioner和Unprovisioned Device之间建立链接,Unprovisioned Device如果接收到该消息，应使用Link ACK消息应答此消息。具体的帧格式如下：
![[assert/Pasted image 20230217191919.png]]
其中Device UUID指的是选中的Unprovisioned Device的UUID。

## Generic Mesh Provisioning Link Ack
该消息用于发送应答消息，表示其收到Link Open的消息了，具体的帧格式如下：
![[assert/Pasted image 20230217191945.png]]
该消息不携带任何参数。

## Generic Mesh Provisioning Link Close
该消息用于关闭Link会话，由于该消息是没有被应答的，因此为了保险起见，Link关闭消息最少要发送3次且Provisioner和Unprovisioned Device均可以发送该消息，并且携带**关闭原因**这个参数。具体的帧格式如下图所示：
![[assert/Pasted image 20230217192030.png]]
	注意：关于**Link Establishment procedure**我们还有以下几点需要大家了解下：
	1.  对于Provisioner而言，其在准备建立Link会话时会先设置一个60秒的定时器，接着发送**Link Open**消息；如果在60秒后没有收到Unprovisioned Device的**Link Ack**的消息，则认为此次Link会话建立失败；
    2.  对于Unprovisioned Device而言，其在收到**Link Open**消息之后，会给Provisioner回应**Link Ack**的消息，紧接着同样也开启一个60秒定时器；如果在60秒后没有收到任何的Provisioning PDU，则认为此次Link会话建立失败；




其余的差异主要在于PDU类型不同所造成的组装方式的差异。使用PB-ADV承载进行传输的数据，受限于MTU Size, 可能需要分更多的包进行传输。