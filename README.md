# libp2p 规格书

<h1 align="center">
  <img src="https://raw.githubusercontent.com/libp2p/libp2p/a13997787e57d40d6315b422afbe1ceb62f45511/logo/libp2p-logo.png" alt="libp2p logo"/>
</h1>

[![](https://img.shields.io/badge/made%20by-Protocol%20Labs-blue.svg?style=flat-square)](http://ipn.io)
[![](https://img.shields.io/badge/project-libp2p-blue.svg?style=flat-square)](http://github.com/libp2p/libp2p)
[![](https://img.shields.io/badge/freenode-%23ipfs-blue.svg?style=flat-square)](http://webchat.freenode.net/?channels=%23ipfs)

> 本文介绍`libp2p`，这是一个模块化和可扩展的网络堆栈，用于克服进行peer-to-peer应用时面临的网络挑战。 IPFS使用`libp2p`作为其网络库。

作者:

- [Juan Benet](https://github.com/jbenet)
- [David Dias](https://github.com/diasdavid)

汉化：
- [elninowang](https://github.com/elninowang)

评审:

- `N/A`

## 抽象

这描述了[IPFS](https://ipfs.io/) 网络协议。 网络层提供网络中任何两个IPFS节点之间的点对点传输（可靠和不可靠）。

本文档定义了在`libp2p`中实现的规范。

## 此规格的状态 ![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)

## 本文件的组织

This RFC is organized by chapters described on the *Table of contents* section. Each of the chapters can be found in its own file.

本RFC由 *目录* 部分中描述的章节组织。 每个章节都可以在自己的文件中找到。

## 目录

- [1 介绍](1-introduction.md)
  - [1.1 动机](1-introduction.md#11-motivation)
  - [1.2 目标](1-introduction.md#12-goals)
- [2 分析网络堆栈中的最新技术](2-state-of-the-art.md)
  - [2.1 客户端-服务器 模式](2-state-of-the-art.md#21-the-client-server-model)
  - [2.2 通过解决方案对网络堆栈协议进行分类](2-state-of-the-art.md#22-categorizing-the-network-stack-protocols-by-solutions)
  - [2.3 目前的缺点](2-state-of-the-art.md#23-current-shortcommings)
- [3 要求](3-requirements.md)
  - [3.1 Transport agnostic](3-requirements.md#34-transport-agnostic)
  - [3.2 Multi-multiplexing](3-requirements.md#35-multi-multiplexing)
  - [3.3 加密](3-requirements.md#33-encryption)
  - [3.4 NAT穿越](3-requirements.md#31-nat-traversal)
  - [3.5 中继](3-requirements.md#32-relay)
  - [3.6 启用多网络拓扑](3-requirements.md#36-enable-several-network-topologies)
  - [3.7 资源发现](3-requirements.md#37-resource-discovery)
  - [3.8 消息](3-requirements.md#38-messaging)
  - [3.9 命名](3-requirements.md#38-naming)
- [4 架构](4-architecture.md)
  - [4.1 Peer路由](4-architecture.md#41-peer-routing)
  - [4.2 Swarm](4-architecture.md#42-swarm)
  - [4.3 分布式记录存储](4-architecture.md#43-distributed-record-store)
  - [4.4 发现](4-architecture.md#44-discovery)
  - [4.5 消息](4-architecture.md#45-messaging)
    - [4.5.1 发布与订阅]()
  - [4.6 命名]()
    - [4.6.1 IPRS]()
    - [4.6.1 IPNS]()
- [5 数据结构](5-datastructures.md)
- [6 接口](6-interfaces.md)
  - [6.1 libp2p](6-interfaces.md#61-libp2p)
  - [6.1 传输](6-interfaces.md)
  - [6.2 连接](6-interfaces.md)
  - [6.3 流的多路复用器](6-interfaces.md)
  - [6.3 Swarm](6-interfaces.md#63-swarm)
  - [6.5 Peer发现](6-interfaces.md#65-peer-discovery)
  - [6.2 Peer路由](6-interfaces.md#62-peer-routing)
  - [6.2 内容路由](6-interfaces.md#62-peer-routing)
    - [6.3.1 分布式记录存储](6-interfaces.md#64-distributed-record-store)
  - [6.6 libp2p interface and UX](6-interfaces.md#66-libp2p-interface-and-ux)
- [7 属性](7-properties.md)
  - [7.1 沟通模式 - 流](7-properties.md#71-communication-model---streams)
  - [7.2 端口 - 受限接入点](7-properties.md#72-ports---constrained-entrypoints)
  - [7.3 传输协议](7-properties.md#73-transport-protocols)
  - [7.4 非IP网络](7-properties.md#74-non-ip-networks)
  - [7.5 在电线上](7-properties.md#75-on-the-wire)
    - [7.5.1 协议复用](7-properties.md#751-protocol-multiplexing)
    - [7.5.2 多数据流 - 自我描述协议流](7-properties.md#752-multistream---self-describing-protocol-stream)
    - [7.5.3 多流选择器 - 自描述协议流选择器](7-properties.md#753-multistream-selector---self-describing-protocol-stream-selector)
    - [7.5.4 流复用](7-properties.md#754-stream-multiplexing)
    - [7.5.5 便携式编码](7-properties.md#755-portable-encodings)
    - [7.5.6 安全通信](7-properties.md#756-secure-communications)
- [8 实现](8-implementations.md)
- [9 参考](9-references.md)

## 还没有进入主文档的其他规格

- [终极](/relay)
- [发布订阅](/pubsub)

## 寻求帮助

需要寻找帮助！ [进入issues](https://github.com/libp2p/specs/issues)!

请注意，所有与多格式相关的交互均受IPFS [行为准则](https://github.com/ipfs/community/blob/master/code-of-conduct.md) 的制约。

## 许可

[CC-BY-SA 3.0 License](https://creativecommons.org/licenses/by-sa/3.0/us/) © Protocol Labs Inc.
