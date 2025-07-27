meerae is the blockchain built using the [Cosmos SDK](https://github.com/cosmos/cosmos-sdk).  
meerae communicates with other sovereign blockchains using [IBC](https://github.com/cosmos/ibc) for Inter-Blockchain Communication.

# meerae Network

## Mainnet Full Node Quick Start

> The current chain ID is `meerae-mainnet-1`.

Mainnet configurations and binaries are available here:  
👉 https://github.com/meerae-ai/meerae

```
- Hardware requirements
CPU: 4 core or higher
RAM: 16 GB (32 GB recommended)
DISK Storage: SSD(NVME) 2 TB (minimum 1 TB)

- Software requirements
OS: Ubuntu Server 22.04 / macOS 13+
Go version: Go 1.21+
```

### Build from source

```bash
git clone https://github.com/meerae-ai/meerae
cd meerae
make install
meeraed version
```

---

### Join mainnet

#### Genesis & Seeds

```bash
wget -O $HOME/.meerae/config/genesis.json https://raw.githubusercontent.com/meerae-ai/meerae/mainnet/config/genesis.json
wget -O $HOME/.meerae/config/config.toml https://raw.githubusercontent.com/meerae-ai/meerae/mainnet/config/config.toml
wget -O $HOME/.meerae/config/app.toml https://raw.githubusercontent.com/meerae-ai/meerae/mainnet/config/app.toml
```

You can also manually edit persistent peers in `config.toml`:

```
persistent_peers = "node1@tnode1.meerae.net:26656,node2@tnode2.meerae.net:26656,node3@tnode3.meerae.net:26656,node4@tnode4.meerae.net:26656"
```

#### Set moniker

```bash
nano ~/.meerae/config/config.toml
# edit: moniker = "<your_custom_moniker>"
```

#### Update minimum gas prices

```toml
# ~/.meerae/config/app.toml
minimum-gas-prices = "0.001umeerae"
```

---

### Start the node

```bash
meeraed start
```

To check node status:

```bash
meeraed status
```

---

### Starting from State Sync (optional)

Tendermint supports [state sync](https://docs.cosmos.network/main/run-node/state-sync) to join without downloading all historical blocks.

```toml
[statesync]
enable = true
rpc_servers = "https://tnode1.meerae.net:443,https://tnode2.meerae.net:443"
trust_height = <trusted_height>
trust_hash = "<trusted_hash>"
trust_period = "168h"
```

> Check current trust height/hash from a synced node or RPC endpoint.

---

## Wallet Setup

### Create a new key

```bash
meeraed keys add <key_name>
```

### Recover from mnemonic

```bash
meeraed keys add <key_name> --recover
```

---

## Become a Validator

Make sure your node is synced and funded before proceeding.

### Check account balance

```bash
meeraed query bank balances <your_address>
```

### Create validator

```bash
meeraed tx staking create-validator \
  --amount=10000000umeerae \
  --pubkey=$(meeraed tendermint show-validator) \
  --moniker="<your_moniker>" \
  --chain-id=meerae-mainnet-1 \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1" \
  --from=<key_name> \
  --fees=200000umeerae
```

### Check validator status

```bash
meeraed query staking validators --chain-id=meerae-mainnet-1
```

---

## Edit Validator Info

```bash
meeraed tx staking edit-validator \
  --moniker=<new_moniker> \
  --website=<your_website> \
  --identity=<keybase_id> \
  --details="<description>" \
  --chain-id=meerae-mainnet-1 \
  --from=<key_name> \
  --commission-rate=0.05
```

To link a thumbnail using Keybase:

```bash
meeraed tx staking edit-validator --identity="KEYBASE_ID" --from <key_name> --chain-id=meerae-mainnet-1
```

---

## Unjail a validator

```bash
meeraed tx slashing unjail --chain-id=meerae-mainnet-1 --fees=500umeerae --from=<key_name>
```

---

## Chain Initialization (local test)

```bash
meeraed keys add <key_name>
meeraed init <moniker> --chain-id=meerae-local-1
meeraed add-genesis-account <key_name> 200000000000umeerae
meeraed gentx <key_name> 100000000000umeerae --chain-id=meerae-local-1
meeraed collect-gentxs
meeraed start
```

---

## Run as a systemd service

```bash
sudo mkdir -p /var/log/meeraed
sudo touch /var/log/meeraed/stdout.log
sudo touch /var/log/meeraed/stderr.log
sudo nano /etc/systemd/system/meeraed.service
```

Paste the following into `meeraed.service`:

```
[Unit]
Description=meeraed daemon
After=network-online.target

[Service]
User=ubuntu
ExecStart=/home/ubuntu/go/bin/meeraed start --home=/home/ubuntu/.meerae
WorkingDirectory=/home/ubuntu/go/bin
StandardOutput=file:/var/log/meeraed/stdout.log
StandardError=file:/var/log/meeraed/stderr.log
Restart=always
RestartSec=3
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl enable meeraed.service
sudo systemctl start meeraed.service
# View logs
sudo journalctl -u meeraed -f
```

---
