# Uncensored

Welcome to the book of Uncensored. This guide provides comprehensive information about the Force Inclusion Mechanism, including what it does, how it works and how can you use it.

## What is Force Inclusion?

Force Inclusion is a mechanism that allows a L2 user to force transactions into the L2's transaction history even when the L2's sequencer is down or is actively censoring the user. Without Force Inclusion, users of the L2 won't be able to execute any transactions or even quit the L2 without sequencer's consent.

## Documentation Structure

- If you are interested in learning more about Force Inclusion mechanism and how it is implemented on different L2s, check out the [Research](research/overview.md) section.
- If you are a developer and devoted to protect your (L2) users from censorship, check out the [Uncensored SDK](sdk/overview.md) section.
- If you are a user and want to try out Force Inclusion, check out the [Uncensored Frontend](frontend/overview.md) section. Though you should ask your wallet to support Force Inclusion via the [Uncensored SDK](sdk/overview.md) which provides user experience far better than the frontend.

## Force Inclusion Wallet Demo

We make some modifications to the great Rabby Wallet and prepare two videos to show you how you will be using Force Inclusion if your wallet supports it.

You can check out the Rabby Wallet modifiication we made to support the force inclusion feature [here](https://github.com/NIC619/Rabby/pull/1).

### 1. Uncensored Mode

If Uncensored Mode is activated, user will be able to directly force include transactions without sending them to the L2 sequencer.

The video first shows user sending a normal transaction to the L2 (Optimism Sepolia) sequencer. Next, when Uncensored Mode is activated, user will directly send force inclusion transaction to L1 (Sepolia) and eventually the transaction will be (force) included in a L2 (Optimism Sepolia) block.

- 00:00 - 00:40: We begin with a normal deposit WETH transaction on Optimism Sepolia.
- 00:45 - 00:50: Activate Uncensored Mode.
- 00:50 - 01:06: Make another deposit WETH transaction but this time it is a force inclusion transaction so we will make the request on Sepolia (L1). Notice the Chain ID, address and data are all different from the previous transaction.
- 01:06 - 01:20: Waiting for the transaction to be included in a L1 block.
- 01:20 - 01:39: Display the transaction on L1 block explorer. Since it can take some time for the transaction to be included on L2, the video leaves out the part of displaying the transaction on L2 block explorer but you can check out the transaction [here](https://sepolia-optimistic.etherscan.io/tx/0x6d47d62577d5a681de8879417a252884a79ec6d404c90dfbf06bef3556541423).

<video width="100%" controls>
    <source src="./assets/videos/Uncensored-Mode.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>

### 2. Force Include Stuck Transactions

It can happen that you send your transaction to the L2 sequencer as usual but then the sequencer goes down or is secretly censoring you. You will find your transaction stuck indefinitely. In this case, you can hit the Force Inclusion button just like you hit the "Speed Up" or "Cancel (transaction)" button, it will prompt you to sign and send a force inclusion transaction to L1 and eventually the transaction will be (force) included in a L2 block.

In the video, we intentionally send a transaction with extreme low gas price so it will not be included by the L2 sequencer, to mimic the situation where the sequencer is down or is secretly censoring. Then we hit the Force Inclusion button and force include our transaction on L1.


- 00:00 - 00:36: We begin with a normal withdraw WETH transaction on Optimism Sepolia but we set the gas price to a very low value so it will not be included by the L2 sequencer.
- 00:36 - 00:43: View the pending transaction in the wallet.
- 00:43 - 00:52: View the Force Inclusion button along with Speed Up button.
- 00:52 - 01:05: Click the Force Inclusion button and view the transaction details.
- 01:05 - 01:29: Waiting for the transaction to be included in a L1 block and display it on L1 block explorer. The video also leaves out the part of displaying the transaction on L2 block explorer but you can check out the transaction [here](https://sepolia-optimistic.etherscan.io/tx/0x5d35eb29eeeb5cc4d78313b75df8658b8f7b369abfdf43dc2b2efd28ecdf0a4c).

<video width="100%" controls>
    <source src="./assets/videos/Force-Include-Stuck-Transactions.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>
