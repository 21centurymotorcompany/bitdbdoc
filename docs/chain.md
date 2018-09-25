---
id: chain
title: What chain does BitDB support?
sidebar_label: What chain does BitDB support?
---

Technically BitDB is compatible with all forks of Bitcoin, which would include BTC, BCH, and many other blockchain projects derived from Bitcoin.

However, the current implementation of BitDB runs on Bitcoin Cash (BCH) network.

This decision was based on pure return on investment assessment in terms of what BitDB is trying to achieve. Here's the thought process:

## 1. Bitcoin as Global Database of Everything

The first goal of BitDB is to provide a user friendly and developer friendly high level API abstraction layer for dealing with large amounts of trust-less and immutable data.

BitDB works best if it is aligned with the underlying ledger's roadmap. For example on BTC, using Bitcoin's blockchain to do things other than money transfers is not encouraged and regarded as spam.

Also, to be able to use Bitcoin as a "database alternative", the most important factor is the cost of insertion (in database terms). The best chain to build a database system on top of is the one with the most consistent and deterministic insertion cost (transaction fee). This fee determinism is important especially because BitDB is trying to make everything programmable.

Currently the only fork of Bitcoin where this somewhat radical vision of "Bitcoin as a global database of everything" is welcome is BCH (Bitcoin Cash).

## 2. Bitcoin as Global Railway of Everything

BitDB indexes everything, from Bitcoin transaction itself to Bitcoin input/output scripts to the graph structure between transactions. And most of this has to do with ON-CHAIN transfer of bits.

Here's the current landscape of bitcoin ecosystem and why BitDB is valuable or not in each chain:

- The BTC roadmap is to avoid scaling on-chain and scale through lightning network to transfer bits. Lightning network is not a ledger for storing data, so BitDB doesn't add any value to lightning network, and therefore to BTC. 
- The BCH roadmap is to scale through 100% on-chain scaling, with the bet that free market capitalism will find ways to scale. If everything is 100% on-chain, BitDB can be very helpful. 

## 3. Bitcoin that scales infinitely

It's important to note the main assumption of BitDB is that its underlying chain WILL scale infinitely without sacrificing security provided by Proof of Work.

Just like having a high performance RAM won't do anything if your disk doesn't work, the chains with fundamentally unscalable models won't get much help from BitDB because they need to figure out their fundamental scaling problem first.

Today the only chain that matches this vision is BCH, and this is why the bitdb.network node is running on top of BCH network today.

