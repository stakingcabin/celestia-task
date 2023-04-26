# Setup Validator on Linux
This doc introduce how to setup a celestia validator from scratch.You can get much help from official discord channels.

## Minimum hardware requirements for validator

* Memory: **8 GB RAM**
* CPU: **6 cores/12vCores**
* Disk: **500 GB SSD Storage**
* Bandwidth: **1 Gbps for Download/1 Gbps for Upload**


## Environment setup
- prepare development environment for building binaries 
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential git make ncdu -y
```

- install go
<code>
ver="1.20.2" 
cd $HOME 
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" 
sudo rm -rf /usr/local/go 
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" 
rm "go$ver.linux-amd64.tar.gz" 
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> /etc/profile
source /etc/profile
</code>

## Build binaries celestia-appd
<code>
git clone https://github.com/celestiaorg/celestia-app.git 
cd celestia-app/ 
git checkout <v0.12.2>
make install
</code>

## setup your own validator.
- add your account
```sh
celestia-appd keys add <your-account-name> [--keyring-backend os]
```
- init config
```sh
celestia-appd init <your-moniker-name> --chain-id <blockspacerace-0>
cd $HOME
wget https://github.com/celestiaorg/networks/blob/master/blockspacerace/genesis.json
mv genesis.json $HOME/.celestia-app/config/
```
config seeds and peers from https://github.com/celestiaorg/networks/blob/master/blockspacerace/
```sh
SEEDS="0293f2cf7184da95bc6ea6ff31c7e97578b9c7ff@65.109.106.95:26656,8f14ec71e1d712c912c27485a169c2519628cfb6@celest-test-seed.theamsolutions.info:22256"
sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" ~/./.celestia-app/config/config.toml
```

## Syncing
This is optional, 
There are two alternatives for speed up syncing.

### Using state sync

State sync uses light client verification to verify state snapshots from peers
and then apply them. State sync relies on weak subjectivity; a trusted header
(specifically the hash and height) must be provided. 

In `$HOME/.celestia-app/config/config.toml`, set

```toml
rpc_servers = ""
trust_height = 0
trust_hash = ""
```

to their respective fields. At least two different rpc endpoints should be provided.
The more, the greater the chance of detecting any fraudulent behavior.

Once setup, you should be ready to start the node as normal. In the logs, you should
see: `Discovering snapshots`. This may take a few minutes before snapshots are found
depending on the network topology.

### Using snapshot data

Quick sync effectively downloads the entire `data` directory from a third-party provider
meaning the node has all the application and blockchain state as the node it was
copied from.


```sh
cd $HOME
rm -rf ~/.celestia-app/data
mkdir -p ~/.celestia-app/data
wget -O - https://snaps.qubelabs.io/celestia/snapshot-name*.tar | tar xf - \
    -C ~/.celestia-app/data/
```

## Start the celestia-app
Now you can start you validator
```sh
celestia-appd start
```

## Create validator
### first you need get some token for create validator
If is testnet you can get from official discord faucet channel
If is mainnet you can buy from DEX or CEX

### Create validator
```sh
MONIKER="your-moniker-name"
VALIDATOR_WALLET="validator"

celestia-appd tx staking create-validator \
    --amount=1000000utia \
    --pubkey=$(celestia-appd tendermint show-validator) \
    --moniker=<your-moniker-name> \
    --chain-id=<blockspacerace-0> \
    --commission-rate=0.1 \
    --commission-max-rate=0.2 \
    --commission-max-change-rate=0.01 \
    --min-self-delegation=1000000 \
    --from=$VALIDATOR_WALLET \
    --evm-address=$EVM_ADDRESS \
    --keyring-backend=os \
	--details="<your company description>" \
	--security-contact="<your-email-address>" \
	--website="<your website url>" \
	--identity="<your keybase identity>"	
```

### Delgate to validator
Make sure you have enough stake to ensure that you are in the active set
```sh
celestia-appd  tx staking delegate $(celestia-appd keys show <your-monkier-name> --bech=val --keyring-backend=os) <1000000000>utia --from stakingcabin  --fees 2000utia --chain-id blockspacerace-0 --keyring-backend=os
```

### check your validator signed latest block
```sh
Status=$(curl localhost:26657/status 2>/dev/null)
validator_address=$(echo $Status |  jq -r .result.validator_info.address)
Height=$(echo $Status | jq -r  .result.sync_info.latest_block_height)
curl localhost:26657/block?height=$Height 2>/dev/null | jq --arg address "$validator_address" '.result.block.last_commit.signatures[] | select(.validator_address == $address)'
```

<!-- sign true or false -->
```sh
Status=$(curl localhost:26657/status 2>/dev/null)
validator_address=$(echo $Status |  jq -r .result.validator_info.address)
Height=$(echo $Status | jq -r  .result.sync_info.latest_block_height)
curl localhost:26657/block?height=$Height 2>/dev/null | jq --arg address "$validator_address" '.result.block.last_commit.signatures[] | select(.validator_address == $address)' | jq 'length != 0' 
```
