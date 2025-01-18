# Force Inclusion on Optimism

<!-- toc -->

---

## Caveats

- The user is assumed to be an EOA. If the user is using a contract account, the process will be a little different and is not covered in this document.

---

## Flow

Steps to successfully complete a force inclusion:

0. **Craft a normal (unsigned) L2 transaction**: This is the transaction that you want to force include but do not sign it.
1. **Request**: Make the force inclusion request from L1. In Optimism's case, it's the `OptimismPortal` contract on L1.
2. **Transaction ID**: Get the identifier to the force included L2 transaction in the request. In Optimism's case, it's the transaction hash of the force included L2 transaction. Unlike a normal L2 transaction, it will incorporate information like L1 blockhash and log index. See below for detail.
3. **Track Progress**: Tracking the progess of the force included L2 transaction in the request. In Optimism's case, query for the transaction's receipt using `getTransactionReceipt` RPC call.
4. **Complete Inclusion**: In Optimism's case, no manual intervention is needed to complete the inclusion. It's up to sequencer to decide when to include the transaction. Sequencer has `SEQEUNCER_WINDOW` amount of time before the transaction is force-included.

---

### Contracts Used
- [`OptimismPortal` contract on L1](https://etherscan.io/address/0xbeb5fc579115071764c7423a4f12edde41f106ed)

### 0: Craft a normal (unsigned) L2 transaction

The users crafts a normal L2 transaction which he wish to execute. Or if he already signed and sent the L2 transaction but the transaction never gets included, then he can just use the transaction data to make the Force Inclusion request.

### 1: Request

The user calls the `depositTransaction` function on `OptimismPortal` contract on L1:
```solidity
function depositTransaction(
    address _to,
    uint256 _value,
    uint64 _gasLimit,
    bool _isCreation,
    bytes memory _data
)
```

**Important notes**:

- The user **MUST** be an EOA.
- The user **MUST** send a L1 transaction to execute this function using his OEA. He can not use a meta-transaction and have a relayer send the L1 transaction on his behalf.

#### Parameters

To better explain the parameters, we will use an example: suppose Bob wants to force include his transaction which executes an ETH->USDC swap on Uniswap on Optimism.

- `_to`: the destination address on L2, i.e., "who you want to call on L2?"
    - in the example, this is the address of the Uniswap ETH-USDC Pool on Optimism
- `_value`: the amount of ETH to **deposit to L2**, i.e., "how much ETH you want to deposit to your address along with the request?"
    - in the example, `_value` will be `0` as Bob does not want to deposit ETH to his address from L1 to L2. He just wants to swap ETH to USDC on L2.
- `_gasLimit`: the gas limit of the force-included L2 transaction, i.e., "how much gas you need to execute your transaction on L2"
    - in the example, this is the gas limit to executes the swap. Bob should simulate the swap and record the gas limit after the simulation.
- `_isCreation`: indicates if this is a transaction which deploys a new contract on L2
    - in the example, `_isCreation` will be `0` as Bob does not want to deploy a new contract on L2.
- `_data`: the data to call the destination address on L2, i.e., "what do you want to do on L2?"
    - in the example, this is the calldata encoding the function and the parameters for the swap

**Important notes**:

- There's a minimum `_gasLimit` value depending on the size of `_data`.
    - `if (_gasLimit < minimumGasLimit(uint64(_data.length))) revert SmallGasLimit();`
    - [minimumGasLimit](https://github.com/ethereum-optimism/optimism/blob/4797ddb70e05d4952685bad53e608cb5606284e6/packages/contracts-bedrock/src/L1/OptimismPortal.sol#L197)
- Size of `_data` can not exceed 120k bytes
    - `if (_data.length > 120_000) revert LargeCalldata();`
- The user will be charged on L1, not L2, for the amount of gas to spend on L2, i.e., the `_gasLimit` value
    - [charge for `_gasLimit` in `depostiTransaction`](https://github.com/ethereum-optimism/optimism/blob/4797ddb70e05d4952685bad53e608cb5606284e6/packages/contracts-bedrock/src/L1/OptimismPortal.sol#L499)
    - [metered](https://github.com/ethereum-optimism/optimism/blob/4797ddb70e05d4952685bad53e608cb5606284e6/packages/contracts-bedrock/src/L1/ResourceMetering.sol#L63): the gas metering modifier

### 2. Transaction ID

Once the request is made successfully, a `TransactionDeposited` event will be emitted. A L2 transaction ([Deposited Transaction](https://specs.optimism.io/protocol/deposits.html#the-deposited-transaction-type)) will be derived from this event and waits to be included by sequencer. You can use Viem to [extract `TransactionDeposited` logs](https://viem.sh/op-stack/utilities/extractTransactionDepositedLogs)

A force inclusion transaction is uniquely identified by
- **Source Hash**: Source hash is used to differentiate two deposits with same parameters. A source hash is comprised of
    - The L1 Blockhash during which the `TransactionDeposited` event was emitted
    - The index of the emitted `TransactionDeposited` log
- and other parameters specified in the `depositTransaction` function

#### Computing Source Hash 

- [Spec](https://specs.optimism.io/protocol/deposits.html#source-hash-computation)
- Implementations
    1. [Viem](https://viem.sh/op-stack/utilities/getSourceHash)
    2. [Optimism Official](https://github.com/ethereum-optimism/optimism/blob/53080c9ff58d2da0c056eba9e2708bde92c61415/op-node/rollup/derive/deposit_source.go#L22-L31)

#### Computing L2 Transaction Hash

This is the Transaction ID used to track the progess of the force inclusion transaction.

- [Spec](https://specs.optimism.io/protocol/deposits.html#the-deposited-transaction-type)
    - It will be a new transaction type defined in Optimism, with slightly different transaction fields.
    - Just like a normal Optimism transaction, you fill out the fields in the transaction, rlp encode it and then `keccak` hash it to get the transaction hash.
- Implementations
    1. [Viem](https://viem.sh/op-stack/utilities/getL2TransactionHash)
    2. [Optimism Official](https://github.com/ethereum-optimism/optimism/blob/53080c9ff58d2da0c056eba9e2708bde92c61415/op-node/rollup/derive/deposit_log.go#L35-L97)

---

### 3. Track Progess

With the computed L2 transaction hash, you can query APIs/RPCs to chekc its status or go to [explorer](https://optimistic.etherscan.io/txsEnqueue) and wait for the transaction to show up. Or if you are using libraries like Viem, you can use `waitForTransactionReceipt`. But note that it could take up to 12 hours for the transaction to be included.

---

### 4. Complete Inclusion

In Optimism's case, no manual intervention is needed to complete the inclusion. It's up to sequencer to decide when to include the transaction, however, sequencer has `SEQEUNCER_WINDOW` amount of time before the transaction is force-included.

Currently `SEQEUNCER_WINDOW` on Ethereum is set to 12 hours ([3600](https://github.com/ethereum-optimism/optimism/blob/53080c9ff58d2da0c056eba9e2708bde92c61415/packages/contracts-bedrock/deploy-config/mainnet.json#L10) L1 blocks).

---

## Gas

### L2 Gas Limit

User should simulate his L2 transaction, record the gas limit after the simulation and use that value as the gas limit for his force inclusion transaction. Perhaps adding a buffer because the force inclusion transaction can take up to 12 hours before inclusion and execution.

### L2 Gas Price

Since gas fee is paid on L1, users don't have to set L2 gas price for their force inclusion transaction.

---

## Transaction Replacement

Force inclusion transaction is not replaceable.

---

## Reasons Force Inclusion transaction could fail

### 1. Not enough ETH to transfer

User should make sure he has enough ETH on Optimism if he wants to transfer ETH in his force inclusion transaction. Alternatively he can deposit the ETH from L1 in his force inclusion transaction.

### 2. Not enough L2 gas limit

User should make sure he sets high enough gas limit so that his force inclusion transaction does not run into "Out of Gas" error.

### 3. L2 execution failed

If force inclusion transaction is not included and executed in time, the transaction might fail. For example, Bob's force inclusion transaction which swaps ETH to USDC on Uniswap Pool might fail because the price has moved quite a bit since he makes the request on L1.

---

## Side effects of Force Inclusion transaction

### EOA's nonce on L2 will be incremented

Nonce of user's address on L2 will be incremented whenever his force inclusion transaction is included and executed, no matter the transaction fail or not. This could result in wallet crafting a transaction with wrong nonce if it was not aware of force inclusion transactions.
