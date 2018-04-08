3 Requirements and considerations 要求和考虑
=================================

## 3.1 Transport agnostic 传输不可知

`libp2p` is transport agnostic, so it can run over any transport protocol. It does not even depend on IP; it may run on top of NDN, XIA, and other new Internet architectures.

`libp2p`是传输不可知的，所以它可以运行在任何传输协议上。 它甚至不依赖于IPPTV
; 它可能运行在NDN，XIA和其他新的互联网体系结构之上。

In order to reason about possible transports, `libp2p` uses [multiaddr](https://github.com/jbenet/multiaddr), a self-describing addressing format. This makes it possible for `libp2p` to treat addresses opaquely everywhere in the system, and have support for various transport protocols in the network layer. The actual format of addresses in `libp2p` is `ipfs-addr`, a multiaddr that ends with an IPFS node id. For example, these are all valid `ipfs-addrs`:

为了推断可能的传输，`libp2p`使用[multiaddr](https://github.com/jbenet/multiaddr)，这是一种自描述的寻址格式。 这使`libp2p`可以将系统中不同地址的地址对待，并且支持网络层中的各种传输协议。 `libp2p`中地址的实际格式是`ipfs-addr`，一个以IPFS节点ID结尾的multiaddr。 例如，这些都是有效的`ipfs-addrs`：

```
# IPFS over TCP over IPv6 (typical TCP)
/ip6/fe80::8823:6dff:fee7:f172/tcp/4001/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over uTP over UDP over IPv4 (UDP-shimmed transport)
/ip4/162.246.145.218/udp/4001/utp/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over IPv6 (unreliable)
/ip6/fe80::8823:6dff:fee7:f172/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over TCP over IPv4 over TCP over IPv4 (proxy)
/ip4/162.246.145.218/tcp/7650/ip4/192.168.0.1/tcp/4001/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu

# IPFS over Ethernet (no IP)
/ether/ac:fd:ec:0b:7c:fe/ipfs/QmYJyUMAcXEw1b5bFfbBbzYu5wyyjLMRHXGUkCXpag74Fu
```

**Note:** At this time, no unreliable implementations exist. The protocol's interface for defining and using unreliable transport has not been defined.

**注意：** 目前，不存在不可靠的实现。 尚未定义用于定义和使用不可靠传输的协议接口。

**TODO:** Define how unreliable transport would work. Base it on WebRTC.

**TODO:** 定义传输如何不可靠。 基于WebRTC。

## 3.2 Multi-multiplexing 多复用

The `libp2p` protocol is a collection of multiple protocols. In order to conserve resources, and to make connectivity easier, `libp2p` can perform all its operations through a single port, such as a TCP or UDP port, depending on the transports used. `libp2p` can multiplex its many protocols through point-to-point connections. This multiplexing is for both reliable streams and unreliable datagrams.

libp2p协议是多个协议的集合。 为了节省资源并使连接更容易，libp2p可以通过单个端口（如TCP或UDP端口）执行所有操作，具体取决于所使用的传输方式。 `libp2p`可以通过点对点连接复用其多种协议。 这种多路复用适用于可靠的数据流和不可靠的数据报文。

`libp2p` is pragmatic. It seeks to be usable in as many settings as possible, to be modular and flexible to fit various use cases, and to force as few choices as possible. Thus the `libp2p` network layer provides what we're loosely referring to as "multi-multiplexing":

`libp2p`是务实的。 它试图在尽可能多的设置中使用，以便模块化和灵活以适应各种使用情况，并尽可能地减少选择。 因此`libp2p`网络层提供了我们松散地提到的“多路复用”：

- can multiplex multiple listen network interfaces
- can multiplex multiple transport protocols
- can multiplex multiple connections per peer
- can multiplex multiple client protocols
- can multiplex multiple streams per protocol, per connection (SPDY, HTTP2, QUIC, SSH)
- has flow control (backpressure, fairness)
- encrypts each connection with a different ephemeral key

- 可以复用多个监听网络接口
- 可以复用多种传输协议
- 可以为每个peer复用多个连接
- 可以复用多个客户端协议
- 可以为每个连接（SPDY，HTTP2，QUIC，SSH）为每个协议复用多个流
- 有流量控制（背压，公平）
- 使用不同的临时密钥加密每个连接

To give an example, imagine a single IPFS node that:

举一个例子，设想一个IPFS节点：

- listens on a particular TCP/IP address
- listens on a different TCP/IP address
- listens on a SCTP/UDP/IP address
- listens on a UDT/UDP/IP address
- has multiple connections to another node X
- has multiple connections to another node Y
- has multiple streams open per connection
- multiplexes streams over HTTP2 to node X
- multiplexes streams over SSH to node Y
- one protocol mounted on top of `libp2p` uses one stream per peer
- one protocol mounted on top of `libp2p` uses multiple streams per peer

- 侦听特定的TCP/IP地址
- 侦听不同的TCP/IP地址
- 侦听SCTP/UDP/IP地址
- 侦听UDT/UDP/IP地址
- 与另一个节点X有多个连接
- 与另一个节点Y有多个连接
- 每个连接打开多个流
- 通过HTTP2将流复用到节点X
- 通过SSH将节点流多路复用到节点Y.
- 一个安装在`libp2p`上的协议每个peer使用一个流
- 安装在`libp2p`之上的一个协议使用每个peer的多个流

Not providing this level of flexbility makes it impossible to use `libp2p` in various platforms, use cases, or network setups. It is not important that all implementations support all choices; what is critical is that the spec is flexible enough to allow implementations to use precisely what they need. This ensures that complex user or application constraints do not rule out `libp2p` as an option.

不提供这种级别的灵活性使得在各种平台，用例或网络设置中不可能使用`libp2p`。 所有的实现都支持所有的选择并不重要; 关键在于该规范足够灵活，以允许实现恰好使用他们需要的东西。 这确保复杂的用户或应用程序约束不排除`libp2p`作为选项。

## 3.3 Encryption 加密

Communications on `libp2p` may be:

有关`libp2p`的通信可能是：

- **encrypted**
- **signed** (not encrypted)
- **clear** (not encrypted, not signed)

- **加密的**
- **签名的** (不加密的)
- **裸的** (不加密，也不签名)

We take both security and performance seriously. We recognize that encryption is not viable for some in-datacenter high performance use cases.

我们认真对待安全和性能。 我们认识到，对于某些数据中心高性能用例，加密不可行。

We recommend that:

我们建议：

- implementations encrypt all communications by default
- implementations are audited
- unless absolutely necessary, users normally operate with encrypted communications only.

- 实现默认情况下加密所有通信
- 审计实施
- 除非绝对必要，用户通常只使用加密通信。

`libp2p` uses cyphersuites like TLS.

`libp2p`使用像TLS这样的cyphersuites。

**Note:** We do not use TLS directly, because we do not want the CA system baggage. Most TLS implementations are very big. Since the `libp2p` model begins with keys, `libp2p` only needs to apply ciphers. This is a minimal portion of the whole TLS standard.

**注意：** 我们不直接使用TLS，因为我们不需要CA系统的行李。 大多数TLS实现非常大。 由于`libp2p`模型以键开头，`libp2p`只需要应用密码。 这是整个TLS标准的一小部分。

## 3.4 NAT traversal NAT穿越

Network Address Translation is ubiquitous in the Internet. Not only are most consumer devices behind many layers of NAT, but most data center nodes are often behind NAT for security or virtualization reasons. As we move into containerized deployments, this is getting worse. IPFS implementations SHOULD provide a way to traverse NATs, otherwise it is likely that operation will be affected. Even nodes meant to run with real IP addresses must implement NAT traversal techniques, as they may need to establish connections to peers behind NAT.

网络地址转换在互联网中无处不在。 不仅大多数消费类设备位于多层NAT之后，而且大多数数据中心节点出于安全性或虚拟化原因经常位于NAT之后。 当我们进入容器部署时，情况正在恶化。 IPFS实现应该提供一种穿越NAT的方式，否则操作可能会受到影响。 即使是以真实IP地址运行的节点也必须实现NAT穿越技术，因为它们可能需要建立到NAT后面的peer的连接。

`libp2p` accomplishes full NAT traversal using an ICE-like protocol. It is not exactly ICE, as IPFS networks provide the possibility of relaying communications over the IPFS protocol itself, for coordinating hole-punching or even relaying communication.

`libp2p`使用类似ICE的协议完成全面的NAT穿越。 这不完全是ICE，因为IPFS网络提供了通过IPFS协议本身中继通信的可能性，用于协调打孔甚至中继通信。

It is recommended that implementations use one of the many NAT traversal libraries available, such as `libnice`, `libwebrtc`, or `natty`. However, NAT traversal must be interoperable.

建议实现使用可用的许多NAT遍历库之一，例如`libnice`，`libwebrtc`或`natty`。 但是，NAT穿越必须能够互操作。

## 3.5 Relay 中继

Unfortunately, due to symmetric NATs, container and VM NATs, and other impossible-to-bypass NATs, `libp2p` MUST fallback to relaying communication to establish a full connectivity graph. To be complete, implementations MUST support relay, though it SHOULD be optional and able to be turned off by end users.

不幸的是，由于对称的NAT，容器和虚拟机NAT以及其他不可能绕过的NAT，`libp2p` 必须回退到中继通信以建立完整的连接图。 为了完整，实现必须支持中继，尽管它应该是可选的并且能够被最终用户关闭。

Connection relaying SHOULD be implemented as a transport, in order to be transparent to upper layers.

连接中继应当作为传输实现，以便对上层透明。

For an instantiation of relaying, see the [p2p-circuit transport](relay).

有关中继的实例，请参见 [p2p-circuit transport](relay)。


## 3.6 Enable several network topologies 启用多个网络拓扑

Different systems have different requirements and with that comes different topologies. In the P2P literature we can find these topologies being enumerated as: unstructured, structured, hybrid and centralized.

不同的系统有不同的要求，并且有不同的拓扑结构。 在P2P文献中，我们可以发现这些拓扑被列举为：非结构化的，结构化的，混合的和集中的。

Centralized topologies are the most common to find in Web Applications infrastructures, it requires for a given service or services to be present at all times in a known static location, so that other services can access them. Unstructured networks represent a type of P2P networks where the network topology is completely random, or at least non deterministic, while structured networks have a implicit way of organizing themselves. Hybrid networks are a mix of the last two.

集中式拓扑结构是Web应用程序基础结构中最常见的拓扑结构，它要求给定的服务或服务始终存在于已知的静态位置，以便其他服务可以访问它们。 非结构化网络代表了一种类型的P2P网络，其中网络拓扑结构完全是随机的，或者至少是非确定性的，而结构化网络具有隐含的组织方式。 混合网络是最后两个混合网络。

With this in consideration, `libp2p` must be ready to perform different routing mechanisms and peer discovery, in order to build the routing tables that will enable services to propagate messages or to find each other.

考虑到这一点，`libp2p`必须准备好执行不同的路由机制和peer发现，以便构建路由表，以便使服务能够传播消息或查找对方。

## 3.7 Resource discovery 资源发现

`libp2p` also solves the problem with discoverability of resources inside of a network through *records*.  A record is a unit of data that can be digitally signed, timestamped and/or used with other methods to give it an ephemeral validity. These records hold pieces of information such as location or availability of resources present in the network. These resources can be data, storage, CPU cycles and other types of services.

`libp2p`还通过*records*解决了网络内部资源可发现性的问题。 记录是可以数字签名，加盖时间戳和/或与其他方法一起使用以使其具有短暂有效性的数据单位。 这些记录包含网络中存在的资源位置或可用性等信息。 这些资源可以是数据，存储，CPU周期和其他类型的服务。

`libp2p` must not put a constraint on the location of resources, but instead offer ways to find them easily in the network or use a side channel.

`libp2p`不能限制资源的位置，而是提供方法在网络中轻松找到它们或使用辅助通道。

## 3.8 Messaging 消息

Efficient messaging protocols offer ways to deliver content with minimum latency and/or support large and complex topologies for distribution. `libp2p` seeks to incorporate the developments made in Multicast and PubSub to fulfil these needs.

高效的消息传递协议提供了以最短延迟交付内容和/或支持大型和复杂拓扑分发的方法。 `libp2p`试图结合Multicast和PubSub中的开发来满足这些需求。

## 3.9 Naming 命名

Networks change and applications need to have a way to use the network in such a way that it is agnostic to its topology, naming appears to solve this issues.

网络变化和应用程序需要有一种方式来使用网络，使其不受其拓扑结构的影响，命名似乎可以解决这个问题。
