# Force Inclusion on Arbitrum

Table of Contents:

- [Force Inclusion on Arbitrum](#force-inclusion-on-arbitrum)
  - [Caveats](#caveats)
  - [Flow](#flow)
    - [Contracts Used](#contracts-used)
    - [0: Sign Normal L2 Transaction](#0-sign-normal-l2-transaction)
    - [1: Request](#1-request)
      - [Parameters](#parameters)
    - [2. Transaction ID](#2-transaction-id)
    - [3. Track Progess](#3-track-progess)
    - [4. Complete Inclusion](#4-complete-inclusion)
      - [Parameters](#parameters-1)
      - [Code Walkthrough](#code-walkthrough)
  - [Gas](#gas)
    - [L2 Gas Limit and L2 Gas Price](#l2-gas-limit-and-l2-gas-price)
  - [Transaction Replacement](#transaction-replacement)
  - [Reasons Force Inclusion transaction could fail](#reasons-force-inclusion-transaction-could-fail)
    - [1. Not enough ETH to transfer](#1-not-enough-eth-to-transfer)
    - [2. Not enough L2 gas limit](#2-not-enough-l2-gas-limit)
    - [3. Gas price too low](#3-gas-price-too-low)
    - [4. L2 execution failed](#4-l2-execution-failed)
  - [Side effects of Force Inclusion transaction](#side-effects-of-force-inclusion-transaction)
    - [EOA's nonce on L2 will be incremented](#eoas-nonce-on-l2-will-be-incremented)

---

## Caveats

- The user is assumed to be an EOA. If the user is using a contract account, the process will be a little different and is not covered in this document.

---

## Flow

Steps to successfully complete a force inclusion on Arbitrum:

0. **Sign Normal L2 Transaction**: Sign the normal L2 transaction as usual. This will be the force included transaction.
1. **Request**: Make the force inclusion request from L1. In Arbitrum's case, it's the `Inbox` contract on L1.
2. **Transaction ID**: Get the identifier to the force included L2 transaction in the request. In Arbitrum's case, it's the transaction hash of the normal L2 transaction signed in step 0.
3. **Track Progress**: Tracking the progess of the force included L2 transaction in the request. In Arbitrum's case, query for the transaction's receipt using `getTransactionReceipt` RPC call.
4. **Complete Inclusion**: In Arbitrum's case, sequencer has around 24 hours to include the force included L2 transaction. If sequencer fails to do so, anyone can complete the force inclusion via the `SequencerInbox` contract on L1.

---

### Contracts Used
- [`Inbox` contract on L1](https://etherscan.io/address/0x4Dbd4fc535Ac27206064B68FfCf827b0A60BAB3f)
- [`Bridge` contract on L1](https://etherscan.io/address/0x1066cecc8880948fe55e427e94f1ff221d626591)
- [`SequencerInbox` contract on L1](https://etherscan.io/address/0x1c479675ad559DC151F6Ec7ed3FbF8ceE79582B6)

### 0: Sign Normal L2 Transaction

The users signs a normal L2 transaction as he used to. Or if he already signed and sent his L2 transaction but the transaction never gets included, then he can just use the transaction to make the Force Inclusion request.

### 1: Request

The user calls the `sendL2Message` function on `Inbox` contract on L1:
```solidity
function sendL2Message(bytes calldata messageData)
```

**Important notes**:

- The user does not have to be the one sending the L1 transaction to call `sendL2Message` function. He can have a relayer carry his signed normal L2 transaction onchain.
- The function has two modifiers: `whenNotPaused` and [`onlyAllowed`](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/AbsInbox.sol#L81-L85)
    - Rollup owner can `pause` the contract, effectively disabling force inclusion.
    - Allow list is currently [not enabled](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/AbsInbox.sol#L52). If enabled, it will check if the `tx.origin` is on the allow list, effectively disabling force inclusion by anyone except the ones in the allow list. Only Rollup owner can [set allow list](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/AbsInbox.sol#L61).

#### Parameters

To better explain the parameters, we will use an example: suppose Bob wants to force include his transaction which executes an ETH->USDC swap on Uniswap on Arbitrum.

Bob will first sign the L2 transaction which will execute an ETH->USDC swap on Uniswap on Arbitrum. Next he will append `0x04` to the signed L2 transaction payload: `0x04 | Bob's signed transaction`. This will be the `messageData` param to the `sendL2Message` function. See the [`sendChildSignedTx` function](https://github.com/OffchainLabs/arbitrum-sdk/blob/5913599cb40d3b948b557c48c64ca02f0ad99046/src/lib/inbox/inbox.ts#L401C16-L415) in Arbitrum SDK as an example.

**Important notes**:

- The size of `messageData` can not exceed [`117964` bytes (~116 KB)](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/AbsInbox.sol#L100-L101)

### 2. Transaction ID

Once the request is made successfully, a `MessageDelivered` event will be emitted. The event will contain all data necessary to complete the force inclusion in step 4. You can use the `getForceIncludableEvent` function in Arbiturm SDK to [extract `MessageDelivered` logs](https://github.com/OffchainLabs/arbitrum-sdk/blob/5913599cb40d3b948b557c48c64ca02f0ad99046/src/lib/inbox/inbox.ts#L307)
. However, that function will exclude any force inclusion transactions that is not eligible yet, i.e., it has not passed the 24 hours **Force Inclusion Grace Period**. Hence if you wish to extract `MessageDelivered` logs before the grace period ends, you will have to copy and modify the function.

Since the user is force including his **signed normal L2 transaction**, naturally the identifier for the force included L2 transaction is the transaction hash of his signed normal L2 transaction. You can use ``ethers.utils.parseTransaction` to [get the L2 transaction hash](https://github.com/OffchainLabs/arbitrum-tutorials/blob/0f2a50e95095947ab770e58da437c188c56c5905/packages/delayedInbox-l2msg/scripts/normalTx.js#L80)

---

### 3. Track Progess

With the L2 transaction hash, you can query APIs/RPCs to chekc its status or query the [explorer](https://arbiscan.io). Or if you are using libraries like Viem, you can use [`waitForTransactionReceipt`](waitForTransactionReceipt). But note that it could take up to 24 hours for the transaction to be included.

---

### 4. Complete Inclusion

In Arbitrum's case, currently sequencer has around 24 hours grace period to include user's transaction. If the sequencer did not include it in 24 hours. Anyone can call the `forceInclusion` function on `SequencerInbox` contract on L1. The [grace period](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/SequencerInbox.sol#L252) on Ethereum is currently set to 5760 L1 blocks (24 hours).

#### Parameters

The emitted `MessageDelivered` event basically contains all data needed to call the `forceInclusion` function. The parameters will be used to [compute the message hash](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/SequencerInbox.sol#L293-L301) and check if it [matches the one stored in the transaction queue](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/SequencerInbox.sol#L311-L314). Message hash is stored in the transaction queue when user [called `sendL2Message`](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/AbsBridge.sol#L182-L195) to make the force inclusion request, i.e., step 1.

#### Code Walkthrough

Message hash computation in `sendL2Message` function:
```solidity=
bytes32 messageHash = Messages.messageHash(
    kind,
    sender,
    blockNumber,
    blockTimestamp,
    count,
    baseFeeL1,
    messageDataHash
);
```

The data used to compute message hash will be [emitted in the `MessageDelivered` event](https://github.com/OffchainLabs/nitro-contracts/blob/fbbcef09c95f69decabaced3da683f987902f3e2/src/bridge/AbsBridge.sol#L196-L205) in `sendL2Message` function:
```solidity=
emit MessageDelivered(
    count,
    prevAcc, // not used by `Messages.messageHash()`
    msg.sender, // not used by `Messages.messageHash()`
    kind,
    sender,
    messageDataHash,
    baseFeeL1,
    blockTimestamp
);
```

Message hash computation in `forceInclusion` function:
```solidity=
function forceInclusion(
        uint256 _totalDelayedMessagesRead,
        uint8 kind,
        uint64[2] calldata l1BlockAndTime,
        uint256 baseFeeL1,
        address sender,
        bytes32 messageDataHash
    ) external {
    ...
    bytes32 messageHash = Messages.messageHash(
        kind, // the `kind` in `MessageDelivered` event
        sender, // the `sender` in `MessageDelivered` event
        l1BlockAndTime[0], // not emitted by the `MessageDelivered` event but you will get it when filtering for event logs
        l1BlockAndTime[1], // the `blockTimestamp` in `MessageDelivered` event
        _totalDelayedMessagesRead - 1, // the `count` parameter in `MessageDelivered` event
        baseFeeL1, // the `baseFeeL1` in `MessageDelivered` event
        messageDataHash // the `messageDataHash` in `MessageDelivered` event
    );
    ...
}
```

---

## Gas

The gas cost of a force inclusion transaction is not paid on L1, like Optimism. Instead, it is paid on L2, when the force included transaction is finally included and executed.

### L2 Gas Limit and L2 Gas Price

User should simulate his L2 transaction, get the estimated gas limit and gas price for his transaction. Perhaps adding a buffer because the force inclusion transaction can take up to 24 hours before inclusion and execution.

---

## Transaction Replacement

Force inclusion transaction is not replaceable.

---

## Reasons Force Inclusion transaction could fail

### 1. Not enough ETH to transfer

User should make sure he has enough ETH on Arbitrum if he wants to transfer ETH in his force inclusion transaction.

### 2. Not enough L2 gas limit

User should make sure he sets high enough gas limit so that his force inclusion transaction does not run into "Out of Gas" error.

### 3. Gas price too low

User should make sure he sets high enough gas price so that his force inclusion transaction does not get rejected due to low gas price.

### 4. L2 execution failed

If force inclusion transaction is not included and executed in time, the transaction might fail. For example, Bob's force inclusion transaction which swaps ETH to USDC on Uniswap Pool might fail because the price has moved quite a bit since he makes the request on L1.

---

## Side effects of Force Inclusion transaction

### EOA's nonce on L2 will be incremented

Nonce of user's address on L2 will be incremented whenever his force inclusion transaction is included and executed, no matter the transaction fail or not. This could result in wallet crafting a transaction with wrong nonce if it was not aware of force inclusion transactions.
