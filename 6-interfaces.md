6 Interfaces 接口
============

`libp2p` is a collection of several protocols working together to offer a common solid interface that can talk with any other network addressable process. This is made possible by shimming currently existing protocols and implementations into a set of explicit interfaces: Peer Routing, Discovery, Stream Muxing, Transports, Connections and so on.

`libp2p`是几个协议的集合，共同提供一个通用的固体接口，可以与任何其他网络可寻址过程交谈。 这可以通过将当前现有的协议和实现填充到一组明确的接口中：Peer路由，发现，流复用，传输，连接等。

## 6.1 libp2p

`libp2p`, the top module that provides an interface to all the other modules that make a `libp2p` instance, must offer an interface for dialing to a peer and plugging in all of the modules (e.g. which transports) we want to support. We present the `libp2p` interface and UX in [section 6.6](#66-libp2p-interface-and-ux), after presenting every other module interface.

`libp2p` 是为所有其他模块创建一个`libp2p`实例的接口，它必须提供一个接口，用于拨号到peer并插入我们想要支持的所有模块（例如，传输）。 在介绍每个其他模块接口之后，我们在[section 6.6](#66-libp2p-interface-and-ux)中介绍`libp2p`接口和UX。

## 6.2 Peer Routing Peer路由

![](https://raw.githubusercontent.com/diasdavid/interface-peer-routing/master/img/badge.png)

A Peer Routing module offers a way for a `libp2p` `Node` to find the `PeerInfo` of another `Node`, so that it can dial that node. In its most pure form, a Peer Routing module should have an interface that takes a 'key', and returns a set of `PeerInfo`s.
See https://github.com/diasdavid/interface-peer-routing for the interface and tests.

对等路由模块为`libp2p` `Node`找到另一个`Node`的`PeerInfo`提供了一种方法，以便它可以拨打该节点。 在其最纯粹的形式中，Peer路由模块应该有一个接口，该接口需要一个“密钥”并返回一组“PeerInfo”。
请参阅 https://github.com/diasdavid/interface-peer-routing 了解接口和测试。

## 6.3 Swarm

Current interface available and updated at:

当前可用的界面和更新：

https://github.com/diasdavid/js-libp2p-swarm#usage

### 6.3.1 Transport 传输

![](https://raw.githubusercontent.com/diasdavid/interface-transport/master/img/badge.png)

https://github.com/diasdavid/interface-transport

### 6.3.2 Connection 连接

![](https://raw.githubusercontent.com/diasdavid/interface-connection/master/img/badge.png)

https://github.com/diasdavid/interface-connection

### 6.3.3 Stream Muxing 流混合器

![](https://github.com/diasdavid/interface-stream-muxer/raw/master/img/badge.png)

https://github.com/diasdavid/interface-stream-muxer

## 6.4 Distributed Record Store 分布式记录存储

![](https://raw.githubusercontent.com/diasdavid/interface-record-store/master/img/badge.png)

https://github.com/diasdavid/interface-record-store

## 6.5 Peer Discovery Peer发现

A Peer Discovery module interface should return `PeerInfo` objects, as it finds new peers to be considered by our Peer Routing modules.

Peer Discovery模块接口应该返回`PeerInfo`对象，因为它发现我们的Peer路由模块要考虑的新Peer点。

## 6.6 libp2p interface and UX libp2p接口和UX

`libp2p` implementations should enable it to be instantiated programmatically, or to use a previous compiled library with some of the protocol decisions already made, so that the user can reuse or expand.

`libp2p` 实现应该使它能够以编程方式实例化，或者在已经做出一些协议决定的情况下使用先前编译的库，以便用户可以重用或扩展。

### Constructing a libp2p instance programatically 以编程方式构建libp2p实例

Example made with JavaScript, should be mapped to other languages:

使用JavaScript制作的示例应映射到其他语言：

```JavaScript
var Libp2p = require('libp2p')

var node = new Libp2p()

// add a swarm instance
node.addSwarm(swarmInstance)

// add one or more Peer Routing mechanisms
node.addPeerRouting(peerRoutingInstance)

// add a Distributed Record Store
node.addDistributedRecordStore(distributedRecordStoreInstance)
```

Configuring `libp2p` is quite straightforward since most of the configuration comes from instantiating the several modules, one at a time.

配置`libp2p`非常简单，因为大多数配置都来自实例化多个模块，一次一个。

### Dialing and Listening for connections to/from a peer 拨号和收听与对等方的连接

Ideally, `libp2p` uses its own mechanisms (PeerRouting and Record Store) to find a way to dial to a given peer:

理想情况下，`libp2p`使用它自己的机制（PeerRouting和Record Store）来找到一种方法来拨打给定的peer：

```JavaScript
node.dial(PeerInfo)
```

To receive an incoming connection, specify one or more protocols to handle:

要接收传入连接，请指定一个或多个协议来处理：

```JavaScript
node.handleProtocol('<multicodec>', function (duplexStream) {

})
```

### Finding a peer 查找一个peer

Finding a peer is done through Peer Routing, so the interface is the same.

寻找一个对等点是通过对等路由完成的，所以接口是一样的。

### Storing and Retrieving Records 存储和检索记录

Like Finding a peer, Storing and Retrieving records is done through Record Store, so the interface is the same.

像查找peer一样，存储和检索记录是通过记录存储完成的，因此界面是相同的。
