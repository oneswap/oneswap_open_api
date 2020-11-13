# OneSwap有关交易发送说明

## 目录

* [Transfer](#Transfer)
  - [transfer](#transfer)

  - [approve](#approve)

* [Swap](#Swap)
  * [swapToken](#swapToken)
  
* [Limit](#Limit)
  * [limitOrder](#limitOrder)
  * [removeOrders](#removeOrders)
  
* [Pool](#Pool)
  * [addLiquidity](#addLiquidity)
  * [removeLiquidity](#removeLiquidity)
  



## Transfer

### transfer

- ERC20转账
- ERC20 ABI
- ***function transfer(address _to, uint256 _value);***

| 参数   | 类型            | 说明                              |
| ------ | --------------- | --------------------------------- |
| _to    | address         | 转账接收地址                      |
| _value | uint256(string) | 转账金额(以Token的decimals为精度) |



### approve

- 授权某个地址操作自身账户指定金额的权限
- ERC20/OneSwapPair ABI
- ***function approve(address _spender, uint256 _value);***

| 参数     | 类型            | 说明                                                |
| -------- | --------------- | --------------------------------------------------- |
| _spender | address         | approve的目标地址                                   |
| _value   | uint256(string) | approve的金额, int('f' * 64, 16)为可approve的最大值 |

- 使用场景:
  1. 下Swap和Limit单时，作为买单且Money不为ETH且Money approve给合约地址的金额小于下单金额或作为卖单且Stock不为ETH且Stock approve给合约地址的金额小于下单金额时
  3. Add or Create Liquidity时，Stock或Money不为ETH且approve给合约地址的金额小于添加的Liquidity时(Stock和Money都符合上述条件时则都需要approve)
  4. Remove Liquidity时，将该交易对看作一个ERC20 Token且approve给合约地址的金额小于提取的流动性代币时
  



## Swap

### swapToken

- Swap单(使用ETH买/卖时对应的Token应填为`0x0000000000000000000000000000000000000000`, 且需要在value中传入买/卖的ETH数额(以10^18为精度))
- OneSwapRouter ABI
- ***function swapToken(address token, uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline);***

| 参数         | 类型      | 说明                                               |
| ------------ | --------- | -------------------------------------------------- |
| token        | address   | 用来兑换的Token合约地址                            |
| amountIn     | int       | 用来兑换的Token数额(以该Token自身decimals为精度)   |
| amountOutMin | int       | 期望兑换到的最少的目标Token数额, 根据滑点计算      |
| path         | address[] | 兑换路径(交易对数组)                               |
| to           | address   | 接收兑换得到的Token地址                            |
| deadline     | int       | 本交易的有效时间, 超过deadline后才打包的交易将失败 |

- 注意事项:
  1. 用来Swap的Token为ETH时，token参数填为`0x0000000000000000000000000000000000000000`，且需要在交易的value中传入买/卖的ETH数额(10^18精度)
  2. 用来Swap的Token为ERC20时，该ERC20 approve给Router合约的金额需要大于等于amountIn，否则需要重新将该ERC20 approve给Router合约




## Limit

### limitOrder

- Limit单(使用ETH买/卖时需要在value中传入买/卖的ETH数额(以10^18为精度))
- OneSwapRouter ABI
- ***function limitOrder(bool isBuy, address pair, uint prevKey, uint price, uint32 id, uint stockAmount, uint deadline);***

| 参数        | 类型    | 说明                                                     |
| ----------- | ------- | -------------------------------------------------------- |
| isBuy       | bool    | 订单是否为买单                                           |
| pair        | address | 订单交易对合约地址                                       |
| prevKey     | int     | 在订单薄链表中, 位于该订单前面的3个订单的ID(OpenAPI查询) |
| price       | int     | 订单价格(换算函数见下述)                                 |
| id          | int     | 订单链上ID(OpenAPI查询)                                  |
| stockAmount | int     | 购买的Stock数量(10^-4精度)                               |
| deadline    | int     | 本交易的有效时间, 超过deadline后才打包的交易将失败       |

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

- 注意事项:
  
  1. 使用ETH卖/买时需要在交易的value中传入买/卖的ETH数额(10^18精度)
  2. 使用ERC20卖/买时，该ERC20 approve给Router合约的金额需要大于等于amountIn，否则需要重新将该ERC20 approve给Router合约
  3. price参数需要根据上述换算函数转换为int类型值
  4. privKey和id参数可以通过OpenAPI的`/api/eth/transaction/prev-info`接口查询




### removeOrders

- 批量删除OrderBook中的订单
- OneSwapPair ABI `直接调用对应交易对合约发送交易`
- ***function removeOrders(uint[] calldata rmList);***

| 参数   | 类型      | 说明                              |
| ------ | --------- | --------------------------------- |
| rmList | uint256[] | isBuy/id/positionID拼接起来的数据 |

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

- 注意事项:
  1. 可以通过OpenAPI的`/api/eth/transaction/remove/info`与`/api/eth/transaction/remove/batch-info`接口查询rmList



## Pool

### addLiquidity

- 给某个交易对添加流动性(交易对不存在时自动创建)(建立与ETH的交易对时对应的Token应填为`0x0000000000000000000000000000000000000000`, 且需要在value中传入需要花费的ETH数额(以10^18为精度))

- OneSwapRouter ABI
- ***function addLiquidity(address stock, address money, bool isOnlySwap, uint amountStockDesired, uint amountMoneyDesired, uint amountStockMin, uint amountMoneyMin, address to, uint deadline);***

| 参数               | 类型    | 说明                                               |
| ------------------ | ------- | -------------------------------------------------- |
| stock              | address | 交易对的Stock                                      |
| money              | address | 交易对的Money                                      |
| isOnlySwap         | bool    | 交易对是否只支持Swap                               |
| amountStockDesired | int     | 用户输入的stock数量(以Stock的decimals为精度)       |
| amountMoneyDesired | int     | 用户输入的money数量(以Money的decimals为精度)       |
| amountStockMin     | int     | 用户期望最少添加进uniswap池子的stock数量           |
| amountMoneyMin     | int     | 用户期望最少添加进uniswap池子的money数量           |
| to                 | address | 接收流动性token的地址                              |
| deadline           | int     | 本交易的有效时间, 超过deadline后才打包的交易将失败 |

- 注意事项:
  1. 交易对的Stock或Money为ETH时对应的参数应填为`0x0000000000000000000000000000000000000000`, 且需要在交易的value中传入需要花费的ETH数额(10^18精度)
  2. 交易对的Stock或Money为ERC20时，该ERC20 approve给Router合约的金额需要大于等于amountStockDesired或amountMoneyDesired，否则需要重新将该ERC20 approve给Router合约(Stock和Money均为ERC20时都需要approve检查)
3. 添加流动性时amountStockDesired / amountMoneyDesired应为当前资金池中Stock与Money的流动性比例




### removeLiquidity

- 移除交易对的流动性代币获得对应的Stock和Money(需要先将交易对合约approve给路由合约)
- OneSwapRouter ABI
- ***function removeLiquidity(address pair, uint liquidity, uint amountStockMin, uint amountMoneyMin, address to, uint deadline);***

| 参数           | 类型    | 说明                                               |
| -------------- | ------- | -------------------------------------------------- |
| pair           | address | 交易对合约地址                                     |
| liquidity      | int     | 移除的交易对流动性代币数量(18位精度)               |
| amountStockMin | int     | 用户期望获得的stock最小数量                        |
| amountMoneyMin | int     | 用户期望获得的money最小数量                        |
| to             | address | 接收流动性token的地址                              |
| deadline       | int     | 本交易的有效时间, 超过deadline后才打包的交易将失败 |

- 注意事项:
  1. 该交易对合约 approve给Router合约的金额需要大于等于liquidity
2. 用户在该交易对中可提取的liquidity可以通过OpenAPI的`/api/eth/pool/<market_contract>/<address>/liquidity`接口查询

