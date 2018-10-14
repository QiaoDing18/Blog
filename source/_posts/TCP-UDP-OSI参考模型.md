title: TCP UDP OSI参考模型
author: 乔丁
tags:
  - 计算机网络
categories:
  - net
date: 2018-04-27 16:45:00
---

### TCP UDP
#### 基本区别
1、TCP基于连接 UDP基于无连接
2、TCP要求系统资源较多，UDP较少
3、UDP程序结构较简单 
4、流模式（TCP）与数据报模式(UDP)
5、TCP保证数据正确性，UDP可能丢包（TCP更可靠，UDP不可靠但是快）

 TCP


**三次握手为了防止服务器端的一直等待而浪费资源**
**第一次握手**：客户端 SYN=1 用来建立连接 seq=x
**第二次握手**：服务端 ACK=x+1 同意建立连接 seq=y SYN=1
**第三次握手**：客户端收到了 ACK=y+1 Seq=x+1

1.第一次握手：建立连接。客户端发送连接请求报文段，将SYN位置为1，Sequence Number为x;然后，客户端进入SYN_SEND状态，等待服务器的确认;
2.第二次握手：服务器收到SYN报文段。服务器收到客户端的SYN报文段，需要对这个SYN报文段进行确认，设置Acknowledgment Number为x+1(Sequence Number+1);同时，自己自己还要发送SYN请求信息，将SYN位置为1，Sequence Number为y;服务器端将上述所有信息放到一个报文段(即SYN+ACK报文段)中，一并发送给客户端，此时服务器进入SYN_RECV状态;
3.第三次握手：客户端收到服务器的SYN+ACK报文段。然后将Acknowledgment Number设置为y+1，向服务器发送ACK报文段，这个报文段发送完毕以后，客户端和服务器端都进入ESTABLISHED状态，完成TCP三次握手。

1.第一次分手：主机1(可以使客户端，也可以是服务器端)，设置Sequence Number和Acknowledgment Number，向主机2发送一个FIN报文段;此时，主机1进入FIN_WAIT_1状态;这表示主机1没有数据要发送给主机2了;
2.第二次分手：主机2收到了主机1发送的FIN报文段，向主机1回一个ACK报文段，Acknowledgment Number为Sequence Number加1;主机1进入FIN_WAIT_2状态;主机2告诉主机1，我也没有数据要发送了，可以进行关闭连接了;
3.第三次分手：主机2向主机1发送FIN报文段，请求关闭连接，同时主机2进入CLOSE_WAIT状态;
4.第四次分手：主机1收到主机2发送的FIN报文段，向主机2发送ACK报文段，然后主机1进入TIME_WAIT状态;主机2收到主机1的ACK报文段以后，就关闭连接;此时，主机1等待2MSL后依然没有收到回复，则证明Server端已正常关闭，那好，主机1也可以关闭连接了。

A--->B Fin, 
B--->A ACK, A属于主动关闭方，收到B的ACK后，A到B的方向连接关闭，即half shutown ，这时A不能再发送数据了。
B--->A Fin, 这种状态下B还是可以单向发送数据的
A--->B ACK  B收到ACK，关闭连接
如ACK没有到达B，B会为FIN这个消息超时重传 timeout retransmit 
那如果A等待时间足够，又收到FIN消息，说明ACK没有到达B，于是再发送ACK，直到在足够的时间内没有收到FIN，说明ACK成功到达。这个等待时间至少是：B的timeout + FIN的传输时间，为了保证可靠，采用更加保守的等待时间2MSL。


## OSI参考模型
从底层到高层
1、物理层 2、数据链路层 3、网络层 4、传输层 5、会话层 6、表示层 7、应用层

网络层： IP,ICMP,IGMP，【ARP,RARP】
传输层：TCP,UDP,UGP
应用层：Telnet,FTP,SMTP,SNMP, HTTP
