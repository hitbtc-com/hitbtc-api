## MEET NEW VERSION OF HITBTC API! 

HitBTC REST & Streaming API version 2.0 provides programmatic access to HitBTC’s next generation trading engine.

We strongly recommend using APIv2 for new users and switching to this newer version for our current traders to get the best trading experience. 

Please find API version 1.0 here: [APIv1 reference](https://github.com/hitbtc-com/hitbtc-api/blob/master/APIv1.md)

By using HitBTC API you confirm that you have read and accept [API License Agreement](https://hitbtc.com/api-license-agreement)

## Table of Contents
 * [Development guide](#development-guide)
 * [Best practices](#best-practices)
 * [REST API Reference](#rest-api-reference)
    * [API Explorer](https://api.hitbtc.com/api/2/explore/)
    * [Examples](#api-sample-code)
    * Market data
        * [Currencies](#currencies)
        * [Symbols](#symbols)
        * [Tickers](#tickers)
        * [Trades](#trades)
        * [Orderbook](#orderbook)
        * [Candles](#candles)
    * [Authentication](#authentication)
    * Trading
        * [Get trading balance](#trading-balance)
        * [Get active orders](#get-active-orders)   
        * [Get active order](#get-active-order-by-clientorderid)   
        * [Create new order](#create-new-order)   
        * [Cancel order](#cancel-order-by-clientorderid)   
        * [Cancel all orders](#cancel-orders)
        * [Trading commission](#get-trading-commission)
    * [Trading history](#trading-history)
        * [Orders history](#orders-history)
        * [Trades history](#trades-history)
        * [Trades by order](#trades-by-order)            
    * [Account](#account-information)
        * [Account balance](#account-balance)
        * [Deposit address](#deposit-crypto-address)
        * [Withdraw crypto](#withdraw-crypto)
        * [Commit or Rollback Withdraw crypto](#withdraw-crypto-commit-or-rollback)
        * [Transfer between trading and account](#transfer-money-between-trading-and-account)
        * [Transactions](#get-transactions-history)    
 * [Socket API Reference](#socket-api-reference)
    * Market Data
        * [Get Currencies](#get-currencies)
        * [Get Symbols](#get-symbols)
        * [Subscribe to ticker](#subscribe-to-ticker)
        * [Subscribe to orderbook](#subscribe-to-orderbook)
        * [Subscribe to trades](#subscribe-to-trades)
        * [Get trades](#get-trades)
        * [Subscribe to candles](#subscribe-to-candles)
    * [Authentication](#authentication-session)
    * [Trading](#socket-trading)
        * [Subscribe to reports](#subscribe-to-reports)
        * [Place New Order](#place-new-order)
        * [Cancel Order](#cancel-order)
        * [Cancel Replace Orders](#cancel-replace-orders)
        * [Get active Orders](#get-active-orders)
        * [Get trading balance](#get-trading-balance)    
    
## DEVELOPMENT GUIDE

### API URLs

| Environment | REST                              | Streaming |
|-------------|:---------------------------------|:-----|
| PROD        | https://api.hitbtc.com/api/2      | wss://api.hitbtc.com/api/2/ws | 

### DateTime Format

All timestamps are returned in ISO8601 format in UTC. Example: "2017-04-03T10:20:49.315Z"

### Number Format

All finance data, i.e. price, quantity, fee etc., should be arbitrary precision numbers and string representation. Example: "10.2000058"

### Pagination

Parameters:

| Parameter | Description |
|-----------|-------------|
| limit | Number of results per call. Accepted values: 0 - 1000. Default 100 |
| offset | Number of results offset. Default 0 |
| sort | Sort direction. Accepted values: ASC, DESC. Default DESC |
| by | Filtration definition. Accepted values: id, timestamp | 
| from | If filter by timestamp, then datetime. Otherwise object id |
| till | If filter by timestamp, then datetime. Otherwise object id |




## BEST PRACTICES
  
The HitBTC API development team strives to bring the best trading experience for our API users. Here is a set of best practices of using the API as efficiently as possible.

### HTTP Persistent Connection
It keeps the underlying TCP connection active for multiple requests/responses. Subsequent requests will result in reduced latency as the TCP handshaking process is no longer required.
    
If you are using an HTTP 1.0 client, please ensure it supports the keep-alive directive and submit the header Connection: Keep-Alive with your request.
    
Keep-Alive is a part of the protocol in HTTP 1.1 and enabled by default on compliant clients. However, you will have to ensure your implementation does not set the connection header to other values.

### Retrieving and updating account state

Use Streaming API for real time updates of your orders and trades and any transactions changes.

## REST API Reference

### HTTP Status codes

 * 200 OK Successful request
 * 400 Bad Request. Returns JSON with the error message
 * 401 Unauthorized. Authorisation required or failed
 * 403 Forbidden. Action is forbidden for API key
 * 429 Too Many Requests. Your connection is being rate-limited
 * 500 Internal Server. Internal Server Error
 * 503 Service Unavailable. Service is down for maintenance
 * 504 Gateway Timeout. Request timeout expired


### Error response

All error responses have error `code` and human readable `message` fields. Some errors contain additional `description` field.

**Example error response:**
```json
{
  "error": {
    "code": 20001,
    "message": "Insufficient funds",
    "description": "Check that the funds are sufficient, given commissions"
  }
}
```
    
**List of errors**
    
| Error code | 	HTTP Status Code | Message | Note |
|------------|-------------------|:--------|:-----|
| 403   | 401 | Action is forbidden for account | |     
| 429   | 429 | Too many requests | Action is being rate limited for account |
| 500   | 500 | Internal Server Error | |
| 503   | 503 | Service Unavailable | Try it again later |
| 504   | 504 | Gateway Timeout | Check the result of your request later |
| 1001  | 401 | Authorisation required | |    
| 1002  | 401 | Authorisation failed | |    
| 1003  | 403 | Action is forbidden for this API key | Check permissions for API key |    
| 1004  | 401 | Unsupported authorisation method | Use Basic authentication |    
| 2001  | 400 | Symbol not found |  |
| 2002  | 400 | Currency not found |  |
| 20001 | 400 | Insufficient funds | Insufficient funds for creating order or any account operation |
| 20002 | 400 | Order not found | Attempt to get active order that  not existing, filled, canceled or expired. Attempt to cancel not existing order. Attempt to cancel already filled or expired order. |
| 20003 | 400 | Limit exceeded | Withdrawal limit exceeded |
| 20004 | 400 | Transaction not found | Requested transaction not found |
| 20005 | 400 | Payout not found |  |
| 20006 | 400 | Payout already committed |  |
| 20007 | 400 | Payout already rolled back |  |
| 20008 | 400 | Duplicate clientOrderId |  |
| 10001 | 400 | Validation error | Input not valid, see more in `message` field| |

### API Explorer

You can explore API using SwaggerUI (https://api.hitbtc.com/api/2/explore/)

### API Sample code

 * Python rest example [example_rest.py](example_rest.py)

### Market data

#### Currencies

`GET /api/2/public/currency`

`GET /api/2/public/currency/{currency}`

Return the actual list of available currencies, tokens, ICO etc.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Currency identifier. In the future, the description will simply use the `currency` |
| fullName | String | Currency full name |
| crypto | Boolean | Is currency belongs to blockchain (false for ICO and fiat, like EUR) |
| payinEnabled | Boolean | Is allowed for deposit (false for ICO) |
| payinPaymentId | Boolean | Is required to provide additional information other than the address for deposit |
| payinConfirmations | Number | Blocks confirmations count for deposit |
| payoutEnabled | Boolean | Is allowed for withdraw (false for ICO) |
| payoutIsPaymentId | Boolean | Is allowed to provide additional information for withdraw |
| transferEnabled | Boolean | Is allowed to transfer between trading and account (may be disabled on maintain) |


Example data:
```json
[
   {
      "id": "BTC",
      "fullName": "Bitcoin",
      "crypto": true,
      "payinEnabled": true,
      "payinPaymentId": false,
      "payinConfirmations": 2,
      "payoutEnabled": true,
      "payoutIsPaymentId": false,
      "transferEnabled": true
   },
   {
      "id": "ETH",
      "fullName": "Ethereum",
      "crypto": true,
      "payinEnabled": true,
      "payinPaymentId": false,
      "payinConfirmations": 2,
      "payoutEnabled": true,
      "payoutIsPaymentId": false,
      "transferEnabled": true
   }
]
```
 
#### Symbols

`GET /api/2/public/symbol`

`GET /api/2/public/symbol/{symbol}`

Return the actual list of currency symbols (currency pairs) traded on HitBTC exchange.
The first listed currency of a symbol is called the base currency, and the second currency is called the quote currency.
The currency pair indicates how much of the quote currency is needed to purchase one unit of the base currency.
[Read more](http://www.investopedia.com/terms/c/currencypair.asp)
 
Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Symbol identifier. In the future, the description will simply use the `symbol` |
| baseCurrency | String | |
| quoteCurrency | String | |
| quantityIncrement | Number | |
| tickSize | Number | |
| takeLiquidityRate | Number | Default fee rate |
| provideLiquidityRate | Number | Default fee rate for market making trades |
| feeCurrency | String | |

Example data:
```json
[      
  {
    "id": "ETHBTC",
    "baseCurrency": "ETH",
    "quoteCurrency": "BTC",
    "quantityIncrement": "0.001",
    "tickSize": "0.000001",
    "takeLiquidityRate": "0.001",
    "provideLiquidityRate": "-0.0001",
    "feeCurrency": "BTC"
  }
]
```
    
    
#### Tickers

`GET /api/2/public/ticker`

`GET /api/2/public/ticker/{symbol}`

Return ticker information

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| ask | Number | Best ask price |
| bid | Number | Best bid price |
| last | Number | Last trade price |
| open | Number | Last trade price 24 hours ago |
| low | Number | Lowest trade price within 24 hours |
| high | Number | Highest trade price within 24 hours |
| volume | Number | Total trading amount within 24 hours in base currency |
| volumeQuote | Number | Total trading amount within 24 hours in quote currency |
| timestamp | Datetime | Last update or refresh ticker timestamp |
| symbol | String |  |
 
Example data: 
```json
[
  {
    "ask": "0.050043",
    "bid": "0.050042",
    "last": "0.050042",
    "open": "0.047800",
    "low": "0.047052",
    "high": "0.051679",
    "volume": "36456.720",
    "volumeQuote": "1782.625000",
    "timestamp": "2017-05-12T14:57:19.999Z",
    "symbol": "ETHBTC"
  }
]
```

#### Trades

`GET /api/2/public/trades/{symbol}`

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| sort | String | Default DESC |
| by | String | Filtration definition. Accepted values: id, timestamp. Default timestamp |
| from | Number or Datetime | |
| till | Number or Datetime | |
| limit | Number | | 
| offset | Number | |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | Number | Trade id |
| price | Number | Trade price |
| quantity | Number | Trade quantity |
| side | String | Trade side `sell` or `buy` |
| timestamp | Datetime | Trade timestamp |


Example response:
```json
[
  {
    "id": 9533117,
    "price": "0.046001",
    "quantity": "0.220",
    "side": "sell",
    "timestamp": "2017-04-14T12:18:40.426Z"
  },
  {
    "id": 9533116,
    "price": "0.046002",
    "quantity": "0.022",
    "side": "buy",
    "timestamp": "2017-04-14T11:56:37.027Z"
  }
]
```
    
#### Orderbook

`GET /api/2/public/orderbook/{symbol}`

An order book is an electronic list of buy and sell orders for a specific symbol, organized by price level.

Parameters: 

| Name | Type | Description |
|:---|:---:|:---|
| limit | Number | Limit of orderbook levels, default 100. Set 0 to view full orderbook levels |


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| ask | Array | Ask side array of levels |
| bid | Array | Bid side array of levels |
| size | Number | Total volume of orders with the specified price |
| price | Number | Price level |


Example response:
```json
{
  "ask": [
    {
      "price": "0.046002",
      "size": "0.088"
    },
    {
      "price": "0.046800",
      "size": "0.200"
    }
  ],
  "bid": [
    {
      "price": "0.046001",
      "size": "0.005"
    },
    {
      "price": "0.046000",
      "size": "0.200"
    }
  ]
}
```
#### Candles

`GET /api/2/public/candles/{symbol}`

An candles used for [OHLC](https://en.wikipedia.org/wiki/Open-high-low-close_chart) a specific symbol.

Parameters: 

| Name | Type | Description |
|:---|:---:|:---|
| limit | Number | Limit of candles, default 100. |
| period | String | One of: M1 (one minute), M3, M5, M15, M30, H1, H4, D1, D7, 1M (one month). Default is M30 (30 minutes). |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| timestamp | Datetime |  |
| open | Number | Open price |
| close | Number | Close price |
| min | Number | Min price |
| max | Number | Max price |
| volume | Number | Volume in base currency |
| volumeQuote | Number | Volume in quote currency |

> Result contain candles only with non zero volume

Example response:
```json
[
  {
    "timestamp": "2017-10-20T20:00:00.000Z",
    "open": "0.050459",
    "close": "0.050087",
    "min": "0.050000",
    "max": "0.050511",
    "volume": "1326.628",
    "volumeQuote": "66.555987736"
  },
  {
    "timestamp": "2017-10-20T20:30:00.000Z",
    "open": "0.050108",
    "close": "0.050139",
    "min": "0.050068",
    "max": "0.050223",
    "volume": "87.515",
    "volumeQuote": "4.386062831"
  }
]
```

### Authentication

Public market data available without authentication, for other requests authentication is required.
 
You should create API keys on [HitBTC API Setting](https://hitbtc.com/settings/api-keys) page. You can create multiple API keys with different permissions for your applications.
 
Use Basic Authentication to access REST API.
 
Example:
 
    curl -u "publicKey:secret" https://api.hitbtc.com/api/2/trading/balance 
    
### Trading Balance

`GET /api/2/trading/balance`    

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String |  |
| available | Number | Amount available for trading or transfer to main account |
| reserved | Number | Amount reserved for active orders or incomplete transfers to main account |

Example response:    
```json
    [
      {
        "currency": "ETH",
        "available": "10.000000000",
        "reserved": "0.560000000"
      },
      {
        "currency": "BTC",
        "available": "0.010205869",
        "reserved": "0"
      }
    ]
```
      
Curl example:
      
    curl -X GET -u "ff20f250a7b3a414781d1abe11cd8cee:fb453577d11294359058a9ae13c94713" "https://api.hitbtc.com/api/2/trading/balance"
      
Python 3 example:
```python3
    import requests    
    
    r = requests.get('https://api.hitbtc.com/api/2/trading/balance', auth=('ff20f250a7b3a414781d1abe11cd8cee', 'fb453577d11294359058a9ae13c94713'))    
    print(r.json())
```      

### Trading
    
Order model:

| Name | Type | Description |
|:---|:---:|:---|    
| id | Number | Unique identifier for Order as assigned by exchange |
| clientOrderId | String | Unique identifier for Order as assigned by trader. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| symbol| String| Trading symbol |
| side | String | **sell**  **buy** |
| status | String | new, suspended, partiallyFilled, filled, canceled, expired |
| type | String | Enum: limit, market, stopLimit, stopMarket |
| timeInForce | String | Time in force is a special instruction used when placing a trade to indicate how long an order will remain active before it is executed or expires <br> **GTC** - Good till cancel.  GTC order won't close until it is filled. <br> **IOC** - An immediate or cancel order is an order to buy or sell that must be executed immediately, and any portion of the order that cannot be immediately filled is cancelled. <br> **FOK** - Fill or kill is a type of time-in-force designation used in securities trading that instructs a brokerage to execute a transaction immediately and completely or not at all. <br> **Day** - keeps the order active until the end of the trading day in UTC. <br> **GTD** - Good till date specified in `expireTime`. |
| quantity | Number | Order quantity | 
| price | Number | Order price |
| cumQuantity | Number | Cumulative executed quantity |
| createdAt | Datetime | |
| updatedAt | Datetime | |
| stopPrice | Number | |
| expireTime | Datetime |  |     
    
#### Get Active orders
    
`GET /api/2/order`

Return array of active orders.

Parameters: 

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter active orders by symbol |

Response: Array of orders

Example response:
```json
[
  {
    "id": 840450210,
    "clientOrderId": "c1837634ef81472a9cd13c81e7b91401",
    "symbol": "ETHBTC",
    "side": "buy",
    "status": "partiallyFilled",
    "type": "limit",
    "timeInForce": "GTC",
    "quantity": "0.020",
    "price": "0.046001",
    "cumQuantity": "0.005",
    "createdAt": "2017-05-12T17:17:57.437Z",
    "updatedAt": "2017-05-12T17:18:08.610Z"
  }
]
```

#### Get Active order by clientOrderId
    
`GET /api/2/order/{clientOrderId}`

Parameters: 

| Name | Type | Description |
|:---|:---:|:---|
| wait | Number | Optional parameter. Time in milliseconds. Max 60000. Default none. Use long polling request, if order is filled, canceled or expired return order info instantly, or after specified wait time returns actual order info |

Response: Order

Example response:
```json
{
    "id": 840450210,
    "clientOrderId": "c1837634ef81472a9cd13c81e7b91401",
    "symbol": "ETHBTC",
    "side": "buy",
    "status": "partiallyFilled",
    "type": "limit",
    "timeInForce": "GTC",
    "quantity": "0.020",
    "price": "0.046001",
    "cumQuantity": "0.005",
    "createdAt": "2017-05-12T17:17:57.437Z",
    "updatedAt": "2017-05-12T17:18:08.610Z"
}
```
        
#### Create New Order

`POST /api/2/order`        
`PUT /api/2/order/{clientOrderId}`        

**Price accuracy and quantity.**

Symbol config contain `tickSize` parameter which means that `price` should be divide by `tickSize` without residue. `Quantity` should be divide by `quantityIncrement` without residue. By default, if `strictValidate` not enabled, server round half down `price` and `quantity` for `tickSize` and `quantityIncrement`.

Example for ETHBTC: `tickSize` = '0.000001' then price '0.046016' - valid, '0.0460165' - invalid    

**Fees**

Fee charged in symbol `feeCurrency`. Maker-taker fees offer a transaction rebate `provideLiquidityRate` to those who provide liquidity (the market maker), while charging customers who take that liquidity `takeLiquidityRate`.

For buy orders you must have enough available balance including fees. `Available balance > price * quantity * (1 + takeLiquidityRate)`                                    

**Result order status**

For orders with `timeInForce` `IOC` or `FOK` REST API returns final order state: filled or expired. 
 
If order can be instantly executed, then REST API return filled or partiallyFilled order's info. 

Parameters: 

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Optional parameter, if skipped - will be generated by server. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| symbol| String| Trading symbol |
| side | String | **sell** **buy** |
| type | String | Optional. Default - limit. One of: limit, market, stopLimit, stopMarket |
| timeInForce | String | Optional. Default - GDC. One of: GTC, IOC, FOK, Day, GTD |
| quantity | Number | Order quantity | 
| price | Number | Order price. Required for limit types. |
| stopPrice | Number | Required for stop types. |
| expireTime | Datetime | Required for GTD timeInForce. |    
| strictValidate | Boolean | Price and quantity will be checked that they increment within tick size and quantity step. See symbol `tickSize` and `quantityIncrement` |

Response: Order

Python 3 example:
```python3
    import requests
    
    orderData = {'symbol':'ethbtc', 'side': 'sell', 'quantity': '0.063', 'price': '0.046016' }
    r = requests.post('https://api.hitbtc.com/api/2/order', data = orderData, auth=('ff20f250a7b3a414781d1abe11cd8cee', 'fb453577d11294359058a9ae13c94713'))        
    print(r.json())

    > {'id': 0, 'side': 'sell', 'clientOrderId': 'd8574207d9e3b16a4a5511753eeef175', 'updatedAt': '2017-05-15T17:01:05.092Z', 'symbol': 'ETHBTC', 'type': 'limit', 'quantity': '0.063', 'price': '0.046016', 'status': 'new', 'cumQuantity': '0.000', 'createdAt': '2017-05-15T17:01:05.092Z', 'timeInForce': 'GTC'}

```
    
Example success created response:
```json
    {
        "id": 0,
        "clientOrderId": "d8574207d9e3b16a4a5511753eeef175",
        "symbol": "ETHBTC",
        "side": "sell",
        "status": "new",
        "type": "limit",
        "timeInForce": "GTC",
        "quantity": "0.063",
        "price": "0.046016",
        "cumQuantity": "0.000",
        "createdAt": "2017-05-15T17:01:05.092Z",
        "updatedAt": "2017-05-15T17:01:05.092Z"
    }
```    
    
Example error **Insufficient funds** response:
```json
{
  "error": {
    "code": 20001,
    "message": "Insufficient funds",
    "description": "Check that the funds are sufficient, given commissions"
  }
}
```    
    
#### Cancel orders

`DELETE /api/2/order`

Cancel all active orders, or all active orders for specified symbol. 

Parameters: 

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter active orders by symbol |


Response: Array of Orders
    
#### Cancel order by clientOrderId
    
`DELETE /api/2/order/{clientOrderId}`

Response: Order

Example response:
```json
{
  "id": 0,
  "clientOrderId": "d8574207d9e3b16a4a5511753eeef175",
  "symbol": "ETHBTC",
  "side": "sell",
  "status": "canceled",
  "type": "limit",
  "timeInForce": "GTC",
  "quantity": "0.000",
  "price": "0.046016",
  "cumQuantity": "0.000",
  "createdAt": "2017-05-15T17:01:05.092Z",
  "updatedAt": "2017-05-15T18:08:57.226Z"
}
```
    
Example error **Order not found** response:
```json
{
  "error": {
    "code": 20002,
    "message": "Order not found",
    "description": ""
  }
} 
```    
#### Get trading commission
    
`GET /api/2/trading/fee/{symbol}`

Get personal trading commission rate.

Example response:
```json
{
  "takeLiquidityRate": "0.001",
  "provideLiquidityRate": "-0.0001"
}
```

### Trading history

Please note, that trading history may be updated with delay up to 30 seconds, with mean delay around 1 second.

#### Orders history

`GET /api/2/history/order`

All orders older then 24 hours without trades are deleted.

Parameters: 
    
| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter active orders by symbol |
| clientOrderId | String | If set, other parameters will be ignored. Without limit and pagination |
| from | Datetime | |
| till | Datetime | |
| limit | Number | Default 100 |
| offset | Number | |

Response: Array of Orders

#### Trades history

`GET /api/2/history/trades`

Parameters: 
    
| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter active orders by symbol |
| sort | String | `DESC` or `ASC`. Default value `DESC` |
| by | String | `timestamp` by default, or `id` |
| from | Datetime or Number | |
| till | Datetime or Number | |
| limit | Number | Default 100 |
| offset | Number | |


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | Number | Unique identifier for Trade as assigned by exchange |
| orderId | Number | Unique identifier for Order as assigned by exchange |
| clientOrderId | String | Unique identifier for Order as assigned by trader. |
| symbol| String| Trading symbol |
| side | String | **sell**  **buy** |
| quantity | Number | Trade quantity | 
| price | Number | Trade price |
| fee | Number | Trade commission. Could be negative – reward. Fee currency see in symbol config |
| timestamp | Datetime | Trade timestamp |


Example response:
```json
[
  {
    "id": 9535486,
    "clientOrderId": "f8dbaab336d44d5ba3ff578098a68454",
    "orderId": 816088377,
    "symbol": "ETHBTC",
    "side": "sell",
    "quantity": "0.061",
    "price": "0.045487",
    "fee": "0.000002775",
    "timestamp": "2017-05-17T12:32:57.848Z"
  },
  {
    "id": 9535437,
    "clientOrderId": "27b9bfc068b44194b1f453c7af511ed6",
    "orderId": 816088021,
    "symbol": "ETHBTC",
    "side": "buy",
    "quantity": "0.038",
    "price": "0.046000",
    "fee": "-0.000000174",
    "timestamp": "2017-05-17T12:30:57.848Z"
  }
]
```

#### Trades by order

`GET /api/2/history/order/{orderId}/trades`

Parameters: 
    
| Name | Type | Description |
|:---|:---:|:---|
| orderId | Number | Order id |

> Provide order id, because clientOrderId can be not uniq during long period.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | Number | Unique identifier for Trade as assigned by exchange |
| orderId | Number | Unique identifier for Order as assigned by exchange |
| clientOrderId | String | Unique identifier for Order as assigned by trader. |
| symbol| String| Trading symbol |
| side | String | **sell**  **buy** |
| quantity | Number | Trade quantity | 
| price | Number | Trade price |
| fee | Number | Trade commission. Could be negative – reward. Fee currency see in symbol config |
| timestamp | Datetime | Trade timestamp |    
    
### Account information

#### Account balance

`GET /api/2/account/balance`

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String |  |
| available | Number | Amount available for withdraw or transfer to trading account |
| reserved | Number | Amount reserved for incomplete transactions |

Response example:
```json
[
  {
    "currency": "BTC",
    "available": "0.0504600",
    "reserved": "0.0000000"
  },
  {
    "currency": "ETH",
    "available": "30.8504600",
    "reserved": "0.0000000"
  }
]
```

#### Deposit crypto address

`GET /api/2/account/crypto/address/{currency}` - Get current address

`POST /api/2/account/crypto/address/{currency}` - Create new address

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| address | String | Address for deposit |
| paymentId | String | Optional additional parameter. Required for deposit if persist |

Response example:
```json
{
  "address": "NXT-G22U-BYF7-H8D9-3J27W",
  "paymentId": "616598347865"
}
```


#### Withdraw crypto

`POST /api/2/account/crypto/withdraw`
 
Parameters: 
    
| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency |
| amount | Number | The amount that will be sent to the specified address |
| address | String |  |
| paymentId | String | Optional parameter |
| networkFee | Number or String | Optional parameter. Too low and too high commission value will be rounded to valid values. |
| includeFee | Boolean | Default false. If set true then total will be spent the specified amount, fee and networkFee will be deducted from the amount |
| autoCommit | Boolean | Default true. If set false then you should commit or rollback transaction in an hour. Used in two phase commit schema. |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Unique identifier for Transaction as assigned by exchange |
 
Response example:
```json
{
  "id": "d2ce578f-647d-4fa0-b1aa-4a27e5ee597b"
}
```
#### Withdraw crypto commit or rollback
 
`PUT /api/2/account/crypto/withdraw/{id}` - Commit

`DELETE /api/2/account/crypto/withdraw/{id}` - Rollback

> Both methods idempotent

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| result | Boolean | True of request completed |

Response example:
```json
{
  "result": true
}
```
 
#### Transfer money between trading and account

`POST /api/2/account/transfer`

Parameters: 
    
| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency |
| amount | Number | The amount that will transfer between balances |
| type | String | **bankToExchange** or **exchangeToBank** |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Unique identifier for Transaction as assigned by exchange |
 
Response example:
```json
{
  "id": "d2ce578f-647d-4fa0-b1aa-4a27e5ee597b"
}
```
    

#### Get transactions history
 
`GET /api/2/account/transactions`
 
`GET /api/2/account/transactions/{id}` - get transaction by transaction id
  
Parameters: 
    
| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency |
| sort | String | `DESC` or `ASC`. Default value `DESC` |
| by | String | `timestamp` or `index`. Default value `timestamp` |
| from | Datetime or Number | If sort by timestamp then Datetime, otherwise Number of index value |
| till | Datetime or Number | If sort by timestamp then Datetime, otherwise Number of index value |
| limit | Number | Default 100 |
| offset | Number | |


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Unique identifier for Transaction as assigned by exchange |
| index | Number | Is the internal index value that represents when the entry was updated |
| currency | String | Currency |
| amount | Number |  |
| fee | Number | |
| networkFee | Number | |
| address | String | |
| paymentId | String | |
| hash | String | Transaction hash  |
| status | String | **pending**, **failed**, **success** |
| type | String | One of: **payout** - crypto withdraw transaction, **payin** - crypto deposit transaction, **deposit**, **withdraw**, **bankToExchange**, **exchangeToBank** |
| createdAt | Datetime |  |
| updatedAt | Datetime |  |

Example response:
```json
[
  {
    "id": "6a2fb54d-7466-490c-b3a6-95d8c882f7f7",
    "index": 20400458,
    "currency": "ETH",
    "amount": "38.616700000000000000000000",
    "fee": "0.000880000000000000000000",
    "networkFee": "0.000420000000000000000000",
    "address": "0xfaEF4bE10dDF50B68c220c9ab19381e20B8EEB2B",        
    "hash": "eece4c17994798939cea9f6a72ee12faa55a7ce44860cfb95c7ed71c89522fe8",
    "status": "pending",
    "type": "payout",
    "createdAt": "2017-05-18T18:05:36.957Z",
    "updatedAt": "2017-05-18T19:21:05.370Z"
  }
]
```

## Socket API Reference

Use JSON-RPC 2.0 over WebSocket connection as transport.

### Request object

A rpc call is represented by sending a Request object to a Server. 

The Request object has the following members:
 - **method** - A String containing the name of the method to be invoked.
 - **params** - A Structured value that holds the parameter values to be used during the invocation of the method.
 - **id** - An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD normally not be Null.

### Notification

A Notification is a Request object without an "id" member. A Request object that is a Notification signifies the Client's lack of interest in the corresponding Response object, and as such no Response object needs to be returned to the client.

The Notification object has the following members:
 - **method** - A String containing the name of the method to be invoked.
 - **params** - A Structured value that holds the parameter values to be used during the invocation of the method.
 
### Response object

When a rpc call is made, the Server MUST reply with a Response, except for in the case of Notifications. 

The Response is expressed as a single JSON Object, with the following members:
 - **result**  - This member is REQUIRED on success. This member MUST NOT exist if there was an error invoking the method. The value of this member is determined by the method invoked on the Server.
 - **error** - This member is REQUIRED on error. This member MUST NOT exist if there was no error triggered during invocation. The value for this member MUST be an Object as defined in section [Error response](#error_response) 
 
### Market Data

The parameters of requests, responses, and errors correspond to REST, but usage flow differ. 
First you should subscribe for interested data. 
Then server send full snapshot of data, after that server send update notification.

Server response for every request. On success subscription response is true. Example:
```json
{
  "jsonrpc": "2.0",
  "result": true,
  "id": 123
}
```


#### Get Currencies

**Request**:

Methods: **getCurrencies**,**getCurrency**

Example:
```json
{
  "method": "getCurrency",
  "params": {
    "currency": "ETH"
  },
  "id": 123
}
```
    
**Response**

Example:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "id": "ETH",
    "fullName": "Ethereum",
    "crypto": true,
    "payinEnabled": true,
    "payinPaymentId": false,
    "payinConfirmations": 2,
    "payoutEnabled": true,
    "payoutIsPaymentId": false,
    "transferEnabled": true
  },
  "id": 123
}
```

#### Get Symbols

**Request**

Methods: **getSymbols**,**getSymbol**

Example:
```json
{
  "method": "getSymbol",
  "params": {
    "symbol": "ETHBTC"
  },
  "id": 123
}
```

**Response**

Example:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "id": "ETHBTC",
    "baseCurrency": "ETH",
    "quoteCurrency": "BTC",
    "quantityIncrement": "0.001",
    "tickSize": "0.000001",
    "takeLiquidityRate": "0.001",
    "provideLiquidityRate": "-0.0001",
    "feeCurrency": "BTC"
  },
  "id": 123
}
```


### Subscribe to Ticker

**Request**:

Methods: **subscribeTicker**, **unsubscribeTicker** 


Example:
```json
{
  "method": "subscribeTicker",
  "params": {
    "symbol": "ETHBTC"
  },
  "id": 123
}
```

**Notification**:

Method: **ticker**

Example:
```json
{
  "jsonrpc": "2.0",
  "method": "ticker",
  "params": {
    "ask": "0.054464",
    "bid": "0.054463",
    "last": "0.054463",
    "open": "0.057133",
    "low": "0.053615",
    "high": "0.057559",
    "volume": "33068.346",
    "volumeQuote": "1832.687530809",
    "timestamp": "2017-10-19T15:45:44.941Z",
    "symbol": "ETHBTC"
  }
}
```

### Subscribe to Orderbook

**Request**

Methods: **subscribeOrderbook**, **unsubscribeOrderbook**

Example:
```json
{
  "method": "subscribeOrderbook",
  "params": {
    "symbol": "ETHBTC"
  },
  "id": 123
}
```

**Notification snapshot**

Message contains a full snapshot of order book.

Method: **snapshotOrderbook**

Example:
```json
{
  "jsonrpc": "2.0",
  "method": "snapshotOrderbook",
  "params": {
    "ask": [
      {
        "price": "0.054588",
        "size": "0.245"
      },
      {
        "price": "0.054590",
        "size": "0.000"
      },
      {
        "price": "0.054591",
        "size": "2.784"
      },
      {
        "price": "0.054592",
        "size": "35.000"
      },
      {
        "price": "0.054595",
        "size": "20.000"
      },
      {
        "price": "0.054600",
        "size": "0.030"
      }
    ],
    "bid": [
      {
        "price": "0.054558",
        "size": "0.500"
      },
      {
        "price": "0.054557",
        "size": "0.076"
      },
      {
        "price": "0.054524",
        "size": "7.725"
      },
      {
        "price": "0.054523",
        "size": "0.522"
      },
      {
        "price": "0.054520",
        "size": "20.000"
      }
    ],
    "symbol": "ETHBTC",
    "sequence": 8073827
  }
}
```

**Notification update**

Message contains incremental changes

> **size = 0** means that level was deleted

> **sequence** is monotone increasing for each update, each symbol has its own sequence

Method: **updateOrderbook**

Example:
```json
{
  "jsonrpc": "2.0",
  "method": "updateOrderbook",
  "params": {
    "data": {
      "ask": [
        {
          "price": "0.054590",
          "size": "0.000"
        },
        {
          "price": "0.054591",
          "size": "0.000"
        },
        {
          "price": "0.054639",
          "size": "2.113"
        }
      ],
      "bid": [
        {
          "price": "0.054521",
          "size": "2.887"
        },
        {
          "price": "0.054504",
          "size": "0.000"
        }
      ],
      "symbol": "ETHBTC",
      "sequence": 8073830
    }
  }
}
```

### Subscribe to Trades

Request

Methods: **subscribeTrades**, **unsubscribeTrades**

Example:
```json
{
  "method": "subscribeTrades",
  "params": {
    "symbol": "ETHBTC"
  },
  "id": 123
}
```    
    
Notification

Method: **snapshotTrades**

Example:
```json
{
  "jsonrpc": "2.0",
  "method": "snapshotTrades",
  "params": {
    "data": [
      {
        "id": 54469456,
        "price": "0.054656",
        "quantity": "0.057",
        "side": "buy",
        "timestamp": "2017-10-19T16:33:42.821Z"
      },
      {
        "id": 54469497,
        "price": "0.054656",
        "quantity": "0.092",
        "side": "buy",
        "timestamp": "2017-10-19T16:33:48.754Z"
      },
      {
        "id": 54469697,
        "price": "0.054669",
        "quantity": "0.002",
        "side": "buy",
        "timestamp": "2017-10-19T16:34:13.288Z"
      }
    ],
    "symbol": "ETHBTC"
  }
}
```

Notification update

Method: **updateTrades**

Example:
```json
{
  "jsonrpc": "2.0",
  "method": "updateTrades",
  "params": {
    "data": [
      {
        "id": 54469813,
        "price": "0.054670",
        "quantity": "0.183",
        "side": "buy",
        "timestamp": "2017-10-19T16:34:25.041Z"
      }
    ],
    "symbol": "ETHBTC"
  }
}    
```

### Get Trades

Request

Method: **getTrades**

> Use use the same parameters as for REST 

Example:
```json
{
  "method": "getTrades",
  "params": {
    "symbol": "ETHBTC",
    "limit": 3,
    "sort": "DESC",
    "by": "id"
  },
  "id": 123
}
```

Response

Example:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "data": [
      {
        "id": 54472171,
        "price": "0.054443",
        "quantity": "2.213",
        "side": "sell",
        "timestamp": "2017-10-19T16:39:20.796Z"
      },
      {
        "id": 54472170,
        "price": "0.054453",
        "quantity": "0.030",
        "side": "sell",
        "timestamp": "2017-10-19T16:39:20.796Z"
      },
      {
        "id": 54472169,
        "price": "0.054454",
        "quantity": "0.052",
        "side": "sell",
        "timestamp": "2017-10-19T16:39:20.796Z"
      }
    ],
    "symbol": "ETHBTC"
  },
  "id": 123
}
```

### Subscribe to Candles

Request

Method: **subscribeCandles**, **unsubscribeCandles**

Params:

Example:
```json
{
  "method": "subscribeCandles",
  "params": {
    "symbol": "ETHBTC",
    "period": "M30"
  },
  "id": 123
}     
```
    
Notification snapshot

Method: **snapshotCandles**

Example:
```json
{
  "jsonrpc": "2.0",
  "method": "snapshotCandles",
  "params": {
    "data": [
      {
        "timestamp": "2017-10-19T15:00:00.000Z",
        "open": "0.054801",
        "close": "0.054625",
        "min": "0.054601",
        "max": "0.054894",
        "volume": "380.750",
        "volumeQuote": "20.844237223"
      },
      {
        "timestamp": "2017-10-19T15:30:00.000Z",
        "open": "0.054616",
        "close": "0.054618",
        "min": "0.054420",
        "max": "0.054724",
        "volume": "348.527",
        "volumeQuote": "19.011854364"
      },
      {
        "timestamp": "2017-10-19T16:00:00.000Z",
        "open": "0.054587",
        "close": "0.054626",
        "min": "0.054408",
        "max": "0.054768",
        "volume": "194.014",
        "volumeQuote": "10.595416973"
      },
      {
        "timestamp": "2017-10-19T16:30:00.000Z",
        "open": "0.054614",
        "close": "0.054443",
        "min": "0.054339",
        "max": "0.054724",
        "volume": "141.213",
        "volumeQuote": "7.706358298"
      }
    ],
    "symbol": "ETHBTC",
    "period": "M30"
  }
}
```

Notification update

Method: **updateCandles**

Example:
```json
{
  "jsonrpc": "2.0",
  "method": "updateCandles",
  "params": {
    "data": [
      {
        "timestamp": "2017-10-19T16:30:00.000Z",
        "open": "0.054614",
        "close": "0.054465",
        "min": "0.054339",
        "max": "0.054724",
        "volume": "141.268",
        "volumeQuote": "7.709353873"
      }
    ],
    "symbol": "ETHBTC",
    "period": "M30"
  }
}
```

### Authentication Session
    
You should authentication session once before request requires authorisation.    
    
Request

Method: **login**

Params:

| Name | Type | Description |
|:---|:---:|:---|
| algo | String | "BASIC" or "HS256" |
| pKey | String | API public key |
| sKey | String | API secret key, required on BASIC algo |
| nonce | String | Random string, required on HS256 algo |
| signature | String | HMAC SHA256 sign nonce with API secret key, required on HS256 algo |

Example BASIC:
```json
{
  "method": "login",
  "params": {
    "algo": "BASIC",
    "pKey": "3ef4a9f8c8bf04bd8f09884b98403eae",
    "sKey": "2deb570ab58fd553a4ed3ee249fd2d51"
  }
}
```    
    
Example HS256:
```json
{
  "method": "login",
  "params": {
    "algo": "HS256",
    "pKey": "3ef4a9f8c8bf04bd8f09884b98403eae",
    "nonce": "N1g287gL8YOwDZr",
    "signature": "b1c0ae399c2d341866a214f7d3ed755b821c1c36fc6f17083ef05fbb55b7f986"
  }
}
```    
    
### Socket Trading

Trade via socket have powerful changes to compared with REST:
 * Fast - time to place new order a bit higher than network latency.
 * Server notify on any order updates.
 * FIFO. Your requests execute in requested order.
    
#### Subscribe to reports

Request:

Method: **subscribeReports**

Example: 
```json
{
  "method": "subscribeReports",
  "params": {}
}
```

Notification snapshot:

> Notification with active orders send after subscription or on any service maintain.

Method: **activeOrders**

Example:
```json
{
  "jsonrpc": "2.0",
  "method": "activeOrders",
  "params": [
    {
      "id": "4345613661",
      "clientOrderId": "57d5525562c945448e3cbd559bd068c3",
      "symbol": "BCCBTC",
      "side": "sell",
      "status": "new",
      "type": "limit",
      "timeInForce": "GTC",
      "quantity": "0.013",
      "price": "0.100000",
      "cumQuantity": "0.000",
      "createdAt": "2017-10-20T12:17:12.245Z",
      "updatedAt": "2017-10-20T12:17:12.245Z",
      "reportType": "status"
    }
  ]
}
```
    
Notification report

Method: **report**

Params:

| Name | Type | Description |
|:---|:---:|:---|    
| id | Number | Unique identifier for Order as assigned by exchange |
| clientOrderId | String | Unique identifier for Order as assigned by trader. |
| symbol| String| Trading symbol |
| side | String | **sell** or  **buy** |
| status | String | new, suspended, partiallyFilled, filled, canceled, expired |
| type | String | Enum: limit, market, stopLimit, stopMarket |
| timeInForce | String | Time in force is a special instruction used when placing a trade to indicate how long an order will remain active before it is executed or expires <br> **GTC** - Good till cancel.  GTC order won't close until it is filled. <br> **IOC** - An immediate or cancel order is an order to buy or sell that must be executed immediately, and any portion of the order that cannot be immediately filled is cancelled. <br> **FOK** - Fill or kill is a type of time-in-force designation used in securities trading that instructs a brokerage to execute a transaction immediately and completely or not at all. <br> **Day** - keeps the order active until the end of the trading day in UTC. <br> **GTD** - Good till date specified in `expireTime`. |
| quantity | Number | Order quantity | 
| price | Number | Order price |
| cumQuantity | Number | Cumulative executed quantity |
| createdAt | Datetime | |
| updatedAt | Datetime | |
| stopPrice | Number | |
| expireTime | Datetime |  |   
| **reportType** | String | One of: status, new, canceled, expired, suspended, trade, replaced |
| tradeId | Number | Required for reportType = trade. Trade Id. |   
| tradeQuantity | Number | Required for reportType = trade. Quantity of trade.  |   
| tradePrice | Number | Required for reportType = trade. Trade price.  |   
| tradeFee | Number | Required for reportType = trade. Fee paid for trade. |   
| originalRequestClientOrderId | String | Identifier of replaced order. |   
   
    
Example:
```json
{
  "jsonrpc": "2.0",
  "method": "report",
  "params": {
    "id": "4345697765",
    "clientOrderId": "53b7cf917963464a811a4af426102c19",
    "symbol": "ETHBTC",
    "side": "sell",
    "status": "filled",
    "type": "limit",
    "timeInForce": "GTC",
    "quantity": "0.001",
    "price": "0.053868",
    "cumQuantity": "0.001",
    "createdAt": "2017-10-20T12:20:05.952Z",
    "updatedAt": "2017-10-20T12:20:38.708Z",
    "reportType": "trade",
    "tradeQuantity": "0.001",
    "tradePrice": "0.053868",
    "tradeId": 55051694,
    "tradeFee": "-0.000000005"
  }
}
```

#### Place New Order

Method: **newOrder**

Params:

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Required parameter. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| symbol| String| Trading symbol |
| side | String | **sell** **buy** |
| type | String | Optional. Default - limit. One of: limit, market, stopLimit, stopMarket |
| timeInForce | String | Optional. Default - GDC. One of: GTC, IOC, FOK, Day, GTD |
| quantity | Number | Order quantity | 
| price | Number | Order price. Required for limit types. |
| stopPrice | Number | Required for stop types. |
| expireTime | Datetime | Required for GTD timeInForce. |    
| strictValidate | Boolean | Price and quantity will be checked that they increment within tick size and quantity step. See symbol `tickSize` and `quantityIncrement` |


Example:
```json
{
  "method": "newOrder",
  "params": {
    "clientOrderId": "57d5525562c945448e3cbd559bd068c4",
    "symbol": "ETHBTC",
    "side": "sell",
    "price": "0.059837",
    "quantity": "0.015"
  },
  "id": 123
}
```

Example error response:
```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": 20001,
    "message": "Insufficient funds",
    "description": "Check that the funds are sufficient, given commissions"
  },
  "id": 123
}
```

Example success response:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "id": "4345947689",
    "clientOrderId": "57d5525562c945448e3cbd559bd068c4",
    "symbol": "ETHBTC",
    "side": "sell",
    "status": "new",
    "type": "limit",
    "timeInForce": "GTC",
    "quantity": "0.001",
    "price": "0.093837",
    "cumQuantity": "0.000",
    "createdAt": "2017-10-20T12:29:43.166Z",
    "updatedAt": "2017-10-20T12:29:43.166Z",
    "reportType": "new"
  },
  "id": 123
}
```

> Notification `report` will  arrived  before request result

#### Cancel Order

Method: **cancelOrder**

Example:
```json
{
  "method": "cancelOrder",
  "params": {
    "clientOrderId": "57d5525562c945448e3cbd559bd068c4"
  },
  "id": 123
}
```
    
Example response:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "id": "4345947689",
    "clientOrderId": "57d5525562c945448e3cbd559bd068c4",
    "symbol": "ETHBTC",
    "side": "sell",
    "status": "canceled",
    "type": "limit",
    "timeInForce": "GTC",
    "quantity": "0.001",
    "price": "0.093837",
    "cumQuantity": "0.000",
    "createdAt": "2017-10-20T12:29:43.166Z",
    "updatedAt": "2017-10-20T12:31:26.174Z",
    "reportType": "canceled"
  },
  "id": 123
}
```

#### Cancel Replace Orders

The order cancel/replace request is used to change the parameters of an existing order.

Cancel/Replace will be used to change quantity and price attribute of an open order

Do not use this message to cancel the remaining quantity of an outstanding order, use the Cancel Request message for this purpose.

Stipulates that a newly entered order is to cancel a prior order entered, but yet to be executed.

Method: **cancelReplaceOrder**

Params:
| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Required parameter. Replaced order|
| requestClientId | String | clientOrderId for new order Required parameter. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| quantity | Number | New order quantity | 
| price | Number | New order price. |
| strictValidate | Boolean | Price and quantity will be checked that they increment within tick size and quantity step. See symbol `tickSize` and `quantityIncrement` |

Example:
```json
{
  "method": "cancelReplaceOrder",
  "params": {
    "clientOrderId": "9cbe79cb6f864b71a811402a48d4b5b1",
    "requestClientId": "9cbe79cb6f864b71a811402a48d4b5b2",
    "quantity": "0.002",
    "price": "0.083837"
  },
  "id": 123
}
```

Example response:
```json
{
  "jsonrpc": "2.0",
  "result": {
    "id": "4346371528",
    "clientOrderId": "9cbe79cb6f864b71a811402a48d4b5b2",
    "symbol": "ETHBTC",
    "side": "sell",
    "status": "new",
    "type": "limit",
    "timeInForce": "GTC",
    "quantity": "0.002",
    "price": "0.083837",
    "cumQuantity": "0.000",
    "createdAt": "2017-10-20T12:47:07.942Z",
    "updatedAt": "2017-10-20T12:50:34.488Z",
    "reportType": "replaced",
    "originalRequestClientOrderId": "9cbe79cb6f864b71a811402a48d4b5b1"
  },
  "id": 123
}
```

#### Get active Orders

Method: **getOrders**

Example:
```json
{
  "method": "getOrders",
  "params": {},
  "id": 123
}
```
    
Example response:
```json
{
  "jsonrpc": "2.0",
  "result": [
    {
      "id": "4346371528",
      "clientOrderId": "9cbe79cb6f864b71a811402a48d4b5b2",
      "symbol": "ETHBTC",
      "side": "sell",
      "status": "new",
      "type": "limit",
      "timeInForce": "GTC",
      "quantity": "0.002",
      "price": "0.083837",
      "cumQuantity": "0.000",
      "createdAt": "2017-10-20T12:47:07.942Z",
      "updatedAt": "2017-10-20T12:50:34.488Z",
      "reportType": "replaced",
      "originalRequestClientOrderId": "9cbe79cb6f864b71a811402a48d4b5b1"
    }
  ],
  "id": 123
}
```  
    
#### Get trading balance
   
Method: **getTradingBalance**

Example:
```json
{
  "method": "getTradingBalance",
  "params": {},
  "id": 123
}
```
    
Example response:
```json
{
  "jsonrpc": "2.0",
  "result": [
    {
      "currency": "BCN",
      "available": "100.000000000",
      "reserved": "0"
    },
    {
      "currency": "BTC",
      "available": "0.013634021",
      "reserved": "0"
    },
    {
      "currency": "ETH",
      "available": "0",
      "reserved": "0.00200000"
    }
  ],
  "id": 123
}
```
