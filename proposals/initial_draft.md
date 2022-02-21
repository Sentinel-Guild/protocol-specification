# The Sentinel

## Abstract

There is an increasing need for off-chain infrastructure in the Supefluid
ecosystem as more projects, protocols, and individuals need to interact with
contracts in complex, scheduled, or automated ways. This includes, but is not
limited to, insolvent stream liquidation, pre-insolvency courtesy stream
closures, scheduled transactions, and meta transactions.

Each project implements their own variant of a Sentinel, and each faces similar
problems of block reorganizations, aggregating data reliably, maintaining a
synchronization with the most recently mined block, and executing arbitrary
actions.

The objective is to create a modular, configurable Sentinel that not only
aggregates Superfluid-specific data, but also can execute any number of
arbitrary actions on smart contracts.

## Superfluid and Super Tokens

The Superfluid protocol enables novel and unique functionality for thieir custom
designed super tokens. This functionality comes from the ERC20 standard, ERC777
standard, and their own innovation, agreements. These agreements encode and
store arbitrary data on super tokens, which then account for a real-time balance
of a given user. Instead of an account balance being solely a single uint256
being read from a storage slot, there's also the per-second balance change from
the Constant Flow Agreement, and the active subscriptions in the Instant
Distribution Agreement.

## Functions to Aggregate

Since the functions called to create agreements are complex, this will be a list
of contracts and their respective functions that will need to be taken into
account. Format is `ContractName.functionToCall`.

- Superfluid.callAgreement
	- ConstantFlowAgreementV1.createFlow
	- ConstantFlowAgreementV1.updateFlow
	- ConstantFlowAgreementV1.deleteFlow
	- InstantDistributionAgreementV1.createIndex
	- InstantDistributionAgreementV1.updateIndex
	- InstantDistributionAgreementV1.distribute 
	- InstantDistributionAgreementV1.approveSubscription
	- InstantDistributionAgreementV1.updateSubscription
	- InstantDistributionAgreementV1.deleteSubscription
	- InstantDistributionAgreementV1.claim
- SuperToken.transfer
- SuperToken.transferFrom
- SuperToken.send
- SuperToken.burn
- SuperToken.operatorSend
- SuperToken.operatorBurn
- SuperToken.selfMint
- SuperToken.selfBurn
- SuperToken.upgrade
- SuperToken.upgradeTo 
- SuperToken.operationTransfer
- SuperToken.operationUpgrade
- SuperToken.operationDowngrade

Each of these functions contribute to balance changes directly, so each will
need to be considered.

## Modularity

Since a Sentinel might be used in a variety of ways, there should be an extreme
degree of configurability to handle as many use cases as possible. A standard
JSON schema should be implemented, where a Sentinel needs no code-changes to
handle the use case.

```json
{
	"name": "Superfluid Liquidation",
	"version": "1",
	"networkId": "137",
	"executables": [
		{
			"target": "0x3E14dC1b13c488a8d5D310918780c983bD5982E7",
			"functionName": "callAgreement",
			"abi": [],
			"condition": {
				"target": "0xCAa7349CEA390F89641fe306D93591f87595dc1F",
				"functionName": "isAccountSolventNow",
				"requiredReturn": false,
				"prediction": {
					"shouldPredict": true,
					"secondsBefore": 0
				}
			},
			"estimatedGas": "3000000",
			"estimatedGasPrice": "30",
			"maxGas": "10000000",
		}
	]
}
```

This is an example spec, targeting superfluid liquidations on USDCx on Polygon.

Instead of looping an calling `isAccountSolventNow`, it should use the
aggregated data to predict insolvency based on the current account flow state.

One-off transfers cannot be predicted, however, and this is why the Sentinel has
to maintain its sync with the blockchain by polling continuously. If a new block
is found to include a trasnfer that makes an account insolvent based on the data
on hand, the Sentinel should also trigger the function call.

This will need further discussion to make this as agnostic as possible. Though
in the context of Superfluid, the Sentinel will likely need to focus on solvency
maintenance and transaction scheduling.

## Special Considerations For Further Discussion

- Block Reorganization
- RPC Provider Rate Limits
- Gas Price Fluctuation
- Meta Transaction Validation and Execution

## Final Notes

This is a work in progress, discussion is welcome, and proposals are encouraged!

