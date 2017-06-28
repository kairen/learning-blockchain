# Ethereum Monitor 安裝
Ethereum 提供了一個 Web-based 的監控儀表板，可以部署該儀表板，並透過 Clinet 端傳送 Ethereum 節點的資訊，來查看整個區塊鏈狀態。本節將說明如何安裝監控儀表板於 Linux 與 Docker 容器中。

這邊可以連結官方的 https://ethstats.net/ 來查看主節點網路的狀態。

###  Ubuntu Server 手動安裝
本部分說明如何手動安裝 eth-netstats 服務，其中會包含以下兩個部分：

- [Monitoring site](#monitoring-site)
- [Client side](#client-side)

#### Monitoring site
首先安裝 Browser Solidity 要使用到的相關套件：
```sh
$ sudo apt-get install -y make g++ git
```

接著安裝 node.js 平台，來建置 App：
```sh
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install nodejs
```

然後透過 git 將專案抓到 local 端，並進入目錄：
```sh
$ git clone https://github.com/cubedro/eth-netstats
$ cd eth-netstats
```

安裝相依套件與建置應用程式，並啟動服務：
```sh
$ sudo npm install
$ sudo npm install -g grunt-cli
$ grunt
$ PORT="3000" WS_SECRET="admin" npm start
```
> 接著就可以開啟 [eth-netstats](http://localhost:3000)。

> 在沒有任何 Clinet 節點連上情況下，會是一個空的網頁。

撰寫一個腳本`eth-netstats.sh`放置到背景服務執行：
```sh
#!/bin/bash
# History:
# 2016/05/22 Kyle Bai Release
#
export PORT="3000"
export WS_SECRET="admin"

echo "Starting private eth-netstats ..."
screen -dmS netstats /usr/bin/npm start
```

透過以下方式執行：
```sh
$ chmod u+x eth-netstats.sh
$ ./eth-netstats.sh
Starting private eth-netstats ...
```
> 透過`screen -x netstats`取得當前畫面。

#### Client side
首先安裝 Browser Solidity 要使用到的相關套件：
```sh
$ sudo apt-get install -y make g++ git
```

接著安裝 node.js 平台，來建置 App：
```sh
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install nodejs
```

然後透過 git 將專案抓到 local 端，並進入目錄：
```sh
$ git clone https://github.com/cubedro/eth-net-intelligence-api
$ cd eth-net-intelligence-api
```

安裝相依套件與建置應用程式：
```sh
$ sudo npm install && sudo npm install -g pm2
```

編輯`app.json`設定檔，並修改以下內容：
```sh
[
  {
    "name"        : "mynode",
    "cwd"         : ".",
    "script"      : "app.js",
    "log_date_format"   : "YYYY-MM-DD HH:mm Z",
    "merge_logs"    : false,
    "watch"       : false,
    "exec_interpreter"  : "node",
    "exec_mode"     : "fork_mode",
    "env":
    {
      "NODE_ENV"    : "production",
      "RPC_HOST"    : "localhost",
      "RPC_PORT"    : "8545",
      "INSTANCE_NAME"   : "mynode-1",
      "WS_SERVER"     : "http://localhost:3000",
      "WS_SECRET"     : "admin",
    }
  },
]
```
> * `RPC_HOST`為 ethereum 的 rpc ip address。

> * `RPC_PORT`為 ethereum 的 rpc port。

> * `INSTANCE_NAME`為 ethereum 的監控實例名稱。

> * `WS_SERVER`為 eth-netstats 的 URL。

> * `WS_SECRET`為 eth-netstats 的 secret。

確認完成後，即可啟動服務：
```sh
$ pm2 start app.json
$ sudo tail -f $HOME/.pm2/logs/mynode-out-0.log
```

### Docker 快速安裝
本部分說明如何手動安裝 eth-netstats 服務，其中會包含以下兩個部分：

- [Docker Monitoring site](#docker-monitoring-site)
- [Docker Client side](#docker-client-side)

#### Docker Monitoring site
自動建置的映像檔現在可以在 [DockerHub](https://hub.docker.com/r/kairen/ethstats/) 找到，建議直接執行以下指令來啟動 eth-netstats 容器：
```sh
$ docker run -d \
            -p 3000:3000 \
            -e WS_SECRET="admin" \
            --name ethstats \
            kairen/ethstats
```
> 接著就可以開啟 [eth-netstats](http://localhost:3000)。

> 在沒有任何 Clinet 節點連上情況下，會是一個空的網頁。

#### Docker Client side
自動建置的映像檔現在可以在 [DockerHub](https://hub.docker.com/r/kairen/ethnetintel/) 找到，也推薦透過執行以下指令來啟動 eth-netintel 容器：
```sh
$ docker run -d \
            -p 30303:30303 \
            -p 30303:30303/udp \
            -e NAME_PREFIX="geth-1" \
            -e WS_SERVER="http://172.17.1.200:3000" \
            -e WS_SECRET="admin" \
            -e RPC_HOST="172.17.1.199" \
            -e RPC_PORT="8545" \
            --name geth-1 \
            kairen/ethnetintel
```
> Monitor 與 Client 需要統一`WS_SECRET`。
