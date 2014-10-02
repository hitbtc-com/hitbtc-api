# HitBTC API Guide

## Table of contents
* [SUMMARY](#summary)<br>
  — [Currency symbols](#cursymbols)
* [RESTFUL API](#restful)<br>
  — [Market data](#marketrestful)<br>
  — [Trading](#tradingrestful)<br>
  — [Payment](#paymentsrestful)
* [SOCKET.IO API](#socketio)<br>
  — [Market Data](#socketio)
* [STREAMING API](#streaming)<br>
  — [Market data](#marketstreaming) <br>
  — [Trading](#tradingstreaming)<br>
  — [Sample code](#sample)

# <a name="summary"/>Summary

This document provides the complete reference for [HitBTC](https://hitbtc.com) API.

HitBTC API has several interfaces to implement them in a custom software:
* <b>RESTful API</b> that allows: 
  - access to the market data: get ticker, order book, trades, etc. See [Market data RESTful API](#marketrestful)
  - performing trading operations: get trading balance, place or cancel orders, get history, etc. See [Trading RESTful API](#tradingrestful)
  - managing funds: get balance of the main account, transfer funds between main and trading accounts, create an outgoing  transactions, etc. See [Payment RESTful API](#paymentsrestful)
* <b>socket.io</b> protocol for receiving the market data. socket.io protocol supports WebSocket and xhr-polling  transport. See [socket.io Market Data](#socketio)
* <b>Streaming API</b> based on WebSocket protocol to get an access to: 
  - market data. See [Market data streaming end-point](#marketstreaming)  
  - trading operations. See [Trading streaming end-point](#tradingstreaming)

Trading and payment operations require user's authentication: each request or message should have a signature. 
You should get your API key and Secret key on the [Settings](https://hitbtc.com/settings) page. See details in [RESTful API authentication](#authenticationrestful) and [WebSocket API authentication](#authenticationwebsocket).

### <a name="cursymbols"/>Currency symbols

The following currency symbols are traded on HitBTC exchange.

| Symbol	| Lot size | Price step
| --- | --- | --- |
| BTCUSD |	0.01 BTC |	0.01 |
| BTCEUR |	0.01 BTC |	0.01 |
| LTCBTC | 0.1 LTC |	0.00001 |
| LTCUSD | 0.1 LTC |	0.001 |
| LTCEUR | 0.1 LTC |	0.001 |
| EURUSD | 1 EUR |	0.0001 |
| DOGEBTC | 1000 DOGE |	0.000000001 |
| XMRBTC | 0.01 XMR |	0.01 |
| BCNBTC | 100 BCN | 0.000000001 |
| XDNBTC | 100 XDN | 0.000000001 |

The actual list of symbols can be obtained by [/api/1/public/symbols](#symbols) method.

Size representation:
* Size values in streaming messages are represented in lots.
* Size values in RESTful market data are represented in money (e.g. in coins or in USD). 
* Size values in RESTful trade are represented in lots (e.g. 1 means 0.01 BTC for BTCUSD)

# <a name="restful"/>RESTful API

RESTful API provides the most functional access to HitBTC facilities.
RESTful API allows: 
  - access to the market data: get ticker, order book, trades, etc. See [Market data RESTful API](#marketrestful)
  - performing trading operations: get trading balance, place or cancel orders, get history, etc. See [Trading RESTful API](#tradingrestful)
  - managing funds: get balance of the main account, transfer funds between main and trading accounts, create an outgoing  transactions, etc. See [Payment RESTful API](#paymentsrestful)

Endpoint URL: [http://api.hitbtc.com](http://api.hitbtc.com).

HitBTC provides a <b>demo trading</b> option.  You can enable demo mode and acquire demo API keys on the  [Settings](https://hitbtc.com/settings) page.<br>Demo endpoint address: [http://demo-api.hitbtc.com](http://demo-api.hitbtc.com)

Trading and payment operations require [authentication](#authentication). See also [error codes](#errors) and [reports representing order status changes](#reports).

## <a name="marketrestful"/>Market data RESTful API

RESTful API provides access to the market data with following methods:
  - get the timestamp- [/api/1/public/time](#time)
  - get the list of symbols - [/api/1/public/symbols](#symbols)
  - get the ticker for specified symbol - [/api/1/public/:symbol/ticker](#ticker)
  - get all tickers - [/api/1/public/ticker](#alltickers)
  - get the order book for specified symbol - [/api/1/public/:symbol/orderbook](#orderbook)
  - get individual trades data for specified symbol - [/api/1/public/:symbol/trades](#trades)
  - get recent trades for specified symbol  - [/api/1/public/:symbol/trades/recent](#recenttrades)


### <a name="time"/>/api/1/public/time

<i>Summary:</i> returns the server time in UNIX timestamp format

<i>Request:</i> `GET /api/1/public/time`

<i>Example:</i> `/api/1/public/time`

<i>Example response:</i>
``` json
{
    "timestamp": 1393492619000
}
```

### <a name="symbols"/>/api/1/public/symbols

<i>Summary:</i> returns the actual list of currency symbols traded on HitBTC exchange, their lot sizes (`lot`  parameter) and price step (`step` parameter).

<i>Request:</i> `GET /api/1/public/symbols`

<i>Example:</i> `/api/1/public/symbols`

<i>Example response:</i>
``` json
{
    "symbols": [
        {
            "symbol": "BTCUSD",
            "step": "0.01",
            "lot": "0.01",
        },
        {
            "symbol": "BTCEUR",
            "step": "0.01",
            "lot": "0.01",
        },
        ...
    ]
}
```

### <a name="ticker"/>/api/1/public/:symbol/ticker

<i>Summary:</i> returns the actual data on exchange rates of the specified cryptocurrency.

Sample usage at HitBTC site: see [https://hitbtc.com/market-overview](https://hitbtc.com/market-overview) for each line.

<i>Request:</i> `GET /api/1/public/:symbol/ticker`
  where `:symbol` is a currency symbol traded on HitBTC exchange (see [Currency symbols](#cursymbols))

<i>Example:</i> `/api/1/public/BTCUSD/ticker`

<i>Example response:</i>
``` json
{
    "last": "550.73",
    "bid": "549.56",
    "ask": "554.12",
    "high": "600.1",
    "low": "400.7",
    "volume": "567.9",
    "timestamp": 1393492619000
}
```

The following fields are used in the `ticker` object:

| Field | Description |
| --- | --- |
| last  | Last price |
| bid | Highest buy order |
| ask | Lowest sell order |
| high | Highest trade price per last 24h + last incomplete minute | 
| low | Lowest trade price per last 24h + last incomplete minute |
| volume | Volume per last 24h + last incomplete minute |
| timestamp | Server time in UNIX timestamp format |


### <a name="alltickers"/>/api/1/public/ticker

<i>Summary:</i> returns the actual data on exchange rates for all traded cryptocurrencies - all tickers.

<i>Example:</i> `/api/1/public/ticker`

<i>Example response:</i>
``` json
{
    "BCNBTC": {
        "ask": "0.000000039",
        "bid": "0.000000036",
        "last": "0.000000037",
        "low": "0.000000035",
        "high": "0.000000048",
        "volume": "580790000",
        "volume_quote": "26.000000000",
        "timestamp": 1409907025743
    },
    "BTCEUR": {
        "ask": "378.23",
        "bid": "376.28",
        "last": "376.29",
        "low": "362.48",
        "high": "382.13",
        "volume": "961.71",
        "volume_quote": "361328.13",
        "timestamp": 1409907025743
    },
    "BTCUSD": {
        "ask": "489.26",
        "bid": "488.08",
        "last": "489.31",
        "low": "478.01",
        "high": "496.23",
        "volume": "404.20",
        "volume_quote": "197735.83",
        "timestamp": 1409907025743
    }
    ....
}
```

### <a name="orderbook"/>/api/1/public/:symbol/orderbook

<i>Summary:</i> returns a list of open orders for specified currency symbol: their prices and sizes.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), <b>Order book</b> tab.

<i>Request:</i> `GET /api/1/public/:symbol/orderbook`
  where `:symbol` is a currency symbol traded on HitBTC exchange (see [Currency symbols](#cursymbols))

<i>Parameters:</i>

| Parameter | Required | Type | Description |
| --- | --- | --- | --- |
| `format_price` | No | `string` or `number` | Format of prices returned: as a string (default) or as a number| 
| `format_amount` | No | `string` or `number` | Format of amount returned: as a string (default) or as a number|
| `format_amount_unit` | No | `currency` or `lot` | Units of amount returned: in currency units (default) or in lots|

<i>Alias:</i> `/api/1/request/:symbol/orderbook.json` -> `/api/1/public/:symbol/orderbook?format_price=number&format_amount=number`

<i>Example:</i> `/api/1/public/BTCUSD/orderbook`

<i>Example response:</i>
``` json
{
    "asks": [
        [ "405.71", "0.09" ],
        [ "406.65", "0.06" ],
        [ "409.51", "0.15" ],
        [ "413.93", "51.6" ],
        [ "414.59", "47.1" ]
    ],
    "bids": [
        [ "398.3", "0.15" ],
        [ "396.99", "0.13" ],
        [ "395", "0.5" ],
        [ "391.93", "42.4" ],
        [ "383.67", "145.4" ]
    ]
}
```

<i>Example:</i> `/api/1/public/BTCUSD/orderbook?format_price=number&format_amount=number`

<i>Example response:</i>
``` json
{
    "asks": [
        [ 405.71, 0.09 ],
        [ 406.65, 0.06 ],
        [ 409.51, 0.15 ],
        [ 413.93, 51.6 ],
        [ 414.59, 47.1 ]
    ],
    "bids": [
        [ 398.3, 0.15 ],
        [ 396.99, 0.13 ],
        [ 395, 0.5 ],
        [ 391.93, 42.4 ],
        [ 383.67, 145.4 ]
    ]
}
```

### <a name="trades"/>/api/1/public/:symbol/trades

<i>Summary:</i> returns data on trades for specified currency symbol in specified ID or timestamp interval.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), <b>Market trades</b> tab.

<i>Request:</i> `GET /api/1/public/:symbol/trades`
  where `:symbol` is a currency symbol traded on HitBTC exchange (see [Currency symbols](#cursymbols))

<i>Parameters:</i>

| Parameter | Reqired | Type | Description | 
| --- | --- | --- | --- |
| `from` | Yes | integer | Returns trades with `trade_id` > specified `trade_id` (if `by=trade_id`) or returns trades with `timestamp` >= specified `timestamp` (if `by=ts`)|
| `till` | No | integer | Returns trades with `trade_id` < specified `trade_id` (if `by=trade_id`) or returns trades with `timestamp` < specified `timestamp` (if `by=ts`)| 
| `by` | Yes | `trade_id` or `ts`| Selects if filtering and sorting is performed by trade_id or by timestamp
| `sort` | No| `asc` or `desc`| Trades are sorted ascending (default) or descending|
| `start_index` | Yes| integer | Start index for the query, zero-based |
| `max_results` | Yes| integer | Maximum quantity of returned items, at most 1000|
| `format_item` | No | `array` or `object` | Format of items returned: as an array (default) or as a list of objects |
| `format_price` | No | `string` or `number`| Format of prices returned: as a string (default) or as a number |
| `format_amount` | No | `string` or `number` | Format of amount returned: as a string (default) or as a number  |
| `format_amount_unit` | No | `currency` or `lot` | Units of amount returned: in currency units (default) or in lots|
| `format_tid` | No | `string` or `number`| Format of trade ID returned: as a string or as a number (default)|
| `format_timestamp` | No | `millisecond` or `second` | Format of trade timestamp returned: in milliseconds (default) or in seconds|
| `format_wrap `| No | `true` or `false` | Selects if the line wrappnig is used in item returned. Default value - `true`|
| `side` | No | `true` or `false` | Selects if the side of a trade is returned|

<i>Alias:</i>
```
/api/1/request/:symbol/trades.json?since=<trade_id> -> /api/1/public/:symbol/trades?from=<trade_id>&by=trade_id&start_index=0&format_numbers=number&format_tradeid=string&format_objects=object&format_timestamp=second
```

<i>Example:</i>
```
/api/1/public/BTCUSD/trades?from=0&by=trade_id&sort=desc&start_index=0&max_results=100
```
<i>Example response:</i>

``` json
{
    "trades": [
        [ 3814483, "575.64", "0.02", 1393492619000 ],
        [ 3814482, "574.3", "0.12", 1393492619000 ],
        [ 3814481, "573.67", "3.8", 1393492619000 ],
        [ 3814479, "571", "0.01", 1393492619000 ],
        ...
    ]
}
```


<i>Example:</i>

```
/api/1/public/BTCUSD/trades?from=0&by=trade_id&sort=desc&start_index=0&max_results=100&format_item=object&format_price=number&format_amount=number&format_tid=string&format_timestamp=second&format_wrap=false
```
<i>Example response:</i>

``` json
[
    {"date": 1393492619, "price": 575.64, "amount": 0.02, "tid": "3814483"},
    {"date": 1393492619, "price": 574.3, "amount": 0.12, "tid": "3814482"},
    {"date": 1393492619, "price": 573.67, "amount": 3.8, "tid": "3814481"},
    {"date": 1393492619, "price": 571, "amount": 0.01, "tid": "3814479"},
    ...
]
```

### <a name="recenttrades"/>/api/1/public/:symbol/trades/recent

<i>Summary:</i> returns recent trades for the specified currency symbol.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), <b>Market trades</b> tab. Select the currency pair at top of the tab.

<i>Request:</i> `/api/1/public/:symbol/trades/recent`
  where `:symbol` is a currency symbol traded on HitBTC exchange (see [Currency symbols](#cursymbols))

<i>Parameters:</i>

| Parameter | Required | Type | Description |
| --- | --- | --- | --- |
| `max_results` | Yes | integer | Maximum quantity of returned results, at most 1000 |
| `format_item` | No | `array` or `object` | Format of items returned: as an array (default) or as a list of objects|
| `side` | No | `true` or `false` | Selects if the side of a trade is returned|

<i>Example:</i> 
```
/api/1/public/LTCEUR/trades/recent?max_results=100&format_item=object
```

<i>Example response:</i>

``` json
{"trades":[
  {"date":1411045690003,"price":"442.12","amount":"0.09","tid":1413901,"side":"buy"},
  {"date":1411045690003,"price":"442.02","amount":"0.60","tid":1413900,"side":"buy"},
  {"date":1411045690003,"price":"441.57","amount":"0.19","tid":1413899,"side":"buy"},
  {"date":1411045635556,"price":"442.00","amount":"0.42","tid":1413889,"side":"buy"},
  {"date":1411045635556,"price":"441.59","amount":"0.19","tid":1413888,"side":"buy"},
  {"date":1411045635556,"price":"441.59","amount":"0.39","tid":1413887,"side":"buy"},
  {"date":1411045621329,"price":"442.00","amount":"0.01","tid":1413882,"side":"buy"},
  {"date":1411045621329,"price":"442.00","amount":"0.32","tid":1413881,"side":"buy"},
  {"date":1411045621329,"price":"441.57","amount":"0.28","tid":1413880,"side":"buy"},
  {"date":1411045621329,"price":"441.56","amount":"0.38","tid":1413879,"side":"buy"}
]
```

## <a name="tradingrestful"/>Trading RESTful API

RESTful API allows to perform trading operations with the following methods:
  - get the trading balance - [/api/1/trading/time](#tradingbalance)
  - get all active orders  - [/api/1/trading/active](#active)
  - place a new order - [/api/1/trading/new_order](#neworder)
  - cancel an order - [/api/1/trading/cancel_order](#cancelorder)
  - get user's recent orders - [/api/1/trading/recent](#recentorders)
  - get user's trading history - [/api/1/trading/trades](#usertrades)

Trading operations require [authentication](#authentication).

[Error codes](#errors) and [reports representing order status changes](#reports) are described below.


### <a name="authenticationrestful"/>Authentication

RESTful Trading API requires HMAC-SHA512 signatures for each request.

To use this API endpoint you should get your API key and Secret key from the [Settings](https://hitbtc.com/settings) page. 

Each request should include the following parameters:

| Parameter | Required | Type | Description |
| --- | --- | --- | --- |
| `nonce` | Yes | query string parameter, less than (2^53-1) | Unique monotonous number that should be generated on the client. Hint: use millisecond or microsecond timestamp | 
| `apikey` | Yes | query string parameter | API key from [Settings](https://hitbtc.com/settings) page|
| `signature` | Yes | _lower-case_ hex representation of hmac-sha512 of concatenated `uri` and `postData` | This parameter should be added as a HTTP header `X-Signature`| 

Signature generation pseudo-code:

```
uri = path + '?' + query (example: /api/1/trading/orders/active?nonce=1395049771755&apikey=f6ab189hd7a2007e01d95667de3c493d&symbols=EURUSD)
message = uri + postData
signature = lower_case(hex(hmac_sha512(message, secret_key)))
```

Javascript code (example):
``` js
   var crypto = require('crypto');

   ...

   var signature = crypto.createHmac('sha512', secretKey).update(message).digest('hex');
```

Useful examples provided by the community:
* C# example code: https://gist.github.com/hitbtc-com/9808530
* PHP example code: https://gist.github.com/hitbtc-com/10885873 

### <a name="errors"/>Error codes

Trading RESTful API can return the following errors:

| HTTP code | Text | Description |
| --- | --- | --- |
| 403 | Invalid API key | API key doesn't exist or API key is currently used on another endpoint (max last 15 min) |
| 403 | Nonce has been used | Nonce is not monotonous |
| 403 | Nonce is not valid | Too big number or not a number |
| 403 | Wrong signature | Specified signature is not correct|

### <a name="reports"/>Execution reports

The API uses `ExecutionReport` as an object that represents change of order status.

The following fields are used in `ExecutionReport` object:

| Field	| Required | Type | Description |
| --- | --- | --- | --- |
| `orderId` | Yes | string | Order ID on the Exchange|
| `clientOrderId` | Yes | string | `clientOrderId` sent in `NewOrder` message (see [/api/1/trading/new_order](#neworder)) |
| `execReportType` | Yes | `new` <br> `canceled` <br> `rejected` <br> `expired` <br> `trade` <br> `status` | Execution report type |
| `orderStatus` | Yes | `new` <br> `partiallyFilled` <br> `filled` <br> `canceled` <br> `rejected` <br> `expired` | Order status |
| `orderRejectReason` | Yes - for orders in `rejected` state| `unknownSymbol` <br> `exchangeClosed` <br>`orderExceedsLimit` <br> `unknownOrder` <br> `duplicateOrder` <br> `unsupportedOrder` <br> `unknownAccount` <br> `other`|Reason of rejection. Relevant only for orders in rejected state |
| `symbol` | Yes | string, e.g. `BTCUSD` | Currency symbol traded on HitBTC exchange (see [Currency symbols](#cursymbols))|
| `side` | Yes| `buy` or `sell` | Side of a trade |
| `timestamp` | No | integer | UTC timestamp in milliseconds |
| `price` | No | decimal | Price of an order|
| `quantity` | Yes | integer | Order quantity in lots |
| `type` | Yes| `limit` | Type of an order. Only `limit` orders are currently supported |
| `timeInForce` | Yes | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill<br>`DAY` - day orders< | Time in force |
| `tradeId` | Yes - for trades| integer | Trade ID on the exchange |
| `lastQuantity` | Yes - for trades | integer | Last quantity |
| `lastPrice` | Yes - for trades | decimal | Last price  |
| `leavesQuantity` | No | integer |  Leaves quantity|
| `cumQuantity` | No | integer | Cumulative quantity |
| `averagePrice` | No | decimal | Average price. Equals 0 if `cumQuantity`=0|

<i>Example response:</i>

``` json
{ "ExecutionReport": { 
   "orderId": "5852103",
   "clientOrderId": "11111112",
   "execReportType": "new",
   "orderStatus": "new",
   "symbol": "BTCUSD",
   "side": "buy",
   "timestamp": 1395236779235,
   "price": 0.1,
   "quantity": 100,
   "type": "limit",
   "timeInForce": "GTC",
   "lastQuantity": 0,
   "lastPrice": 0,
   "leavesQuantity": 100,
   "cumQuantity": 0,
   "averagePrice": 0 
} }
```


### <a name="tradingbalance"/>/api/1/trading/balance

<i>Summary:</i> returns trading balance.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), the upper panel, <i>Trading</i> line. The black number displays total trade balance of the currency (`cash` parameter), the gray number is amount reserved against unexecuted orders and unfinished transactions (`reserved` parameter).

<i>Request:</i> `GET /api/1/trading/balance`

<i><i>Parameters:</i></i> no parameters

<i>Example response:</i>
``` json
{"balance": [
  {"currency_code": "BTC","cash": 0.045457701,"reserved": 0.01},
  {"currency_code": "EUR","cash": 0.0445544,"reserved": 0},
  {"currency_code": "LTC","cash": 0.7,"reserved": 0.1},
  {"currency_code": "USD","cash": 2.9415029,"reserved": 1.001}
]}
```

### <a name="active"/>/api/1/trading/orders/active

<i>Summary:</i> returns all orders in status `new` or `partiallyFilled`.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), <b>My orders</b> tab, <b>Active</b> group.

<i>Request:</i> `GET /api/1/trading/orders/active`

<i>Parameters:</i> 

| Parameter | Required | Type | Description |
| --- | --- | --- | --- |
| `symbols` | No | string | Comma-separated list of symbols. Default - all symbols|

<i>Example response:</i>

``` json
{"orders": [
  {
    "orderId": "51521638",
    "orderStatus": "new",
    "lastTimestamp": 1394798401494,
    "orderPrice": 1000,
    "orderQuantity": 1,
    "avgPrice": 0,
    "quantityLeaves": 1,
    "type": "limit",
    "timeInForce": "GTC",
    "cumQuantity": 0,
    "clientOrderId": "7fb8756ec8045847c3b840e84d43bd83",
    "symbol": "LTCBTC",
    "side": "sell",
    "execQuantity": 0
  }
]}
```

### <a name="neworder"/>/api/1/trading/new_order

<i>Summary:</i> place a new order. Returns a JSON object `ExecutionReport` that respresent a status of the order.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), <b>Sell Order</b> and <b>Buy Order</b> panels.

<i>Request:</i> `POST /api/1/trading/new_order`

<i>Parameters:</i>

| Parameter | Required| Type | Description |
| --- | --- | --- | --- |
| `clientOrderId` | Yes | string | Unique order ID generated by client. From 8 to 32 characters |
| `symbol` | Yes| string | Currency symbol traded on HitBTC exchange (see [Currency symbols](#cursymbols)), e.g. `BTCUSD` |
| `side` | Yes | `buy` or `sell`| Side of a trade |
| `price` | Yes - for limit orders | decimal | Order price |
| `quantity` | No | integer | Order quantity in lots |
| `type` | No | `limit` or `market` | Order type |
| `timeInForce` | No | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill<br>`DAY` - day | Time in force. Default value - `GTC` |

<i>Example:</i>

```
post url: /api/1/trading/new_order?nonce=1395049771755&apikey=f6ab189hd7a2007e01d95667de3c493d
post data: clientOrderId=11111112&symbol=BTCUSD&side=buy&price=0.1&quantity=100&type=limit&timeInForce=GTC
```

<i>Example response:</i>
``` json
{ "ExecutionReport": 
   { "orderId": "58521038",
     "clientOrderId": "11111112",
     "execReportType": "new",
     "orderStatus": "new",
     "symbol": "BTCUSD",
     "side": "buy",
     "timestamp": 1395236779235,
     "price": 0.1,
     "quantity": 100,
     "type": "limit",
     "timeInForce": "GTC",
     "lastQuantity": 0,
     "lastPrice": 0,
     "leavesQuantity": 100,
     "cumQuantity": 0,
     "averagePrice": 0 } }
```

### <a name="cancelorder"/>/api/1/trading/cancel_order

<i>Summary:</i> cancels an order. Returns `ExecutionReport` JSON object or `CancelReject` JSON object.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), <b>My Orders</b> tab. Click <b>Cancel</b> button in required order line.

<i>Request:</i> `POST /api/1/trading/cancel_order`

<i>Parameters:</i>

| Parameter | Required| Type | Description |
| --- | --- | --- | ---| 
| `clientOrderId` | Yes | string | Order ID, the same as in cancelling order, from 8 to 32 characters|
| `cancelRequestClientOrderId` | Yes | string | Unqiue ID generated by client, from 8 to 32 characters|
| `symbol` | Yes | string | Currency symbol, the same as in cancelling order |
| `side` | Yes | `buy` or `sell`| Side of a trade, the same as in cancelling order |

<i>Example:</i>

```
post url: /api/1/trading/cancel_order?nonce=1395049771755&apikey=f6ab189hd7a2007e01d95667de3c493d
post data: clientOrderId=11111112&cancelRequestClientOrderId=38257825798349578945&symbol=BTCUSD&side=buy
```

<i>Example response:</i>
``` json
{ "ExecutionReport": 
   { "orderId": "58521038",
     "clientOrderId": "11111112",
     "execReportType": "canceled",
     "orderStatus": "canceled",
     "symbol": "BTCUSD",
     "side": "buy",
     "timestamp": 1395236779346,
     "price": 0.1,
     "quantity": 100,
     "type": "limit",
     "timeInForce": "GTC",
     "lastQuantity": 0,
     "lastPrice": 0,
     "leavesQuantity": 0,
     "cumQuantity": 0,
     "averagePrice": 0 } }
```

`CancelReject` example:

``` json 
{ "CancelReject": { 
    "cancelRequestClientOrderId": "011111112",
    "clientOrderId": "11111112",
    "rejectReasonCode": "orderNotFound" 
} }
```

### <a name="recentorders"/>/api/1/trading/orders/recent

<i>Summary:</i> returns an array of user's recent orders (`order` objects) for last 24 hours, sorted by order update time.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), <b>My orders</b> tab.

<i>Request:</i> `GET /api/1/trading/orders/recent`

<i><i>Parameters:</i></i> 

| Parameter | Required | Type | Description |
| --- | --- | --- | ---|
| `start_index` | No | integer | Zero-based index, 0 by default |
| `max_results` | Yes | integer | Maximum quantity of returned items, at most 1000 |
| `sort` | No | `asc` or `desc` | Orders are sorted ascending (default) or descending |
| `symbols` | No | string | Comma-separated list of currency symbols |
| `statuses` | No | `new` <br> `partiallyFilled` <br> `filled` <br> `canceled` <br> `expired` <br> `rejected` | Comma-separated list of order statuses |

The following fields are used in `order` object:

| Field	| Required | Type | Description |
| --- | --- | --- | ---|
| `orderId` | No | integer | Order ID on the exchange |
| `orderStatus` | No | `new` <br> `partiallyFilled` <br> `filled` <br> `canceled` <br> `expired` <br> `rejected` | Order status |
| `lastTimestamp` | Yes | integer | UTC timestamp of the last change, in milliseconds |
| `orderPrice` | Yes - for limit orders | decimal | Order price | 
| `orderQuantity` | Yes | integer | Order quantity, in lots |
| `avgPrice` | Yes | decimal | Average price | 
| `quantityLeaves` | Yes | integer | Remaining quantity, in lots | 
| `type` | No | `limit` or `market` | Type of an order |
| `timeInForce` | No | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill<br>`DAY` - day | Time in force |
| `clientOrderId` | No | string | Unique client-generated ID |
| `symbol` | No | string | Currency symbol |
| `side` | No | `buy` or `sell` | Side of a trade |
| `execQuantity` | No | integer | Last executed quantity, in lots |

<i>Example response:</i>

``` json
{"orders": [
  {
    "orderId": "1",
    "orderStatus": "new",
    "lastTimestamp": 1395659434845,
    "orderPrice": 1,
    "orderQuantity": 1,
    "avgPrice": 0,
    "quantityLeaves": 1,
    "type": "limit",
    "timeInForce": "GTC",
    "clientOrderId": "111111111111111111111111",
    "symbol": "BTCUSD",
    "side": "buy",
    "execQuantity": 0
  },
  {
    "orderId": "2",
    "orderStatus": "new",
    "lastTimestamp": 1395664550770,
    "orderPrice": 1,
    "orderQuantity": 1,
    "avgPrice": 0,
    "quantityLeaves": 1,
    "type": "limit",
    "timeInForce": "GTC",
    "clientOrderId": "111111111111111111111112",
    "symbol": "BTCUSD",
    "side": "sell",
    "execQuantity": 0
  },
  {
    "orderId": "3",
    "orderStatus": "canceled",
    "lastTimestamp": 1395664737500,
    "orderPrice": 1,
    "orderQuantity": 1,
    "avgPrice": 0,
    "quantityLeaves": 1,
    "type": "limit",
    "timeInForce": "GTC",
    "clientOrderId": "111111111111111111111113",
    "symbol": "BTCUSD",
    "side": "buy",
    "execQuantity": 0
  }
]}
```


### <a name="usertrades"/>/api/1/trading/trades

<i>Summary:</i> returns the trading history - an array of user's trades (`trade` objects).

<i>Sample usage at HitBTC site:</i> [https://hitbtc.com/trading-history](https://hitbtc.com/trading-history). Trades for preceding 24 hours see [https://hitbtc.com/terminal](https://hitbtc.com/terminal), <b>My trades</b> tab. 

<i>Request:</i> `GET /api/1/trading/trades`

<i>Parameters:</i>

| Parameter | Required | Type | Description |
| --- | --- | --- | --- |
| `by` | Yes |  `trade_id` or `ts` | Selects if filtering and sorting is performed by `trade_id` or by `timestamp` |
| `start_index` | Yes | integer | Zero-based index. Default value is 0 |
| `max_results` | Yes | integer | Maximum quantity of returned results, at most 1000 |
| `symbols` | No | string | Comma-separated list of currency symbols |
| `sort` | No | `asc` or `desc` | Trades are sorted ascending (default) or descending |
| `from` | No | integer | Returns trades with `trade_id` > specified `trade_id` (if `by=trade_id`) or returns trades with `timestamp` >= specified timestamp` (if `by=ts`) |
| `till` | No | integer | Returns trades with `trade_id` < specified `trade_id` (if `by=trade_id`) or returns trades with `timestamp` < specified `timestamp` (if `by=ts`)|

The following fields are used in `trade` object:

| Field	|Required | Type | Description |
| --- | --- | --- | --- |
| `tradeId` | Yes | integer | Trade ID on the exchange |
| `execPrice` | Yes | decimal | Trade price | 
| `timestamp` |  Yes | integer | Timestamp, in milliseconds |
| `originalOrderId` | No | integer | Order ID on the exchange |
| `fee` | No | decimal | Fee for the trade, negative value means rebate |
| `clientOrderId` | No | string | Unique order ID generated by client. From 8 to 32 characters |
| `symbol` | No | string | Currency symbol |
| `side` | No | `buy` or `sell` | Side of a trade |
| `execQuantity` | No | integer | Trade size, in lots |


<i>Example response:</i>

``` json
{"trades": [
  {
    "tradeId": 39,
    "execPrice": 150,
    "timestamp": 1395231854030,
    "originalOrderId": "114",
    "fee": 0.03,
    "clientOrderId": "FTO18jd4ou41--25",
    "symbol": "BTCUSD",
    "side": "sell",
    "execQuantity": 10
  },
  {
    "tradeId": 38,
    "execPrice": 140.1,
    "timestamp": 1395231853882,
    "originalOrderId": "112",
    "fee": 0.028,
    "clientOrderId": "FTO18jd4ou3n--15",
    "symbol": "BTCUSD",
    "side": "buy",
    "execQuantity": 10
  },
  {
    "tradeId": 2,
    "execPrice": 150,
    "timestamp": 1394789991659,
    "originalOrderId": "24",
    "fee": 0.03,
    "clientOrderId": "FTO18ivvcbvt--25",
    "symbol": "BTCUSD",
    "side": "sell",
    "execQuantity": 10
  },
  {
    "tradeId": 1,
    "execPrice": 140,
    "timestamp": 1394789991527,
    "originalOrderId": "22",
    "fee": 0.028,
    "clientOrderId": "FTO18ivvcbvj--15",
    "symbol": "BTCUSD",
    "side": "buy",
    "execQuantity": 10
  }
]}
```


## <a name="paymentsrestful"/>Payment RESTful API

RESTful API allows to manage funds with the following methods:
  - get multi-currency balance of the main account - [/api/1/payment/balance](#paymentbalance)
  - transfer funds between main and trading accounts - [/api/1/payment/transfer_to_trading, /api/1/payment/transfer_to_main](#transfer)
  - get the last created incoming cryptocurrency address or create a new one -  [/api/1/payment/address/ (GET)](#getaddress), [/api/1/payment/address/ (POST)](#postaddress),
  - create an outgoing crypotocurrency transaction - [/api/1/payment/payout](#payout)
  - get a list of payment transactions - [/api/1/payment/transactions](#transactions)
 
Payment operations require [authentication](#authentication)

### <a name="paymentbalance"/>/api/1/payment/balance

<i>Summary:</i> returns multi-currency balance of the main account.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/account](https://hitbtc.com/account).

<i>Request:</i> `GET /api/1/payment/balance`

<i><i>Parameters:</i></i> no parameters

<i>Example response:</i>

``` json
{
  "balance": [
  {
    "currency_code": "USD",
    "balance": 13.12
  },
  {
    "currency_code": "EUR",
    "balance": 0
  },
  {
    "currency_code": "LTC",
    "balance": 1.07
  }, 
  {"currency_code": "BTC", 
  "balance": 11.9
  }
]}
```

### <a name="transfer"/>/api/1/payment/transfer_to_trading and /api/1/payment/transfer_to_main

<i>Request:</i> `POST /api/1/payment/transfer_to_trading, /api/1/payment/transfer_to_main`

<i>Summary:</i> transfers funds between main and trading accounts; returns a transaction ID or an error.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/account](https://hitbtc.com/account). Click the appropriate arrow in the currency line then specify required amount and click <b>Transfer</b> button.

<i><i>Parameters:</i></i>

| Parameter |Required | Type | Description |
| --- | --- | --- | --- |
| `amount` | Yes | decimal | Funds amount to transfer |
| `currency_code` | Yes | string | Currency symbol, e.g. `BTC` |

<i>Example responses:</i>

```json
{"message": "Balance not enough", "statusCode": 409, "body": "Balance not enough"}
```

```json
{"transaction": "52976-103925-18443984"}
```

### <a name="getaddress"/>/api/1/payment/address/ (GET)

<i>Summary:</i> returns the last created incoming cryptocurrency address that can be used to deposit cryptocurrency to your account. 

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/account](https://hitbtc.com/account). In the required currency line click <b>Fund</b> icon then click <b>copy address</b> link.

<i>Request:</i> `GET /api/1/payment/address/:currency`

<i><i>Parameters:</i></i> no parameters

<i>Example:</i> `GET /api/1/payment/address/BTC`

<i>Example response:</i>

```json
{"address":"1HDtDgG9HYpp1YJ6kFYSB6NgaG2haKnxUH"}
```

### <a name="postaddress"/>/api/1/payment/address/ (POST)

<i>Summary:</i> creates an address that can be used to deposit cryptocurrency to your account; returns a new cryptocurrency address.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/account](https://hitbtc.com/account). In the required currency line click <b>Fund</b> icon then click <b>create new address</b> link.

<i>Request:</i> `POST /api/1/payment/address/:currency`

<i><i>Parameters:</i></i> no parameters

<i>Example:</i> `POST /api/1/payment/address/BTC`

<i>Example response:</i>

```json
{"address":"1HDtDgG9HYpp1YJ6kFYSB6NgaG2haKnxUH"}
```

### <a name="payout"/>/api/1/payment/payout

<i>Summary:</i> withdraws money and creates an outgoing crypotocurrency transaction; returns a transaction ID on the exchange or an error.

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/account](https://hitbtc.com/account). In the required currency line click <b>Fund</b> icon then click <b>copy address</b> link.

<i>Request:</i> `POST /api/1/payment/payout`

<i>Parameters:</i>

| Parameter | Required | Type | Description |
| --- | --- | --- | --- |
| `amount` | Yes | decimal | Funds amount to withdraw |
| `currency_code` | Yes | string | Currency symbol, e.g. `BTC`|
| `address` | Yes | string | BTC/LTC address to withdraw to |

<i>Example:</i> ```amount=0.001&currency_code=BTC&address=1LuWvENyuPNHsHWjDgU1QYKWUYN9xxy7n5```

<i>Example response:</i>
```json
{"transaction": "51545-103004-18442681"}
```

### <a name="transactions"/>/api/1/payment/transactions

<i>Summary:</i> returns a list of payment transactions and their statuses (array of transactions).

<i>Sample usage at HitBTC site:</i> see [https://hitbtc.com/account](https://hitbtc.com/account), <b>History</b> panel.

<i>Request:</i> `GET /api/1/payment/transactions`

<i>Parameters:</i>

| Parameter | Required | Type | Description |
| --- | --- | --- | --- |
| `offset` | No | integer | Start index for the query, default = 0 |
| `limit` | Yes | integer | Maximum results for the query |
| `dir` | No | `ask` or `desc` | Transactions are sorted ascending or descending (default) |

<i>Example response:</i>
```json
{"transactions": [
  {
    "id": "49720-104765-18440728",
    "type": "payin",
    "status": "pending",
    "created": 1397056648,
    "finished": 1397056646,
    "amount_from": 0.001,
    "currency_code_from": "BTC",
    "amount_to": 0.001,
    "currency_code_to": "BTC",
    "destination_data": null,
    "commission_percent": 0,
    "bitcoin_address": "1KnVXD1Wc1wZbTHiB1TDgMcnSRi2PnMHAV",
    "bitcoin_return_address": "1QBuhFksjoWTQWJKWUPyQ37wsQohLAhJvK"
  }
]}
```


# <a name="socketio"/>socket.io Market Data

The API provides socket.io version 1.0.x protocol for receiving market data. It supports:
* WebSocket and xhr-polling transport
* multiplexing a single connection with socket.io namespaces (see [`trades` namespace](#tradesnamespace))

Useful links:
* official socket.io documentation - `http://socket.io/`
* chrome extension Simple WebSocket Client - `https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo`. See also the [Sample code](#sample)

API links:
* socket.io URL: `http://api.hitbtc.com:8081`
* socket.io demo URL: `http://demo-api.hitbtc.com:8081`
* live example (both demo and primary API): `http://jsfiddle.net/rn7zy75n/1/`

### <a name="tradesnamespace"/>`trades` namespace

<i>Namespace:</i> `trades`

<i>URLs:</i> `/trades/:symbol` e.g. `/trades/BTCUSD`

<i>Event:</i> `trade`

<i>Event example:</i>
```json
{"price":478.33,"amount":0.15}
```


# <a name="streaming"/>Streaming API

Streaming API is based on [WebSocket protocol](http://en.wikipedia.org/wiki/WebSocket). All messages are in JSON format.

Streaming API provides an access to: 
  - market data. See [Market data streaming end-point](#marketstreaming)  
  - trading operations. See [Trading streaming end-point](#tradingstreaming)


## <a name="marketstreaming"/>Market data streaming end-point

API links:
* URL: <ws://api.hitbtc.com:80>
* Demo URL: <ws://demo-api.hitbtc.com:80>
* 
Once client connects to this URL the session is started. 

The server broadcasts the following types of messages:
* [MarketDataSnapshotFullRefresh](#MarketDataSnapshotFullRefresh) message contains a full snapshot of the order book.
* [MarketDataIncrementalRefresh](#MarketDataIncrementalRefresh) message contains incremental changes

Some recommendations to consider:
* The application could receive the first snapshot and maintain the order book by applying incremental updates.
* It's recommended to invalidate a state of the application periodically using snapshots.
* It's recommended to check sequence numbers and to drop updates with non-monotonous sequence numbers.

### <a name="MarketDataSnapshotFullRefresh"/>MarketDataSnapshotFullRefresh message

<i>Summary:</i> contains a full snapshot of the order book.

<i>Example message:</i>
```json
{"MarketDataSnapshotFullRefresh": {
    "snapshotSeqNo": 899009,
    "symbol": "BTCUSD",
    "exchangeStatus": "working",
    "ask": [
        {
            "price": 101.42,
            "size": 7
        },
        {
            "price": 101.85,
            "size": 5
        },
        {
            "price": 102.59,
            "size": 1
        },
        {
            "price": 114.53,
            "size": 3
        },
        {
            "price": 114.54,
            "size": 6
        },
        {
            "price": 114.55,
            "size": 19
        }
    ],
    "bid": [
        {
            "price": 89.72,
            "size": 79
        },
        {
            "price": 89.71,
            "size": 158
        },
        {
            "price": 89.7,
            "size": 166
        },
        {
            "price": 89.69,
            "size": 231
        },
        {
            "price": 89.68,
            "size": 169
        },
        {
            "price": 89.67,
            "size": 186
        },
        {
            "price": 89.66,
            "size": 178
        }
    ]
}}
```

The following fields are used in `MarketDataSnapshotFullRefresh` object:

| Field | Description |
| --- | --- |
| `seqNo` | Monotone increasing number of the snapshot, each symbol has its own sequence |
| `timestamp` | UTC timestamp, in milliseconds |
| `symbol` | Currency symbol traded on HitBTC exchange |
| `exchangeStatus` | Exchange status: `on` - trading is open; `off` - trading is suspended |
| `ask`, `bid` | Sorted arrays of price levels in the order book; full snapshot (all price levels) is provided |

### <a name="MarketDataIncrementalRefresh"/>MarketDataIncrementalRefresh message

<i>Summary:</i> contains incremental changes of the order book and individual trades.

<i>Example message:</i>
```json
{"MarketDataIncrementalRefresh": {
    "seqNo": 546693,
    "timestamp": 1381394357861,
    "symbol": "BTCUSD",
    "exchangeStatus": "on",
    "ask": [],
    "bid": [
        {
            "price": 100.98,
            "size": 3
        }
    ],
    "trade": [
        {
            "price": 100.98,
            "size": 5,
            "timestamp": 1346691273926
        }
    ]
}}
```

The following fields are used in `MarketDataIncrementalRefresh` object:

| Field | Description |
| --- | --- |
| `seqNo`	| Monotone increasing number of the snapshot, each symbol has its own sequence |
| `timestamp` |	UTC timestamp, in milliseconds |
| `symbol`	| Currency symbol traded on HitBTC exchange |
| `exchangeStatus` |  Exchange status: `on` - trading is open; `off` - trading is suspended |
| `ask`, `bid`, `trade` | An array of changes in the order book where `price` is a price, `size` is new size. `size`=0 means that the price level has been removed |


## <a name="tradingstreaming"/>Trading streaming end-point

API links:
* URL: <wss://api.hitbtc.com:8080>
* Demo URL: <ws://demo-api.hitbtc.com:8080>

Trading endpoint requires sending login message after connection established. All client messages should be signed and should contain valid and active API key (see [API keys and message signatures](#authenticationwebsocket)).


The following message types are supported:

| Type |  |
| --- | --- | --- |
| [Login](#Login) | Client -> Server |
| [NewOrder](#NewOrder) | Client -> Server |
| [OrderCancel](#OrderCancel) | Client -> Server  |
| [ExecutionReport](#ExecutionReport) | Server -> Client |
| [CancelReject](#CancelReject) | Server -> Client |


### <a name="authenticationwebsocket"/>API keys and message signatures

All client messages should be signed in the following manner:

```json
{
    "apikey": "e418f5b4a15608b78185540ef583b9fc",
    "signature": "FN6/9dnMfLh3wZj+cAFr82HcSvmwuniMQqUlRxSQ9WxRqFpYrjY2xlvDzLC5+qSZAHts8R7KR7HbjiI3SzVxHg==",
    "message":{
        "nonce": 12,
        "payload": {
            "Login": {}
        }
    }
}
```
<i>Fields:</i>

| Field | Description |
| --- | --- |
|`nonce` | Unique monotonous number that should be generated on the client. Should be monotonous within the same connection |
| `signature` | Signature - hash-based message authentication code: base64 [hmac-sha512](http://en.wikipedia.org/wiki/Hash-based_message_authentication_code) (binary representation of the message) |

### <a name="Login"/>Login 

<i>Example:</i>
```json
{
	"apikey": "e418f5b4a15608b78185540ef583b9fc",
	"signature": "FN6/9dnMfLh3wZj+cAFr82HcSvmwuniMQqUlRxSQ9WxRqFpYrjY2xlvDzLC5+qSZAHts8R7KR7HbjiI3SzVxHg==",
	"message":{
		"nonce": 12, 
		"payload": {
			"Login": {}
		}
	}
}
```

<i><i>Parameters:</i></i> no parameters

If client doesn't send valid logon message in 10 second the connection will be dropped.

### <a name="NewOrder"/>NewOrder

<i>Example:</i>

```json
{
    "apikey": "e418f5b4a15608b78185540ef583b9fc",
    "signature": "FN6/9dnMfLh3wZj+cAFr82HcSvmwuniMQqUlRxSQ9WxRqFpYrjY2xlvDzLC5+qSZAHts8R7KR7HbjiI3SzVxHg==",
    "message":{
        "nonce": 12,
        "payload": {
            "NewOrder": {
                "clientOrderId": "68f82819-723a-4b60-ad6b",
                "symbol": "BTCUSD",
                "side": "buy",
                "quantity": 10,
                "type": "limit",
                "price": 788.12,
                "timeInForce": "GTC"
            }
        }
    }
}
```

<i><i>Parameters:</i></i>

| Parameter	| Type | Description |
| --- | --- | --- | 
| `clientOrderId` | string  | Unique order ID generated by client. From 8 to 32 characters |
| `symbol` | string | Currency symbol traded on HitBTC exchange |
| `side` | `buy`, `sell`| Order side  |
| `quantity` | integer | Quantity, in lots | 
| `type` | `limit` or `market` | Order type. Only `limit` orders are currently supported |
| `price`	| decimal | Price, in currency units, consider price steps |
| `timeInForce` | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill | Time in force |  |

### <a name="OrderCancel"/>OrderCancel

<i>Example:</i>

```json
{
    "apikey": "e418f5b4a15608b78185540ef583b9fc",
    "signature": "FN6/9dnMfLh3wZj+cAFr82HcSvmwuniMQqUlRxSQ9WxRqFpYrjY2xlvDzLC5+qSZAHts8R7KR7HbjiI3SzVxHg==",
    "message":{
        "nonce": 12,
        "payload": {
            "OrderCancel": {
                "clientOrderId": "68f82819-723a-4b60-ad6b",
                "cancelRequestClientOrderId":   "2c4d7127-6fbc-450c-b851-c6c1e8954545",
                "symbol": "BTCUSD",
                "side": "buy"
            }
        }
    }
}
```

<i><i>Parameters:</i></i>

| Parameter	| Type | Description |
| --- | --- | --- |
| `clientOrderId` | string  | `clientOrderId` parameter sent in `NewOrder` message |
| `cancelRequestClientOrderId` | string | Unqiue ID generated by client, from 8 to 32 characters |
| `symbol` | string | Currency symbol traded on HitBTC exchange |
| `side` |  `buy`, `sell`| Order side |
| `type` | `limit` or `market` | Order type. Only `limit` orders are currently supported |


### <a name="ExecutionReport"/>ExecutionReport

<i>Example:</i>

```json
{
    "ExecutionReport":{
        "orderId": "64283442",
        "clientOrderId": "68f82819-723a-4b60-ad6b",
        "execReportType": "new",
        "orderStatus": "new"
        "symbol": "BTCUSD",
        "side": "buy",
        "timestamp": 1346691273926,
        "price": 690.99,
        "quantity": 0.1,
        "type": "limit",
        "timeInForce": "GTC"
    }
}
```

The following fields are used in `ExecutionReport` object:

| Field	| Required | Type | Description |
| --- | --- | --- | --- |
| `orderId` | Yes | string | Order ID on the Exchange|
| `clientOrderId` | Yes | string | `clientOrderId` sent in `NewOrder` message (see [/api/1/trading/new_order](#neworder)) |
| `execReportType` | Yes | `new` <br> `canceled` <br> `rejected` <br> `expired` <br> `trade` <br> `status` | Execution report type |
| `orderStatus` | Yes | `new` <br> `partiallyFilled` <br> `filled` <br> `canceled` <br> `rejected` <br> `expired` | Order status |
| `orderRejectReason` | Yes - for orders in `rejected` state| `unknownSymbol` <br> `exchangeClosed` <br>`orderExceedsLimit` <br> `unknownOrder` <br> `duplicateOrder` <br> `unsupportedOrder` <br> `unknownAccount` <br> `other`|Reason of rejection. Relevant only for orders in rejected state |
| `symbol` | Yes | string, e.g. `BTCUSD` | Currency symbol traded on HitBTC exchange (see [Currency symbols](#cursymbols))|
| `side` | Yes| `buy` or `sell` | Side of a trade |
| `timestamp` | No | integer | UTC timestamp in milliseconds |
| `price` | No | decimal | Price of an order|
| `quantity` | Yes | integer | Order quantity in lots |
| `type` | Yes| `limit` | Type of an order. Only `limit` orders are currently supported |
| `timeInForce` | Yes | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill<br>`DAY` - day orders< | Time in force |
| `tradeId` | Yes - for trades| integer | Trade ID on the exchange |
| `lastQuantity` | Yes - for trades | integer | Last quantity |
| `lastPrice` | Yes - for trades | decimal | Last price  |
| `leavesQuantity` | No | integer |  Leaves quantity|
| `cumQuantity` | No | integer | Cumulative quantity |
| `averagePrice` | No | decimal | Average price. Equals 0 if `cumQuantity`=0|


### <a name="CancelReject"/>CancelReject

<i>Example:</i>
```json
{"CancelReject": {
    "clientOrderId": "68f82819-723a-4b60-ad6b",
    "cancelRequestClientOrderId": "2c4d7127-6fbc-450c-b851",
    "rejectReasonCode": "orderNotFound",
    "rejectReasonText": "Order not found",
    "timestamp": 726892347829
}}
```

The following fields are used in `CancelReject` object:

| Field	| Required | Type | Description |
| --- | --- | --- | --- |
| `cancelRequestClientOrderId` | Yes | string | `cancelRequestClientOrderId` parameter from [`OrderCancel`](#ordercancel) |
| `clientOrderId` | Yes | string | `clientOrderId` parameter from [`OrderCancel`](#ordercancel) |
| `rejectReasonCode` | Yes | `orderNotFound` <br> `unknownSymbol` <br> `unknownUser` <br> `other` | Code of the reason why the order was cancelled|
| `rejectReasonText` | No | string | Optional text explaining reject reason |


## <a name="sample"/>Sample code

### Node.js snippet: message signature

```javascript
    var crypto = require('crypto');
    
    ...
    
    var msg = {
        'apikey': apikey,
        'signature': '',
        'message': {
            'nonce': nonce,
            'payload': {
                'NewOrder': {
                    'clientOrderId': clientOrderId,
                    'symbol': symbol,
                    'side': side,
                    'quantity': quantity,
                    'type': type,
                    'price': price,
                    'timeInForce': timeInForce
                }
            }
        }
    };
    msg.signature = crypto.createHmac('sha512', secretkey).update(JSON.stringify(msg.message)).digest('base64');
    return JSON.stringify(msg);
```
