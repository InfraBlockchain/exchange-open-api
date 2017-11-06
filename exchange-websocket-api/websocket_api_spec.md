ModernArt Websocket API spec
===
(revision 20171106)

#### **_akka-actor-based event-driven exchange server cluster implementation_**

ModernArt Websocket Server Location
---

* test server : [ws://alphama.artstockx.com/ws/ETHEREUM_ADDRESS](ws://alphama.artstockx.com/ws/ETHEREUM_ADDRESS)


Websocket Test Page
---

* test server : [**http://alphama.artstockx.com/websocketTest/ETHEREUM_ADDRESS**](http://alphama.artstockx.com/websocketTest/ETHEREUM_ADDRESS)
* eg) [http://alphama.artstockx.com/websocketTest/0x38df6b04c455a7512a71a12f4f03f7bead9774e8](http://alphama.artstockx.com/websocketTest/0x38df6b04c455a7512a71a12f4f03f7bead9774e8)

Authentication
---
* **Same Origin Check** to protect ["Cross-Site WebSocket Hijacking"](http://www.christian-schneider.net/CrossSiteWebSocketHijacking.html)  
  allowed 'Origin' for test server => **"localhost"** , **"websocket.org"**  
* currently no authentication for Ethereum account ownership
* only check whether Ethereum account address string is in valid address format
* example of websocket connection url for a specific Ethereum account  
  **ws://alphama.artstockx.com/ws/0x38df6b04c455a7512a71a12f4f03f7bead9774e8**

JSON-RPC-style ModernArt Websocket API message convention
---

> Request Message Format (Client -> Server)

```json
{
  "t" : "OB",
  "rqI" : "183746",
  "m" : {
    "param_1" : "0x2343...", 
    "param_2" : "AS_PC_GN"
  }
}
```

| field | description |
| :---: | --- |
| **t** | Message Type (API ID) |
| **rqI** | Request ID, API client can generate unique request id to identify each API call |
| **m** | API request parameter json message |
| param_1, ... | API specific parameter field |

if an invalid json string is sent to server, websocket session shall be disconnected immediately.

> Response Message Format (Server -> Client)

```json
{
  "t" : "OBR",
  "rqI" : "183746",
  "r" : {
    "res" : "OK",
    "res_field_1" : "abcd",
    "res_field_2" : 1234,
    "error" : "err msg ..." 
  }
}
```

| field | description |
| :---: | --- |
| **t** | (Message Type) + 'R' (Response) |
| **rqI** | Request ID, this json is the response message for the API call request having the same request ID |
| **r** | response json message |
| **res** | result code, 'OK' means successful API call |
| **error** | for 'error' result case ('res' != 'OK'), this field contains error message string or json |
| res_field_1, ... | API specific data fields |

Common Websocket Messages
---

#### Websocket Disconnect Request
```json
{ "t" : "Disconnect" }
```

#### Unrecognized Message Type response
```json
{
  "t" : "UnrecognizedMessageType",
  "sentType" : "QWEFW"
}
```

### BadRequest response
```json
{
  "t" : "OBR",
  "rqI" : "283462",
  "r" : {
    "res" : "BadRequest",
    "error" : "error messages ...."
  }
}
```

Exchange Transaction APIs
---

### PlaceBuyOrder API \[OB\] (OrderBuy)

> WS Request Message

```json
{  
  "t" : "OB",
  "rqI" : "asdfwef",
  "m" : {  
    "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
    "tt" : "OB",
    "ea" : "0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
    "sy" : "AS_PC_GN",
    "am" : "50",
    "pr" : "52100000",
    "mfr" : 10,
    "tfr" : 20,
    "ts" : 1504795445476,
    "si" : "0xd88776a935a36b81f20957741521a913b2f1cba101c352ea4bf7d6e91d98c44a7fcb06cce53fca1cecd7cbacc32696db78908f82bc6c5b3a6ff0759e77c54e381c"
   }
}
```

| field | description |
| :---: | --- |
| **xa**​ | exchange Ethereum smart contract address (Exchange Vault) identifying the exchange server |
| **tt**​ | transaction type (‘OB’ : Order Buy, ‘OS’ : Order Sell, ‘TB’ : Trade-Buy, ‘TS’ : Trade-Sell, ‘CB’ : Cancel-Buy, ‘CS’ : Cancel-Sell, ‘WR’ : Withdrawal-Request, ‘CWR’ : Cancel-Withdrawal-Request, ‘WC’ : Withdrawal-Confirm) |
| **ea​** | user’s Ethereum account address |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **am**​ | Asset-Share token amount |
| **pr​** | dUSD price for 1 Asset-Share (6 decimals big integer) |
| **mfr**​ | maker fee rate (unit in percentage(0-100) multiplied by 100 (0(0%)-10000(100%), ex: 102 = 1.02%, 10 = 0.1%) |
| **tfr**​ | taker fee rate (same unit as ‘mfr’) |
| **ts** | client timestamp at which transaction message is made |
| **si**​ | crypto signature signed by the user’s Ethereum account for transaction message (tightly packed (no whitespace) stringified json object) |


> WS Response Message

```json
{
   "t" : "OBR",
   "rqI" : "asdfwef",
   "r" : {
      "res" : "OK",
      "bI" : "QmdChdLRoCQi5BjfSG3QXvgHW5rLAGsqUJVcpAbX3QZ9Sp",
      "uf" : "30"
   }
}
```

| field | description |
| :---: | --- |
| **bI** | buy order id (IPFS hash address of buy order message json file) |
| **uf** | unfilled token amount |

Response codes (res)
* InvalidExchangeContractAddress
* EthereumAccountMismatchWebSocketSession
* InvalidTransactionType
* InvalidTokenAmount
* InvalidTokenPrice
* InvalidSignature
* InvalidTimestamp
* BuyOrderJsIPFSFailed
* AlreadyProcessed
* NoETHAccount
* NotEnoughASD


### PlaceSellOrder API \[OS\] (OrderSell)

> WS Request Message

```json
{
  "t" : "OS",
  "rqI" : "qwef21",
  "m" : {
    "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
    "tt" : "OS",
    "ea" : "0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
    "sy" : "AS_PC_GN",
    "am" : "5",
    "pr" : "55800000",
    "mfr" : 10,
    "tfr" : 20,
    "ts" : 1508490825659,
    "si" : "0x99093b874eb52e6d11f9a1892eb659d2af2c46b8ec39332193150f4ede0a88b63da0680587298043cd2b1817e8f30afaa99e289e713e8675253337dd711251e21c"
  }
}
```

| field | description |
| :---: | --- |
| **xa**​ | exchange Ethereum smart contract address (Exchange Vault) identifying the exchange server |
| **tt**​ | transaction type (‘OB’ : Order Buy, ‘OS’ : Order Sell, ‘TB’ : Trade-Buy, ‘TS’ : Trade-Sell, ‘CB’ : Cancel-Buy, ‘CS’ : Cancel-Sell, ‘WR’ : Withdrawal-Request, ‘CWR’ : Cancel-Withdrawal-Request, ‘WC’ : Withdrawal-Confirm) |
| **ea​** | user’s Ethereum account address |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **am**​ | Asset-Share token amount |
| **pr​** | dUSD price for 1 Asset-Share (6 decimals big integer) |
| **mfr**​ | maker fee rate (unit in percentage(0-100) multiplied by 100 (0(0%)-10000(100%), ex: 102 = 1.02%, 10 = 0.1%) |
| **tfr**​ | taker fee rate (same unit as ‘mfr’) |
| **ts** | client timestamp at which transaction message is made |
| **si**​ | crypto signature signed by the user’s Ethereum account for transaction message (tightly packed (no whitespace) stringified json object) |

> WS Response Message

```json
{
   "t" : "OSR",
   "rqI" : "qwef21",
   "r" : {
      "res" : "OK",
      "sI" : "QmQySFBQfu7eASiYFZCL2VXCMjJNsXZfrmwZW1eNNDXUXP",
      "uf" : "5"
   }
}
```

| field | description |
| :---: | --- |
| **sI** | sell order id (IPFS hash address of sell order message json file) |
| **uf** | unfilled token amount |

Response codes (res)
* InvalidExchangeContractAddress
* EthereumAccountMismatchWebSocketSession
* InvalidTransactionType
* InvalidTokenAmount
* InvalidTokenPrice
* InvalidTimestamp
* InvalidSignature
* SellOrderJsIPFSFailed
* AlreadyProcessed
* NoETHAccount
* NotEnoughToken
* NotEnoughASD



### CancelBuyOrder API \[CB\] (CancelBuy)

> WS Request Message

```json
{
  "t" : "CB",
  "rqI" : "123123",
  "m" : {
    "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
    "tt" : "CB",
    "ea" : "0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
    "sy" : "AS_PC_GN",
    "bI" : "QmNfov2eogp22isRdm893nXtRcmCb6WkzjcFYrVnN2H2kx",
    "ts" : 1508721792153,
    "si" : "0xfb1e1efc8ca542e7ce1c0c3914732f116fdad8a7b9317f3fa2f9f3d4d4d024713d2540cb31293d739fb612a11513667c0b35400a9400ffcd065e0be1013adc371b"
  }
}
```

| field | description |
| :---: | --- |
| **xa**​ | exchange Ethereum smart contract address (Exchange Vault) identifying the exchange server |
| **tt**​ | transaction type (‘OB’ : Order Buy, ‘OS’ : Order Sell, ‘TB’ : Trade-Buy, ‘TS’ : Trade-Sell, ‘CB’ : Cancel-Buy, ‘CS’ : Cancel-Sell, ‘WR’ : Withdrawal-Request, ‘CWR’ : Cancel-Withdrawal-Request, ‘WC’ : Withdrawal-Confirm) |
| **ea​** | user’s Ethereum account address |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **bI** | buy-order-id to cancel |
| **ts** | client timestamp at which transaction message is made |
| **si**​ | crypto signature signed by the user’s Ethereum account for transaction message (tightly packed (no whitespace) stringified json object) |

> WS Response Message

```json
{
   "t" : "CBR",
   "rqI" : "123123",
   "r" : {
      "res" : "OK",
      "cI" : "QmQivaSWXPxUeTRJhVdk96ker4tVWRuEkBEfXUAAE99vjh",
      "p" : "52100000",
      "uf" : "15"
   }
}
```

| field | description |
| :---: | --- |
| **cI** | cancel order id (IPFS hash address of cancel order message json file) |
| **p** | dUSD price for 1 Asset-Share (6 decimals big integer) |
| **uf** | unfilled token amount |

Response codes (res)
* InvalidExchangeContractAddress
* EthereumAccountMismatchWebSocketSession
* InvalidTransactionType
* InvalidTimestamp
* InvalidSignature
* CancelBuyOrderJsIPFSFailed
* AlreadyProcessed
* NotFound
* NotMatchETHAccount
* NotMatchTokenSymbol
* NotInOrderBook


### CancelSellOrder API \[CS\] (CancelSell)

> WS Request Message

```json
{
  "t" : "CS",
  "rqI" : "234234",
  "m" : {
    "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
    "tt" : "CS",
    "ea" : "0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
    "sy" : "AS_PC_GN",
    "sI" : "QmcFoXX5wkXQnsSkBL19v4U3tcNyauEHV7qq3TtKbT9W4b",
    "ts" : 1508728212595,
    "si" : "0x6c32665c468f77a3d3eb72b10cfc3d478290ce70a2341f01325b6f504b10861718ca74f50c1a06e320ec7c94d989abaec5a53225221afcf48df1966caa64c8981c"
  }
}
```

| field | description |
| :---: | --- |
| **xa**​ | exchange Ethereum smart contract address (Exchange Vault) identifying the exchange server |
| **tt**​ | transaction type (‘OB’ : Order Buy, ‘OS’ : Order Sell, ‘TB’ : Trade-Buy, ‘TS’ : Trade-Sell, ‘CB’ : Cancel-Buy, ‘CS’ : Cancel-Sell, ‘WR’ : Withdrawal-Request, ‘CWR’ : Cancel-Withdrawal-Request, ‘WC’ : Withdrawal-Confirm) |
| **ea​** | user’s Ethereum account address |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **sI** | sell-order id to cancel |
| **ts** | client timestamp at which transaction message is made |
| **si**​ | crypto signature signed by the user’s Ethereum account for transaction message (tightly packed (no whitespace) stringified json object) |

> WS Response Message

```json
{
   "t" : "CSR",
   "rqI" : "234234",
   "r" : {
      "res" : "OK",
      "cI" : "QmPx7bz1Pu5UyQS1aat9o6MikdUqPd4LwMH88f2Gv1fLhy",
      "p" : "55800000",
      "uf" : "5"
   }
}
```

| field | description |
| :---: | --- |
| **cI** | cancel order id (IPFS hash address of cancel order message json file) |
| **p** | dUSD price for 1 Asset-Share (6 decimals big integer) |
| **uf** | unfilled token amount |

Response codes (res)
* InvalidExchangeContractAddress
* EthereumAccountMismatchWebSocketSession
* InvalidTransactionType
* InvalidTimestamp
* InvalidSignature
* CancelSellOrderJsIPFSFailed
* AlreadyProcessed
* NotFound
* NotMatchETHAccount
* NotMatchTokenSymbol
* NotInOrderBook



### WithdrawalRequest API \[WR\]

> WS Request Message

```json
{
  "t" : "WR",
  "rqI" : "345345",
  "m" : {
    "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
    "tt" : "WR",
    "ea" : "0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
    "sy" : "ASD",
    "am" : "3000000000",
    "txF" : "500000",
    "ts" : 1508401521096,
    "si" : "0x6b19d44a1c13ae81009366d286e4279949a4964a6bc492931268610070fd042871f7c2af95266641f54f2493b4f2b369028be4254f24a021f5eb5ba070df0ca61b"
  }
}
```

| field | description |
| :---: | --- |
| **xa**​ | exchange Ethereum smart contract address (Exchange Vault) identifying the exchange server |
| **tt**​ | transaction type (‘OB’ : Order Buy, ‘OS’ : Order Sell, ‘TB’ : Trade-Buy, ‘TS’ : Trade-Sell, ‘CB’ : Cancel-Buy, ‘CS’ : Cancel-Sell, ‘WR’ : Withdrawal-Request, ‘CWR’ : Cancel-Withdrawal-Request, ‘WC’ : Withdrawal-Confirm) |
| **ea​** | user’s Ethereum account address |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) or ‘dUSD’ |
| **am**​ | Asset-Share token amount or dUSD amount (6 decimals big integer, 1 dUSD = 1000000, 30.5 dUSD = 30500000) |
| **txF** | transaction fee dUSD amount in a Withdrawal-Request transaction (6 decimals big integer) |
| **ts** | client timestamp at which transaction message is made |
| **si**​ | crypto signature signed by the user’s Ethereum account for transaction message (tightly packed (no whitespace) stringified json object) |

> WS Response Message

```json
{
   "t" : "CSR",
   "rqI" : "234234",
   "r" : {
      "res" : "OK",
      "wrI" : "QmcZgfjBfsTiRv2LrEupDdDqgfXdwLSfUVJhFjQZn6u2XY",
      "wsI" : "QmQDMQtjpqnu3ErgDCipmPfcRpk1gs6eshzN5rE486Ryja",
      "wsJs" : {"xa":"0x33e50109a188cdbf976c60fb93e34119cba0a088","doc":"WS","tsS":1508401523214, "..." : "..."}
   }
}
```

| field | description |
| :---: | --- |
| **wrI** | withdrawal request id (IPFS hash address of withdrawal request message json file) |
| **wsI** | withdrawal settlement id (IPFS hash address of withdrawal settlement data json file) |
| **wsJs** | withdrawal settlement data json (file contents of IPFS file whose address if 'wsI' |

Response codes (res)
* InvalidExchangeContractAddress
* EthereumAccountMismatchWebSocketSession
* InvalidTransactionType
* InvalidTokenAmount
* InvalidTransactionFeeAmount
* InvalidTimestamp
* InvalidSignature
* WithdrawRequestJsIPFSFailed
* SettlementVerificationFailed
* SettlementDataSaveFailed
* AlreadyProcessed
* NoETHAccount
* NotEnoughASD
* NotEnoughToken
* AccountLocked
* WithdrawalInProgress
* NotMatchETHAccount
* NotMatchTokenSymbol
* NotInOrderBook

> Withdrawal Settlement Data json

```json
{  
   "xa":"0x33e50109a188cdbf976c60fb93e34119cba0a088",
   "doc":"WS",
   "tsS":1508401523214,
   "ea":"0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
   "sy":"ASD",
   "am":"3000000000",
   "txF":"500000",
   "wrI":"QmcZgfjBfsTiRv2LrEupDdDqgfXdwLSfUVJhFjQZn6u2XY",
   "cTxS":734,
   "cAB":{  
      "a":"2974937693000",
      "aE":"10960950000",
      "ATs":[  
         {  
            "s":"AS_GH_SN",
            "t":"10",
            "tE":"0",
            "aE":"0"
         },
         {  
            "s":"AS_AG_WM",
            "t":"100",
            "tE":"0",
            "aE":"0"
         },
         {  
            "s":"AS_RL_WH",
            "t":"300",
            "tE":"0",
            "aE":"0"
         },
         {  
            "s":"AS_PC_GN",
            "t":"120",
            "tE":"0",
            "aE":"0"
         },
         {  
            "s":"AS_DH_LG",
            "t":"0",
            "tE":"0",
            "aE":"10960950000"
         }
      ],
      "OOs":[  
         {  
            "Xt":"OB",
            "Xsy":"AS_DH_LG",
            "Xp":"36500000",
            "Xta":"300",
            "Oi":"QmTYrcppMS4ub9rF63EPN2rvkrZA7UkmwWEKH95H2BYmFX",
            "OmF":10,
            "OtF":20,
            "Ouf":"300"
         }
      ]
   },
   "TXs":[  
      {  
         "Xt":"W",
         "Xs":623,
         "Xsy":"ASD",
         "Xts":1504094280444,
         "Xta":"3000000000",
         "Wi":"QmYkwS3S7qBgXJYWmX3m19Uv2YmocW4Hvwo9ayVK1PWm3e",
         "WsI":"QmQDMQtjpqnu3ErgDCipmPfcRpk1gs6eshzN5rE486Ryja",
         "WtxF":"500000",
         "Wts":1504094252731,
         "Wsi":"0x51281c66b6dc03ba2d31313f69035de69ad53cb0bd55cfd24d23a75e11c7662a24ef2b0540c5def7fbd9d3e767f60651785cf53a7207bc42f2c3e0d3494af71d1c",
         "WtxH":"0x3258f49bc8a3d113e7f46333c86e1f8a6651dbe1595cf5b27224f745c7bb16b8"
      },
      {  
         "Xt":"OB",
         "Xs":646,
         "Xsy":"AS_AG_WM",
         "Xts":1504627218290,
         "Xp":"13500000",
         "Xta":"100",
         "Oi":"QmT1rpn42bum5URhmHRR1Zaty57kM7mcgGH7kEBRjiXTRp",
         "OmF":10,
         "OtF":20,
         "Ots":1504627216428,
         "Osi":"0xc5a96a106e58bb543a73e0dabe977322d2cb6d4dd9b8c3d1d7bd8559a48166873fe3bb57e286164c737768adeb16cde512a8c63ed89f0cac484e60d43c1ed25f1c"
      },
      {  
         "Xt":"T",
         "Xs":647,
         "Xsy":"AS_AG_WM",
         "Xts":1504627218290,
         "Xp":"13500000",
         "Xta":"100",
         "Ti":"QmYfj7THttNwCmE8RQ1zAaW5bsTRLujcVPKxKUHmk3hY1D",
         "TbI":"QmT1rpn42bum5URhmHRR1Zaty57kM7mcgGH7kEBRjiXTRp",
         "TsI":"QmSmZKYpmTcCJgT48MW4qFVJfYVgPYsanjo5wWV89vxvNs",
         "Tty":"B",
         "Tmt":"T",
         "Tf":"2700000"
      },
      {  
         "Xt":"OB",
         "Xs":659,
         "Xsy":"AS_PC_GN",
         "Xts":1504795447954,
         "Xp":"52100000",
         "Xta":"50",
         "Oi":"QmdChdLRoCQi5BjfSG3QXvgHW5rLAGsqUJVcpAbX3QZ9Sp",
         "OmF":10,
         "OtF":20,
         "Ots":1504795445476,
         "Osi":"0xd88776a935a36b81f20957741521a913b2f1cba101c352ea4bf7d6e91d98c44a7fcb06cce53fca1cecd7cbacc32696db78908f82bc6c5b3a6ff0759e77c54e381c"
      },
      {  
         "Xt":"T",
         "Xs":660,
         "Xsy":"AS_PC_GN",
         "Xts":1504795447954,
         "Xp":"52100000",
         "Xta":"50",
         "Ti":"QmYH1FsPTJbJgShtBMUBZiRQ8csqyhD31MtChdVQUSVM5g",
         "TbI":"QmdChdLRoCQi5BjfSG3QXvgHW5rLAGsqUJVcpAbX3QZ9Sp",
         "TsI":"QmZr5rvj51nTKfiU9FqqA3pG11Fov5fptwctswgJ4ANfeg",
         "Tty":"B",
         "Tmt":"T",
         "Tf":"5210000"
      },
      {  
         "Xt":"OB",
         "Xs":661,
         "Xsy":"AS_PC_GN",
         "Xts":1504796042497,
         "Xp":"52100000",
         "Xta":"20",
         "Oi":"QmRqTu7a2is2ny1JM2rygugX5e5dcmBnsjgsc4SitdbacX",
         "OmF":10,
         "OtF":20,
         "Ots":1504796039932,
         "Osi":"0x8cc4724038f6e000573019f9538dee5f1dfff1049ab7df053524da9609b2cd7a37e3a243456e5a8984e735307b19540801c4f0db48cce7fa75b23514e60ddc511c"
      },
      {  
         "Xt":"T",
         "Xs":662,
         "Xsy":"AS_PC_GN",
         "Xts":1504796042497,
         "Xp":"52100000",
         "Xta":"20",
         "Ti":"QmX3HxZJLs8A19BJQXHaKA9HZ9wbTEH3rFJE1FDdqWzDbQ",
         "TbI":"QmRqTu7a2is2ny1JM2rygugX5e5dcmBnsjgsc4SitdbacX",
         "TsI":"QmZr5rvj51nTKfiU9FqqA3pG11Fov5fptwctswgJ4ANfeg",
         "Tty":"B",
         "Tmt":"T",
         "Tf":"2084000"
      },
      {  
         "Xt":"OB",
         "Xs":725,
         "Xsy":"AS_RL_WH",
         "Xts":1506117600712,
         "Xp":"11000000",
         "Xta":"300",
         "Oi":"QmPH9dygt6JvaEfVKvAPbsnofn6aWpbVwinSLsLH2qXsrV",
         "OmF":10,
         "OtF":20,
         "Ots":1506117597480,
         "Osi":"0x7db0380538e00655caa7cc2ba35a1da80d1ed75df6a314905963868910dd11f06997839edf5fd66456ecf2a600205d812d6258c11cbec9866ebd1939e68d02111c"
      },
      {  
         "Xt":"T",
         "Xs":726,
         "Xsy":"AS_RL_WH",
         "Xts":1506117600712,
         "Xp":"11000000",
         "Xta":"300",
         "Ti":"QmZvfEuP6XaT9Ze4EjtsZ1aZqT15ErxM5ryRE7vEu8HGNL",
         "TbI":"QmPH9dygt6JvaEfVKvAPbsnofn6aWpbVwinSLsLH2qXsrV",
         "TsI":"QmYH2gPug9yDXCeoWgdRnKuLwMPz9RwqwDSqWCansgMVvJ",
         "Tty":"B",
         "Tmt":"T",
         "Tf":"6600000"
      },
      {  
         "Xt":"OB",
         "Xs":727,
         "Xsy":"AS_DH_LG",
         "Xts":1506118070953,
         "Xp":"36500000",
         "Xta":"300",
         "Oi":"QmTYrcppMS4ub9rF63EPN2rvkrZA7UkmwWEKH95H2BYmFX",
         "OmF":10,
         "OtF":20,
         "Ots":1506118068248,
         "Osi":"0x48b168f25e17fd6f262cb93eee33b41255e6d0dd539ea7044bb5d94e9a4d3cbe2ab788e90552d3eabd0615fadba28f932ea886c86620b951a9a6b2a1a1a0510d1c"
      }
   ],
   "pTxS":622,
   "pAB":{  
      "a":"2997212737000",
      "aE":"0",
      "ATs":[  
         {  
            "s":"AS_GH_SN",
            "t":"10",
            "tE":"0",
            "aE":"0"
         },
         {  
            "s":"AS_PC_GN",
            "t":"50",
            "tE":"0",
            "aE":"0"
         }
      ],
      "OOs":[
      ]
   },
   "pSI":"QmQDMQtjpqnu3ErgDCipmPfcRpk1gs6eshzN5rE486Ryja",
   "siS":"0xeecd061d0078384475cc92fec17623679fd8b57a916e1f7b1253fd977d19ebf22bfc34506c50ad135b224b3fb87e4ddeb0cddc13cad4b6e4ff7655d848c647de1c"
}
```
| field | description |
| :---: | --- |
| **cTxs** | current transaction sequence |
| **cAB** | current account balances |
| **a​** | dUSD balance |
| **aE** | dUSD escrowed |
| **ATs** | Asset-Share tokens |
| **s** | symbol |
| **t​** | token amount |
| **tE​** | token escrowed |
| **OOs** | open orders |
| **TXs​** | transactions |
| **Xt** | transaction type |
| **Xs** | transaction sequence |
| **Xts** | transaction server timestamp |
| **Xsy** | transaction token symbol |
| **Xp** | transaction price |
| **Xta** | transaction token amount |
| **Oi​** | order id |
| **OmF** | order maker fee rate |
| **OtF** | order taker fee rate |
| **Ouf** | order unfilled amount |
| **Ots​** | order client timestamp |
| **Osi** | order user signature |
| **Wi​** | withdrawal id |
| **WsI**​ | withdrawal settlement id |
| **WtxF** |​ withdrawal transaction fee |
| **Wts** | withdrawal client timestamp |
| **Wsi** | withdrawal user signature |
| **WtxH​** | withdrawal ethereum transaction hash |
| **Di​** | deposit id (ethereum transaction hash) |
| **Ti** | trade id |
| **TbI​** | trande buy order id |
| **TsI​** | trade sell order id | 
| **Tty** | trade type |
| **Tmt​** | trade maker or taker |
| **Tf​** | transaction fee |
| **pTxS** | previous settlement transaction sequence |
| **pAB** | previous settlement account balances |
| **pSI** | previous settlement hash id |
| **pWtxH​** | previous withdrawal ethereum transaction hash |


### CancelWithdrawalRequest API \[CWR\]

> WS Request Message

```json
{
  "t" : "CWR",
  "rqI" : "456456",
  "m" : {
    "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
    "tt" : "CWR",
    "ea" : "0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
    "wrI" : "QmcZgfjBfsTiRv2LrEupDdDqgfXdwLSfUVJhFjQZn6u2XY" , 
    "ts" : 1508490809453,
    "si" : "0x75313dce30f110a5797600c611e9031ad596cb0a6457829a7c6f7e91a004ce9a35fe256d970432cc19b831af312aeb3f0a5c36202a2f54e6e83f3b6e199ded061c"
  }
}
```

| field | description |
| :---: | --- |
| **xa**​ | exchange Ethereum smart contract address (Exchange Vault) identifying the exchange server |
| **tt**​ | transaction type (‘OB’ : Order Buy, ‘OS’ : Order Sell, ‘TB’ : Trade-Buy, ‘TS’ : Trade-Sell, ‘CB’ : Cancel-Buy, ‘CS’ : Cancel-Sell, ‘WR’ : Withdrawal-Request, ‘CWR’ : Cancel-Withdrawal-Request, ‘WC’ : Withdrawal-Confirm) |
| **ea​** | user’s Ethereum account address |
| **wrI** | withdrawal-request id in a Cancel-Withdrawal-Request or Withdrawal-Confirm transaction |
| **ts** | client timestamp at which transaction message is made |
| **si**​ | crypto signature signed by the user’s Ethereum account for transaction message (tightly packed (no whitespace) stringified json object) |

> WS Response Message

```json
{
   "t" : "CWRR",
   "rqI" : "456456",
   "r" : {
      "res" : "OK",
      "cwrI" : "QmdDhwPvjtkpAgA9aasciC3FmEqF2GX3RLB8ioZnwV9vn1"
   }
}
```

| field | description |
| :---: | --- |
| **cwrI** | cancel-withdrawal-request id (IPFS hash address of cancel-withdrawal-request message json file) |

Response codes (res)
* InvalidExchangeContractAddress
* EthereumAccountMismatchWebSocketSession
* InvalidTransactionType
* InvalidTimestamp
* InvalidSignature
* CancelWithdrawalRequestJsIPFSFailed
* AlreadyProcessed
* NoWithdrawalRequest
* NotMatchETHAccount
* ETHWithdrawalTxInProgress


### ConfirmWithdrawalSettlement API \[WC\] (WithdrawalConfirm)

> WS Request Message

```json
{
  "t" : "WC",
  "rqI" : "456456",
  "m" : {
    "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
    "tt" : "WC",
    "ea" : "0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
    "sy" : "ASD",
    "am" : "3000000000",
    "txF" : "500000",
    "wrI" : "QmTvVrXNyQdsMBb5pG8vFDzE61e3dKfe5eofR3yPD6peV9",
    "wrTs" : 1504094158548,
    "wsI" : "QmQDMQtjpqnu3ErgDCipmPfcRpk1gs6eshzN5rE486Ryja",
    "ts" : 1504094252731,
    "si" : "0x51281c66b6dc03ba2d31313f69035de69ad53cb0bd55cfd24d23a75e11c7662a24ef2b0540c5def7fbd9d3e767f60651785cf53a7207bc42f2c3e0d3494af71d1c"
  }
}
```

| field | description |
| :---: | --- |
| **xa**​ | exchange Ethereum smart contract address (Exchange Vault) identifying the exchange server |
| **tt**​ | transaction type (‘OB’ : Order Buy, ‘OS’ : Order Sell, ‘TB’ : Trade-Buy, ‘TS’ : Trade-Sell, ‘CB’ : Cancel-Buy, ‘CS’ : Cancel-Sell, ‘WR’ : Withdrawal-Request, ‘CWR’ : Cancel-Withdrawal-Request, ‘WC’ : Withdrawal-Confirm) |
| **ea​** | user’s Ethereum account address |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) or ‘dUSD’ |
| **am**​ | Asset-Share token amount or dUSD amount (6 decimals big integer, 1 dUSD = 1000000, 30.5 dUSD = 30500000) |
| **txF** | transaction fee dUSD amount in a Withdrawal-Request transaction (6 decimals big integer) |
| **wrI** | withdrawal-request id in a Cancel-Withdrawal-Request or Withdrawal-Confirm transaction |
| **wrTs** | withdrawal-request timestamp |
| **wsI** | withdrawal-settlement-data file hash id |
| **ts** | client timestamp at which transaction message is made |
| **si**​ | crypto signature signed by the user’s Ethereum account for transaction message (tightly packed (no whitespace) stringified json object) |

> WS Response Message

```json
{
   "t" : "WCR",
   "rqI" : "456456",
   "r" : {
      "res" : "OK",
      "wI" : "QmYkwS3S7qBgXJYWmX3m19Uv2YmocW4Hvwo9ayVK1PWm3e",
      "wTxH" : "0x3258f49bc8a3d113e7f46333c86e1f8a6651dbe1595cf5b27224f745c7bb16b8"
   }
}
```

| field | description |
| :---: | --- |
| **wI** | withdrawal id (IPFS hash address of withdrawal-confirm message json file) |
| **wTxH** | blockchain transaction hash for withdrawal Ethereum transaction of Exchange Vault smart contract) |

Response codes (res)
* InvalidExchangeContractAddress
* EthereumAccountMismatchWebSocketSession
* InvalidTransactionType
* InvalidTimestamp
* InvalidSignature
* ConfirmWithdrawalSettlementJsIPFSFailed
* NotWithdrawalRequestStatus
* NotMatchSymbol
* NotMatchTokenAmount
* NotMatchTxFeeAmount
* NotMatchWithdrawalRequestClientTimestamp
* EthTxSendFailed
* AlreadyProcessed
* NoWithdrawalRequest
* NotMatchETHAccount
* NotMatchSettlementId
* ETHWithdrawalTxInProgress


### GetPendingWithdrawalRequest API \[GPWR\]

> WS Request Message

```json
{
  "t" : "GPWR",
  "rqI" : "qwef23",
  "m" : {}
}
```
Ethereum account address parameter is implicit by websocket connection

> WS Response Message

```json
{
  "t" : "GPWRR",
  "rqI" : "qwef23",
  "r" : {
    "res" : "OK",
    "st" : "WX",
    "wrI" : "QmNfB5sWGgqA4MefrDBoHa2pvEvx6tEKXRAeb2zNnV9CVd",
    "sy":"ASD",
    "ta" : "2000000000",
    "txF":"500000",
    "ts" : 1508728224481,
    "si" : "0xdd6ef3c5a0bed391f94412da5d63e6cff623965c3bbccebdbd51d4414ecf79145b39b21d3713b61913b474d3355b106a42cd45e7c1f1acea5c37e91492347b981b",
    "wsI" : "QmRcJAXHUV1CgSRrxNTgWWbzGUzMHu4Uv4QyvF8tS8YEsC",
    "pEthTxH" : "0x3258f49bc8a3d113e7f46333c86e1f8a6651dbe1595cf5b27224f745c7bb16b8"
  }
}
```

| field | description |
| :---: | --- |
| **st** | null : no withdrawal in progress, 'WR' : in withdrawal request status, 'WX' : withdrawal Ethereum transaction in progress |
| **wI** | withdrawal id (IPFS hash address of withdrawal-confirm message json file) |
| **wrI** | withdrawal-request id in a Cancel-Withdrawal-Request or Withdrawal-Confirm transaction |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) or ‘dUSD’ |
| **ta**​ | Asset-Share token amount or dUSD amount (6 decimals big integer, 1 dUSD = 1000000, 30.5 dUSD = 30500000) |
| **txF** | transaction fee dUSD amount in a Withdrawal-Request transaction (6 decimals big integer) |
| **ts** | withdrawal request timestamp |
| **si** | crypto signature of withdrawal request |
| **wsI** | withdrawal-settlement-data file hash id |
| **pEthTxH** | pending blockchain transaction hash for withdrawal Ethereum transaction of Exchange Vault smart contract) |

Response codes (res)
* NoETHAccount
* NotInWithdrawalProcess


Exchange Account APIs
---

### GetTradingCurrencyBalance API \[GTCB\]

> WS Request Message

```json
{
  "t" : "GTCB",
  "rqI" : "23427",
  "m" : {}
}
```
Ethereum account address parameter is implicit by websocket connection

> WS Response Message

```json
{
  "t" : "GTCBR",
  "rqI" : "23427",
  "r" : {
    "res" : "OK",
    "st" : "WX",
    "asd" : "2975216135000",
    "asdE" : "10960950000"
  }
}
```

| field | description |
| :---: | --- |
| **asd** | trading currency amount (dUSD, 6 decimals big integer, 1 dUSD = 1000000, 30.5 dUSD = 30500000) |
| **asdE** | trading currency amount locked(escrowed) in user's open orders |

Response codes (res)
* NoETHAccount


### GetAssetTokenBalance API \[GATB\]

> WS Request Message

```json
{
  "t" : "GATB",
  "rqI" : "7654567",
  "m" : {
    "sy" : "AS_PC_GN"
  }
}
```

| field | description |
| :---: | --- |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |

Ethereum account address parameter is implicit by websocket connection

> WS Response Message

```json
{
  "t" : "GATBR",
  "rqI" : "7654567",
  "r" : {
    "res" : "OK",
    "st" : "WX",
    "asd" : "2975216135000",
    "asdE" : "10960950000",
    "asdF" : "1000000000000",
    "token" : 115,
    "tokenE" : 20,
    "tokenASDE":"312500000"
  }
}
```

| field | description |
| :---: | --- |
| **asd** | trading currency amount available (dUSD, 6 decimals big integer, 1 dUSD = 1000000, 30.5 dUSD = 30500000) |
| **asdE** | trading currency amount locked(escrowed) in user's open orders |
| **asdF** | (alpha version only) trading currency amount issued to this account through free faucet |
| **token**​ | Asset-Share token amount available |
| **tokenE**​ | Asset-Share token amount locked(escrowed) in user's open sell order for this asset token |
| **tokenASDE** | trading currency amount locked(escrowed) in user's open orders for this asset token |

Response codes (res)
* NoETHAccount


### GetUserOpenOrders API \[GUOO\]

> WS Request Message

```json
{
  "t" : "GUOO",
  "rqI" : "98754",
  "m" : {
    "sy" : "AS_DH_LG"
  }
}
```

| field | description |
| :---: | --- |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type), if 'sy' field is not specified, all open orders for the every asset tokens will be returned |

Ethereum account address parameter is implicit by websocket connection

> WS Response Message

```json
{
  "t" : "GUOOR",
  "rqI" : "98754",
  "r" : {
    "res" : "OK",
    "Os" : [
      {
        "id" : "QmTYrcppMS4ub9rF63EPN2rvkrZA7UkmwWEKH95H2BYmFX",
        "sy" : "AS_DH_LG",
        "ty" : "B",
        "p" : "36500000",
        "t" : 300,
        "uf" : 300,
        "aDE" : "10960950000",
        "ts" : 1506118070953
      }
    ]
  }
}
```

| field | description |
| :---: | --- |
| **Os** | open order item array |
| **id** | (buy/sell) order ID (IPFS hash address) |
| **sy** | asset token symbol (Optional) |
| **ty** | order type ("B" : Buy, "S" : Sell) |
| **p** | asset token price in trading currency unit (dUSD, 6 decimals big integer) |
| **t** | order asset token amount |
| **uf** | unfilled asset token amount (support partial order fill) |
| **aDE** | trading currency amount locked(escrowed) escrowed in this order |
| **ts** | timestamp of (buy/sell) order |

Response codes (res)
* NoETHAccount
* NoAssetToken


### GetUserTradeHistory API \[GUTH\]

> WS Request Message

```json
{
  "t" : "GUTH",
  "rqI" : "1234234",
  "m" : {
    "sy" : "AS_PC_GN"
  }
}
```

| field | description |
| :---: | --- |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type), if 'sy' field is not specified, all trading history for the every asset tokens will be returned |

Ethereum account address parameter is implicit by websocket connection

> WS Response Message

```json
{
  "t" : "GUTHR",
  "rqI" : "1234234",
  "r" : {
    "res" : "OK",
    "Ts" : [
      {
        "id" : "QmSxPYZqFrFpgutiEkJCbeur6SsQG4nHm2UQAErde7DTmu",
        "sy" : "AS_PC_GN",
        "p" : "55800000",
        "t" : 5,
        "ty" : "S",
        "mt" : "T",
        "f" : "558000",
        "ts" : 1508490827298
      },
      {
        "id" : "QmX3HxZJLs8A19BJQXHaKA9HZ9wbTEH3rFJE1FDdqWzDbQ",
        "sy" : "AS_PC_GN",
        "p" : "52100000",
        "t" : 20,
        "ty" : "B",
        "mt" : "T",
        "f" : "2084000",
        "ts" : 1504796042497
      }
    ]
  }
}
```

| field | description |
| :---: | --- |
| **Ts** | trade item array |
| **id** | trade ID (IPFS hash address of 'trade' transaction data) |
| **sy** | asset token symbol (Optional) |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **t** | traded asset token amount |
| **ty** | trade type ("B" : Buy, "S" : Sell) |
| **mt** | user's position ("T": Taker, "M": Maker) |
| **f** |  charged transaction fee amount in trading currency unit(dUSD) |
| **ts** | timestamp of trade exchange transaction |

Response codes (res)
* NoETHAccount
* NoAssetToken

> Trade Transaction Data Format

```json
{
  "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
  "tt" : "TS",
  "mea" : "0x38df6b04c455a7512a71a12f4f03f7bead9774e8",
  "tea" : "0x5677e23889387f0d0e774f2e930e91bcee9dcaa6",
  "bI" : "QmZX5FutrbDkmJiVGyVdeCTpMtE3oHr3ZBdrHaW1Bq8T4K",
  "sI" : "QmQySFBQfu7eASiYFZCL2VXCMjJNsXZfrmwZW1eNNDXUXP",
  "sy" : "AS_PC_GN",
  "am" : "5",
  "pr" : "55800000",
  "mf" : "279000",
  "tf" : "558000",
  "tsS" : 1508490827298,
  "siS" : "0xc1033b5c82d9856642a4af667b1cd6e87211635c1fb2666bcaef587059001e685bcb72997d665e3ebf8f4abb6fe64adb076f8057e8ca3dd054612469b40f83b41c"
}
``` 
| field | description |
| :---: | --- |
| **xa**​ | exchange Ethereum smart contract address (Exchange Vault) identifying the exchange server |
| **tt**​ | transaction type (‘OB’ : Order Buy, ‘OS’ : Order Sell, ‘TB’ : Trade-Buy, ‘TS’ : Trade-Sell, ‘CB’ : Cancel-Buy, ‘CS’ : Cancel-Sell, ‘WR’ : Withdrawal-Request, ‘CWR’ : Cancel-Withdrawal-Request, ‘WC’ : Withdrawal-Confirm) |
| **mea** | Ethereum account address of the maker |
| **tea** | Ethereum account address of the taker |
| **bI** | buy-order id (IPFS hash address of buy-order json file) |
| **sI** | sell-order id (IPFS hash address of sell-order json file) |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **am**​ | Asset-Share token amount traded |
| **pr​** | price in trading currency unit(dUSD) for 1 Asset-Share (6 decimals big integer) |
| **mf** | charged maker fee amount in trading currency unit(dUSD) |
| **tf** | charged taker fee amount in trading currency unit(dUSD) |
| **tsS** | server timestamp when the transaction message is made on exchange server |
| **siS** | crypto signature signed by the exchange server’s Ethereum account for a transaction message |



Trading Data APIs
---

### GetOrderBook API \[GOB\]

> WS Request Message

```json
{
  "t" : "GOB",
  "rqI" : "23r23rq",
  "m" : {
    "sy" : "AS_PC_GN"
  }
}
```

| field | description |
| :---: | --- |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |

> WS Response Message

```json
{  
   "t" : "GOBR",
   "rqI" : "23r23rq",
   "r" : {  
      "res" : "OK",
      "ts":1509901269898,
      "bops":[  
         {  
            "p":"55800000",
            "s":1312
         },
         {  
            "p":"42750000",
            "s":1056
         },
         {  
            "p":"40780000",
            "s":2500
         }
      ],
      "sops":[  
         {  
            "p":"63050000",
            "s":10081
         },
         {  
            "p":"65090000",
            "s":50000
         },
         {  
            "p":"80000000",
            "s":80000
         }
      ]
   }
}
```

| field | description |
| :---: | --- |
| **ts** | timestamp of order-book |
| **bops** | list of Buy-Order prices |
| **sops** | list of Sell-Order prices |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **s** | sum of the asset token amounts at a specific token price |


Response codes (res)
* NoAssetToken


### GetTradeHistory API \[GTH\]

> WS Request Message

```json
{
  "t" : "GTH",
  "rqI" : "12r3rfvew",
  "m" : {
    "sy" : "AS_PC_GN",
    "ts" : 1506316965565,
    "l" : 5
  }
}
```

| field | description |
| :---: | --- |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **ts**​ | (optional) timestamp-before, trade data whose timestamp is before this 'ts' will be returned. (default) return recent trade data |
| **l** | (optional)(default=10) limit count for returned trade data items

> WS Response Message

```json
{  
   "t" : "GTHR",
   "rqI" : "12r3rfvew",
   "r" : {  
      "res" : "OK",
      "Ts" : [  
         {  
            "ty" : "B",
            "p" : "63050000",
            "t" : 9919,
            "ts" : 1506316965565
         },
         {  
            "ty" : "B",
            "p" : "59980000",
            "t" : 200,
            "ts" : 1506316965565
         },
         {  
            "ty" : "B",
            "p " :"62800000",
            "t" : 9781,
            "ts" : 1506316965565
         },
         {  
            "ty" : "B",
            "p" : "57430000",
            "t" : 100,
            "ts" : 1506316965565
         },
         {  
            "ty" : "S",
            "p" : "42750000",
            "t" : 20,
            "ts" : 1505411757356
         }
      ]
   }
}
```

| field | description |
| :---: | --- |
| **Ts** | list of trade history items |
| **ty** | trade type ('B': Trade-Buy(taker bought token), 'S': Trade-Sell(taker sold token) |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **t**​ | Asset-Share token amount traded |
| **ts** | timestamp of trade transaction |

Response codes (res)
* NoAssetToken


### SubscribeAssetTokenEvent API \[SATE\]

> WS Request Message

```json
{
  "t" : "SATE",
  "rqI" : "oiuyg23r",
  "m" : {
    "sy" : "AS_PC_GN"
  }
}
```

| field | description |
| :---: | --- |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) to subscribe to the asynchronous token trade event data |

Ethereum account address parameter is implicit by websocket connection

> WS Response Message

```json
{
  "t" : "SATER",
  "rqI" : "oiuyg23r",
  "r" : {
    "res" : "OK"
  }
}
```

| field | description |
| :---: | --- |
| **res** | if res = 'OK', asynchronous real-time trade events(buy/sell-order, cancel-order, trade) for the requested asset token will be delivered to current websocket connection. |

@see [Server-initiated Events - Token Event]

Response codes (res)
* FailedGetTokenEventPublisher
* FailedSubscribeToTokenEventPublisher


### UnSubscribeAssetTokenEvent API \[USATE\]

> WS Request Message

```json
{
  "t" : "USATE",
  "rqI" : "svew783r",
  "m" : {
    "sy" : "AS_PC_GN"
  }
}
```

| field | description |
| :---: | --- |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) to unsubscribe token trade event data |

Ethereum account address parameter is implicit by websocket connection

> WS Response Message

```json
{
  "t" : "USATER",
  "rqI" : "svew783r",
  "r" : {
    "res" : "OK"
  }
}
```

| field | description |
| :---: | --- |
| **res** | if res = 'OK', the delivery of asynchronous real-time trade events(buy/sell-order, cancel-order, trade) for the requested asset token will be stopped for current websocket connection |

@see [Server-initiated Events - Token Event]

Response codes (res)
* FailedGetTokenEventPublisher
* FailedUnSubscribeToTokenEventPublisher



Server-initiated Event Messages
---

Websocket response messages asynchronously initiated by server

> Server-initiated Event Message Format

```json
{
  "t" : "S_TE",
  "res_field_1" : "abcd",
  "res_field_2" : 1234
}
```

| field | description |
| :---: | --- |
| **t** | Message-Type prefixed with **'S_'** |
| res_field_1, ... | Message-Type specific data fields |


Server-initiated Events - Token Event
---
 
The server-initiated real-time token trade events (OB,OS,TB,TS,CB,CS) are broadcast
to all the token event subscribers through active websocket connections that have sent 'SubscribeAssetTokenEvent' message.


### Token Event - Order Buy \[S_TE\]\[OB\]

> WS Response Message

```json
{
  "t" : "S_TE",
  "sy" : "AS_PC_GN",
  "e" : {
    "ty" : "OB",
    "p" : "62500000",
    "t" : "20",
    "ts" : 1509891166587
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_TE': Sever-initiated Event - Token Event) |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **e**​ | event data |
| **ty**​ | token event type ('OB' : Order-Buy) |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **t**​ | Asset-Share token amount ordered |
| **ts** | timestamp of buy-order transaction |


### Token Event - Order Sell \[S_TE\]\[OS\]

> WS Response Message

```json
{
  "t" : "S_TE",
  "sy" : "AS_PC_GN",
  "e" : {
    "ty" : "OS",
    "p" : "62500000",
    "t" : "50",
    "ts" : 1509890916450
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_TE': Sever-initiated Event - Token Event) |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **e**​ | event data |
| **ty**​ | token event type ('OS' : Order-Sell) |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **t**​ | Asset-Share token amount ordered |
| **ts** | timestamp of sell-order transaction |


### Token Event - Trade Buy \[S_TE\]\[TB\]

> WS Response Message

```json
{
  "t" : "S_TE",
  "sy" : "AS_PC_GN",
  "e" : {
    "tx" : "TB",
    "p" : "62500000",
    "t" : "15",
    "ts" : 1509891917020
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_TE': Sever-initiated Event - Token Event) |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **e**​ | event data |
| **ty**​ | token event type ('TB' : Trade-Buy) |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **t**​ | Asset-Share token amount traded |
| **ts** | timestamp of trade transaction |


### Token Event - Trade Sell \[S_TE\]\[TB\]

> WS Response Message

```json
{
  "t" : "S_TE",
  "sy" : "AS_PC_GN",
  "e" : {
    "tx" : "TS",
    "p" : "63700000",
    "t" : "75",
    "ts" : 1509896359286
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_TE': Sever-initiated Event - Token Event) |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **e**​ | event data |
| **ty**​ | token event type ('TS' : Trade-Sell) |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **t**​ | Asset-Share token amount traded |
| **ts** | timestamp of trade transaction |


### Token Event - Cancel Buy \[S_TE\]\[CB\]

> WS Response Message

```json
{
  "t" : "S_TE",
  "sy" : "AS_PC_GN",
  "e" : {
    "ty" : "CB",
    "p" : "34790000",
    "uf" : "180",
    "ts" : 1509895438484
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_TE': Sever-initiated Event - Token Event) |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **e**​ | event data |
| **ty**​ | token event type ('CB' : Cancel-Buy) |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **uf** | unfilled token amount |
| **ts** | timestamp of cancel-buy transaction |


### Token Event - Cancel Sell \[S_TE\]\[CS\]

> WS Response Message

```json
{
  "t" : "S_TE",
  "sy" : "AS_PC_GN",
  "e" : {
    "ty" : "CS",
    "p" : "62500000",
    "uf" : "15",
    "ts" : 1509895600437
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_TE': Sever-initiated Event - Token Event) |
| **sy**​ | Asset-Share token symbol (id for an Asset-Share type) |
| **e**​ | event data |
| **ty**​ | token event type ('CS' : Cancel-Sell) |
| **p** | Asset-Share token price in trading currency unit (dUSD, 6 decimals big integer) |
| **uf** | unfilled token amount |
| **ts** | timestamp of cancel-sell transaction |




Server-initiated Events - Account Event
---

The Ethereum account specific events (T,DTC,DAT,WTC,WAT) are delivered in real-time 
through active websocket connections of the relevant Ethereum accounts

### Account Event - Trade \[S_AE\]\[T\]

'Trade' event is delivered in real-time to the maker and taker Ethereum account through the active websocket connection

> WS Response Message

```json
{
  "t" : "S_AE",
  "e" : {
    "ty" : "T",
    "tI" : "QmSXWt2WV2AtbEXRh33xs268VHRr4ibHDhVEhg7Rmuu4k6",
    "m" : {
      "xa" : "0x33e50109a188cdbf976c60fb93e34119cba0a088",
      "tt" : "TB",
      "mea" : "0x38df6b04c455a7512a71a12f4f03f7bead9774e8",
      "tea" : "0x38909c720ead8049eae29f3b3c439cd7b25e0590",
      "bI" : "QmabbDLgQ83ZzJe1c8SkgvuDxxAbH4djw1zXZGBQFWZzwA",
      "sI" : "QmWe9bz89dwRVGYELnyfJbwKMSVLvs3dP58biBZPGsbzCn",
      "sy" : "AS_PC_GN",
      "am" : "15",
      "pr" : "62500000",
      "mf" : "937500",
      "tf" : "1875000",
      "tsS" : 1509891917020,
      "siS" : "0xfece9054619c860b80e9d7f9243e3978a9f4efb9c6ba11aca27d2c3edb80ee6b371267a2685561b700b5531379e7ad8faf853dee7833de938a8c70f3536bc6431c"
    }
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_AE': Sever-initiated Event - Account Event) |
| **e**​ | event data |
| **ty**​ | account event type ('T' : Trade(order-filled)) |
| **tI** | trade ID (IPFS hash address of 'trade' transaction data) |
| **m** | trade transaction json data (@see [Trade Transaction Data Format]) |


### Account Event - Deposit Trading Currency \[S_AE\]\[DTC\]

fired when the deposit Ethereum transaction is confirmed

> WS Response Message

```json
{
  "t" : "S_AE",
  "e" : {
    "ty" : "DTC",
    "am" : "4999800000",
    "dI" : "0x156a58b1b616045027a63a34fbf5333b92fb40553e7d5f9baad9dfd2ff0c8854"
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_AE': Sever-initiated Event - Account Event) |
| **e**​ | event data |
| **ty**​ | account event type ('DTC' : Deposit-Trading-Currency) |
| **am**​ | trading currency amount deposited (dUSD, 6 decimals big integer, 1 dUSD = 1000000, 30.5 dUSD = 30500000) |
| **dI** | deposit ID (Ethereum blockchain transaction hash of 'Deposit' transaction) |


### Account Event - Deposit Asset Token \[S_AE\]\[DAT\]

fired when the deposit Ethereum transaction is confirmed

> WS Response Message

```json
{
  "t" : "S_AE",
  "e" : {
    "ty" : "DAT",
    "sy" : "AS_PC_GN",
    "am" : "55",
    "dI" : "0xd487960fe7bf64bfaa5cba79af322f85248e87e8138efb75e8055ca2dbcc1d45"
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_AE': Sever-initiated Event - Account Event) |
| **e**​ | event data |
| **ty**​ | account event type ('DAT' : Deposit-Asset-Token) |
| **am**​ | Asset-Share token amount deposited |
| **dI** | deposit ID (Ethereum blockchain transaction hash of 'Deposit' transaction) |


### Account Event - Withdrawal Trading Currency \[S_AE\]\[WTC\]

fired when the withdrawal Ethereum transaction is confirmed

> WS Response Message

```json
{
  "t" : "S_AE",
  "e" : {
    "ty" : "WTC",
    "am" : "1050000000",
    "wrI" : "QmW4ufRscYWpkwBQs8mYGu2URnHZcgS6tBVbY1gWMY6st2",
    "wI" : "QmPEV4VBXpFdov6gy1RtyydyhDpZrbMz9uwrSfXGWFKoKr",
    "wsI" : "QmWCGsT2tUdDkEwXMP1Gyb7x7tbZDZYVxodnA4E4Ddsooe",
    "wTxH" : "0x26a09940acdf3691897936bbecab98167407ec213dd262812d8c3de77a1cab60"
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_AE': Sever-initiated Event - Account Event) |
| **e**​ | event data |
| **ty**​ | account event type ('WTC' : Withdrawal-Trading-Currency) |
| **am**​ | trading currency amount withdrawn (dUSD, 6 decimals big integer, 1 dUSD = 1000000, 30.5 dUSD = 30500000) |
| **wrI** | withdrawal request id (IPFS hash address of withdrawal request message json file) |
| **wI** | withdrawal id (IPFS hash address of withdrawal-confirm message json file) |
| **wsI** | withdrawal settlement id (IPFS hash address of withdrawal settlement data json file) |
| **wTxH** | blockchain transaction hash for withdrawal Ethereum transaction of Exchange Vault smart contract) |


### Account Event - Withdrawal Asset Token \[S_AE\]\[WAT\]

fired when the withdrawal Ethereum transaction is confirmed

> WS Response Message

```json
{
  "t" : "S_AE",
  "e" : {
    "ty" : "WAT",
    "sy" : "AS_PC_GN",
    "am" : "35",
    "wrI" : "QmQ7hcoFqyWNsmPvZNY8bhSfrbv4GdPCUhL65RpNvhQxGJ",
    "wI" : "Qmb7egrDHcKgksYFpk6Nj5X4X7qrvYydAqsX6R2pSSsqS5",
    "wsI" : "QmafmHnkTX2SoSPWg2cpGxy5DkUAdsBFyKcXbUSBYYr6gA",
    "wTxH" : "0x01ac82377e8669a6e0dd7cb57eb3fb633ec5bf7e3520bc23deaa4cc16b3aa3ff"
  }
}
```

| field | description |
| :---: | --- |
| **t**​ | Message-Type ('S_AE': Sever-initiated Event - Account Event) |
| **e**​ | event data |
| **ty**​ | account event type ('WAT' : Withdrawal-Asset-Token) |
| **am**​ | Asset-Share token amount withdrawn |
| **wrI** | withdrawal request id (IPFS hash address of withdrawal request message json file) |
| **wI** | withdrawal id (IPFS hash address of withdrawal-confirm message json file) |
| **wsI** | withdrawal settlement id (IPFS hash address of withdrawal settlement data json file) |
| **wTxH** | blockchain transaction hash for withdrawal Ethereum transaction of Exchange Vault smart contract) |
