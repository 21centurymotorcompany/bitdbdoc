---
id: quickstart
title: Quickstart
sidebar_label: Quickstart
---

Go to https://bitdb.network/explorer and try the following queries. (Don't forget the `"v": 2`, this entire documentation is based on version 2.0, and the previous version has been deprecated)

## 1. Exact String Match

Find all transactions where the first push data is "hello" in UTF8 

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAicSI6IHsKICAgICJmaW5kIjogeyAib3V0LnMxIjogImhlbGxvIiB9LAogICAgImxpbWl0IjogMTAKICB9Cn0=)

```
{
  "v": 2,
  "q": {
    "find": { "out.s1": "hello" },
    "limit": 10
  }
}
```

## 2. Multiple Push Data Query

Find all transactions that start with "6d02" in hex as the first push data. 

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAiZSI6IHsgIm91dC5iMSI6ICJoZXgiIH0sCiAgInEiOiB7CiAgICAiZmluZCI6IHsgIm91dC5iMSI6ICI2ZDAyIiB9LAogICAgImxpbWl0IjogMTAKICB9Cn0=)

```
{
  "v": 2,
  "e": { "out.b1": "hex" },
  "q": {
    "find": { "out.b1": "6d02" },
    "limit": 10
  }
}
```

## 3. Exact Match + Full Text Search

Find all transactions where one of its outputs starts with "6d02" in hex, but also includes the string "bet" in any of its output push data. 

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAiZSI6IHsgIm91dC5iMSI6ICJoZXgiIH0sCiAgInEiOiB7CiAgICAiZmluZCI6IHsKICAgICAgIiR0ZXh0IjogewogICAgICAgICIkc2VhcmNoIjogImJldCIKICAgICAgfSwKICAgICAgIm91dC5iMSI6ICI2ZDAyIgogICAgfSwKICAgICJsaW1pdCI6IDEwCiAgfQp9)

```
{
  "v": 2,
  "e": { "out.b1": "hex" },
  "q": {
    "find": {
      "$text": {
        "$search": "bet"
      },
      "out.b1": "6d02"
    },
    "limit": 10
  }
}
```

## 4. Regular Expression + Full Text Search

It is possible to run regular expression matches on fields. But due to the way MongoDB works, running pure regular expression queries can be inefficient unless efficiently structured. (Related Article)

It is recommended that you use it in conjunction with a full text search to filter it down to a subset that matches the full text search results, and then apply the regex operation. (Also, make sure to read the linked article above)

Example: Find all transactions where the second push data matches the UTF8 regular expression `/hello.*/i` 

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAicSI6IHsKICAgICJmaW5kIjogewogICAgICAiJHRleHQiOiB7CiAgICAgICAgIiRzZWFyY2giOiAiaGVsbG8iCiAgICAgIH0sCiAgICAgICJvdXQuczIiOiB7CiAgICAgICAgIiRyZWdleCI6ICJoZWxsby4qIiwgIiRvcHRpb25zIjogImkiCiAgICAgIH0KICAgIH0sCiAgICAibGltaXQiOiAxMAogIH0KfQ==)

```
{
  "v": 2,
  "q": {
    "find": {
      "$text": {
        "$search": "hello"
      },
      "out.s2": {
        "$regex": "hello.*", "$options": "i"
      }
    },
    "limit": 10
  }
}
```

## 5. Query by Address

Find all transactions sent from the address `qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe`

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAicSI6IHsKICAgICJmaW5kIjogewogICAgICAiaW4uZS5hIjogInFxNGtwM3czeWhodnk0Z200amdlemE0dnVzOHZweGdyd2M5MG44cmh4ZSIKICAgIH0sCiAgICAibGltaXQiOiAxMAogIH0KfQ==)

```
{
  "v": 2,
  "q": {
    "find": {
      "in.e.a": "qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe"
    },
    "limit": 10
  }
}
```

## 6. Query by Multiple Addresses


Find all transactions sent from the address `qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe` and received by `qpne29ue8chsv9pxv653zxdhjn45umm4esyds75nx6`

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAicSI6IHsKICAgICJmaW5kIjogewogICAgICAiaW4uZS5hIjogInFxNGtwM3czeWhodnk0Z200amdlemE0dnVzOHZweGdyd2M5MG44cmh4ZSIsCiAgICAgICJvdXQuZS5hIjogInFwbmUyOXVlOGNoc3Y5cHh2NjUzenhkaGpuNDV1bW00ZXN5ZHM3NW54NiIKICAgIH0sCiAgICAibGltaXQiOiAxMAogIH0KfQ==)


```
{
  "v": 2,
  "q": {
    "find": {
      "in.e.a": "qq4kp3w3yhhvy4gm4jgeza4vus8vpxgrwc90n8rhxe",
      "out.e.a": "qpne29ue8chsv9pxv653zxdhjn45umm4esyds75nx6"
    },
    "limit": 10
  }
}
```

## 7. Find OP_RETURN Transactions

Find all transactions that contain an **OP_RETURN** output (**output.b0** `{ op: 106 }`) 

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAicSI6IHsKICAgICJmaW5kIjogewogICAgICAib3V0LmIwIjogewogICAgICAgICJvcCI6IDEwNgogICAgICB9CiAgICB9LAogICAgImxpbWl0IjogMTAKICB9Cn0=)

```
{
  "v": 2,
  "q": {
    "find": {
      "out.b0": {
        "op": 106
      }
    },
    "limit": 10
  }
}
```

## 8. Find Multisig Transactions

Find all transactions that contain Multisig transactions (**output.b5** `{ op: 174 }`) 

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAicSI6IHsKICAgICJmaW5kIjogewogICAgICAib3V0LmI1IjogewogICAgICAgICJvcCI6IDE3NAogICAgICB9CiAgICB9LAogICAgImxpbWl0IjogMTAKICB9Cn0=)

```
{
  "v": 2,
  "q": {
    "find": {
      "out.b5": {
        "op": 174
      }
    },
    "limit": 10
  }
}
```

## 9. Filter down to the actual matched script

By default all queries return transaction objects since a "row" in BitDB is a transaction. However we can take advantage of the flexible MongoDB queries to drill down into the matched subdocuments.

For example, if we were looking for a specific match in an output, we can ask the query to return the actual matched output instead of the full transaction, using MongoDB's project and $ operator:

[Try Query](https://bitdb.network/explorer/ewogICJ2IjogMiwKICAiZSI6IHsgIm91dC5iMSI6ICJoZXgiIH0sCiAgInEiOiB7CiAgICAiZmluZCI6IHsgIm91dC5iMSI6ICI2ZDAyIiB9LAogICAgImxpbWl0IjogMTAsCiAgICAicHJvamVjdCI6IHsKICAgICAgIm91dC4kIjogMQogICAgfQogIH0KfQ==)

```
{
  "v": 2,
  "e": { "out.b1": "hex" },
  "q": {
    "find": { "out.b1": "6d02" },
    "limit": 10,
    "project": {
      "out.$": 1
    }
  }
}
```
