# Validator manual

**Note: This manual assumes familiarity with Unix command line and blockchain validator nodes.
Prior experience with Cosmos SDK will be helpful.**

## Prepare machine and install dependencies

1. Prepare a Linux machine with a minimum of 1GB RAM (2GB recommended), at least 2 cores and at
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

Repeat to create multiple transactors for the gateway. Use the **same passphrase** as the validator
account for all accounts.

To get the list of accounts, run:

```sh
sgncli keys list --keyring-backend=file
```

8. Prepare an Ethereum gateway URL. You can use services like [Infura](https://infura.io/),
[Alchemy](https://alchemyapi.io/) or run your own node.

9. Prepare an Ethereum keystore JSON file for the validator, eg.:

```shellscript
geth account new --lightkdf --keystore <path-to-keystore-folder>
```

Save it as `$HOME/.sgncli/eth_ks.json`.

10. For testnet, join our [Discord](https://discord.gg/uGx4fjQ) server and ping us to obtain some
Ropsten mock CELR tokens. You should also obtain a few Ropsten ETH from places like the MetaMask
[faucet](https://faucet.metamask.io).

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

2. Delegate CELR to the candidate:

```sh
sgnops delegate --candidate <candidate-eth-address> --amount <delegate-amount>
```

3. Verify candidate status:

```sh
sgncli query validator candidate <candidate-eth-address>
```

4. Verify validator is in the SGN validator set:

```sh
sgncli query validator validator <validator-account-address>
```

5. Verify validator is in the Tendermint validator set:

```sh
sgncli query tendermint-validator-set
```

## Setup gateway

You can start a gateway service alongside the validator.

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