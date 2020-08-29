Linux高性能服务器学习
=
## 一.TCP/IP协议
### a.协议层次<br>
1.TCP、IP协议是一个四层的结构：<br>
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/001.png)
2.数据链路层，常用的协议是ARP协议和RARP协议，实现了IP地址和物理地址之间的转化<br>
3.网络层，常用协议有ICMP、IP<br>
4.传输层，常用协议有TCP、UDP<br>
5.应用层，不举例<br>
### b.协议封装<br>
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/002.png)
## 二.IP协议
### a.IPV4<br>
1.头部结构：<br>
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
- 最多40位的扩展信息<br>

2.IP分片<br>
IP报文长度超过数据帧的MTU时，分片；MTU为1500字节，因此携带的最多数据为1480字节<br>
3.IP路由<br>
route命令查看路由表<br>
### b.IPV6
1.头部结构：<br>
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/004.png)
- 4位版本号固定为6；
- 8位通信类型类似于IPV4的TOS；
- 20位标签流，指示数据优先级，如实时视频；
- 16位净荷长度，指扩展头部和数据部分的长度；
- 8位下一个包头，指出紧跟在固定头部后的包头类型，如扩展头部或上层协议的包头，类似于IPV4的协议字段；
- 8位跳数限制，类似于TTL；
- 128位源地址；
- 128位目的地址；<br>
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
- 16位紧急指针，是一个正的偏移量，是当前报文紧急数据存放处的偏移的下一字节；<br>
### c.三次握手四次挥手
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/006.png)
### d.半关闭状态
通信的一方发送结束报文给对方，但仍允许接收数据，直到对方也发送了结束报文，这种状态就是半关闭状态<br>
### e.TCP状态转移
![](https://github.com/CodeDrugger/HPLSP/raw/master/pic/007.png)
TIME_WAIT状态是一方收到另一方的结束报文时，不直接关闭，而是等待2个最大报文生存时间，保证连接可靠关闭以及迟到数据正确接收<br>
### f.带外数据
TCP没有真正的带外数据，可以通过紧急指针实现，带外数据只有1字节<br>
### g.超时重传
Linux发送端没收到确认报文时，触发超时重传，每次的间隔时间是上次间隔时间的一倍（从0.2s开始），重传都失败后，IP和ARP协议开始接管，直到发送方主动放弃<br>
重传相关的内核参数为：/proc/sys/net/ipv4/tcp_retries1和/proc/sys/net/ipv4/tcp_retries2，前者确定底层协议接管前的重试次数，后者确定放弃前的重试次数<br>
### h.拥塞控制
拥塞控制包括慢启动、拥塞避免、快速重传、快速恢复<br>
1.慢启动和拥塞避免<br>
- SWND：发送窗口，指一次发送中发送端写入的数据量，该值为RWND和CWND的较小者<br>
- RWND：接收通告窗口<br>
- CWND：拥塞窗口，发送端的控制变量<br>

TCP连接建立好后，CWND被设置为初始值，SWND=CWND，发送后收到确认报文段后CWND=min(N,SMSS)，其中N为确认的报文段的长度，SMSS为发送端最大段大小，初始阶段窗口大小指数增长；<br>
当CWND达到ssthresh后，慢启动结束，进入拥塞控制阶段；<br>
拥塞控制阶段每次收到确认报文段时，CWND+=1，此为拥塞避免<br>
2.快速重传和快速恢复<br>
快速重传根据收到的确认报文判断，如果收到3个重复的确认报文，就立即重传，而不是等到计时器超时才重传；<br>
> 举例：发送端发送了1,2,3三个报文段，3丢失，接收方回复了1,2的确认报文，发送方接着发送了4,5,6报文，发送方仍然回复了3个2报文的确认报文，这时发送端快速重传3，接收方回复6的接收报文<br>

快速恢复即将ssthresh减为CWND的一半（Reno），CWND=ssthresh，重新开始拥塞控制<br>
## Linux网络编程基础API
### a.字节序转换
``` C++
#include <netinet/in.h>
unsigned long int htonl(unsigned long int hostlong); //host to network long
unsigned long int ntohl(unsigned long int netlong); //network to host long
```
### b.TCP专用地址结构体
``` C++
struct sockaddr_in {
short sin_family; //Address family一般来说AF_INET（地址族）PF_INET（协议族）
unsigned short sin_port; //Port number(必须要采用网络数据格式,普通数字可以用htons()函数转换成网络数据格式的数字)
struct in_addr sin_addr; //IP address in network byte order（Internet address）
unsigned char sin_zero[8]; //Same size as struct sockaddr没有实际意义,只是为了　跟SOCKADDR结构在内存中对齐
};

struct in_addr {
u_int32_t s_addr;
};
```
### c.点分十进制和整数的转化
``` C++
#include <arpa/inet.h>
in_addr_t inet_addr(const char* strptr) //点分十进制转化为整数
int inet_aton(const char* cp, struct in_addr* inp) //点分十进制转化为整数，结果放在inp中，成功返回1，失败返回0
char* inet_ntoa(struct in_addr in) //整型转化为点分十进制
```
### d.创建socket
``` C++
#include <sys/types.h>
#include <sys/socket.h>
/*
domain：使用哪个底层协议PF_INET IPV4 PF_INET6 IPV6
type：使用哪个服务类型，SOCK_STREAM流服务 SOCK_UGRAM数据报
type接受上述值与SOCK_NONBLOCK（非阻塞） SOCK_CLOEXEC（使用fork创建子进程时关闭该socket）相与
protocol默认0
失败返回-1
*/
int socket(int domain, int type, int protocol) 
```
