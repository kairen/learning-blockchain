# 部署 Docker Private Testnet
本節將說明如何透過 Docker 部署 Go Ethereum private testnet。並利用簡單的指令來進行 Demo。

### 事前準備
準備能夠安裝 Docker 的主機與作業系統，這邊採用 Ubuntu Server 14.04 LTS，主機規格如下：

| RAM | Disk |CPUs |
|-----|------|-----|
| 4GB | 40GB |2vCPU|

首先在主機安裝 Docker Engine 與相關套件，可透過以下指令安裝：
```sh
$ curl -fsSL "https://get.docker.com/" | sh
```

### 部署 Ethereum
目前 Go Ethereum 已經有自己的 Docker image，故這邊只需要執行以下指令抓取：
```sh
$ docker pull ethereum/client-go
```

這邊也有撰寫好腳本，可以透過以下方式來建立測試網路：

接著手動執行以下指令來啟動 docker-geth 容器：
```sh
$ git clone https://github.com/kairen/kubereum.git
$ cd kubereum/docker
$ make init && make run_node
```
> 這邊可以透過```docker logs -f <ContainerID>```來查看伺服器狀態。

接著進入容器內，並透過以下指令來 attach geth：
```sh
$ make console
```
