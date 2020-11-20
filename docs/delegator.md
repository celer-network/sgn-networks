# Delegator Manual

**Note: This manual assumes familiarity with Ethereum transactions using MetaMask.**

All delegator operations can be done through the SGN web portal ([mainnet](https://sgn.celer.network), [testnet](https://sgntest.celer.network)). 

Testnet uses Ropsten as the mainchain. Ropsten mock CELR tokens can be obtained on the [Ropsten Uniswap](https://app.uniswap.org/#/swap?outputCurrency=0xb37f671dfc6c7c03462c76313ec1a35b0c0a76d5). Ropsten ETH can be obtained from places like the [MetaMask faucet](https://faucet.metamask.io).

You can see a list of validators and their status on the homepage. More validator information can be found once you click its address.

## Delegate stake

1. Approve CELR to the DPoS contract through the **`Approve CELR`** button at the lower-left corner. The value should be no less than the total amount you plan to delegate to the validators. The ERC20 allowance is updated at the upper-left corner.

2. Once clicked into a validator, you can see an **`Actions`** button at the upper-right corner. Delegate through **`Actions -> Delegate`**. The minimal staking amount per operation is 1 CELR.

## Withdraw stake

1. Within a validator page, input the amount to withdraw through **`Actions -> Initialize Withdraw`**. The minimal withdrawal amount per operation is 1 CELR.

2. Confirm your withdrawal through **`Actions -> Confirm Withdraw`** after the mainchain `slashTimeout`
(currently 100 ETH block, approximately 30 minutes)

## Claim reward

1. On the reward page, click **`Initialize Redeem`**.

2. Wait for the `collecting signatures` timer to finish, then click **`Redeem Reward`**.