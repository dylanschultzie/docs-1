---
description: General instructions to join the Juno mainnet after network genesis.
cover: ../.gitbook/assets/Gitbook Banner large 6 (6).png
coverY: 0
---

# Joining Mainnet

## Mainnet binary version

The correct version of the binary for mainnet at genesis is `v1.0.0`. Its release page can be found [here](https://github.com/CosmosContracts/juno/releases/tag/v1.0.0).

## Mainnet chain-id

Below is the list of Juno mainnet id's and their current status. You will need to know the version tag for installation of the `junod` binary.&#x20;

| chain-id | Description                                        |  Status | Block Start | Block Finish |
| -------- | -------------------------------------------------- | :-----: | ----------- | ------------ |
| juno-1   | This is the first chain-id from the genesis event. | current | 0           | N/A          |

## Recommended Minimum Hardware

The minimum recommended hardware requirements for running a validator for the Juno mainnet are:

| Chain-id | Requirements                                                                                          |
| -------- | ----------------------------------------------------------------------------------------------------- |
| juno-1   | <p></p><ul><li>4 Cores (modern CPU's)</li><li>16GB RAM</li><li>1TB of storage (SSD or NVME)</li></ul> |

{% hint style="danger" %}
These specifications are the minimum recommended. As Juno Network is a smart contract platform, it can at times be very demanding on hardware. Low spec validators WILL get stuck on difficult to process blocks.
{% endhint %}

{% hint style="warning" %}
Note that the mainnet will accumulate data as the blockchain continues. This means that you will need to expand your storage as the blockchain database gets larger with time.
{% endhint %}

## junod Installation

To get up and running with the junod binary, please follow the instructions [here](getting-setup.md).

{% hint style="warning" %}
Mainnet will initially use the `v1.0.0` [tag](https://github.com/CosmosContracts/juno/releases/tag/v1.0.0) on GitHub. Make sure you build [this version](https://github.com/CosmosContracts/juno/tree/v1.0.0) of the Juno binary.
{% endhint %}

## Configuration of Shell Variables

For this guide, we will be using shell variables. This will enable the use of the client commands verbatim. It is important to remember that shell commands are only valid for the current shell session, and if the shell session is closed, the shell variables will need to be re-defined.&#x20;

If you want variables to persist for multiple sessions, then set them explicitly in your shell .profile, as you did for the Go environment variables.

To clear a variable binding, use `unset $VARIABLE_NAME` . Shell variables should be named with ALL CAPS.

### Choose the required mainnet chain-id

Choose the `<chain-id>` for the mainnet you would like to join from [here](joining-mainnet.md#mainnet-chain-id). Set the `CHAIN_ID`:

```bash
CHAIN_ID=<chain-id>

# Example
CHAIN_ID=juno-1
```

You can also set this in your `.profile` file:

```bash
export CHAIN_ID=juno-1
```

Then source it:

```bash
source .profile
```

### Set your moniker name

Choose your `<moniker-name>`, this can be any name of your choosing and will identify your validator in the explorer. Set the `MONIKER_NAME`:

```bash
MONIKER_NAME=<moniker-name>

# Example
MONIKER_NAME="Validatron 9000"
```

### **Set persistent peers**

Persistent peers will be required to tell your node where to connect to other nodes and join the network. To retrieve the peers for the chosen mainnet:

```bash
# Set the base repo URL for mainnet & retrieve peers
CHAIN_REPO="https://raw.githubusercontent.com/CosmosContracts/mainnet/main/$CHAIN_ID" && \
export PEERS="$(curl -s "$CHAIN_REPO/persistent_peers.txt")"
```

{% hint style="info" %}
NB: If you are unsure about this, you can ask in discord for the current peers and explicitly set them in `~/.juno/config/config.toml` instead.
{% endhint %}

### Set minimum gas prices

In `$HOME/.juno/config/app.toml`, set minimum gas prices:

```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025ujuno\"/" ~/.juno/config/app.toml
```

## Setting up the Node

These instructions will direct you on how to initialize your node, synchronize to the network and upgrade your node to a validator.&#x20;

### **Initialize the chain**

```bash
junod init $MONIKER_NAME --chain-id $CHAIN_ID
```

This will generate the following files in `~/.juno/config/`

* `genesis.json`&#x20;
* `node_key.json`&#x20;
* `priv_validator_key.json`

### Download the genesis file

```
curl https://raw.githubusercontent.com/CosmosContracts/mainnet/main/$CHAIN_ID/genesis.json > ~/.juno/config/genesis.json
```

This will replace the genesis file created using `junod init` command with the mainnet `genesis.json`. ****&#x20;

### **Set persistent peers**

Using the peers variable we set earlier, we can set the `persistent_peers` in `~/.juno/config/config.toml`:&#x20;

```bash
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.juno/config/config.toml
```

### **Create (or restore) a local key pair**

Either create a new key pair, or restore an existing wallet for your validator:

```bash
# Create new keypair
junod keys add <key-name>

# Restore existing juno wallet with mnemonic seed phrase.
# You will be prompted to enter mnemonic seed.
junod keys add <key-name> --recover

# Query the keystore for your public address
junod keys show <key-name> -a
```

Replace `<key-name>` with a key name of your choosing.

{% hint style="danger" %}
After creating a new key, the key information and seed phrase will be shown. It is essential to write this seed phrase down and keep it in a safe place. The seed phrase is the only way to restore your keys.
{% endhint %}

### **Get some Juno tokens**

You will require some Juno tokens to bond to your validator. To be in the active set you will need to have enough tokens to be in the top 125 validators by delegation weight.

If you do not have any Juno tokens for you validator you can purchase tokens on Osmosis or Emeris.

## Setup cosmovisor

Follow [these](setting-up-cosmovisor.md) instructions to setup cosmovisor and start the node.

{% hint style="info" %}
Using cosmovisor is completely optional. If you choose not to use cosmovisor, you will need to be sure to attend network upgrades to ensure your validator does not have downtime and get jailed.
{% endhint %}

## Syncing the node

After starting the junod daemon, the chain will begin to sync to the network. The time to sync to the network will vary depending on your setup and the current size of the blockchain, but could take a very long time. To query the status of your node:

```bash
# Query via the RPC (default port: 26657)
curl http://localhost:26657/status | jq .result.sync_info.catching_up
```

If this command returns `true` then your node is still catching up. If it returns `false` then your node has caught up to the network current block and you are safe to proceed to upgrade to a validator node.

## Upgrade to a validator

{% hint style="danger" %}
**Do not attempt to upgrade your node to a validator until the node is fully in sync as per the previous step.**
{% endhint %}

To upgrade the node to a validator, you will need to submit a `create-validator` transaction:

```bash
junod tx staking create-validator \
  --amount 9000000ujuno \
  --commission-max-change-rate "0.1" \
  --commission-max-rate "0.20" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --details "validators write bios too" \
  --pubkey=$(junod tendermint show-validator) \
  --moniker $MONIKER_NAME \
  --chain-id $CHAIN_ID \
  --gas-prices 0.025ujuno \
  --from <key-name>
```

{% hint style="info" %}
The above transaction is just an example. There are many more flags that can be set to customise your validator, such as your validator website, or keybase.io id, etc. To see a full list:

```bash
junod tx staking create-validator --help
```
{% endhint %}

## Backup critical files

There are certain files that you need to backup to be able to restore your validator if, for some reason, it damaged or lost in some way. Please make a secure backup of the following files located in `~/.juno/config/`:

* `priv_validator_key.json`
* `node_key.json`

It is recommended that you encrypt the backup of these files.
