# Gnoland Pearl Services

Public infrastructure and tools for Gnoland Pearl testnet operators by [Apollo Validator](https://apollo-validator.eu).

## Services

| Service | URL | Status |
|---------|-----|--------|
| Public RPC | https://rpc.apollo-validator.eu/gnoland-pearl/ | Active |
| Snapshots | https://snapshots.apollo-validator.eu/gnoland-pearl/ | Active |
| Peers | https://snapshots.apollo-validator.eu/gnoland-pearl/peers.html | Active |
| Snapshot API | https://snapshots.apollo-validator.eu/api/gnoland-pearl/snapshots/latest | Active |
| Faucet | https://pearl.testnets.gno.land/faucet | Active |
| Validators | https://pearl.testnets.gno.land/r/gnops/valopers | Active |
| Active Set | https://pearl.testnets.gno.land/r/sys/validators/v3 | Active |

## Quick Start

See [guide.md](guide.md) for a complete step-by-step installation guide.

### Snapshot Restore

```bash
# Install lz4
sudo apt install lz4 -y

# Stop node
sudo systemctl stop gnoland-pearl

# Backup validator state
cp ~/gno-pearl/gnoland-data/secrets/priv_validator_state.json ~/priv_validator_state.json.bak

# Download and extract snapshot
cd ~/gno-pearl/gnoland-data
wget -O latest.tar.lz4 https://snapshots.apollo-validator.eu/gnoland-pearl/snapshots/latest.tar.lz4
rm -rf db wal
lz4 -d latest.tar.lz4 | tar -xvf -
rm latest.tar.lz4

# Restore validator state
cp ~/priv_validator_state.json.bak ~/gno-pearl/gnoland-data/secrets/priv_validator_state.json

# Start node
sudo systemctl start gnoland-pearl
```

### Persistent Peers

See [peers.md](peers.md) for the current peer list.

Official seeds:
```
g1m37xukfq6yl555k93fcyzns83qnmgyax9zm875@seed-1.pearl.testnets.gno.land:26656
g1ngukqd3khekaqjf90k45cglzm0l25wwzl2fkn2@seed-2.pearl.testnets.gno.land:26656
```

## Network Info

| Parameter | Value |
|-----------|-------|
| Chain ID | pearl-1 |
| Branch | chain/pearl |
| Genesis | [Download](https://github.com/gnolang/gno/releases/download/chain/pearl/genesis.json) |
| Genesis SHA256 | `c45fe60c8c8a1f859d9e4d5aad7ce4d100ff0eb78302e71318ba0de481a8dc91` |
| P2P Port | 47656 |
| RPC Port | 47657 |
| ABCI Port | 47658 |

## Validator Registration

```bash
# Get validator public key
gnoland secrets get validator_key --data-dir ~/gno-pearl/gnoland-data/secrets

# Get GNOT from faucet
# https://pearl.testnets.gno.land/faucet

# Register as validator candidate
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

After registration, a GovDAO member must create and pass a proposal to add you to the active validator set.

## Links

- [Gno Pearl Branch](https://github.com/gnolang/gno/tree/chain/pearl)
- [VALIDATOR.md](https://github.com/gnolang/gno/blob/chain/pearl/misc/deployments/pearl.gno.land/VALIDATOR.md)
- [Pearl Faucet](https://pearl.testnets.gno.land/faucet)
- [Pearl Validators](https://pearl.testnets.gno.land/r/gnops/valopers)
- [GnoScan](https://gnoscan.io)

---

*Provided by [Apollo Validator](https://apollo-validator.eu)*
