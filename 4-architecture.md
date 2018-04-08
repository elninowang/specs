4 Architecture 架构
==============

`libp2p` was designed around the Unix Philosophy of creating small components that are easy to understand and test. These components should also be able to be swapped in order to accommodate different technologies or scenarios and also make it feasible to upgrade them over time.

`libp2p`是围绕Unix哲学设计的，它创建了易于理解和测试的小型组件。 这些组件也应该能够交换以适应不同的技术或场景，并且随着时间的推移可以升级它们。

Although different peers can support different protocols depending on their capabilities, any peer can act as a dialer and/or a listener for connections from other peers, connections that once established can be reused from both ends, removing the distinction between clients and servers.

虽然不同的peer可以根据其能力支持不同的协议，但任何peer都可以充当来自其他peer的连接的拨号程序和/或侦听程序，一旦建立的连接就可以从两端重新使用，消除客户端和服务器之间的区别。

The `libp2p` interface acts as a thin veneer over a multitude of subsystems that are required in order for peers to be able to communicate. These subsystems are allowed to be built on top of other subsystems as long as they respect the standardized interface. The main areas where these subsystems fit are:

`libp2p` 接口在许多需要的子系统上扮演着薄弱的角色，以便于peer进行通信。 只要尊重标准化接口，这些子系统就可以建立在其他子系统之上。 这些子系统适合的主要领域是：

- Peer Routing - Mechanism to decide which peers to use for routing particular messages. This routing can be done recursively, iteratively or even in a broadcast/multicast mode.
- Swarm - Handles everything that touches the 'opening a stream' part of `libp2p`, from protocol muxing, stream muxing, NAT traversal and connection relaying, while being multi-transport.
- Distributed Record Store - A system to store and distribute records. Records are small entries used by other systems for signaling, establishing links, announcing peers or content, and so on. They have a similar role to DNS in the broader Internet.
- Discovery - Finding or identifying other peers in the network.

- Peer路由 - 决定使用哪些对等体来路由特定消息的机制。 这种路由可以递归地，迭代地或甚至以广播/多播模式进行。
- Swarm - 处理`libp2p`的'打开流'部分的所有内容，包括协议复用，流复用，NAT遍历和连接中继，同时是多传输。
- 分布式记录存储 - 用于存储和分发记录的系统。 记录是其他系统用于发送信号，建立链接，通告同行或内容等的小条目。 它们在更广泛的互联网中与DNS有相似的作用。
- 发现 - 查找或识别网络中的其他对等点。

Each of these subsystems exposes a well known interface (see [chapter 6](6-interfaces.md) for Interfaces) and may use each other in order to fulfill their goal. A global overview of the system is:

这些子系统中的每一个都公开了一个众所周知的接口（见接口的[chapter 6](6-interfaces.md)），并且可以相互使用以实现其目标。 系统的全局概述是：

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                  libp2p                                         │
└─────────────────────────────────────────────────────────────────────────────────┘
┌─────────────────┐┌─────────────────┐┌──────────────────────────┐┌───────────────┐
│   Peer Routing  ││      Swarm      ││ Distributed Record Store ││  Discovery    │
└─────────────────┘└─────────────────┘└──────────────────────────┘└───────────────┘
```

## 4.1 Peer Routing Peer路由

A Peer Routing subsystem exposes an interface to identify which peers a message should be routed to in the DHT. It receives a key and must return one or more `PeerInfo` objects.

Peer路由子系统公开了一个接口来识别消息应该被路由到DHT中的哪些Peer。 它接收一个键并且必须返回一个或多个`PeerInfo`对象。

We present two examples of possible Peer Routing subsystems, the first based on a the Kademlia DHT and the second based on mDNS. Nevertheless, other Peer Routing mechanisms can be implemented, as long as they fulfil the same expectation and interface.

我们提出了两个可能的Peer路由子系统的例子，第一个基于Kademlia DHT，第二个基于mDNS。 尽管如此，只要它们实现相同的期望和接口，其他对等路由机制也可以实现。

```
┌──────────────────────────────────────────────────────────────┐
│       Peer Routing                                           │
│                                                              │
│┌──────────────┐┌────────────────┐┌──────────────────────────┐│
││ kad-routing  ││ mDNS-routing   ││ other-routing-mechanisms ││
││              ││                ││                          ││
││              ││                ││                          ││
│└──────────────┘└────────────────┘└──────────────────────────┘│
└──────────────────────────────────────────────────────────────┘
```

### 4.1.1 kad-routing

kad-routing implements the Kademlia Routing table, where each peer holds a set of k-buckets, each of them containing several `PeerInfo` objects from other peers in the network.

kad-routing实现了Kademlia路由表，其中每个peer拥有一组k-bucket，每个k-bucket包含来自网络中其他peers的多个`PeerInfo`对象。

### 4.1.2 mDNS-routing

mDNS-routing uses mDNS probes to identify if local area network peers have a given key or they are simply present.

mDNS路由使用mDNS探测来识别局域网peers是否具有给定密钥或者它们是否仅存在。

## 4.2 Swarm

### 4.2.1 Stream Muxer 流复用器

The stream muxer must implement the interface offered by [interface-stream-muxer](https://github.com/diasdavid/interface-stream-muxer).

流复用器必须实现[interface-stream-muxer](https://github.com/diasdavid/interface-stream-muxer)提供的接口。

### 4.2.2 Protocol Muxer 协议复用器

Protocol muxing is handled on the application level instead of the conventional way at the port level (where different services/protocols listen at different ports). This enables us to support several protocols to be muxed in the same socket, saving the cost of doing NAT traversal for more than one port.

协议混合是在应用层面上处理的，而不是在端口层面（不同的服务/协议在不同的端口监听）的传统方式。 这使我们能够支持将多个协议混合在同一个套接字中，从而节省了为多个端口进行NAT穿越的成本。

Protocol multiplexing is done through [`multistream`](https://github.com/jbenet/multistream), a protocol to negotiate different types of streams (protocols) using [`multicodec`](https://github.com/jbenet/multicodec).

协议复用是通过 [`multistream`](https://github.com/jbenet/multistream) 完成的，一种使用[`multicodec`](https://github.com/jbenet/multicodec)协商不同类型的流（协议）的协议。

### 4.2.3 Transport 传输

### 4.2.4 Crypto 加密

### 4.2.5 Identify 识别

**Identify** is one of the protocols mounted on top of Swarm, our Connection handler. However, it follows and respects the same pattern as any other protocol when it comes to mounting it on top of Swarm. Identify enables us to trade `listenAddrs` and `observedAddrs` between peers, which is crucial for the working of IPFS. Since every socket open implements `REUSEPORT`, an `observedAddr` by another peer can enable a third peer to connect to us, since the port will be already open and redirect to us on a NAT.

**Identify** 是安装在我们的连接处理程序Swarm之上的协议之一。 但是，它将它安装在Swarm之上时，遵循和遵守与其他协议相同的模式。 识别使我们能够在peer之间交易 `listenAddrs`和`observedAddrs`，这对于IPFS的工作至关重要。 由于每个打开的socket都实现`REUSEPORT`，所以另一个peer的`observedAddr`可以让第三个peer连接到我们，因为这个端口已经打开并且在NAT上重定向到我们。

### 4.2.6 Relay 中继

See [Circuit Relay](/relay).

参见 [Circuit Relay](/relay).

## 4.3 Distributed Record Store 分布式记录存储

### 4.3.1 Record

Follows [IPRS spec](https://github.com/ipfs/specs/tree/master/iprs).

遵循  [IPRS规范](https://github.com/ipfs/specs/tree/master/iprs)。

### 4.3.2 abstract-record-store

### 4.3.3 kad-record-store

### 4.3.4 mDNS-record-store

### 4.3.5 s3-record-store

## 4.4 Discovery 发现

### 4.4.1 mDNS-discovery mDNS-发现

mDNS-discovery is a Discovery Protocol that uses [mDNS](https://en.wikipedia.org/wiki/Multicast_DNS) over local area networks. It emits mDNS beacons to find if there are more peers available. Local area network peers are very useful to peer-to-peer protocols, because of their low latency links.

mDNS发现是一种发现协议，它通过局域网使用[mDNS](https://en.wikipedia.org/wiki/Multicast_DNS)。 它发出mDNS信标来查找是否有更多的同级可用。 局域网对等体对于对等协议非常有用，因为它们的低延迟链路。

mDNS-discovery is a standalone protocol and does not depend on any other `libp2p` protocol. mDNS-discovery can yield peers available in the local area network, without relying on other infrastructure. This is particularly useful in intranets, networks disconnected from the Internet backbone, and networks which temporarily lose links.

mDNS发现是一个独立的协议，不依赖于任何其他的`libp2p` 协议。 mDNS发现可以产生局域网中可用的peers，而不依赖于其他基础设施。 这在Intranet，与Internet骨干网断开的网络以及临时丢失链接的网络中特别有用。

mDNS-discovery can be configured per-service (i.e. discover only peers participating in a specific protocol, like IPFS), and with private networks (discover peers belonging to a private network).

mDNS发现可配置为每个服务（即 仅发现参与特定协议的peers，如IPFS），并与私有网络（发现属于专用网络的peers）进行配置。

We are exploring ways to make mDNS-discovery beacons encrypted (so that other nodes in the local network cannot discern what service is being used), though the nature of mDNS will always reveal local IP addresses.

我们正在探索加密mDNS发现信标的方法（以使本地网络中的其他节点无法辨别正在使用的服务），尽管mDNS的本质将始终显示本地IP地址。

**Privacy note:** mDNS advertises in local area networks, which reveals IP addresses to listeners in the same local network. It is not recommended to use this with privacy-sensitive applications or oblivious routing protocols.

**隐私注意事项：** mDNS在局域网中发布广告，向同一本地网络中的侦听器显示IP地址。 不建议将此与隐私敏感的应用程序或不经意的路由协议一起使用。

#### 4.4.2 random-walk 随机行走

Random-Walk is a Discovery Protocol for DHTs (and other protocols with routing tables). It makes random DHT queries in order to learn about a large number of peers quickly. This causes the DHT (or other protocols) to converge much faster, at the expense of a small load at the very beginning.

随机游走是DHT（和其他具有路由表的协议）的发现协议。 它使随机的DHT查询，以便快速了解大量的peer。 这会导致DHT（或其他协议）以更快的速度收敛，代价是一开始就有小负载。

#### 4.4.3 bootstrap-list 引导列表

Bootstrap-List is a Discovery Protocol that uses local storage to cache the addresses of highly stable (and somewhat trusted) peers available in the network. This allows protocols to "find the rest of the network". This is essentially the same way that DNS bootstraps itself (though note that changing the DNS bootstrap list -- the "dot domain" addresses -- is not easy to do, by design).

Bootstrap-List是一种发现协议，它使用本地存储来缓存网络中可用的高度稳定（并且可信）peers的地址。 这允许协议“查找网络的其余部分”。 这与DNS引导本身的方式基本相同（但请注意，更改DNS引导程序列表 - “点域”地址 - 通过设计并不容易）。

  - The list should be stored in long-term local storage, whatever that means to the local node (e.g. to disk).
  - Protocols can ship a default list hardcoded or along with the standard code distribution (like DNS).
  - In most cases (and certainly in the case of IPFS) the bootstrap list should be user configurable, as users may wish to establish separate networks, or place their reliance and trust in specific nodes.

  - 列表应该存储在长期本地存储器中，无论对本地节点（例如磁盘）如何。
  - 协议可以发送硬编码的默认列表，或者随标准代码分发（如DNS）一起发送。
  - 在大多数情况下（当然在IPFS的情况下），引导程序列表应该是用户可配置的，因为用户可能希望建立单独的网络，或将其依赖性和信任度置于特定节点中。

## 4.5 Messaging 消息

#### 4.5.1 PubSub 发布订阅

## 4.6 Naming 命名

#### 4.5.2 IPNS IPNS
