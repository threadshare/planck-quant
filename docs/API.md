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

| 函数名     | 说明                                                         |
| :--------- | :----------------------------------------------------------- |
| GetTicker  | [获取 tick 行情](#exchange.GetTicker())                      |
| GetRecords | [获取 K 线数据](javascript:gotoElement('user-content-exchange.getrecords')) |
| GetDepth   | [获取订单薄数据 (订单深度数据)](javascript:gotoElement('user-content-exchange.getdepth')) |
| GetTrades  | [获取市场上最近交易记录](javascript:gotoElement('user-content-exchange.gettrades')) |

#### exchange.GetTicker()

`exchange.GetTicker()`，获取市场当前行情，返回值:`Ticker` 结构体。

[`Ticker` 结构体](#Ticker)