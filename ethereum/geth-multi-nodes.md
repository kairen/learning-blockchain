# Ethereum 多節點建置（TBD...）
當安裝完成 getch 後，可以透過以下指令建立 01 節點：
```sh
$ mkdir data-01
$ geth --datadir="data-01/" -verbosity 6 --ipcdisable --port 30301 --rpcport 8101 console 2>> data-01/01.log
instance: Geth/v1.4.3-stable/linux/go1.5.1
> admin.nodeInfo.enode
"enode://caea94be1fcabb93a7969d1bf7d4d51f76b2836ae42809d61be32da6b5879dc3da5ef53f52440ab79d0d8e7c94d5b8eb5cd9fb137576cd36e5442b2aecb53ad9@[::]:30301"
```

接著可以透過以下指令建立 02 節點：
```sh
$ mkdir data-02
$ geth --datadir="data-02" --verbosity 6 --ipcdisable --port 30302 --rpcport 8102 console 2>> data-02/02.log
instance: Geth/v1.4.3-stable/linux/go1.5.1
> admin.nodeInfo.enode
"enode://88363d54490b9565074cbdcfb68b5f4586ce3c1a82afa32ccbefcc5571d2907e2ff97017d4f23d7bda2f8e88e7df7da12bbbcdb79793775300332abdd3f55451@[::]:30302"
```

讓兩個互聯，在 02 執行以下指令：
```sh
> encode = "enode://caea94be1fcabb93a7969d1bf7d4d51f76b2836ae42809d61be32da6b5879dc3da5ef53f52440ab79d0d8e7c94d5b8eb5cd9fb137576cd36e5442b2aecb53ad9@[::]:30301"

> admin.addPeer(encode)
true

> net.listening
true

> net.peerCount
1

>  admin.peers
[{
    caps: ["eth/61", "eth/62", "eth/63"],
    id: "caea94be1fcabb93a7969d1bf7d4d51f76b2836ae42809d61be32da6b5879dc3da5ef53f52440ab79d0d8e7c94d5b8eb5cd9fb137576cd36e5442b2aecb53ad9",
    name: "Geth/v1.4.3-stable/linux/go1.5.1",
    network: {
      localAddress: "[::1]:59116",
      remoteAddress: "[::1]:30301"
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
