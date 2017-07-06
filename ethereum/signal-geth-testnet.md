# Ubuntu Geth Private Testnet
本節將說明如何透過 Ubuntu 部署 Go Ethereum。並利用簡單的指令來進行 Demo。

### 事前準備
準備一台 Ubuntu Server 14.04 LTS 主機規格如下：

| RAM         | Disk            | CPUs       |
|-------------|-----------------|------------|
| 4 GB 記憶體 | 40 GB 儲存空間  | 兩核處理器 |

### 安裝 Ethereum
若要在 Ubuntu 安裝 Ethereum 的話，可以透過以下指令快速安裝：
```sh
$ sudo apt-get install -y software-properties-common
$ sudo add-apt-repository -y ppa:ethereum/ethereum
$ sudo apt-get update && sudo apt-get install ethereum
```

完成後即可使用`geth`指令，首先我們先建立一個`custom.json`檔案來定義起源區塊(Genesis Block)，內容如下：
```json
{
	"coinbase" : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x40000",
  "extraData" : "",
  "gasLimit" : "0xffffffff",
  "nonce" : "0x0000000000000042",
  "mixhash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp" : "0x00",
  "config": {
		"chainId": 123,
		"homesteadBlock": 0,
		"eip155Block": 0,
		"eip158Block": 0
	},
	"alloc": { }
}
```
> 其中以上 Genesis block 欄位說明如下
* **nonce**：一個 64 位元雜湊(Hash)，用來證明與結合 mix-hash，即已經在此區塊上進行足夠的運算量：工作證明 (PoF)。mixhash 與 nonce 的組合必須滿足一個在 Yellowpaper, 4.3.4. Block Header Validity, (44) 數學描述的條件。它允許驗證該區塊已經被加密並開採。nonce 是加密的安全已開採驗證工作，這證明沒有任何疑點的計算特定數量已在此 Token 值被消費了。(Yellowpager, 11.5. Mining Proof-of-Work).

> * **mixhash**：一個 256 位元雜湊(hash)用來證明與結合 nonce，即已經在此區塊上進行足夠的運算量：工作證明 (PoF)。mixhash 與 nonce 的組合必須滿足一個在 Yellowpaper, 4.3.4. Block Header Validity, (44) 數學描述的條件。它允許驗證該區塊已經被加密並開採。

> * **difficulty**：此區塊相應的 nonce 檢測中應用的難度指標值。它定義了採礦的目標，這可以從先前區塊的難度水平與時間戳記來計算。難度越高，統計上需要更多的計算礦工來發現有效的區塊。此值被用來控制一個區塊鍊(Blockchain)的區塊產生時，保持在目標範圍內的區塊產生頻率。在測試網路(Testnet)，我們盡量保持低值。

> * **alloc**：允許定義一個預先存值的錢包列表。這是一個 Ethereum 特定功能，用來處理售前(Pre-sale)時期的貨幣。若是在測試網路中，由於我們很快就會拿挖到寶，所以可以不用設定。

> * **timestamp**：在這個區塊開始的一個 Unix time() 純量值。
這種機制在區塊之間的時間強制一個動態平衡。在過去的兩個區塊之間的一個較小的時間區段增加難度，因此需要額外的計算來找到下一個有效的區塊。如果時間過大，難度與預計時間會在下一個區塊被下降。時間戳記也允許被驗證區塊的順序(Yellowpaper, 4.3.4. (43))。

> * **parentHash**：整個父區塊標頭的 Keccak 256 位元雜湊(包含它的 mixhash 與 nonce)。指向到父區塊，從而有效的建立區塊的鏈。在起源區塊(Genesis block)下，只有這種情況下，它是 0。

> * **extraData**：一個自由選擇性參數，但是最大為 32 byte 長的空間，用來保存 ethernity 的智能事情。

> * **gasLimit**：一個純量值等於當前區塊鏈中每個區塊的氣體(Gas = 表示礦源？)消耗限制。在測試環境設高一點，這樣可以避免限制。P.S. 這不表示我們應該忽略該參數重要性。

上面設定完成後，就可以透過以下指令來建立：
```sh
$ geth init --datadir=data/ custom.json
$ geth --datadir data/ --networkid 123 --nodiscover --maxpeers 0 console
...
instance: Geth/v1.4.3-stable/linux/go1.5.1
coinbase: 0xf939ee9f8b3da46dc94f49b28bf920809bc1cb46
at block: 67 (Fri, 20 May 2016 19:49:36 UTC)
 datadir: data
```
> 這邊透過 custom.json 建立 blockchain，並使用 datadir 參數來儲存新建的 blockchain 狀態與數據。

當正常進入到 geth console 後，就可以建立第一個 account，並獲取位址：
```sh
> personal.newAccount();
Passphrase:
Repeat passphrase:
"0x80ae1f1579924bedc59953f13140cd6c4918b812"
```
> 要輸入密碼兩次。

接著透過鍵盤輸入```[Ctrl-D]```離開，然後執行以下指令刪除 keystore：
```sh
$ cd data
$ rm -rf `ls | grep -v keystore`
```

再次編輯```custom.json```檔，在```alloc```加入以下內容：
```json
{
    "nonce": "0xdeadbeefdeadbeef",
    "timestamp": "0x0",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "extraData": "Custem Ethereum Genesis Block",
    "gasLimit": "0x8000000",
    "difficulty": "0x400",
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "coinbase": "0x3333333333333333333333333333333333333333",
    "alloc": {
        "<Your_Address>": {
            "balance": "10000000000000000000"
        }
    }
}
```
> 這邊```<Your_Address>```為 0x80ae1f1579924bedc59953f13140cd6c4918b812。

接著再次執行 geth 指令，來透該賬戶存值：
```sh
$ geth --datadir data/ --networkid 123 --nodiscover --maxpeers 0 console
```

成功進入後，可以執行查看賬戶資訊：
```sh
> personal.listAccounts
["0x80ae1f1579924bedc59953f13140cd6c4918b812"]
```

查看 primary 賬戶的錢包狀態：
```sh
> kairen = eth.accounts[0];
"0x80ae1f1579924bedc59953f13140cd6c4918b812"

> web3.fromWei(eth.getBalance(kairen), "ether");
10
```

接著就可以進行採礦了，首先設定要開採的賬戶：
```sh
> miner.setEtherbase(kairen)
true
```

當賬戶設定完成後，就可以執行以下指令進行採礦：
```sh
> miner.start(8)
true

I0520 20:52:23.637262 miner/worker.go:555] commit new work on block 1 with 0 txs & 0 uncles. Took 252.728µs
I0520 20:52:23.637480 ethash.go:259] Generating DAG for epoch 0 (size 1073739904) (0000000000000000000000000000000000000000000000000000000000000000)
...
```

若要停止採礦可以按一次 Enter 直接輸入以下指令：
```sh
> miner.stop()
true
I0520 20:53:26.935445 miner/worker.go:337] 🔨  Mined stale block (#5 / fba716e0).
I0520 20:53:26.938490 miner/worker.go:337] 🔨  Mined stale block (#6 / d82943d3).
```

經過一段時間後，我們可以在查看一下賬戶：
```sh
> web3.fromWei(eth.getBalance(kairen), "ether");
102.34375
```
> 會發現增加了許多。

當成開採區塊後，就可以檢查賬戶的 ether balance 的數值：
```
> eth.getBalance(eth.coinbase).toNumber()
102343750000000000000
```

### 進行交易
Ethereum 有兩種類型的狀態：
* 一般或外部控制的賬戶
* 合約，即表示程式碼片段，可想成是一個類別(class)。

這兩種類型的賬戶都具有 ether balance。

首先我們先建立另一個賬戶，並查看所有賬戶列表：
```sh
> personal.newAccount();
Passphrase:
Repeat passphrase:
"0x99fa5fcf20495dac5eccebeac33917818236d2db"

> personal.listAccounts
["0x80ae1f1579924bedc59953f13140cd6c4918b812", "0x99fa5fcf20495dac5eccebeac33917818236d2db"]
```
> 其中```0x80ae1f1579924bedc59953f13140cd6c4918b812```表示 kairen，```0x99fa5fcf20495dac5eccebeac33917818236d2db```表示 pingyu。

利用變數定義與查看兩者的錢包：
```sh
> kairen = eth.accounts[0];
"0x80ae1f1579924bedc59953f13140cd6c4918b812"

> web3.fromWei(eth.getBalance(kairen), "ether");
102.34375

> pingyu = eth.accounts[1];
"0x99fa5fcf20495dac5eccebeac33917818236d2db"

> web3.fromWei(eth.getBalance(pingyu), "ether");
0
```

在移轉之前，首先要解鎖賬戶：
```sh
> personal.unlockAccount(kairen)
true
```
> 也可以直接透過以下方式輸入密碼```personal.unlockAccount(kairen, "r00tme")```。

都確認好解鎖賬戶後，就可以進行 Ether 移轉，透過以下方式進行移轉：
```sh
> eth.sendTransaction({from: kairen, to: pingyu, value: web3.toWei(100, "ether")})
I0520 21:17:20.950180 eth/api.go:1180] Tx(0x1712f84c3c22a356620f93410699f8b656098bfa6ffc3a3299ef1f57a0c5d681) to: 0x99fa5fcf20495dac5eccebeac33917818236d2db
"0x1712f84c3c22a356620f93410699f8b656098bfa6ffc3a3299ef1f57a0c5d681"
```

轉移後，就可以確認正在 Pending 的交易：
```sh
> txpool.status
{
  pending: 1,
  queued: 0
}

> eth.getBlock("pending", true).transactions
[{
    blockHash: "0x1748c022911bfff7e71b0bc7191abc3068531f5a9d57eb6031608f4ff84d1e0b",
    blockNumber: 40,
    from: "0xbe33553d18ba9d5aa477246bebb6fd05d6834793",
    gas: 90000,
    gasPrice: 21781760000,
    hash: "0x482b043cef3dd612827d722a35e383a7e326277c7df335f889e56ee969813584",
    input: "0x",
    nonce: 0,
    to: "0xa1cc1f3f979d33fca8c5543cad1613fd4834e1ef",
    transactionIndex: 0,
    value: 10000000000000000000
}]
```
> 可以看到這筆交易被存在 blockNumber 為 40 的區塊中。

這時候查看 pingyu 的賬戶，會發現依然沒有改變，So sad：
```sh
> web3.fromWei(eth.getBalance(pingyu), "ether");
0
```

這是因為該交易區塊沒有任何人協助進行運算驗證，因此狀態會一直處於 pending，這時候只要在執行以下指令即可：
```sh
> miner.start(8)
true

I0525 06:50:47.885729 miner/worker.go:555] commit new work on block 40 with 1 txs & 2 uncles. Took 861.24µs
I0525 06:50:48.261245 miner/worker.go:337] 🔨  Mined block (#40 / bbaee562). Wait 5 blocks for confirmation
```
> 經過一段時間確認交易區塊被開採完畢後，就可以透過```miner.stop()```停止。

再次觀察交易狀態，會發現已經完成交易，可以透過以下指令查詢：
```sh
> txpool.status
{
  pending: 0,
  queued: 0
}
```

也可以透過 eth web3 的 API 來查找指定區塊的資訊：
```sh
> eth.getTransactionFromBlock(40)
{
  blockHash: "0xbbaee56226a92a2c99b8981ad102bc29940d5cb4f2949559d269bd618e3930d8",
  blockNumber: 40,
  from: "0xbe33553d18ba9d5aa477246bebb6fd05d6834793",
  gas: 90000,
  gasPrice: 20000000000,
  hash: "0x482b043cef3dd612827d722a35e383a7e326277c7df335f889e56ee969813584",
  input: "0x",
  nonce: 0,
  to: "0xa1cc1f3f979d33fca8c5543cad1613fd4834e1ef",
  transactionIndex: 0,
  value: 10000000000000000000
}
```

接著透過建立一個 Java Script 函式印出所有賬戶 Balance：
```javascript
function checkAllBalances() {
    var i =0;
    eth.accounts.forEach( function(e){
        console.log("  eth.accounts["+i+"]: " +  e + " \tbalance: " + web3.fromWei(eth.getBalance(e), " ether") + "ether"); i++;
    })
};
```
> 直接輸入即可。

然後透過以下方式呼叫函式：
```sh
> checkAllBalances();
  eth.accounts[0]: 0x80ae1f1579924bedc59953f13140cd6c4918b812 	balance: 101.34375 ether
  eth.accounts[1]: 0x99fa5fcf20495dac5eccebeac33917818236d2db 	balance: 10 ether
```

### 參考連結
- [JavaScript-Console Guide](https://github.com/ethereum/go-ethereum/wiki/JavaScript-Console)
