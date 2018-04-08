7 Properties 属性
============

## 7.1 Communication Model - Streams 沟通模式 - 流

The Network layer handles all the problems of connecting to a peer, and exposes simple bidirectional streams. Users can both open a new stream (`NewStream`) and register a stream handler (`SetStreamHandler`). The user is then free to implement whatever wire messaging protocol she desires. This makes it easy to build peer-to-peer protocols, as the complexities of connectivity, multi-transport support, flow control, and so on, are handled.

网络层处理连接到peer的所有问题，并公开简单的双向流。 用户可以打开一个新的流（`NewStream`）并注册流处理程序（`SetStreamHandler`）。 然后用户可以自由地实现她希望的任何有线消息协议。 这使得构建peer-to-peer协议变得很容易，因为处理了连接性，多传输支持，流量控制等复杂性。

To help capture the model, consider that:

为了帮助捕捉模型，请考虑：

- `NewStream` is similar to making a Request in an HTTP client.
- `SetStreamHandler` is similar to registering a URL handler in an HTTP server

- `NewStream` 类似于在HTTP客户端中创建请求。
- `SetStreamHandler` 类似于在HTTP服务器中注册URL处理程序

So a protocol, such as a DHT, could: 

所以一个协议，比如DHT，可以：

```go
node := p2p.NewNode(peerid)

// register a handler, here it is simply echoing everything.
node.SetStreamHandler("/helloworld", func (s Stream) {
  io.Copy(s, s)
})

// make a request.
buf1 := []byte("Hello World!")
buf2 := make([]byte, len(buf1))

stream, _ := node.NewStream("/helloworld", peerid) // open a new stream
stream.Write(buf1)  // write to the remote
stream.Read(buf2)   // read what was sent back
fmt.Println(buf2)   // print what was sent back
```

## 7.2 Ports - Constrained Entrypoints  端口 - 受限入口点

In the Internet of 2015, we have a processing model where a program may be running without the ability to open multiple -- or even single -- network ports. Most hosts are behind NAT, whether of the household ISP variety or the new containerized data-center type. And some programs may even be running in browsers, with no ability to open sockets directly (sort of). This presents challenges to completely peer-to-peer networks that aspire to connect _any_ hosts together -- whether they're running on a page in the browser, or in a container within a container.

在2015年的互联网中，我们有一种处理模式，程序可能无法打开多个甚至单个网络端口。 大多数主机位于NAT之后，无论是家庭ISP类型还是新的容器内数据中心类型。 有些程序甚至可能在浏览器中运行，无法直接打开socket。 这对于想要将_any_主机连接在一起的完全P2P网络（无论它们是在浏览器中的页面上运行还是在容器内的容器中）提出了挑战。

IPFS only needs a single channel of communication with the rest of the network. This may be a single TCP or UDP port, or a single connection through WebSockets or WebRTC. In a sense, the role of the TCP/UDP network stack -- i.e. multiplexing applications and connections -- may now be forced to happen at the application level.

IPFS只需要与网络其余部分进行单一通信。 这可能是单个TCP或UDP端口，也可能是通过WebSocket或WebRTC的单个连接。 从某种意义上讲，TCP/UDP网络堆栈的作用 - 即多路复用应用程序和连接 - 现在可能被迫在应用程序级发生。

## 7.3 Transport Protocols 运输协议

IPFS is transport-agnostic. It can run on any transport protocol. The `ipfs-addr` format (which is an IPFS-specific [multiaddr](https://github.com/jbenet/multiaddr)) describes the transport. For example:

IPFS与传输无关。 它可以在任何传输协议上运行。 `ipfs-addr`格式（这是一个IPFS特定的[multiaddr](https://github.com/jbenet/multiaddr)）描述了传输。 例如：

```sh
# ipv4 + tcp
/ip4/10.1.10.10/tcp/29087/ipfs/QmVcSqVEsvm5RR9mBLjwpb2XjFVn5bPdPL69mL8PH45pPC

# ipv6 + tcp
/ip6/2601:9:4f82:5fff:aefd:ecff:fe0b:7cfe/tcp/1031/ipfs/QmRzjtZsTqL1bMdoJDwsC6ZnDX1PW1vTiav1xewHYAPJNT

# ipv4 + udp + udt
/ip4/104.131.131.82/udp/4001/udt/ipfs/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ

# ipv4 + udp + utp
/ip4/104.131.67.168/udp/1038/utp/ipfs/QmU184wLPg7afQjBjwUUFkeJ98Fp81GhHGurWvMqwvWEQN
```

IPFS delegates the transport dialing to a multiaddr-based network package, such as [go-multiaddr-net](https://github.com/jbenet/go-multiaddr-net). It is advisable to build modules like this in other languages, and scope the implementation of other transport protocols.

IPFS将传输拨号委托给基于multiaddr的网络包，例如 [go-multiaddr-net](https://github.com/jbenet/go-multiaddr-net)。 建议在其他语言中构建这样的模块，并确定其他传输协议的实现范围。

Some of the transport protocols we will be using:

我们将使用的一些传输协议：

- UTP
- UDT
- SCTP
- WebRTC (SCTP, etc)
- WebSockets
- TCP Remy

## 7.4 Non-IP Networks 非IP网络
 
Efforts like [NDN](http://named-data.net) and [XIA](http://www.cs.cmu.edu/~xia/) are new architectures for the Internet, which are closer to the model IPFS uses than what IP provides today. IPFS will be able to operate on top of these architectures trivially, as there are no assumptions made about the network stack in the protocol. Implementations will likely need to change, but changing implementations is vastly easier than changing protocols.

像 [NDN](http://named-data.net) 和[XIA](http://www.cs.cmu.edu/~xia/) 这样的努力是互联网的新架构，它们更接近模型IPFS的使用比现在的IP提供的要多。 IPFS将能够在这些体系结构的基础上进行操作，因为没有对协议中的网络堆栈做出任何假设。 实现可能需要改变，但改变实现比改变协议要容易得多。

## 7.5 On the wire 在电线上

We have the **hard constraint** of making IPFS work across _any_ duplex stream (an outgoing and an incoming stream pair, any arbitrary connection) and work on _any_ platform.

我们有使IPFS跨越_any_ duplex流（输出和输入流对，任意连接）工作并且在_any_平台上工作的 **硬约束**。

To make this work, IPFS has to solve a few problems:

为了做到这一点，IPFS必须解决一些问题：

- [Protocol Multiplexing](#751-protocol-multiplexing) - running multiple protocols over the same stream
  - [multistream](#752-multistream-self-describing-protocol-stream) - self-describing protocol streams
  - [multistream-select](#753-multistream-selector-self-describing-protocol-stream-selector) - a self-describing protocol selector
  - [Stream Multiplexing](#754-stream-multiplexing) - running many independent streams over the same wire
- [Portable Encodings](#755-portable-encodings) - using portable serialization formats
- [Secure Communications](#756-secure-communication) - using ciphersuites to establish security and privacy (like TLS)

- [协议复用](#751-protocol-multiplexing) - 在同一个流上运行多个协议
  - [多流](#752-multistream-self-describing-protocol-stream) - 自我描述协议流
  - [多流选择](#753-multistream-selector-self-describing-protocol-stream-selector) - 自描述协议选择器
  - [流复用](#754-stream-multiplexxing) - 通过同一条线路运行许多独立的流
- [便携式编码](#755-portable-encodings) - 使用便携式序列化格式
- [安全通信](#756-secure-communication) - 使用密码组建立安全和隐私（如TLS）

### 7.5.1 Protocol-Multiplexing 协议复用

Protocol Multiplexing means running multiple different protocols over the same stream. This could happen sequentially (one after the other), or concurrently (at the same time, with their messages interleaved). We achieve protocol multiplexing using three pieces:

- [multistream](#752-multistream-self-describing-protocol-stream) - self-describing protocol streams
- [multistream-select](#753-multistream-selector-self-describing-protocol-stream-selector) - a self-describing protocol selector
- [Stream Multiplexing](#754-stream-multiplexing) - running many independent streams over the same wire

### 7.5.2 multistream - self-describing protocol stream 多数据流 - 自我描述协议流

[multistream](https://github.com/jbenet/multistream) is a self-describing protocol stream format. It is extremely simple. Its goal is to define a way to add headers to protocols that describe the protocol itself. It is sort of like adding versions to a protocol, but extremely explicit.

[多流](https://github.com/jbenet/multistream) 是一种自我描述的协议流格式。 这非常简单。 其目标是定义一种将头部添加到描述协议本身的协议的方法。 这有点像为协议添加版本，但非常明确。

For example: 例如

```
/ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
<dht-message>
<dht-message>
...
```

### 7.5.3 multistream-selector - self-describing protocol stream selector 多流选择器 - 自描述协议流选择器

[multistream-select](https://github.com/jbenet/multistream/tree/master/multistream) is a simple [multistream](https://github.com/jbenet/multistream) protocol that allows listing and selecting other protocols. This means that Protomux has a list of registered protocols, listens for one, and then _nests_ (or upgrades) the connection to speak the registered protocol. This takes direct advantage of multistream: it enables interleaving multiple protocols, as well as inspecting what protocols might be spoken by the remote endpoint.

[multistream选择](https://github.com/jbenet/multistream/tree/master/multistream) 是一个简单的[multistream](https://github.com/jbenet/multistream) 协议，允许列出和选择其他协议。 这意味着Protomux有一个注册协议列表，监听一个，然后_nests_（或升级）连接以说出注册的协议。 这直接利用了多流：它可以交叉多个协议，并检查远程端点可能会说什么协议。

For example: 例如

```
/ipfs/QmdRKVhvzyATs3L6dosSb6w8hKuqfZK2SyPVqcYJ5VLYa2/multistream-select/0.3.0
/ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
<dht-message>
<dht-message>
...
```

### 7.5.4 Stream Multiplexing 流复用

Stream Multiplexing is the process of multiplexing (or combining) many different streams into a single one. This is a complicated subject because it enables protocols to run concurrently over the same wire, and all sorts of notions regarding fairness, flow control, head-of-line blocking, etc. start affecting the protocols. In practice, stream multiplexing is well understood and there are many stream multiplexing protocols. To name a few:

流复用是将多个不同流复用（或合并）为一个流的过程。 这是一个复杂的主题，因为它使协议能够在同一条线路上同时运行，并且关于公平性，流量控制，线头阻塞等的各种概念开始影响协议。 实际上，流复用是很好理解的，并且有许多流复用协议。 仅举几例：

- HTTP/2
- SPDY
- QUIC
- SSH

IPFS nodes are free to support whatever stream multiplexors they wish, on top of the default one. The default one is there to enable even the simplest of nodes to speak multiple protocols at once. The default multiplexor will be HTTP/2 (or maybe QUIC?), but implementations for it are sparse, so we are beginning with SPDY. We simply select which protocol to use with a multistream header.

IPFS节点可以自由支持他们希望的任何流多路复用器，在默认的节点之上。 默认的一个即使是最简单的节点也可以一次说多个协议。 默认的多路复用器是HTTP/2（或者可能是QUIC？），但是它的实现是稀疏的，所以我们从SPDY开始。 我们只需选择使用多流头的协议。

For example: 例如:

```
/ipfs/QmdRKVhvzyATs3L6dosSb6w8hKuqfZK2SyPVqcYJ5VLYa2/multistream-select/0.3.0
/ipfs/Qmb4d8ZLuqnnVptqTxwqt3aFqgPYruAbfeksvRV1Ds8Gri/spdy/3
<spdy-header-opening-a-stream-0>
/ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-dht/0.2.3
<dht-message>
<dht-message>
<spdy-header-opening-a-stream-1>
/ipfs/QmVXZiejj3sXEmxuQxF2RjmFbEiE9w7T82xDn3uYNuhbFb/ipfs-bitswap/0.3.0
<bitswap-message>
<bitswap-message>
<spdy-header-selecting-stream-0>
<dht-message>
<dht-message>
<dht-message>
<dht-message>
<spdy-header-selecting-stream-1>
<bitswap-message>
<bitswap-message>
<bitswap-message>
<bitswap-message>
...
```

### 7.5.5 Portable Encodings 便携式编码

In order to be ubiquitous, we _must_ use hyper-portable format encodings, those that are easy to use in various other platforms. Ideally these encodings are well-tested in the wild, and widely used. There may be cases where multiple encodings have to be supported (and hence we may need a [multicodec](https://github.com/jbenet/multicodec) self-describing encoding), but this has so far not been needed.
For now, we use [protobuf](https://github.com/google/protobuf) for all protocol messages exclusively, but other good candidates are [capnp](https://capnproto.org/), [bson](http://bsonspec.org/), and [ubjson](http://ubjson.org/).

为了无处不在，我们必须使用超便携式格式编码，这些编码易于在各种其他平台中使用。 理想情况下，这些编码在野外得到了充分测试，并被广泛使用。 可能有些情况下必须支持多种编码（因此我们可能需要 [multicodec](https://github.com/jbenet/multicodec) 自描述编码），但迄今为止并不需要这种编码。
目前，我们仅对所有协议消息使用 [protobuf](https://github.com/google/protobuf)，但其他好的应用程序是[capnp](https://capnproto.org/)，[bson](http://bsonspec.org/) 和 [ubjson](http://ubjson.org/)。

### 7.5.6 Secure Communications 安全通信

The wire protocol is -- of course -- wrapped with encryption. We use cyphersuites similar to TLS. This is explained further in the [network spec](./#encryption).

有线协议当然包含加密。 我们使用类似于TLS的cyphersuites。 这在 [network spec](./#encryption) 中有进一步解释。

### 7.5.7 Protocol Multicodecs 协议多节点

Here, we present a table with the multicodecs defined for each IPFS protocol that has a wire componenent. This list may change over time and currently exists as a guide for implementation.

在这里，我们为每个具有线组件的IPFS协议定义一个多节点表。 这份清单可能会随着时间而改变，目前作为实施指南存在。

protocol | multicodec | comment
:---- | :---- | :----
secio | /secio/1.0.0 |
TLS | /tls/1.3.0 | not implemented
plaintext | /plaintext/1.0.0 |
spdy | /spdy/3.1.0 |
yamux | /yamux/1.0.0 |
multiplex | /mplex/6.7.0 |
identify | /ipfs/id/1.0.0 |
ping | /ipfs/ping/1.0.0 |
circuit-relay | /libp2p/relay/circuit/0.1.0 | [spec](/relay)
diagnostics | /ipfs/diag/net/1.0.0 |
Kademlia DHT | /ipfs/kad/1.0.0 |
bitswap | /ipfs/bitswap/1.0.0 |
