Linux高性能服务器学习
=
## 一.TCP/IP协议
### a.协议层次
1.TCP、IP协议是一个四层的结构：
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/001.png)
2.数据链路层，常用的协议是ARP协议和RARP协议，实现了IP地址和物理地址之间的转化
3.网络层，常用协议有ICMP、IP
4.传输层，常用协议有TCP、UDP
5.应用层，不举例
### b.协议封装
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/002.png)
## 二.IP协议
### a.IPV4
1.头部结构：
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/003.png)
- 4位版本号固定为4；
- 4位头部长度表示头部有多少个4字节，4位最大表示15，所以头部长度最大为60字节；
- 8位服务类型包括：3位的优先权（已废弃），4位的TOS字段（分别表示最小延时，最大吞吐量，最高可用性，最小费用，只能有一个置为1），1位的保留字段（必须为0）；
- 16位总长度指整个报文的长度，单位为字节；
- 16位标识，唯一标识每一个报文，初始值由系统生成，每发一个报文，该值+1，同一个报文的不同分片有相同的值；
- 3位标志包括：1位保留，1位DF（禁止分片），1位MF（更多分片）；
- 13位分片偏移，是分片相对于原始报文开始处（数据部分）的偏移，实际偏移量为该值的8倍，因此除最后一个分片外，其他报文的数据部分的长度必须是8的整数倍；
- 8位生存时间，路由跳数；
- 8位协议，在/etc/protocols中定义；
- 16位头部校验和，CRC校验；
- 32位源IP；
- 32位目的IP；
- 最多40位的边长信息
2.IP分片
IP报文长度超过数据帧的MTU时，分片；MTU为1500字节，因此携带的最多数据为1480字节
3.IP路由
route命令查看路由表
### b.IPV6
1.头部结构：
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/004.png)
- 4位版本号固定为6；
- 8位通信类型类似于IPV4的TOS；
- 20位标签流，指示数据优先级，如实时视频；
- 16位净荷长度，指扩展头部和数据部分的长度；
- 8位下一个包头，指出紧跟在固定头部后的包头类型，如扩展头部或上层协议的包头，类似于IPV4的协议字段；
- 8位跳数限制，类似于TTL；
- 128位源地址；
- 128位目的地址；
## 三.TCP协议
### a.特点：面向连接，字节流，可靠传输
### b.头部结构
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/005.png)
- 16位源端口号；
- 16位目的端口号；
- 3序号，一次TCP通信某一方向上的字节流的每个字节编号，即当前报文中第一个字节相对于字节流头部的偏移，单位为字节；
- 32位确认号，用作对另一方发来的报文段的响应，其值为收到的报文段的序号+1；
- 4位头部长度，标识该报文头有多少个4字节，TCP报文头最大长度为60字节；
- 6位标志位包括：
  - URG标志，表示紧急指针知否有效；
  - ACK标志，表示确认号是否有效，携带ACK标志的报文为确认报文；
  - PSH标志，提示接收端应立即从缓冲区取走数据；
  - RST标志，要求对方重新建立连接，携带RST标志的报文为复位报文，如访问不存在的端口或一方非正常关闭；
  - SYN标志，表示请求建立连接，携带SYN标志的报文为同步报文；
  - FIN标志，通知对方本端要关闭连接了，携带FIN标志的报文为结束报文；
- 16位窗口大小；
- 16位校验和，CRC校验；
- 16位紧急指针，是一个正的偏移量，是当前报文紧急数据存放处的偏移；
### c.三次握手四次挥手
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/006.png)
### d.半关闭状态
通信的一方发送结束报文给对方，但仍允许接收数据，直到对方也发送了结束报文，这种状态就是半关闭状态
### e.TCP状态转移
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/007.png)
TIME_WAIT状态是一方收到另一方的结束报文时，不直接关闭，而是等待2个最大报文生存时间，保证连接可靠关闭以及迟到数据正确接收
