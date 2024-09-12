# CAT Protocol Tracker

This guide will walk you through setting up the CAT Protocol Tracker, from cloning the repository to minting tokens. Follow each step carefully to ensure a smooth installation and execution process.

## 1. Clone the Repository

Start by cloning the repository to your local machine:

```bash
git clone https://github.com/CATProtocol/cat-token-box.git
cd cat-token-box
```

## 1.1 Prerequisites

Node.js Environment
Make sure you have Node.js >=20 and yarn installed.

You can follow the guide here to install Node.js.

Also, you can check its version use this command:
```bash
node -v
```

Use this command to install yarn if it's not installed:

```bash
npm i -g yarn
```

Full Node

Postgres Database

You can install and run the above two components on your own or follow the instructions here in tracker package to start them in docker containers.

#### Installation

```bash
yarn install
```

#### Build

```bash
yarn build
```

## 1.2 CAT Tracker

Move to the `tracker` directory:

```bash
cd packages/tracker
```

The tracker reads CAT token transactions from the blockchain, stores them in a database (Postgres) in a structured way, which can be quickly retrieved via RESTful APIs. The Swagger documentation for all the APIs can be found at http://127.0.0.1:3000 after running.


#### Installation

```bash
yarn install
```

#### Build

```bash
yarn build
```


#### Before Run
The tracker needs a full node and Postgres. We use Fractal node as an example here.

Make sure you have docker installed, you can follow this guide to install it.

Update .env file with your own configuration.
Update directory permission
```bash
sudo chmod 777 docker/data
sudo chmod 777 docker/pgdata
```

Run postgresql and bitcoind:
```bash
docker compose up -d
```

#### Run the tracker service
Use Docker (Recommended)
Build docker image under the project root directory
```bash
cd ../../ && docker build -t tracker:latest .
```

Run the container
```bash
docker run -d \
  --name tracker \
  --network tracker_default \
  --add-host="host.docker.internal:host-gateway" \
  -e DATABASE_HOST="host.docker.internal" \
  -e RPC_HOST="host.docker.internal" \
  -p 3000:3000 \
  tracker:latest

```

#### Check tracker logs
```bash
docker logs -f tracker
```

#### Use yarn
development mode

```bash
yarn run start
```

production mode
```bash
yarn run start:prod
```
Note: Make sure the tracker syncs to the latest block before you run CLI over it. The sync-up progress can be found at http://127.0.0.1:3000/api after running.

## 2. Install Dependencies

Move to the `cli` directory, which handles the command-line operations:

```bash
cd packages/cli
```

Use Node.js 20
```bash
nvm use 20
```

Install the necessary dependencies:

```bash
yarn install
```

Build CLI

```bash
yarn build
```

## 2.1 CAT CLI

`cli` requires a synced [tracker](../tracker/README.md).


## Usage

1. Copy [config.example.json](config.example.json) as `config.json`. Update `config.json` with your own configuration.

All commands use the `config.json` in the current working directory by default. You can also specify a customized configuration file with `--config=your.json`.

2. Create a wallet

```bash
yarn cli wallet create
```

You should see an output similar to:

```
? What is the mnemonic value of your account? (default: generate a new mnemonic) ********
Your wallet mnemonic is:  ********
exporting address to the RPC node ... 
successfully.
```

3. Show address

```bash
yarn cli wallet address
```

You should see an output similar to:

```
Your address is bc1plfkwa8r7tt8vvtwu0wgy2m70d6cs7gwswtls0shpv9vn6h4qe7gqjjjf86
```

4. Fund your address

Deposit some satoshis to your address.


5. Show token balances

```bash
yarn cli wallet balances
```

You should see an output similar to:

```
┌──────────────────────────────────────────────────────────────────────┬────────┬─────────┐
│ tokenId                                                              │ symbol │ balance │
┼──────────────────────────────────────────────────────────────────────┼────────┼─────────┤
│ '45ee725c2c5993b3e4d308842d87e973bf1951f5f7a804b21e4dd964ecd12d6b_0' │ 'CAT'  │ '18.89' │
┴──────────────────────────────────────────────────────────────────────┴────────┴─────────┘
```

5. load wallet

    This step may not be necessary, but I still did it.

    Unload the current wallet
    If you have an already loaded wallet causing the issue, you can unload it before trying to load the new one. This prevents two wallets from trying to access the same file simultaneously:

    ```bash 
    docker exec -it tracker-bitcoind-1 bitcoin-cli -rpcuser=bitcoin -rpcpassword=opcatAwesome unloadwallet <wallet_name>
    ```

    After that, try loading the wallet again:

    ```bash
    docker exec -it tracker-bitcoind-1 bitcoin-cli -rpcuser=bitcoin -rpcpassword=opcatAwesome loadwallet <wallet_name>
    ```
    Don't forget to replace <wallet_name> with the name of your wallet. Like for cat-a464xxx3. The wallet directory should be something like cat-token-box/packages/cli/wallet.json



6. Deploy token

- deploy with a metadata json:

```bash
yarn cli deploy --metadata=example.json
```

`example.json`:

```json
{
    "name": "cat",
    "symbol": "CAT",
    "decimals": 2,
    "max": "21000000",
    "limit": "5",
    "premine": "0"
}
```

- deploy with command line options:

```bash
yarn cli deploy --name=cat --symbol=CAT --decimals=2 --max=21000000 --premine=0 --limit=5
```

You should see an output similar to:

```
Token CAT has been deployed.
TokenId: 45ee725c2c5993b3e4d308842d87e973bf1951f5f7a804b21e4dd964ecd12d6b_0
Genesis txid: 45ee725c2c5993b3e4d308842d87e973bf1951f5f7a804b21e4dd964ecd12d6b
Reveal txid: 9a3fcb5a8344f53f2ba580f7d488469346bff9efe7780fbbf8d3490e3a3a0cd7
```

> **Note:** `max` * 10^`decimals` must be less than 2^31, since Bitcoin Script only supports 32-bit signed integers. We plan to support 64 or higher bit in the future.


7. Mint token

When `amount` is not specified, `limit` number of tokens will be minted.
```bash
yarn cli mint -i [tokenId] [amount?]
```
You should see an output similar to:

```
Minting 5.00 CAT tokens in txid: 4659529141de4996ad8482910ef3e0cf63665c39e62b86f17d5d398b5748b66b ...
```

> **Note:** There is a slight chance you happen to use the same minter UTXO with another user who is also minting at the same time, and your mint attempt fails due to [UTXO contention](https://catprotocol.org/cat20#parallel-mint). Just retry till you succeed.


8. Send token

```bash
yarn cli send -i [tokenId] [receiver] [amount]
```
You should see an output similar to:

```
Sending 1.11 CAT tokens to bc1pmc274s6lalf6afrll2e23m2qmk50dwaj6srjupe5vyu4dcy66zyss2r3dy
in txid: 94e3254c1237ba7cd42eaeeae713c646ee5dd1cd6c4dd6ef07241d5336cd2aa7
```


-----------------

### FeeRate

`deploy`, `mint`, and `send` commands can all specify a fee rate via option `--fee-rate`.

## 7. Create a Wallet

Move back to the `cli` directory and create a wallet:

```bash
yarn cli wallet create
```

You will be prompted to enter or generate a mnemonic. After successfully creating the wallet, verify the address:

```bash
yarn cli wallet address
```

## 8. Fund the Wallet

Deposit some satoshis to the wallet address generated above.

## 9. Mint Tokens

Once you have your wallet funded, mint tokens using the command:

```bash
yarn cli mint -i [tokenId] [amount?] [fee-rate?]
```

Example:

```bash
yarn cli mint -i 45ee725c2c5993b3e4d308842d87e973bf1951f5f7a804b21e4dd964ecd12d6b_0 5 --fee-rate=20
```

## 10. Monitor the Sync Progress

Visit `http://127.0.0.1:3000/api` to monitor the sync progress and API status.

## 11. Send Tokens

You can send tokens with the following command:

```bash
yarn cli send -i [tokenId] [receiver] [amount]
```

For example:

```bash
yarn cli send -i 45ee725c2c5993b3e4d308842d87e973bf1951f5f7a804b21e4dd964ecd12d6b_0 bc1q... 1.5
```

## 12. Troubleshooting

- Ensure that `bitcoind` is synced and the tracker has caught up with the blockchain.
- Verify that Docker is running without issues by checking the container status with `docker ps`.

This guide covers the entire process from cloning to minting tokens on the CAT Protocol Tracker. Make sure all services are running, and you're good to go!

