# Uncensored Frontend - Send Transaction

If user's wallet does not support [Uncensored SDK](../uncensored-sdk/overview.md), he can use the Uncensored Frontend to force include his transactions.

**NOTE: Only Optimism Chains are supported right now**

---

Here we will go over different ways to use the Uncensored Frontend.

## 1. Build your transaction from scratch

Just like Safe Wallet's Transaction Builder, you can build your transaction on the Uncensored Frontend, supplying it with the contract address you want to interact with and its ABI. The frontend will try to fetch the ABI from explorer, if it fails, you can manually supply it.

Next you can click the `Select Function` button to select the function you want to call from the dropdown, fill in the parameters, and click `Generate Data`.

If your transaction requires sending ETH along the way, remember to fill in the `value (ETH)` field.

Lastly, the tricky part is to estimate the gas limit of this transaction. Right now you have to fill in the gas limit manually. In the future, we plan to support automatic gas estimation.

Once all the fields are filled in, click `Force Include` to send out the transaction.

---

## 2. Copy from existed transaction

It could happen that you already sent a transaction but it never gets included or if you find it difficult to build the transaction from scratch. In either case, you can copy the fields from an existed transaction.

### 2.1 Copy from a stuck transaction

You should be able to view the stuck transaction's detail either directly on the wallet's transaction history page or on the explorer. Then you can copy the `to`, `value`, `data`, and `gasLimit` fields from the transaction details.

If your wallet does not show specific fields like `data` and the transaction also does not show up on the explorer, you might have to try the next approach.

### 2.2 Interact with DApp and copy from prompted transaction request

If you are interacting with a DApp, for example, interacting with Uniswap frontend to execute a swap. After you have chosen the pair and the amount and clicked `Swap`, your wallet will be prompted with transaction signing request. Then you can copy the `to`, `value`, `data`, and `gasLimit` fields from the transaction viewing page.
