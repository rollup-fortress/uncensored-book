# Uncensored

Welcome to the book of Uncensored. This guide provides comprehensive information about the Force Inclusion Mechanism, including what it does, how it works and how can you use it.

## What is Force Inclusion?

Force Inclusion is a mechanism that allows a L2 user to force transactions into the L2's transaction history even when the L2's sequencer is down or is actively censoring the user. Without Force Inclusion, users of the L2 won't be able to execute any transactions or even quit the L2 without sequencer's consent.

## Documentation Structure

- If you are interested in learning more about Force Inclusion mechanism and how it is implemented on different L2s, check out the [Research](research/overview.md) section.
- If you are a developer and devoted to protect your (L2) users from censorship, check out the [Uncensored SDK](sdk/overview.md) section.
- If you are a user and want to try out Force Inclusion, check out the [Uncensored Frontend](frontend/overview.md) section. Though you should ask your wallet to support Force Inclusion via the [Uncensored SDK](sdk/overview.md) which provides user experience far better than the frontend.

## Demo

We have prepared a few demo videos to show you (1) how to use the Uncensored Frontend and (2) how you will be using Force Inclusion if your wallet supports it.

### 1. Uncensored Frontend

_Sorry. Work In Progress_

### 2. Wallet Supports Force Inclusion via Uncensored SDK

#### 2-1. Uncensored Mode

If Uncensored Mode is activated, user will be able to directly force include transactions without sending them to the L2 sequencer.

The video first shows user sending a normal transaction to the L2 (Optimism Sepolia) sequencer. Next, when Uncensored Mode is activated, user will directly send force inclusion transaction to L1 (Sepolia) and eventually the transaction will be (force) included in a L2 (Optimism Sepolia) block.

<video width="100%" controls>
    <source src="assets/videos/Uncensored-Mode.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

#### 2-2. Force Include Stuck Transactions

It can happen that you send your transaction to the L2 sequencer as usual but then the sequencer goes down or is secretly censoring you. You will find your transaction stuck indefinitely. In this case, you can hit the Force Inclusion button just like you hit the "speed up" or "cancel (transaction)" button, it will prompt you to sign and send a force inclusion transaction to L1 and eventually the transaction will be (force) included in a L2 block.

In the video, we intentionally send a transaction with extreme low gas price so it will not be included by the L2 sequencer, to mimic the situation where the sequencer is down or is secretly censoring. Then we hit the Force Inclusion button and force include our transaction on L1.

<video width="100%" controls>
    <source src="assets/videos/Force-Include-Stuck-Transactions.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>
