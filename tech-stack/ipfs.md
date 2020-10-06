# Peer-to-Peer Communication

Building a decentralized system requires means of communication for peers. The two most essential primitives for implementing a distributed ledger are sending messages and fetching data. While this could be implemented from scratch, several applications/protocols that enable such peer-to-peer communication already exist, for example, BitTorrent and IPFS. This prototype uses IPFS and therefore the following is a short introduction into its mechanics. However, since the prototype is designed in a modular fashion, IPFS could be replaced by other communication modules that provide broadcast message exchange and decentralized data access functionalities.

## IPFS

InterPlanetary File System \(IPFS\) is a protocol that enables sharing of data in a peer-to-peer fashion. Its original use case is the "decentralized web", in which websites are not hosted by central servers but by peers participating in the IPFS network. IPFS consists of several subsystems that provide different functionalities. Pariticpating in the IPFS network typically means runing an IPFS daemon on the local system. This daemon provides RPC-over-HTTP endpoints for interaction with other applications. It is also common for applications to use integration libraries that provide language-specific functions for IPFS functions.

### Basics

IPFS allows peers to fetch data from other peers by its CID \(content identificator\). The latter concept is commonly called content-based addressing. IPFS peers use distributed hash tables \(DHTs\) to find peers that serve data by mapping the CID of the data to peer IDs. A second DHT is used to find IP addresses of peers by their peer ID. Peers advertise data blocks that they want to provide to the network of other IPFS peers. These blocks are distributed to other peers at their discretion to provide redundancy and availability of the data. Therefore, IPFS allows for persistent storage of data and subsequent retrieval over longer periods of time. The actual data exchange is facilitated by [IPFS Bitswap](https://docs.ipfs.io/concepts/bitswap/), an algorithm inspired by Bittorrent.  

![Bitswap Concept](../.gitbook/assets/diagram-of-the-want-have-want-block-process.6ef862a2.png)

The diagram above shows the Bitswap concept. Peer A acquires a block of data by requesting it from other peers that store it. The candidate peers are aquired according to CID to peer ID mappings stored in the DHT. __

More information about IPFS's core functionality and usage can be found in the [official IPFS documentation](https://docs.ipfs.io/concepts/). 

### PubSub

[IPFS PubSub](https://docs.libp2p.io/concepts/publish-subscribe/) enables peer-to-peer message passing within an IPFS network. Messages can be sent to different channels. Peers can subscribe to these channels and then receive all messages sent to that channel. The messages are propagated according to routing algorithms \(such as GossipSub\) to distribute the networking load among participating peers.  

![IPFS PubSub](../.gitbook/assets/message_delivered_to_all.png)

As you can see in the figure above, since all the peers are subscribed to one topic \(cloud\), each peer \(black dot\) receives the message \(small message bubbles\) after one peer has sent it \(large message bubble\).

### Python integration library

The Python library [py-ipfs-http-client](https://github.com/ipfs-shipyard/py-ipfs-http-client) \(formerly ipfsapi\) offers a native API for interaction with the IPFS daemon's HTTP endpoints. The following example shows the ease with which communication with the IPFS client is possible by using an integration library. 

```text
>>> import ipfshttpclient
>>> client = ipfshttpclient.connect()  # Connects to: /dns/localhost/tcp/5001/http
>>> res = client.add('test.txt')
>>> res
{'Hash': 'QmWxS5aNTFEc9XbMX1ASvLET1zrqEaTssqt33rVZQCQb22', 'Name': 'test.txt'}
>>> client.cat(res['Hash'])
'fdsafkljdskafjaksdjf\n'
```

While working with an IPFS integration library is almost as simple as if it was part of the application, communication is still happening over HTTP \(notably it is not highly reliant inter-process communication\). Therefore, problems such as network stack failures and timeouts between the integration library's HTTP client and the IPFS daemon's HTTP server need to be taken into consideration.

### Obstacles

Several peculiarities of working with IPFS and decentralized file systems, in general, need to be considered. They are mentioned in the [IPFS part of the chapter Scaling](/@hpimpss2020-1/s/documentation/~/drafts/-MIORDvcGuJiv7thwshH/scaling/scaling-ipfs). 

