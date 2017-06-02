# Alpha version. Awaiting your proposals. 

## GETTING STARTED

HitBTC REST & Streaming API provides programmatic access to HitBTC’s next generation trading engine.

## DEVELOPMENT GUIDE

### API URLs

| Environment | REST                              | Streaming |
|-------------|:---------------------------------|:-----|
| PROD        | https://api.hitbtc.com/api/2      | wss://api.hitbtc.com/api/2 | 

### DateTime Format

All timestamps are returned in ISO8601 format in UTC. Example: "2017-04-03T10:20:49.315Z"

### Number Format

All finance data - price, quantity, fee and others, should be arbitrary precision numbers and string representation. Example: "10.2000058"

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
  
The HIitBTC API development team strives to bring the best overall experience for our API users. Here is a set of best practices to use the API as efficiently as possible.

### HTTP Persistent Connection
It keeps the underlying TCP connection active for multiple requests/responses. Subsequent requests will result in reduced latency as the TCP handshaking process is no longer required.
    
If you are using an HTTP 1.0 client, ensure it supports the keep-alive directive and submit the header Connection: Keep-Alive with your request.
    
Keep-Alive is part of the protocol in HTTP 1.1 and enabled by default on compliant clients. However, you will have to ensure your implementation does not set the connection header to other values.

### Retrieving and updating account state

Use Streaming API for real time updates of your orders and trades, any transactions changes.

## REST API Reference

### HTTP Status codes

 * 200 OK Successful request
 * 400 Bad Request. Returns JSON with the error message
 * 401 Unauthorized. Authorisation required or failed
 * 403 Forbidden. Action is forbidden for API key
 * 429 Too Many Requests. Your connection is being rate limited
 * 500 Internal Server. Internal Server Error
 * 503 Service Unavailable. Service is down for maintenance
 * 504 Gateway Timeout. Request timeout expired


### Error response

All error responses have error `code` and human readable `message` fields. Some errors contain additional `description` field.

**Example error response:**
         
    {
        'error': {
            'code': 20001, 
            'message': 'Insufficient funds', 
            'description': 'Check that the funds are sufficient, given commissions'
        }
    }
    
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
| 10001 | 400 | Validation error | Input not valid, see more in `message` field| |

### API Explorer

You can explore API using SwaggerUI (https://api.hitbtc.com/api/2/explore/)

### API Sample code

 * Python rest example [example_rest.py](example_rest.py)

### Market data
 
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
      },
    
    
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
| amount | Number | Amount available for trading or transfer to main account |
| reserved | Number | Amount reserved for active orders or incomplete transfers to main account |

Example response:    
    
    [
      {
        "currency": "ETH",
        "amount": "10.000000000",
        "reserved": "0.560000000"
      },
      {
        "currency": "BTC",
        "amount": "0.010205869",
        "reserved": "0"
      }
    ]
      
Curl example:
      
    curl -X GET -u "ff20f250a7b3a414781d1abe11cd8cee:fb453577d11294359058a9ae13c94713" "https://api.hitbtc.com/api/2/trading/balance"
      
Python 3 example:
      
    import requests    
    
    r = requests.get('https://api.hitbtc.com/api/2/trading/balance', auth=('ff20f250a7b3a414781d1abe11cd8cee', 'fb453577d11294359058a9ae13c94713'))    
    print(r.json())

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

#### Get Active order by clientOrderId
    
`GET /api/2/order/{clientOrderId}`

Parameters: 

| Name | Type | Description |
|:---|:---:|:---|
| wait | Number | Optional parameter. Time in milliseconds. Max 60000. Default none. Use long polling request, if order is filled, canceled or expired return order info instantly, or after specified wait time returns actual order info |

Response: Order

Example response:
   
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
| strictValidate | boolean | Price and quantity will be checked that they increment within tick size and quantity step. See symbol `tickSize` and `quantityIncrement` |

Response: Order

Python 3 example:
    
    import requests
    
    orderData = {'symbol':'ethbtc', 'side': 'sell', 'quantity': '0.063', 'price': '0.046016' }
    r = requests.post('https://api.hitbtc.com/api/2/order', data = orderData, auth=('ff20f250a7b3a414781d1abe11cd8cee', 'fb453577d11294359058a9ae13c94713'))        
    print(r.json())

    > {'id': 0, 'side': 'sell', 'clientOrderId': 'd8574207d9e3b16a4a5511753eeef175', 'updatedAt': '2017-05-15T17:01:05.092Z', 'symbol': 'ETHBTC', 'type': 'limit', 'quantity': '0.063', 'price': '0.046016', 'status': 'new', 'cumQuantity': '0.000', 'createdAt': '2017-05-15T17:01:05.092Z', 'timeInForce': 'GTC'}
    
Example success created response:
    
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
    
Example error **Insufficient funds** response:
    
    {
      "error": {
        "code": 20001,
        "message": "Insufficient funds",
        "description": "Check that the funds are sufficient, given commissions"
      }
    }
    
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
    
Example error **Order not found** response:
    
    {
      "error": {
        "code": 20002,
        "message": "Order not found",
        "description": ""
      }
    } 

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
    
    
### Account information

#### Account balance
`GET /api/2/account/balance`

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String |  |
| amount | Number | Amount available for withdraw or transfer to trading account |
| reserved | Number | Amount reserved for incomplete transactions |

Response example:

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

#### Deposit crypto address

`GET /api/2/account/crypto/address/{currency}` - Get current address

`POST /api/2/account/crypto/address/{currency}` - Create new address

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| address | String | Address for deposit |
| paymentId | String | Optional additional parameter. Required for deposit if persist |

Response example:

    {
      "address": "NXT-G22U-BYF7-H8D9-3J27W",
      "paymentId": "616598347865"
    }


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


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Unique identifier for Transaction as assigned by exchange |
 
Response example:

    {
      "id": "d2ce578f-647d-4fa0-b1aa-4a27e5ee597b"
    }
    
 
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

    {
      "id": "d2ce578f-647d-4fa0-b1aa-4a27e5ee597b"
    }
    

#### Get transactions history
 
`GET /api/2/account/transactions`
 
`GET /api/2/account/transactions/{id}` - get transaction by transaction id
  
Parameters: 
    
| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency |
| sort | String | `DESC` or `ASC`. Default value `DESC` |
| from | Datetime | |
| till | Datetime | |
| limit | Number | Default 100 |
| offset | Number | |


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Unique identifier for Transaction as assigned by exchange |
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
      
   
    [
      {
        "id": "6a2fb54d-7466-490c-b3a6-95d8c882f7f7",
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
