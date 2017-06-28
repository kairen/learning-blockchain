# 部署 Hyperledger 於 Docker
本文章將說明如何透過 Docker 部署 Hyperledger。並利用簡單的指令來進行交易。

## 事前準備
準備一台 Ubuntu Server 14.04 LTS 主機規格如下：

| Role       | RAM         | Disk            | CPUs       | IP Address |
|------------|-------------|-----------------|------------|------------|
| hyperledger| 4 GB 記憶體 | 40 GB 儲存空間  | 兩核處理器 | 172.16.1.78|

然後安裝 Docker Engine 與相關套件，透過以下指令：
```sh
$ sudo apt-get install -y python-pip git
$ curl -fsSL "https://get.docker.com/" | sh
```

接著安裝 Docker-compose，透過以下指令安裝：
```sh
$ sudo pip install --upgrade pip
$ sudo pip install docker-compose
```

## 安裝 Hyperledger
安裝前首先下載 Hyperledger compose 專案，透過以下指令：
```sh
$ git clone https://github.com/yeasy/docker-compose-files.git
$ cd docker-compose-files/hyperledger
```

執行以下步驟來下載與取代映像檔：
```sh
$ docker pull openblockchain/baseimage:0.0.9
$ docker pull yeasy/hyperledger:latest
$ docker tag yeasy/hyperledger:latest hyperledger/fabric-baseimage:latest
$ docker pull yeasy/hyperledger-peer:noops
$ docker pull yeasy/hyperledger-peer:pbft
$ docker pull yeasy/hyperledger-membersrvc:latest
```

接著執行 docker compose 來部署四個節點的 Hyperledger：
```sh
$ docker-compose up
Attaching to hyperledger_vp0_1, hyperledger_vp3_1, hyperledger_vp1_1, hyperledger_vp2_1
vp0_1  | 07:42:07.870 [crypto] main -> INFO 001 Log level recognized 'info', set to INFO
...
```

## 驗證 Hyperledger peer
進入到第一個 hyperledger 容器裡面，透過以下指令：
```sh
$ docker exec -ti hyperledger_vp0_1 bash
```

查看目前所有建立的節點：
```sh
$ peer node status
08:09:14.715 [crypto] main -> INFO 001 Log level recognized 'info', set to INFO
08:09:14.715 [logging] LoggingInit -> DEBU 002 Setting default logging level to DEBUG for command 'node'
08:09:14.715 [peer] func1 -> INFO 003 Auto detected peer address: 172.17.0.2:30303
08:09:14.716 [peer] func1 -> INFO 004 Auto detected peer address: 172.17.0.2:30303
08:09:14.716 [peer] func1 -> INFO 005 Auto detected peer address: 172.17.0.2:30303
status:STARTED
```

進入到容器後，首先部署一個 chaincode：
```sh
$ peer chaincode deploy -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 \
-c '{"Function":"init", "Args": ["kairen","100", "pingyu", "200"]}'
...
81a73fa1fabe6e385f3c609cef8915a732ee74179abde55f4ac7addf4e7c35ac4a669a7d9a17b2c9a6b3c28b45565b97dc69f4c8f53381ba13251adf5ac6d23d
```
> 上面會取得一組 Key。

首先查詢 kairen 的金錢有多少：
```sh
$ my_key="81a73fa1fabe6e385f3c609cef8915a732ee74179abde55f4ac7addf4e7c35ac4a669a7d9a17b2c9a6b3c28b45565b97dc69f4c8f53381ba13251adf5ac6d23d"
$ peer chaincode query -n ${my_key} \
-c '{"Function": "query", "Args": ["kairen"]}'
...
100
```

接著執行一個交易，我們讓 kairen 付保護費給 pingyu：
```sh
$ peer chaincode invoke -n ${my_key} \
-c '{"Function": "invoke", "Args": ["kairen", "pingyu", "10"]}'
```

確認完成交易後，可以查看 pingyu：
```sh
$ peer chaincode query -n ${my_key} \
-c '{"Function": "query", "Args": ["pingyu"]}'
```
