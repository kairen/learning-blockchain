# Browser Solidity
Browser Solidity 是一個 Web-based 的 Solidity 編譯器與 IDE。本節將說明如何安裝於 Linux 與 Docker 中。

這邊可以連結官方的 https://ethereum.github.io/browser-solidity 來使用; 該網站會是該專案的最新版本預覽。

###  Ubuntu Server 手動安裝
首先安裝 Browser Solidity 要使用到的相關套件：
```sh
$ sudo apt-get install -y apache2 make g++ git
```

接著安裝 node.js 平台，來建置 App：
```sh
$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
$ sudo apt-get install nodejs
```

然後透過 git 將專案抓到 local 端，並進入目錄：
```sh
$ git clone https://github.com/ethereum/browser-solidity.git
$ cd browser-solidity
```

安裝相依套件與建置應用程式：
```sh
$ sudo npm install
$ sudo npm run build
```

完成後，將所以有目錄的資料夾與檔案搬移到 Apache HTTP Server 的網頁根目錄：
```sh
$ sudo cp ./* /var/www/html/
```
> 完成後就可以開啟網頁了。

### Docker 快速安裝
本節將說明如何透過抓取線上的映像檔來快速執行 Dashboard。只要透過以下指令既可以進行下載映像檔與佈屬：
```sh
$ docker run -d \
            -p 80:80 \
            --name solidity \
            kairen/solidity
```

![](images/snapshot-dash.png)
