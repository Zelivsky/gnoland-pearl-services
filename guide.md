# Gnoland Pearl — Validator Installation Guide

Complete step-by-step guide for setting up a Gnoland Pearl testnet node and registering as a validator.

## Requirements

- Ubuntu 22.04+ or similar Linux
- Go 1.22+
- 4GB+ RAM
- 50GB+ disk
- Open ports: 47656 (P2P), 47657 (RPC)

## 1. Install Go

```bash
wget https://go.dev/dl/go1.22.5.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.22.5.linux-amd64.tar.gz
echo 'export PATH=/usr/local/go/bin:$HOME/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
go version
```

## 2. Build Binaries

```bash
git clone https://github.com/gnolang/gno.git
cd gno && git checkout chain/pearl
make -C gno.land install.gnoland install.gnokey
```

Verify:
```bash
gnoland version
gnokey version
```

## 3. Download Genesis

```bash
mkdir -p ~/gno-pearl/gnoland-data
wget -O ~/gno-pearl/genesis.json \
  https://github.com/gnolang/gno/releases/download/chain/pearl/genesis.json
```

Verify checksum:
```bash
shasum -a 256 ~/gno-pearl/genesis.json
# Expected: c45fe60c8c8a1f859d9e4d5aad7ce4d100ff0eb78302e71318ba0de481a8dc91
```

## 4. Initialize Node

```bash
export GNOROOT=$HOME/gno

gnoland config init --config-path ~/gno-pearl/gnoland-data/config/config.toml
gnoland secrets init --data-dir ~/gno-pearl/gnoland-data/secrets
```

## 5. Configure Node

```bash
CFG=~/gno-pearl/gnoland-data/config/config.toml

# Ports (to avoid conflicts with other networks)
gnoland config set rpc.laddr "tcp://0.0.0.0:47657" --config-path $CFG
gnoland config set p2p.laddr "tcp://0.0.0.0:47656" --config-path $CFG
gnoland config set proxy_app "tcp://127.0.0.1:47658" --config-path $CFG

# Persistent peers
gnoland config set p2p.persistent_peers "g1m37xukfq6yl555k93fcyzns83qnmgyax9zm875@seed-1.pearl.testnets.gno.land:26656,g1ngukqd3khekaqjf90k45cglzm0l25wwzl2fkn2@seed-2.pearl.testnets.gno.land:26656" --config-path $CFG

# Required settings
gnoland config set application.prune_strategy syncable --config-path $CFG
gnoland config set consensus.timeout_commit 3s --config-path $CFG
gnoland config set consensus.peer_gossip_sleep_duration 10ms --config-path $CFG
gnoland config set p2p.flush_throttle_timeout 10ms --config-path $CFG

# Recommended
gnoland config set mempool.size 10000 --config-path $CFG
gnoland config set p2p.max_num_outbound_peers 40 --config-path $CFG
gnoland config set p2p.pex true --config-path $CFG
gnoland config set moniker "YourMoniker" --config-path $CFG
gnoland config set p2p.external_address "YOUR_IP:47656" --config-path $CFG
```

## 6. Create Systemd Service

```bash
sudo tee /etc/systemd/system/gnoland-pearl.service > /dev/null << 'EOF'
[Unit]
Description=Gnoland Pearl Node
After=network-online.target
Wants=network-online.target

[Service]
User=YOUR_USER
WorkingDirectory=/home/YOUR_USER/gno
Environment=GNOROOT=/home/YOUR_USER/gno
Environment=GOROOT=/usr/local/go
Environment=PATH=/usr/local/go/bin:/home/YOUR_USER/go/bin:/usr/bin:/bin
Environment=HOME=/home/YOUR_USER
ExecStart=/home/YOUR_USER/go/bin/gnoland start --chainid pearl-1 --genesis /home/YOUR_USER/gno-pearl/genesis.json --skip-genesis-sig-verification --data-dir /home/YOUR_USER/gno-pearl/gnoland-data --log-level info
Restart=on-failure
RestartSec=5s
LimitNOFILE=65535
StandardOutput=journal
StandardError=journal
SyslogIdentifier=gnoland-pearl

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable gnoland-pearl
sudo systemctl start gnoland-pearl
```

## 7. Sync from Snapshot (Fast)

```bash
# Install lz4
sudo apt install lz4 -y

# Stop node
sudo systemctl stop gnoland-pearl

# Download and extract snapshot
cd ~/gno-pearl/gnoland-data
wget -O latest.tar.lz4 https://snapshots.apollo-validator.eu/gnoland-pearl/snapshots/latest.tar.lz4
rm -rf db wal
lz4 -d latest.tar.lz4 | tar -xvf -
rm latest.tar.lz4

# Start node
sudo systemctl start gnoland-pearl
```

## 8. Check Sync Status

```bash
# Check height
curl -s http://127.0.0.1:47657/status | jq -r .result.sync_info.latest_block_height

# Check if catching up
curl -s http://127.0.0.1:47657/status | jq -r .result.sync_info.catching_up

# View logs
sudo journalctl -u gnoland-pearl -f
```

## 9. Register as Validator

### Get validator public key
```bash
export GNOROOT=$HOME/gno
gnoland secrets get validator_key --data-dir ~/gno-pearl/gnoland-data/secrets
# Note the gpub1... value
```

### Get testnet GNOT
Visit https://pearl.testnets.gno.land/faucet and request tokens for your g1... address.

### Register
```bash
gnokey maketx call \
  --pkgpath gno.land/r/gnops/valopers \
  --func Register \
  --args "YourMoniker" \
  --args "Your description" \
  --args "data-center" \
  --args "YOUR_G1_ADDRESS" \
  --args "YOUR_GPUB1_KEY" \
  --gas-fee 1000000ugnot --gas-wanted 50000000 \
  --chainid pearl-1 \
  --remote tcp://127.0.0.1:47657 \
  --broadcast \
  YOUR_KEY_NAME
```

### Wait for GovDAO approval
After registration you become a **candidate**. A GovDAO member must create and pass a proposal to add you to the active validator set.

Check status:
- Registered candidates: https://pearl.testnets.gno.land/r/gnops/valopers
- Active validators: https://pearl.testnets.gno.land/r/sys/validators/v3

## 10. Useful Commands

```bash
# Service management
sudo systemctl status gnoland-pearl
sudo systemctl restart gnoland-pearl
sudo journalctl -u gnoland-pearl -f

# Node info
curl -s http://127.0.0.1:47657/status | jq .result.node_info
curl -s http://127.0.0.1:47657/net_info | jq .result.n_peers

# Validator info
curl -s http://127.0.0.1:47657/validators | jq '.result.validators | length'

# Wallet balance
gnokey query --remote tcp://127.0.0.1:47657 auth/accounts/YOUR_G1_ADDRESS
```

## Troubleshooting

### Validator not signing blocks
Reset validator state:
```bash
sudo systemctl stop gnoland-pearl
echo '{"height":"0","round":"0","step":0}' > ~/gno-pearl/gnoland-data/secrets/priv_validator_state.json
sudo systemctl start gnoland-pearl
```

**Format note:** `height` and `round` must be strings, `step` must be a number:
- Correct: `{"height":"0","round":"0","step":0}`
- Wrong: `{"height":0,"round":0,"step":0}`

### Node won't start (GNOROOT)
Make sure GNOROOT is set in the systemd service environment.

### Port conflicts
Pearl uses ports 47xxx. Make sure no other service uses these ports:
```bash
ss -tlnp | grep 47
```

### Logs suppressed
If logs seem to disappear, check with:
```bash
sudo journalctl -u gnoland-pearl --no-pager -n 1000
```

## Links

| Resource | URL |
|----------|-----|
| Branch | https://github.com/gnolang/gno/tree/chain/pearl |
| Genesis | https://github.com/gnolang/gno/releases/download/chain/pearl/genesis.json |
| VALIDATOR.md | https://github.com/gnolang/gno/blob/chain/pearl/misc/deployments/pearl.gno.land/VALIDATOR.md |
| Faucet | https://pearl.testnets.gno.land/faucet |
| Validators | https://pearl.testnets.gno.land/r/gnops/valopers |
| Active Set | https://pearl.testnets.gno.land/r/sys/validators/v3 |
| Public RPC | https://rpc.apollo-validator.eu/gnoland-pearl/ |
| Snapshots | https://snapshots.apollo-validator.eu/gnoland-pearl/ |
| GnoScan | https://gnoscan.io |

---

*Provided by [Apollo Validator](https://apollo-validator.eu)*
