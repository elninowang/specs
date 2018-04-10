# libp2p pubsub specification libp2p的pubsub规范

This is the specification for generalized pubsub over libp2p. Pubsub in libp2p is currently still experimental and this specification is subject to change. This document does not go over specific implementation of pubsub routing algorithms, it merely describes the common wire format that implementations will use.

这是针对libp2p的通用pubsub的规范。 libp2p中的Pubsub目前仍处于试验阶段，并且此规范可能会发生变化。 本文档没有详细介绍pubsub路由算法的具体实现，它仅仅描述了实现将使用的常见连线格式。

libp2p pubsub currently uses reliable ordered streams between peers. It assumes that each peer is certain of the identity of each peer is it communicating with.  It does not assume that messages between peers are encrypted, however encryption defaults to being enabled on libp2p streams.

libp2p pubsub当前在peer之间使用可靠的有序流。 它假定每个peer都确定与它通信的每个peer的身份。 它并不假定对等体之间的消息是加密的，但是加密默认为在libp2p流上启用。

You can find information about the PubSub research and notes in the following repos:

您可以在以下回购协议中找到有关PubSub研究和注释的信息：

- https://github.com/libp2p/research-pubsub
- https://github.com/libp2p/pubsub-notes

## The RPC

All communication between peers happens in the form of exchanging protobuf RPC messages between participating peers.

peers之间的所有通信都以交换参与peer之间的protobuf RPC消息的形式发生。

The RPC protobuf is as follows:

RPC protobuf如下所示：

```protobuf
message RPC {
	repeated SubOpts subscriptions = 1;
	repeated Message publish = 2;

	message SubOpts {
		optional bool subscribe = 1;
		optional string topicid = 2;
	}
}
```

This is a relatively simple message containing zero or more subscription messages, and zero or more content messages. The subscription messages contain a topicid string that specifies the topic, and a boolean signifying whether to subscribe or unsubscribe to the given topic. True signifies 'subscribe' and false signifies 'unsubscribe'. 

这是一个相对简单的消息，包含零个或多个订阅消息以及零个或多个内容消息。 订阅消息包含指定主题的topicid字符串以及表示是否订阅或取消订阅给定主题的布尔值。 True表示“订阅”，false表示“取消订阅”。

## The Message 消息

The RPC message can contain zero or more messages of type 'Message' (perhaps this should be named better?). The Message protobuf looks like this:

RPC消息可以包含零个或多个'Message'类型的消息（也许这应该命名得更好？）。 Message protobuf看起来像这样：

```protobuf
message Message {
	optional string from = 1;
	optional bytes data = 2;
	optional bytes seqno = 3;
	repeated string topicIDs = 4;
}
```

The `from` field denotes the author of the message, note that this is not necessarily the peer who sent the RPC this message is contained in. This is done to allow content to be routed through a swarm of pubsubbing peers. 

`from`字段表示消息的作者，请注意，这不一定是发送该消息所包含的RPC的peer。这样做是为了允许内容通过一组pubsubbing对等体进行路由。

The `data` field is an opaque blob of data, it can contain any data that the publisher wants it to. 

`data`字段是一个不透明的数据块，它可以包含发布者想要的任何数据。

The `seqno` field is a linearly increasing, number unique among messages originating from each given peer. No two messages on a pubsub topic from the same peer should have the same seqno value, however messages from different peers may have the same sequence number, so this number alone cannot be used to address messages. (Notably the 'timecache' in use by the floodsub implementation uses the concatenation of the seqno and the 'from' field) 

`seqno` 字段是线性递增的，在源自每个给定peer的消息之中唯一的数字。 来自同一对等方的pubsub主题上的两条消息应该具有相同的seqno值，但来自不同peer的消息可能具有相同的序列号，因此仅使用此数字不能用于寻址消息。 （值得注意的是floodsub实现使用的'timecache'使用seqno和'from'字段的连接）

The `topicIDs` field specifies a set of topics that this message is being published to. 

`topicIDs` 字段指定了该消息发布到的一组主题。

Note that messages are currently *not* signed. This will come in the near future.

请注意，邮件目前 *not* 签名。 这将在不久的将来。

## The Topic Descriptor 主题描述符

The topic descriptor message is used to define various options and parameters of a topic. It currently specifies the topics human readable name, its authentication options, and its encryption options. 

主题描述符消息用于定义主题的各种选项和参数。 它目前指定了主题人类可读名称，其身份验证选项及其加密选项。

The TopicDescriptor protobuf is as follows:

TopicDescriptor的 protobuf如下所示：

```protobuf
message TopicDescriptor {
	optional string name = 1;
	optional AuthOpts auth = 2;
	optional EncOpts enc = 3;

	message AuthOpts {
		optional AuthMode mode = 1;
		repeated bytes keys = 2;

		enum AuthMode {
			NONE = 0;
			KEY = 1;
			WOT = 2;
		}
	}

	message EncOpts {
		optional EncMode mode = 1;
		repeated bytes keyHashes = 2;

		enum EncMode {
			NONE = 0;
			SHAREDKEY = 1;
			WOT = 2;
		}
	}
}
```

The `name` field is a string used to identify or mark the topic, It can be descriptive or random or anything the creator chooses. 

`name`字段是用于识别或标记主题的字符串，它可以是描述性的或随机的，或创建者选择的任何内容。

The `auth` field specifies how authentication will work for this topic. Only authenticated peers may publish to a given topic. See 'AuthOpts' below for details.

`auth`字段指定认证如何对这个主题起作用。 只有经过身份验证的peers才可以发布到给定主题 有关详细信息，请参阅下面的“AuthOpts”。

The `enc` field specifies how messages published to this topic will be encrypted. See 'EncOpts' below for details.

`enc`字段指定发布到该主题的消息将如何加密。 有关详细信息，请参阅下面的“EncOpts”。

### AuthOpts

The `AuthOpts` message describes an authentication scheme. The `mode` field specifies which scheme to use, and the `keys` field is an array of keys. The meaning of the `keys` field is defined by the selected `AuthMode`.

AuthOpts消息描述了一个认证方案。 `mode`字段指定使用哪个方案，`keys`字段是一组键。 `keys`字段的含义由所选的`AuthMode`定义。

There are currently three options defined for the `AuthMode` enum:

目前为`AuthMode`枚举定义了三个选项：

#### AuthMode 'NONE'
No authentication, anyone may publish to this topic.

没有认证，任何人都可以发布到这个话题。

#### AuthMode 'KEY'
Only peers whose peerIDs are listed in the `keys` array may publish to this topic, messages from any other peer should be dropped.

只有在`keys`数组中列出peerID的peer才可以发布此主题，所以应该删除来自任何其他对等方的消息。

#### AuthMode 'WOT'
Web Of Trust, Any trusted peer may publish to the topic. A trusted peer is one whose peerID is listed in the `keys` array, or any peer who is 'trusted' by another trusted peer. The mechanism of signifying trust in another peer is yet to be defined.

信任网站，任何信任的peer可以发布到该主题。 可信peer是其“peerID”列在“keys”阵列中的任何一个，或者是由另一个可信peer“信任”的任何peer。 在另一个peer中表示信任的机制尚未确定。


### EncOpts

The `EncOpts` message describes an encryption scheme for messages in a given topic. The `mode` field denotes which encryption scheme will be used, and the `keyHashes` field specifies a set of hashes of keys whose purpose may be defined by the selected mode.

EncOpts消息描述了给定主题中消息的加密方案。 `mode`字段表示将使用哪种加密方案，`keyHashes`字段指定一组密钥，其目的可以由所选模式定义。

There are currently three options defined for the `EncMode` enum:

目前为`EncMode`枚举定义了三个选项：

#### EncMode 'NONE'
Messages are not encrypted, anyone can read them.

消息不加密，任何人都可以阅读。

#### EncMode 'SHAREDKEY'
Messages are encrypted with a preshared key. The salted hash of the key used is denoted in the `keyHashes` field of the `EncOpts` message. The mechanism for sharing the keys and salts is undefined.

消息使用预共享密钥进行加密。 所使用的密钥的哈希散列表示在EncOpts消息的`keyHashes`字段中。 共享密钥和盐的机制未定义。

#### EncMode 'WOT'
Web Of Trust publishing. Messages are encrypted with some certificate or certificate chain shared amongst trusted peers. (Spec writers note: this is the least clearly defined option and my description here may be wildly incorrect, needs checking).

Web信任发布。 消息用一些证书或证书链在信任的peer之间共享进行加密。 （规范作者注意：这是最不明确定义的选项，我的描述可能是疯狂不正确的，需要检查）。