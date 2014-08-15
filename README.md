<a name="summary"/>
## Summary

This document provides the complete reference for [HitBTC](https://hitbtc.com) API.

HitBTC API has several interfaces to implement them in a custom software:
* <b>RESTful API</b> that allows: 
  - access to the market data: view ticker, order book, trades, etc. See [Market data RESTful API](#marketrestful)
  - performing trading operations: view trading balance, place or cancel orders, view history, etc. See [Trading RESTful API](#tradingrestful)
  - managing funds: view balance of the main account, transfer funds between main and trading accounts, create an outgoing  transactions, etc. See [Payment RESTful API](#paymentsrestful)
* <b>socket.io</b> protocol for receiving the market data. socket.io protocol supports WebSocket, xhr-polling and jsonp-polling transports. See [socket.io Market Data](#socketio)
* <b>Streaming API</b> based on WebSocket protocol to get an access to: 
  - market data. See [Market data streaming end-point](#marketstreaming)  
  - trading operations. See [Trading streaming end-point](#tradingstreaming)

Trading and payment operations require user's authentification: each request or message should have a signature. 
You should get your API key and Secret key on the [Settings](https://hitbtc.com/settings) page. See details in [RESTful API authentification](#authenticationrestful) and [WebSocket API authentification](#authenticationwebsocket).

<a name="cursymbols"/>
### Currency symbols

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

<a name="restful"/>
<a name="marketrestful"/>
## Market data RESTful API

RESTful API provides access to the market data with following methods:
  - get the timestamp- [/api/1/public/time](#time)
  - get the list of symbols - [/api/1/public/symbols](#symbols)
  - get the ticker for specified symbol - [/api/1/public/:symbol/ticker](#ticker)
  - get the order book for specified symbol - [/api/1/public/:symbol/orderbook](#orderbook)
  - get the individual trades data for specified symbol - [/api/1/public/:symbol/trades](#trades)

Endpoint URL: [http://api.hitbtc.com](http://api.hitbtc.com)

Demo endoint: [http://demo-api.hitbtc.com](http://demo-api.hitbtc.com)

<a name="time"/>
### /api/1/public/time

Summary: returns the server time in UNIX timestamp format

Request: `GET /api/1/public/time`

Example: `/api/1/public/time`
``` json
{
    "timestamp": 1393492619000
}
```

<a name="symbols"/>
### /api/1/public/symbols

Summary: returns actual list of symbols

Request: `GET /api/1/public/symbols`

Example: `/api/1/public/symbols`
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

<a name="ticker"/>
### /api/1/public/:symbol/ticker

Summary: returns actual data on cryptocurrency exchange rates

Request: `GET /api/1/public/:symbol/ticker`

Example: `/api/1/public/BTCUSD/ticker`

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

* last - last price
* bid - highest buy order
* ask - lowest sell order
* high - highest trade price per last 24h + last incomplete minute
* low - lowest trade price per last 24h + last incomplete minute
* volume - volume per last 24h + last incomplete minute
* timestamp - the server time in UNIX timestamp format

<a name="orderbook"/>
### /api/1/public/:symbol/orderbook

Summary: returns a list of open orders: price and amount

Request: `GET /api/1/public/:symbol/orderbook`

| Parameter | Description |
| --- | --- |
| format_price | optional, "string" (default) or "number" |
| format_amount | optional, "string" (default) or "number" |
| format_amount_unit | optional, "currency" (default) or "lot" |

Alias: `/api/1/request/:symbol/orderbook.json` -> `/api/1/public/:symbol/orderbook?format_price=number&format_amount=number`

Example: `/api/1/public/BTCUSD/orderbook`
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

Example: `/api/1/public/BTCUSD/orderbook?format_price=number&format_amount=number`
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

<a name="trades"/>
### /api/1/public/:symbol/trades

Summary: returns data on trades in specified ID or timestamp interval

Request: `GET /api/1/public/:symbol/trades`

Parameters:

| Parameter | Description | Reqired	| Type |
| --- | --- | --- | --- |
| from | returns trades with trade_id > specified trade_id <br> returns trades with timestamp >= specified timestamp | required| int, trade_id or timestamp |
| till | returns trades with trade_id < specified trade_id <br> returns trades with timestamp < specified timestamp | | int, trade_id or timestamp |
| by | | required | filter and sort by `trade_id` or `ts` (timestamp) |
| sort | | | `asc` (default) or `desc` |
| start_index | zero-based | required | int |
| max_results | | required | int, max value = 1000 |
| format_item | | | "array" (default) or "object" |
| format_price | | | "string" (default) or "number" |
| format_amount | | | "string" (default) or "number" |
| format_amount_unit | |  |"currency" (default) or "lot" |
| format_tid | |  | "string" or "number" (default) |
| format_timestamp | |  | "millisecond" (default) or "second" |
| format_wrap | |  | "true" (default) or "false" |

Alias:
```
`/api/1/request/:symbol/trades.json?since=<trade_id>` -> `/api/1/public/:symbol/trades?from=<trade_id>&by=trade_id&start_index=0&format_numbers=number&format_tradeid=string&format_objects=object&format_timestamp=second`
```

Example:
```
`/api/1/public/BTCUSD/trades?from=0&by=trade_id&sort=desc&start_index=0&max_results=100`
```

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


Example: 
````/api/1/public/BTCUSD/trades?from=0&by=trade_id&sort=desc&start_index=0&max_results=100&format_item=object&format_price=number&format_amount=number&format_tid=string&format_timestamp=second&format_wrap=false`
```

``` json
[
    {"date": 1393492619, "price": 575.64, "amount": 0.02, "tid": "3814483"},
    {"date": 1393492619, "price": 574.3, "amount": 0.12, "tid": "3814482"},
    {"date": 1393492619, "price": 573.67, "amount": 3.8, "tid": "3814481"},
    {"date": 1393492619, "price": 571, "amount": 0.01, "tid": "3814479"},
    ...
]
```

<a name="tradingrestful"/>
## Trading RESTful API

RESTful API allows to perform trading operations with the following methods:
  - get the trading balance - [/api/1/trading/time](#tradingbalance)
  - get all active orders  - [/api/1/trading/active](#active)
  - place a new order - [/api/1/trading/new_order](#neworder); WebSocket - [NewOrder](#NewOrder)
  - cancel an order - [/api/1/trading/cancel_order](#cancelorder); WebSocket - [OrderCancel](#OrderCancel)
  - get user's trading history - [/api/1/trading/trades](#usertrades)
  - get user's recent orders (RESTful - [/api/1/trading/recent](#recentorders)

Base URL: [https://api.hitbtc.com](https://api.hitbtc.com)

HitBTC provides a demo trading option.  You can enable demo mode and acquire demo API keys on the [Settings](https://hitbtc.com/settings) page.
Demo endoint address: [http://demo-api.hitbtc.com](http://demo-api.hitbtc.com)

Real trading operations require [authentication](#authentication).

[Error codes](#errors) and [reports representing order status changes](#reports) are described below.


<a name="authenticationrestful"/>
### Authentication

RESTful Trading API requires HMAC-SHA512 signatures for each request.

To use this API endpoint you should get your API key and Secret key from the [Settings](https://hitbtc.com/settings) page. 

Each request should include three parameters: `apikey`, `signature` and `nonce`.

* `nonce` - unique monotonous number that should be generated on the client. Hint: use millisecond or microsecond timestamp for `nonce`. This parameter should be added as a query string parameter `nonce`. `nonce` should be < (2^53-1).
* `apikey` - API key from [Settings](https://hitbtc.com/settings) page. This parameter should be added as a query string parameter `apikey`.
* `signature` - _lower-case_ hex representation of hmac-sha512 of concatenated `uri` and `postData`. This parameter should be added as a HTTP header `X-Signature`.

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

Check out examples provided by the community:
* C# example code: https://gist.github.com/hitbtc-com/9808530
* PHP example code: https://gist.github.com/hitbtc-com/10885873 

<a name="errors"/>
### Error codes

Trading RESTful API can return the following errors:

| HTTP code | Text | Description |
| --- | --- | --- |
| 403 | Invalid API key | API key doesn't exist or API key is currently used on another endpoint (max last 15 min) |
| 403 | Nonce has been used | nonce is not monotonous |
| 403 | Nonce is not valid | too big number or not a number |
| 403 | Wrong signature | |

<a name="reports"/>
### Execution reports

The API uses `ExecutionReport` as an object that represents change of order status.

The following fields are used in this object:

| Field	| Description | Type / Enum | Required |
| --- | --- | --- | --- |
| orderId | Order ID on the Exchange | string | required |
| clientOrderId | clientOrderId sent in NewOrder message | string | required |
| execReportType | execution report type | `new` <br> `canceled` <br> `rejected` <br> `expired` <br> `trade` <br> `status` | required |
| orderStatus | order status | `new` <br> `partiallyFilled` <br> `filled` <br> `canceled` <br> `rejected` <br> `expired` | required |
| orderRejectReason | Relevant only for the orders in rejected state | `unknownSymbol` <br> `exchangeClosed` <br>`orderExceedsLimit` <br> `unknownOrder` <br> `duplicateOrder` <br> `unsupportedOrder` <br> `unknownAccount` <br> `other`| for rejects |
| symbol | | string, e.g. `BTCUSD` | required |
| side | | `buy` or `sell` | required |
| timestamp | UTC timestamp in milliseconds | | |
| price | | decimal | |
| quantity | | integer | required |
| type | | only `limit` orders are currently supported | required |
| timeInForce | time in force | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill<br>`DAY` - day orders< | required |
| tradeId | Trade ID on the exchange | | for trades |
| lastQuantity | | integer | for trades |
| lastPrice | | decimal | for trades |
| leavesQuantity | | integer |  |
| cumQuantity | | integer | |
| averagePrice | | decimal, will be 0 if 'cumQuantity'=0 | |

Example:

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


<a name="tradingbalance"/>
### /api/1/trading/balance

Summary: returns trading balance.

Request: `GET /api/1/trading/balance`

Parameters: no parameters

Example: 
``` json
{"balance": [
  {
    "currency_code": "BTC",
    "cash": 0.045457701,
    "reserved": 0.01
  },
  {
    "currency_code": "EUR",
    "cash": 0.0445544,
    "reserved": 0
  },
  {
    "currency_code": "LTC",
    "cash": 0.7,
    "reserved": 0.1
  },
  {
    "currency_code": "USD",
    "cash": 2.9415029,
    "reserved": 1.001
  }
]}
```

<a name="active"/>
### /api/1/trading/orders/active

Summary: returns all active orders (`new` or `partiallyFilled`).

Request: `GET /api/1/trading/orders/active`

Parameters: 

| Parameter | Type | Description |
| --- | --- | --- |
| symbols | string, comma-delimeted list of symbols, optional, default - all symbols | |

Example:

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

<a name="neworder"/>
### /api/1/trading/new_order

Summary: place a new order. Returns a JSON object `ExecutionReport` that respresent a status of the order.

Request: `POST /api/1/trading/new_order`

Parameters: 

| Parameter | Type | Description |
| --- | --- | --- |
| clientOrderId | string, >=8 characters, <= 32 characters, required | unique order id generated by client |
| symbol | string, required | e.g. `BTCUSD` |
| side | `buy` or `sell`, required | |
| price | decimal, required | order price, required for limit orders |
| quantity | int | order quantity in lots |
| type | `limit` or `market` | order type |
| timeInForce | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill<br>`DAY` - day | use `GTC` by default |

Example:

```
post url: /api/1/trading/new_order?nonce=1395049771755&apikey=f6ab189hd7a2007e01d95667de3c493d
post data: clientOrderId=11111112&symbol=BTCUSD&side=buy&price=0.1&quantity=100&type=limit&timeInForce=GTC
```

Example response:
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

<a name="cancelorder"/>
### /api/1/trading/cancel_order

Summary: cancels an order. Returns `ExecutionReport` JSON object or `CancelReject` JSON object.

Request: `POST /api/1/trading/cancel_order`

Parameters: 

| Parameter | Type | Description |
| --- | --- | --- |
| clientOrderId | string, >=8 characters, <= 32 characters, required | the same an in your order |
| cancelRequestClientOrderId | string, >=8 characters, <= 32 characters, required | unqiue id generated by client |
| symbol | string, required | the same an in your order |
| side | `buy` or `sell`, required | the same an in your order |

Example:

```
post url: /api/1/trading/cancel_order?nonce=1395049771755&apikey=f6ab189hd7a2007e01d95667de3c493d
post data: clientOrderId=11111112&cancelRequestClientOrderId=38257825798349578945&symbol=BTCUSD&side=buy
```

Example response:
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

<a name="usertrades"/>
### /api/1/trading/trades

Summary: returns the trading history - an array of user's trades (`trade` objects).

Request: `GET /api/1/trading/trades`

Parameters: 

| Parameter | Type | Description |
| --- | --- | --- |
| `by` | `trade_id` or `ts` (timestamp) | |
| `start_index` | int, optional, default(0) | zero-based index |
| `max_results` | int, required, <=1000 | |
| `symbols` | string, comma-delimited | |
| `sort` | `asc` (default) or `desc` | |
| `from` | optional | start `trade_id` or `ts`, see `by` |
| `till` | optional | end `trade_id` or `ts`, see `by` |

The following fields are used in `trade` object:

| Field	| Description | Type / Enum |
| --- | --- | --- |
| `tradeId` | Trade ID on the exchange  | int, required | 
| `execPrice` | Trade price  | decimal, required | 
| `timestamp` |   | millisecond timestamp, required |
| `originalOrderId` | Order ID on the exchange | |
| `fee` | Fee for the trade | decimal, negative means rebate |
| `clientOrderId` | Unique client-generated ID | string |
| `symbol` | | string |
| `side` | | `buy` or `sell` |
| `execQuantity` | trade size, in lots | int |

Example response:

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

<a name="recentorders"/>
### /api/1/trading/orders/recent

Summary: returns an array of user's recent orders (`order` objects) for last 24 hours, sorted by order update time.

Request: `GET /api/1/trading/orders/recent`

Parameters: 

| Parameter | Type | Description |
| --- | --- | --- |
| `start_index` | int, optional, default(0) | zero-based index |
| `max_results` | int, required, <=1000 | |
| `sort` | `asc` or `desc` (`asc` by default) | |
| `symbols` | string, comma-delimited | |
| `statuses` | string, comma-delimited, `new`, `partiallyFilled`, `filled`, `canceled`, `expired`, `rejected` | |

The following fields are used in `order` object:

| Field	| Description | Type / Enum |
| --- | --- | --- |
| `orderId` | Order ID on the exchange | |
| `orderStatus` |  | `new`, `partiallyFilled`, `filled`, `canceled`, `expired`, `rejected` |
| `lastTimestamp` | last change  | millisecond timestamp, required |
| `orderPrice` | Order price  | decimal, required for limit orders | 
| `orderQuantity` | Order quantity in lots | int, required | 
| `avgPrice` | Avg. price  | decimal | 
| `quantityLeaves` | Remaining quantity in lots | int, required | 
| `type` |  | `limit` or `market` |
| `timeInForce` | | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill<br>`DAY` - day |
| `clientOrderId` | Unique client-generated ID | string |
| `symbol` | | string |
| `side` | | `buy` or `sell` |
| `execQuantity` | last executed quantity in lots | int |

Example response:

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

<a name="paymentsrestful"/>
## Payment RESTful API

RESTful API allows to manage funds with the following methods:
  - get multi-currency balance of the main account - [/api/1/payment/balance](#paymentbalance)
  - transfer funds between main and trading accounts - [/api/1/payment/transfer_to_trading, /api/1/payment/transfer_to_main](#transfer)
  - get the last created incoming cryptocurrency address or create a new one -  [/api/1/payment/address/:currency](#address)
  - create an outgoing crypotocurrency transaction - [/api/1/payment/payout](#payout)
  - get a list of payment transactions - [/api/1/payment/transactions](#transactions)
 
Base URL: [https://api.hitbtc.com](https://api.hitbtc.com)

Demo endoint: the Payment API is not available in demo mode.

Payment operations require [authentication](#authentication)

<a name="paymentbalance"/>
### /api/1/payment/balance

Summary: returns multi-currency balance of the main account.

Request: `GET /api/1/payment/balance`

Parameters: no parameters

Example response:

``` json
{"balance": [{"currency_code": "USD", "balance": 13.12}, {"currency_code": "EUR", "balance": 0}, {"currency_code": "LTC", "balance": 1.07}, {"currency_code": "BTC", "balance": 11.9}]}
```

<a name="transfer"/>
### /api/1/payment/transfer_to_trading and /api/1/payment/transfer_to_main

Request: `POST /api/1/payment/transfer_to_trading, /api/1/payment/transfer_to_main`

Summary: transfers funds between main and trading accounts; returns a transaction ID or an error

Parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `amount` | decimal, required | amount |
| `currency_code` | string, required, e.g. `BTC` | |

Example responses:

```json
{"message": "Balance not enough", "statusCode": 409, "body": "Balance not enough"}
```

```json
{"transaction": "52976-103925-18443984"}
```

<a name="getaddress"/>
### /api/1/payment/address/ (GET)

Summary: returns the last created incoming cryptocurrency address that can be used to deposit cryptocurrency. 

Request: `GET /api/1/payment/address/:currency`

Parameters: no parameters

Example request: `GET /api/1/payment/address/BTC`

Example response:

```json
{"address":"1HDtDgG9HYpp1YJ6kFYSB6NgaG2haKnxUH"}
```

<a name="postaddress"/>
### /api/1/payment/address/ (POST)

Summary: creates an address that can be used to deposit cryptocurrency; returns a new cryptocurrency address.

Request: `POST /api/1/payment/address/:currency`

Parameters: no parameters

Example request: `POST /api/1/payment/address/BTC`

Example response:

```json
{"address":"1HDtDgG9HYpp1YJ6kFYSB6NgaG2haKnxUH"}
```

<a name="payout"/>
### /api/1/payment/payout

Summary: withdraws money and creates an outgoing crypotocurrency transaction; returns a transaction ID on the exchange or an error.

Request: `POST /api/1/payment/payout`

Parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `amount` | decimal, required | amount |
| `currency_code` | string, required, e.g. `BTC` | |
| `address` | string, required | BTC/LTC address |

Example: ```amount=0.001&currency_code=BTC&address=1LuWvENyuPNHsHWjDgU1QYKWUYN9xxy7n5```

Example response:
```json
{"transaction": "51545-103004-18442681"}
```

<a name="transactions"/>
### /api/1/payment/transactions

Summary: returns a list of payment transactions and their statuses (array of transactions).

Request: `GET /api/1/payment/transactions`

Parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `offset` | int, optional, default = 0 | start index for the query |
| `limit` | int, required | max results for the query |
| `dir` | string, optional, `ask` or `desc` default = `desc` | sort direction |

Example response:
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



<a name="socketio"/>
## socket.io Market Data

The API provides socket.io protocol for receiving market data. It supports:
* WebSocket, xhr-polling and jsonp-polling transports
* multiplexing a single connection with socket.io namespaces (see [`trades` namespace](#tradesnamespace))

Useful links:
* official socket.io documentation - `http://socket.io/`
* chrome extension Simple WebSocket Client - `https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo`. See also the [Sample code](#sample)

API links:
* socket.io URL: `http://api.hitbtc.com:8081`
* socket.io demo URL: `http://demo-api.hitbtc.com:8081`
* live example (both demo and primary API): `http://jsfiddle.net/He6AU/13/`

<a name="tradesnamespace"/>
### `trades` namespace

Namespace: `trades`

URLs: `/trades/:symbol` e.g. `/trades/BTCUSD`

Event: `trade`

Event example:
```json
{"price":478.33,"amount":0.15}
```


<a name="streaming"/>
<a name="marketstreaming"/>
## Market data streaming end-point

Streaming API is based on [WebSocket protocol](http://en.wikipedia.org/wiki/WebSocket). All messages are in JSON format.

URL: [ws://api.hitbtc.com](ws://api.hitbtc.com)
Demo URL: [ws://demo-api.hitbtc.com](ws://demo-api.hitbtc.com)
Once client connects to this URL the session is started. 

The server broadcasts the following types of messages:
* [MarketDataSnapshotFullRefresh](#MarketDataSnapshotFullRefresh) message contains a full snapshot of the order book.
* [MarketDataIncrementalRefresh](#MarketDataIncrementalRefresh) message contains incremental changes

Some recommendations to consider:
* The application could receive the first snapshot and maintain the order book by applying incremental updates.
* It's recommended to invalidate a state of the application periodically using snapshots.
* It's recommended to check sequence numbers and to drop updates with non-monotonous sequence numbers.

<a name="MarketDataSnapshotFullRefresh"/>
### MarketDataSnapshotFullRefresh message

Summary: contains a full snapshot of the order book.

Example message:
```json
{"MarketDataSnapshotFullRefresh": {
    "snapshotSeqNo": 899009,
    "symbol": "BTCUSD",
    "exchangeStatus": "working",
    "ask": [
        {http://jsfiddle.net/He6AU/13/
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

Fields:

| Field | Description |
| --- | --- |
| seqNo | monotone increasing number, each symbol has an own sequence |
| timestamp | millisecond timestamp UTC |
| symbol | |
| exchangeStatus | `on` or `off`, `off` means the trading is suspended |
| ask, bid | sorted arrays of price levels in the order book; full snapshot (all price levels) is provided |

<a name="MarketDataIncrementalRefresh"/>
### MarketDataIncrementalRefresh message

Summary: contains incremental changes of the order book and individual trades.

Example message:
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

Fields:

| Field | Description |
| --- | --- |
| seqNo	| monotone increasing number, each symbol has an own sequence
| timestamp |	millisecond timestamp UTC |
| symbol	| |
| exchangeStatus |  `on` or `off`, `off` means the trading is suspended |
| ask, bid, trade | an array of changes in the order book; <br> `size` means new size, `size`=0 means price level has been removed |


<a name="tradingstreaming"/>
## Trading streaming end-point

Streaming API is based on [WebSocket protocol](http://en.wikipedia.org/wiki/WebSocket). All messages are in JSON format.

URL: <wss://api.hitbtc.com:8080>
Demo URL: <ws://demo-api.hitbtc.com:8080>

Trading endpoint requires sending login message after connection established. All client messages should be signed and should contain valid and active API key (see [API keys and message signatures](#authenticationwebsocket)).


The following message types are supported:

| Type |  |
| --- | --- | --- |
| [Login](#Login) | Client -> Server |
| [NewOrder](#NewOrder) | Client -> Server |
| [OrderCancel](#OrderCancel) | Client -> Server  |
| [ExecutionReport](#ExecutionReport) | Server -> Client |
| [CancelReject](#CancelReject) | Server -> Client |


<a name="authenticationwebsocket"/>
### API keys and message signatures

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

| Field | Description |
| --- | --- |
| nonce | should be monotonous within the same connection |
| signature | base64 [hmac-sha512](http://en.wikipedia.org/wiki/Hash-based_message_authentication_code)(binary representation of the message) |

<a name="Login"/>
### Login 

Example:
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

Parameters: no parameters

If client doesn't send valid logon message in 10 second the connection will be dropped.

<a name="NewOrder"/>
### NewOrder

Example:

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

Parameters:

| Parameter	| Description | Type / Enum |
| --- | --- | --- |
| clientOrderId | should be unique, <= 32 characters | |
| symbol | | |
| side | order side | `buy`, `sell` |
| quantity | quantity in lots | integer |
| type | order type	| only `limit` orders are currently supported |
| price	| price (in currency) | decimal, consider price steps |
| timeInForce | time in force | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill |

<a name="OrderCancel"/>
### OrderCancel

Example:

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

Parameters:

| Parameter	| Description | Type / Enum |
| --- | --- | --- |
| clientOrderId | clientOrderId sent in NewOrder message | |
| cancelRequestClientOrderId | <= 32 characters | |
| symbol | | |
| side | order side | `buy`, `sell` |
| type | order type	| only `limit` orders are currently supported |

<a name="ExecutionReport"/>
### ExecutionReport

Example:

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

Fields:

| Field	| Description | Type / Enum | Required |
| --- | --- | --- | --- |
| orderId | Order ID on the Exchange | string | required |
| clientOrderId | clientOrderId sent in NewOrder message | string | required |
| execReportType | execution report type | `new` <br> `canceled` <br> `rejected` <br> `expired` <br> `trade` <br> `status` | required |
| orderStatus | order status | `new` <br> `partiallyFilled` <br> `filled` <br> `canceled` <br> `rejected` <br> `expired` | required |
| orderRejectReason | Relevant only for the orders in rejected state | `unknownSymbol` <br> `exchangeClosed` <br>`orderExceedsLimit` <br> `unknownOrder` <br> `duplicateOrder` <br> `unsupportedOrder` <br> `unknownAccount` <br> `other`| |
| symbol | | | required |
| side | | `buy` or `sell` | required |
| timestamp | UTC timestamp in milliseconds | | |
| price | | decimal | |
| quantity | | integer | required |
| type | | only `limit` orders are currently supported | required |
| timeInForce | time in force | `GTC` - Good-Til-Canceled <br>`IOC` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill | required |
| tradeId | Trade ID on the exchange | | for trades |
| lastQuantity | | integer | for trades |
| lastPrice | | decimal | for trades |
| leavesQuantity | | integer |  |
| cumQuantity | | integer | |
| averagePrice | | decimal, will be 0 if 'cumQuantity'=0 | |

<a name="CancelReject"/>
### CancelReject

Example:
```json
{"CancelReject": {
    "clientOrderId": "68f82819-723a-4b60-ad6b",
    "cancelRequestClientOrderId": "2c4d7127-6fbc-450c-b851",
    "rejectReasonCode": "orderNotFound",
    "rejectReasonText": "Order not found",
    "timestamp": 726892347829
}}
```

Fields:

| Field	| Description | Type / Enum | Required |
| --- | --- | --- | --- |
| cancelRequestClientOrderId | `cancelRequestClientOrderId` from OrderCancel | string | required |
| clientOrderId | `clientOrderId` from OrderCancel | string | required |
| rejectReasonCode | | `orderNotFound` <br> `unknownSymbol` <br> `unknownUser` <br> `other` | required |
| rejectReasonText | Optional reject reason text | string | |


<a name="sample"/>
## Sample code

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
