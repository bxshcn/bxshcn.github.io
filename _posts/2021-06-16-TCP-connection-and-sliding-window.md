---
layout: post
title:  "TCP连接管理和sliding window协议"
comments: true
categories: 网络
---
## TCP协议
TCP是一个connection-oriented，reliable（基于校验码和速度control两种机制）, gives flow control的协议

TCP将节点之间数据的传输视为一个按内容字节编号的segments的有序集，每个TCP segment的Header中有一个4字节长度的数字，用于表示当前segment携带内容的首字节的编号，即下图的sequence number：
![TCP-Header](/assets/2021-06-16-TCP-handshake-and-flowControl/TCP-Header.png)
另外一个需要特别注意的字段是window size，表示接收窗口的大小，我们在后面flow control中会涉及。
> **注意**
> 
> If the sequence number for a TCP segment at any instance was 5000 and the Segment carry 500 bytes, the sequence number for the next Segment will be 5000+500+1. That means TCP segment only carries the sequence number of the first byte in the segment.

由segments组成的有序集实质上就是一个单向的stream。每个TCP connection has **two streams one in each direction**. Outgoing stream for outgoing messages and incoming stream for incoming messages.

## TCP连接管理
TCP连接管理分为建立连接（connection establish）和断开连接（connection termination）两部分

### connection establish using 3-way handshaking
我们知道， TCP是**双工通讯**，不论是Server端还是Client端，都要
1. 告知对方自己准备发送数据的sequence number，作为后续flow control的初始值，是为同步机制（synchronization）
2. 收到对方的ACK响应以确认对方收到了自己的初始SYN消息

为了让双方都达到上述目的，最简单的模式就是3-way handshaking：
![3-way-handshaking](/assets/2021-06-16-TCP-handshake-and-flowControl/3-way-handshaking.png)
1. The client sends the first segment, a SYN segment, in which only the SYN flag is set. This segment is for synchronization of sequence numbers. It consumes one sequence number. When the data transfer starts, the sequence number is incremented by 1. We can say that the SYN segment carries no real data, but we can think of it as containing 1 imaginary byte.
2. The server sends the second segment, a SYN + ACK segment, with 2 flag bits set: SYN and ACK. This segment has a dual purpose. It is a SYN segment for communication in the other direction and serves as the acknowledgment for the SYN segment. It consumes one sequence number. 这里server的ACK应答中给出的ack=8001，告诉Client发送方下一个发送的数据应该从8001字节开始。
3. The client sends the third segment. This is just an ACK segment. It acknowledges the receipt of the second segment with the ACK flag and acknowledgment number field. Note that the sequence number in this segment is the same as the one in the SYN segment; the ACK segment does not consume any sequence numbers. 注意此时client发送的seq仍然是8000，因为这并非实际的数据，而只是一个单纯ACK应答。
> 注意
> 
> 初始时，Server端必须先进入passive open模式，相当于对外发起了邀约，此后才能建立TCP连接。也就是说，TCP握手的发起者其实是Server。

### connection termination
仍然是因为TCP的**双工通讯**特点，为了关闭双向流，常规的做法仍然是3-way handshaking，只不过此时使用FIN和ACK两个标记来实现。

![termination-with-3-way-handshaking](/assets/2021-06-16-TCP-handshake-and-flowControl/termination-with-3-way-handshaking.png)
1. Client发送的FIN segment can include the last chunk of data sent by the client, or it can be just a control segment. 上图中是单纯control segment的情况（红框）。
2. server发送的FIN+ACK segment can also contain the last chunk of data from the server. or it can be just a control segment. 上图中是单纯control segment的情况（蓝框）。
3. Client发送最后一个ACK control segment。双向连接同时关闭。

但存在另外一种常见情况，就是一端停止发送数据但同时能接收数据，这就是half-close。注意任何一方都能发起half-close，虽然多数情况下都是Client发起half-close，比如客户端请求服务端进行排序，只有当客户端发送完所有的数据后，服务端才能开始排序，Client发起half-close相当于通知Server，它已经发送完了所有的数据，但同时它会等待接收Server的排序结果。

![half-close](/assets/2021-06-16-TCP-handshake-and-flowControl/half-close.png)
整个过程比较简单，注意到Server端的第一次ACK只是ACK，而不是ACK+FIN。

> 国内有把这种half-close的情况称为“四次挥手”，不论如何，3次还是4次，取决于关闭时的场景。

## sliding window协议
sliding window是一种通用协议，它是a feature of packet-based（sending a batch of data, the packet, along with additional data that allows the receiver to ensure it was received correctly，比如校验码checksum） data transmission protocols，比如在data link layer (OSI layer 2)中，更典型的，在TCP协议栈的实现中都有实现。

TCP使用sliding window protocol控制数据发送方的发送速度，因此sliding window也叫send window。

一般来说，slinding window的大小由两方面控制，一是传输网络的容量，二是接收方的消费速度。我们把前者称为congestion control，它主要根据发送方根据收到ack的特征信息（比如接收指定ack超时，或者连续三次接收到同一个ack等）来判断**整个网络的拥塞程度**，从而决定是否发送或者发送多少数据；我们把后者称为flow control，是发送方根据收到的ack的自身信息（即ack包中所带的sequence number和window size）来判断**接收端的数据处理进度**。

### flow control
所谓流量控制就是根据接收方的消费速度，控制发送方的生产速度。避免无效的数据发送尝试。

根据数据（字节）发送后，是否收到ack响应确认，以及接收方缓存是否存在空闲两个要素，可以将整个TCP数据流分为四种类型：
1. 已发送已确认。数据流中已经发送并收到ack确认。如下图所示， 31个字节已经发送并确认。
2. 已发送但尚未确认。已发送但尚未得到ack确认的字节。下图所示14字节为第2类。
3. 未发送而接收方已Ready，即接收方存在空闲缓存。如图， 第3类有6字节。
4. 未发送而接收方Not Ready 由于接收方not ready， 还不允许将这部分数据发出。
![flow-control-categories](/assets/2021-06-16-TCP-handshake-and-flowControl/flow-control-categories.png)

其中1因为接收方已经处理我们不再关心；2和3两部分是关键：只要2和3确定，则4就被确定。

实际上，接收方会在最近一次ack响应时，通过TCP Header中的window size告知发送方可以接收的窗口大小，这个窗口大小正好包含了2和3两个部分。正是通过ack响应中的sequence number和window size两个字段，接收方实现了流量控制。我们把这两个字段决定的窗口称为receive window，简称rwnd。
![flow-control](/assets/2021-06-16-TCP-handshake-and-flowControl/flow-control.png)

从上图中可以看出，B进行了三次流量控制。第一次把窗口控制到rwnd=300 ，第二次又减到了rwnd=100 ，最后减到 rwnd=0 ，即不允许发送方再发送数据了。这种使发送方暂停发送的状态将持续到主机B重新发出一个新的窗口值为止。注意B向A发送的三个报文段都设置了标记位 ACK=1 ，只有在ACK=1时确认号字段才有意义。

### congestion control
除了接收方的处理速度外，发送方还要考虑网络的承载能力：若信息发送途径网络中的设备（比如交换机、路由器，网关等）处理能力不足，也会影响数据传输的有效性。

所谓拥塞控制，就是根据网络负载情况，控制发送方的生产速度。避免过多的数据发送到网络中，恶化整个网络的负载。

网络负载是否合理的判断依据是接收端对发送数据包的ack的响应情况，如果响应及时，则认为网络状态良好，如果响应超时，或者收到重复响应，则认为网络负载过高。

为了识别网络负载并恰当的应对，发送方TCP协议栈主要使用两个变量来控制可发送数据包：
1. cwnd，即congestion window，这是发送方当下可发送字节的大小。
2. ssthresh，即slow start threash，初始值为2。这个阈值是切换 `slow-start` 和 `congestion avoidance`两种发送策略的标准

所谓slow-start，是指初始化cwnd=1，然后按照**指数增长cwnd**的大小，直到超出ssthresh的值，之后就按照congestion avoidance算法，**线性的增长cwnd**的大小，直到探测到**网络出现拥塞**，就设定ssthresh=1/2*cwnd，然后再开始slow-start。

判定网络出现拥塞的基本策略有如下两种：
1. 超时。如果发送方设置的超时计时器时限已到但还没有收到ack确认，那么很可能是网络出现了拥塞
2. 重复收到3次针对同一sequence number包的确认。前提是tcp协议栈实现时，接收方每收到一个失序的报文段后就立即发出重复确认，而不是等自己发送数据时才进行捎带确认。失序的报文可能意味着数据包丢失，重复3次针对同一sequence number包进行ack确认，意味着发送数据丢失的概率极大。

![congestion-control](/assets/2021-06-16-TCP-handshake-and-flowControl/congestion-control.png)

### 小结
发送方既要考虑rwnd大小，也要考虑cwnd大小，因此其发送窗口(sliding window)只能取`min{rwnd, cwnd`}`

the **receive window** is managed by the receiver, who sends out window sizes to the sender. The window sizes announce the number of bytes still free in the receiver buffer, i.e. the number of bytes the sender can still send without needing an acknowledgement from the receiver. The flow-control is run on the receive side and is communicated to the sender whenever the receiver sends a packet (usually, an ACK) to the sender. 

The **congestion window** is a sender imposed window that was implemented to avoid overrunning some routers in the middle of the network path. The sender, with each segment sent, increases the congestion window slightly. But if the sender detects packet loss, it will cut the window in half. **The rationale behind this is that the sender assumes that packet loss has occurred because of a buffer overflow somewhere in the net (which is almost always true)**, so the sender wants to keep less data "in flight" to avoid further packet loss in the future.

参考：
- [1] Pamela Fox. Transmission Control Protocol (TCP)@khanacademy.org 
- [2] Zhang Jiawen. 网络基本功系列
- [3] 匿名. TCP流量控制和拥塞控制
- [4] 匿名. difference between sliding window and congestion window@stackoverflow.com
- [5] 匿名. Sliding_window_protocol@wikipedia.org
- [6] 匿名. [Transport Layer : TCP Connection](https://upscfever.com/upsc-fever/en/gatecse/en-gatecse-chp111.html)