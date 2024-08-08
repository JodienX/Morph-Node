## Menjalankan Validator Node Morph

Panduan ini menjelaskan pendekatan untuk menjalankan node validator Morph. Jika Anda tidak terbiasa dengan tugas validator, silakan lihat desain [optimistic zkEVM](https://docs.morphl2.io/docs/how-morph-works/optimistic-zkevm).

Buat folder `~/.morph` sebagai direktori awal untuk contoh ini.

## Bangun binary yang dapat dieksekusi

### Clone Morph

```bash
mkdir -p ~/.morph
cd ~/.morph
git clone https://github.com/morph-l2/morph.git
```
Saat ini, kami menggunakan tag v0.1.0-beta sebagai geth versi beta kami.

```bash
cd morph
git checkout v0.1.0-beta
```
### Bangun Geth

Perhatian: Anda memerlukan C compiler untuk membangun geth

```bash
make nccc_geth
```

### Build Node

```bash
cd ~/.morph/morph/node
make build
```

## Sinkronkan dari blok genesis
### Persiapan Konfigurasi

1. Unduh file konfigurasi dan buat direktori data

```bash
cd ~/.morph
wget https://raw.githubusercontent.com/morph-l2/config-template/main/holesky/data.zip
unzip data.zip
```

2. Buat secret bersama dengan node

```bash
cd ~/.morph
openssl rand -hex 32 > jwt-secret.txt
```

## Skrip untuk mulai proses

### Geth

```bash

NETWORK_ID=2810

nohup ./morph/go-ethereum/build/bin/geth \
--datadir=./geth-data \
--verbosity=3 \
--http \
--http.corsdomain="*" \
--http.vhosts="*" \
--http.addr=0.0.0.0 \
--http.port=8545 \
--http.api=web3,eth,txpool,net,engine \
--ws \
--ws.addr=0.0.0.0 \
--ws.port=8546 \
--ws.origins="*" \
--ws.api=web3,eth,txpool,net,engine \
--networkid=$NETWORK_ID \
--authrpc.addr="0.0.0.0" \
--authrpc.port="8551" \
--authrpc.vhosts="*" \
--authrpc.jwtsecret=$JWT_SECRET_PATH \
--gcmode=archive \
--metrics \
--metrics.addr=0.0.0.0 \
--metrics.port=6060 \
--miner.gasprice="100000000"
```

tail -f geth.log untuk memeriksa apakah Geth berjalan dengan benar, atau Anda juga dapat menjalankan perintah curl di bawah ini untuk memeriksa apakah Anda terhubung ke peer.

```bash
curl --lokasi --permintaan POST 'localhost:8545/' \
--header 'Jenis-Konten: aplikasi/json' \
--data-raw '{
"jsonrpc":"2.0",
"metode":"eth_blockNumber",
"id":1
}'

{"jsonrpc":"2.0","id":1,"hasil":"0x148e39"}
```

### Node

```bash
cd ~/.morph
ekspor L1MessageQueueWithGasPriceOracle=0x778d1d9a4d8b6b9ade36d967a9ac19455ec3fd0b
ekspor START_HEIGHT=1434640
ekspor Rollup=0xd8c5c541d56f59d65cf775de928ccf4a47d4985c
./morph/node/build/bin/morphnode --validator --home ./node-data \
--l2.jwt-secret ./jwt-secret.txt \
--l2.eth http://localhost:8545 \
--l2.engine http://localhost:8551 \
--l1.rpc $(Ethereum Holesky RPC) \
--l1.beaconrpc $(Ethereum Holesky beacon chain RPC) \
--l1.chain-id 17000 \
--validator.privateKey $(Kunci Validator Anda) \
--sync.depositContractAddr $(L1MessageQueueWithGasPriceOracle) \
--sync.startHeight $(START_HEIGHT) \
--derivation.rollupAddress $(Rollup) \
--derivation.startHeight $(START_HEIGHT) \
--derivation.fetchBlockRange 200 \
--log.filename ./node.log
```
## Periksa Status

Jika node Anda berhasil, Anda akan melihat respons berikut:

```bash
I[2024-06-06|15:57:35.216] metrics server enabled module=derivation host=0.0.0.0 port=26660
derivation node starting
ID> 24-06-06|15:57:35.216] initial sync start module=syncer msg="Menjalankan sinkronisasi awal pesan L1 sebelum memulai sequencer, ini mungkin memerlukan waktu sementara..."
I[2024-06-06|15:57:35.242] sinkronisasi awal selesai modul=syncer latestSyncedBlock=1681622
I[2024-06-06|15:57:35.242] derivasi mulai tarik rollupData formulir l1 modul=derivasi startBlock=1681599 akhir=1681622
I[2024-06-06|15:57:35.244] mengambil rollup tx modul=derivasi txNum=8 latestBatchIndex=59201
I[2024-06-06|15:57:35.315] mengambil transaksi rollup berhasil modul=derivasi txNonce=8764 txHash=0x5fb8a98472d1be73be2bc6be0807b9e0c68b7ba14a648c8a17bdaff7b26eb923 l1BlockNumber=1681599 firstL2BlockNumber=1347115 lastL2BlockNumber=1347129
I[2024-06-06|15:57:35.669] new l2 block success module=derivation blockNumber=1347115
```

Anda dapat menggunakan perintah berikut untuk memeriksa tinggi blok terbaru guna memastikan Anda selaras.

```bash
curl --location --request POST 'localhost:8545/' \
--header 'Content-Type: application/json' \
--data-raw '{
"jsonrpc":"2.0",
"method":"eth_blockNumber",
"id":1
}'
{"jsonrpc":"2.0","id":1,"result":"0x148e39"}
```

Pastikan Anda memeriksa status validator secara terus-menerus, jika Anda menemukan respons

```bash
[2024-06-14|16:43:50.904] hash root atau hash penarikan tidak sama originStateRootHash=0x13f91d1c272e48e2d864ce7bfb421506d5b2a04def64d45c75391cdcdd69cd78 deriveStateRootHash=0x27e10420c0e34676a7d75c4189d7ccd1c3407cc8fd0b3eafb01c15e250a1215f batchWithdrawalRoot=0xa3e4a7cf45c7591a6bd9868f1fa7485ae345f10067acaade5f5b07d418b2e172 deriveWithdrawalRoot=0xa3e4a7cf45c7591a6bd9868f1fa7485ae345f10067acaade5f5b07d418b2e172
```

Ini berarti validator Anda menemukan ketidakkonsistenan antara penyerahan sequencer dan pengamatan Anda sendiri.
