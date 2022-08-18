---
id: checkpoint-sync-concept
title: "Checkpoint sync: Conceptual overview"
sidebar_label: "Checkpoint sync"
---
import CheckpointSyncPresent from '@site/static/img/checkpoint-sync-present.png';
import CheckpointSyncAbsent from '@site/static/img/checkpoint-sync-absent.png';
import NetworkPng from '@site/static/img/network.png';
import BlockchainSimplified from '@site/static/img/blockchain-simplified.png';
import ClientStackPng from '@site/static/img/client-stack.png';
import EpochsBlocks from '@site/static/img/epochs-blocks.png';
import Finality from '@site/static/img/finality.png';

import {HeaderBadgesWidget} from '@site/src/components/HeaderBadgesWidget.js';

# Checkpoint sync: Conceptual overview

<HeaderBadgesWidget commaDelimitedContributors="Mick,Potuz" />

:::info Looking for the how-to?

This is a conceptual overview. See [How to sync from a checkpoint](../prysm-usage/checkpoint-sync.md) to learn how to configure checkpoint sync. 

:::

**Checkpoint sync** is a feature that significantly speeds up the initial sync between your beacon node and the Beacon Chain. With checkpoint sync configured, your beacon node will begin syncing from a recently finalized checkpoint instead of syncing from genesis. This can make installations, validator migrations, recoveries, and testnet deployments *way* faster.

This conceptual overview will help you understand **how checkpoint sync works** and **the value that it provides to the Ethereum network**. We'll speed-run through some foundational concepts before digging into checkpoint sync.

## Foundations

### Ethereum, nodes, and networks

Ethereum is a decentralized **network** of **nodes** that communicate via peer-to-peer connections. These connections are formed by computers running Ethereum's specialized client software (like Prysm):

<img style={{maxWidth: 461 + 'px'}} src={NetworkPng} />

An Ethereum **node** is a running instance of Ethereum's client software. This software is responsible for running the Ethereum blockchain. There are two primary types of nodes in Ethereum: **execution nodes** and **beacon nodes**. Colloquially, a "node" refers to an execution node and beacon node working together:

<img src={ClientStackPng} /> 

Nodes can be configured to run on one of Ethereum's many networks. Production apps and real ETH live on Ethereum Mainnet, while test networks allow protocol and application developers to test new features using fake ETH. See [Nodes and networks](nodes-networks.md) for a more detailed refresher on Ethereum's nodes and networks.


### Blockchain data structure

Ethereum's "world state" is stored in a blockchain data structure. At the most rudimentary level, a blockchain is a [linked list](https://en.wikipedia.org/wiki/Linked_list) on steroids. Like a linked list, Ethereum's blockchain is a sequence of items that starts with a "tail" and ends at its "head". Ethereum's tail is called its **genesis block** - the first block that was ever created. Its head is referred to as the "head of the chain":

<img src={BlockchainSimplified} />

Ethereum's nodes work together to process one batch of transactions at a time. These batches are Ethereum's blocks. As users and applications submit transactions to the network, new blocks are proposed and ultimately finalized.


### Blocks, epochs, and slots

While transactions are grouped into blocks, blocks are grouped into **epochs**. An epoch is a fixed-length timespan lasting 384 seconds. Every epoch is divided into 32 slots, each slot lasting 12 seconds. Every slot is an opportunity for one new block of transactions to be tentatively accepted by the Ethereum network:

<img src={EpochsBlocks} />

The above diagram illustrates [Epoch 1](https://ethscan.org/epoch/1) on Ethereum Mainnet. This epoch, like all other epochs, contains 32 slots. Every slot contains one block. Some blocks have been approved by the network. The block in this epoch's fourth slot, or the zero-based [slot 35 of the chain](https://ethscan.org/block/35) (31 + 4 = 35), included [154 transactions](https://etherchain.org/block/0x8d3f027beef5cbd4f8b29fc831aba67a5d74768edca529f5596f07fd207865e1#pills-txs). One of these transactions was a transfer of 3.7 ETH. Try finding this transaction and working your way back to [Epoch 1](https://ethscan.org/epoch/1).

It can be helpful to think of Ethereum as a "world computer" with a processor and hard drive. Its processor is the [Ethereum Virtual Machine](https://ethereum.org/en/developers/docs/evm/), and its hard drive is the blockchain underlying Ethereum Mainnet. Ethereum's processor makes tentative changes to its hard drive once every slot, or once every 12 seconds. These tentative changes are accepted as "canonical" (going from "truthy" to "truth") *at most* once every epoch, or once every 32 slots, or once every 384 seconds.


### Justification, finality, and checkpoints

We mentioned "tentative" and "canonical" in the previous section. Ethereum's nodes are responsible for broadcasting, verifying, and finalizing transactions as they're submitted by users and apps. **Finality** describes a state in which the probability of transaction reversal is near-zero. Transactions become finalized when they're included within a block that gets finalized.

Let's imagine that Bob wants to send Alice some ETH. In the happy scenario, Bob's transaction would flow through the following (oversimplified) transaction lifecycle:

 1. **Transaction signed**: Bob signs a transaction that moves ETH from his wallet to Alice's wallet using the private key associated with his wallet.
 2. **Transaction submitted**: Bob submits this transaction to the Ethereum network. Theoretically, all nodes receive it.
 3. **Proposer selected**: The Ethereum network protocol randomly selects a validator node to fill the current epoch's current slot with a new block. This validator node will be the only node in the world that's allowed to propose a block into this slot.  
 4. **Block created**: This randomly selected block proposer pops a batch of transactions off of its internal queue, verifies their legitimacy, and builds a new block that contains Bob's transaction.
 5. **Block proposed**: The block proposer broadcasts this proposed new block to peer nodes.
 6. **Attesters selected**: The Ethereum network protocol randomly selects a small committee of other validator nodes to attest to the legitimacy of the proposed block and the transactions it contains.
 7. **Block justified**: After a sufficient number and percentage of committee members have attested to the legitimacy of the proposed block, the block is marked as "justified".
 8. **Block finalized**: After the next epoch is justified, the epoch containing the block containing Bob's transaction is "finalized".

Familiarity with this lifecycle can help conceptualize the difference between **justified blocks**, **finalized blocks** and **checkpoints**. Ethereum uses checkpoints to set certain blocks "in stone". At any given time, Ethereum's current "candidate checkpoint" block is the first block in the first slot of the first epoch following the most recent checkpoint.

<img src={Finality} />


Checkpoints are created...



As soon as this first block is justified, the previous epoch's first block is marked as a finalized checkpoint
This is true if and only if the previous epoch was already justified

candidate checkpoint is the last block that is either the block at slot 0 in that epoch or before that

and all of the blocks within the previous epoch are also finalized. Is this right?
No, this is a common mistake that is even in our codebase so for someone reading our database methods is easy to get confused by this: you finalize everything that is an ancestor of the finalized checkpoint, so that means all the blocks in the previous to the previous epoch in this case

This is a very good question, I think the description above already answered this question since in fact the slots are only finalized 2 epochs before not 1, but still, there are a number of attacks that indeed happen because you are proposing the latest blocks in the Epoch. All of the known attacks (or at least the ones that I know) exploit the fact that the chain didn't have enough votes to justify before it was your turn to propose. And now you are proposing and you have the votes necessary to justify. In this case an evil proposer does not show his block, hoardes it and reveals it late. This is causing now 33 slot-reorgs on Goerli



### Peer-to-peer synchronization

When Ethereum's nodes first come online, they ask other peer nodes to provide 



## Checkpoint sync

### Syncing without checkpoint sync

Beacon nodes maintain a local copy of the Ethereum's [Beacon Chain](https://ethereum.org/en/upgrades/beacon-chain/). When you tell Prysm's beacon node to start running for the first time, Prysm will fetch the very first Beacon Chain block (the Beacon Chain's [genesis block](https://beaconscan.com/slots?epoch=0)). Your beacon node will then "replay" the history of the Beacon Chain, fetching the oldest blocks from peers until the entire chain has been downloaded:

<img src={CheckpointSyncAbsent} /> 

This sync process can take a very long time - on the magnitude of days to weeks. 

### Syncing with checkpoint sync

Checkpoint sync speeds things up by telling your beacon node to sync from a recently finalized checkpoint, allowing it to skip over the majority of the Beacon Chain's history:

<img src={CheckpointSyncPresent} /> 

Note that currently, Prysm's implementation syncs forward-only. The process of syncing backwards towards the genesis block is called "backfilling", and will be supported in a future Prysm release. Backfilling isn't required to run a validator - it's only required if you want to run an archive node, serve peer node requests, or query chain history through your local beacon node.


## Frequently asked questions

**Where can I learn more about checkpoint sync and related concepts?** <br/>

 - [Gasper](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/gasper/) describes the concept of finality in the context of Gasper, the consensus mechanism securing proof-of-stake Ethereum.
 - [What happens after finality in Ethereum PoS](https://hackmd.io/@prysmaticlabs/finality) provides an in-depth explanation of finality.
 - [Proof of Stake](https://ethereum.org/pt/developers/docs/consensus-mechanisms/pos/) is a great place to learn about Ethereum Proof of Stake.



<RequestUpdateWidget />