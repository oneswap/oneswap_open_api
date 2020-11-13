# OneSwap Transactions API 

## Contents 

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

- ERC20 transfer 
- ERC20 ABI
- ***function transfer(address _to, uint256 _value);***

| Parameter   | Type            | Remark                            |
| ------ | --------------- | --------------------------------- |
| _to    | address         | recipient's address                     |
| _value | uint256(string) | transfer amount (precision is based on token decimals) |



### approve

- Authorize an address to conduct operation on specified amount of its own account 
- ERC20/OneSwapPair ABI
- ***function approve(address _spender, uint256 _value);***

| Parameter     | Type            | Remark                                                |
| -------- | --------------- | --------------------------------------------------- |
| _spender | address         | approve address                                   |
| _value   | uint256(string) | approve amount, int('f' * 64, 16) is the approvable maximum amount |

- Applications:
  1. When placing swap and limit order, as a buying order, money is not ETH and money approved to contract address is smaller than order value; or, as a selling order, stock is not ETH and stock approved to contract address is smaller than order value.
  2. When adding or creating liquidity, stock or money is not ETH and one of them approved to contract address is smaller than the added liquidity (If both of stock and money satisfy the above conditions, then they all need to be approved).
  3. When removing liquidity, consider that trading pair as a ERC20 token and the value approved to contract address is smaller than the removed liquidity.



## Swap

### swapToken

- Swap order (When using ETH to buy/sell, the corresponding token should be `0x0000000000000000000000000000000000000000`, and the buying/selling ETH amount is required to be transmitted to the transaction value (token decimals: 18))
- OneSwapRouter ABI
- ***function swapToken(address token, uint amountIn, uint amountOutMin, address[] calldata path, address to, uint deadline);***

| Parameter         | Type      | Remark                                               |
| ------------ | --------- | -------------------------------------------------- |
| token        | address   | contract address used for swap                            |
| amountIn     | int       | token amount used for swap (precision is based on token decimals)  |
| amountOutMin | int       | minimum token amount that user expects to swap, calculate based on the slippage     |
| path         | address[] | router (trading pairs) swap router (trading pairs)                               |
| to           | address   | address to receive the swapped token                            |
| deadline     | int       | effective duration of this transaction, transaction that is packaged after the deadline will be failed  |

- Notes:
  1. When token used for swap is ETH, token parameter should be `0x0000000000000000000000000000000000000000`, and buying/selling ETH amount is required to be transmitted to the transaction value (token decimals: 18). 
  2. When token used for swap is ERC20, the amount that ERC20 approved to router contract should be equivalent to or larger than amountIn, otherwise that ERC20 is required to be re-approved to router contract. 




## Limit

### limitOrder

- Limit Order (When using ETH to buy/sell, the buying/selling ETH amount is required to be transmitted to the transaction value (token decimals: 18))
- OneSwapRouter ABI
- ***function limitOrder(bool isBuy, address pair, uint prevKey, uint price, uint32 id, uint stockAmount, uint deadline);***

| Parameter        | Type    | Remark                                                     |
| ----------- | ------- | -------------------------------------------------------- |
| isBuy       | bool    | buy order or not                                          |
| pair        | address | contract address of trading pair                                      |
| prevKey     | int     | in the order book, the first three orders' IDs before this order (inquire via OpenAPI) |
| price       | int     | price (conversion function is as follows)                                |
| id          | int     | on-chain ID (inquire via OpenAPI)                                  |
| stockAmount | int     | stock amount (token decimals: 4)                               |
| deadline    | int     | effective duration of this transaction, transaction that is packaged after the deadline will be failed      |

- price conversion function(Python3):

  ```python
  from decimal import Decimal
  
  
  def build_price(significant: int, exponent: int) -> int:
      """construct Price data"""
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
      """OneSwap analyzes Price data"""
      price = int(price_hex, 16)
      exponent = int(price >> 27)
      significant = Decimal(price & ((1 << 27) - 1))
      return significant * (Decimal('10') ** Decimal(exponent - 23))
  
  >>> parser_price(hex(2159829326))		# 2159829326 => 1.2345678
  >>> parser_price(hex(2294047054))		# 2294047054 => 12.345678
  >>> parser_price(hex(2428264782))		# 2428264782 => 123.45678
  >>> parser_price(hex(2025611598))		# 2025611598 => 0.12345678
  ```

- Notes: 
  
  1. When using ETH to buy/sell, buying/selling ETH amount is required to be transmitted to the transaction value (token decimals: 18).
  2. When using ERC20 to sell/buy, the amount that ERC20 is approved to router contract should be equivalent to amountIn or above, otherwise ERC20 should be re-approved to router contract. 
  3. Price parameter should be converted into int type value according to the above conversion function.
  4. privKey and id parameters can be inquired via OpenAPI's `/api/eth/transaction/prev-info`.




### removeOrders

- Batch delete orders in order book
- OneSwapPair ABI `directly call the contract with corresponding trading pair to send transaction`
- ***function removeOrders(uint[] calldata rmList);***

| Parameter   | Type      | Remark                              |
| ------ | --------- | --------------------------------- |
| rmList | uint256[] | rmList is data combined bv Buy/id/positionID |

- rmList data structure:

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

- Note: 
  1. You can inquire rmList via OpenAPI's `/api/eth/transaction/remove/info` and `/api/eth/transaction/remove/batch-info`.



## Pool

### addLiquidity

- Add liquidity to a certain trading pair (trading pair will be automatically created if it does not existed), when creating trading pair with ETH, the corresponding token should be `0x0000000000000000000000000000000000000000`, the to-be-spent ETH amount is required to be transmitted to the transaction value (token decimals: 18))

- OneSwapRouter ABI
- ***function addLiquidity(address stock, address money, bool isOnlySwap, uint amountStockDesired, uint amountMoneyDesired, uint amountStockMin, uint amountMoneyMin, address to, uint deadline);***

| Parameter               | Type    | Remark                                             |
| ------------------ | ------- | -------------------------------------------------- |
| stock              | address | stock of trading pair                                      |
| money              | address | money of trading pair                                      |
| isOnlySwap         | bool    | swap only or not                            |
| amountStockDesired | int     | stock entered by user (precision is based on stock decimals)     |
| amountMoneyDesired | int     | money entered by user (precision is based on money decimals)       |
| amountStockMin     | int     | minimum stock that user expects to add to uniswap pool           |
| amountMoneyMin     | int     | minimum money that user expects to add to uniswap pool            |
| to                 | address | address to receive liquidity                            |
| deadline           | int     | effective duration of this transaction, transaction that is packaged after the deadline will be failed  |

- Notes:
  1. When trading pair's stock or money is ETH, the corresponding parameter should be `0x0000000000000000000000000000000000000000`, and the to-be-spent ETH amount is required to be transmitted to the transaction value (token decimals: 18)
  2. When trading pair's stock or money is ERC20, the amount that ERC20 is approved to router contract should be equivalent to or above amountStockDesired / amountMoneyDesired, otherwise ERC20 should be re-approved to router contract (If both stock and money are ERC20, then both of them should be approved).
3. When adding liquidity, amountStockDesired / amountMoneyDesired should be the liquidity proportion of stock and money of current liquidity pool.




### removeLiquidity

- Removing liquidity of trading pair can get corresponding stock and money (trading pair contract should be approved to router contract first)
- OneSwapRouter ABI
- ***function removeLiquidity(address pair, uint liquidity, uint amountStockMin, uint amountMoneyMin, address to, uint deadline);***

| Parameter        | Type    | Remark                                              |
| -------------- | ------- | -------------------------------------------------- |
| pair           | address | contract address of trading pair                                     |
| liquidity      | int     | liquidity amount removed (18 decimals)               |
| amountStockMin | int     | minimum stock that user expects to get                         |
| amountMoneyMin | int     | minimum money that user expects to get                        |
| to             | address | address to receive liquidity                             |
| deadline       | int     | effective duration of this transaction, transaction packaged after the deadline will be failed|

- Notes: 
  1. The amount that this trading pair contract approved to router contract should be equivalent to or larger than liquidity. 
2. User can inquire the removable liquidity of this trading pair via OpenAPI's `/api/eth/pool/<market_contract>/<address>/liquidity`.

