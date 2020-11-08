# Test user manual

**Note: This manual assumes familiarity with Unix command line and a basic understanding of how
Ethereum and Celer state channels work.**

1. Clone the `sgn` repository and install the `sgnops` binary:

```sh
git clone https://github.com/celer-network/sgn
cd sgn
git checkout master
make install-ops
cd ..
```

2. Clone the `sgn-networks` repository:

```sh
git clone https://github.com/celer-network/sgn-networks
cd sgn-networks
git checkout master
cd <network-chain-id>
```

3. Obtain an Ethereum endpoint URL from [Infura](https://infura.io/).

4. Fill in the `ETHEREUM_GATEWAY_URL` in `sgn.toml`. You can leave the other placeholders unfilled.

5. Create two keystores with **empty passphrase** for testing purpose. Eg.:

```sh
geth account new --lightkdf --keystore <path-to-keystore-folder>
```

6. For testnet, join our [Discord](https://discord.gg/uGx4fjQ)
server and ping us to obtain some Ropsten mock CELR tokens. You should also obtain a few Ropsten
ETH from places like the MetaMask [faucet](https://faucet.metamask.io). For mainnet, prepare your
own CELR and ETH.

7. Send CELR and ETH to `peer0` and `peer1`. Make sure `peer0` has at least 1 CELR. You can import
the keystore JSON files into MetaMask for ease of use.

8. Start the local test server:

```sh
sgnops guard-test --config ./sgn.toml --peer0 <path-to-peer0-keystore> --peer1 <path-to-peer1-keystore> --sgn-gateway <sgn-gateway>
```

For `sgn-gateway`, use `https://sgntest.celer.network/gateway` for testnet and
`https://sgn.celer.network/gateway` for mainnet.

The test program will open a Celer Channel between the peers and subscribe `peer0` to the SGN. Once
the bootstrap process is done, you will see "opened channel channel ID: <channel-id>" and "Starting
RPC HTTP server on 127.0.0.1:1317" in the end.

9. After a few minutes, in another terminal window, check if the subscription succeeded:

```sh
curl <sgn-gateway>/guard/subscription/<peer0-address>
```

10. In the following command, the two peers co-sign a new state with sequence number 10. `peer0` then
   sends the state to SGN to be guarded.

```sh
curl -X POST http://127.0.0.1:1317/requestGuard -d '{ "seq_num": "10" }'
```

11. Check if the guard request succeeded:

```sh
curl <sgn-gateway>/guard/request/<channel-id>/<peer0-address>
```

12. Now let `peer1` try to maliciously settle the channel with sequence number 9:

```sh
curl -X POST http://127.0.0.1:1317/intendSettle -d '{ "seq_num": "9" }'
```

13. Keep checking if the SGN guards the channel successfully:

```sh
curl http://127.0.0.1:1317/channelInfo
```

The channel will initially be in the `Operable` status. It will change to the `Settling` status with
a `SeqNum` of 9 once the settlement request from `peer1` is confirmed on-chain. After a few minutes,
the SGN should guard the channel and advance the `SeqNum` to 10. While the test program does not
confirm the settlement so the channel will not be `Settled`, the malicious settlement request is
already defeated once the SGN advances the `SeqNum`.
