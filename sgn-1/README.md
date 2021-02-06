## SGN 1 (Mainnet)

sgn-1 runs SGN v0.2.6.

### Genesis hash

```shellscript
sha256sum genesis.json
13b4bc97668a12a5e6053408a941fa026a47b99cb576ab81df29ede021c22eda  genesis.json
```

### Web portal

- [sgn.celer.network](https://sgn.celer.network/)

### Ethereum contracts

- [DPoS contract](https://etherscan.io/address/0x5216db4d4cb22d1ba38866867c38d8e862974e82)
- [SGN contract](https://etherscan.io/address/0xfe413cf641478c0ac9fe4b6dd93776e0342621d6)

### Selected parameters

- `Validator whitelist: enabled.` Need to get whitelisted in order to claim validator on mainchain.
- `Slash execution: disabled.` Slash events are logged but not executed.
- `Reward distribution: disabled.`
- `Minimal validator staking pool size: 100,000 CELR.`
- `Slash timeout: 43200.` The mainchain locking block time for funds to be potentially slashed.
- `Maximum validator number: 11.`

- Reward per sidechain block (~5sec): 23782343987823439878 CELR (150M CELR / year)

Full mainchain and sidechain states and parameters can be queried through the [mainchain contract](https://etherscan.io/address/0x5216db4d4cb22d1ba38866867c38d8e862974e82#readContract) and the sidechain validator CLI.
