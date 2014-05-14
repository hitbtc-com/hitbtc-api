## Summary

This document provides the complete reference for [hitbtc](https://hitbtc.com) API.


The following symbols are traded on hitbtc exchange.

| Symbol	| Lot size | Price step
| --- | --- | --- |
| BTCUSD |	0.01 BTC |	0.01 |
| BTCEUR |	0.01 BTC |	0.01 |
| LTCBTC | 0.1 LTC |	0.00001 |
| LTCUSD | 0.1 LTC |	0.001 |
| LTCEUR | 0.1 LTC |	0.001 |
| EURUSD | 1 EUR |	0.0001 |

Size representation:
* Size values in streaming messages are represented in lots.
* Size values in RESTful market data are represented in money (e.g. in coins or in USD). 
* Size values in RESTful trade are represented in lots (e.g. 1 means 0.01 BTC for BTCUSD)

### Demo trading

Hitbtc provides a demo trading option.  You can enable demo mode and acquire demo API keys on the Settings page.

## Market data RESTful API

Endpoint URL: [http://api.hitbtc.com](http://api.hitbtc.com)

Demo endoint: [http://demo-api.hitbtc.com](http://demo-api.hitbtc.com)

### /api/1/public/time

Request: `GET /api/1/public/time`

Example: `/api/1/public/time`
``` json
{
    "timestamp": 1393492619000
}
```

### /api/1/public/symbols

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

### /api/1/public/:symbol/ticker

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

* 24h means last 24h + last incomplete minute
* high - highest trade price / 24 h
* low - lowest trade price / 24 h
* volume - volume / 24h

### /api/1/public/:symbol/orderbook

Request: `GET /api/1/public/:symbol/orderbook`

| Parameter | Type | Description |
| --- | --- | --- |
| format_price | optional, "string" (default) or "number" | |
| format_amount | optional, "string" (default) or "number" | |
| format_amount_unit | optional, "currency" (default) or "lot" | |

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

### /api/1/public/:symbol/trades

Request: `GET /api/1/public/:symbol/trades`

Parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| from | required, int, trade_id or timestamp | returns trades with trade_id > specified trade_id <br> returns trades with timestamp >= specified timestamp |
| till | optional, int, trade_id or timestamp | returns trades with trade_id < specified trade_id <br> returns trades with timestamp < specified timestamp |
| by | required, filter and sort by `trade_id` or `ts` (timestamp) | |
| sort | optional, `asc` (default) or `desc` | |
| start_index | required, int | zero-based |
| max_results | required, int, max value = 1000 | |
| format_item | optional, "array" (default) or "object" |  |
| format_price | optional, "string" (default) or "number" | |
| format_amount | optional, "string" (default) or "number" | |
| format_amount_unit | optional, "currency" (default) or "lot" | |
| format_tid | optional, "string" or "number" (default) | |
| format_timestamp | optional, "millisecond" (default) or "second" | |
| format_wrap | optional, "true" (default) or "false" | |

Alias: `/api/1/request/:symbol/trades.json?since=<trade_id>` -> `/api/1/public/:symbol/trades?from=<trade_id>&by=trade_id&start_index=0&format_numbers=number&format_tradeid=string&format_objects=object&format_timestamp=second`

Example: `/api/1/public/BTCUSD/trades?from=0&by=trade_id&sort=desc&start_index=0&max_results=100`
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


Example: `/api/1/public/BTCUSD/trades?from=0&by=trade_id&sort=desc&start_index=0&max_results=100&format_item=object&format_price=number&format_amount=number&format_tid=string&format_timestamp=second&format_wrap=false`
``` json
[
    {"date": 1393492619, "price": 575.64, "amount": 0.02, "tid": "3814483"},
    {"date": 1393492619, "price": 574.3, "amount": 0.12, "tid": "3814482"},
    {"date": 1393492619, "price": 573.67, "amount": 3.8, "tid": "3814481"},
    {"date": 1393492619, "price": 571, "amount": 0.01, "tid": "3814479"},
    ...
]
```

## Trading RESTful API

Base URL: [https://api.hitbtc.com](https://api.hitbtc.com)

Demo endoint: [http://demo-api.hitbtc.com](http://demo-api.hitbtc.com)

### Authentication

RESTful Trading API requires a HMAC-SHA512 signatures for each request.

You should get your API key and Secret key from the Settings page on [https://hitbtc.com](https://hitbtc.com) in order to use this API endpoint. 

Each request should include three parameters: `apikey`, `signature` and `nonce`.

* `nonce` - unique monotonous number that should be generated on the client. Hint: use millisecond or microsecond timestamp for `nonce`. This parameter should be added as a query string parameter `nonce`. `nonce` should be < (2^53-1).
* `apikey` - API key from Settings page. This parameter should be added as a query string parameter `apikey`.
* `signature` - _lower-case_ hex representation of hmac-sha512 of concatenated `uri` and `postData`. This parameter should be added as a HTTP header `X-Signature`.

Signture generation pseudo-code:

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

Check out examples that are provided by the community:
* C# example code: https://gist.github.com/hitbtc-com/9808530
* PHP example code: https://gist.github.com/hitbtc-com/10885873 

### Error codes

RESTful Trading API can return the following errors:

| HTTP code | Text | Description |
| --- | --- | --- |
| 403 | Invalid apikey | API key doesn't exist or API key is currently used on another endpoint (max last 15 min) |
| 403 | Nonce has been used | nonce is not monotonous |
| 403 | Nonce is not valid | too big number or not a number |
| 403 | Wrong signature | |

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


### /api/1/trading/balance

Request: `GET /api/1/trading/balance`

Summary: returns trading balance.

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

### /api/1/trading/orders/active

Request: `GET /api/1/trading/orders/active`

Summary: returns all active orders (`new` or `partiallyFilled`).

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

### /api/1/trading/new_order

Request: `POST /api/1/trading/new_order`

Summary: place a new order.

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

Return value: returns a JSON object `ExecutionReport` that respresent a status of the order.

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

### /api/1/trading/cancel_order

Request: `POST /api/1/trading/cancel_order`

Summary: cancels an order.

Parameters: 

| Parameter | Type | Description |
| --- | --- | --- |
| clientOrderId | string, >=8 characters, <= 32 characters, required | the same an in your order |
| cancelRequestClientOrderId | string, >=8 characters, <= 32 characters, required | unqiue id generated by client |
| symbol | string, required | the same an in your order |
| side | `buy` or `sell`, required | the same an in your order |

Return values: could return `ExecutionReport` JSON object or `CancelReject` JSON object.

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

### /api/1/trading/trades

Request: `GET /api/1/trading/trades`

Summary: returns an user's trading history.

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

Return values: could return an array of user's trades.

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

### /api/1/trading/orders/recent

Request: `GET /api/1/trading/orders/recent`

Summary: returns an user's recent orders.

Parameters: 

| Parameter | Type | Description |
| --- | --- | --- |
| `start_index` | int, optional, default(0) | zero-based index |
| `max_results` | int, required, <=1000 | |
| `symbols` | string, comma-delimited | |
| `statuses` | string, comma-delimited, `new`, `partiallyFilled`, `filled`, `canceled`, `expired`, `rejected` | |

Return values: could return an array of user's recent orders.

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

## Payment RESTful API

Base URL: [https://api.hitbtc.com](https://api.hitbtc.com)

Demo endoint: the Payment API is not available in demo mode.

### /api/1/payment/balance

Request: `GET /api/1/payment/balance`

Summary: returns a balance of the main account.

Parameters: no parameters

Return values: returns multi-currency balance of the main account.

Example response:

``` json
{"balance": [{"currency_code": "USD", "balance": 13.12}, {"currency_code": "EUR", "balance": 0}, {"currency_code": "LTC", "balance": 1.07}, {"currency_code": "BTC", "balance": 11.9}]}
```

### /api/1/payment/transfer_*

Request: `POST /api/1/payment/transfer_to_trading, /api/1/payment/transfer_to_main`

Summary: transfer funds between main and trading accounts.

Parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `amount` | decimal, required | amount |
| `currency_code` | string, required, e.g. `BTC` | |

Return values: returns a transaction id or an error.

Example responses:

```json
{"message": "Balance not enough", "statusCode": 409, "body": "Balance not enough"}
```

```json
{"transaction": "52976-103925-18443984"}
```

### /api/1/payment/address/

Request: `GET /api/1/payment/address/:currency`

Example request: `GET /api/1/payment/address/BTC`

Summary: returns the last created incoming cryptocurrency address

Parameters: no parameters

Return values: returns an address that can be used to deposit cryptocurrency.

Example response:

```json
{"address":"1HDtDgG9HYpp1YJ6kFYSB6NgaG2haKnxUH"}
```

### /api/1/payment/address/

Request: `POST /api/1/payment/address/:currency`

Example request: `POST /api/1/payment/address/BTC`

Summary: creates a incoming new cryptocurrency address.

Parameters: no parameters

Return values: returns an address that can be used to deposit cryptocurrency.

Example response:

```json
{"address":"1HDtDgG9HYpp1YJ6kFYSB6NgaG2haKnxUH"}
```

### /api/1/payment/payout

Request: `POST /api/1/payment/payout`

Summary: withdraws money and creates an outgoing crypotocurrency transaction.

Parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `amount` | decimal, required | amount |
| `currency_code` | string, required, e.g. `BTC` | |
| `address` | string, required | BTC/LTC address |

Example: ```amount=0.001&currency_code=BTC&address=1LuWvENyuPNHsHWjDgU1QYKWUYN9xxy7n5```

Return values: returns a transaction id on the exchange or an error.

Example response:
```json
{"transaction": "51545-103004-18442681"}
```

### /api/1/payment/transactions

Request: `GET /api/1/payment/transactions`

Summary: return a list of payment transactions and their statuses.

Parameters:

| Parameter | Type | Description |
| --- | --- | --- |
| `offset` | int, optional, default = 0 | start index for the query |
| `limit` | int, required | max results for the query |
| `dir` | string, optional, `ask` or `desc` default = `desc` | sort direction |

Response: array of transactions.

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

## socket.io Market Data

The socket.io market data based on socket.io protocol and supports WebSocket, xhr-polling and jsonp-polling transports.

The API support multiplexing a single connection with socket.io namespaces. 

Please refer to official socket.io documentation on http://socket.io/.

Socket.io URL: `http://api.hitbtc.com:8081`

Socket.io demo URL: `http://demo-api.hitbtc.com:8081`

### `trades` namespace

Namespace: `trades`

URLs: `/trades/:symbol` e.g. `/trades/BTCUSD`

Event: `trade`

Event example:
```json
{"price":478.33,"amount":0.15}
```

Live example (both demo and primary api): http://jsfiddle.net/He6AU/13/

## Market data streaming end-point

Streaming API is based on [WebSocket protocol](http://en.wikipedia.org/wiki/WebSocket). All messages are in JSON format.

URL: [ws://api.hitbtc.com](ws://api.hitbtc.com)

Demo URL: [ws://demo-api.hitbtc.com]ws://demo-api.hitbtc.com)

Once client connects to this URL the session is started. 

The server broadcasts the following types of messages:

* [MarketDataSnapshotFullRefresh](#MarketDataSnapshotFullRefresh) message contains a full snapshot of the order book.
* [MarketDataIncrementalRefresh](#MarketDataIncrementalRefresh) message contains incremental changes

Some recommendations to consider:

* The application could receive the first snapshot and maintain the order book by applying incremental updates.
* It's recommended to invalidate a state of the application periodically using snapshots.
* It's recommended to check sequence numbers and to drop updates with non-monotonous sequence numbers.

<a name="MarketDataSnapshotFullRefresh"/>
### MarketDataSnapshotFullRefresh

MarketDataSnapshotFullRefresh message contains a full snapshot of the order book.

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
### MarketDataIncrementalRefresh

MarketDataIncrementalRefresh contains incremental changes of the order book and individual trades.

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

## Trading streaming end-point

Streaming API is based on [WebSocket protocol](http://en.wikipedia.org/wiki/WebSocket). All messages are in JSON format.

URL: <wss://api.hitbtc.com:8080>

Demo URL: <ws://demo-api.hitbtc.com:8080>

Trading endpoint requires sending login message after connection esteblished. All client messages should be signed and should contain valid and active API key

The following message types are supported:

| Type |  |
| --- | --- | --- |
| [Login](#Login) | Client -> Server |
| [NewOrder](#NewOrder) | Client -> Server |
| [OrderCancel](#OrderCancel) | Client -> Server  |
| [ExecutionReport](#ExecutionReport) | Server -> Client |
| [CancelReject](#CancelReject) | Server -> Client |


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

## Useful tools

Chrome extension Simple WebSocket Client (https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo)

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


