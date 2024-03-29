# Validator Manual

**Note: This manual assumes familiarity with Unix command line and blockchain validator nodes.
Prior experience with Cosmos SDK and Ethereum transactions will be helpful.**

## Table of Contents
- [Prepare machine and install dependencies](#prepare-machine-and-install-dependencies)
- [Setup binary, config and accounts](#setup-binary-config-and-accounts)
- [Run validator with systemd](#run-validator-with-systemd)
- [Claim validator status](#claim-validator-status)
- [Setup gateway](#setup-gateway)
- [Withdraw stakes / Unbond validator](#withdraw-stakes--unbond-validator)
- [Withdraw rewards](#withdraw-rewards)
- [Other operations](#other-operations)

If you only want to run a standby unbonded node to sync the blocks and query sidechain information, stop after [run validator with systemd](#run-validator-with-systemd).

## Prepare machine and install dependencies

1. Prepare a Linux machine with a minimum of 16GB RAM, at least 2 cores and at
least 100GB of disk space. We assume Ubuntu in this manual but most Linux distros should work.

2. Install go (version 1.14+), gcc, make, git and libleveldb-dev.

3. Set \$GOBIN and add \$GOBIN to \$PATH. Edit `$HOME/.profile` and add:

```sh
export GOBIN=$HOME/go/bin; export GOPATH=$HOME/go; export PATH=$PATH:$GOBIN
```

to the end, then:

```sh
source $HOME/.profile
```

## Setup binary, config and accounts

1. From the `/home/ubuntu` directory, clone the `sgn` repository, then install the `sgnd`, `sgncli`
and `sgnops` binaries:

```sh
git clone https://github.com/celer-network/sgn
cd sgn
git checkout master # Use master for latest stable version
WITH_CLEVELDB=yes make install-all
cd ..
```

2. From the `/home/ubuntu` directory, clone the `sgn-networks` repository:

```sh
git clone https://github.com/celer-network/sgn-networks
cd sgn-networks
git checkout master
cd <chain-id>
```

3. Copy the config files and initialize the validator node:

```sh
mkdir -p $HOME/.sgncli/config
cp sgn.toml $HOME/.sgncli/config
sgnd init <validator-name> --chain-id <chain-id>
cp genesis.json config.toml $HOME/.sgnd/config
```

4. Initialize common Cosmos SDK CLI configs, which will be saved to `$HOME/.sgncli/config/config.toml`:

```sh
sgncli config trust-node true; sgncli config indent true; sgncli config output json
```

5. Fill out the `moniker` field in `$HOME/.sgnd/config/config.toml` with the name of the validator.
Fill out the `external_address` field with `<public-ip:26656>`.

6. Get the validator public key:

```sh
sgnd tendermint show-validator
```

Make a note of the output.

7. Add an SGN validator account key:

```sh
sgncli keys add <key-name> --keyring-backend=file
```

Repeat to create multiple transactors for the gateway. You can use `<validator_name>_operator` to
name your validator account and `<validator_name_transactor0>`, `<validator_name_transactor1>`, etc.
to name your transactor accounts. **Use the same passphrase as the validator account for all accounts**.

To get the list of accounts, run:

```sh
sgncli keys list --keyring-backend=file
```

8. Prepare an Ethereum gateway URL. You can use services like [Infura](https://infura.io/),
[Alchemy](https://alchemyapi.io/) or run your own node.

9. Prepare an Ethereum keystore JSON file for the validator, eg.:

```sh
geth account new --lightkdf --keystore <path-to-keystore-folder>
```

Save it as `$HOME/.sgncli/eth_ks.json`.

10. For testnet, obtain some Ropsten ETH from places like the MetaMask
[faucet](https://faucet.metamask.io). Buy Ropsten mock CELR tokens on the
[Ropsten Uniswap](https://app.uniswap.org/#/swap?outputCurrency=0x38e63080C53b788Ee8e126c198696D22Fbf483A8).
If there is not enough CELR in the Uniswap pool, join our [Discord](https://discord.gg/uGx4fjQ)
server and ping us.

For mainnet, currently we have not opened validator spots. Please stay tuned for more information.

11. Send ETH and CELR to the validator ETH address. Make sure it has enough CELR for self-staking
and a few ETH for gas.

12. Fill in the placeholders in `$HOME/.sgncli/config/sgn.toml`.
    | Field | Description |
    | ----- | ----------- |
    | ETHEREUM_GATEWAY_URL | The Ethereum gateway URL obtained from step 8 |
    | KEYSTORE_PATH | The path to the keystore file in step 9 |
    | KEYSTORE_PASSPHRASE | The passphrase to the keystore |
    | VALIDATOR_ACCOUNT_ADDRESS | The sgn-prefixed address obtained in step 7 |
    | TRANSACTOR_PASSPHRASE | The passphrase you typed in step 7 |
    | VALIDATOR_PUBKEY | The validator public key obtained in step 6 |
    | TRANSACTOR_ADDRESS | If planning to run gateway, fill in one or multiple transactor accounts obtained in step 7. Otherwise, leave it as an empty array |

## Run validator with systemd

We recommend using systemd to run your validator. Feel free to experiment with other setups on your
own.

1. Prepare the sgnd system service:

```sh
sudo mkdir -p /var/log/sgnd
sudo touch /var/log/sgnd/tendermint.log
sudo touch /var/log/sgnd/app.log
sudo touch /etc/systemd/system/sgnd.service
```

Add the following to `/etc/systemd/system/sgnd.service`:

```
[Unit]
Description=SGN daemon
After=network-online.target

[Service]
ExecStart=/home/ubuntu/go/bin/sgnd start \
  --home /home/ubuntu/.sgnd/ \
  --cli-home /home/ubuntu/.sgncli \
  --config /home/ubuntu/.sgncli/config/sgn.toml
StandardOutput=append:/var/log/sgnd/tendermint.log
StandardError=append:/var/log/sgnd/app.log
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

`tendermint.log` contains Tendermint logs (i.e. block height, peer info, etc.) while `sgnd.log`
contains SGN-specific logs.

2. (Optional) For log rotation, create a `/etc/logrotate.d/sgn` file and add the following:

```
/var/log/sgnd/*.log {
    compress
    copytruncate
    maxsize 50M
    rotate 5
}
```

3. Enable and start the sgnd service:

```sh
sudo systemctl enable sgnd.service
sudo systemctl start sgnd.service
```

4.  Monitor `tendermint.log` and wait for the node to sync:

```sh
tail -f /var/log/sgnd/tendermint.log
```

You can tell the node is synced when new blocks show up about every 5 seconds.

## Claim validator status

1. Initialize the validator candidate:

```sh
sgnops init-candidate --commission-rate <commission-rate> --min-self-stake <min-self-stake> --rate-lock-period <rate-lock-period>
```

Wait for the transaction to be confirmed. Refer to the [CLI doc](https://github.com/celer-network/sgn/blob/master/docs/sgnops/sgnops_init-candidate.md) and [design doc](https://www.celer.network/docs/celercore/sgn/architecture.html#become-a-validator) for more details. Note that `rateLockPeriod + currentBlockNum = rateLockEndTime`

2. Delegate CELR to the candidate:

```sh
sgnops delegate --candidate <candidate-eth-address> --amount <delegate-amount>
```

Wait for the transaction to be confirmed. Refer to the [CLI doc](https://github.com/celer-network/sgn/blob/master/docs/sgnops/sgnops_delegate.md) and [design doc](https://www.celer.network/docs/celercore/sgn/architecture.html#delegate-stake) for more details.

3. Note that it will take some time for the existing SGN validators to sync your new validator from
the mainchain. Afterwards, verify your validator status:

```sh
sgncli query validator candidate <candidate-eth-address>
```

You should be able to see that your candidate has a `delegatedStake` matching your delegated amount
denominated in wei.

4. Verify validator is in the SGN validator set:

```sh
sgncli query validator validator <validator-account-address>
```

`<validator-account-address>` is the sgn-prefixed address you created. You should see your validator
and its `identity` should equal to your `<candidate-eth-address>`. Make a note of the
`consensus_pubkey` - the address prefixed with `sgnvalconspub`.

If your validator doesn't appear in the query, check the candidate status on mainchain through
```sh
sgnops get-candidate-info --candidate <candidate-eth-address>
```
If `Status = 0` (unbonded), try claiming the status manually:

```sh
sgnops claim-validator
```

If `Status = 1` (bonded), try triggering the sidechain info sync manually:

```sh
sgnops sync sync-validator --candidate <candidate-eth-address>
```

Then wait for a while and retry the query again.

5. Verify validator is in the Tendermint validator set:

```sh
sgncli query tendermint-validator-set
```

You should see an entry with `pub_key` matching the `consensus_pubkey` obtained in step 4.

6. Go to the SGN [mainnet explorer](https://sgn-v1.celer.network/explorer) to view your validator.

7. Update validator public contact info:

```sh
sgncli tx validator edit-candidate-description --contact [email address] --website [entity website]
```

## Setup gateway

Optionally, you can start a gateway service alongside the validator to serve HTTP requests.

1. Prepare the SGN gateway service:

```sh
sudo mkdir -p /var/log/sgn-gateway
sudo touch /var/log/sgn-gateway/gateway.log
sudo touch /var/log/sgn-gateway/gateway_error.log
sudo touch /etc/systemd/system/sgn-gateway.service
```

Add the following to `/etc/systemd/system/sgn-gateway.service`:

```
[Unit]
Description=SGN gateway
After=network-online.target

[Service]
ExecStart=/home/ubuntu/go/bin/sgncli gateway \
  --laddr tcp://0.0.0.0:1317 \
  --home /home/ubuntu/.sgncli \
  --config /home/ubuntu/.sgncli/config/sgn.toml
StandardOutput=append:/var/log/sgn-gateway/gateway.log
StandardError=append:/var/log/sgn-gateway/gateway_error.log
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

2. (Optional) Add the log folder to `/etc/logrotate.d/sgn`:

```
/var/log/sgnd/*.log
/var/log/sgn-gateway/*.log {
...
```

3. Enable and start the service:

```sh
sudo systemctl enable sgn-gateway.service
sudo systemctl start sgn-gateway.service
```

## Withdraw stakes / Unbond validator
Please refer to the [design doc](https://www.celer.network/docs/celercore/sgn/architecture.html#withdraw-stake) for better understanding of the stake withdrawal process.

1. Initialize a withdrawal of your self-delegated stake:
```sh
sgnops withdraw intend --candidate <candidate-eth-address> --amount <delegate-amount>
```
If the self-delegated stake after the withdrawal is below your `min-self-stake` amout, the validator will transit to `unbonding` status.

2. After the slash timeout, confirm the withdrawal of your stake:
```sh
sgnops withdraw confirm --candidate <candidate-eth-address>
```
If your validator is in unbonding status, also confirm it to become unbonded.
```sh
sgnops confirm-unbonded-candidate --candidate <candidate-eth-address>
```

## Withdraw rewards
Please refer to the [design doc](https://www.celer.network/docs/celercore/sgn/architecture.html#reward) for better understanding of the reward withdrawal process.

1. Initialize withdrawal of rewards at sidechain:
```sh
sgncli tx validator withdraw-reward <self-eth-address>
```

2. Wait for the validators to co-sign the reward message, and query the reward message at sidechain:
```sh
sgncli query validator reward <self-eth-address>
```

3. After new reward message is signed with signer stakes greater than quorum stakes, submit the message to mainchain:
```sh
sgnops withdraw reward
```

## Other operations

Please refer to the CLI docs of [sgnops](https://github.com/celer-network/sgn/blob/master/docs/sgnops/sgnops.md) and [sgncli](https://github.com/celer-network/sgn/blob/master/docs/sgncli/sgncli.md), or try `sgnops --help` and `sgncli --help` to learn more about other operations.

## Reset local database

In case the local state of your validator is corrupted, you can try resetting the state by
running:

```sh
sgnd unsafe-reset-all
```

Normally this shouldn't be necessary. Please contact us on [Discord](https://discord.gg/uGx4fjQ)
before doing so.
