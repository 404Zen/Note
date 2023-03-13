# 与Heartbeat的区别
-   Heartbeat
    1.  最常用的就是告诉同一个Mesh网络的其他节点，我还"活"着，没有掉线；
    2.  同一个Mesh网络中接受到心跳包时，可以判断其离发送者有多远，从而配置合理的TTL值；

-   Health Model    
    1.  汇报当前节点的故障信息，例如："设备过热，就会发出过热的故障信息"，"设备电量不足，就会发出电量不足的故障信息"等等；
    2.  该模型在一定程度上，也可以使用**Health Current Status**来告之同一个Mesh网络的其他节点，我还"活"着，没有掉线；

总得来说，这两者是相互独立，又可以同时共存；只不是前者仅仅是侧重于节点是否正常工作，而后者不但能监控到节点是否工作，同时还能汇报是什么原因导致不能工作，也就是说可以提供更多的细节；

# Health Server Model
该模型跟**Configuration Server Model**是一样的，也是SIG强制要求必须要有的一个模型，也就是说每个节点都会有至少一个Health Server Model；其主要的特性如下所示：

-   同样，该模型也是一个根模型
-   跟其他普通的SIG模型一样，该模型也支持发布与订阅的功能
-   跟**Configuration Server Model**不同的是，其不但可以存在于首要元素中，也可以存在于次要元素中；同时，还可以在一个节点的不同元素中同时存在
![[assert/Pasted image 20230218141154.png]]
-   与该模型的通讯均采用AppKey加解密
-   其SIG Model ID为0x0002


# Health Client Model

该模型跟上述的[Health Server Model](#Health-Server-Model)一般都是成双成对的，用于发送Client命令查询或者配置对端设备的状态值；换句话说，无非就是Client用于发送配置或者查询命令，而Server用于接收命令并响应相对应的命令，基本上能有该模型的大部分都是Provisioner。其主要的特性大体跟Server是一样的，详情如下所示：

-   同样，该模型也是一个根模型
-   该模型不但可以存在于首要元素中，也可以存在于次要元素中
-   与该模型的通讯均采用AppKey加解密
-   其SIG Model ID为0x0003

