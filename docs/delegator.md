# Delegator Manual

**Note: This manual assumes familiarity with Ethereum transactions using MetaMask.**

All delegator operations can be done through the SGN web portal ([mainnet](https://sgn.celer.network), [testnet](https://sgntest.celer.network)). 

If using testnet, switch to Ropsten on MetaMask. Ropsten mock CELR tokens for the testnet can be obtained on the [Ropsten Uniswap](https://app.uniswap.org/#/swap?outputCurrency=0xb37f671dfc6c7c03462c76313ec1a35b0c0a76d5). Ropsten ETH can be obtained from places like the [MetaMask faucet](https://faucet.metamask.io).

You can see a list of validators and their status on the homepage. More validator information can be found once you click its address.

## Delegate stake

Delegation needs one Ethereum mainchain transaction, assuming enough enough ERC20 allowance has been approved to the DPoS contract.

1. Approve CELR to the DPoS contract through the **`Approve CELR`** button at the lower-left corner. The value should be no less than the total amount you plan to delegate to the validators. The ERC20 allowance is updated at the upper-left corner. 
   
   This approval transaction needs to be done at least once as required by ERC20 standard. We recommend to approve a large enough amount to avoid unnecessary approval transactions before each delegation.

2. Once clicked into a validator, you can delegate through **`Actions -> Delegate`** at the upper-right corner of the page. The minimal staking amount per operation is 1 CELR. Information in the `Delegators` list on the page will be updated a few minutes after the tx is submitted, as the sidechain validators need some time to sync the mainchain updates to sidechain.

## Withdraw stake

Withdrawal from a bonded or unbonding validator requires two Ethereum mainchain transactions.

1. Within a validator page, input the amount to withdraw through **`Actions -> Initialize Withdraw`**. The minimal withdrawal amount per operation is 1 CELR.

2. Confirm your withdrawal through **`Actions -> Confirm Withdraw`** after the mainchain `slashTimeout`
(currently 100 ETH block, approximately 30 minutes)

## Claim reward

Claming reward requires one SGN sidechain transaction followed by one Ethereum mainchain transaction.

1. On the reward page, click **`Initialize Redeem`**.

2. Wait for the `collecting signatures` timer to finish, then click **`Redeem Reward`**.