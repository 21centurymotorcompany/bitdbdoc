---
id: diy
title: BitDB vs. DIY Custom Backend
sidebar_label: BitDB vs. DIY Custom Backend
---

---

## Obvious Benefits

### A. Universal General Purpose Bitcoin Database

First, if you're trying to build any sort of sophisticated Bitcoin powered application, there currently is no way to do what BitDB does unless you decide to spend weeks to months setting up your own custom infrastructure from scratch. 

Currently JSON-RPC is the only way to query the blockchain, but it only gives you simple key/value queries, and there is no way to filter and query the entire blockchain as a whole.

This means anyone who wants to build a sophisticated Bitcoin app will need to build out their own custom backend infrastructure to index the blockchain.

What's needed is a fully indexed, full-fledged, general purpose high level database that lets you query the entire blockchain as a whole.

That's BitDB.

### B. Open Source and Decentralized

Second, BitDB is completely open source. There's a free public API endpoint you can use (https://bitdb.network) but the point is that anyone can run a BitDB node if they have a Bitcoin node running. The future plan is to encourage proliferation of various public nodes.

Eventually you won't even need a server to build an app. Imagine building apps with no server to maintain, yet you still make money from users transacting in bitcoins.

---

## Non-Obvious Benefits

### A. Portable

Without something like BitDB, application developers need to write their entire backend from scratch. This "backend" includes:

1. Bitcoin full node
2. Custom crawler
3. Custom indexer
4. Custom database
5. Custom API interface

What does this all mean? Having such a complex backend that's run only on your own server means it's no longer "decentralized" nor "censorship resistant". Your application is only online as long as you keep running your server.

For this reason, **you probably would want to open source your entire backend.** But there are two issues here:

1. **Open sourcing is A LOT of work:** You need to clean up your code, write documentation, and make sure people understand what's going on inside the code if your goal of open sourcing is to make your application truly decentralized. Generally, the process of open sourcing takes way more time and energy than the process of building itself.
2. **Who would run it when you move on?:** Imagine what will happen when you decide to move on from your project. Who will take over? This is where BitDB can potentially shine. Because BitDB is a general purpose database, it can be used for ANY Bitcoin app. So if you decide to shut down your application, you can redirect your users to another BitDB node seamlessly. But if you roll your own backend, you can't take advantage of this potential network effect.

This is why portability matters. 

- If one backend goes down, it should be trivial for the app to migrate to another backend, instantly.
- Even if NO public website exists, users should be able to use the app by directly serving from Bitcoin.
- Just like how Bitcoin is forkable, the backend for these decentralized apps should also be forkable to resolve governance issues.

Using a shared standard backend scheme such as BitDB is great for portability because it means there will be various compatible endpoints you can plug into, and this network effect will grow stronger as more apps arise who run their own BitDB node.

### B. Open Source by Default

#### 1. Open Source Backend by Default

First, BitDB itself is 100% open sourced and the goal is to lower the barrier to running BitDB nodes as much as possible. This means adopting BitDB makes your application automatically open source. No need to clean up your backend code, write documentation, and all the hard work that come with open source maintenance.

The only additional thing you need to do to make it completely decentralized is to open source the frontend (Remember, there is no "server"). In most cases you don't even have to do anything here because if you're building a web app, the code is open by default as HTML anyway.

#### 2. Open Source Protocol by Default

But there's more. It's important to note that simply open sourcing on its own doesn't guarantee decentralization. There are a lot of projects out there that are completely open source, but doesn't contribute to decentralization because to actually use the technology it makes more economic sense for users to be locked into the vendor who released the open source code in the first place.

In order to ensure true decentralization, the entire application logic needs to be portable so it doesn't lead to any lock-in.

This is where BitDB's declarative query language (a JSON-based meta-language over MongoDB query language) comes into play. There is no proprietary programming language to interact with BitDB. You simply formulate your query in JSON and can build a bitdb powered app using the language or framework of your choice, simply by making an HTTP request to a BitDB node endpoint.

Because each query is 100% self contained, you can describe everything about your application protocol with a set of BitDB queries. This way, you can enjoy the following benefits:

- **3rd party client ecosystem:** if someone wants to build an alternative client for your application protocol, they can easily do so.
- **Mashup:** if someone wants to build a mashup app between your application protocol and another bitcoin application protocol, they can also easily do so, simply by combining the bitdb queries.
- **Seamless Protocol Upgrade:** You can easily upgrade your protocol simply by updating the query constructs, instead of rewriting the database schema and the backend. Everything takes place in the frontend, completely server-less.
- **Protocol Validation:** You can use JSON schema to validate incoming bitcoin transactions that are valid according to your own application protocol (Without BitDB, you need to write a whole program to do this, which again is NOT portable)


### C. Truly Decentralized

In this world, decentralized app developers won't even need to run their own node. Of course, you could run a node if you wanted to. And the more nodes the more resilient the entire ecosystem is.

But the point is: in this future, it is no longer a relevant issue whether a developer runs a server or not. There is no single party that runs a backend for an app. We all do.

Just like how not all users need to run their own bitcoin node, not every developer should need to run their own BitDB node. Applications can be run as a purely frontend applications that connect to best available BitDB nodes. Applications become portable. Portability leads to censorship resistance and decentralization.

Because the destiny of an application no longer depends on whether the original developer keeps maintaining the server or not, it becomes possible to create truly serverless and eternal applications which can last forever even after the original developer moves on.

Most importantly, BitDB is Bitcoin. BitDB doesn't compromise Proof of Work. No need for political or oligarchy schemes to try to implement distributed consensus, which will have no place in the world where Bitcoin scales to infinity.
