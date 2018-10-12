# Hyperledger-Fabric Chaincode
## CouchDB
### CouchDB分類

![NoSQL_DB][1]

----------

### CAP特性

![CAP][2]

----------

 - **文件儲存**

每個文件有一個唯一的id但是沒有必須的文件schema。

 - **ACID語意**

其通過多版本並行控制的形式來實現，處理大量並行讀寫不會產生衝突。

 - **Map/Reduce 視圖 和 索引**

儲存的資料通過視圖進行組裝。能夠對視圖進行索引。

 - **支援複製的分散式架構**

支援雙向的複製（同步）和離線操作。

 - **REST API**

使用HTTP方法 POST，GET，PUT和DELETE來操作所有的資源。

 - **最終一致性**

保證最終一致性，使其能夠同時提供可用性和分割容忍。

 - **離線支援**

能夠同步複製到可能會離線的終端裝置，再次在線時處理資料同步。

[CouchDB Admin][3]

----------

## Go Lang

 - a programming language created in 2009 by Google 
 - a statically typed, compiled language in the tradition of C 
 - with memory safety, garbage collection, structural typing, and CSP-style concurrency.
 - The compiler, tools, and source code are all free and open source. 
 - Go was awarded TIOBE programming language of the year 2016.

[《学习GO语言》][4]

----------

## Chaincode
**What is Chaincode**

 - 一段由程式語言編寫，並能實現預定義介面的程式。
 - 運行在一個受保護的Docker容器中，與背書節點的運行互相隔離。
 - 可通過應用提交的交易對帳本狀態初始化並進行管理。
 - 處理由網路中的成員一致認可的業務邏輯（智慧合約）
 - 創建的（帳本）狀態是與其他chaincode互相隔離，不能被其他chaincode直接訪問

----------

### Chaincode Lifecycle

![ChainCode Lifecycle][5]
----------

### Chaincode for Development
[Chaincode for develop][6]

 - Go
 - node.js
 - Java, will be supported
 
 [basic.go][7]
 
----------

### SHIM ChaincodeStubInterface
**獲得調用的參數**

 - GetArgs() [][]byte 
 
	以byte數組的數組的形式獲得傳入的參數列表

 - GetStringArgs() []string 
 
	以字符串數組的形式獲得傳入的參數列表

 - GetFunctionAndParameters() (string, []string) 
 
	將字符串數組的參數分為兩部分，數組第一個字是Function，剩下的都是Parameter

 - GetArgsSlice() ([]byte, error) 
 
	以byte切片的形式獲得參數列表

[SHIM][8]

----------

### SHIM Library
**增刪改查State DB**

 - PutState(key string, value []byte) error 
 - GetState(key string)([]byte, error) 
 - DelState(key string) error

**復合鍵的處理 (ref: mable02 example)**

 - CreateCompositeKey(objectType string, attributes []string) (string, error)
 
*用U+0000分割各個字段*

 - SplitCompositeKey(compositeKey string) (string, []string, error)
 
*用U+0000 Split開，結果第一個是objectType，其他是復合鍵用到的列的值。*

 - GetStateByPartialCompositeKey(objectType string, keys []string) (StateQueryIteratorInterface, error)
 
*雖然是部分復合鍵的查詢，但是不允許拿後面部分的復合鍵進行匹配，必須是前面部分*

**獲得當前用戶GetCreator() ([]byte, error)**

獲得調用這個ChainCode的客戶端的用戶證書，雖然返回的是byte數組，但其實是字符串

```
-----BEGIN CERTIFICATE----- 
MIICGjCCAcCgAwIBAgIRAMVe0+QZL+67Q+R2RmqsD90wCgYIKoZIzj0EAwIwczEL 
MAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNhbiBG 
cmFuY2lzY28xGTAXBgNVBAoTEG9yZzEuZXhhbXBsZS5jb20xHDAaBgNVBAMTE2Nh 
Lm9yZzEuZXhhbXBsZS5jb20wHhcNMTcwODEyMTYyNTU1WhcNMjcwODEwMTYyNTU1 
WjBbMQswCQYDVQQGEwJVUzETMBEGA1UECBMKQ2FsaWZvcm5pYTEWMBQGA1UEBxMN 
U2FuIEZyYW5jaXNjbzEfMB0GA1UEAwwWVXNlcjFAb3JnMS5leGFtcGxlLmNvbTBZ 
MBMGByqGSM49AgEGCCqGSM49AwEHA0IABN7WqfFwWWKynl9SI87byp0SZO6QU1hT 
JRatYysXX5MJJRzvvVsSTsUzQh5jmgwkPbFcvk/x4W8lj5d2Tohff+WjTTBLMA4G 
A1UdDwEB/wQEAwIHgDAMBgNVHRMBAf8EAjAAMCsGA1UdIwQkMCKAIO2os1zK9BKe 
Lb4P8lZOFU+3c0S5+jHnEILFWx2gNoLkMAoGCCqGSM49BAMCA0gAMEUCIQDAIDHK 
gPZsgZjzNTkJgglZ7VgJLVFOuHgKWT9GbzhwBgIgE2YWoDpG0HuhB66UzlA+6QzJ 
+jvM0tOVZuWyUIVmwBM= 
-----END CERTIFICATE-----
```
常見需求是在ChainCode中獲得當前用戶的信息，方便進行權限管理。
把證書字符串轉換為Certificate Object，通過Subject獲得當前用戶的名字。

**高級查詢**

 - GetStateByRange(startKey, endKey string) (StateQueryIteratorInterface, error)

 - GetQueryResult(query string) (StateQueryIteratorInterface, error)
 
    富查詢CouchDB使用Mango查詢

 - GetHistoryForKey(key string) (HistoryQueryIteratorInterface, error)
 
    對同一個數據（也就是Key相同）的更改

**調用別的鏈代碼**
 
- InvokeChaincode(chaincodeName string, args [][]byte, channel string) pb.Response

----------

### Sample: Fabcar
**Car** 
 - Make 
 - Model 
 - Colour 
 - Owner

**Methods**
 - queryCar() 
 - initLedger() 
 - createCar() 
 - queryAllCars() 
 - changeCarOwner()

[Fabcar][9]

----------

### Sample: Marble02
marble
	ObjectType
	Name      
	Color     
	Size      
	Owner 
	
initMarble()
transferMarble()
transferMarblesBasedOnColor()
delete()
readMarble()
queryMarblesByOwner()
queryMarbles()
getHistoryForMarble()
getMarblesByRange()

[Marble02][10]

----------

### Sample: Auction

![AuctionFlow1][11]
[Auction][12]

**Query**
```
GetItem
GetUser
GetAuctionRequest
GetTransaction
GetBid
GetLastBid
GetHighestBid
GetNoOfBidsReceived
GetListOfBids
GetItemLog
GetItemListByCat
GetUserListByCat
GetListOfInitAucs
GetListOfOpenAucs
ValidateItemOwnership
```
**Invoke**
```
PostItem
PostUser
PostAuctionRequest
PostTransaction
PostBid
OpenAuctionForBids
BuyItNow
TransferItem
CloseAuction
CloseOpenAuctions
```
![AuctionFlow2][13]
----------

## Transaction

![此处输入图片的描述][14]

### Transaction Flow

![dataflow][15]


  [1]: http://2.bp.blogspot.com/-sO1yDmK_yT8/U2NC8WyonCI/AAAAAAAAeHA/psRndXmCIvM/s1600/NoSQL_DB_MongoDB.png
  [2]: http://1.bp.blogspot.com/-D1TvAfHOYJU/UDRMyKOiT9I/AAAAAAAACGw/wJNSyCfr0Yg/s640/0_1315316512jhTH.gif
  [3]: http://172.18.136.123:5984/_utils/#/database/mychannel_fabcar/_all_docs
  [4]: https://mikespook.com/learning-go/
  [5]: ./CC_Lifecycle.jpg
  [6]: https://hyperledger-fabric.readthedocs.io/en/release-1.1/chaincode4ade.html#chaincode-api
  [7]: ./basic.go
  [8]: https://godoc.org/github.com/hyperledger/fabric/core/chaincode/shim#Chaincode
  [9]: https://github.com/hyperledger/fabric-samples/blob/release-1.1/chaincode/fabcar/go/fabcar.go
  [10]: https://github.com/hyperledger/fabric-samples/blob/release-1.1/chaincode/marbles02/go/marbles_chaincode.go
  [11]: AuctionFlow1.png
  [12]: https://github.com/ITPeople-Blockchain/auction/tree/v1.1.0/art/artchaincode
  [13]: AuctionFlow2.png
  [14]: dataTx.png
  [15]: dataflow.png