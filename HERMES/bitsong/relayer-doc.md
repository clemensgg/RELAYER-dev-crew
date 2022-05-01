---
description: >-
  This section contains instructions on how to setup the rust relayer "hermes"
  and relay IBC packets between bitsong-2b and other IBC-enabled networks.
---

# Relayer

## Official BitSong IBC Channels

| source chain-id | source channel | source denom | destination chain-id | destinaion channel | IBC token-address on destinaion chain                                |
| --------------- | -------------- | ------------ | -------------------- | ------------------ | -------------------------------------------------------------------- |
| bitsong-2b      | channel-0      | ubtsg        | osmosis-1            | channel-73         | IBC/4E5444C35610CC76FC94E7F7886B93121175C28262DDFDDE6F84E82BF2425452 |
| bitsong-2b      | channel-1      | ubtsg        | cosmoshub-4          | channel-229        | IBC/E7D5E9D0E9BF8B7354929A817DD28D4D017E745F638954764AA88522A7A409EC |
| osmosis-1       | channel-73     | uosmo        | bitsong-2b           | channel-0          | IBC/ED07A3391A112B175915CD8FAF43A2DA8E4790EDE12566649D0C2F97716B8518 |
| cosmoshub-4     | channel-229    | uatom        | bitsong-2b           | channel-1          | IBC/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9 |

## Relayer Tutorial

Hardware specs:

* 16+ vCPUs or Intel or AMD 16 core CPU
* at least 64GB RAM
* 4TB+ nVME drives

To assist operators in setting up relayers, Bitsong provides tutorials for the following IBC relayers:

### Hermes (rust)

[https://hermes.informal.systems/](https://hermes.informal.systems)

Pre-requisites:

* latest go-version [https://golang.org/doc/install](https://golang.org/doc/install)
* Fresh Rust installation: For instructions on how to install Rust on your machine please follow the official Notes about Rust installation at [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install)
* build-essential, git
* openssl for rust. The OpenSSL library with its headers is required. Refer to [https://docs.rs/openssl/0.10.38/openssl/](https://docs.rs/openssl/0.10.38/openssl/)

_It is recommended to always build binaries on dedicated machine (dev-box), as dev dependencies (rust & go) shouldn't be on your production machine_ 
```sh
sudo apt install librust-openssl-dev build-essential git
```

#### Setup full nodes & configure seeds, peers and endpoints

To successfully relay IBC packets you need to run private full nodes (custom pruning or archive node) on all networks you want to support. Since relaying-success highly depends on latency and disk-IO-rate it is currently recommended to service these full/archive nodes on the same machine as the relayer process.

Because the relaying process needs to be able to query the chain back in height for at least 2/3 of the unstaking period ("trusting period") it is recommended to use pruning settings that will keep the full chain-state for a longer period of time than the unstaking period:

_edit app.toml - note: at an average block time of 6.5sec pruning-keep-recent=400000 will result in a retained chainstate of \~30d. This will suffice for most cosmos-sdk chains with an unstaking period < 30d_
```toml
pruning="custom"
pruning-keep-recent=400000 
pruning-keep-every=0 
pruning-interval=100
```

hermes needs to be able to query the RPC- and gRPC-endpoints of your nodes, you will need to maintain a well-organized port-setup.

_edit app.toml & config.toml, choose a unique port-config for each chain, write down your port-config_

_app.toml_
```toml
[grpc]

# Enable defines if the gRPC server should be enabled.
enable = true

# Address defines the gRPC server address to bind to.
address = "0.0.0.0:7012"
```

_config.toml - choose unique pprof\_laddr port_
```toml
# pprof listen address (https://golang.org/pkg/net/http/pprof)
pprof_laddr = "localhost:7019"
```
```toml
[rpc]

# TCP or UNIX socket address for the RPC server to listen on
laddr = "tcp://127.0.0.1:7013"
```
```toml
[p2p]

# Address to listen for incoming connections
laddr = "tcp://0.0.0.0:7010"
```

_config.toml - set persistent-peers & seeds for each chain_

bitsong-2b seeds:
```sh
 ffa27441ca78a5d41a36f6d505b67a145fd54d8a@95.217.156.228:26656,efd52c1e56b460b1f37d73c8d2bd5f860b41d2ba@65.21.62.83:26656
```

bitsong-2b persistent-peers: 
```sh
a62038142844828483dbf16fa6dd159f6857c81b@173.212.247.98:26656,e9fea0509b1a2d16a10ef9fdea0a4e3edc7ca485@185.144.83.158:26656,8208adac8b09f3e2499dfaef24bb89a2d190a7a3@164.68.109.246:26656,cf031ac1cf44c9c311b5967712899391a434da9a@161.97.97.61:26656,d6b2ae82c38927fa7b7630346bd84772e632983a@157.90.95.104:15631,a5885669c1f7860bfe28071a7ec00cc45b2fcbc3@144.91.85.56:26656,325a5920a614e2375fea90f8a08d8b8d612fdd1e@137.74.18.30:26656,ae2787a337c3599b16410f3ac09d6918da2e5c37@46.101.238.149:26656,9336f75cd99ff6e5cdb6335e8d1a2c91b81d84b9@65.21.0.232:26656,9c6e52e78f112a55146b09110d1d1be47702df27@135.181.211.184:36656
```

osmosis-1 seeds: 
```sh
83adaa38d1c15450056050fd4c9763fcc7e02e2c@ec2-44-234-84-104.us-west-2.compute.amazonaws.com:26656,23142ab5d94ad7fa3433a889dcd3c6bb6d5f247d@95.217.193.163:26656,f82d1a360dc92d4e74fdc2c8e32f4239e59aebdf@95.217.121.243:26656,e437756a853061cc6f1639c2ac997d9f7e84be67@144.76.183.180:26656,f515a8599b40f0e84dfad935ba414674ab11a668@osmosis.blockpane.com:26656
```

osmosis-1 persistent-peers: 
```sh
147d0fe101bbd9e200ccbe3d353d5e7762cb02ee@207.154.201.8:26656, 9f77af7811da143f339402394ee71e42d5e2fe61@46.101.171.174:26656, d518832e4ded0484183fef3509d9f23ebb70b528@46.101.202.54:26656, 8f67a2fcdd7ade970b1983bf1697111d35dfdd6f@52.79.199.137:26656, 00c328a33578466c711874ec5ee7ada75951f99a@35.82.201.64:26656, cfb6f2d686014135d4a6034aa6645abd0020cac6@52.79.88.57:26656, 8d9967d5f865c68f6fe2630c0f725b0363554e77@134.255.252.173:26656, 785bc83577e3980545bac051de8f57a9fd82695f@194.233.164.146:26656, 778fdedf6effe996f039f22901a3360bc838b52e@161.97.187.189:36657, 64d36f3a186a113c02db0cf7c588c7c85d946b5b@209.97.132.170:26656, 4d9ac3510d9f5cfc975a28eb2a7b8da866f7bc47@37.187.38.191:26656, 2115945f074ddb038de5d835e287fa03e32f0628@95.217.43.85:26656
```

cosmoshub-4 seeds:
```sh
bf8328b66dceb4987e5cd94430af66045e59899f@public-seed.cosmos.vitwit.com:26656,cfd785a4224c7940e9a10f6c1ab24c343e923bec@164.68.107.188:26656,d72b3011ed46d783e369fdf8ae2055b99a1e5074@173.249.50.25:26656,ba3bacc714817218562f743178228f23678b2873@public-seed-node.cosmoshub.certus.one:26656,3c7cad4154967a294b3ba1cc752e40e8779640ad@84.201.128.115:26656
```

cosmoshub-4 persistent-peers:
```sh
ee27245d88c632a556cf72cc7f3587380c09b469@45.79.249.253:26656,538ebe0086f0f5e9ca922dae0462cc87e22f0a50@34.122.34.67:26656,d3209b9f88eec64f10555a11ecbf797bb0fa29f4@34.125.169.233:26656,bdc2c3d410ca7731411b7e46a252012323fbbf37@34.83.209.166:26656,585794737e6b318957088e645e17c0669f3b11fc@54.160.123.34:26656,11dfe200894f38e411beca77928e9dd118e66813@94.130.98.157:26656
```

_please reference_ [_https://github.com/cosmos/chain-registry_](https://github.com/cosmos/chain-registry) _for a maintained list of peers & seeds._

To simplify the config process you can use Environment-Variables in the systemd file:
`sudo vim /etc/systemd/system/bitsongd.service`
```service
[Unit]
Description=Bitsong Daemon

[Service]
User=relay
Environment=BITSONGD_P2P_LADDR=tcp://0.0.0.0:7010
Environment=BITSONGD_RPC_LADDR=tcp://0.0.0.0:7011
Environment=BITSONGD_GRPC_ADDRESS=127.0.0.1:7012
Environment=BITSONGD_PPROF_LADDR=localhost:7019
Environment=BITSONGD_P2P_PERSISTENT_PEERS="a62038142844828483dbf16fa6dd159f6857c81b@173.212.247.98:26656,e9fea0509b1a2d16a10ef9fdea0a4e3edc7ca485@185.144.83.158:26656,8208adac8b09f3e2499dfaef24bb89a2d190a7a3@164.68.109.246:26656,cf031ac1cf44c9c311b5967712899391a434da9a@161.97.97.61:26656,d6b2ae82c38927fa7b7630346bd84772e632983a@157.90.95.104:15631,a5885669c1f7860bfe28071a7ec00cc45b2fcbc3@144.91.85.56:26656,325a5920a614e2375fea90f8a08d8b8d612fdd1e@137.74.18.30:26656,ae2787a337c3599b16410f3ac09d6918da2e5c37@46.101.238.149:26656,9336f75cd99ff6e5cdb6335e8d1a2c91b81d84b9@65.21.0.232:26656,9c6e52e78f112a55146b09110d1d1be47702df27@135.181.211.184:36656"
Environment=BITSONGD_P2P_SEEDS="ffa27441ca78a5d41a36f6d505b67a145fd54d8a@95.217.156.228:26656,efd52c1e56b460b1f37d73c8d2bd5f860b41d2ba@65.21.62.83:26656"
Environment=BITSONGD_SNAPSHOT_INTERVAL=1000
Environment=BITSONGD_P2P_MAX_NUM_INBOUND_PEERS=100
Environment=BITSONGD_P2P_MAX_NUM_OUTBOUND_PEERS=100
LimitNOFILE=500000
ExecStart=/usr/local/bin/bitsongd start --pruning custom --pruning-keep-recent 400000 --pruning-keep-every=0 --pruning-interval 100 --home --x-crisis-skip-assert-invariants
Environment=BITSONGD_LOG_LEVEL=info

Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

#### Build & setup Hermes

_Beware that for security reasons this step should be done 'on some remote pc'_

Make the directory where you'll place the source, clone the hermes source repository and build it using the latest release. Optional: copy binary to /usr/bin (or preferred directory for systemd execution)
```sh
mkdir -p $HOME/hermes
git clone https://github.com/informalsystems/ibc-rs.git hermes
cd hermes
git checkout v0.14.0
cargo install ibc-relayer-cli --bin hermes --locked
sudo cp ~/.cargo/bin/hermes /usr/bin
```

_If you have built your binary on a remote machine, move the binary to your producion environment_

Make hermes config & keys directory, copy config-template to config directory:
```sh
mkdir -p $HOME/.hermes
mkdir -p $HOME/.hermes/keys
cp config.toml $HOME/.hermes
```

Check hermes version & config dir setup
```sh
hermes version
hermes 0.14.0
```

Edit hermes config (use ports according to your port config, set filter=true to filter channels you don't relay)
`vim ~/.hermes/config.toml`
```toml
# The global section has parameters that apply globally to the relayer operation.
[global]

# Specify the strategy to be used by the relayer. Default: 'packets'
# Two options are currently supported:
#   - 'all': Relay packets and perform channel and connection handshakes.
#   - 'packets': Relay packets only.
strategy = 'packets'

# Enable or disable the filtering mechanism. Default: 'false'
# Valid options are 'true', 'false'.
# Currently Hermes supports two filters:
# 1. Packet filtering on a per-chain basis; see the chain-specific
#   filter specification below in [chains.packet_filter].
# 2. Filter for all activities based on client state trust threshold; this filter
#   is parametrized with (numerator = 1, denominator = 3), so that clients with
#   thresholds different than this will be ignored.
# If set to 'true', both of the above filters will be enabled.
#filter = true # breaking in v0.11.0 where filter = true by default

# Specify the verbosity for the relayer logging output. Default: 'info'
# Valid options are 'error', 'warn', 'info', 'debug', 'trace'.
log_level = 'info'

# Parametrize the periodic packet clearing feature.
# Interval (in number of blocks) at which pending packets
# should be eagerly cleared. A value of '0' will disable
# periodic packet clearing. Default: 100
clear_packets_interval = 25

# Toggle the transaction confirmation mechanism.
# The tx confirmation mechanism periodically queries the `/tx_search` RPC
# endpoint to check that previously-submitted transactions
# (to any chain in this config file) have delivered successfully.
# Experimental feature. Affects telemetry if set to false.
# Default: true.
tx_confirmation = true


# The REST section defines parameters for Hermes' built-in RESTful API.
# https://hermes.informal.systems/rest.html
[rest]

# Whether or not to enable the REST service. Default: false
enabled = true

# Specify the IPv4/6 host over which the built-in HTTP server will serve the RESTful
# API requests. Default: 127.0.0.1
host = '127.0.0.1'

# Specify the port over which the built-in HTTP server will serve the restful API
# requests. Default: 3000
port = 3000


# The telemetry section defines parameters for Hermes' built-in telemetry capabilities.
# https://hermes.informal.systems/telemetry.html
[telemetry]

# Whether or not to enable the telemetry service. Default: false
enabled = true

# Specify the IPv4/6 host over which the built-in HTTP server will serve the metrics
# gathered by the telemetry service. Default: 127.0.0.1
host = '127.0.0.1'

# Specify the port over which the built-in HTTP server will serve the metrics gathered
# by the telemetry service. Default: 3001
port = 3001

[[chains]]
id = 'osmosis-1'
rpc_addr = 'http://localhost:7001'
grpc_addr = 'http://localhost:7002'
websocket_addr = 'ws://localhost:7001/websocket'
rpc_timeout = '10s'
account_prefix = 'osmo'
key_name = 'osmosis'
address_type = { derivation = 'osmosis' }
store_prefix = 'ibc'
default_gas = 5000000
max_gas = 15000000
gas_price = { price = 0.000, denom = 'uosmo' }
gas_adjustment = 0.1
max_msg_num = 20
max_tx_size = 2097152
clock_drift = '20s'
max_block_time = '10s'
trusting_period = '10days'
memo_prefix = ''
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
  ['transfer', 'channel-73']
]

[[chains]]
id = 'bitsong-2b'
rpc_addr = 'http://127.0.0.1:7011'
grpc_addr = 'http://127.0.0.1:7012'
websocket_addr = 'ws://127.0.0.1:7011/websocket'
rpc_timeout = '10s'
account_prefix = 'bitsong'
key_name = 'bitsong'
address_type = { derivation = 'bitsong' }
store_prefix = 'ibc'
default_gas = 2000000
max_gas = 4000000
gas_price = { price = 0.026, denom = 'ubtsg' }
gas_adjustment = 0.1
max_msg_num = 25
max_tx_size = 1800000
clock_drift = '10s'
max_block_time = '10s'
trusting_period = '14d'
memo_prefix = ''
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
 ['transfer', 'channel-0'],
 ['transfer', 'channel-1']
 ]

[[chains]]
id = 'cosmoshub-4'
rpc_addr = 'http://127.0.0.1:7021'
grpc_addr = 'http://127.0.0.1:7022'
websocket_addr = 'ws://127.0.0.1:7021/websocket'
rpc_timeout = '10s'
account_prefix = 'cosmos'
key_name = 'cosmos'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 2000000
max_gas = 3000000
gas_price = { price = 0.001, denom = 'uatom' }
gas_adjustment = 0.1
max_msg_num = 25
max_tx_size = 180000
clock_drift = '10s'
max_block_time = '10s'
trusting_period = '14days'
memo_prefix = ''
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
   ['transfer', 'channel-229']
 ]
```

Add your relaying-wallets to hermes' keyring (located in $HOME/.hermes/keys)

Best practice is to use the same mnemonic over all networks, do not use your relaying-addresses for anything else because it might lead to mismatched account sequence errors.
```sh
hermes keys restore bitsong-2b -m "24-word mnemonic seed" --hd-path m/44'/639'/0'/0/0 
hermes keys restore osmosis-1 -m "24-word mnemonic seed"
hermes keys restore cosmoshub-4 -m "24-word mnemonic seed"
```

You can validate your hermes configuration file:
```sh
hermes config validate
INFO ThreadId(01) using default configuration from '/home/relay/.hermes/config.toml'
Success: "validation passed successfully"
```

Create hermes service file:
```service
[Unit]
Description=hermes

[Service]
User=relay
ExecStart=/usr/bin/hermes start
LimitNOFILE=500000

Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Refresh service files, enable hermes on system-startup, enable all node-daemons on system-startup, start node-daemons, sync, start hermes.

_Tipp: use chainlayer quicksync to bootstrap your nodes faster:_ [_https://quicksync.io/networks/osmosis.html_](https://quicksync.io/networks/osmosis.html)

```sh
sudo systemctl daemon-reload
sudo systemctl enable hermes.service bitsongd.service osmosisd.service gaiad.service
sudo systemctl start bitsongd osmosisd gaiad
```

Watch node-daemon output to check if your nodes are syncing:
```sh
journaltctl -u bitsongd -f
```

When your nodes are fully synced you can start the hermes daemon:
```sh
sudo systemctl start hermes && journalctl -u hermes -f
```

Hermes does a chain-health-check at startup. Watch the output to check if all connected nodes are up and synced
```sh
INFO ThreadId(01) using default configuration from '/home/relay/.hermes/config.toml'
INFO ThreadId(01) telemetry service running, exposing metrics at http://0.0.0.0:3001/metrics
INFO ThreadId(01) starting REST API server listening at http://127.0.0.1:3000
INFO ThreadId(01) [osmosis-1] chain is healthy
INFO ThreadId(01) [cosmoshub-4] chain is healthy
INFO ThreadId(01) [bitsong-2b] chain is healthy
...
INFO ThreadId(01) Hermes has started
```

Hermes will try & clear any unreceived packets after startup has completed.

#### Snippets

Query Hermes for unreceived packets & acknowledgements (check if channels are "clear")
```sh
hermes query packet unreceived-packets bitsong-2b transfer channel-0
hermes query packet unreceived-acks bitsong-2b transfer channel-0
```
```sh
hermes query packet unreceived-packets osmosis-1 transfer channel-73
hermes query packet unreceived-acks bitsong-1 transfer channel-73
```

Query Hermes for packet commitments:
```sh
hermes query packet commitments osmosis-1 transfer channel-73
hermes query packet commitments bitsong-2b transfer channel-0
```

Clear channel (only works on hermes `v0.12.0` and higher)
```sh
hermes clear packets omniflixhub-1 transfer channel-1
hermes clear packets osmosis-1 transfer channel-199
```

Clear unreceived packets manually. _Experimental: you'll need to stop your hermes daemon for it not to get confused with account sequences._
```sh
hermes tx raw packet-recv osmosis-1 bitsong-2b transfer channel-0
hermes tx raw packet-recv bitsong-2b osmosis-1 transfer channel-73
```
