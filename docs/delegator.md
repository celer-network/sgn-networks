# Delegator Manual

**Note: This manual assumes familiarity with Ethereum transactions using MetaMask.**

All delegator operations can be done through the [SGN web portal](https://sgn-v1.celer.network).

You can see a list of validators and their status on the homepage. More validator information can be found once you click its address.

## Delegate stake

Delegation needs one Ethereum mainchain transaction, assuming enough ERC20 allowance has been approved to the DPoS contract.

Once clicked into a validator, you can delegate through **`Delegate`** at the upper-right corner of the page. The minimal staking amount per operation is 1 CELR. Information in the `Delegators` list on the page will be updated a few minutes after the tx is submitted, as the sidechain validators need some time to sync the mainchain updates to sidechain. You can also view your delegator info in the mainchain DPoS contract through the `Contract Reader` tab.

If you are delegating for the first time or do not have enough ERC20 allowance, an ERC20 approval transaction to the DPoS contract will be triggered before the delegation transaction. The default approval amount is unlimited, which can be adjusted in MetaMask before you confirm the transaction.

## Withdraw stake

Withdrawal from a bonded or unbonding validator requires two Ethereum mainchain transactions.

1. Within a validator page, input the amount to withdraw through **`Initialize Withdraw`**. The minimal withdrawal amount per operation is 1 CELR.

    After the transaction is mined on Ethereum, your delegator amount will be deducted in the delegators list. You can check your undelegating stakes and pending withdrawals in the DPoS contract from the `Contract Reader` tab.

2. Confirm your withdrawal through **`Confirm Withdraw`** after the mainchain `slashTimeout`. The timeout value can be queried in the `Contract Reader` tab, currently 0 ETH block on mainnet.

    Check your delegator info in the DPoS contract from the `Contract Reader` tab, and only submit `Confirm Withdraw` if there is undelegating stake and the current ETH block has passed the `intent block + slash timeout`.

## Claim reward

Claming reward requires one SGN sidechain transaction followed by one Ethereum mainchain transaction.

1. On the reward page, click **`Initialize Redeem`**.

2. Wait for the `collecting signatures` timer to finish, then click **`Redeem Reward`**.
