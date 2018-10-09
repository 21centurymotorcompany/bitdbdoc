---
id: query
title: Query Engine
sidebar_label: Query Engine
---

BitDB is powered by MongoDB.

On top of MongoDB's own query language, BitDB's query language adds one additional layer to create a completely self-contained and portable query language. The additional bitdb layer lets you express the query language protocol version, query encoding per push data, and the query itself.

- BitDB internally uses MongoDB to index the transactions in a structured manner. This means you can query it just like any regular MongoDB collection.
- BitDB supports most MongoDB methods such as find, aggregate, sort, limit, project, etc.
- To query the database, you simply make a request to BitDB with a JSON-based query language that looks like this:

![ql](assets/ql.png)


## 1. BitDB Document Format

Before looking at the query language, first make sure you understand how a bitcoin transaction is decoded and transformed and stored into a query-optimized format in BitDB.

You can learn about BitDB document format in the Indexer section: [Learn Indexer](indexer)

## 2. BitDB Query Language

A BitDB query is a self-contained declarative JSON query object built on top of MongoDB query language. Every query consists of 3 top level attributes:

1. **v:** stands for "version". This documentation is based on v: 3 so always use v: 3 (v:1 and 2 will be deprecated)
2. **q:** stands for "query".
    - **find:** MongoDB query filter object. Learn more about MongoDB query filter
    - **aggregate:** MongoDB aggregationg pipeline stages array. Learn more about Mongodb aggregate stages
    - **project:** MongoDB project operator for selectively returning attributes. Learn more about Mongodb projection
    - **sort:** MongoDB sort operator. Learn more about Mongodb sort operator
    - **limit:** MongoDB limit operator. Limit the number of results to return
    - **skip:** MongoDB skip operator. Combining the "limit" and "skip" you can implement pagination
    - **db:** The array of database types to query against. The default is ["c", "u"] (each representing "confirmed" and "unconfirmed" respectively), but you can explicitly query against only one of them, using ["c"], or ["u"]
3. **r:** stands for "response". response processor.
  - **f:** stands for "filter". This is a single line filter string written in [jq](https://stedolan.github.io/jq), **a turing complete, stack based JSON processing language.**

### A. Encoding

In order to handle encoding, BitDB introduces virtual variables with **h-prefix** (h0, h1, h2, h3, ...) which are hex encoded representation of the b0, b1, b2, b3, etc. counterparts.

Internally these h-variables are not stored in the DB but generated at query-time and response-time. Here's an example:

```
{
  "v": 3,
  "q": {
    "find": {
      "out.h1": "6d02"
    }
  }
}
```

Above query is transformed at query time into:


```
{
  "v": 3,
  "q": {
    "find": {
      "out.b1": "bQI="
    }
  }
}
```

And then sent to MongoDB (which only contains b-prefixed and s-prefixed attributes). Then it returns the response that looks like this:

```
{
  ...
  confirmed: [{
    ...
    out: [{
      b0: {
        op: 106
      },
      b1: "bQI=",
      ...
    }]
    ...
  }]
  ...
}
```

But the query engine doesn't stop here. It decodes above response to attach additional h-prefixed attributes, interpreted from the returned b-attributes. So the final response that's returned as the result is:

```
{
  ...
  confirmed: [{
    ...
    out: [{
      b0: {
        op: 106
      },
      b1: "bQI=",
      h1: "6d02"
      ...
    }]
    ...
  }]
  ...
}
```

This way, BitDB implements virtual variables without having to store them in the db.


### B. Query

The query object becomes translated into native MongoDB API call.

Currently supported query API methods are: `find`, `aggregate`, `project`, `sort`, `limit`, and `skip`, but this can be expanded according to need.

To learn how to query BitDB, you can just think of this as a regular MongoDB instance. You can learn more about MongoDB queries [here](https://docs.mongodb.com/manual/tutorial/query-documents/)


### C. Response Handling

With version 3, Bitquery introduces **"response filter"**. Previously it was not easy to filter the response so that it can be used immediately in the client-side without any additional program. The only way you could do this was by using MongoDB's `aggregate` queries to write a huge sequence of complicated pipelines, but this was bad because:

1. It's too convoluted
2. It takes up too much resource on the database side

The new approach is to let the database just make simple queries, and on top of that introduce an adapter that processes the database response before returning to the client.

And **BitDB chooses a turing complete, stack based JSON processing language called [jq](https://stedolan.github.io/jq/).** Here are the benefits of jq:

1. **Powerful:** It's Turing Complete, meaning you can probably transform any JSON into any format you want.
2. **Efficient:** It offloads the work from the database by doing all the filtering outside of the DB engine.
3. **Portable:** Most importantly, the language fits into JSON format as a single line of string, thanks to its stack based programming paradigm. This is important for building decentralized applications on top of Bitcoin because a query API and its response format needs to be as self-descriptive as possible.

Here's an example of a jq-powered response filtering:

Find 100 transactions with an output OP_RETURN script that starts with `0x6d02` (memo.cash). And then return only its block index, block time, and content.

```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 100
  },
  "r": {
    "f": "[{ block: .blk.i?, timestamp: .blk.t?, content: .out[1].s2 }]"
  }
}
```

---

## 3. Example

> All examples below contain **"v": 3**, (version 3) you should also always include the version number ("v": 3 at the moment) in all your queries to future-proof your applications.

### A. queries

---

Find 10 OP_RETURN transactions that contains "hello" as the second push data:

1. The first push data (b0) is `{"op": 106}` (OP_RETURN)
2. The second push data in UTF encoding (s1) is "hello"

[Try Query](https://bitdb.network/v3/explorer/ewogICJ2IjogMywKICAicSI6IHsKICAgICJmaW5kIjogewogICAgICAib3V0LmIwIjogeyAib3AiOiAxMDYgfSwKICAgICAgIm91dC5zMSI6ICJoZWxsbyIKICAgIH0sCiAgICAibGltaXQiOiAxMAogIH0KfQ==)

```
{
  "v": 3,
  "q": {
    "find": {
      "out.b0": { "op": 106 },
      "out.s1": "hello"
    },
    "limit": 10
  }
}
```

---

Find 10 transactions where the second push data is "6d02" in hex encoding

[Try Query](https://bitdb.network/v3/explorer/ewogICJ2IjogMywKICAicSI6IHsKICAgICJmaW5kIjogeyAib3V0LmgxIjogIjZkMDIiIH0sCiAgICAibGltaXQiOiAxMAogIH0KfQ==)

```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 10
  }
}
```

---

Find 10 transactions where [the second push data is "00424554"](https://github.com/fyookball/ChainBet/blob/master/PROTOCOL.md#op_return-communication-messages)

[Try Query](https://bitdb.network/v3/explorer/ewogICJ2IjogMywKICAicSI6IHsKICAgICJmaW5kIjogeyAib3V0LmgxIjogIjAwNDI0NTU0IiB9LAogICAgImxpbWl0IjogMTAKICB9Cn0=)


```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "00424554" },
    "limit": 10
  }
}
```

---

Find 10 transactions where the second push data is "6d02" in hex encoding, and the second push data matches "bet" in UTF8 (Note that it's combined with a full text search query for efficiency. Learn more about speeding up MongoDB regular expression queries here: [How to Speed-Up MongoDB Regex Queries by a Factor of up-to 10](https://medium.com/statuscode/how-to-speed-up-mongodb-regex-queries-by-a-factor-of-up-to-10-73995435c606))

[Try Query](https://bitdb.network/v3/explorer/ewogICJ2IjogMywKICAicSI6IHsKICAgICJmaW5kIjogewogICAgICAiJHRleHQiOiB7ICIkc2VhcmNoIjogImJldCIgfSwKICAgICAgIm91dC5oMSI6ICI2ZDAyIiwKICAgICAgIm91dC5zMiI6IHsgIiRyZWdleCI6ICJiZXQiLCAiJG9wdGlvbnMiOiAiaSIgfQogICAgfSwKICAgICJsaW1pdCI6IDEwCiAgfQp9)

```
{
  "v": 3,
  "q": {
    "find": {
      "$text": { "$search": "bet" },
      "out.h1": "6d02",
      "out.s2": { "$regex": "bet", "$options": "i" }
    },
    "limit": 10
  }
}
```

---

Find 10 transactions with an input script with the sender `qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe`

[Try Query](https://bitdb.network/v3/explorer/ewogICJ2IjogMywKICAicSI6IHsKICAgICJmaW5kIjogeyAiaW4uZS5hIjogInFxNGtwM3czeWhodnk0Z200amdlemE0dnVzOHZweGdyd2M5MG44cmh4ZSIgfSwKICAgICJsaW1pdCI6IDEwCiAgfQp9)

```
{
  "v": 3,
  "q": {
    "find": { "in.e.a": "qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe" },
    "limit": 10
  }
}
```

---

### B. queries + response filter

In addition to the `q` attribute, we have a response handler called `r`.

Currently the `r` attribute only has a subattribute called `f`, which stands for "filter", meaning it's used for filtering the response before returning to the client.

For this purse we use a turing complete, stack based JSON processing language called [jq](https://stedolan.github.io/jq/), which is normally used as a command line tool for processing JSON with a single line command. Here are some examples:


Find 100 transactions with an output OP_RETURN script that starts with `0x6d02` (memo.cash). And then return only its block index, block time, and content.

```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 100
  },
  "r": {
    "f": "[ { block: .blk.i?, timestamp: .blk.t?, content: .out[1].s2 } ]"
  }
}
```


More complex example:

1. Find 100 OP_RETURN transactions that start witn `0x6d02` (memo.cash)
2. Group by block index (blk.i)
3. Map the items to only return the message (.out[1].s2) and transaction hash (.tx.h)

```
{
  "v": 3,
  "q": {
    "db": ["c"],
    "find": { "out.h1": "6d02" },
    "limit": 100
  },
  "r": {
    "f": "[ group_by(.blk.h)[] | { blocks: { (.[0].blk.i | tostring): [.[] | {message: .out[1].s2, tx: .tx.h} ] } } ]"
  }
}
```


## 4. DB-side filtering

Another thing you can do to filter the query response is ask the database to do it. This can be done using MongoDB's `project`.

Unlike the `response filter` from the last section which takes place on the API server before responding to the client, the DB-side filtering actually takes place on the database engine therefore sometimes may be helpful (but not always)

Just a caution: Adding too much stuff to the query via `aggregate`, etc. can put a large load on the server, so generally the `response filter` from the last section is the recommended way to go (The response filter is fully capable of manipulating the response in all kinds of ways since it's powered by a turing complete JSON processing language called [jq](https://stedolan.github.io/jq).

However, simple `project` queries are generally good because it reduces the DB load.

### A. 'project' reduces the DB load

'project' reduces the amount of data the database has to send back, even before it reaches the response filter. The smaller this size is the faster the query will be.

### B. 'project' can return ONLY the matched subtree

For example, you may be looking for a transaction that matches a certain OP_RETURN pattern in its output.

By default all BitDB queries return the entire transaction, so you may want to ask it to only return the matched OP_RETURN output. You can do this by using MongoDB's [Positional Operator $](https://docs.mongodb.com/manual/reference/operator/projection/positional/).

[Try Query](https://bitdb.network/v3/explorer/ewogICJ2IjogMiwKICAiZSI6IHsgIm91dC5iMSI6ICJoZXgiIH0sCiAgInEiOiB7CiAgICAiZmluZCI6IHsgIm91dC5iMSI6ICI2ZDAyIiB9LAogICAgImxpbWl0IjogMTAsCiAgICAicHJvamVjdCI6IHsKICAgICAgIm91dC4kIjogMQogICAgfQogIH0KfQ==)

```
{
  "v": 3,
  "q": {
    "find": { "out.h1": "6d02" },
    "limit": 10,
    "project": { "out.$": 1 }
  }
}
```
