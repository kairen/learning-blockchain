# 部署 Go Ethereum 於 Docker
本節將說明如何透過 Docker 部署 Go Ethereum。並利用簡單的指令來進行 Demo。

### 事前準備
準備能夠安裝 Docker 的主機與作業系統，這邊採用 Ubuntu Server 14.04 LTS，主機規格如下：

| RAM | Disk |CPUs |
|-----|------|-----|
| 4GB | 40GB |2vCPU|

然後安裝 Docker Engine 與相關套件，透過以下指令：
```sh
$ curl -fsSL "https://get.docker.com/" | sh
```

### 安裝 Ethereum
首先透過以下指令抓取 Go Ethereum 映像檔：
```sh
$ docker pull ethereum/client-go
```

然後部署名稱為```ethereum-1```的節點，執行以下指令來部署：
```sh
$ docker run -it -p 30303:30303 --name ethereum-1 ethereum/client-go console
I0520 10:14:55.442553 ethdb/database.go:82] Alloted 128MB cache and 1024 file handles to /root/.ethereum/chaindata
I0520 10:14:55.524803 ethdb/database.go:169] closed db:/root/.ethereum/chaindata
I0520 10:14:55.524901 cmd/utils/flags.go:601] WARNING: No etherbase set and no accounts found as default
...
```
> 完成後會直接進入 console。

在```ethereum-1```節點執行以下指令：
```sh
> admin.nodeInfo.enode
"enode://c85dc1f295407b70f2c33978c3633a317acef260d306764b625535e2272718bf8331162c073c64727f4301fec712c10e57e14db194474343abc7805bf16bdb88@[::]:30303"
```
> 這邊```[::]```可以表示為```127.0.0.1```。

確認正常執行後，接著部署```ethereum-2```節點，透過以下指令：
```sh
$ docker run -it -p 30304:30303 --name ethereum-2 ethereum/client-go console
I0520 10:15:17.050718 ethdb/database.go:82] Alloted 128MB cache and 1024 file handles to /root/.ethereum/chaindata
I0520 10:15:17.061346 ethdb/database.go:169] closed db:/root/.ethereum/chaindata
I0520 10:15:17.061481 cmd/utils/flags.go:601] WARNING: No etherbase set and no accounts found as default
...
```
> 完成後會直接進入 console。

在```ethereum-2```節點執行以下指令：
```sh
> admin.nodeInfo.enode
"enode://d09a99483f260819cd56e8faefd5fad33ea65ac8d7ba062534085c86e14b8b926bd190c67a50272195afbb432d17afc027d78551b9a7494c036bd4366d7f00ff@[::]:30303"
```
> 這邊```[::]```可以表示為```127.0.0.1```。

透過 docker 指令查詢```ethereum-1``` 網路位址：
```sh
$ docker inspect -f '{{.NetworkSettings.IPAddress}}' ethereum-1
172.17.0.2
```

接著我們要將節點串接起來，在```ethereum-2```節點執行以下指令來連接：
```sh
> admin.addPeer("enode://c85dc1f295407b70f2c33978c3633a317acef260d306764b625535e2272718bf8331162c073c64727f4301fec712c10e57e14db194474343abc7805bf16bdb88@172.17.0.2:30303")
true
```
> 這邊要注意```ethereum-1```的 IP 是否正確。

完成後在任一節點確認是否串接成功，透過以下指令查看：
```sh
> net.listening
true

> net.peerCount
1

> admin.peers
[{
    caps: ["eth/61", "eth/62", "eth/63"],
    id: "c85dc1f295407b70f2c33978c3633a317acef260d306764b625535e2272718bf8331162c073c64727f4301fec712c10e57e14db194474343abc7805bf16bdb88",
    name: "Geth/v1.4.3-stable/linux/go1.5.1",
    network: {
      localAddress: "172.17.0.3:53702",
      remoteAddress: "172.17.0.2:30303"
    },
    protocols: {
      eth: {
        difficulty: 17179869184,
        head: "d4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3",
        version: 63
      }
    }
}]
```
> 這邊可以看到 remote 與 local 兩端。

透過 admin.nodeInfo 查看節點資訊：
```sh
> admin.nodeInfo
{
  enode: "enode://ba86c458d5f1233c0046675e03318ce6b1b8c30b94b4d35b99ea6ce36c29a9d5175698c31b941ce39a2c986dc6eaf51e4d6fa4441ad8013c22c3a364e43c5fff@[::]:30303",
  id: "ba86c458d5f1233c0046675e03318ce6b1b8c30b94b4d35b99ea6ce36c29a9d5175698c31b941ce39a2c986dc6eaf51e4d6fa4441ad8013c22c3a364e43c5fff",
  ip: "::",
  listenAddr: "[::]:30303",
  name: "Geth/v1.4.3-stable/linux/go1.5.1",
  ports: {
    discovery: 30303,
    listener: 30303
  },
  protocols: {
    eth: {
      difficulty: 17179869184,
      genesis: "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3",
      head: "0xd4e56740f876aef8c010b86a40d5f56745a118d0906a34e69aec8c0db1cb8fa3",
      network: 1
    }
  }
}
```
