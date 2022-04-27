# Orbs Validator Instructions

Documentation for Orbs Guardians and node operators explaining how to launch and maintain an Orbs node.

&nbsp;

## Orbs Public Network (V2.5)

### Launch Orbs Node

To launch a **public** Orbs node and participate in the Orbs Proof-of-Stake v2.5 elections:

* [Run the node on cloud service like AWS](./public/orbs-public-blockchain.md) *(recommended)*

* [Run the node on your own infrastructure *ubuntu 18* step by step](./public/orbs-public-own-infra-steps.md)

* [Run the node on your own infrastructure](./public/orbs-public-own-infra.md)

### Firewall Port Opening
In case your node is behinf a firewall, for best performance, 
open the following ports traffic.:

* ```80``` - Services status and logs

* ```10000-10100``` - Virtual chains communication

* ```7666``` - Node management

* ```9100``` - prometheus metrics

### Common Guardian Actions using Etherscan + Metamask

* [Withdraw Guardian fees and bootstrap](./public/withdraw_fees_bootstrap.md)

* [Claim staking rewards](./public/claim_staking_rweards.md)

&nbsp;
