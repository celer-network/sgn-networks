# Delegator Manual

**Note: This manual assumes familiarity with Ethereum transactions using MetaMask.**

All delegator operations can be done through the SGN web portal([mainnet](sgn.celer.network), [testnet](sgntest.celer.network)). If using testnet, switch to Ropsten on MetaMask.

You can see a list of validators and their status on the homepage. More information of a validator can be found once you click its address.

All operations below except `Initialize Redeem` are Ethereum mainnet or Ropsten transactions.

## Delegate stake

1. Approve CELR to the DPoS contract through the `Approve CELR` button at lower left corner. The value should be equal or greater than the total CELR amount you plan to delegator to the validators. You will be able to see the valud updated at the upper left allowance field.

2. Once clicked into a validator, you can see a `Actions` button in the upper right corner. Delegate through `Actions -> Delegate`. 

## Withdraw stake

1. Input the amount of stake to withdraw through `Actions -> Initialize Withdraw` witin a validator page.

2. After the mainchain `slashTimeout` (currently 100 ETH block, approximately 30 minutes), confirm your withdrawal through `Actions -> Confirm Withdraw`.

## Claim reward

1. In the reward page, click `Initialize Redeem`.

2. Wait for the `collecting signatures` timer to finish, then click `Redeem Reward`.