This section contains instructions on how to setup the rust relayer "hermes" and relay IBC packets between bitsong-2b and other IBC-enabled networks.


## official Bitsong IBC Channels

| source chain-id  | source channel  | source denom | destination chain-id  | destinaion channel | IBC token-address on destinaion chain |
| ---------------- | --------------- | ------------ | --------------------- | ------------------ | ------------------------------------------- |
| bitsong-2b | channel-0 | ubtsg | osmosis-1  | channel-73  | IBC/4E5444C35610CC76FC94E7F7886B93121175C28262DDFDDE6F84E82BF2425452 |
| bitsong-2b | channel-1 | ubtsg | cosmoshub-4 | channel-229 | IBC/E7D5E9D0E9BF8B7354929A817DD28D4D017E745F638954764AA88522A7A409EC |
| osmosis-1 | channel-73 | uosmo | bitsong-2b  | channel-0  | IBC/ED07A3391A112B175915CD8FAF43A2DA8E4790EDE12566649D0C2F97716B8518 |
| cosmoshub-4 | channel-229 | uatom | bitsong-2b | channel-1 | IBC/C4CFF46FD6DE35CA4CF4CE031E643C8FDC9BA4B99AE598E9B0ED98FE3A2319F9 |


## Relayer Tutorial

To assist operators in setting up relayers, Bitsong provides tutorials for the following IBC relayers:

### Hermes (rust) https://hermes.informal.systems/

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
address = "tcp://0.0.0.0:7011"
```
```
[grpc]

# Enable defines if the gRPC server should be enabled.
enable = true

# Address defines the gRPC server address to bind to.
address = "0.0.0.0:7012"
```

*config.toml - choose unique pprof_laddr port*
```
# pprof listen address (https://golang.org/pkg/net/http/pprof)
pprof_laddr = "localhost:7019"
```
```
[rpc]

# TCP or UNIX socket address for the RPC server to listen on
laddr = "tcp://127.0.0.1:7013"
```
```
[p2p]

# Address to listen for incoming connections
laddr = "tcp://0.0.0.0:7010"
```

*config.toml - set persistent-peers & seeds for each chain*
```
bitsong-2b seeds: "ffa27441ca78a5d41a36f6d505b67a145fd54d8a@95.217.156.228:26656,efd52c1e56b460b1f37d73c8d2bd5f860b41d2ba@65.21.62.83:26656"
```
```
bitsong-2b persistent-peers: "a62038142844828483dbf16fa6dd159f6857c81b@173.212.247.98:26656,e9fea0509b1a2d16a10ef9fdea0a4e3edc7ca485@185.144.83.158:26656,8208adac8b09f3e2499dfaef24bb89a2d190a7a3@164.68.109.246:26656,cf031ac1cf44c9c311b5967712899391a434da9a@161.97.97.61:26656,d6b2ae82c38927fa7b7630346bd84772e632983a@157.90.95.104:15631,a5885669c1f7860bfe28071a7ec00cc45b2fcbc3@144.91.85.56:26656,325a5920a614e2375fea90f8a08d8b8d612fdd1e@137.74.18.30:26656,ae2787a337c3599b16410f3ac09d6918da2e5c37@46.101.238.149:26656,9336f75cd99ff6e5cdb6335e8d1a2c91b81d84b9@65.21.0.232:26656,9c6e52e78f112a55146b09110d1d1be47702df27@135.181.211.184:36656"
```
```
osmosis-1 seeds: "83adaa38d1c15450056050fd4c9763fcc7e02e2c@ec2-44-234-84-104.us-west-2.compute.amazonaws.com:26656,23142ab5d94ad7fa3433a889dcd3c6bb6d5f247d@95.217.193.163:26656,f82d1a360dc92d4e74fdc2c8e32f4239e59aebdf@95.217.121.243:26656,e437756a853061cc6f1639c2ac997d9f7e84be67@144.76.183.180:26656,f515a8599b40f0e84dfad935ba414674ab11a668@osmosis.blockpane.com:26656"
```
```
osmosis-1 persistent-peers: "147d0fe101bbd9e200ccbe3d353d5e7762cb02ee@207.154.201.8:26656, 9f77af7811da143f339402394ee71e42d5e2fe61@46.101.171.174:26656, d518832e4ded0484183fef3509d9f23ebb70b528@46.101.202.54:26656, 8f67a2fcdd7ade970b1983bf1697111d35dfdd6f@52.79.199.137:26656, 00c328a33578466c711874ec5ee7ada75951f99a@35.82.201.64:26656, cfb6f2d686014135d4a6034aa6645abd0020cac6@52.79.88.57:26656, 8d9967d5f865c68f6fe2630c0f725b0363554e77@134.255.252.173:26656, 785bc83577e3980545bac051de8f57a9fd82695f@194.233.164.146:26656, 778fdedf6effe996f039f22901a3360bc838b52e@161.97.187.189:36657, 64d36f3a186a113c02db0cf7c588c7c85d946b5b@209.97.132.170:26656, 4d9ac3510d9f5cfc975a28eb2a7b8da866f7bc47@37.187.38.191:26656, 2115945f074ddb038de5d835e287fa03e32f0628@95.217.43.85:26656"
```
```
cosmoshub-4 seeds: "bf8328b66dceb4987e5cd94430af66045e59899f@public-seed.cosmos.vitwit.com:26656,cfd785a4224c7940e9a10f6c1ab24c343e923bec@164.68.107.188:26656,d72b3011ed46d783e369fdf8ae2055b99a1e5074@173.249.50.25:26656,ba3bacc714817218562f743178228f23678b2873@public-seed-node.cosmoshub.certus.one:26656,3c7cad4154967a294b3ba1cc752e40e8779640ad@84.201.128.115:26656"
```
```
cosmoshub-4 persistent-peers: "ee27245d88c632a556cf72cc7f3587380c09b469@45.79.249.253:26656,538ebe0086f0f5e9ca922dae0462cc87e22f0a50@34.122.34.67:26656,d3209b9f88eec64f10555a11ecbf797bb0fa29f4@34.125.169.233:26656,bdc2c3d410ca7731411b7e46a252012323fbbf37@34.83.209.166:26656,585794737e6b318957088e645e17c0669f3b11fc@54.160.123.34:26656,11dfe200894f38e411beca77928e9dd118e66813@94.130.98.157:26656"
```

please reference https://github.com/cosmos/chain-registry for a maintained list of peers & seeds.
