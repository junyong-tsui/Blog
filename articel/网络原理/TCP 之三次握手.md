## 动画：TCP 三次握手

### 写在前边

TCP 三次握手过程对于面试是必考的一个，所以不但要掌握 TCP 整个握手的过程，其中有些小细节也更受到面试官的青睐。

对于这部分掌握以及 TCP 的四次挥手，小鹿将会以动画的形式呈现给每个人，这样将复杂的知识简单化，理解起来也容易了很多，尤其对于一个初学者来说。



### 学习导图

![](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/思维导图.png)



### 一、TCP 是什么？

`TCP（Transmission Control Protocol 传输控制协议）`是一种面向连接的、可靠的、基于字节流的传输层通信协议。

我们知道了上述了解到了 `TCP `的定义，通俗一点的讲，`TCP `就是一个双方通信的一个规范标准（协议）。

我们在学习 `TCP` 握手过程之前，首先必须了解 `TCP` 报文头部的一些标志信息，因为在 `TCP `握手的过程中，会使用到这些报文信息，如果没有掌握这些信息，在学习握手过程中，整个人处于懵逼状态，也是为了能够深入 `TCP` 三次握手的原理。



### 二、TCP 头部报文

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/%E5%A4%B4%E9%83%A8%E6%8A%A5%E6%96%87.png](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/头部报文.png)



#### 2.1 `source port` 和 `destination port`

> 两者分别为「源端口号」和「目的端口号」。源端口号就是指本地端口，目的端口就是远程端口。

一个数据包（`pocket`）被解封装成数据段（`segment`）后就会涉及到连接上层协议的端口问题。

可以这么理解，我们可以想象发送方很多的窗户，接收方也有很多的窗户，这些窗口都标有不同的端口号，源端口号和目的端口号就分别代表从哪个规定的串口发送到对方接收的窗口。不同的应用程度都有着不同的端口，之前网络分层的文章中有提到过。

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/%E6%BA%90%E7%9B%AE%E7%9A%84%E7%AB%AF%E5%8F%A3%E5%8F%B7.png](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/源目的端口号.png)



> 扩展：应用程序的端口号和应用程序所在主机的 IP 地址统称为 socket（套接字），IP:端口号, 在互联网上 socket 唯一标识每一个应用程序，源端口+源IP+目的端口+目的IP称为”套接字对“，一对套接字就是一个连接，一个客户端与服务器之间的连接。



#### 2.2 `Sequence Numbe`
>称为「序列号」。用于 TCP 通信过程中某一传输方向上字节流的每个字节的编号，为了确保数据通信的有序性，避免网络中乱序的问题。接收端根据这个编号进行确认，保证分割的数据段在原始数据包的位置。



![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/Sqn.gif](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/Sqn.gif)



再通俗一点的讲，每个字段在传送中用序列号来标记自己位置的，而这个字段就是用来完成双方传输中确保字段原始位置是按照传输顺序的。（发送方是数据是怎样一个顺序，到了接受方也要确保是这个顺序）

>PS：初始序列号由自己定，而后绪的序列号由对端的 ACK 决定：SN_x = ACK_y (x 的序列号 = y 发给 x 的 ACK)，这里后边会讲到。



#### 2.3 `Acknowledgment Numbe`
>称为「确认序列号」。确认序列号是接收确认端所期望收到的下一序列号。确认序号应当是上次已成功收到数据字节序号加1，只有当标志位中的 ACK 标志为 1 时该确认序列号的字段才有效。主要用来解决不丢包的问题。

若确认号=N，则表明：到序号N-1为止的所有数据都已正确收到。

在这里，现在我们只需知道它的作用是什么，就是在数据传输的时候是一段一段的，都是由序列号进行标识的，所以说，接收端每接收一段，之后就想要的下一段的序列号就称为「确认序列号」。



#### 2.4 `TCP Flag`

`TCP` 首部中有 6 个标志比特，它们中的多个可同时被设置为 `1`，主要是用于操控 `TCP` 的状态机的，依次为`URG，ACK，PSH，RST，SYN，FIN`。

不要求初学者全部掌握，在这里只讲三个重点的标志：



##### 2.4.1 `ACK`
这个标识可以理解为发送端发送数据到接收端，发送的时候 ACK 为 0，标识接收端还未应答，一旦接收端接收数据之后，就将 ACK 置为 1，发送端接收到之后，就知道了接收端已经接收了数据。

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/ACK.gif](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/ACK.gif)



> 此标志表示「应答域有效」，就是说前面所说的TCP应答号将会包含在 TCP 数据包中；有两个取值：0 和 1，为 1 的时候表示应答域有效，反之为 0；



##### 2.4.2 `SYN`
>表示「同步序列号」，是 TCP 握手的发送的第一个数据包。

用来建立 TCP 的连接。SYN 标志位和 ACK 标志位搭配使用，当连接请求的时候，SYN=1，ACK=0连接被响应的时候，SYN=1，ACK=1；这个标志的数据包经常被用来进行端口扫描。扫描者发送一个只有 SYN 的数据包，如果对方主机响应了一个数据包回来 ，就表明这台主机存在这个端口。看下面动画：

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/SYN.gif](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/SYN.gif)



##### 2.4.3 `FIN`
>表示发送端已经达到数据末尾，也就是说双方的数据传送完成，没有数据可以传送了，发送FIN标志位的 TCP 数据包后，连接将被断开。这个标志的数据包也经常被用于进行端口扫描。

这个很好理解，就是说，发送端只剩最后的一段数据了，同时要告诉接收端后边没有数据可以接受了，所以用FIN标识一下，接收端看到这个FIN之后，哦！这是接受的最后的数据，接受完就关闭了。动画如下：

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/FIN.gif](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/FIN.gif)

#### 2.5 `Window size`
称为滑动窗口大小。所说的滑动窗口，用来进行流量控制。



### 3、为什么进行 TCP 三次握手？
如果之前你不了解网络分层的话，建议看看写的文章。

<font color=blue  face="黑体">[网络分层协议]([https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/%E7%BD%91%E7%BB%9C%E5%88%86%E5%B1%82%E5%88%92%E5%88%86%EF%BC%88%E4%B8%8A%EF%BC%89.md](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/网络分层划分（上）.md))</font>

第一，为了确认双方的接收与发送能力是否正常。第二，指定自己的初始化序列号，为后面的可靠传送做准备。第三，如果是 https 协议的话，三次握手这个过程，还会进行数字证书的验证以及加密密钥的生成到。

如果你了解 UDP 的话，TCP 的出现正式弥补了 UDP 不可靠传输的缺点。但是 TCP 的诞生，也必然增加了连接的复杂性。



### 4、TCP 三次握手过程？
TCP 三次握手的过程掌握最重要的两点就是客户端和服务端状态的变化，另一个是三次握手过程标志信息的变化，那么掌握 TCP 的三次握手就简单多了。下面我们就以动画形式进行拆解三次握手过程。

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B%E5%9B%BE.png](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/三次握手图.png)

- <font color=#A52A2A face="黑体">**初始状态**</font>：客户端处于 `closed(关闭) `状态，服务器处于 `listen(监听) ` 状态。

[https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/%E5%88%9D%E5%A7%8B%E5%8C%96%E7%8A%B6%E6%80%81.png](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/初始化状态.png)



-  <font color=#A52A2A face="黑体">**第一次握手**</font>：客户端发送请求报文将 `SYN = 1 `同步序列号和初始化序列号`seq = x`发送给服务端，发送完之后客户端处于` SYN_Send `状态。

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/%E7%AC%AC%E4%B8%80%E6%AC%A1%E6%8F%A1%E6%89%8B.gif](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/第一次握手.gif)



- <font color=#A52A2A face="黑体">**第二次握手**</font>：服务端受到 `SYN` 请求报文之后，如果同意连接，会以自己的同步序列号`SYN(服务端) = 1`、初始化序列号 `seq = y`和确认序列号（期望下次收到的数据包）`ack = x+ 1` 以及确认号`ACK = 1`报文作为应答，服务器为`SYN_Receive `状态。

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/%E7%AC%AC%E4%BA%8C%E6%AC%A1%E6%8F%A1%E6%89%8B.gif](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/第二次握手.gif)



- <font color=#A52A2A face="黑体">**第三次握手**</font>： 客户端接收到服务端的 `SYN + ACK`之后，知道可以下次可以发送了下一序列的数据包了，然后发送同步序列号 `ack = y  + 1`和数据包的序列号 `seq = x + 1`以及确认号`ACK = 1`确认包作为应答，客户端转为`established`状态。

![https://github.com/luxiangqiang/Blog/blob/master/articel/%E7%BD%91%E7%BB%9C%E5%8E%9F%E7%90%86/images/%E7%AC%AC%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.gif](https://github.com/luxiangqiang/Blog/blob/master/articel/网络原理/images/第三次握手.gif)



### 5、为什么不是一次、二次握手？
>防止了服务器端的一直等待而浪费资源。

为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。如果此时客户端发送的延迟的握手信息服务器收到，然后服务器进行响应，认为客户端要和它建立连接，此时客户端并没有这个意思，但 `server` 却以为新的运输连接已经建立，并一直等待 `client` 发来数据。这样，`server` 的很多资源就白白浪费掉了。

