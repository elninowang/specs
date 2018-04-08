5 Data structures 数据结构
=================

The network protocol deals with these data structures:

网络协议处理这些数据结构：

- a `PrivateKey`, the private key of a node.
- a `PublicKey`, the public key of a node.
- a `PeerId`, a hash of a node's public key.
- a `PeerInfo`, an object containing a node's `PeerId` and its known multiaddrs.
- a `Transport`, a transport used to establish connections to other peers. See <https://github.com/diasdavid/interface-transport>.
- a `Connection`, a point-to-point link between two nodes. Must implement <https://github.com/diasdavid/interface-connection>.
- a `Muxed-Stream`, a duplex message channel.
- a `Stream-Muxer`, a stream multiplexer. Must implement <https://github.com/diasdavid/interface-stream-muxer>.
- a `Record`, IPLD (IPFS Linked Data) described object that implements IPRS.
- a `multiaddr`, a self describable network address. See <https://github.com/jbenet/multiaddr>.
- a `multicodec`, a self describable encoding type. See <https://github.com/jbenet/multicodec>.
- a `multihash`, a self describable hash. See <https://github.com/jbenet/multihash>.

- 一个`PrivateKey`，一个节点的私钥。
- 一个`PublicKey`，一个节点的公钥。
- 一个`PeerId`，一个节点的公钥的散列。
- 一个`PeerInfo`，一个包含一个节点的`PeerId`及其已知的multiaddrs的对象。
- 一个`Transport`，一种用于与其他peers建立连接的传输工具。 请参阅<https://github.com/diasdavid/interface-transport>。
- 一个`Connection`，即两个节点之间的点对点链接。 必须实现<https://github.com/diasdavid/interface-connection>。
- 一个`Muxed-Stream`，一个双工信息通道。
- 一个`Stream-Muxer`，一个流复用器。 必须实现<https://github.com/diasdavid/interface-stream-muxer>。
- 一个`Record`，即IPLD（IPFS关联数据）描述的实现IPRS的对象。
- 一个`multiaddr`，一个可自我描述的网络地址。 请参阅<https://github.com/jbenet/multiaddr>。
- 一个`multicodec`，一个可自我描述的编码类型。 参见<https://github.com/jbenet/multicodec>。
- 一个`multihash`，一个可自我描述的哈希。 请参阅<https://github.com/jbenet/multihash>。
