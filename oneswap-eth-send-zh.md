# OneSwao有关交易发送说明

## 目录

* [统一预估Gas接口](#统一预估Gas接口)

* [Transfer](#Transfer)

  - [ethTransfer](#ethTransfer)
  - [transfer](#transfer)

  - [approve](#approve)

* [Swap](#Swap)
  
  * [swapToken](#swapToken)
  
* [Limit](#Limit)
  * [limitOrder](#limitOrder)
  * [removeOrder](#removeOrder)
  * [removeOrders](#removeOrders)
  
* [Pool](#Pool)
  
  * [addLiquidity](#addLiquidity)
  * [removeLiquidity](#removeLiquidity)
  
* [Gov](#Gov)
  * [vote](#vote)

- [Locksend](#Locksend)
  - [LockSend](#LockSend)
  - [unlock](#unlock)

## 统一预估Gas接口

- POST: `/res/transaction/preparation`
- 参数:

| Name             | Description                                                  | Type   | Required |
| ---------------- | ------------------------------------------------------------ | ------ | -------- |
| tx_type          | 交易类型(见下述交易类型说明)                                 | string | yes      |
| contract_address | 合约地址                                                     | string | yes      |
| from_address     | 发送地址                                                     | string | yes      |
| data             | 内容 | list   | yes       |

- 交易类型说明:

| Name            | Description                                    |
| --------------- | ---------------------------------------------- |
| ethTransfer     | ETH转账                                        |
| transfer        | ERC20转账                                      |
| approve         | 授权交易类型                                   |
| swapToken       | ERC20 Swap ERC20 或 ERC20 Swap ETH             |
| limitOrder      | ERC20 Limit ERC20 或 ERC20 Limit ETH           |
| removeOrder     | 删除OrderBook中的单个订单                      |
| removeOrders    | 批量删除OrderBook中的订单                      |
| addLiquidity    | 给某个交易对添加流动性(交易对不存在时自动创建) |
| removeLiquidity | 移除交易对的流动性代币获得对应的Stock和Money   |
| vote            | 提案投票                                       |
| lockSend        | 锁定转账                                       |
| unlock          | 解锁到期的锁定转账                             |

- body: (各种交易类型的body结构见下述具体交易说明)
- return:

```json
{
  "code": 0,
  "message": "success",
  "data": {
    "nonce": 14,
    "gas": 136890
  }
}
```



## Transfer

### ethTransfer

- ERC20转账
- contract_address参数传入0x0000000000000000000000000000000000000000

| 参数   | 类型            | 说明                              |
| ------ | --------------- | --------------------------------- |
| _to    | address         | 转账接收地址                      |
| _value | uint256(string) | 转账金额(以Token的decimals为精度) |

- 预估Gas接口body:

```json
[
	"0xd3CdA913deB6f67967B99D67aCDFa1712C293601",
	"1000"
]
```



### transfer

- ERC20转账
- ERC20 ABI
- ***function transfer(address _to, uint256 _value);***

| 参数   | 类型            | 说明                              |
| ------ | --------------- | --------------------------------- |
| _to    | address         | 转账接收地址                      |
| _value | uint256(string) | 转账金额(以Token的decimals为精度) |

- 预估Gas接口body:

```json
[
  "0xff36AFfa3A36581Ef55a126854F049543f874Ae5",
  "100000000000000000000"
]
```



### approve

- 授权某个地址操作自身账户指定金额的权限
- ERC20/OneSwapPair ABI
- ***function approve(address _spender, uint256 _value);***

| 参数     | 类型            | 说明                                   |
| -------- | --------------- | -------------------------------------- |
| _spender | address         | approve的目标地址                      |
| _value   | uint256(string) | approve的金额(设定为int('f' * 64, 16)) |

- 使用场景:
  1. 下Swap和Limit单时，作为买单且Money不为ETH且Money approve给合约地址的金额小于下单金额或作为卖单且Stock不为ETH且Stock approve给合约地址的金额小于下单金额时
  2. Swap时，From Token不为ETH且From Token approve给合约地址的金额小于下单金额时
  3. Add or Create Liquidity时，Stock或Money不为ETH且approve给合约地址的金额小于添加的Liquidity时(Stock和Money都符合上述条件时则都需要approve)
  4. Remove Liquidity时，将该交易对看作一个ERC20 Token且approve给合约地址的金额小于提取的流动性代币时

- 接口支持:
  1. +根据交易对合约和用户地址查询该用户在这个交易对合约、Stock、Money approve给路由合约的金额
  2. +查询各个合约地址(路由合约、回购合约等)

- 预估Gas接口body:

```json
[
  "0xff36AFfa3A36581Ef55a126854F049543f874Ae5",
  "115792089237316195423570985008687907853269984665640564039457584007913129639935"
]
```



## Swap

### swapToken

- Swap单(使用ETH买/卖时对应的Token应填为`0x0000000000000000000000000000000000000000`, 且需要在value中传入买/卖的ETH数额(以10^18为精度))
- OneSwapRouter ABI
- ***function swapToken(address token, uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline);***

| 参数         | 类型        | 说明                                                         |
| ------------ | ----------- | ------------------------------------------------------------ |
| token        | address     | 用来兑换的Token合约地址                                      |
| amountIn     | int(string) | 用来兑换的Token数额(以该Token自身decimals为精度)             |
| amountOutMin | int(string) | 期望兑换到的最少的目标Token数额(建议填0)                     |
| path         | address[]   | 兑换路径(交易对数组)                                         |
| to           | address     | 接收兑换得到的Token地址                                      |
| deadline     | int(string) | 本交易的有效时间(暂时写死1min: (int(time.time()) + 60) * 100) |

- 使用场景:
  1. Trade页Swap功能用于兑换的Token不为ETH时使用本函数完成Swap交易
  2. Swap页From Token不为ETH时使用本函数完成Swap交易

- 接口支持:
  1. *需要添加查询交易对Stock和Money的decimals信息
  2. +Swap页需要查询From Token到To Token之间的兑换路径

- 预估Gas接口body:

```json
[
  "0xE01162FEaCBBE25C027334464b9a7B8C88F0ca2B",
  "100000000000",
  "0",
  ["0x0eBD94ad8FfD1754cA012BaC214f1E494D590f1A"],
  "0x91aF9599B84880e5aCbb4E425f9c1826B0e1D687",
  "159833693300"
]
```



## Limit

### limitOrder

- Limit单(使用ETH买/卖时对应的Token应填为`0x0000000000000000000000000000000000000000`, 且需要在value中传入买/卖的ETH数额(以10^18为精度))
- OneSwapRouter ABI
- ***function limitOrder(bool isBuy, address pair, uint prevKey, uint price, uint32 id, uint stockAmount, uint deadline);***

| 参数        | 类型        | 说明                                                         |
| ----------- | ----------- | ------------------------------------------------------------ |
| isBuy       | bool        | 订单是否为买单                                               |
| pair        | address     | 订单交易对合约地址                                           |
| prevKey     | int(string) | 在订单薄链表中, 位于该订单前面的3个订单的ID(后端提供接口查询) |
| price       | int(string) | 订单价格(换算函数见下述)                                     |
| id          | int(string) | 订单i链上ID(后端提供接口查询)                                |
| stockAmount | int(string) | 购买的Stock数量(10^-4精度)                                   |
| deadline    | int(string) | 本交易的有效时间(暂时写死1min: (int(time.time()) + 60) * 100) |

- price换算函数(Python3):

  ```python
  from decimal import Decimal
  
  
  def build_price(significant: int, exponent: int) -> int:
      """构建Price数据"""
      if not (10000000 <= significant <= 99999999):
          return 0
      if not (-16 <= exponent <= 15):
          return 0
      return int(((exponent + 16) << 27) | significant)
  
  >>> build_price(12345678, 0)	# 1.2345678 => 2159829326
  >>> build_price(12345678, 1)	# 12.345678 => 2294047054
  >>> build_price(12345678, 2)	# 123.45678 => 2428264782
  >>> build_price(12345678, -1)	# 0.12345678 => 2025611598
  
  
  def parser_price(price_hex: str) -> Decimal:
      """OneSwap解析Price数据"""
      price = int(price_hex, 16)
      exponent = int(price >> 27)
      significant = Decimal(price & ((1 << 27) - 1))
      return significant * (Decimal('10') ** Decimal(exponent - 23))
  
  >>> parser_price(hex(2159829326))		# 2159829326 => 1.2345678
  >>> parser_price(hex(2294047054))		# 2294047054 => 12.345678
  >>> parser_price(hex(2428264782))		# 2428264782 => 123.45678
  >>> parser_price(hex(2025611598))		# 2025611598 => 0.12345678
  ```

- 使用场景:
  
  1. Trade页Limit功能用于兑换的Token不为ETH时使用本函数完成Swap交易

- 接口支持:
  
1. +需要提供计算prevKey与id的接口

- 预估Gas接口body:

```json
[
  true,
  "0x0eBD94ad8FfD1754cA012BaC214f1E494D590f1A",
  "0",
  "2159829326",
  "0",
  "100000",
  "159833693300"
]
```



### removeOrder

- 删除OrderBook中的单个订单`(建议统一使用removeOrders)`
- OneSwapPair ABI `直接调用对应交易对合约发送交易`
- ***function removeOrder(bool isBuy, uint32 id, uint72 positionID);***

| 参数       | 类型        | 说明                                                         |
| ---------- | ----------- | ------------------------------------------------------------ |
| isBuy      | bool        | 订单是否为买单                                               |
| id         | int(string) | 订单链上ID                                                   |
| positionID | int(string) | 在订单薄链表中, 位于该订单前面的3个订单的ID(后端提供接口查询) |

- 使用场景:
  1. 删除挂单中的订单

- 接口支持:
  1. /remove/info

- 预估Gas接口body:

```json
[
  true,
  "1",
  "0"
]
```



### removeOrders

- 批量删除OrderBook中的订单
- OneSwapPair ABI `直接调用对应交易对合约发送交易`
- ***function removeOrders(uint[] calldata rmList);***

| 参数   | 类型              | 说明                                        |
| ------ | ----------------- | ------------------------------------------- |
| rmList | uint256(string)[] | isBuy/id/positionID拼接起来的数据(下文介绍) |

- 使用场景:
  1. 批量删除挂单中的订单

- 接口支持:
  1. /remove/batch-info

- 预估Gas接口body:

```json
[
  ["513", "2199023256577", "618970093429669730822652161", "512", "1237940094625613595539408129", "2199023255808", "36893492545465615105"]
]
```

- rmList数据结构:

```javascript
function removeOrders(uint[] calldata rmList) external override lock {
  for(uint i = 0; i < rmList.length; i++) {
    uint rmInfo = rmList[i];
    bool isBuy = uint8(rmInfo) != 0;
    uint32 id = uint32(rmInfo>>8);
    uint72 prevKey = uint72(rmInfo>>40);
    _removeOrder(isBuy, id, prevKey);
  }
}
```



## Pool

### addLiquidity

- 给某个交易对添加流动性(交易对不存在时自动创建)(建立与ETH的交易对时对应的Token应填为`0x0000000000000000000000000000000000000000`, 且需要在value中传入需要花费的ETH数额(以10^18为精度))

- OneSwapRouter ABI
- ***function addLiquidity(address stock, address money, bool isOnlySwap, uint amountStockDesired, uint amountMoneyDesired, uint amountStockMin, uint amountMoneyMin, address to, uint deadline);***

| 参数               | 类型        | 说明                                                         |
| ------------------ | ----------- | ------------------------------------------------------------ |
| stock              | address     | 交易对的Stock                                                |
| money              | address     | 交易对的Money                                                |
| isOnlySwap         | bool        | 交易对是否只支持Swap                                         |
| amountStockDesired | int(string) | 用户输入的stock数量(以Stock的decimals为精度)                 |
| amountMoneyDesired | int(string) | 用户输入的money数量(以Money的decimals为精度)                 |
| amountStockMin     | int(string) | 用户期望最少添加进uniswap池子的stock数量(建议直接写成0)      |
| amountMoneyMin     | int(string) | 用户期望最少添加进uniswap池子的money数量(建议直接写成0)      |
| to                 | address     | 接收流动性token的地址                                        |
| deadline           | int(string) | 本交易的有效时间(暂时写死1min: (int(time.time()) + 60) * 100) |

- 使用场景:
  1. 给交易对资金池添加流动性且该交易对的Stock和Money均不是ETH时
  2. 创建交易对且该交易对的Stock和Money均不是ETH时

- 接口支持:
  1. +需要添加查询某个交易对的资金池总流动性Token数量、总Stock数量、总Money数量以及某个用户在这个资金池中的流动性Token数量的接口
- 预估Gas接口body:

```json
[
  "0xE01162FEaCBBE25C027334464b9a7B8C88F0ca2B",
  "0x6Bf2d07B26b2665Be7116CAafb64f063eBf49BC2",
  false,
  "500000000000",
  "500000000000000000000",
  "0",
  "0",
  "0x91aF9599B84880e5aCbb4E425f9c1826B0e1D687",
  "159833693300"
]
```



### removeLiquidity

- 移除交易对的流动性代币获得对应的Stock和Money(需要先将交易对合约approve给路由合约)
- OneSwapRouter ABI
- ***function removeLiquidity(address pair, uint liquidity, uint amountStockMin, uint amountMoneyMin, address to, uint deadline);***

| 参数           | 类型        | 说明                                                         |
| -------------- | ----------- | ------------------------------------------------------------ |
| pair           | address     | 交易对合约地址                                               |
| liquidity      | int(string) | 移除的交易对流动性代币数量(18位精度)                         |
| amountStockMin | int(string) | 用户期望获得的stock最小数量(建议直接写成0)                   |
| amountMoneyMin | int(string) | 用户期望获得的money最小数量(建议直接写成0)                   |
| to             | address     | 接收流动性token的地址                                        |
| deadline       | int(string) | 本交易的有效时间(暂时写死1min: (int(time.time()) + 60) * 100) |

- 使用场景:
  1. 减少交易对资金池流动性且该交易对的Stock和Money均不是ETH时

- 接口支持:
  1. +需要添加查询某个交易对的资金池总流动性Token数量、总Stock数量、总Money数量以及某个用户在这个资金池中的流动性Token数量的接口

- 预估Gas接口body:

```json
[
  "0x0eBD94ad8FfD1754cA012BaC214f1E494D590f1A",
  "100000000000000000000",
  "0",
  "0",
  "0x91aF9599B84880e5aCbb4E425f9c1826B0e1D687",
  "159833693300"
]
```



## Gov

### vote

- 提案投票(同时只有一个有效提案)(未投过时新增投票、已投过且投票选项未修改时追加投票、已投过且投票选项修改时修改投票)(需要先将ones approve到GOV合约地址)
- OneSwapGov ABI
- ***function vote(uint8 opinion, uint112 voteAmt);***

| 参数    | 类型        | 说明                         |
| ------- | ----------- | ---------------------------- |
| opinion | int(string) | 投票选项(1: Yes/ 2: No)      |
| voteAmt | int(string) | 投票票数(以ones为精度(18位)) |

- 使用场景:
  1. 提案未投票时对该提案发起投票操作

- 预估Gas接口body:

```json
[
  "0",
  "1000000000000000000"
]
```



## Locksend

### LockSend

- 锁定转账(需要提前将Token approve给锁仓合约)

- OneSwapLocksend ABI
- ***function lockSend(address to, uint amount, address token, uint32 unlockTime) external;***

| 参数       | 类型        | 说明                                                      |
| ---------- | ----------- | --------------------------------------------------------- |
| to         | address     | 转账接收地址                                              |
| amount     | int(string) | 转账金额(* Token精度)                                     |
| token      | address     | 转账的ERC20 Token                                         |
| unlockTime | int(string) | 以小时为单位, 表示从1970年1月1日0时0分0秒起至现在的小时数 |

- 使用场景:
  1. 锁定转账时

- 预估Gas接口Body:

```json
[
  "0x8f0288BaB864C1206475c4EA9ABa5B79A09E4d58",
  "100000000000000000000",
  "0xA451f8062A3aad9A6779EF7866078759B5951B00",
  444825
]
```



### unlock

- 解锁到期的锁定转账
- OneSwapLocksend ABI
- function unlock(address from, address to, address token, uint32 unlockTime) external;

| 参数       | 类型        | 说明                                                      |
| ---------- | ----------- | --------------------------------------------------------- |
| from       | address     | 转账发送地址                                              |
| to         | address     | 转账接收地址                                              |
| token      | address     | 转账的ERC20 Token                                         |
| unlockTime | int(string) | 以小时为单位, 表示从1970年1月1日0时0分0秒起至现在的小时数 |

- 使用场景:
  1. 解锁到期的锁定转账时

- 预估Gas接口Body:

```json
[
  "0x8f0288BaB864C1206475c4EA9ABa5B79A09E4d58",
  "0x8f0288BaB864C1206475c4EA9ABa5B79A09E4d58",
  "0xA451f8062A3aad9A6779EF7866078759B5951B00",
  444824
]
```

