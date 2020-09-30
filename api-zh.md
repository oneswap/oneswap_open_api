# HTTP API

## 目录

* [状态说明](#状态明细(所有接口按照以下状态进行取值))
* [Markets](#Markets)
  * [根据stock和money获取交易对信息](#根据stock和money获取交易对信息)
  * [根据stock和money获取交易对列表](#根据stock和money获取交易对列表)
  * [市场列表](#市场列表)
  * [市场交易信息](#市场交易信息)
  * [市场k线](#市场k线)
  * [市场深度](#市场深度)
  * [市场订单薄](#市场订单薄)
  * [市场最新成交](#市场最新成交)
* [Orders](#Orders)
  * [当前委托](#当前委托)
  * [历史委托](#历史委托)
  * [成交记录](#成交记录)
  * [订单成交明细](#订单成交明细)
  * [计算订单prevkey和id](#计算订单prevkey和id)
  * [删除单个订单信息](#删除单个订单信息)
  * [批量删除订单信息](#批量删除订单信息)
* [Swap](#Swap)
  * [兑换记录](#兑换记录)
* [Pool](#Pool)
  * [获取用户在某个市场的流动性](#获取用户在某个市场的流动性)
  * [地址的市场流动性](#地址在各个市场的流通性)
  * [地址的流动性操作记录](#地址的流动性操作记录)
  * [资金池变动记录](#资金池变动记录)
* [Transaction](#Transaction)
  * [广播交易](#广播交易)


## 状态明细(所有接口按照以下状态进行取值)

```
# 订单买卖方向
ORDER_BUY = 1  # 买
ORDER_SELL = 2  # 卖

# 订单类型
ORDER_TYPE_SWAP = 1  # swap单
ORDER_TYPE_LIMIT = 2  # 限价单

# 订单状态
ORDER_STATUS_UNCONFIRMED = 1  # 未确认
ORDER_STATUS_PENDING = 2  # 挂单中
ORDER_STATUS_FINISH = 3  # 已完成
ORDER_STATUS_CANCEL = 4  # 已取消
ORDER_STATUS_FAIL = 5 # 失败(swap订单失败)

# 成交类型
DEAL_TYPE_POOL = 1  # 与Pool成交类型
DEAL_TYPE_ORDER_BOOK = 2  # 与OrderBook成交

# 以Taker/Maker身份成交
DEAL_ROLE_MAKER = 1  # Maker
DEAL_ROLE_TAKER = 2  # Taker

# 交易对资金池操作类型
POOL_OPERATION_TYPE_CREATE = 1  # 创建资金池
POOL_OPERATION_TYPE_ADD = 2  # 注入资金
POOL_OPERATION_TYPE_REMOVE = 3  # 提取资金
POOL_OPERATION_TYPE_LIMIT = 4  # 限价单与资金池成交
POOL_OPERATION_TYPE_SWAP = 5  # swap单与资金池成交

# 交易发送状态
TX_STATUS_UNCONFIRMED = 1  # 未确认
TX_STATUS_SUCCESS = 2  # 成功
TX_STATUS_FAIL = 3  # 失败

```

## Markets

### 根据stock和money获取交易对信息

* method: `GET`
* path: `/api/eth/token/market`
* params:

name|Description|Type|Required
---|:---|:---|:---
`stock` | stock合约地址 | String | Yes 
`money` | money合约地址 | String | Yes
`is_only_swap` | 是否只有swap单：0-否, 1-是 | String | Yes 

* result:

name|Description|Type
---|:---|:---
`market_contract` | 市场合约地址 | String
`stock_contract` | stock合约地址 | String
`money_contract` | money合约地址 | String
`liquidity_stock` | stock的流动性 | String
`liquidity_money` | money的流动性 | String
`liquidity_usd` | 市场流动性对于usd市值 | String
`only_swap` | 否只有swap单 | Boolean

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

### 根据stock和money获取交易对列表

* method: `GET`
* path: `/api/eth/token/markets`
* params:

name|Description|Type|Required
---|:---|:---|:---
`stock` | stock合约地址 | String | Yes 
`money` | money合约地址 | String | Yes

* result:

name|Description|Type
---|:---|:---
`market_contract` | 市场合约地址 | String
`stock_contract` | stock合约地址 | String
`money_contract` | money合约地址 | String
`liquidity_stock` | stock的流动性 | String
`liquidity_money` | money的流动性 | String
`liquidity_usd` | 市场流动性对于usd市值 | String
`only_swap` | 否只有swap单 | Boolean

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

### 市场列表

* method: `GET`
* path: `/api/eth/market/list`
* params:

name|Description|Type|Required
---|:---|:---|:---
`others` | 是否是其它类型 1-是 0-不是或不传 | int | No
`money_contract` | money合约地址 | str | No


* result:

name|Description|Type
---|:---|:---
`name` | name | String
`market_contract` | 合约地址 | String 
`price` | 价格 | String 
`value` | 市值 | List 
`volume` | 成交额 | String 
`exchange_percent` | 变化百分比 | String 


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

### 市场交易信息

* method: `GET`
* path: `/api/eth/market/<contract>`
* params:

* result:

name|Description|Type
---|:---|:---
`stock_name` | name | String
`stock_symbol` | stock_symbol | String 
`stock_contract` | 合约地址 | String 
`money_name` | money_name | List 
`money_symbol` | money_symbol | String 
`market_contract` | 合约地址 | String 
`price` | price | String 
`high` | 24小时最高价 | String 
`low` | 24小时最低价 | String 
`open` | 开盘价 | String 
`close` | 收盘价 | String 
`amount` | 24小时成交量 | String 
`volume` | 24小时成交额 | String 
`active_address_num` | 24小时地址活跃数 | String 
`money_contract` | 合约地址 | String 
`only_swap` | 是否只能swap单, true是,false否 | String 


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

### 市场k线

* method: `GET`
* path: `/api/eth/market/<contract>/kline`
* params:

name|Description|Type|Required
---|:---|:---|:---
`interval` | 时间间隔选择 1.min 2.hour 3.day | String | Yes
`start_time` | 结束时间 | Int | Yes
`end_time` | 结束时间 | Int | Yes

* result:

name|Description|Type
---|:---|:---
`t` | timestamp | String
`o` | open | String 
`h` | high | String 
`l` | low | String 
`c` | close | String 
`a` | 成交量 | String 
`v` | 成交额 | String 


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

### 市场深度

* method: `GET`
* path: `/api/eth/market/<contract>/depth`
* params:

* result:

name|Description|Type
---|:---|:---
`ask` | 卖 | String
`p` | price | String
`a` | amount | String
`bid` | 买 | String 
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

### 市场订单薄

* method: `GET`
* path: `/api/eth/market/<contract>/order-book`
* params:

* result:

name|Description|Type
---|:---|:---
`ask` | 卖 | String
`price` | price | String
`amount` | amount | String
`bid` | 买 | String 
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

### 市场最新成交

* method: `GET`
* path: `/api/eth/market/<contract>/deals/latest`
* params:

* result:

name|Description|Type
---|:---|:---
`price` | price | String
`amount` | amount | String
`side` | 方向 1-买 2-卖| String 
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

### 当前委托

* method: `GET`
* path: `/api/eth/address/<address>/order/pending`
* params:

name|Description|Type|Required
---|:---|:---|:---
`side` | 下单方向 0-all 1-买 2-卖 | Int | Yes
`order_type` | 订单类型 0-all 1-swap 2-limit | Int | Yes
`start_time` | 开始时间 | Int | No
`end_time` | 结束时间 | Int | No
`market_contract` | market_contract | String | No
`search_key` | 搜索内容 | String | No
`page` | page | Int | Yes
`limit` | limit | Int | Yes

* result:

name|Description|Type
---|:---|:---
`timestamp` | timestamp | Int
`order_type` | 订单类型 1.swap 2.limit | String
`order_id` | 订单id | String
`deal_time` | 成交时间 | String
`name` | name | String
`market_contract` | 合约地址 | String
`side` | 方向 | String
`price` | 委托价格 | String
`amount` | 委托数量 | String
`deal_stock` | 成交数量 | String
`avg_price` | 成交均价 | String
`status` | 订单状态, 1-下单中, 2-已完成  | String


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

### 历史委托

* method: `GET`
* path: `/api/eth/address/<address>/order/history`
* params:

name|Description|Type|Required
---|:---|:---|:---
`side` | 下单方向 0-all 1-买 2-卖 | Int | Yes
`order_type` | 订单类型 0-all 1-swap 2-limit | Int | Yes
`start_time` | 开始时间 | Int | No
`end_time` | 结束时间 | Int | No
`order_status` | 隐藏撤销订单 0-不隐藏 3-已成交(隐藏)| Int | No
`market_contract` | market_contract | String | No
`search_key` | 搜索内容 | String | No
`page` | page | Int | Yes
`limit` | limit | Int | Yes

* result:

name|Description|Type
---|:---|:---
`timestamp` | timestamp | Int
`order_type` | 订单类型 1.swap 2.limit | String
`order_id` | 订单id | String
`deal_time` | 成交时间 | String
`name` | name | String
`market_contract` | 合约地址 | String
`side` | 方向 | String
`amount` | 委托数量 | String
`deal_stock` | 成交数量 | String
`deal_money` | 成交额 | String
`avg_price` | 成交均价 | String
`status` | 订单状态, 3-已成交, 4-已撤销  | String


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

### 成交记录

* method: `GET`
* path: `/api/eth/address/<address>/order/deals`
* params:

name|Description|Type|Required
---|:---|:---|:---
`side` | 下单方向 0-all 1-买 2-卖 | Int | Yes
`order_type` | 下单类型 0-all 1-swap 2-limit | Int | Yes
`start_time` | 开始时间 | Int | No
`end_time` | 结束时间 | Int | No
`market_contract` | market_contract | String | No
`page` | page | Int | Yes
`limit` | limit | Int | Yes

* result:

name|Description|Type
---|:---|:---
`timestamp` | timestamp | Int
`order_type` | 订单类型 1.swap 2.limit | String
`order_id` | 订单id | String
`name` | name | String
`market_contract` | 合约地址 | String
`side` | 订单方向 1-买 2-卖 | String
`price` | 价格 | String
`deal_stock` | 成交数量 | String
`deal_money` | 成交额 | String


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


### 订单成交明细

* method: `GET`
* path: `/api/eth/address/order/<order_id>/deals`
* params:

* result:

name|Description|Type
---|:---|:---
`timestamp` | timestamp | Int
`order_type` | 订单类型 1.swap 2.limit | String
`order_id` | 订单id | String
`name` | name | String
`market_contract` | 合约地址 | String
`side` | 订单方向 1-买 2-卖 | String
`price` | 价格 | String
`amount` | 成交数量 | String
`volume` | 成交额 | String


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

### 计算订单prevkey和id

* method: `GET`
* path: `/api/eth/transaction/prev-info`
* params:

name|Description|Type|Required
---|:---|:---|:---
`is_buy` | 订单是否为买单 | Bool | Yes 
`market_contract` | 合约地址 | String | Yes 
`price` | 价格 | String | Yes 
`amount` | 数量 | String | Yes 

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

### 删除单个订单信息

* method: `GET`
* path: `/api/eth/transaction/remove/info`
* params:

name|Description|Type|Required
---|:---|:---|:---
`order_id` | 订单id | String | Yes 
`market_contract` | 合约地址 | String | Yes 


* result:

name|Description|Type
---|:---|:---
`rm_list` | 删除列表 | String


```json
{
    "code": 0,
    "message": "success",
    "data": {
      "rm_list": []
    }
}
```

### 批量删除订单信息

* method: `GET`
* path: `/api/eth/transaction/remove/batch-info`
* params:

name|Description|Type|Required
---|:---|:---|:---
`market_contract` | 订单id | String | Yes 
`address` | 用户地址 | String | Yes 


* result:

name|Description|Type
---|:---|:---
`rm_list` | 删除列表 | String

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

### 兑换记录

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
`timestamp` | 时间戳 | Int


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

### 获取用户在某个市场的流动性

* method: `GET`
* path: `/api/eth/pool/<market_contract>/<address>/liquidity`
* params:

name|Description|Type|Required
---|:---|:---|:---
`market_contract` | 市场对合约地址 | String | Yes 
`address` | ethereum地址 | String | Yes 

* result:

name|Description|Type
---|:---|:---
`total_liquidity` | 市场对总的流动性 | String
`stock_liquidity` | stock流动性 | String
`money_liquidity` | money流动性 | String
`user_liquidity` | 查询的ethereum地址的流动性 | String

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

### 地址的市场流动性

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
`market_contract` | 合约地址 | String
`dst_token` | dst_token | String
`dst_amount` | dst_amount | String
`status` | 1-pending 3-finished | Int
`timestamp` | 时间戳 | Int


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

### 地址的流动性操作记录

* method: `GET`
* path: `/api/eth/address/<address>/pool/liquidity/history`
* params:

name|Description|Type|Required
---|:---|:---|:---
`page` | page  | Int | Yes
`limit` | limit  | Int | Yes
`operation_type` | 操作类型  | 0-all 1-add+create 3-remove 4-swap 5-limit | Yes


* result:

name|Description|Type
---|:---|:---
`stock` | stock | String
`money` | money | String
`market_contract` | 合约地址 | String
`operation_type` | 0-all 1-create 2-add 3-remove | Int
`stock_amount` | stock_amount | String
`money_amount` | money_amount | String
`value` | 价值 | String
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

### 资金池变动记录

* method: `GET`
* path: `/api/eth/pool/<contract>/transactions`
* params:

name|Description|Type|Required
---|:---|:---|:---
`page` | page  | Int | Yes
`limit` | limit  | Int | Yes
`operation_type` | 操作类型  | 0-all 1-add+create 3-remove 4-limit 5-swap | Yes

* result:

name|Description|Type
---|:---|:---
`src_token` | stock | String
`dst_token` | money | String
`operation_type` | 类型 1-add 2-create 3-remove 4-limit 5-swap | Int
`address` | 合约地址 | String
`value` | 价值 | String
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

## Transaction

### 广播交易

* method: `POST`
* path: `/api/eth/transaction/broadcast`
* params:

name|Description|Type|Required
---|:---|:---|:---
`raw_transaction` | 广播的交易(oneswap walllet用) | String | Yes

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
