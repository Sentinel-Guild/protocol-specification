# Event Driven Sentinel

## Overview

The Event Driven Sentinel approach to the adaptable sentinel combines the
existing Sentinel architecture in Javascript with a configurable system for
handling events with arbitrary function execution.

## Philosophy

The event driven, Javascript native Sentinel design allows for a wider range of
contributions to the codebase, and allows Javascript developers to write their
own event handlers. Ideally, an event could trigger multiple events with
different priority levels.

## Design

The design is similar to that of an Express.js app, but for a general purpose
Sentinel.

```js
import Sentinel, { SentinelEvent } from '@sentinel-guild/sentinel'
import { liquidateAccount } from './customHandlers'

const config = {
	privateKey: process.env.PRIVATE_KEY,
	rpcUrl: process.env.RPC_URL,
	maxGas: process.env.MAX_GAS
}

const sentinel = new Sentinel(config)

sentinel.on(SentinelEvent.ADDRESS_INSOLVENT, async (account, superToken)  => {
	const receipt = await liquidateAccount(account, superToken)
	console.log(receipt)
})
```

The events to handle would include, but not necessarily be limited to the
following.

```ts
enum SentinelEvent {
	ADDRESS_INSOLVENT,
	ADDRESS_NEAR_INSOLVENT,
	ADDRESS_CRITICAL,
	ADDRESS_NEAR_CRITICAL,
	TIME_INTERVAL
}
```

This design could allow for multiple use cases to be run in parallel, that is,
if certain "jobs" on different account/token pairs are more profitable than
others, conditions could be set to execute one or the other. This is designed to
conform to the existing Superfluid Sentinel needs first, as it is the original
use case, and Sentinels will likely run liquidation jobs where other incentives
are not in play.

