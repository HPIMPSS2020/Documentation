# Application

## Motivation

Decentralized learning with a Tangle for communication works in theory as proven by Schmid et al. However, in order to evaluate the practicability of this approach further aspects such as its performance with regards to "faults introduce by real-world network condition \[...\] \[and\] resource efficiency" \(Schmid et al., 2020\) need to be assessed. 

Fort this reason the prototype [Learning Tangle](https://gitlab.hpi.de/osm/mpss2020/ipfs-learning-tangle) was created based on the architecture and technology stack outlined in the previous chapter. It was forked from the project [Tangle Learning](https://gitlab.hpi.de/osm/tangle-learning/learning-tangle). The initial prototype worked on a very small scale \(less than four peers\) and only for a short duration. Therefore, most of the work on creating Learning Tangle went into changes required for scaling the initial prototype to a network size that allowed us to verify the findings of Schmid et al.

Our initial goal was to simulate a use case for decentralized learning such as the one described in the chapter [D-EUSS](../../background/use-case.md). For this, we worked on scaling the number of peers that the prototype network can handle, the time it runs before becoming unstable and hence the total number of trainings performed and transactions published to the Tangle.

In order to practically enable this simulations we worked on enabling both a vertical scaling scenario in which we used a large workstation for running a very high number of peers on one node as well as a horizontal scaling scenario in which we built an orchestration system that enables clusters of hetergeneous nodes to run `n` peers on `m` hosts \(`n>=m`\) with real-world network connections between them. 

Both approaches are described in more detail in the remainder of this chapter.  

\*\*\*\*

