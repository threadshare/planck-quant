## 目录

- [策略全局变量](#策略全局变量)
    + [exchange](#exchange)
    + [exchanges](#exchanges)
- [订单状态](#订单状态)
- [订单买卖类型](#订单买卖类型)
- [数据结构定义](#数据结构定义)
    + [Trade](#trade)
    + [Ticker](#ticker)
    + [Record](#record)
    + [Order](#order)
    + [MarketOrder](#marketorder)
    + [Depth](#depth)
    + [Account](#account)
- [行情 API](#行情-api)
  * [exchange.GetTicker()](#exchangegetticker)
  * [exchange.GetDepth()](#exchangegetdepth)
  * [exchange.GetTrades()](#exchangegettrades)
  * [exchange.GetRecords()](#exchangegetrecords)
  * [exchange.GetRate()](#exchangegetrate)
  * [exchange.GetUSDCNY()](#exchangegetusdcny)
- [交易 API](#交易-api)
  * [exchange.Buy(Price, Amount)](#exchangebuyprice-amount)
  * [exchange.Sell(Price, Amount)](#exchangesellprice-amount)
  * [exchange.CancelOrder(Id)](#exchangecancelorder-id)
  * [exchange.GetOrder(Id)](#exchangegetorder-id)
  * [exchange.GetOrders()](#exchangegetorders)
  * [exchange.SetPrecision(...)](#exchangesetprecision)
  * [exchange.SetRate(Rate)](#exchangesetraterate)
- [账户信息](#账户信息)
  * [exchange.GetAccount()](#exchangegetaccount)
  * [exchange.GetCurrency()](#exchangegetcurrency)
  * [exchange.GetQuoteCurrency()](#exchangegetquotecurrency)
- [TA - 常用指标优化库](#ta---常用指标优化库)
  * [MACD, 指数平滑异同平均线](#macd-指数平滑异同平均线)
  * [KDJ, 随机指标](#kdj-随机指标)
  * [RSI, 强弱指标](#rsi-强弱指标)
  * [ATR, 平均真实波幅](#atr-平均真实波幅)
  * [OBV, 能量潮](#obv-能量潮)
  * [MA, 移动平均线](#ma-移动平均线)
  * [EMA, 指数平均数指标](#ema-指数平均数指标)
  * [BOLL, 布林带](#boll-布林带)
  * [Highest, 周期最高价](#highest-周期最高价)
  * [Lowest, 周期最低价](#lowest-周期最低价)

### 策略全局变量

##### exchange

可视为一个交易所对象，默认为策略参数中添加的第一个交易所，所有与交易所的数据交互，都通过这个对象里面的函数实现。

##### exchanges

`exchange` 的数组，包含多个交易所对象，`exchanges[0]` 即是 `exchange`。

### 订单状态

`Order` 结构中的 `Status` 属性。

`Order` 结构中的 `Status` 属性。

| 常量名               | 定义                 | 值   |
| :------------------- | :------------------- | :--- |
| ORDER_STATE_PENDING  | 未完成               | 0    |
| ORDER_STATE_CLOSED   | 已经完成             | 1    |
| ORDER_STATE_CANCELED | 已经取消             | 2    |
| ORDER_STATE_UNKNOWN  | 未知状态（其它状态） | 3    |

**ORDER_STATE_UNKNOWN** 状态，可以调用 `exchange.GetRawJSON()` 获取原始订单状态信息，查询交易所文档，查看具体描述。



### 订单买卖类型

| 常量名          | 定义 | 值   |
| :-------------- | :--- | :--- |
| ORDER_TYPE_BUY  | 买单 | 0    |
| ORDER_TYPE_SELL | 卖单 | 1    |



### 数据结构定义

部分函数会附带在调用时请求返回的原始 JSON 信息，该原始 JSON 数据储存在返回对象的 **Info** 属性中，以下是各个数据结构的主要属性描述。



##### Trade



获取所有交易历史 (非自己), 由 `exchange.GetTrades()` 函数返回。



```javascript
{
    Time    : 1567736576000,    // 时间(Unix timestamp 毫秒)
    Price   : 1000,             // 价格          
    Amount  : 1,                // 数量
    Type    : 0                 // 订单类型,参考常量里的订单类型,0即为ORDER_TYPE_BUY，ORDER_TYPE_BUY的值为0
}
```



##### Ticker

市场行情由 `exchange.GetTicker()` 函数返回。

```javascript
{
    Info    : {...},             // rest Request 请求交易所接口后，交易所接口应答的原始数据
    High    : 1000,              // 最高价
    Low     : 500,               // 最低价
    Sell    : 900,               // 卖一价
    Buy     : 899,               // 买一价
    Last    : 900,               // 最后成交价
    Volume  : 10000000,          // 最近成交量
    Time    : 1567736576000      // 毫秒级别时间戳
}
```



##### Record

标准 OHLC 结构，用来画 K 线和指标分析用的，由 `exchange.GetRecords()` 函数返回此结构数组。

```javascript
{
    Time    : 1567736576000,     // 一个时间戳,精确到毫秒，与Javascript的new Date().getTime()得到的结果格式一样
    Open    : 1000,              // 开盘价
    High    : 1500,              // 最高价
    Low     : 900,               // 最低价
    Close   : 1200,              // 收盘价
    Volume  : 1000000            // 交易量
}
```



##### Order

订单结构，由 `exchange.GetOrder()`，`exchange.GetOrders()` 函数返回。

```javascript
{
    Info        : {...},         // rest Request 请求交易所接口后，交易所接口应答的原始数据
    Id          : 123456,        // 交易单唯一标识
    Price       : 1000,          // 下单价格
    Amount      : 10,            // 下单数量
    DealAmount  : 10,            // 成交数量
    AvgPrice    : 1000,          // 成交均价，注意，有些交易所不提供该数据，不提供的设置为0
    Status      : 1,             // 订单状态,参考常量里的订单状态ORDER_STATE_CLOSED
    Type        : 0,             // 订单类型,参考常量里的订单类型ORDER_TYPE_BUY
    Offset      : 0              // 数字货币期货和商品期货的订单数据中，订单的开平仓方向，ORDER_OFFSET_OPEN为开仓，ORDER_OFFSET_CLOSE为平仓方向
}
```

##### MarketOrder

市场深度单，即 `exchange.GetDepth()` 返回数据中 **Bids**、**Asks** 数组中的元素的数据结构。

```javascript
{
    Price   : 1000,              // 价格
    Amount  : 1                  // 数量
}
```

##### Depth

市场深度，由 `exchange.GetDepth()` 函数返回。

```javascript
{
    Asks    : [...],             // 卖单数组,MarketOrder数组,按价格从低向高排序
    Bids    : [...],             // 买单数组,MarketOrder数组,按价格从高向低排序
    Time    : 1567736576000      // 毫秒级别时间戳
}
```



##### Account

账户信息，由 `exchange.GetAccount()` 函数返回。

```javascript
{
    Info            : {...},     // rest Request 请求交易所接口后，交易所接口应答的原始数据
    Balance         : 1000,      // 余额(人民币或者美元,在Poloniex交易所里ETC_BTC这样的品种,Balance就指的是BTC的数量,Stocks指的是ETC数量)
    FrozenBalance   : 0,         // 冻结的余额
    Stocks          : 1,         // BTC/LTC数量,数字货币现货为当前可操作币的余额(去掉冻结的币),数字货币期货的话为合约当前可用保证金(传统期货无此属性)
    FrozenStocks    : 0          // 冻结的BTC/LTC数量(传统期货无此属性)
}
```

### 行情 API

主要的行情接口函数：

| 函数名     | 说明                                               |
| :--------- | :------------------------------------------------- |
| GetTicker  | [获取 tick 行情](#exchangegetticker)               |
| GetRecords | [获取 K 线数据](#exchangegetrecord)                |
| GetDepth   | [获取订单薄数据 (订单深度数据)](#exchangegetdepth) |
| GetTrades  | [获取市场上最近交易记录](#exchangegettrades)       |

#### exchange.GetTicker()

`exchange.GetTicker()`，获取市场当前行情，返回值:`Ticker` 结构体。

[`Ticker` 结构体](#Ticker)

如果没有行情推送过来时，`exchange.GetTicker()` 函数会阻塞，等待行情推送。
`exchange.GetDepth()`、`exchange.GetTrades()`、`exchange.GetRecords()` 同理。如果不希望阻塞，可以使用**切换行情模式**：

- [exchange.IO](http://exchange.io/)("mode", 0)
  立即返回模式，如果当前还没有接收到交易所最新的行情数据推送，就立即返回旧的行情数据，如果有新的数据就返回新的数据。
- [exchange.IO](http://exchange.io/)("mode", 1)
  缓存模式 (默认模式), 如果当前还没有收到交易所最新的行情数据 (同上一次接口获取的数据比较), 就等待接收然后再返回，如果调用该函数之前收到了最新的行情数据，就立即返回最新的数据。
- [exchange.IO](http://exchange.io/)("mode", 2)
  强制更新模式，进入等待一直到接收到交易所下一次的最新推送数据后返回。



#### exchange.GetDepth()

`exchange.GetDepth()`，获取交易所订单薄。返回值:`Depth` 结构体。

`Depth` 结构体包含两个结构体数组，分别是 `Asks[]` 和 `Bids[]`，Asks 和 Bids 包含以下结构体变量:

| 数据类型 | 变量名 | 说明 |
| :------- | :----- | :--- |
| number   | Price  | 价格 |
| number   | Amount | 数量 |

- [`Depth` 结构体](#depth)



#### exchange.GetTrades()

`exchange.GetTrades()`，获取交易所交易历史（非自己）。返回值:`Trade` 结构体数组。部分交易所不支持，具体返回的数据是多少范围内的成交记录，因交易所而定，需要根据具体情况处理。
返回数据是一个数组，其中每个元素的时间顺序和 `exchange.GetRecords` 函数的返回数据顺序一致，即：数组最后一个元素是距离当前时间最近的数据。

- [`Trade` 结构体](#trade)



#### exchange.GetRecords()

`exchange.GetRecords(Period)`，返回一个 K 线历史，K 线周期在创建时指定，如果在调用 `exchange.GetRecords()` 函数时指定了参数，获取的就是按照该参数周期的 K 线数据，如果没有参数，按照开始参数上设置的 K 线周期或者回测页面设置的 K 线周期返回 K 线数据。参数值:**PERIOD_M1** 指 1 分钟，**PERIOD_M5** 指 5 分钟，**PERIOD_M15** 指 15 分钟，**PERIOD_M30** 指 30 分钟，**PERIOD_H1** 指 1 小时，**PERIOD_D1** 指一天。返回值：Record 结构体数组。K 线数据，会随时间累积，最多累积到 2000 根，然后会更新加入一根，同时删除一根最早时间的 K 线柱（如队列进出）。部分交易所没有提供 K 线接口，按照情况系统实时收集数据生成 K 线。

`Period` 参数设置 5，即为请求 5 秒为周期的 K 线数据。
如果 `Period` 参数不能被 60 整除（即代表的周期是不可用分钟为单位的周期），系统底层则使用 `GetTrades` 的相关接口获取成交记录数据，合成所需的 K 线数据。
如果 `Period` 参数能被 60 整除，则使用 1 分钟 K 线数据，合成所需的 K 线数据。

- [`Record` 结构体](#record)

注意：
`exchange.GetRecords()` 函数获取 K 线数据有 2 种情况：

- 1、交易所提供了 K 线数据接口，这种情况，获取的数据是交易所直接返回的数据。
- 2、交易所没有提供 K 线数据接口，需要底层在每次用户调用 `exchange.GetRecords()` 时获取交易所最近成交记录，即 `exchange.GetTrades()` 函数，合成 K 线数据。

#### exchange.GetRate()

`exchange.GetRate()`，返回交易所使用的流通货币与当前显示的计价货币的汇率，返回 1 表示禁用汇率转换。返回值:number 类型。

注意：

- 如果没有调用 `exchange.SetRate()` 设置过转换汇率，`exchange.GetRate()` 默认返回的汇率值是 1，即当前显示的计价货币没有发生过汇率转换。
- 如果使用 `exchange.SetRate()` 设置过一个汇率值，例如 7，那么当前 `exchange` 这个交易所对象代表的交易所的流通货币的行情、深度、下单价格等等所有价格信息，都会被乘以设置的汇率 7，进行转换。
- 例如 `exchange` 是以美元为计价货币的交易所，调用 `exchange.SetRate(7)` 后，机器人所有价格都会被乘 7 转换成接近 CNY 的价格。此时使用 `exchange.GetRate()` 获取的汇率值就是 7。

#### exchange.GetUSDCNY()

`exchange.GetUSDCNY()`，返回美元最新汇率 (yahoo 提供的数据源) 或 OKEX 期货合约使用的美元汇率，返回值:number 类型。

注意：

- 当交易所对象 `exchange` 为**非 OKEX 期货交易所对象**时：返回的是 yahoo 提供的美元汇率。
- 当交易所对象 `exchange` 为 **OKEX 期货交易所对象**时：返回的是 OKEX 交易所设定的美元汇率，可能和 yahoo 提供的数据，或者一些外汇数据上的美元汇率不一致。



### 交易 API

以下函数均可通过 `exchange` 或 `exchanges[0]` 对象调用，例如：`exchange.Sell(100, 1);` 在交易所下一个价格为 100，数量为 1 的卖单。

#### exchange.Buy(Price, Amount)

`exchange.Buy(Price, Amount)`，下买单，返回一个订单 ID。参数值：price 为订单价格，number 类型，Amount 为订单数量，number 类型。返回值：string 类型或数值类型（具体类型根据各个交易所返回类型而定）。

返回订单编号，可用于查询订单信息和取消订单。



#### exchange.Sell(Price, Amount)

`exchange.Sell(Price, Amount)`，下卖单，返回一个订单 ID。参数值：price 为订单价格，number 类型，Amount 为订单数量，number 类型。返回值：string 类型或数值类型（具体类型根据各个交易所返回类型而定）。

返回订单编号，可用于查询订单信息和取消订单。



#### exchange.CancelOrder(Id)

`exchange.CancelOrder(orderId)`，取消某个订单，参数值：**orderid** 为订单编号，string 类型或数值类型（具体类型根据各个交易所下单时返回类型而定），返回值：bool 类型。

返回操作结果，true 表示取消订单请求发送成功，false 取消订单请求发送失败（只是发送请求成功，交易所是否取消订单最好调用 `exchange.GetOrders()` 查看）。



#### exchange.GetOrder(Id)

`exchange.GetOrder(orderId)`，根据订单号获取订单详情，参数值:**orderid** 为要获取的订单号，string 类型或数值类型。（具体类型根据各个交易所返回类型而定）返回值:`Order` 结构体。
（部分交易所不支持）

- [`Order` 结构体](#order)
- `AvgPrice`，成交均价（有些交易所不支持该字段，不支持则设置 0）。



#### exchange.GetOrders()

`exchange.GetOrders()`，获取所有未完成的订单。返回值:`Order` 结构体数组。
order 结构体可参考 `exchange.GetOrder()` 函数说明。



#### exchange.SetPrecision(...)

`exchange.SetPrecision(PricePrecision, AmountPrecision)`，设置价格与品种下单量的小数位精度，设置后会自动截断，参数值：PricePrecision 为 number 类型，用来控制价格后面的小数点位，AmountPrecision 为 number 类型，用来控制数量后面的小数点位。PricePrecision 和 AmountPrecision 都必须是整型 number。



#### exchange.SetRate(Rate)

`exchange.SetRate(Rate)`，设置交易所的流通货币的汇率，参数值：`Rate` 为 `number` 类型。返回值：为 `number` 类型。

注意：

- 如果使用 `exchange.SetRate()` 设置过一个汇率值，例如 7，那么当前 `exchange` 这个交易所对象代表的交易所的流通货币的行情、深度、下单价格等等所有价格信息，都会被乘以设置的汇率 7，进行转换。
- 例如 exchange 是以美元为计价货币的交易所，`exchange.SetRate(7)` 后，机器人所有价格都会被乘 7 转换成接近 CNY 的价格。



### 账户信息

#### exchange.GetAccount()

`exchange.GetAccount()`，将返回交易所账户信息。返回值:`Account` 结构结构体。

- [`Account` 结构体](#account)

#### exchange.GetCurrency()

`exchange.GetCurrency()`，返回交易所操作的货币对名称如 LTC_BTC, 返回值:string 类型。

#### exchange.GetQuoteCurrency()

`exchange.GetQuoteCurrency()`。返回交易所操作的基础货币名称，例如 BTC_CNY 就返回 CNY,ETH_BTC 就返回 BTC。返回值:string 类型。



### TA - 常用指标优化库

#### MACD, 指数平滑异同平均线

`TA.MACD(数据, 快周期, 慢周期, 信号周期)`，默认参数为 (12, 26, 9), 返回二维数组，分别是 `[DIF, DEA, MACD]`。



#### KDJ, 随机指标

`TA.KDJ(数据, 周期1, 周期2, 周期3)`，默认参数为 (9, 3, 3), 返回二维数组，分别是 `[K, D, J]`。



#### RSI, 强弱指标

`TA.RSI(数据, 周期)`，默认参数为 14, 返回一个一维数组。



#### ATR, 平均真实波幅

`TA.ATR(数据, 周期)`，ATR (数据，周期), 默认参数为 14, 返回一个一维数组。



#### OBV, 能量潮

`TA.OBV(数据)`，返回一个一维数组。



#### MA, 移动平均线

`TA.MA(数据, 周期)`，MA (数据，周期), 默认参数为 9, 返回一个一维数组。



#### EMA, 指数平均数指标

`TA.EMA(数据, 周期)`，指数平均数指标。默认参数为 9, 返回一个一维数组。



#### BOLL, 布林带

`TA.BOLL(数据, 周期, 乘数)`，BOLL (数据，周期，乘数)，布林线指标，默认参数为 (20, 2) 返回一个二维数组 `[上线, 中线, 下线]`。



#### Highest, 周期最高价

`TA.Highest(数据, 周期, 属性)`，返回最近周期内的最大值 (不包含当前 Bar)，如 `TA.Highest(records, 30, 'High')`，如果周期为 0 指所有，如属性不指定则视数据为普通数组，返回一个价格（数值类型）。



#### Lowest, 周期最低价

`TA.Lowest(数据, 周期, 属性)`，返回最近周期内的最小值 (不包含当前 Bar)，如 `TA.Lowest(records, 30, 'Low')`，如果周期为 0 指所有，如属性不指定则视数据为普通数组。返回一个价格（数值类型）。







