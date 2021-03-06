# 传输层 TCP UDP

## ISO七层模型中表示层和会话层功能是什么？

ISOOSI七层网络模型：

<p align="center">
    <img width="70%" height="70%" src="http://images.iterate.site/blog/image/20200523/W5Vq5kMIzcDH.png?imageslim">
</p>


- 表示层：图像、视频编码解，数据加密。
- 会话层：建立会话，如session认证、断点续传。

## 描述TCP头部？

- 序号（32bit）：传输方向上字节流的字节编号。初始时序号会被设置一个随机的初始值（ISN），之后每次发送数据时，序号值 = ISN + 数据在整个字节流中的偏移。假设A -> B且ISN = 1024，第一段数据512字节已经到B，则第二段数据发送时序号为1024 + 512。用于解决网络包乱序问题。
- 确认号（32bit）：接收方对发送方TCP报文段的响应，其值是收到的序号值 + 1。
- 首部长（4bit）：标识首部有多少个4字节 * 首部长，最大为15，即60字节。
- 标志位（6bit）：
    - URG：标志紧急指针是否有效。
    - ACK：标志确认号是否有效（确认报文段）。用于解决丢包问题。
    - PSH：提示接收端立即从缓冲读走数据。
    - RST：表示要求对方重新建立连接（复位报文段）。
    - SYN：表示请求建立一个连接（连接报文段）。
    - FIN：表示关闭连接（断开报文段）。
- 窗口（16bit）：接收窗口。用于告知对方（发送方）本方的缓冲还能接收多少字节数据。用于解决流控。
- 校验和（16bit）：接收端用CRC检验整个报文段有无损坏。

## 三次握手过程？

- 第一次：客户端发含SYN位，SEQ_NUM = S的包到服务器。（客 -> SYN_SEND）
- 第二次：服务器发含ACK，SYN位且ACK_NUM = S + 1，SEQ_NUM = P的包到客户机。（服 -> SYN_RECV）
- 第三次：客户机发送含ACK位，ACK_NUM = P + 1的包到服务器。（客 -> ESTABLISH，服 -> ESTABLISH）

## 四次挥手过程？


- 第一次：客户机发含FIN位，SEQ = Q的包到服务器。（客 -> FIN_WAIT_1）
- 第二次：服务器发送含ACK且ACK_NUM = Q + 1的包到服务器。（服 -> CLOSE_WAIT，客 -> FIN_WAIT_2）
    - 此处有等待
- 第三次：服务器发送含FIN且SEQ_NUM = R的包到客户机。（服 -> LAST_ACK，客 -> TIME_WAIT）
    - 此处有等待
- 第四次：客户机发送最后一个含有ACK位且ACK_NUM = R + 1的包到客户机。（服 -> CLOSED）

## 为什么握手是三次，挥手是四次？


- 对于握手：握手只需要确认双方通信时的初始化序号，保证通信不会乱序。（第三次握手必要性：假设服务端的确认丢失，连接并未断开，客户机超时重发连接请求，这样服务器会对同一个客户机保持多个连接，造成资源浪费。）
- 对于挥手：TCP是双工的，所以发送方和接收方都需要FIN和ACK。只不过有一方是被动的，所以看上去就成了4次挥手。

6. TCP连接状态？
    - CLOSED：初始状态。

    - LISTEN：服务器处于监听状态。

    - SYN_SEND：客户端socket执行CONNECT连接，发送SYN包，进入此状态。

    - SYN_RECV：服务端收到SYN包并发送服务端SYN包，进入此状态。

    - ESTABLISH：表示连接建立。客户端发送了最后一个ACK包后进入此状态，服务端接收到ACK包后进入此状态。

    - FIN_WAIT_1：终止连接的一方（通常是客户机）发送了FIN报文后进入。等待对方FIN。

    - CLOSE_WAIT：（假设服务器）接收到客户机FIN包之后等待关闭的阶段。在接收到对方的FIN包之后，自然是需要立即回复ACK包的，表示已经知道断开请求。但是本方是否立即断开连接（发送FIN包）取决于是否还有数据需要发送给客户端，若有，则在发送FIN包之前均为此状态。

    - FIN_WAIT_2：此时是半连接状态，即有一方要求关闭连接，等待另一方关闭。客户端接收到服务器的ACK包，但并没有立即接收到服务端的FIN包，进入FIN_WAIT_2状态。

    - LAST_ACK：服务端发动最后的FIN包，等待最后的客户端ACK响应，进入此状态。

    - TIME_WAIT：客户端收到服务端的FIN包，并立即发出ACK包做最后的确认，在此之后的2MSL时间称为TIME_WAIT状态。

7. 解释FIN_WAIT_2，CLOSE_WAIT状态和TIME_WAIT状态？
    - FIN_WAIT_2：
        - 半关闭状态。

        - 发送断开请求一方还有接收数据能力，但已经没有发送数据能力。

    - CLOSE_WAIT状态：
        - 被动关闭连接一方接收到FIN包会立即回应ACK包表示已接收到断开请求。

        - 被动关闭连接一方如果还有剩余数据要发送就会进入CLOSED_WAIT状态。

    - TIME_WAIT状态：
        - 又叫2MSL等待状态。

        - 如果客户端直接进入CLOSED状态，如果服务端没有接收到最后一次ACK包会在超时之后重新再发FIN包，此时因为客户端已经CLOSED，所以服务端就不会收到ACK而是收到RST。所以TIME_WAIT状态目的是防止最后一次握手数据没有到达对方而触发重传FIN准备的。

        - 在2MSL时间内，同一个socket不能再被使用，否则有可能会和旧连接数据混淆（如果新连接和旧连接的socket相同的话）。

8. 解释RTO，RTT和超时重传？
    - 超时重传：发送端发送报文后若长时间未收到确认的报文则需要重发该报文。可能有以下几种情况：

        - 发送的数据没能到达接收端，所以对方没有响应。

        - 接收端接收到数据，但是ACK报文在返回过程中丢失。

        - 接收端拒绝或丢弃数据。

    - RTO：从上一次发送数据，因为长期没有收到ACK响应，到下一次重发之间的时间。就是重传间隔。
        - 通常每次重传RTO是前一次重传间隔的两倍，计量单位通常是RTT。例：1RTT，2RTT，4RTT，8RTT......

        - 重传次数到达上限之后停止重传。

    - RTT：数据从发送到接收到对方响应之间的时间间隔，即数据报在网络中一个往返用时。大小不稳定。

9. 流量控制原理？
    - 目的是接收方通过TCP头窗口字段告知发送方本方可接收的最大数据量，用以解决发送速率过快导致接收方不能接收的问题。所以流量控制是点对点控制。

    - TCP是双工协议，双方可以同时通信，所以发送方接收方各自维护一个发送窗和接收窗。

        - 发送窗：用来限制发送方可以发送的数据大小，其中发送窗口的大小由接收端返回的TCP报文段中窗口字段来控制，接收方通过此字段告知发送方自己的缓冲（受系统、硬件等限制）大小。

        - 接收窗：用来标记可以接收的数据大小。

    - TCP是流数据，发送出去的数据流可以被分为以下四部分：已发送且被确认部分 | 已发送未被确认部分 | 未发送但可发送部分 | 不可发送部分，其中发送窗 = 已发送未确认部分 + 未发但可发送部分。接收到的数据流可分为：已接收 | 未接收但准备接收 | 未接收不准备接收。接收窗 = 未接收但准备接收部分。

    - 发送窗内数据只有当接收到接收端某段发送数据的ACK响应时才移动发送窗，左边缘紧贴刚被确认的数据。接收窗也只有接收到数据且最左侧连续时才移动接收窗口。

10. 拥塞控制原理？
    - 拥塞控制目的是防止数据被过多注网络中导致网络资源（路由器、交换机等）过载。因为拥塞控制涉及网络链路全局，所以属于全局控制。控制拥塞使用拥塞窗口。

    - TCP拥塞控制算法：
        - 慢开始 & 拥塞避免：先试探网络拥塞程度再逐渐增大拥塞窗口。每次收到确认后拥塞窗口翻倍，直到达到阀值ssthresh，这部分是慢开始过程。达到阀值后每次以一个MSS为单位增长拥塞窗口大小，当发生拥塞（超时未收到确认），将阀值减为原先一半，继续执行线性增加，这个过程为拥塞避免。

        - 快速重传 & 快速恢复：略。

        - 最终拥塞窗口会收敛于稳定值。

11. 如何区分流量控制和拥塞控制？
    - 流量控制属于通信双方协商；拥塞控制涉及通信链路全局。

    - 流量控制需要通信双方各维护一个发送窗、一个接收窗，对任意一方，接收窗大小由自身决定，发送窗大小由接收方响应的TCP报文段中窗口值确定；拥塞控制的拥塞窗口大小变化由试探性发送一定数据量数据探查网络状况后而自适应调整。

    - 实际最终发送窗口 = min{流控发送窗口，拥塞窗口}。

12. TCP如何提供可靠数据传输的？
    - 建立连接（标志位）：通信前确认通信实体存在。

    - 序号机制（序号、确认号）：确保了数据是按序、完整到达。

    - 数据校验（校验和）：CRC校验全部数据。

    - 超时重传（定时器）：保证因链路故障未能到达数据能够被多次重发。

    - 窗口机制（窗口）：提供流量控制，避免过量发送。

    - 拥塞控制：同上。

13. TCP soctet交互流程？
    - 服务器：
        - 创建socket -> int socket(int domain, int type, int protocol);
            - domain：协议域，决定了socket的地址类型，IPv4为AF_INET。

            - type：指定socket类型，SOCK_STREAM为TCP连接。

            - protocol：指定协议。IPPROTO_TCP表示TCP协议，为0时自动选择type默认协议。

        - 绑定socket和端口号 -> int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
            - sockfd：socket返回的套接字描述符，类似于文件描述符fd。

            - addr：有个sockaddr类型数据的指针，指向的是被绑定结构变量。
            ```c++
                // IPv4的sockaddr地址结构
                struct sockaddr_in {
                    sa_family_t sin_family;    // 协议类型，AF_INET
                    in_port_t sin_port;    // 端口号
                    struct in_addr sin_addr;    // IP地址
                };
                struct in_addr {
                    uint32_t s_addr;
                }
            ```

            - addrlen：地址长度。

        - 监听端口号 -> int listen(int sockfd, int backlog);
            - sockfd：要监听的sock描述字。

            - backlog：socket可以排队的最大连接数。

        - 接收用户请求 -> int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
            - sockfd：服务器socket描述字。

            - addr：指向地址结构指针。

            - addrlen：协议地址长度。

            - 注：一旦accept某个客户机请求成功将返回一个全新的描述符用于标识具体客户的TCP连接。

        - 从socket中读取字符 -> ssize_t read(int fd, void *buf, size_t count);
            - fd：连接描述字。

            - buf：缓冲区buf。

            - count：缓冲区长度。

            - 注：大于0表示读取的字节数，返回0表示文件读取结束，小于0表示发生错误。

        - 关闭socket -> int close(int fd);

            - fd：accept返回的连接描述字，每个连接有一个，生命周期为连接周期。

            - 注：sockfd是监听描述字，一个服务器只有一个，用于监听是否有连接；fd是连接描述字，用于每个连接的操作。

    - 客户机：
        - 创建socket -> int socket(int domain, int type, int protocol);

        - 连接指定计算机 -> int connect(int sockfd, struct sockaddr* addr, socklen_t addrlen);
            - sockfd客户端的sock描述字。

            - addr：服务器的地址。

            - addrlen：socket地址长度。

        - 向socket写入信息 -> ssize_t write(int fd, const void *buf, size_t count);
            - fd、buf、count：同read中意义。

            - 大于0表示写了部分或全部数据，小于0表示出错。

        - 关闭oscket -> int close(int fd);
            - fd：同服务器端fd。
