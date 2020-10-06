# Architecture Overview

A simplified architecture of the functional components of the prototype is visualized in the following two diagrams. The first one highlights how the components of a prototype network interact. The second one shows the software architecture of a single network peer.  

![Prototype System Architecture](.gitbook/assets/image%20%2816%29.png)

The two main components are a single bootstrap node and multiple peer nodes. Each peer node communicates with the bootstrap node to enter the network and find other peers by peer exchange. The peer performs the actual application logic. 

![Peer Architecture](.gitbook/assets/image%20%2822%29.png)

The most complicated part of the prototype is the peer. Its components include several modules for machine learning model training, a scheduler for starting periodic tasks, message broker and storage components for communication with other peers, a transaction listener to process incoming messages and a tangle data structure to store data necessary to perform machine learning and contribute to the tangle consensus.

