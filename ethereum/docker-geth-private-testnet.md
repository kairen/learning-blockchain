# 部署 Docker Private Testnet
本節將說明如何透過 Docker 部署 Go Ethereum private testnet。並利用簡單的指令來進行 Demo。

### 事前準備
準備能夠安裝 Docker 的主機與作業系統，這邊採用 Ubuntu Server 14.04 LTS，主機規格如下：

| RAM | Disk |CPUs |
|-----|------|-----|
| 4GB | 40GB |2vCPU|

首先在主機安裝 Docker Engine 與相關套件，可透過以下指令安裝：
```sh
$ curl http://files.imaclouds.com/scripts/docker_install.sh | sh
```

### 部署 Ethereum
目前已有本團隊已有自行建置的測試用 Docker 映像檔，目前存放在 [docker-geth](https://github.com/imac-cloud/docker-geth) 上。這邊建議採用 DockerHub 上自動建立的映像檔：
```sh
$ docker pull imaccloud/docker-geth:1.5.0
```

接著手動執行以下指令來啟動 docker-geth 容器：
```sh
$ docker run -d -p 8545:8545 -p 30303:30303 \
-e 'ETH_NETWORK_ID=123' -e 'MINER_THREADS=1' \
--name geth imaccloud/docker-geth:1.5.0
```
> 這邊可以透過```docker logs -f <ContainerID>```來查看伺服器狀態。

接著進入容器內，並透過以下指令來 attach geth：
```sh
$ docker exec -ti <ID> bash
$ geth attach ipc:ethdata/geth.ipc
```
