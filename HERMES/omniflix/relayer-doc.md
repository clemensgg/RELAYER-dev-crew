## Official IBC Channels for the OmniFlix Hub

Below is a list of IBC channels setup for the OmniFlix Hub.

| source chain-id  | source channel  | destination chain | destination chain-id  | destination channel | comments |
| ---------------- | --------------- | --------------------- | ------------------ | ------------------ | --------------------- |
| omniflixhub-1 | channel-1 | Osmosis | osmosis-1 | channel-199 | No change |
| omniflixhub-1 | channel-4 | Terra | columbus-5 | channel-27 | No change |
| omniflixhub-1 | channel-5 | Chihuahua| chihuahua-1 | channel-17 | No change |
| omniflixhub-1 | channel-8 |  Stargaze | stargaze-1 | channel-36 | No change |
| omniflixhub-1 | channel-12 |  Cosmos Hub | cosmoshub-4 | channel-306 | Updated from `ch-0` on OmniFlix and `ch-290` on the Cosmos Hub |
| omniflixhub-1 | channel-14 | Gravity Bridge | gravity-bridge-3 | channel-51 | Updated from `ch-3` on OmniFlix and `ch-35` on the Gravity Bridge |
| omniflixhub-1 | channel-15 | Sifchain | sifchain-1 | channel-49 | Updated from `ch-6` on OmniFlix and `ch-44` on the Sifchain |
| omniflixhub-1 | channel-16 |  Akash Net | akashnet-2 | channel-42 | Updated from `ch-7` on OmniFlix and `ch-39` on the Akash Network |
| omniflixhub-1 | channel-17 |  Ki Chain | kichain-2 | channel-13 | Updated from `ch-9` on OmniFlix and `ch-10` on the Ki Chain |
| omniflixhub-1 | channel-18 |  Sentinel Hub | sentinelhub-2 | channel-54 | Updated from `ch-10` on OmniFlix and `ch-53` on the Sentinel Hub |
| omniflixhub-1 | channel-19 |  Crypto.org / Cronos | crypto-org-chain-mainnet-1 | channel-55 | Updated from `ch-11` on OmniFlix and `ch-54` on the the Crypto.com chain |
| omniflixhub-1 | channel-20 | Juno Network | juno-1 | channel-78 | Updated from `ch-13` on OmniFlix and `ch-74` on the Juno Network. Before these, the channels were `ch-2` on OmniFlix and `ch-63` on the Juno Network |

---

## Relayer Tutorial

Hardware specs:

* 16+ vCPUs or Intel or AMD 16 core CPU
* at least 64GB RAM
* 4TB+ nVME drives

To assist operators in setting up relayers, Omniflix provides tutorials for the following IBC relayers:

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


omniflixhub-1 seeds:
```toml
seeds = "9d75a06ebd3732a041df459849c21b87b2c55cde@35.187.240.195:26656,19feae28207474eb9f168fff9720fd4d418df1ed@35.240.196.102:26656"
```
osmosis-1 seeds:
```toml
seeds = "83adaa38d1c15450056050fd4c9763fcc7e02e2c@ec2-44-234-84-104.us-west-2.compute.amazonaws.com:26656,23142ab5d94ad7fa3433a889dcd3c6bb6d5f247d@95.217.193.163:26656,f82d1a360dc92d4e74fdc2c8e32f4239e59aebdf@95.217.121.243:26656,e437756a853061cc6f1639c2ac997d9f7e84be67@144.76.183.180:26656,f515a8599b40f0e84dfad935ba414674ab11a668@osmosis.blockpane.com:26656"
```
cosmoshub-4 seeds: 
```toml
seeds = "bf8328b66dceb4987e5cd94430af66045e59899f@public-seed.cosmos.vitwit.com:26656,cfd785a4224c7940e9a10f6c1ab24c343e923bec@164.68.107.188:26656,d72b3011ed46d783e369fdf8ae2055b99a1e5074@173.249.50.25:26656,ba3bacc714817218562f743178228f23678b2873@public-seed-node.cosmoshub.certus.one:26656,3c7cad4154967a294b3ba1cc752e40e8779640ad@84.201.128.115:26656"
```

*please reference https://github.com/cosmos/chain-registry for a maintained list of seeds & peers.*


To simplify the config process you can set Environment-Variables in the systemd file:

```sudo vim /etc/systemd/system/omniflixhubd.service```

```service
[Unit]
Description=Omniflix Daemon

[Service]
User=relay
Environment=OMNIFLIXHUBD_P2P_LADDR=tcp://0.0.0.0:7000
Environment=OMNIFLIXHUBD_RPC_LADDR=tcp://0.0.0.0:7001
Environment=OMNIFLIXHUBD_GRPC_ADDRESS=127.0.0.1:7002
Environment=OMNIFLIXHUBD_API_ADDRESS=tcp://127.0.0.1:7003
Environment=OMNIFLIXHUBD_P2P_SEEDS="9d75a06ebd3732a041df459849c21b87b2c55cde@35.187.240.195:26656,19feae28207474eb9f168fff9720fd4d418df1ed@35.240.196.102:26656"
LimitNOFILE=180000
ExecStart=/usr/local/bin/omniflixhubd start --grpc-web.enable="false" --grpc.enable="true" --pruning custom --pruning-keep-recent 40000 --pruning-keep-every 0 --pruning-interval 100 --x-crisis-skip-assert-invariants
Environment=OMNIFLIXHUBD_LOG_LEVEL=info

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

Edit hermes config (use ports according to your port config, add only chains you relay)
```toml
# The global section has parameters that apply globally to the relayer operation.
[global]

# Specify the strategy to be used by the relayer. Default: 'packets'
# Two options are currently supported:
#   - 'all': Relay packets and perform channel and connection handshakes.
#   - 'packets': Relay packets only.
strategy = 'packets'

# Specify the verbosity for the relayer logging output. Default: 'info'
# Valid options are 'error', 'warn', 'info', 'debug', 'trace'.
log_level = 'info'

# Parametrize the periodic packet clearing feature.
# Interval (in number of blocks) at which pending packets
# should be eagerly cleared. A value of '0' will disable
# periodic packet clearing. Default: 100
clear_packets_interval = 100

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
id = 'omniflixhub-1'
rpc_addr = 'http://127.0.0.1:7001'
grpc_addr = 'http://127.0.0.1:7002'
websocket_addr = 'ws://127.0.0.1:7001/websocket'
rpc_timeout = '15s'
account_prefix = 'omniflix'
key_name = 'omniflix'
store_prefix = 'ibc'
max_gas = 4000000
default_gas = 1000000
fee_granter = '<omniflix granter address>' # to receive the granter address please fill out the relayer-coordination-sheet provided by the team
gas_price = { price = 0.001, denom = 'uflix' }
gas_adjustment = 0.1
max_msg_num = 25
max_tx_size = 180000
clock_drift = '15s'
trusting_period = '14days'
trust_threshold = { numerator = '1', denominator = '3' }
memo_prefix = ''
[chains.packet_filter]
policy = 'allow'
list = [
  ['transfer', 'channel-0'],  #cosmos
  ['transfer', 'channel-1'],  #osmosis
#  ['transfer', 'channel-2'],  #juno
#  ['transfer', 'channel-3'],  #gravity
  ['transfer', 'channel-4'],  #terra
#  ['transfer', 'channel-5'],  #chihuahua
#  ['transfer', 'channel-6'],  #sifchain
#  ['transfer', 'channel-7'],  #akash
#  ['transfer', 'channel-8'],  #stargaze
#  ['transfer', 'channel-9'],  #kichain
#  ['transfer', 'channel-10'],  #sentinel
]

[[chains]]
id = 'cosmoshub-4'
rpc_addr = 'http://127.0.0.1:7011'
grpc_addr = 'http://127.0.0.1:7012'
websocket_addr = 'ws://127.0.0.1:7011/websocket'
rpc_timeout = '10s'
account_prefix = 'cosmos'
key_name = 'cosmos'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 2000000
max_gas = 10000000
gas_price = { price = 0.005, denom = 'uatom' }
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
   ['transfer', 'channel-290'], # omniflixhub-1
]

[[chains]]
id = 'osmosis-1'
rpc_addr = 'http://127.0.0.1:7021'
grpc_addr = 'http://127.0.0.1:7022'
websocket_addr = 'ws://127.0.0.1:7021/websocket'
rpc_timeout = '10s'
account_prefix = 'osmo'
key_name = 'osmosis'
address_type = { derivation = 'osmosis' }
store_prefix = 'ibc'
default_gas = 5000000
max_gas = 15000000
gas_price = { price = 0.0026, denom = 'uosmo' }
gas_adjustment = 0.1
max_msg_num = 20
max_tx_size = 209715
clock_drift = '20s'
max_block_time = '10s'
trusting_period = '10days'
memo_prefix = ''
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
  ['transfer', 'channel-199'], # omniflixhub-1
]

[[chains]]
id = 'columbus-5'
rpc_addr = 'http://127.0.0.1:7031'
grpc_addr = 'http://127.0.0.1:7032'
websocket_addr = 'ws://127.0.0.1:7031/websocket'
rpc_timeout = '10s'
account_prefix = 'terra'
key_name = 'terra'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 3000000
max_gas = 10000000
gas_price = { price = 170, denom = 'ukrw' } #0.0147 uluna
gas_adjustment = 0.2
max_msg_num = 30
max_tx_size = 180000
clock_drift = '15s'
max_block_time = '10s'
trusting_period = '14days'
memo_prefix = ''
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
 ['transfer', 'channel-27'], # omniflixhub-1
]

```

Add your relayer-wallets to Hermes' keyring (located in $HOME/.hermes/keys)

Best practice is to use the same mnemonic over all networks, do not use your relaying-addresses for anything else because it will lead to account sequence errors. If you do not use the same mnemonic, double check and let the team know in case of any further issues. 

```sh
hermes keys restore omniflixhub-1 -m "24-word mnemonic seed"
hermes keys restore cosmoshub-4 -m "24-word mnemonic seed"
hermes keys restore osmosis-1 -m "24-word mnemonic seed"
hermes keys restore columbus-5 -m "24-word mnemonic seed"
```

You can validate your hermes configuration file:
```sh
hermes config validate
INFO ThreadId(01) using default configuration from '/home/relay/.hermes/config.toml'
Success: "validation passed successfully"
```

Create a systemd service-file for hermes production:

```sudo vim /etc/systemd/system/hermes.service```

```service
[Unit]
Description=hermes

[Service]
User=relay
ExecStart=/usr/bin/hermes start
LimitNOFILE=180000

Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Refresh service files, enable hermes on system-startup, enable all node-daemons on system-startup, start node-daemons, sync, start hermes.

```sh
sudo systemctl daemon-reload
sudo systemctl enable hermes.service
sudo systemctl enable omniflixhubd.service
sudo systemctl enable gaiad.service
sudo systemctl enable osmosisd.service
sudo systemctl enable terrad.service
```

```sh
sudo systemctl start omniflixhubd
sudo systemctl start gaiad
sudo systemctl start osmosisd
sudo systemctl start terrad
```

Watch the node-daemon output to check if your nodes are syncing:
```sh
journaltctl -u omniflixhubd -f
```

Perform health check to see if all connected nodes are up and synced:

```hermes health-check```

```sh
INFO ThreadId(01) using default configuration from '/home/relay/.hermes/config.toml'
INFO ThreadId(01) telemetry service running, exposing metrics at http://0.0.0.0:3001/metrics
INFO ThreadId(01) starting REST API server listening at http://127.0.0.1:3000
INFO ThreadId(01) [omniflixhub-1] chain is healthy
INFO ThreadId(01) [cosmoshub-4] chain is healthy
INFO ThreadId(01) [osmosis-1] chain is healthy
INFO ThreadId(01) [columbus-5] chain is healthy
```

When your nodes are fully synced you can start the hermes daemon:
```sh
sudo systemctl start hermes && journalctl -u hermes -f
```

Watch hermes' output for successfully relayed packets or any errors.
It will try & clear any unreceived packets after startup has completed.


## Snippets

Query Hermes for unreceived packets & acknowledgements (check if channels are "clear")
```sh
hermes query packet unreceived-packets omniflixhub-1 transfer channel-1
hermes query packet unreceived-acks omniflixhub-1 transfer channel-1
```
```sh
hermes query packet unreceived-packets osmosis-1 transfer channel-199
hermes query packet unreceived-acks osmosis-1 transfer channel-199
```

Query Hermes for packet commitments:
```sh
hermes query packet commitmentss omniflixhub-1 transfer channel-1
hermes query packet commitments osmosis-1 transfer channel-199
```

Clear channel (only works on hermes `v0.12.0` and higher)
```sh
hermes clear packets omniflixhub-1 transfer channel-1
hermes clear packets osmosis-1 transfer channel-199
```

Clear unreceived packets manually. *Experimental: you'll need to stop your hermes daemon for it not to get confused with account sequences.*
```sh
hermes tx raw packet-recv osmosis-1 omniflixhub-1 transfer channel-1
hermes tx raw packet-ack osmosis-1 omniflixhub-1 transfer channel-1
hermes tx raw packet-recv omniflixhub-1 osmosis-1 transfer channel-199
hermes tx raw packet-ack omniflixhub-1 osmosis-1 transfer channel-199
```
