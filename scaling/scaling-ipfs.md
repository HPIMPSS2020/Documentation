# IPFS

The IPFS executable is intended for running one daemon per host. In order to make it fit for a scaling scenario we took several steps. The most important ones are listed in this chapter. 

### Dockerizing the IPFS Daemon

Official Docker images by IPFS exist under the name [`ipfs/go-ipfs`](https://hub.docker.com/r/ipfs/go-ipfs/). Since they do not provide support for architectures other than `linux/amd64` we built our own Docker images based on `python:3.7` in order to be able to build for `linux/arm32` as well as `linux/arm64`. This allowed us to target different architectures with one `Dockerfile` and enabled us to scale-out to ARM-based edge devices such as ODROIDs. Since it is a lot simpler, we chose a different approeach for the _bootstrap_ container: Depending on the target platform, different Dockerfiles with either [`ifps/go-ipfs`](https://hub.docker.com/r/ipfs/go-ipfs/) or the unofficial [`yrzr/go-ipfs-arm64v8`](https://hub.docker.com/r/yrzr/go-ipfs-arm32v7) or [`yrzr/go-ipfs-arm64v8`](https://hub.docker.com/r/yrzr/go-ipfs-arm64v8) respectively need to be used. This enabled us to explore both approaches and can be changed trivially if a homogenous ARM-based cluster is used. 

### Scalable Configuration

During the course of the prototype development and benchmarking, we experimented with almost all of the configuration options IPFS provides. Most of the settings did not provide significant reproducible impact on benchmarking results. However, some of the configuration values did have an impact on the results of benchmarking. Some like deactivating specific network protocols unnecessary in a simulation environment or enabling more sophisticated security protocols like Noise instead of TLS are set statically since there is no need to adapt those for different benchmarking situation. Those that can be dynamically configured at runtime for different benchmarking runs are listed in the following. 

#### Connection Pruning

For obvious reasons IPFS peers need to maintain connections to other peers.  By default, these values are configured for a single daemon per host system: 

```text
{
  "Swarm": {
    "ConnMgr": {
      "Type": "basic",
      "LowWater": 600,
      "HighWater": 900,
      "GracePeriod": "20s"
    }
  }
}
```

For a simulation scenario where many nodes run on one system, these values overwhelm the operating system kernel with network connections it needs to maintain. When simulating 100 peers on one host, this host would have to have to handle 90,000 connections in a worst-case scenario even without considering more short-lived connections that are not considered by pruning. This has led to parts of the network stack failing in some of our benchmarking scenarios by overflowing the ARP cache.  By default, we reduced these values to `LowWater=40`, `HighWater=70` and `GracePeriod=60s`. This means that after exceeding 70 connections which are open for more than 60 seconds, each peer will trim down to 40 connections. We have experimented with lots of values and found these to be at a good trade-off between scalability and network stability. 

#### Garbage Collection

By default garbage collection is disabled. This means that the stoarge requirement monotonically increases with uptime. In a simulation scenario with a high number of transactions per hour this inevitably consumes all of the storage available. 

We activated garbage collection by passing `--enable-gc` to the IPFS daemon upon start. The config value defaults `Datastore.StorageMax` and `Datastore.GCPeriod` are configurable at runtime and default to 10GB and 1h. `Datastore.StorageGCWatermark` is left statically at the sensible default of 90%.

Furthermore, we ran an additional garbage collection job that manually triggered IPFS garbage collection in order to make sure it was definitively run by executing `ipfs repo gc` after randomized time periods inside of each container. 

#### Reproviding

Reproviding means re-announcing information to the network about data blocks a peer can provide. These blocks can have either been created by the node itself or cached because it previously acquired these blocks from other peers. It is dynamically configurable and the out-of-the-box default was changed from `Reprovider.Interval=12h` to 1 hour in order to enable re-started peers to acquire information more quickly. The `Reprovider.Strategy=all` is left statically at the default since it provides the greatest redundancy possible by letting all peers advertise all data blocks they have stored. 

### Further Challenges

IPFS is intended as a replacement for centralized content servers on the web. Therefore, native scalibility is part of the project's most important goals. However, we have experienced lots of obstacles on the way to enabling the prototype to scale up to very large simulation scenarios. 

The first reason for obstacles was different assumptions as to how IPFS is used. In a decentralized web scenario, communication is infrequent, sporadic and local. Infrequent because web clients are typically not fetching or sending data most of the day. Sporadic because clients typically only need to fetch specific or the most recent data, not everything that was published since they have last fetched data. Local because data is ususally supplied by few nodes that might be in close proximity to the client. In contrast, in a DEUSS or even generic learning tangle scenario data is fetched frequently \(typically before every learning round\), continuously \(all data published needs to be fetched by each peer\) and global \(data is ususally fetched from all peers\). We assume that these difference have an impact on the type of scalability IPFS is designed for. 

The second reason for obstacles was immaturity of the IPFS project. IPFS is still in heavy development and to our knowledge has never been applied productively neither on a large nor a small scale. Therefore, bugs, frequent changes and imperfect implementations that do not cover all edge cases and usage scenarios are to be expected. 

Some of the obstacles are listed in remainder of this chapter.  

* Peers tend to have a different view of the network. Since IPFS is best-effort communication, it is possible for some peers to be able to reach files or receive messages that other peers can't access and haven't seen, respectively. 
* Receiving files is a relatively complex process that involves multiple rounds of communication. Therefore, it can fail arbitrarily and has non-deterministic timing behaviour. Furthermore, timeouts need to be set in order for the daemon to know when to stop looking for data that might not be available \(any more\). These timeouts are static and it is hard to estimate an upper bound of fetching duration for _available_ data. 
* Sending files can take a long time if an IPFS daemon is busy with fetching data or takes a long time to inform other peers of data's availability due to networking bottlenecks. Furthermore, files may not be accessible to any node yet once the daemon returns for a successful sending operation. 
* Availability of data by a certain peer requires that at least one other peer reachable by the former stores and advertises said data. If peers go down data might become unavailable globally if it is not stored redundantly. While we have not implemented it for this prototype, [IPFS Cluster](https://cluster.ipfs.io/) might be a way to organize redundancy in a decentralized way across a learning tangle network.
* The IPFS daemon is designed for running on one dedicated machine. Therefore, it is not very resource efficient \(particularly when it comes to memory, storage and network communication consumption\). This makes simulations hard and might prohibit some use cases that would rely on small edge devices. 
* There is a myriad of configuration possibilities. Furthermore, defaults are not static and new configuration values are introduced in new versions \(such as in IPFS 5\). Configuration possibilities include those listed in the beginning of this chapter.
* When using an integration library its target version needs to match the daemon's API version if the API contains breaking changes or behaves differently. 

