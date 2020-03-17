# BSC Stratum Protocol

## Basic knowledge of BSC PoW algo

### header[144]
```
nNonce[4]
nVersion[4]
nTime[4] 
nBits[4]
hashPrevBlock[32]
hashMerkleRoot[32]
hashStateRoot[32] // bsc
hashUTXORoot[32] // bsc
```

### hash calculation
```
d[128] = header[0:128]
c[32] = header[112:144]

left_hash[64] = swap(blake2b_512(d[128]))
right_hash[32] = sha3(d[128] + c[0:8])
final_hash[32] = blake2b_256(left_hash[64] + c[32] + right_hash[32])
```

## Stratum Protocol

### client to server

#### mining.authorize("username", "password")

The result from an authorize request is usually true (successful), or false. The password may be omitted if the server does not require passwords.

request example:
```
{
    "id":"1",
    "jsonrpc":"2.0",
    "method":"mining.authorize",
    "params":["user","x"]
}
```

response example:
```
{
    "id":"1",
    "result":true,
    "error":null
}
```

```
{
    "id":"1",
    "result":null,
    "error":[code, reason, false]
}
```

#### mining.subscribe("user agent/version", "extranonce1")

The optional second parameter specifies a mining.notify subscription id the client wishes to resume working with (possibly due to a dropped connection). If provided, a server MAY (at its option) issue the connection the same extranonce1. Note that the extranonce1 may be the same (allowing a resumed connection) even if the subscription id is changed!

request example:
```
{
    "id":"2",
    "jsonrpc":"2.0",
    "method":"mining.subscribe"
    "params":["miner v1.2.3","97b5f8b6"]
}
```

response example:
```
{
    "id":"2",
    "result":[[["mining.set_difficulty", "subscription id 1"], ["mining.notify", "subscription id 2"]], "extranonce1", extranonce2_size],
    "error": null
}
```

The result contains three items:
* Subscriptions. - An array of 2-item tuples, each with a subscription type and id.
* ExtraNonce1. - Hex-encoded, per-connection unique string which will be used for creating generation transactions later.
* ExtraNonce2_size. - The number of bytes that the miner users for its ExtraNonce2 counter.

#### mining.suggest_difficulty(preferred share difficulty Number)

Used to indicate a preference for share difficulty to the pool. Servers are not required to honour this request, even if they support the stratum method.


#### mining.submit("username", "job id", "ExtraNonce2", "nTime", "nOnce")

request example:
```
{
    "id":"5",
    "jsonrpc":"2.0",
    "method":"mining.submit"
    "params":["user.001","27420ba0434a195c","d3ebfd42","5e42878d","97b5f8b6"]
}
```

response example:
```
{
    "id":"5",
    "result":true,
    "error": null
}
```

```
{
    "id":"5",
    "result": null,
    "error":[code, reason, false]
}
```


### server to client

#### mining.set_difficulty(difficulty)

The server can adjust the difficulty required for miner shares with the "mining.set_difficulty" method. The miner should begin enforcing the new difficulty on the next job received. Some pools may force a new job out when set_difficulty is sent, using clean_jobs to force the miner to begin using the new difficulty immediately.

example:
```
{
    "id":null,
    "method":"mining.set_difficulty",
    "params":[32]
}
```


#### mining.set_extranonce("extranonce1", extranonce2_size)

replace the initial subscription values beginning with the next mining.notify job.

#### mining.notify(...)
```
{
    "id":null,
    "method":"mining.notify",
    "params":[
        "job id",
        "hashPrevBlock",
        "coinb1", // Generation transaction (part 1). The miner inserts ExtraNonce1 and ExtraNonce2 after this section of the transaction data.
        "coinb2", // Generation transaction (part 2). The miner appends this after the first part of the transaction data and the two ExtraNonce values.
        "merkle branches", // The generation transaction is hashed against the merkle branches to build the final merkle root.
        "nVersion",
        "nBits",
        "nTime",
        "hashStateRoot",
        "hashUTXORoot",
        "Clean Jobs" // If true, miners should abort their current work and immediately use the new job. If false, they can still use the current job, but should move to the new one after exhausting the current nonce range.
    ]
}
```






