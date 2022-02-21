# WebAssembly for Arbitrary Code Execution

## Overview

Web Assembly, being a safety-first binary instruction format for a stack-based
virtual machine, makes for a good candidate for Sentinel code execution. For the
sake of developer experience, ideally, code to execute would be written using
the strict Typescript subset, Assemblyscript. The Graph already has type
bindings for everything a Sentinel would need for EVM interactions, so
developing a new Sentinel-executable module would be comparable to developing
a subgraph.

## WebAssembly and Assemblyscript

WebAssembly (WASM) was designed to execute binary encoded programs in the
browser at near-native speeds. The safety mechanisms built into WASM such as
restricted file system access, make it a useful language for writing code to be
executed in a virtual machine with strong safety guarantees to the host machine.

Assemblyscript (AS) was designed as a Typescript subset, meant to be a thin
layer on top of WASM, proving Typescript developers access to high performance
applications.

## The Graph's Assemblyscript Bindings

The graph has already built AS bindings for handling ethereum-related
interactions, event log handling, and more. Subgraphs are developed by the
developer by writing AS to map each event log to the Graph Node's storage.

The package is open source with a dual license of MIT and AGPLv3.

## The Sentinel

The Sentinel should be divided into two layers, the aggregation layer and the
execution layer.

### Aggregation Layer

This will be written in Rust, and will have minimal configurability, since the
aggregation layer is meant to aggregate _all_ relevant data related to
superfluid and super tokens. While this may be overkill for small use cases
needing only a fraction of the data, this also allows Sentinels to run multiple
execution modules on the same aggregation layer, reducing duplicate work.

### Execution Layer

The Sentinel should have two major layers, the aggregation layer, and the
execution layer. The aggregation layer functions exactly the same for all
Sentinels. Storing Superfluid related data and maintaining a synchronization
with the blockchain is the priority of the aggregation layer.

The execution layer consists of a set of AS bindings and a Yaml config to handle
an array of conditions. Standard conditions include, but may not be limited to
the following.

- BLOCK_TIMESTAMP_INTERVAL
- ADDRESS_INSOLVENT
- ADDRESS_CRITICAL
- ADDRESS_INSOLVENT_BATCH
- ADDRESS_CRITICAL_BATCH

With `predict: number` being an option to attempt to predict conditions when
possible.

### Predictions

Predictions rely on the aggregated data to execute functions/transactions before
a condition is met. In the case of account insolvency prediction, the Sentinel
needs to know all of the account's super token data. This includes every active
flow, in and out, and it needs to listen for transfers, flow updates, and IDA
interactions. This is important because an IDA distribution call from a
near-insolvent address can make the account insolvent within the next block.

There are cases where address-based predictions cannot be handled, for
example, when an account has a number of streams greater than which can be
batched in a single transaction, while also executing a transaction for their
entire real-time balance. Even front-running the transaction might not be
feasible, even if mempool support was added in the future.

### Liquidation Example

The following is an example of how the AS binging may be structured, as well as
the Yaml config to handle function execution.

```typescript
// ./src/index.as
import { Address, Bytes, ethereum } from "@graphprotocol/graph-ts"
import { encodeWithSignature } from "example-sentinel-package"

export function handleInsolvent(calldata: Bytes): CallResult {
	// decode addresses
	const decoded = ethereum.decode("(address,address,address)", calldata)
	const token = decoded[0]
	const sender = decoded[1]
	const receiver = decoded[2]

	// encode delete flow agreement call
	return encodeWithSignature(
		"callAgreement(address,bytes,bytes)",
		ethereum.Value.fromBytes(
			encodeWithSignature(
				"deleteFlow(address,address,address,bytes)",
				ethereum.Value.fromAddress(token),
				ethereum.Value.fromAddress(sender),
				ethereum.Value.fromAddress(receiver),
				ethereum.Value.fromBytes(Bytes.empty())
			)
		),
		ethereum.Value.fromBytes(Bytes.empty())
	)
}
```

```yaml
# ./config.yaml
version: 0.0.0
description: Liquidation Config For Sentinel
assemblyscript_mappings: ./src/index.as
handlers:
  - name: Insolvency Handler
    condition: ADDRESS_INSOLVENT
    predict: 0
    delay: 0
    handler: handleInsolvent
    params:
      - address: token
      - address: sender
      - address: receiver
    target_address: 0x3E14dC1b13c488a8d5D310918780c983bD5982E7
    state_mutability: non_payable
```

## TODO

There may need to be some more arbitrary logic handling than this, for example,
checking if the user is the PIC to determine prediction and delay on the
liquidation example.

The current `graph-ts` encoding/decoding API makes for messy AS bindings, there
may be need for some helpers like the `encodeWithSignature` as listed above.

Looking at a few subgraph implementations, there is a lot of generated code to
help with developer experience as well. This may be ideal in the long term, but
is likely out of scope for an MVP.

