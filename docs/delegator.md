# Delegator Manual

**Note: This manual assumes familiarity with Ethereum transactions using MetaMask.**

All delegator operations can be done through the SGN web portal([mainnet](https://sgn.celer.network), [testnet](https://sgntest.celer.network)). If using testnet, switch to Ropsten on MetaMask.

You can see a list of validators and their status on the homepage. More information of a validator can be found once you click its address.

## Delegate stake

1. Approve CELR to the DPoS contract through the `Approve CELR` button at lower left corner. 
   
   The value should be equal or greater than the total CELR amount you plan to delegator to the validators. You will be able to see the value updated at the upper left allowance field after the transaction is confirmed.

2. Once clicked into a validator, you can see a `Actions` button in the upper right corner. Delegate through `Actions -> Delegate`.
   The minimal staking amount per operation is 1 CELR. 

## Withdraw stake

1. Input the amount of stake to withdraw through `Actions -> Initialize Withdraw` witin a validator page. The minimal withdrawal amount per operation is 1 CELR. 

2. Confirm your withdrawal through `Actions -> Confirm Withdraw` after the mainchain `slashTimeout` (currently 100 ETH block, approximately 30 minutes)

## Claim reward

1. In the reward page, click `Initialize Redeem`.

2. Wait for the `collecting signatures` timer to finish, then click `Redeem Reward`.