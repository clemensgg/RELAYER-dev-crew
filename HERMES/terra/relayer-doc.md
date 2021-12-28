This section contains instructions on how to setup the rust relayer "hermes" and relay IBC packets between Terra and other IBC-enabled networks.


# official Terra IBC Channels

| source chain-id  | source channel  | source denom | destination chain-id  | destinaion channel | IBC token-address on destinaion chain |
| ---------------- | --------------- | ------------ | --------------------- | ------------------ | ------------------------------------------- |
| columbus-5 | channel-1 | uluna | osmosis-1  | channel-72  | IBC/0EF15DF2F02480ADE0BB6E85D9EBB5DAEA2836D3860E9F97F9AADE4F57A31AA0 |
| columbus-5 | channel-1 | uusd | osmosis-1  | channel-72 | IBC/BE1BB42D4BE3C30D50B68D7C41DB4DFCE9678E8EF8C539F6E6A9345048894FCC |
| columbus-5 | channel-1 | ukrw | osmosis-1  | channel-72  | IBC/204A582244FC241613DBB50B04D1D454116C58C4AF7866C186AA0D6EEAD42780 |

| columbus-5 | channel-2 | uluna | cosmoshub-4 | channel-219 | IBC/2154552F1CE0EF16FAC73B41A837A7D91DD9D2B6E193B53BE5C15AB78E1CFF40 |
| columbus-5 | channel-2 | uusd | cosmoshub-4 | channel-219 | IBC/5BAC41A24DE0C0703EA1A77866AC86BF1EF4EA71F26F132D1EFEEAF64405AB54 |
| columbus-5 | channel-2 | ukrw | cosmoshub-4 | channel-219 | IBC/BDE165955287C32EE4B125781D1CE37A7F965764FEC3E5ED15DF7CB873BA4279 |

| columbus-5 | channel-16 | uluna | secret-4 | channel-2 | IBC/D70B0FBF97AEB04491E9ABF4467A7F66CD6250F4382CE5192D856114B83738D2 |
| columbus-5 | channel-16 | uusd | secret-4 | channel-2 | IBC/F341E5FD11E5A747A62A1BA11D13D25AF9708D1C63944E1E4762ADA883BC46F5 |
| columbus-5 | channel-16 | ukrw | secret-4 | channel-2 | IBC/4294C3DB67564CF4A0B2BFACC8415A59B38243F6FF9E288FBA34F9B4823BA16E |

| osmosis-1 | channel-72 | uosmo | columbus-5 | channel-1 | IBC/0471F1C4E7AFD3F07702BEF6DC365268D64570F7C1FDC98EA6098DD6DE59817B |
| cosmoshub-4 | channel-219 | uatom | columbus-5 | channel-2 | IBC/9117A26BA81E29FA4F78F57DC2BD90CD3D26848101BA880445F119B22A1E254E |
| secret-4 | channel-2 | uscrt | columbus-5 | channel-16 | IBC/EB2CED20AB0466F18BE49285E56B31306D4C60438A022EA995BA65D5E3CF7E09 |


# Relayer Tutorial

Hardware specs:

- 16+ vCPUs or Intel or AMD 16 core CPU
- at least 64GB RAM
- 4TB+ nVME drives

To assist operators in setting up relayers, Terra provides tutorials for the following IBC relayers:

## Hermes (rust) https://hermes.informal.systems/

Pre-requisites: 

- latest go-version https://golang.org/doc/install
- Fresh Rust installation: For instructions on how to install Rust on your machine please follow the official Notes about Rust installation at https://www.rust-lang.org/tools/install
- build-essential, git
- openssl for rust. The OpenSSL library with its headers is required. Refer to https://docs.rs/openssl/0.10.38/openssl/

```
sudo apt install librust-openssl-dev build-essential git
```


### Setup full nodes & configure seeds, peers and endpoints

To successfully relay IBC packets you need to run private full nodes (custom pruning or archive node) on all networks you want to support. Since relaying-success highly depends on latency and disk-IO-rate it is currently recommended to service these full/archive nodes on the same machine as the relayer process. 

Because the relaying process needs to be able to query the chain back in height for at least 2/3 of the unstaking period ("trusting period") it is recommended to use pruning settings that will keep the full chain-state for a longer period of time than the unstaking period:

*edit app.toml - note: at an average block time of 6.5sec pruning-keep-recent=400000 will result in a retained chainstate of ~30d. This will suffice for most cosmos-sdk chains with an unstaking period < 30d*

```
pruning=custom 
pruning-keep-recent=400000 
pruning-keep-every=0 
pruning-interval=100
```

hermes needs to be able to query the API- and gRPC-endpoints of your nodes, you will need to maintain a well-organized port-setup.

*edit app.toml & config.toml, choose a unique port-config for each chain, write down your port-config*

*app.toml*
```
[api]

# Enable defines if the API server should be enabled.
enable = true

# Swagger defines if swagger documentation should automatically be registered.
swagger = false

# Address defines the API server to listen on.
address = "tcp://0.0.0.0:7001"
```
```
[grpc]

# Enable defines if the gRPC server should be enabled.
enable = true

# Address defines the gRPC server address to bind to.
address = "0.0.0.0:7002"
```

*config.toml - choose unique pprof_laddr port*
```
# pprof listen address (https://golang.org/pkg/net/http/pprof)
pprof_laddr = "localhost:7009"
```
```
[rpc]

# TCP or UNIX socket address for the RPC server to listen on
laddr = "tcp://127.0.0.1:7003"
```
```
[p2p]

# Address to listen for incoming connections
laddr = "tcp://0.0.0.0:7000"
```

*config.toml - set persistent-peers & seeds for each chain*

osmosis-1 seeds:
```
"83adaa38d1c15450056050fd4c9763fcc7e02e2c@ec2-44-234-84-104.us-west-2.compute.amazonaws.com:26656,23142ab5d94ad7fa3433a889dcd3c6bb6d5f247d@95.217.193.163:26656,f82d1a360dc92d4e74fdc2c8e32f4239e59aebdf@95.217.121.243:26656,e437756a853061cc6f1639c2ac997d9f7e84be67@144.76.183.180:26656,f515a8599b40f0e84dfad935ba414674ab11a668@osmosis.blockpane.com:26656"
```
osmosis-1 persistent-peers: 
```
"147d0fe101bbd9e200ccbe3d353d5e7762cb02ee@207.154.201.8:26656, 9f77af7811da143f339402394ee71e42d5e2fe61@46.101.171.174:26656, d518832e4ded0484183fef3509d9f23ebb70b528@46.101.202.54:26656, 8f67a2fcdd7ade970b1983bf1697111d35dfdd6f@52.79.199.137:26656, 00c328a33578466c711874ec5ee7ada75951f99a@35.82.201.64:26656, cfb6f2d686014135d4a6034aa6645abd0020cac6@52.79.88.57:26656, 8d9967d5f865c68f6fe2630c0f725b0363554e77@134.255.252.173:26656, 785bc83577e3980545bac051de8f57a9fd82695f@194.233.164.146:26656, 778fdedf6effe996f039f22901a3360bc838b52e@161.97.187.189:36657, 64d36f3a186a113c02db0cf7c588c7c85d946b5b@209.97.132.170:26656, 4d9ac3510d9f5cfc975a28eb2a7b8da866f7bc47@37.187.38.191:26656, 2115945f074ddb038de5d835e287fa03e32f0628@95.217.43.85:26656"
```
cosmoshub-4 seeds: 
```
"bf8328b66dceb4987e5cd94430af66045e59899f@public-seed.cosmos.vitwit.com:26656,cfd785a4224c7940e9a10f6c1ab24c343e923bec@164.68.107.188:26656,d72b3011ed46d783e369fdf8ae2055b99a1e5074@173.249.50.25:26656,ba3bacc714817218562f743178228f23678b2873@public-seed-node.cosmoshub.certus.one:26656,3c7cad4154967a294b3ba1cc752e40e8779640ad@84.201.128.115:26656"
```
cosmoshub-4 persistent-peers: 
```
"ee27245d88c632a556cf72cc7f3587380c09b469@45.79.249.253:26656,538ebe0086f0f5e9ca922dae0462cc87e22f0a50@34.122.34.67:26656,d3209b9f88eec64f10555a11ecbf797bb0fa29f4@34.125.169.233:26656,bdc2c3d410ca7731411b7e46a252012323fbbf37@34.83.209.166:26656,585794737e6b318957088e645e17c0669f3b11fc@54.160.123.34:26656,11dfe200894f38e411beca77928e9dd118e66813@94.130.98.157:26656"
```
secret-4 persistent-peers: 
```
"971911193b09a17c347565d311a3cc4f6004156d@peer.node.scrtlabs.com:26656, 7649dcfda0eb77b38fde8e817da8071faea3cd13@bootstrap.scrt.network:26656"
```

*please reference https://github.com/cosmos/chain-registry for a maintained list of peers & seeds.*

To simplify the config process you can set Environment-Variables in the systemd file:

```sudo vim /etc/systemd/system/terrad.service```

```
[Unit]
Description=Terra Daemon

[Service]
User=relay
Environment=TERRAD_P2P_LADDR=tcp://0.0.0.0:7000
Environment=TERRAD_RPC_LADDR=tcp://0.0.0.0:7001
Environment=TERRAD_GRPC_ADDRESS=127.0.0.1:7002
Environment=TERRAD_API_ADDRESS=tcp://127.0.0.1:7003
Environment=TERRAD_PPROF_LADDR=localhost:7009
Environment=TERRAD_P2P_PERSISTENT_PEERS="3ddf51347ba7c2bc4a8e1e26ee9d1cbf81034516@162.55.244.250:27656"
Environment=TERRAD_P2P_SEEDS="87048bf71526fb92d73733ba3ddb79b7a83ca11e@public-seed.terra.dev:26656,877c6b9f8fae610a4e886604399b33c0c7a985e2@terra.mainnet.seed.forbole.com:10056,6d8e943c049a80c161a889cb5fcf3d184215023e@public-seed2.terra.dev:26656,1600175a7a05f67ee0231cd94d24e7d1eb52ba53@terra-seed-mainnet.blockngine.io:26676,66aa2bac58b137e17865035567fc85f06953d4af@terra-seed01.stakers.finance:26656,058831b15272669d2de342862fc2667d5ef8be1d@seed.terra.stakewith.us:48856,af55a55a67ca0c000a7f768559ff38883d17f694@seed.terra.staker.space:26656,4df743bfcf507e603411c712d8a9b3adb5e44498@seed.terra.genesislab.net:26656,fb75d47bdcfe1924bb67d05b78d02ebf68ddc971@ec2-54-183-112-31.us-west-1.compute.amazonaws.com:26656"
Environment=TERRAD_SNAPSHOT_INTERVAL=0
Environment=TERRAD_P2P_MAX_NUM_INBOUND_PEERS=400
Environment=TERRAD_P2P_MAX_NUM_OUTBOUND_PEERS=200
LimitNOFILE=5000000
ExecStart=/usr/local/bin/terrad start --pruning custom --pruning-keep-recent 400000 --pruning-keep-every=0 --pruning-interval 100 --home --x-crisis-skip-assert-invariants
Environment=TERRAD_LOG_LEVEL=info

[Install]
WantedBy=multi-user.target
```


### Build & setup Hermes

Make the directory where you'll place the binary, clone the hermes source repository and build it using the latest release. Copy to ~/.cargo/bin & /usr/bin (or preferred directory for systemd execution)
```
mkdir -p $HOME/hermes
git clone https://github.com/informalsystems/ibc-rs.git hermes
cd hermes
git checkout v0.9.0
cargo install ibc-relayer-cli --bin hermes --locked
cp target/release/hermes $HOME/.cargo/bin
sudo cp target/release/hermes /usr/local/bin
```

Make hermes config & keys directory, copy config-template to config directory:
```
mkdir -p $HOME/.hermes
mkdir -p $HOME/.hermes/keys
cp config.toml $HOME/.hermes
```

Check hermes version & config dir setup
```
hermes version
INFO ThreadId(01) using default configuration from '/home/relay/.hermes/config.toml'
hermes 0.9.0
```

Edit hermes config (use ports according to your port config, set filter=true to filter channels you don't relay)
```
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
filter = true

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
id = 'columbus-5'
rpc_addr = 'http://127.0.0.1:7001'
grpc_addr = 'http://127.0.0.1:7002'
websocket_addr = 'ws://127.0.0.1:7001/websocket'
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
max_tx_size = 1800000
clock_drift = '15s'
max_block_time = '10s'
trusting_period = '14days'
memo_prefix = ''
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
 ['transfer', 'channel-1'], #osmosis
 ['transfer', 'channel-2'], #cosmos
 ['transfer', 'channel-16'] #secret
]

[[chains]]
id = 'osmosis-1'
rpc_addr = 'http://localhost:7011'
grpc_addr = 'http://localhost:7012'
websocket_addr = 'ws://localhost:7011/websocket'
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
  ['transfer', 'channel-72'] #terra
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
   ['transfer', 'channel-219'] #terra
 ]


[[chains]]
id = 'secret-4'
rpc_addr = 'http://127.0.0.1:7031'
grpc_addr = 'http://127.0.0.1:7032'
websocket_addr = 'ws://127.0.0.1:7031/websocket'
rpc_timeout = '10s'
account_prefix = 'secret'
key_name = 'scrt'
address_type = { derivation = 'cosmos' }
store_prefix = 'ibc'
default_gas = 2000000
max_gas = 10000000
max_msg_num = 20
max_tx_size = 2097152
gas_price = { price = 0.05, denom = 'uscrt' }
gas_adjustment = 0.1
clock_drift = '15s'
max_block_time = '8s'
trusting_period = '14days'
memo_prefix = ''
trust_threshold = { numerator = '1', denominator = '3' }
[chains.packet_filter]
policy = 'allow'
list = [
  ['transfer', 'channel-2'] #terra
]
```

Add your relaying-wallets to hermes' keyring (located in $HOME/.hermes/keys)

Best practice is to use the same mnemonic over all networks, do not use your relaying-addresses for anything else because it will lead to account sequence errors.
```
hermes keys restore columbus-5 -m "24-word mnemonic seed"
hermes keys restore osmosis-1 -m "24-word mnemonic seed"
hermes keys restore cosmoshub-4 -m "24-word mnemonic seed"
hermes keys restore secret-4 -m "24-word mnemonic seed"
```

You can validate your hermes configuration file:
```
hermes config validate
INFO ThreadId(01) using default configuration from '/home/relay/.hermes/config.toml'
Success: "validation passed successfully"
```

Refresh service files, enable hermes on system-startup, enable all node-daemons on system-startup, start node-daemons, sync, start hermes.

*Tipp: use chainlayer quicksync to bootstrap your nodes faster: https://quicksync.io/networks/osmosis.html*
```
sudo systemctl daemon-reload
sudo systemctl enable hermes.service
sudo systemctl enable terrad.service
sudo systemctl enable osmosisd.service
sudo systemctl enable gaiad.service
sudo systemctl enable secretd.service
```

```
sudo systemctl start terrad
sudo systemctl start osmosisd
sudo systemctl start gaiad
sudo systemctl start secretd
```

Watch node-daemon output to check if your nodes are syncing:
```
journaltctl -u terrad -f
```

When your nodes are fully synced you can start the hermes daemon:
```
sudo systemctl start hermes && journalctl -u hermes -f
```

Hermes does a chain-health-check at startup. Watch the output to check if all connected nodes are up and synced
```
INFO ThreadId(01) Hermes has started
INFO ThreadId(01) [columbus-5] chain is healthy
INFO ThreadId(01) [osmosis-1] chain is healthy
INFO ThreadId(01) [cosmoshub-4] chain is healthy
INFO ThreadId(01) [secret-4] chain is healthy
```

Be patient, starting all the clients & workers will take a few minutes. Watch hermes' output for successfully relayed packets or any errors.

It will try & clear any unreceived packets after startup has completed.


### Snippets

Query Hermes for unreceived packets & acknowledgements (check if channels are "clear")
```
hermes query packet unreceived-packets columbus-5 transfer channel-1
hermes query packet unreceived-acks columbus-5 transfer channel-1
```
```
hermes query packet unreceived-packets osmosis-1 transfer channel-72
hermes query packet unreceived-acks osmosis-1 transfer channel-72
```

Query Hermes for packet commitments:
```
hermes query packet commitments columbus-5 transfer channel-1
hermes query packet commitments osmosis-1 transfer channel-72
```

Clear unreceived packets manually. *Experimental: you'll need to stop your hermes daemon for it not to get confused with account sequences.*
```
hermes tx raw packet-recv osmosis-1 columbus-5 transfer channel-1
hermes tx raw packet-ack osmosis-1 columbus-5 transfer channel-1
hermes tx raw packet-recv columbus-5 osmosis-1 transfer channel-72
hermes tx raw packet-ack columbus-5 osmosis-1 transfer channel-72
```
