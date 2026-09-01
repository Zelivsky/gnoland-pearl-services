# Gnoland Pearl Services

Public infrastructure and tools for [Gnoland Pearl](https://github.com/gnolang/gno/tree/chain/pearl) testnet operators by [Apollo Validator](https://apollo-validator.eu).

[![Pearl](https://img.shields.io/badge/Chain-pearl--1-blue)](https://pearl.testnets.gno.land)
[![RPC](https://img.shields.io/badge/RPC-rpc.apollo--validator.eu-green)](https://rpc.apollo-validator.eu/gnoland-pearl/)
[![Snapshots](https://img.shields.io/badge/Snapshots-snapshots.apollo--validator.eu-orange)](https://snapshots.apollo-validator.eu/gnoland-pearl/)

## Services

| # | Service | Endpoint | Status |
|---|---------|----------|--------|
| 1 | [Public RPC](#public-rpc) | [rpc.apollo-validator.eu/gnoland-pearl/](https://rpc.apollo-validator.eu/gnoland-pearl/) | 🟢 Active |
| 2 | [Snapshots](#snapshots) | [snapshots.apollo-validator.eu/gnoland-pearl/](https://snapshots.apollo-validator.eu/gnoland-pearl/) | 🟢 Active |
| 3 | [Peers List](#peers-list) | [peers.md](peers.md) (auto-updated every 6h) | 🟢 Active |
| 4 | [Installation Guide](#installation-guide) | [guide.md](guide.md) | 🟢 Active |
| 5 | [Monitor Bot](#monitor-bot) | [@gnoland_pearl_apollo_bot](https://t.me/gnoland_pearl_apollo_bot) | 🟢 Active |

---

## Public RPC

Base URL: `https://rpc.apollo-validator.eu/gnoland-pearl/`

### Available Endpoints

| Endpoint | Description |
|---|---|
| `/status` | Node sync status, block height |
| `/net_info` | Network peers info |
| `/validators` | Active validator set |
| `/block?height=N` | Block by height |
| `/commit?height=N` | Block commit |

### Usage Examples

```bash
# Check node status
curl -s https://rpc.apollo-validator.eu/gnoland-pearl/status | jq

# Get current block height
curl -s https://rpc.apollo-validator.eu/gnoland-pearl/status | jq -r '.result.sync_info.latest_block_height'

# Get network peers
curl -s https://rpc.apollo-validator.eu/gnoland-pearl/net_info | jq -r '.result.n_peers'
```

### Configuration for Wallets/Tools

```
RPC URL: https://rpc.apollo-validator.eu/gnoland-pearl/
Chain ID: pearl-1
```

---

## Snapshots

Page: [https://snapshots.apollo-validator.eu/gnoland-pearl/](https://snapshots.apollo-validator.eu/gnoland-pearl/)

Snapshots are created every 6 hours and stored for fast node synchronization. Only the latest snapshot is kept.

### Latest Snapshot

| Field | Value |
|-------|-------|
| **Height** | 124,979 |
| **Size** | 2.2G |
| **Created** | 2026-09-01 15:53 UTC |
| **Update frequency** | Every 6 hours |

See the live snapshot page for the most up-to-date information:
https://snapshots.apollo-validator.eu/gnoland-pearl/

### Download Latest

```bash
# Get snapshot info
curl -s https://snapshots.apollo-validator.eu/api/gnoland-pearl/snapshots/latest | jq

# Download latest snapshot
wget -O gnoland-pearl-snapshot.tar.lz4 https://snapshots.apollo-validator.eu/gnoland-pearl/snapshots/latest.tar.lz4
```

### Restore from Snapshot

```bash
# Install lz4
sudo apt install lz4 -y

# Stop node
sudo systemctl stop gnoland-pearl

# Backup validator state (validators only)
cp ~/gno-pearl/gnoland-data/secrets/priv_validator_state.json ~/priv_validator_state.json.bak

# Download latest snapshot
cd ~/gno-pearl/gnoland-data
wget -O latest.tar.lz4 https://snapshots.apollo-validator.eu/gnoland-pearl/snapshots/latest.tar.lz4

# Restore
rm -rf db wal
lz4 -d latest.tar.lz4 | tar -xvf -
rm latest.tar.lz4

# Restore validator state
cp ~/priv_validator_state.json.bak ~/gno-pearl/gnoland-data/secrets/priv_validator_state.json

# Start node
sudo systemctl start gnoland-pearl
```

---

## Peers List

Auto-updated every 6 hours from `/net_info` RPC.

Full list: [peers.md](peers.md)

### Quick Setup

```bash
# Copy the persistent_peers line from peers.md
# Edit ~/gno-pearl/gnoland-data/config/config.toml
persistent_peers = "<peers_from_peers.md>"

# Restart node
sudo systemctl restart gnoland-pearl
```

### Official Seeds

```
g1m37xukfq6yl555k93fcyzns83qnmgyax9zm875@seed-1.pearl.testnets.gno.land:26656
g1ngukqd3khekaqjf90k45cglzm0l25wwzl2fkn2@seed-2.pearl.testnets.gno.land:26656
```

---

## Installation Guide

Full guide: [guide.md](guide.md)

### Quick Start

```bash
# Install Go
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
echo 'export PATH=/usr/local/go/bin:$HOME/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Build binaries
git clone https://github.com/gnolang/gno.git
cd gno && git checkout chain/pearl
make -C gno.land install.gnoland install.gnokey

# Download genesis
mkdir -p ~/gno-pearl/gnoland-data
wget -O ~/gno-pearl/genesis.json https://github.com/gnolang/gno/releases/download/chain/pearl/genesis.json

# Initialize and configure
export GNOROOT=$HOME/gno
gnoland config init --config-path ~/gno-pearl/gnoland-data/config/config.toml
gnoland secrets init --data-dir ~/gno-pearl/gnoland-data/secrets

# Configure ports (47xxx to avoid conflicts)
CFG=~/gno-pearl/gnoland-data/config/config.toml
gnoland config set rpc.laddr "tcp://0.0.0.0:47657" --config-path $CFG
gnoland config set p2p.laddr "tcp://0.0.0.0:47656" --config-path $CFG
gnoland config set proxy_app "tcp://127.0.0.1:47658" --config-path $CFG
gnoland config set p2p.persistent_peers "g1m37xukfq6yl555k93fcyzns83qnmgyax9zm875@seed-1.pearl.testnets.gno.land:26656,g1ngukqd3khekaqjf90k45cglzm0l25wwzl2fkn2@seed-2.pearl.testnets.gno.land:26656" --config-path $CFG

# Start node
gnoland start --chainid pearl-1 --genesis ~/gno-pearl/genesis.json --skip-genesis-sig-verification --data-dir ~/gno-pearl/gnoland-data
```

---

## Monitor Bot

Telegram bot for real-time validator monitoring and missed block alerts.

**Features:**
- `/status` — network status
- `/peers` — live peer list
- `/snapshot` — latest snapshot info
- `/add <moniker>` — add validator to watchlist
- `/remove <moniker>` — remove from watchlist
- `/list` — your watchlist
- `/check` — check uptime for watchlist
- `/map <moniker>` — validator info lookup
- `/threshold <n>` — missed blocks alert threshold

**Bot:** [@gnoland_pearl_apollo_bot](https://t.me/gnoland_pearl_apollo_bot)

---

## Chain Information

| Parameter | Value |
|---|---|
| Chain ID | pearl-1 |
| Branch | chain/pearl |
| Genesis | [Download](https://github.com/gnolang/gno/releases/download/chain/pearl/genesis.json) |
| Genesis SHA256 | `c45fe60c8c8a1f859d9e4d5aad7ce4d100ff0eb78302e71318ba0de481a8dc91` |
| P2P Port | 47656 |
| RPC Port | 47657 |
| ABCI Port | 47658 |

---

## Links

- [GitHub](https://github.com/gnolang/gno/tree/chain/pearl)
- [VALIDATOR.md](https://github.com/gnolang/gno/blob/chain/pearl/misc/deployments/pearl.gno.land/VALIDATOR.md)
- [Faucet](https://pearl.testnets.gno.land/faucet)
- [GnoScan](https://gnoscan.io)

---

*Provided by [Apollo Validator](https://apollo-validator.eu)*
