1 介绍
==============

在开发[IPFS, the InterPlanetary FileSystem](https://ipfs.io/)时，我们开始了解由于必须在异构设备之上运行分布式文件系统而带来的几个挑战，它们具有不同的网络设置和功能。 在这个过程中，我们不得不重新审视整个网络堆栈并精心制定解决方案，以克服由多个层和协议的设计决策带来的障碍，同时不会破坏兼容性或重新创建技术。

为了建立这个库，我们专注于独立解决问题，通过强大的抽象创建不太复杂的解决方案，这些解决方案在编写时可以为peer-to-peer应用程序的成功运行提供一个环境。

## 1.1 动机

`libp2p`是我们建立分布式系统的集体经验的成功，因为它使开发人员有责任决定他们希望应用程序如何与网络中的其他人进行互操作，并支持配置和可扩展性，而不是对网络建立进行假设。

实质上，使用`libp2p`的peer应该能够使用各种不同的传输方式与另一个peer进行通信，包括连接中继，以及通过不同协议进行通信，并根据需要进行协商。

## 1.2 目标

Our goals for the `libp2p` specification and its implementations are:

我们的libp2p规范及其实现的目标是：

  - Enable the use of various:
    - transports: TCP, UDP, SCTP, UDT, uTP, QUIC, SSH, etc.
    - authenticated transports: TLS, DTLS, CurveCP, SSH
  - Make efficient use of sockets (connection reuse)
  - Enable communications between peers to be multiplexed over one socket (avoiding handshake overhead)
  - Enable multiprotocols and respective versions to be used between peers, using a negotiation process
  - Be backwards compatible
  - Work in current systems
  - Use the full capabilities of current network technologies
  - Have NAT traversal
  - Enable connections to be relayed
  - Enable encrypted channels
  - Make efficient use of underlying transports (e.g. native stream muxing, native auth, etc.)

  - 启用各种使用：
    - 传输：TCP，UDP，SCTP，UDT，uTP，QUIC，SSH
    - 认证传输：TLS，DTLS，CurveCP，SSH
  - 有效使用套接字（连接重用）
  - 使peer之间的通信能够通过一个套接字多路复用（避免握手开销）
  - 使用协商过程启用多方协议和peer之间使用的相应版本
  - 向后兼容
  - 在当前系统中工作
  - 使用当前网络技术的全部功能
  - 有NAT穿越
  - 使连接能够被中继
  - 启用加密通道
  - 高效使用底层传输（例如，本地流复用，本地认证等）
