---
namespace-identifier: iota-caip19
title: IOTA - Asset
author: Enrico Marconi (@UMR1352)
status: Draft
type: Standard
created: 2025-07-30
requires: ["CAIP-2", "CAIP-10", "CAIP-19"]
---

# CAIP-19

*For context, see the [CAIP-19][] specification.*

## Rationale

IOTA is an object-based DLT, where objects are the foundamental unit of storage instead of accounts.
All objects have some common metadata, such as the address of their owner and a unique address that
allows to address each and everyone of them. Besides common metadata, objects have custom fields that
are determined by their type. For instance, `Coin` objects have a unsigned integer field `balance`.

Object types are declared inside of IOTA smart contracts - i.e. _Move packages_ - which are just a special
kind of immutable objects containing a collection of IOTA Move bytecode modules. Each package can define
multiple types of objects, by describing their structure, as well as the functionalities each object type
offers.

Lastly, it is possible to identify one last type of address in IOTA, key-pair based accounts. Accounts
are identified by a 32 bytes address that is computed by hashing their public key's bytes with the 
`blake2b` algorithm.

IOTA objects, packages, and key-pair derived addresses all shared the same address-space as defined in 
[IOTA CAIP-10].

## Syntax

Although IOTA objects, packages, and account addresses all shared the same address-space we define three
different [CAIP-19]'s namespaces, one for each of them, in order to easily discern what is referenced by
a given asset ID.

### Objects

IOTA Move objects are all part of the `object` asset ID's namespace, indipendently of their actual Move
type. That is, "system" objects (such as `Coin`s) and user-defined objects, both fall in the `object`
namespace.

Therefore, IOTA Move objects can be encoded as the asset ID `object:<object address>`, or using regular
expressions: `object:0x[0-9a-f]{64}`.

### Move Packages

Although Move packages are just a special type of object, given their different functionalities, they 
have their own `package` namespace.
The assed ID of a Move package looks like `package:<package address>`, or as a regular expression:
`package:0x[0-9a-f]{64}`.

### Accounts

Key-pair based accounts have their own `account` namespace and their assed ID looks like `account:<account address>`
or `account:0x[0-9a-f]{64}` in regular expression.

## Examples

```
# An account on IOTA mainnet
iota:mainnet/account:0x7b4a34f6a011794f0ecbe5e5beb96102d3eef6122eb929b9f50a8d757bfbdd67
# Note how this is the same as CAIP-10 account ID iota:mainnet:0x7b4a34f6a011794f0ecbe5e5beb96102d3eef6122eb929b9f50a8d757bfbdd67

# A Move package on IOTA testnet
iota:testnet/package:0x3403da7ec4cd2ff9bdf6f34c0b8df5a2bd62c798089feb0d2ebf1c2e953296dc

# An object on IOTA mainnet
iota:mainnet/object:0x67253dbb9a2bec41032108355524e85606ee90f6c96d6c6134cc231a60e4bf7a
```

## Resolution Mechanism

Once the correct endpoint for a given chain ID has been found (see [IOTA CAIP-2]), an asset ID can be resolved
through an RPC call.

### Object
Assets with the `object` namespace can be resolved through the `iota_getObject` RPC.

Example RPC call:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "iota_getObject",
  "params": [
    "0x53e4567ccafa5f36ce84c80aa8bc9be64e0d5ae796884274aef3005ae6733809",
    {
      "showType": true,
      "showOwner": true,
      "showPreviousTransaction": true,
      "showDisplay": false,
      "showContent": true,
      "showBcs": false,
      "showStorageRebate": true
    }
  ]
}
```
Example RPC response:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "data": {
      "objectId": "0x53e4567ccafa5f36ce84c80aa8bc9be64e0d5ae796884274aef3005ae6733809",
      "version": "1",
      "digest": "33K5ZXJ3RyubvYaHuEnQ1QXmmbhgtrFwp199dnEbL4n7",
      "type": "0x2::coin::Coin<0x2::iota::IOTA>",
      "owner": {
        "AddressOwner": "0xc8ec1d5b84dd6289e193b9f88de4a994358c9f856135236c3e75a925e1c77ac3"
      },
      "previousTransaction": "5PLgmQye6rraDYqpV3npV6H1cUXoJZgJh1dPCyRa3WCv",
      "storageRebate": "100",
      "content": {
        "dataType": "moveObject",
        "type": "0x2::coin::Coin<0x2::iota::IOTA>",
        "fields": {
          "balance": "100000000",
          "id": {
            "id": "0x53e4567ccafa5f36ce84c80aa8bc9be64e0d5ae796884274aef3005ae6733809"
          }
        }
      }
    }
  },
  "id": 1
}
```

### Move packages

Assets with the `package` namespace can be resolved with the `iota_getObject` RPC, though the response is not
as human-readable, as it contains Move bytecode.

Example RPC call:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "iota_getObject",
  "params": [
    "0x84cf5d12de2f9731a89bb519bc0c982a941b319a33abefdd5ed2054ad931de08",
    {
      "showType": false,
      "showOwner": true,
      "showPreviousTransaction": false,
      "showDisplay": false,
      "showContent": true,
      "showBcs": false,
      "showStorageRebate": false
    }
  ]
}
```
Example response:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "data": {
      "objectId": "0x53e4567ccafa5f36ce84c80aa8bc9be64e0d5ae796884274aef3005ae6733809",
      "version": "1",
      "digest": "98248Q5Debb3FjhwMQvVcFvuhMA8ZpPM13qe9RBWtxAo",
      "owner": "Immutable",
      "content": {
        "dataType": "package",
        "disassembled": { /* Move bytecode */ }
      }
    }
  },
  "id": 1
}
```

### Accounts

Assets with the `account` namespace cannot be resolved, as they are not Move objects,
but they support other RPC, such as `iotax_getBalance` or `iota_getOwnedObjects`.

Example RPC request:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "iotax_getOwnedObjects",
  "params": [
    "0x0cd4bb4d4f520fe9bbf0cf1cebe3f2549412826c3c9261bff9786c240123749f",
    {
      "filter": {
        "MatchAll": [
          {
            "StructType": "0x2::coin::Coin<0x2::iota::IOTA>"
          },
          {
            "AddressOwner": "0x0cd4bb4d4f520fe9bbf0cf1cebe3f2549412826c3c9261bff9786c240123749f"
          },
          {
            "Version": "13488"
          }
        ]
      },
      "options": {
        "showType": true,
        "showOwner": true,
        "showPreviousTransaction": true,
        "showDisplay": false,
        "showContent": false,
        "showBcs": false,
        "showStorageRebate": false
      }
    },
    "0x8a417a09c971859f8f2b8ec279438a25d8876ea3c60e345ac3861444136b4a1b",
    3
  ]
}
```

Example RPC response:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "data": [
      {
        "data": {
          "objectId": "0xd87765d1aadec2db8cc24c312542c8359b6b5e3bfeab524e5edaad3a204b4053",
          "version": "13488",
          "digest": "8qCvxDHh5LtDfF95Ci9G7vvQN2P6y4v55S9xoKBYp7FM",
          "type": "0x2::coin::Coin<0x2::iota::IOTA>",
          "owner": {
            "AddressOwner": "0x0cd4bb4d4f520fe9bbf0cf1cebe3f2549412826c3c9261bff9786c240123749f"
          },
          "previousTransaction": "kniF9zCBVYevxq3ZmtKxDDJk27N1qEgwkDtPiyeve4Y",
          "storageRebate": "100"
        }
      },
      {
        "data": {
          "objectId": "0x26ed170e0427f9416a614d23284116375c16bd317738fd2c7a885362e04923f5",
          "version": "13488",
          "digest": "5Ka3vDaDy9h5UYk3Maz3vssWHrhbcGXQgwg8fL2ygyTi",
          "type": "0x2::coin::Coin<0x2::iota::IOTA>",
          "owner": {
            "AddressOwner": "0x0cd4bb4d4f520fe9bbf0cf1cebe3f2549412826c3c9261bff9786c240123749f"
          },
          "previousTransaction": "FLSfkL1pVTxv724z5kfPbTq2KsWP1HEKBwZQ57uRZU11",
          "storageRebate": "100"
        }
      },
      {
        "data": {
          "objectId": "0x2aea16d6fb49b7d1ae51f33b01ed8e1ac66916858610c124bb6fd73bb13e141c",
          "version": "13488",
          "digest": "D3hfhfecVcmRcqNEkxkoFMzbuXvBqWj1JkCNU9zbnYMo",
          "type": "0x2::coin::Coin<0x2::iota::IOTA>",
          "owner": {
            "AddressOwner": "0x0cd4bb4d4f520fe9bbf0cf1cebe3f2549412826c3c9261bff9786c240123749f"
          },
          "previousTransaction": "GEoTGQuWicnPLM9Rg3vW1Q2y3kvnAgEkbyn8Z3RnAYai",
          "storageRebate": "100"
        }
      }
    ],
    "nextCursor": "0x2aea16d6fb49b7d1ae51f33b01ed8e1ac66916858610c124bb6fd73bb13e141c",
    "hasNextPage": true
  },
  "id": 1
}
```

## Reference

[CAIP-2]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[CAIP-10]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-10.md
[CAIP-19]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[IOTA CAIP-2]: ./caip10.md
[IOTA CAIP-10]: ./caip10.md
[IOTA Docs]: https://docs.iota.org
[IOTA RPC API]: https://docs.iota.org/iota-api-ref
[IOTA Object Model]: https://docs.iota.org/developer/iota-101/objects/object-model


## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

