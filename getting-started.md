---
description: >-
  This guide will help you to set up and run the project on a Kubernetes
  cluster. Each section is dependent on the other one, so try to go through each
  step in the order provided.
---

# Getting Started

## Prerequisites

* Docker installed. You can find a guide [here](https://docs.docker.com/engine/install/ubuntu/). 
* Docker-Compose installed. You can find the installation guide [here](https://docs.docker.com/compose/install/).  For Installing on ARM devices, use [this](https://withblue.ink/2019/07/13/yes-you-can-run-docker-on-raspbian.html) guide.
* Kubectl installed. Official installation [documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/).
* Kubernetes installed and cluster configured. Official installation guide under [this link](https://rancher.com/docs/k3s/latest/en/installation/). 

## Seting Up the Project

In this section, we go through the steps of setting up the project:

1. Clone the project from the [GitLab repository](https://gitlab.hpi.de/osm/mpss2020/ipfs-learning-tangle).
2. Download the training data and genesis transaction from [here](https://owncloud.hpi.de/s/QApb0yznb6Its8a).
3. Put the `data` folder \(contains training data\) into the `peer` folder.
4. Put the `genesis.npy` \(example genesis transaction\) file form the `tangle` folder into the `data` folder.

## Building and Pushing Images

To benefit from the newest changes made in the code, the images need to be built and pushed to the container registry. The images will be pushed to a private repository on [DockerHub](https://hub.docker.com/). Let's go step by step through the process of building and publishing the images to the private registry:

1. Login to DockerHub with the `docker login` command. The username of the private registry is `hpimpss2020` and the password can be obtained from the creators of this project.
2. Depending on the target architecture\(s\) for the network run the following command to build and push your images:

{% tabs %}
{% tab title="x86" %}
```bash
docker-compose build && docker-compose push
```
{% endtab %}

{% tab title="arm32" %}
```bash
docker-compose -f docker-compose.yml -f docker-compose.arm.yml build && docker-compose -f docker-compose.yml -f docker-compose.arm.yml push
```
{% endtab %}

{% tab title="arm64" %}
```bash
docker-compose -f docker-compose.yml -f docker-compose.arm.yml -f docker-compose.arm64.yml build && docker-compose -f docker-compose.yml -f docker-compose.arm.yml -f docker-compose.arm64.yml push
```
{% endtab %}
{% endtabs %}

## Running with Kubernetes

### Configuring peer deployment

Different components of the project need to be deployed on the Kubernetes cluster. You can find all service, deployment, config, and role files in the `k8s` folder in the root of the project. You can configure and change each deployment file. This includes especially the `peer-deployment.yaml` file, in which you can set multiple environment variables to configure your peer accordingly. The environment variables are described in the table below:

| ENV Name | Description | Values |
| :--- | :---: | :---: |
| STORAGE | Type of data storage to fetch Tangle data \(e.g., weights\) | ipfs |
| MESSAGE\_BROKER | Message passing used to transmit Tangle messages \(e.g., new transaction announcements\) | ipfs |
| MODEL | Machine learning \(or mock\) module used for training | no\_tf or femnist |
| TIMEOUT | Timeout for connections to the IPFS daemon | None or a positive integer |
| TRAINING\_INTERVAL | Training interval period | positive integer |
| NUM\_OF\_TIPPS | Number of preceding tips approved by a transaction | positive integer |
| NUM\_OF\_SAMPLING\_ROUND | Number of transactions that are considered for determining the current consensus  | positive integer |
| ACTIVE\_QUOTA | Probability of one peer becoming active in each training period | Floating point number between 0 and 1  |
| CONNECTION\_PRUNING\_LOW | Minimum number of open IPFS peer connections \(see [IPFS](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmconnmgrlowwater)\) | positive integer |
| CONNECTION\_PRUNING\_HIGH | Maximum number of open IPFS peer connections \(see [IPFS](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmconnmgrhighwater)\) | positive integer  |
| CONNECTION\_PRUNING\_GRACE\_PERIOD | Time in seconds after which open IPFS peer connections become relevant for pruning \(see [IPFS](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#swarmconnmgrgraceperiod)\) | positive integer |
| LOGGER | Log output destination | print or file |
| LOGGING\_LEVEL | Logging verbosity | info or warn or error |
| IPFS\_DATASTORE\_STORAGEMAX | Limit for the storage usage of each IPFS peer \(see [IPFS](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#datastorestoragemax)\) | e.g. 10GB |
| IPFS\_DATASTORE\_GCPERIOD | How often IPFS garbage collection is run \(see [IPFS](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md#datastorestoregcperiod)\) | e.g. 1h |
| IPFS\_REPROVIDER\_INTERVAL | How often provided blocks are re-announced to the IPFS network \(see [IPFS](https://github.com/ipfs/go-ipfs/blob/master/docs/config.md)\) | e.g. 1h |

### Setting up image pull secret

A Kubernetes cluster uses the secret of `docker-registry` type to authenticate with a container registry to pull a private image.

If you already ran `docker login`, you can copy that credential into Kubernetes:

```bash
kubectl create secret generic regcred \
    --from-file=.dockerconfigjson=<path/to/.docker/config.json> \
    --type=kubernetes.io/dockerconfigjson
```

{% hint style="info" %}
You can find more about pulling images from a private registry in the official Kubernetes [documentation](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#log-in-to-docker).
{% endhint %}

### Deploy components on cluster

To deploy different components like B_ootstrap_, _Peer_, _Grafana_, _Prometheus_, and _Visualization_ on the cluster you can execute the following command from the root of the project:

```bash
kubectl apply -f k8s/
```

{% hint style="info" %}
If you want to deploy on a specific architecture \(e.g., arm64\) change the image names in `k8s/*-deployment.yaml`
{% endhint %}

To scale the number of peers you can use this command:

```bash
kubectl scale --replicas=<number-of-replicas> -f k8s/peer-deployment.yaml
```

## Accessing the Services on Kubernetes

After successfully deploying all the components, you can access each component using its designated service. Each service uses the `LoadBalancerIP` and the `port` it is exposing. In other words, by accessing `http://<Service-LoadBlancerIP>:<Service-Port>` in your browser you will be able to reach each service's HTTP server. In the table below you can see each service and the port number exposed by default:

| Service Name | Port Number | Reachable from Outside of  the Cluster |
| :--- | :---: | :---: |
| Bootstrap | 4001 |  ❌ |
| Grafana | 3000 | ✅ |
| Visualisation | 9000 | ✅ |
| Peer | 52342 | ❌ |
| Prometheus | 9090 | ✅ |

Note that there is no need to reach every service from the outside of the cluster. Services for Peer and Bootstrap are used internally for services discovery and pass data between pods.

