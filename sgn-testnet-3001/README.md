## (DECOMMISSIONED) SGN Testnet 3001

sgn-testnet-3001 runs SGN v0.2.6.

### Genesis hash

```shellscript
sha256sum genesis.json
0bd5c4b6075ff4d2ca29d2102ce3407d5810b1bfc611e0f5a75365b283d6d402  genesis.json
```

### Ropsten contracts

- [DPoS contract](https://ropsten.etherscan.io/address/0x92d1f7266464b6a22f0e0749a310bf97b168e7e0)
- [SGN contract](https://ropsten.etherscan.io/address/0xbbf421bb74fda247fa692f7d3f26b22dfd8b08a3)

### Selected parameters

- `Validator whitelist: enabled.` Need to get whitelisted in order to claim validator on mainchain.
- `Slash execution: disabled.` Slash events are logged but not executed.
- `Reward distribution: enabled.`
- `Minimal validator staking pool size: 10,000 CELR.`
- `Slash timeout: 100.` The mainchain block time for tokens to be locked in unbonding status.
- `Maximum validator number: 11.`

Full mainchain and sidechain states and parameters can be queried through the [mainchain contract](https://ropsten.etherscan.io/address/0x92d1f7266464b6a22f0e0749a310bf97b168e7e0#readContract) and the sidechain validator CLI.
