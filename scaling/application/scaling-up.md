# Scaling-Up

**Scaling-Up** or **vertical scaling** describes the process of adding resources to a single computer or changing to a more capable computer in terms of memory or CPU power. This allows for fast testing and benchmarking of the implementation of a distributed system's components with regards to how it behaves when a high number of nodes participate in the distributed system by running many virtual nodes on one physical host. Notably, it does not require the changes needed for supporting multiple hosts. If resource requirements exceed the capacity of the host changing its hardware configuration or changing the host to a more capable model is required.

Two machines were used for vertical scaling of the decentralized learning system:

<table>
  <thead>
    <tr>
      <th style="text-align:left">
        <p></p>
        <p>Machine/Specification</p>
      </th>
      <th style="text-align:left">RAM (GB)</th>
      <th style="text-align:left">CPU</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">VM &quot;Tangle&quot;</td>
      <td style="text-align:left">32</td>
      <td style="text-align:left">16 Virtual CPUs</td>
    </tr>
    <tr>
      <td style="text-align:left">Future SOC&apos;s dl560-1.fsoc</td>
      <td style="text-align:left">1500</td>
      <td style="text-align:left">
        <p>160 CPUs (x86_64)</p>
        <p>Intel(R) Xeon(R) Gold 6148 CPU @ 2.40GHz</p>
      </td>
    </tr>
  </tbody>
</table>

[The Future SOC Lab](https://hpi.de/forschung/future-soc-lab.html) is a "place for scientific exchange in the fields of in-memory computing, cloud computing, and non-CPU \(GPU, FPGA\) computing". It provides IT resources \(such as the _dl560_ in the table above\) for research projects free of charge. This machine was used for the largest benchmarks performed in the course of this project. 

### Strategy

For successful vertical scaling, it is important to identify the limiting resource by setting up proper [monitoring](https://app.gitbook.com/@hpimpss2020/s/documentation/~/drafts/-MHvTzTh9o1S8F474HLH/scaling/distributed-monitoring-and-benchmark-system). For the decentralized machine learning prototype, we identified memory consumption as the main bottleneck. After switching to a bigger machine with 1.5 TB of memory we identified the next issue: the network stack. For proper performance, the IPFS Stack had to be adjusted, as well as some kernel options. One solution turned out to be to set the parallel-connection limits on IPFS much lower than the default values intended for a one-daemon-per-host setup. The actual changes made to the IPFS configuration for enabling single-machine scale-up are listed in the chapter [IPFS Networking](../scaling-ipfs.md). Furtermore, the ARP cache's size was increased since it was constantly overflowing and restarting the whole network stack due to the high number of network connections required for the simulation. 

### Tools

**Docker-Compose** allows for easy vertical scaling by specifying the `--scale service=<number-of-replicas>` for a service it manages. To start a network of decentralized machine learning peers it can simply be started with the preferred number of peers.

```bash
docker-compose build && docker-compose up --scale peer=<number-of-replicas>
```

Docker-Compose starts to get problematic at around 30 peers because of the communication between Docker-Compose and the Docker daemon which is done through HTTP-connections. It is required to increase the timeout for these connections. Otherwise, the terminal will think that the containers did not start on time.

```text
COMPOSER_HTTP_TIMEOUT=120; docker-compose build && docker-compose up --scale peer=50
```

Due to decreasing responsiveness of the Docker daemon \(and therefore also Docker-Compose\) when increasing the number of peers we were unable to scale the system further than 50 peers. To overcome these problems we decided to use orchestration systems for scaling-up. Due to their nature, this part of the journey is described in the next chapter but equally relevant for benchmarks that we exectured later on on a single host. 

### **Things to pay attention too**

Docker has some design decisions, which can be challenging for scaling. It favours speed over storage usage and does not do any automatic storage cleaning, and it had to be triggered manually after every run with _docker prune -all_. Docker also has some folders which do not get cleaned up with _docker prune -all_, such as the _overlay2_ folder and we had to delete its content after every run.

