import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

:::info Already running a node?

This guidance is targeted at new users. If you're already running a node, see [Prepare for The Merge](../../prepare-for-merge.md).

:::

<br />

First, <strong>select a quickstart configuration</strong>. If you're looking for the simplest configuration, select `Geth` and `IPC`:

import MultidimensionalContentControlsPartial from '@site/docs/partials/_multidimensional-content-controls-partial.md';

<MultidimensionalContentControlsPartial />

## Introduction

Prysm is an implementation of the [Ethereum proof-of-stake consensus specification](https://github.com/ethereum/consensus-specs). In this quickstart, you’ll use Prysm to run an Ethereum node and optionally a validator. This will let you stake 32 ETH using hardware that you manage.

This is a beginner-friendly guide. Familiarity with the command line is expected, but otherwise this guide makes no assumptions about your technical skills or prior knowledge.

At a high level, we'll walk through the following flow:

 1. Configure an **execution node** using an execution-layer client.
 2. Configure a **beacon node** using Prysm, a consensus-layer client.
 3. Configure a **validator** and stake ETH using Prysm (optional).

<br />

:::info Knowledge Check

**Not familiar with nodes, networks, and related terminology?** Consider reading [Nodes and networks](../../concepts/nodes-networks.md) before proceeding. 

:::
