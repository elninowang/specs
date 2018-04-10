# Circuit Relay v0.1.0 回路中继 v0.1.0

> Circuit Switching for libp2p, also known as TURN or Relay in Networking literature.

> 针对libp2p的回路交换，在网络文献中也被称为TURN或Relay。

## Implementations 实现

- [js-libp2p-circuit](https://github.com/libp2p/js-libp2p-circuit)
- [go-libp2p-circuit](https://github.com/libp2p/go-libp2p-circuit)

## Table of Contents 目录

- [Overview 概述](#overview)
- [Dramatization 戏剧](#dramatization)
- [Addressing](#addressing)
- [Wire protocol](#wire-protocol)
- [Interfaces](#interfaces)
- [Implementation Details](#implementation-details)
- [Removing existing relay protocol](#removing-existing-relay-protocol)

## Overview  概述

The circuit relay is a means to establish connectivity between libp2p nodes (e.g. IPFS nodes) that wouldn't otherwise be able to establish a direct connection to each other.

回路中继是一种在libp2p节点（例如IPFS节点）之间建立连接的方法，否则这些节点不能建立彼此的直接连接。

Relay is needed in situations where nodes are behind NAT, reverse proxies, firewalls and/or simply don't support the same transports (e.g. go-ipfs vs. browser-ipfs). Even though libp2p has modules for NAT traversal ([go-libp2p-nat](https://github.com/libp2p/go-libp2p-nat)), piercing through NATs isn't always an option. The circuit relay protocol exists to overcome those scenarios.

在节点位于NAT后面，反向代理，防火墙和/或根本不支持相同传输的情况下（例如，go-ipfs与browser-ipfs），需要中继。 即使libp2p有用于NAT穿越的模块 ([go-libp2p-nat](https://github.com/libp2p/go-libp2p-nat))，穿透NAT并不总是一种选择。 回路中继协议的存在是为了克服这些情况。

Unlike a transparent **tunnel**, where a libp2p peer would just proxy a communication stream to a destination (the destination being unaware of the original source), a circuit relay makes the destination aware of the original source and the circuit followed to establish communication between the two. This provides the destination side with full knowledge of the circuit which, if needed, could be rebuilt in the opposite direction.

与透明的**隧道**不同，一个libp2p节点只需将一个通信流代理到一个目的地（目的地不知道原始源），中继使得目的地知道原始源和遵循的回路以建立 两者之间的沟通。 这为目的地方提供了回路的完整知识，如果需要的话，可以在相反方向进行重建。

Apart from that, this relayed connection behaves just like a regular connection would, but over an existing swarm stream with another peer (instead of e.g. TCP). A node asks a relay node to connect to another node on its behalf. The relay node short-circuits streams between the two nodes, enabling them to reach each other.

除此之外，这个中继连接的行为就像一个常规连接，但通过与另一个peer（而不是例如TCP）的现有群集流。 一个节点要求一个中继节点代表它连接到另一个节点。 中继节点将两个节点之间的流短路，使它们能够相互到达。

Relayed connections are end-to-end encrypted just like regular connections.

中继连接与常规连接一样是端对端加密的。

The circuit relay is both a tunneled transport and a mounted swarm protocol. The transport is the means of ***establishing*** and ***accepting*** connections, and the swarm protocol is the means to ***relaying*** connections.

回路中继既是隧道传输又是安装的群组协议。 运输是***建立***和***接受***连接的手段，并且群协议是***中继***连接的手段。

```
+-----+    /ip4/.../tcp/.../ws/p2p/QmRelay    +-------+    /ip4/.../tcp/.../p2p/QmTwo       +-----+
|QmOne| <------------------------------------>|QmRelay|<----------------------------------->|QmTwo|
+-----+   (/libp2p/relay/circuit multistream) +-------+ (/libp2p/relay/circuit multistream) +-----+
     ^                                         +-----+                                         ^
     |           /p2p-circuit/QmTwo            |     |                                         |
     +-----------------------------------------+     +-----------------------------------------+
```

**Notes for the reader:** **读者须知：**

- We're using the `/p2p` multiaddr protocol instead of `/ipfs` in this document. `/ipfs` is currently the canonical way of addressing a libp2p or IPFS node, but given the growing non-IPFS usage of libp2p, we'll migrate to using `/p2p`.

- 我们在本文档中使用 `/p2p` multiaddr协议而不是 `/ipfs`。 `/ipfs` 目前是寻址libp2p或IPFS节点的标准方式，但鉴于libp2p的非IPFS使用增长，我们将迁移到使用 `/p2p`。

## Dramatization 戏剧

Cast:
- QmOne, the dialing node (browser).
- QmTwo, the listening node (go-ipfs).
- QmRelay, a node which speaks the circuit relay protocol (go-ipfs or js-ipfs).
>
- QmOne, 拨号节点 (browser).
- QmTwo, 监听节点 (go-ipfs).
- QmRelay, 一个说回路中继协议的节点 (go-ipfs or js-ipfs).


Scene 1: 场景1:
- QmOne wants to connect to QmTwo, and through peer routing has acquired a set of addresses of QmTwo.
- QmTwo doesn't support any of the transports used by QmOne.
- Awkward silence.
>
- QmOne想连接到QmTwo，并通过peer路由获得了一组QmTwo的地址。
- QmTwo不支持QmOne使用的任何传输。
- 尴尬的沉默。

Scene 2: 场景2:
- All three nodes have learned to speak the `/ipfs/relay/circuit` protocol.
- QmRelay is configured to allow relaying connections between other nodes.
- QmOne is configured to use QmRelay for relaying.
- QmOne automatically added `/p2p-circuit/p2p/QmTwo` to its set of QmTwo addresses.
- QmOne tries to connect via relaying, because it shares this transport with QmTwo.
- A lively and prolonged dialogue ensues.
>
- 所有三个节点都学会说`/ipfs/relay/circuit`协议。
- QmRelay被配置为允许中继其他节点之间的连接。
- QmOne配置为使用QmRelay进行中继。
- QmOne自动将 `/p2p-circuit/p2p/QmTwo` 添加到其一组QmTwo地址中。
- QmOne尝试通过中继进行连接，因为它与QmTwo共享此传输。
- 随后进行热烈而持久的对话。

## Addressing

`/p2p-circuit` multiaddrs don't carry any meaning of their own. They need to encapsulate a `/p2p` address, or be encapsulated in a `/p2p` address, or both.

`/p2p-circuit` multiaddrs不具有任何自己的意义。 他们需要封装一个`/p2p`地址，或者封装在`/p2p`地址中，或者两者都封装。

As with all other multiaddrs, encapsulation of different protocols determines which metaphorical tubes to connect to each other.

与所有其他多路加密器一样，不同协议的封装决定了哪些隐喻管彼此连接。

A `/p2p-circuit` circuit address, is formated following:

`/p2p-circuit` 回路地址，格式如下：

`[<relay peer multiaddr>]/p2p-circuit/<destination peer multiaddr>`

Examples: 例如：

- `/p2p-circuit/p2p/QmVT6GYwjeeAF5TR485Yc58S3xRF5EFsZ5YAF4VcP3URHt` - Arbitrary relay node - 任意中继节点
- `/ip4/127.0.0.1/tcp/5002/p2p/QmdPU7PfRyKehdrP5A3WqmjyD6bhVpU1mLGKppa2FjGDjZ/p2p-circuit/p2p/QmVT6GYwjeeAF5TR485Yc58S3xRF5EFsZ5YAF4VcP3URHt` - Specific relay node - 特定的中继节点

This opens the room for multiple hop relay, where the second relay is encapsulated in the first relay multiaddr, such as:

这为多跳中继打开了空间，其中第二个中继被封装在第一个中继多地址中，例如：

`<1st relay>/p2p-circuit/<2nd relay>/p2p-circuit/<dst multiaddr>`

A few examples: 一些例子：

Using any relay available: 使用任何可用的中继：

- `/p2p-circuit/p2p/QmTwo`
  - Dial QmTwo, through any available relay node (or find one node that can relay).
  - The relay node will use peer routing to find an address for QmTwo if it doesn't have a direct connection.
- `/p2p-circuit/ip4/../tcp/../p2p/QmTwo`
  - Dial QmTwo, through any available relay node, but force the relay node to use the encapsulated `/ip4` multiaddr for connecting to QmTwo.
>
- `/p2p-circuit/p2p/QmTwo`
  - 拨打QmTwo，通过任何可用的中继节点（或找到一个可以中继的节点）。
  - 如果没有直接连接，中继节点将使用peer路由查找QmTwo的地址。
- `/p2p-circuit/ip4/../tcp/../p2p/QmTwo`
  - 拨打QmTwo，通过任何可用的中继节点，但强制中继节点使用封装的 `/ip4` multiaddr连接到QmTwo。

Specify a relay:

指定一个继电器：

- `/p2p/QmRelay/p2p-circuit/p2p/QmTwo`
  - Dial QmTwo, through QmRelay.
  - Use peer routing to find an address for QmRelay.
  - The relay node will also use peer routing, to find an address for QmTwo.
- `/ip4/../tcp/../p2p/QmRelay/p2p-circuit/p2p/QmTwo`
  - Dial QmTwo, through QmRelay.
  - Includes info for connecting to QmRelay.
  - The relay node will use peer routing to find an address for QmTwo.
>
- `/p2p/QmRelay/p2p-circuit/p2p/QmTwo`
   - 通过QmRelay拨打QmTwo。
   - 使用peer路由查找QmRelay的地址。
   - 中继节点还将使用peer路由来查找QmTwo的地址。
- `/ip4/../tcp/../p2p/QmRelay/p2p-circuit/p2p/QmTwo`
   - 通过QmRelay拨打QmTwo。
   - 包括连接到QmRelay的信息。
   - 中继节点将使用peer路由查找QmTwo的地址。

Double relay: 双中继：

- `/p2p-circuit/p2p/QmTwo/p2p-circuit/p2p/QmThree`
  - Dial QmThree, through a relayed connection to QmTwo.
  - The relay nodes will use peer routing to find an address for QmTwo and QmThree.
  - We'll **not support nested relayed connections for now**, see [Future Work](#future-work) section.
>
- `/p2p-circuit/p2p/QmTwo/p2p-circuit/p2p/QmThree`
  - 通过与QmTwo的中继连接拨打QmThree。
  - 中继节点将使用peer路由查找QmTwo和QmThree的地址。
  - 我们现在 **不支持嵌套中继连接**，请参阅 [Future Work](#future-work) 部分。

## Wire format Wire格式

We start the description of the Wire format by illustrating a possible flow scenario and then describing them in detail by phases.

我们通过说明可能的流程场景，然后分阶段详细描述它们，开始描述Wire格式。

### Relay Message

Every message in the relay protocol uses the following protobuf:

中继协议中的每条消息使用以下protobuf：

```
message CircuitRelay {

  enum Status {
    SUCCESS                    = 100;
    HOP_SRC_ADDR_TOO_LONG      = 220;
    HOP_DST_ADDR_TOO_LONG      = 221;
    HOP_SRC_MULTIADDR_INVALID  = 250;
    HOP_DST_MULTIADDR_INVALID  = 251;
    HOP_NO_CONN_TO_DST         = 260;
    HOP_CANT_DIAL_DST          = 261;
    HOP_CANT_OPEN_DST_STREAM   = 262;
    HOP_CANT_SPEAK_RELAY       = 270;
    HOP_CANT_RELAY_TO_SELF     = 280;
    STOP_SRC_ADDR_TOO_LONG     = 320;
    STOP_DST_ADDR_TOO_LONG     = 321;
    STOP_SRC_MULTIADDR_INVALID = 350;
    STOP_DST_MULTIADDR_INVALID = 351;
    STOP_RELAY_REFUSED         = 390;
    MALFORMED_MESSAGE          = 400;
  }

  enum Type { // RPC identifier, either HOP, STOP or STATUS
    HOP = 1;
    STOP = 2;
    STATUS = 3;
    CAN_HOP = 4; // is peer a relay?
  }

  message Peer {
    required bytes id = 1;    // peer id
    repeated bytes addrs = 2; // peer's known addresses
  }

  optional Type type = 1;     // Type of the message

  optional Peer srcPeer = 2;  // srcPeer and dstPeer are used when Type is HOP or STOP
  optional Peer dstPeer = 3;

  optional Status code = 4;   // Status code, used when Type is STATUS
}
```

### High level overview of establishing a relayed connection

### 建立中继连接的高级概述

**Setup:** **建立:**
- Peers involved, A, B, R
- A wants to connect to B, but needs to relay through R
>
- 参与Peers，A，B，R
- A想连接到B，但需要通过R进行中继

**Assumptions:** **假设:**
- A has connection to R, R has connection to B
>
- A连接到R，R连接到B.

**Events:** **事件:**
- phase I: Open a request for a relayed stream (A to R).
  - A dials a new stream `sAR` to R using protocol `/libp2p/circuit/relay/0.1.0`.
  - A sends a CircuitRelay message with `{ type: 'HOP', srcPeer: '/p2p/QmA', dstPeer: '/p2p/QmB' }` to R through `sAR`.
  - R receives stream `sAR` and reads the message from it.
- phase II: Open a stream to be relayed (R to B).
  - R opens a new stream `sRB` to B using protocol `/libp2p/circuit/relay/0.1.0`.
  - R sends a CircuitRelay message with `{ type: 'STOP', srcPeer: '/p2p/QmA', dstPeer: '/p2p/QmB' }` on `sRB`.
  - R sends a CircuitRelay message with `{ type: 'STATUS', code: 'OK' }` on `sAR`.
- phase III: Streams are piped together, establishing a circuit
  - B receives stream `sRB` and reads the message from it
  - B sends a CircuitRelay message with `{ type: 'STATUS', code: 'OK' }` on `sRB`.
  - B passes stream to `NewConnHandler` to be handled like any other new incoming connection.
>
- 阶段I：打开一个中继流请求（A到R）。
  - 使用协议`/libp2p/circuit/relay/0.1.0` 拨打一个新流`sAR`到R。
  - A通过`sAR`向R发送一个带有 `{ type: 'HOP', srcPeer: '/p2p/QmA', dstPeer: '/p2p/QmB' }` 的回路中继消息。
  - R接收流`sAR`并从中读取消息。
- 阶段II：打开要中继的流（R到B）。
  - R使用协议 `/libp2p/circuit/relay/0.1.0` 打开一个新流`sRB`到B.
  - R在`sRB`上发送带有 `{ type: 'STOP', srcPeer: '/p2p/QmA', dstPeer: '/p2p/QmB' }` 的回路中继消息。
  - R在`sAR`上发送一个回路中继消息，其中包含`{ type: 'STATUS', code: 'OK' }`。
- 第三阶段：流汇集在一起，建立一条回路
  - B接收流`SRB`并从中读取消息
  - B在`sRB`上发送带有`{ type: 'STATUS', code: 'OK' }`的回路中继消息。
  - B将流传递给`NewConnHandler`以像其他任何新的传入连接一样处理。

### Under the microscope 在显微镜下

- We've defined a max length for the multiaddrs of arbitrarily 1024 bytes
- Multiaddrs are transfered on its binary packed format
- Peer Ids are transfered on its non base encoded format (aka byte array containing the multihash of the Public Key).

- 我们已经为任意1024字节的多字节定义了最大长度
- Multiaddrs以二进制打包格式传输
- peer IDs 以其非基本编码格式（又名包含公钥的多哈希的字节数组）传输。


### Status codes table 状态码表

This is a table of status codes and sample messages that may occur during a relay setup. Codes in the 200 range are returned by the relay node. Codes in the 300 range are returned by the destination node.

这是在中继设置过程中可能出现的状态代码和示例消息表。 200范围内的代码由中继节点返回。 300范围内的代码由目标节点返回。


| Code  | Message                                           | Meaning    |
| ----- |:--------------------------------------------------|:----------:|
| 100   | OK                                                | Relay was setup correctly |
| 220   | "src address too long"                            | |
| 221   | "dst address too long"                            | |
| 250   | "failed to parse src addr: no such protocol ipfs" | The `<src>` multiaddr in the header was invalid |
| 251   | "failed to parse dst addr: no such protocol ipfs" | The `<dst>` multiaddr in the header was invalid |
| 260   | "passive relay has no connection to dst"          | |
| 261   | "active relay couldn't dial to dst: conn refused" | relay could not form new connection to target peer |
| 262   | "couldn't' dial to dst"                           | relay has conn to dst, but failed to open a stream |
| 270   | "dst does not support relay"                      | |
| 280   | "can't relay to itself"                           | The relay got its own address as destination |
| 320   | "src address too long"                            | |
| 321   | "dst address too long"                            | |
| 350   | "failed to parse src addr"                        | src multiaddr in the header was invalid |
| 351   | "failed to parse dst addr"                        | dst multiaddr in the header was invalid |
| 390   | "connection refused by stop endpoint"             | The stop endpoint couldn't accept the connection |
| 400   | "malformed message"                               | A malformed or too long message was received |

| Code  | Message                                           | Meaning    |
| ----- |:--------------------------------------------------|:----------:|
| 100   | OK                                                | Relay was setup correctly |
| 220   | "src地址太长"                           | |
| 221   | "dst地址太长"                           | |
| 250   | "无法解析src addr：没有这样的协议ipfs" | 标题中的`<src>`multiaddr无效 |
| 251   | "无法解析dist addr：没有这样的协议ipfs" | 标题中的`<dst>`multiaddr无效 |
| 260   | "被动中继没有连接到dst"          | |
| 261   | "主动中继无法拨号到dst：连接拒绝" | 中继无法与目标peer形成新的连接 |
| 262   | "无法拨号到dst"                           | 中继已连接到dst，但未能打开流 |
| 270   | "dst不知道中继"                      | |
| 280   | "不能中继给自己"                           | 中继有自己的地址作为目的地 |
| 320   | "src地址太长"                            | |
| 321   | "dst地址太长"                            | |
| 350   | "未解析src地址"                        | 标题中的src multiaddr无效 |
| 351   | "未解析dst地址"                        | 标题中的dst multiaddr无效 |
| 390   | "由停止端点拒绝连接"             | 停止端点不能接受连接 |
| 400   | "畸形的消息"                               | 收到了格式错误或过长的消息 |

## Implementation details 实现细节

### Interfaces 接口

> These are go-ipfs specific 这些是go-ipfs特定的

As explained above, the relay is both a transport (`tpt.Transport`) and a mounted stream protocol (`p2pnet.StreamHandler`). In addition it provides a means of specifying relay nodes to listen/dial through.

如上所述，中继既是一个传输(`tpt.Transport`)又是一个安装的流协议(`p2pnet.StreamHandler`)。 另外它提供了一种指定中继节点来监听/拨号的方法。

Note: the usage of p2pnet.StreamHandler is a little bit off, but it gets the point across.

注意：p2pnet.StreamHandler的使用稍微偏离了一点，但它可以得到重点。

```go
import (
  tpt "github.com/libp2p/go-libp2p-transport"
  p2phost "github.com/libp2p/go-libp2p-host"
  p2pnet "github.com/libp2p/go-libp2p-net"
  p2proto "github.com/libp2p/go-libp2p-protocol"
)

const ID p2proto.ID = "/libp2p/circuit/relay/0.1.0"

type CircuitRelay interface {
  tpt.Transport
  p2pnet.StreamHandler

  EnableRelaying(enabled bool)
}

fund NewCircuitRelay(h p2phost.Host)
```

### Removing existing relay protocol in Go 在Go中删除现有的中继协议

Note that there is an existing swarm protocol colloqiually called relay. It lives in the go-libp2p package and is named `/ipfs/relay/line/0.1.0`.

请注意，现有的swarm协议同时称为中继。 它位于go-libp2p包中，名为 `/ipfs/relay/line/0.1.0`。

- Introduced in ipfs/go-ipfs#478 (28-Dec-2014).
- No changes except for ipfs/go-ipfs@de50b2156299829c000b8d2df493b4c46e3f24e9.
  - Changed to use multistream muxer.
- Shortcomings
  - No end-to-end encryption.
  - No rate limiting (DoS by resource exhaustion).
  - Doesn't verify src id in ReadHeader(), easy to fix.
- Capable of *accepting* connections, and *relaying* connections.
- Not capable of *connecting* via relaying.
>
- 在 ipfs/go-ipfs#478 (28-Dec-2014) 中引入。
- 除 ipfs/go-ipfs@de50b2156299829c000b8d2df493b4c46e3f24e9 外没有更改。
   - 更改为使用多码流复用器。
- 缺点
   - 没有端到端的加密。
   - 没有速率限制(资源枯竭导致的DoS)。
   - 不在ReadHeader() 中验证src id，易于修复。
- 能够 *接受* 连接和 *中继* 连接。
- 不能 通过中继 *连接*。

Since the existing protocol is incomplete, insecure, and certainly not used, we can safely remove it.

由于现有的协议是不完整的，不安全的，当然也没有使用，我们可以安全地将其删除。

## Future work 未来的工作

We have considered more features but won't be adding them on the first iteration of Circuit Relay, the features are:

我们已经考虑了更多的功能，但不会在回路中继的第一次迭代中添加它们，其特点是：

- Multihop relay - With this specification, we are only enabling single hop relays to exist. Multihop relay will come at a later stage as Packet Switching.
- Relay discovery mechanism - At the moment we're not including a mechanism for discovering relay nodes. For the time being, they should be configured statically.

- 多跳中继 - 使用此规范，我们只能使单跳中继存在。 多跳中继将在稍后的阶段作为分组交换。
- 中继发现机制 - 目前我们还没有包括发现中继节点的机制。 目前，它们应该静态配置。