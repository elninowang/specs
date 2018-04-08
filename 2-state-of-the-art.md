2 分析网络堆栈技术状态
====================================================

This section presents to the reader an analysis of the available protocols and architectures for network stacks. The goal is to provide the foundations from which to infer the conclusions and understand why `libp2p` has the requirements and architecture that it has.

本节向读者介绍网络堆栈的可用协议和体系结构。 目标是为推断结论提供基础，并理解为什么`libp2p`具有它的要求和体系结构。

## 2.1 The client-server model 客户端-服务器模型

The client-server model indicates that both parties at the ends of the channel have different roles, that they support different services and/or have different capabilities, or in other words, that they speak different protocols.

客户端-服务器模型表明，信道末端的双方具有不同的角色，支持不同的服务和/或具有不同的能力，或换句话说，他们说不同的协议。

Building client-server applications has been the natural tendency for a number of reasons:

构建客户端 - 服务器应用程序一直是很多原因的自然趋势：

- The bandwidth inside a data center is considerably higher than that available for clients to connect to each other.
- Data center resources are considerably cheaper, due to efficient usage and bulk stocking.
- It makes it easier for the developer and system admin to have fine grained control over the application.
- It reduces the number of heterogeneous systems to be handled (although the number is still considerable).
- Systems like NAT make it really hard for client machines to find and talk with each other, forcing a developer to perform very clever hacks to traverse these obstacles.
- Protocols started to be designed with the assumption that a developer will create a client-server application from the start.

- 数据中心内的带宽远远高于客户端互相连接的带宽。
- 由于高效使用和大量库存，数据中心资源相当便宜。
- 它使开发人员和系统管理员可以更轻松地对应用程序进行细粒度的控制。
- 它减少了要处理的异构系统的数量（尽管数量仍然相当可观）。
- 像NAT这样的系统让客户端机器很难找到并彼此交谈，迫使开发人员执行非常聪明的黑客来穿越这些障碍。
- 协议开始设计的假设是开发人员将从一开始就创建一个客户端-服务器应用程序。

We even learned how to hide all the complexity of a distributed system behind gateways on the Internet, using protocols that were designed to perform a point-to-point operation, such as HTTP, making it opaque for the application to see and understand the cascade of service calls made for each request.

我们甚至还学会了如何使用旨在执行点对点操作的协议（如HTTP）来隐藏Internet上的网关后面的分布式系统的所有复杂性，这使得应用程序看不到并理解级联 为每个请求所做的服务调用。

`libp2p` offers a move towards dialer-listener interactions, from the client-server listener, where it is not implicit which of the entities, dialer or listener, has which capabilities or is enabled to perform which actions. Setting up a connection between two applications today is a multilayered problem to solve, and these connections should not have a purpose bias, and should instead support several other protocols to work on top of the established connection. In a client-server model, a server sending data without a prior request from the client is known as a push model, which typically adds more complexity; in a dialer-listener model in comparison, both entities can perform requests independently.

`libp2p`为客户端-服务器监听器提供了一个 拨号器-监听器 交互的过程，在这个过程中，并不隐含哪个实体，拨号器或监听器具有哪些功能或能够执行哪些操作。 现在在两个应用程序之间建立连接是一个需要解决的多层次问题，这些连接不应该有目的偏见，而应该支持其他几个协议在已建立的连接之上工作。 在 客户端-服务器 模型中，发送数据的服务器没有来自客户端的先前请求被称为推送模型，其通常增加更多的复杂性; 在比较的拨号收听者模型中，两个实体都可以独立地执行请求。

## 2.2 Categorizing the network stack protocols by solutions 通过解决方案对网络堆栈协议进行分类

Before diving into the `libp2p` protocols, it is important to understand the large diversity of protocols already in wide use and deployment that help maintain today's simple abstractions. For example, when one thinks about an HTTP connection, one might naively just think that HTTP/TCP/IP are the main protocols involved, but in reality many more protocols participate, depending on the usage, the networks involved, and so on. Protocols like DNS, DHCP, ARP, OSPF, Ethernet, 802.11 (Wi-Fi) and many others get involved. Looking inside ISPs' own networks would reveal dozens more.

在深入研究 `libp2p`协议之前，了解已经广泛使用和部署的大量协议以帮助维护当今简单的抽象是很重要的。 例如，当人们考虑HTTP连接时，可能天真地认为HTTP/TCP/IP是涉及的主要协议，但实际上有更多的协议参与其中，这取决于使用情况，涉及的网络等等。 像DNS，DHCP，ARP，OSPF，以太网，802.11（Wi-Fi）等许多协议都会涉及到。 看看ISPs自己的网络会发现数十个。

Additionally, it's worth noting that the traditional 7-layer OSI model characterization does not fit `libp2p`. Instead, we categorize protocols based on their role, i.e. the problem they solve. The upper layers of the OSI model are geared towards point-to-point links between applications, whereas the `libp2p` protocols speak more towards various sizes of networks, with various properties, under various different security models. Different `libp2p` protocols can have the same role (in the OSI model, this would be "address the same layer"), meaning that multiple protocols can run simultaneously, all addressing one role (instead of one-protocol-per-layer in traditional OSI stacking). For example, bootstrap lists, mDNS, DHT discovery, and PEX are all forms of the role "Peer Discovery"; they can coexist and even synergize.

另外，值得注意的是传统的7层OSI模型特性不适合`libp2p`。 相反，我们根据协议的角色对协议进行分类，即他们解决的问题。 OSI模型的上层适用于应用程序之间的点对点链接，而“libp2p”协议更多地针对各种不同规模的网络，以及各种不同的属性，在各种不同的安全模型下。 不同的`libp2p`协议可以具有相同的作用（在OSI模型中，这将是“寻址同一层”），这意味着多个协议可以同时运行，所有这些协议都可以处理一个角色（而不是每层一个协议 传统的OSI堆叠）。 例如，引导程序列表，mDNS，DHT发现和PEX都是角色“peer发现”的所有形式; 它们可以共存甚至协同作用。

### 2.2.1 Establishing the physical link 建立物理链接

- Ethernet
- Wi-Fi
- Bluetooth
- USB

### 2.2.2 Addressing a machine or process 定位机器或过程

- IPv4
- IPv6
- Hidden addressing, like SDP

### 2.2.3 Discovering other peers or services 在服务找到其他peer

- ARP
- DHCP
- DNS
- Onion

### 2.2.4 Routing messages through the network 在网络上路由消息

- RIP(1, 2)
- OSPF
- BGP
- PPP
- Tor
- I2P
- cjdns

### 2.2.5 Transport 传输协议

- TCP
- UDP
- UDT
- QUIC
- WebRTC data channel

### 2.2.6 Agreed semantics for applications to talk to each other 商定应用程序相互交谈的语义

- RMI
- Remoting
- RPC
- HTTP

## 2.3 Current shortcomings 目前的缺点

Although we currently have a panoply of protocols available for our services to communicate, the abundance and variety of solutions creates its own problems. It is currently difficult for an application to be able to support and be available through several transports (e.g. the lack of TCP/UDP stack in browser applications).

虽然我们目前有一整套协议可供我们的服务进行通信，但丰富多样的解决方案会造成它自己的问题。 目前，应用程序难以支持并可通过多种传输方式（例如浏览器应用程序中缺少TCP/UDP堆栈）。

There is also no 'presence linking', meaning that there isn't a notion for a peer to announce itself in several transports, so that other peers can guarantee that it is always the same peer.

也没有 'presence linking'，这意味着没有一个peer在多个传输中宣布自己的概念，以便其他peer可以保证它始终是同一个peer。
