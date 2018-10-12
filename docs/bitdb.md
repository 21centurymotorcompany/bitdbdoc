---
id: bitdb
title: BitDB
sidebar_label: BitDB
---

**Random Access Memory for Bitcoin**

[https://bitdb.network](https://bitdb.network)

<br>

![ram](assets/ram.png)

<br>

> Bitcoin is the most secure hard disk that will ever exist. 
>
> BitDB is the memory that lets you make the most out of it.

---

## Hello BitDB

Before diving in, here are some useful links that may be helpful in your journey with BitDB:

> **Twitter:** Follow [@_unwriter](https://twitter.com/_unwriter) on Twitter for bitdb related announcements
>
>**Github:** [BitDB Github Repository](https://github.com/21centurymotorcompany/bitd). BitDB is 100% open sourced.
>
>**Telegram:** Have questions? [Join the chat](https://t.me/joinchat/HH1DDQ8pZlSlsdNcKgIcxw) and meet other BitDB users and developers.

---

## What

BitDB is an autonomous database that continuously synchronizes itself with Bitcoin, providing a flexible, fast, powerful, and portable query interface into the Bitcoin universe.

By **functioning as a "Memory" to Bitcoin's "Hard Disk"**, BitDB enables a whole new category of powerful in-memory data processing that used to be impossible with Bitcoin's JSON-RPC alone, and makes it as simple as a MongoDB query.

![arch](assets/arch.png)

---

## Why

Why would one need a "Random Access Memory" on top of Bitcoin? What problems does BitDB solve?

It's actually the same reason why computer systems have RAM (working memory) on top of hard disk (long term memory).

Let's start with an example:

### 1. Problem Example

Bitcoin's transaction inputs and outputs are encoded in a low level language called "script" which looks like this:

```
OP_DUP OP_HASH160 20 0x3e2288390ab7a25cad792e662431db4bda0b9c6b OP_EQUALVERIFY OP_CHECKSIG
```

Bitcoin validates this transaction by walking through this script from left to right, computing each push data based on the opcodes and bitcoin script rules. While this is the simplest and most elegant way to implement programmable money, this elegance comes at a cost:

**Bitcoin script is not optimized for letting the "outside world" make sense of and take programmatic actions through powerful queries into Bitcoin.**

Another way of saying this is: it's not easy to build sophisticated user-facing Bitcoin applications without building out some complex custom backend infrastructure to index and process all the data. For example, trying to find an answer to a problem like the following has been impossible:

```
Find all the transactions that were:
- Sent from the address qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe
- AFTER block 543210
- That contain an OP_RETURN output that contains the text "SLP"
```

This is just one example of what's not easily possible with Bitcoin today, but there are many more.

### 2. Three Types of Databases We Need

While we're at it, let's use our imagination and think of some hypothetical database systems for bitcoin that would be extremely useful to have in the future but don't exist yet:

1. **Bitcoin Transaction Database:** We need a powerful modern database interface for interacting with Bitcoin. Bitcoin's own JSON-RPC API provides some data but it's mostly limited to simple key-value queries about the chain state. It's not meant for massive queries that require huge memory consumption. It doesn't let us query into the entire universe of bitcoin transactions as a whole, and doesn't let us filter transactions using a powerful query language. What we need is an abstraction on top of Bitcoin which most bitcoin application developers don't even need to think about and use like a regular database to build their own decentralized apps.

2. **Bitcoin Script Database:** We need a database of every bitcoin script content from every transaction. This will let us build external applications that seamlessly parse, query, and interact with bitcoin scripts (both input scripts and output scripts) through powerful queries and even use the database to programmatically power real world actions. This database would **index every single push data of every single bitcoin script** and provide a flexible query interface.

3. **Bitcoin Graph Database:** We need a database that keeps track of the entire **graph structure between all bitcoin transactions**, such as which input is connected to which output, which address is connected to which input/outputs, etc. and allow for graph aggregation and traversal queries and analysis.

<br>

Good news: You don't have to wait because it's already here, it's called BitDB.

BitDB is all three of those databases in one:

- BitDB is a [Bitcoin Transaction Database](indexer#level-1-transaction)
- BitDB is a [Bitcoin Script Database](indexer#level-2-script)
- BitDB is a [Bitcoin Transaction Graph Database](indexer#level-3-graph)

---

## How

Now let's take a look at how BitDB actually works, and how it solves all your problems.

### 1. Solution Example

With BitDB, problems like:

```
Find all the transactions that were:
- Sent from the address qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe
- AFTER block 543210
- That contain an OP_RETURN output that contains the text "SLP"
```

can be solved with a single NoSQL query, as simple as:

```
{
  "v": 2,
  "q": {
    "find": {
      "in.e.a": "qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe",
      "blk.i": {
        "$gt": 543210
      },
      "$text": {
        "$search": "SLP"
      }
    }
  }
}
```

It's just a regular MongoDB query wrapped in a meta-query object "q":

- `"in.e.a"` stands for **input edge address**
- `"blk.i"` stands for **block index**

This type of powerful query is possible because BitDB crawls, parses, and indexes all Bitcoin transactions in a structured document format, to which you can run any complex queries.

> Learn more about BitDB Document Format [here](https://docs.bitdb.network/docs/indexer)
>
> Learn more about BitDB Query Language [here](https://docs.bitdb.network/docs/query_v3)

Also, since the query language itself is JSON, the query is portable, can be implemented in any programming language, and can even be sent over HTTP. BitDB nodes can provide HTTP API endpoints that decentralized 3rd party applications can connect to.

The standardized database query language protocol ensures that your app lives on forever even after you move on, because all that's required to run the application is the query, which itself is portable because of its declarative nature.

And just the fact that this is possible gives users the trust that their favorite decentralized app won't just disappear tomorrow.

### 2. Architecture Overview

![ciq](assets/ciq.png)

Here's a high level overview of BitDB:

1. **[Crawler](crawler):** BitDB crawls through every bitcoin transaction on the blockchain and turns bitcoin's 1-dimensional script into a 2-dimensional JSON object.
2. **[Indexer](indexer):** The deserialized JSON object is stored and indexed in a NoSQL database.
3. **[Query Engine](query):** Once the data is stored, it becomes trivial to make flexible filtering, aggregation, full text search, and graph queries.

### 3. BitDB is Readonly

Humans can't directly write to BitDB. The only "user" with "write" permission to BitDB is **Bitcoin** itself.

BitDB utilizes Bitcoin as the single source of truth and doesn't require any human intervention once it starts running as an autonomous daemon because it's a self-contained crawler + parser + indexer + query engine, all in one. 


With this approach, the "database" is merely an index built from the canonical Bitcoin blockchain, just like how web search engines are merely indexes built from websites they crawl and the "source of truth" is always the original website.

### 4. Writing to Bitcoin

So if Bitcoin is the only entity that can write to BitDB, how can a lowly human being get data into BitDB?

Simple, you go directly to the source (Bitcoin) and write to it. And how do you write to Bitcoin? Make a Bitcoin transaction!

You'll see that BitDB immediately synchronizes with Bitcoin and your transaction shows up on BitDB instantly. Here are some libraries that provide various flexible ways to WRITE to Bitcoin:

<div class='well'>

#### i. BitBox SDK

https://developer.bitcoin.com/bitbox

<img src='assets/bitbox.png' class='text-left'>

---

#### ii. Bitcoincash.js

https://bitcoincashjs.github.io/

<img src='assets/bitcoincashjs.png' class='text-left'>

</div>

### 5. BitDB is Bitcoin

In essence, BitDB is a database constructed by a one-way synchronization function that transforms a raw Bitcoin transaction into a structured data format that can be queried against.

![function](assets/function.png)

This means we can reconstruct identical BitDB databases as many times as we want from Bitcoin. If there's a nuclear war and every single BitDB node goes down, as long as there are Bitcoin nodes running, you can recover from the destruction by traversing the transform function through time (ie: the blockchain).

- Because BitDB is backed by Bitcoin as canonical storage, it comes with all the benefits of Bitcoin's decentralization (Anyone can reconstruct and run a BitDB node from scratch as long as they have a Bitcoin node)

- Because BitDB is powered by MongoDB as index, it comes with all the benefits of NoSQL databases, such as highly flexible and portable query interface and better user experience.


