# HTTP API

## Contents

* [Status Descriptions](#Status Details(All API will arrange value according to the following status))
* [Markets](#Markets)
  * [Get trading pair info according to stock and money](#Get trading pair info according to stock and money)
  * [Get trading pair list according to stock and money](#Get trading pair list according to stock and money)
  * [Market list](#Market list)
  * [Market tx info](#Market tx info)
  * [Market candlestick charts](#Market candlestick charts)
  * [Market depth](#Market depth)
  * [Market order book](#Market order book)
  * [Market latest execution](#Market latest execution)
* [Orders](#Orders)
  * [Current orders](#Current orders)
  * [Order history](#Order history)
  * [Execution history](#Execution history)
  * [Order execution details](#Order execution details)
  * [Calculate order prevkey and id](#Calculate order prevkey and id)
  * [Delete single order info](#Delete single order info)
  * [Batch delete order info](#Batch delete order info)
* [Swap](#Swap)
  * [Swap records](#Swap records)
* [Pool](#Pool)
  * [Get user's liquidity in a certain market](#Get user's liquidity in a certain market)
  * [Address's current liquidity in each market](#Address's current liquidity in each market)
  * [Address's liquidity operation records in markets](#Address's liquidity operation records in markets)
  * [Liqudity pool change](#Liquidity pool change)
* [Transactions](#Transactions)
  * [Broadcast transactions](#Broadcast transactions)


## Status Details(All API will arrange value according to the following status)

```
# Order side
ORDER_BUY = 1  # Buy
ORDER_SELL = 2  # Sell

# Order type
ORDER_TYPE_SWAP = 1  # Swap
ORDER_TYPE_LIMIT = 2  # Limit 

# Order status
ORDER_STATUS_UNCONFIRMED = 1  # Unconfirmed
ORDER_STATUS_PENDING = 2  # Pending
ORDER_STATUS_FINISH = 3  # Succeeded
ORDER_STATUS_CANCEL = 4  # Cancelled
ORDER_STATUS_FAIL = 5 # Failed(swap order failed)

# Execution type
DEAL_TYPE_POOL = 1  # Executed with Pool
DEAL_TYPE_ORDER_BOOK = 2  # Executed with Order Book

# Executed as Taker/ Maker
DEAL_ROLE_MAKER = 1  # Maker
DEAL_ROLE_TAKER = 2  # Taker

# Liquidity pool operation type
POOL_OPERATION_TYPE_CREATE = 1  # Create liquidity pool
POOL_OPERATION_TYPE_ADD = 2  # Add liquidity
POOL_OPERATION_TYPE_REMOVE = 3  # Remove liquidity
POOL_OPERATION_TYPE_LIMIT = 4  # Limit order executed with liquidity pool 
POOL_OPERATION_TYPE_SWAP = 5  # Swap order executed with liquidity pool 

# Tx sending status 
TX_STATUS_UNCONFIRMED = 1  # Unconfirmed
TX_STATUS_SUCCESS = 2  # Succeeded 
TX_STATUS_FAIL = 3  # Failed

```

## Markets

### Get trading pair info according to stock and money

* method: `GET`
* path: `/api/eth/token/market`
* params:

name|Description|Type|Required
---|:---|:---|:---
`stock` | stock contract address | String | Yes 
`money` | money contract address | String | Yes
`is_only_swap` | swap only: 0-No, 1-Yes | String | Yes 

* result:

name|Description|Type
---|:---|:---
`market_contract` | market contract address | String
`stock_contract` | stock contract address | String
`money_contract` | money contract address | String
`liquidity_stock` | stock liquidity | String
`liquidity_money` | money liquidity | String
`liquidity_usd` | market liquidity (USD) | String
`only_swap` | swap only | Boolean

* example:

```json
{
    "code": 0,
    "message": "OK",
    "data": {
        "market_contract": "0xE37c49E057C2d5103E4C612052E85E8619eAAE82",
        "stock_contract": "0xbfE3805Ee22E3E21ae0a629581d11E89debf9A28",
        "money_contract": "0xC73eA7cc46151f4558D498541b1709a2bb9346Ff",
        "liquidity_money": "5.032000000000000000",
        "liquidity_stock": "503.200000000000000000",
        "liquidity_usd": "0.0000",
        "only_swap": false,
    }, 
}
```

### Get trading pair list according to stock and money

* method: `GET`
* path: `/api/eth/token/markets`
* params:

name|Description|Type|Required
---|:---|:---|:---
`stock` | stock contract address | String | Yes 
`money` | money contract address | String | Yes

* result:

name|Description|Type
---|:---|:---
`market_contract` | market contract address | String
`stock_contract` | stock contract address | String
`money_contract` | money contract address | String
`liquidity_stock` | stock liquidity| String
`liquidity_money` | money liquidity | String
`liquidity_usd` | market liquidity (USD) | String
`only_swap` | swap only | Boolean

* example:

```json
{
    "code": 0,
    "message": "OK",
    "data": [
        {
            "market_contract": "0xE37c49E057C2d5103E4C612052E85E8619eAAE82",
            "stock_contract": "0xbfE3805Ee22E3E21ae0a629581d11E89debf9A28",
            "money_contract": "0xC73eA7cc46151f4558D498541b1709a2bb9346Ff",
            "liquidity_money": "5.032000000000000000",
            "liquidity_stock": "503.200000000000000000",
            "liquidity_usd": "0.0000",
            "only_swap": false
        }
    ]
}
```

### Market list

* method: `GET`
* path: `/api/eth/market/list`
* params:

name|Description|Type|Required
---|:---|:---|:---
`others` | other types 1-Yes 0-No or NG | int | No
`money_contract` | money contract address | str | No


* result:

name|Description|Type
---|:---|:---
`name` | name | String
`market_contract` | contract address | String 
`price` | price | String 
`value` | market value | List 
`volume` | execution value | String 
`exchange_percent` | change percentage | String 


```json
{
    "code": 0,
    "message": "success",
    "data": [
      {
        "name": "BTC",
        "market_contract": "", 
        "price": "1.8989",
        "value": "90", 
        "volume": "9090", 
        "exchange_percent": "89"
      }
    ]
}
```

### Market tx info

* method: `GET`
* path: `/api/eth/market/<contract>`
* params:

* result:

name|Description|Type
---|:---|:---
`stock_name` | name | String
`stock_symbol` | stock_symbol | String 
`stock_contract` | contract address | String 
`money_name` | money_name | List 
`money_symbol` | money_symbol | String 
`market_contract` | contract address | String 
`price` | price | String 
`high` | 24H highest | String 
`low` | 24H lowest | String 
`open` | opening price | String 
`close` | closing price | String 
`amount` | 24H execution amount | String 
`volume` | 24H execution value | String 
`active_address_num` | 24H active addresses | String 
`money_contract` | contract address | String 
`only_swap` | swap only, true,false | String 


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "stock_name": "Ones Token",
      "stock_symbol": "ONES",
      "stock_contract": "0x53b983fe73e16f6ed8178f6c0e0b91f23dc9dad4cb30d0831f178291ffeb8750",
      "money_name": "Etherum",
      "money_symbol": "ETH",
      "money_contract": "0x53b983fe73e16f6ed8178f6c0e0b91f23dc9dad4cb30d0831f178291ffeb8750",
      "market_contract": "",
      "price": "1.8989",
      "high": "178", 
      "low": "89", 
      "open": "",
      "close": "",
      "amount": "18989", 
      "volume": "18675", 
      "active_address_num": "78", 
      "only_swap": "true" 
    }
}
```

### Market candlestick charts

* method: `GET`
* path: `/api/eth/market/<contract>/kline`
* params:

name|Description|Type|Required
---|:---|:---|:---
`interval` | intervals 1.min 2.hour 3.day | String | Yes
`start_time` | start time | Int | Yes
`end_time` | end time | Int | Yes

* result:

name|Description|Type
---|:---|:---
`t` | timestamp | String
`o` | open | String 
`h` | high | String 
`l` | low | String 
`c` | close | String 
`a` | execution amount | String 
`v` | execution value| String 


```json
{
    "code": 0,
    "message": "success",
    "data": [
      {
        "t": 1597987920, 
        "o": "2.19002962", 
        "h": "2.19002962", 
        "l": "2.19002962", 
        "c": "2.19002962", 
        "a": "0", 
        "v": "0" 
      }
    ]
}
```

### Market depth

* method: `GET`
* path: `/api/eth/market/<contract>/depth`
* params:

* result:

name|Description|Type
---|:---|:---
`ask` | sell | String
`p` | price | String
`a` | amount | String
`bid` | buy | String 
`p` | price | String
`a` | amount | String


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "ask": [
          {
            "p": "1.8989",
            "a": "18"
          }
      ],
      "bid": [
          {
            "p": "1.8989",
            "a": "18"
          }
      ]
    } 
}
```

### Market order book

* method: `GET`
* path: `/api/eth/market/<contract>/order-book`
* params:

* result:

name|Description|Type
---|:---|:---
`ask` | sell | String
`price` | price | String
`amount` | amount | String
`bid` | buy | String 
`price` | price | String
`amount` | amount | String


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "ask": [
          {
            "price": "1.8989",
            "amount": "18"
          }
      ],
      "bid": [
          {
            "price": "1.8989",
            "amount": "18"
          }
      ]
    }
}
```

### Market latest execution

* method: `GET`
* path: `/api/eth/market/<contract>/deals/latest`
* params:

* result:

name|Description|Type
---|:---|:---
`price` | price | String
`amount` | amount | String
`side` | side 1-buy 2-sell| String 
`timestamp` | timestamp | String


```json
{
    "code": 0,
    "message": "success",
    "data": [
      {
        "price": "1.8989",
        "amount": "8989",
        "side": 1, 
        "timestamp": 1787665335
      }
    ]
}
```

## Orders

### Current orders

* method: `GET`
* path: `/api/eth/address/<address>/order/pending`
* params:

name|Description|Type|Required
---|:---|:---|:---
`side` | side 0-all 1-buy 2-sell | Int | Yes
`order_type` | order types 0-all 1-swap 2-limit | Int | Yes
`start_time` | start time | Int | No
`end_time` | end time | Int | No
`market_contract` | market_contract | String | No
`search_key` | search key| String | No
`page` | page | Int | Yes
`limit` | limit | Int | Yes

* result:

name|Description|Type
---|:---|:---
`timestamp` | timestamp | Int
`order_type` | order types 1.swap 2.limit | String
`order_id` | order id | String
`deal_time` | execution time | String
`name` | name | String
`market_contract` | contract address | String
`side` | side | String
`price` | price| String
`amount` | amount | String
`deal_stock` | execution amount | String
`avg_price` | average price | String
`status` | order status, 1-pending, 2-succeeded  | String


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "records": [
        {
          "timestamp": 1767576567, 
          "order_type": 1, 
          "order_id": "",
          "deal_time": 1676567532, 
          "name": "BTC/USDT",
          "market_contract": "",
          "side": 1, 
          "price": "1.898", 
          "amount": "9090", 
          "deal_stock": "19", 
          "avg_price": "1.7878", 
          "status": 1 
        }
      ],
      "total": 90,
      "limit": 10,
      "has_next": false
    }
}
```

### Order history

* method: `GET`
* path: `/api/eth/address/<address>/order/history`
* params:

name|Description|Type|Required
---|:---|:---|:---
`side` | side 0-all 1-buy 2-sell | Int | Yes
`order_type` | order types 0-all 1-swap 2-limit | Int | Yes
`start_time` | start time | Int | No
`end_time` | end time | Int | No
`order_status` | hide cancelled orders 0-don't hide 3-executed (hide)| Int | No
`market_contract` | market_contract | String | No
`search_key` | search key | String | No
`page` | page | Int | Yes
`limit` | limit | Int | Yes

* result:

name|Description|Type
---|:---|:---
`timestamp` | timestamp | Int
`order_type` | order types 1.swap 2.limit | String
`order_id` | order id | String
`deal_time` | execution time | String
`name` | name | String
`market_contract` | contract address | String
`side` | side | String
`amount` | amount | String
`deal_stock` | execution amount | String
`deal_money` | execution value | String
`avg_price` | average price | String
`status` | order status, 3-executed, 4-cancelled| String


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "records": [
        {
          "timestamp": 1767576567, 
          "order_type": 1, 
          "name": "BTC/USDT",
          "market_contract": "",
          "order_id": "",
          "deal_time": 178687687, 
          "side": 1, 
          "amount": "9090", 
          "deal_stock": "19", 
          "deal_money": "90", 
          "avg_price": "1.7878", 
          "status": 3
        }
      ],
      "total": 90,
      "limit": 10,
      "has_next": false
    }
}
```

### Execution history

* method: `GET`
* path: `/api/eth/address/<address>/order/deals`
* params:

name|Description|Type|Required
---|:---|:---|:---
`side` | side 0-all 1-buy 2-sell | Int | Yes
`order_type` | order types 0-all 1-swap 2-limit | Int | Yes
`start_time` | start time | Int | No
`end_time` | end time | Int | No
`market_contract` | market_contract | String | No
`page` | page | Int | Yes
`limit` | limit | Int | Yes

* result:

name|Description|Type
---|:---|:---
`timestamp` | timestamp | Int
`order_type` | order types 1.swap 2.limit | String
`order_id` | order id | String
`name` | name | String
`market_contract` | contract address | String
`side` | order side 1-buy 2-sell | String
`price` | price | String
`deal_stock` | execution amount| String
`deal_money` | execution value | String


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "records": [
        {
          "timestamp": 1767576567, 
          "order_type": 1, 
          "name": "BTC/USDT",
          "market_contract": "",
          "order_id": "",
          "side": 1, 
          "price": "1.99", 
          "deal_stock": "9090", 
          "deal_money": "7878" 
        }
      ],
      "total": 90,
      "limit": 10,
      "has_next": false
    }
}
```


### Order execution details

* method: `GET`
* path: `/api/eth/address/order/<order_id>/deals`
* params:

* result:

name|Description|Type
---|:---|:---
`timestamp` | timestamp | Int
`order_type` | order types 1.swap 2.limit | String
`order_id` | order id | String
`name` | name | String
`market_contract` | contract address | String
`side` | side 1-buy 2-sell | String
`price` | price | String
`amount` | execution amount | String
`volume` | execution value | String


```json
{
    "code": 0,
    "message": "success",
    "data": [
        {
          "timestamp": 1767576567, 
          "order_type": 1, 
          "name": "BTC/USDT",
          "market_contract": "",
          "order_id": "",
          "side": 1, 
          "price": "1.99", 
          "amount": "9090", 
          "volume": "7878" 
        }
      ]
}
```

### Calculate order prevkey and id

* method: `GET`
* path: `/api/eth/transaction/prev-info`
* params:

name|Description|Type|Required
---|:---|:---|:---
`is_buy` | buy order | Bool | Yes 
`market_contract` | contract address | String | Yes 
`price` | price | String | Yes 
`amount` | amount | String | Yes 

* result:

name|Description|Type
---|:---|:---
`prev_key` | prev_key | String
`id` | id | String

```json
{
    "code": 0,
    "message": "success",
    "data": {
      "prev_key": "afasdf",
      "id": 13234
    }
}
```

### Delete single order info

* method: `GET`
* path: `/api/eth/transaction/remove/info`
* params:

name|Description|Type|Required
---|:---|:---|:---
`order_id` | order id | String | Yes 
`market_contract` | contract address| String | Yes 


* result:

name|Description|Type
---|:---|:---
`rm_list` | deletion list | String


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "rm_list": []
    }
}
```

### Batch delete order info

* method: `GET`
* path: `/api/eth/transaction/remove/batch-info`
* params:

name|Description|Type|Required
---|:---|:---|:---
`market_contract` | order id | String | Yes 
`address` | user address | String | Yes 


* result:

name|Description|Type
---|:---|:---
`rm_list` | deletion list | String

```json
{
    "code": 0,
    "message": "success",
    "data": {
      "rm_list": [],
    }
}
```

## Swap

### Swap records

* method: `GET`
* path: `/api/eth/address/<address>/swap/transactions`
* params:

name|Description|Type|Required
---|:---|:---|:---
`page` | page  | Int | Yes
`limit` | limit  | Int | Yes


* result:

name|Description|Type
---|:---|:---
`src_token` | src_token | String
`src_amount` | src_amount | String
`dst_token` | dst_token | String
`dst_amount` | dst_amount | String
`status` | 1-pending 3-finished | String
`timestamp` | timestamp | Int


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "records": [
          {
            "src_token": "BTC",
            "src_amount": "89",
            "dst_token": "ETH",
            "dst_amount": "78",
            "timestamp": 16757657687,
            "status": 1 
          }
      ],
      "total": 90,
      "limit": 10,
      "has_next": false
    }
}
```

## Pool

### Get user's liquidity in a certain market

* method: `GET`
* path: `/api/eth/pool/<market_contract>/<address>/liquidity`
* params:

name|Description|Type|Required
---|:---|:---|:---
`market_contract` | market contract address | String | Yes 
`address` | ethereum address | String | Yes 

* result:

name|Description|Type
---|:---|:---
`total_liquidity` | total liquidity | String
`stock_liquidity` | stock liquidity | String
`money_liquidity` | money liquidity | String
`user_liquidity` | enquire liquidity of an ethereum address | String

* example:

```json
{
    "code": 0,
    "message": "success",
    "data": {
        "total_liquidity": 1000000000,
        "stock_liquidity": 100,
        "money_liquidity": 1000,
        "user_liquidity": 1000,
    }
}
```

### Address's current liquidity in each market

* method: `GET`
* path: `/api/eth/address/<address>/pool/liquidity`
* params:

name|Description|Type|Required
---|:---|:---|:---
`page` | page  | Int | Yes
`limit` | limit  | Int | Yes

* result:

name|Description|Type
---|:---|:---
`name` | name | String
`market_contract` | contract address | String
`dst_token` | dst_token | String
`dst_amount` | dst_amount | String
`status` | 1-pending 3-finished | Int
`timestamp` | timestamp | Int


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "records": [
          {
            "name": "BTC/USDT",
            "market_contract": "",
            "stock_amount": "7898",
            "money_amount": "7667",
            "value": "7870"
          }
      ],
      "total": 89,
      "limit": 10,
      "has_next": false
    }
}
```

### Address's liquidity operation records in markets

* method: `GET`
* path: `/api/eth/address/<address>/pool/liquidity/history`
* params:

name|Description|Type|Required
---|:---|:---|:---
`page` | page  | Int | Yes
`limit` | limit  | Int | Yes
`operation_type` | operation types  | 0-all 1-add+create 3-remove 4-swap 5-limit | Yes


* result:

name|Description|Type
---|:---|:---
`stock` | stock | String
`money` | money | String
`market_contract` | contract address | String
`operation_type` | 0-all 1-create 2-add 3-remove | Int
`stock_amount` | stock_amount | String
`money_amount` | money_amount | String
`value` | value | String
`timestamp` | timestamp | Int


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "records": [
          {
            "stock": "BTC",
            "money": "USDT",
            "market_contract": "",
            "operation_type": 1, 
            "stock_amount": "7898",
            "money_amount": "7667",
            "value": "7870", 
            "timestamp": 1787867698
          }
      ],
      "total": 89,
      "limit": 10,
      "has_next": false
    }
}
```

### Liquidity pool change 

* method: `GET`
* path: `/api/eth/pool/<contract>/transactions`
* params:

name|Description|Type|Required
---|:---|:---|:---
`page` | page  | Int | Yes
`limit` | limit  | Int | Yes
`operation_type` | operation types  | 0-all 1-add+create 3-remove 4-limit 5-swap | Yes

* result:

name|Description|Type
---|:---|:---
`src_token` | stock | String
`dst_token` | money | String
`operation_type` | type 1-add 2-create 3-remove 4-limit 5-swap | Int
`address` | contract address | String
`value` | value | String
`stock_amount` | stock_amount | String
`money_amount` | money_amount | String
`timestamp` | timestamp | Int

```json
{
    "code": 0,
    "message": "success",
    "data": {
      "records": [
          {
            "src_token": "BTC",
            "dst_token": "USDT",
            "operation_type": 1, 
            "address": "0xa1ad...cebd", 
            "value": "78", 
            "stock_amount": "78",
            "money_amount": "67",
            "timestamp": 1786756587
          }
      ],
      "total": 89,
      "limit": 10,
      "has_next": false
    }
}
```

## Transactions

### Broadcast transactions

* method: `POST`
* path: `/api/eth/transaction/broadcast`
* params:

name|Description|Type|Required
---|:---|:---|:---
`raw_transaction` | broadcast transactions(for oneswap walllet) | String | Yes

* result:

name|Description|Type
---|:---|:---
`tx_hash` | tx_hash | String

```json
{
    "code": 0,
    "message": "success",
    "data": {
      "tx_hash": ""
    }
}
```
