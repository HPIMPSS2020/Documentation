---
description: Femnist Model
---

# Setup

The prototype was implemented to evaluate the feasibility of a learning tangle implementation in a real-world scenario and to compare its performance with established federated learning algorithms and with the conceptual learning tangle simulation.   


## Model

The model used is the same as the model from the paper _Tangle Ledger for Decentralized Learning_, which uses an implementation by the LEAF framework. The LEAF framework provides reference models for numerous federated learning data sets. This evaluation only uses the FEMNIST data set, which is a federated version of the [MNIST](http://yann.lecun.com/exdb/mnist/) data set. 

The evaluation is benchmarked against three baselines:

* The original baseline from the paper _Communication-Efficient Learning of Deep Networks from Decentralized Data_ , which evaluates to around 99% accuracy
* The baseline from the _LEAF Framework_, which evaluates to roughly 80% accuracy
* The paper _Tangle Ledger for Decentralized Learning_ introduced earlier evaluates a simulation \(henceforth called _conceptual simulation_ for simplicity\) with the same approach as our ours and achieves an accuracy of roughly 70%

## Focus

While the focus is the evaluation of the performance of the implementation, other aspects are also observed and rated. For us, this is the feasibility of the implementation on edge devices, and therefore we also evaluate factors such as stability, network communication, and resource usage on the device, based on the requirements of the previously introduced D-EUSS use case:

| Possible D-EUSS Rrequirements |
| :--- |
| 200,000 peers |
| Peers are online ≥ 99% of time \(very stable network, very few re-entries\) |
| ≤ 1 update per week per peer |
| no central server with all training data |

## Limitations of the Implementation

The prototype and evaluation have some limitations. The same tip-selection used in the conceptual simulation is used, which always starts at the first transaction instead of starting at the beginning of a sliding window or from some previously selected transaction. 

We also do not introduce any hostile peers into the simulation. The conceptual simulation considered malicious peers and concluded that even with 20% of all peers being malicious, the tangle would still be stable.

## How are Benchmarks Done?

For all benchmarks K3s was used to orchestrate the peers and other containers. An more in-depth introduction and set-up can be found in the chapter [Containerization & Orchestration](../tech-stack/containerization-orchestration.md). The runbook for benchmarks looked similar to the following one. 

1. Adjust the settings in `k8s/peer-deployment.yaml`
2. Start all containers with `kubectl apply -f k8s/`
3. Scale to intended number `n` of peers`kubectl scale --replicas=n -f k8s/peer-deployment.yaml`
4. Check if everything started with `watch kubectl get pods`
5. Let the benchmark run for the required time
6. Extract required metrics from Grafana

{% hint style="info" %}
For each host a dedicated`peer-deployment.yaml`file depending on its capacity was used in scale-out benchmarks running on small devices. This maximizes the maximum number of peers in resource limited scenarios by improving the distribution of peers over the available hosts. 
{% endhint %}



|  |
| :--- |


