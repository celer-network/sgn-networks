## SGN Testnet 3002

sgn-testnet-3002 runs SGN v0.2.7.

### Genesis hash

```shellscript
sha256sum genesis.json
3d945ac09aeaf5b51effc21bb0297cfcb7841d762b1ab0cb83bf514d5bef7b57  genesis.json
```

### Web portal

- [sgntest.celer.network](https://sgntest.celer.network/)

### Ropsten contracts

- [DPoS contract](https://ropsten.etherscan.io/address/0xBC31fA28E01CAB33a9bcD7b30A7C6291479e47c4)
- [SGN contract](https://ropsten.etherscan.io/address/0x7C079d6d889A27ac597F1520168F663523c59FB4)

### Selected parameters

- `Validator whitelist: enabled.` Need to get whitelisted in order to claim validator on mainchain.
- `Slash execution: disabled.` Slash events are logged but not executed.
- `Reward distribution: enabled.`
- `Minimal validator staking pool size: 100,000 CELR.`
- `Slash timeout: 100.` The mainchain block time for tokens to be locked in unbonding status.
- `Maximum validator number: 21.`

Full mainchain and sidechain states and parameters can be queried through the [mainchain contract](https://ropsten.etherscan.io/address/0xBC31fA28E01CAB33a9bcD7b30A7C6291479e47c4#readContract) and the sidechain validator CLI.
