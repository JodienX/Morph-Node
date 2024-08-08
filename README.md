# Morph-Node
Tutorial Menjalankan Node MorphL2

Panduan ini menguraikan langkah-langkah untuk memulai node Morph. Contoh ini mengasumsikan direktori awal adalah `~/.morph`

### Persyaratan perangkat keras

Menjalankan node morph memerlukan 2 proses: L`geth` dan `node`.

- `Geth`: lapisan eksekusi Morph yang perlu memenuhi [persyaratan perangkat keras go-ethereum](https://github.com/ethereum/go-ethereum#hardware-requirements), tetapi dengan penyimpanan yang lebih sedikit, 500 GB sudah cukup sejauh ini.

- `Node`: lapisan konsensus Morph yang tertanam di tendermint yang perlu memenuhi [persyaratan perangkat keras tendermint](https://docs.tendermint.com/v0.34/tendermint-core/running-in-production.html#processor-and-memory).

::: tip
Karena keterbatasan dalam implementasi geth saat ini, hanya mode arsip yang didukung, yang berarti ukuran penyimpanan akan terus bertambah dengan blok yang diproduksi.
:::

### Bangun biner yang dapat dieksekusi

#### Clone morph

```
mkdir -p ~/.morph
cd ~/.morph
git clone https://github.com/morph-l2/morph.git
```

Saat ini, kami menggunakan tag v0.1.0-beta sebagai versi beta kami.

```
cd morph
git checkout v0.1.0-beta
```

#### Build Geth

Perhatian: Anda memerlukan C compiler untuk Build geth

```
make nccc_geth
```

#### Build Node

```
cd ~/.morph/morph/node
make build
```

### Sinkronkan dari blok genesis

#### Persiapan Konfigurasi

Unduh file konfigurasi dan buat direktori data

```
cd ~/.morph
wget https://raw.githubusercontent.com/morph-l2/config-template/main/holesky/data.zip
unzip data.zip
```

Buat secret bersama dengan node

```
cd ~/.morph
openssl rand -hex 32 > jwt-secret.txt
```

#### Skrip untuk memulai proses

*Geth*

```
./morph/go-ethereum/build/bin/geth --morph-holesky \
--datadir "./geth-data" \
--http --http.api=web3,debug,eth,txpool,net,engine \
--authrpc.addr localhost \
--authrpc.vhosts="localhost" \
--authrpc.port 8551 \
--authrpc.jwtsecret=./jwt-secret.txt \
--miner.gasprice="100000000" \
--log.filename=./geth.log
```

*tail -f geth.log* untuk memeriksa apakah Geth berjalan dengan benar, atau Anda juga dapat menjalankan perintah curl di bawah ini untuk memeriksa apakah Anda terhubung ke peer.

```
curl -X POST -H 'Content-Type: application/json' --data
'{"jsonrpc":"2.0","method":"net_peerCount","params":[],"id":74}'
localhost:8545

{"jsonrpc":"2.0","id":74,"result":"0x3"}
```

*Node*

```
./morph/node/build/bin/morphnode --home ./node-data \
--l2.jwt-secret ./jwt-secret.txt \
--l2.eth http://localhost:8545 \
--l2.engine http://localhost:8551 \
--log.filename ./node.log
```

tail -f node.log untuk memeriksa apakah node berjalan dengan benar, dan Anda juga dapat menjalankan perintah curl untuk memeriksa status koneksi node Anda.

```
curl http://localhost:26657/net_info

{
"jsonrpc": "2.0",
"id": -1,
"result": {
"listening": true,
"listeners": [
"Listener(@)"
],
"n_peers": "3",
"peers": [
{
"node_info": {
"protocol_version": {
"p2p": "8",
"block": "11",
"app": "0"
},
"id": "0fb5ce425197a462a66de015ee5fbbf103835b8a",
"listen_addr": "tcp://0.0.0.0:26656",
"network": "chain-morph-holesky",
"version": "0.37.0-alpha.1",
"channels": "4020212223386061",
"moniker": "morph-dataseed-node-1",
"other": {
"tx_index": "on",
"rpc_address": "tcp://0.0.0.0:26657"
}
},
"is_outbound": true,
```

### Periksa status sinkronisasi

curl http://localhost:26657/status untuk memeriksa status sinkronisasi node

```
{
"jsonrpc": "2.0",
"id": -1,
"result": {
"node_info": {
"protocol_version": {
"p2p": "8",
"block": "11",
"app": "0"
},
"id": "b3f34dc2ce9c4fee5449426992941aee1e09670f",
"listen_addr": "tcp://0.0.0.0:26656",
"network": "chain-morph-holesky",
"version": "0.37.0-alpha.1",
"channels": "4020212223386061",
"moniker": "my-morph-node",
"other": {
"tx_index": "on",
"rpc_address": "tcp://0.0.0.0:26657"
}
},
"sync_info": {
"latest_block_hash": "71024385DDBEB7B554DB11FD2AE097ECBD99B2AF826C11B2A74F7172F2DEE5D2",
"hash_aplikasi_terbaru": "",
"ketinggian_blok_terbaru": "2992",
"waktu_blok_terbaru": "2024-04-25T13:48:27.647889852Z",
"hash_blok_terawal": "C7A73D3907C6CA34B9DFA043FC6D4529A8EAEC8F059E100055653E46E63F6F8E",
"hash_aplikasi_terawal": "",
"ketinggian_blok_terawal": "1",
"waktu_blok_terawal": "2024-04-25T09:06:30Z",
"catching_up": false
},
"validator_info": {
"address": "5FB3D3734640792F14B70E7A53FBBD39DB9787A8",
"pub_key": {
"type": "tendermint/PubKeyEd25519",
"value": "rzN67ZJWsaLSGGpNj7HOWs8nrL5kr1n+w0OckWUCetw="
},
"voting_power": "0"
}
}
}
```

"catching_up" yang dikembalikan menunjukkan apakah node tersebut sinkron atau tidak. True berarti node tersebut sinkron. Sementara itu, latest_block_height yang dikembalikan menunjukkan tinggi blok terbaru ini
