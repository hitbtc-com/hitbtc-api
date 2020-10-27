## GETTING STARTED

HitBTC REST & Streaming API provides programmatic access to HitBTC’s next generation trading engine.

By using HitBTC API you confirm that you've read and accept [API License Agreement](https://hitbtc.com/api-license-agreement)

## Table of Contents
 * [Development guide](#development-guide)
    * [API Explorer](#api-explorer)
    * [API URLs](#api-urls)    
    * [DateTime format](#datetime-format)        
    * [Number format](#number-format)
    * [Pagination](#pagination)   
* [Rate limiting](#rate-limiting)
* [Best practices](#best-practices)
* [REST API Reference](#rest-api-reference)
    * [Market Data](#market-data)
        * [Currencies](#currencies)
        * [Symbols](#symbols)
        * [Tickers](#tickers)
        * [Trades](#trades)
        * [Orderbook](#orderbook)
        * [Candles](#candles)
    * [Authentication](#authentication)
    * [Trading](#trading)
        * [Get trading balance](#trading-balance)
        * [Order model](#order-model)        
        * [Get active orders](#get-active-orders)   
        * [Get active order](#get-active-order-by-clientorderid)   
        * [Create new order](#create-new-order)   
        * [Cancel all orders](#cancel-orders)
        * [Cancel order](#cancel-order-by-clientorderid)   
        * [Trading commission](#get-trading-commission)
    * [Marging Trading](#trading)
        * [Isolated margin account model](#isolated-margin-account-model)
        * [Position model](#position-model)        
        * [Get isolated margin accounts](#get-isolated-margin-accounts)   
        * [Retrieve all margin](#retrieve-all-margin)   
        * [Get isolated margin account](#get-isolated-margin-account)
        * [Create or update isolated margin account](#create/update-isolatedmargin-account) 
        * [Close isolated margin account](#close-isolated-margin-account)    
        * [Get margin positions](#get-margin-positions) 
        * [Close margin positions](#close-margin-positions)      
        * [Get margin position](#get-margin-position) 
        * [Close margin position](#close-margin-position)   
        * [Get active margin orders](#get-active-margin-orders)    
        * [Get active margin order](#get-active-margin-order) 
        * [Create or update active margin orders](#create-or-update-active-margin-orders)   
        * [Cancel margin orders](#cancel-margin-orders) 
        * [Cancel margin order](#cancel-margin-order)            
    * [Trading history](#trading-history)
        * [Orders history](#orders-history)
        * [Trades history](#trades-history)    
        * [Trades by order](#trades-by-order)           
    * [Account management](#account-management)
        * [Account balance](#account-balance)
        * [Deposit address](#deposit-crypto-address)
        * [Last 10 deposit crypto address](#last-10-deposit-crypto-address)
        * [Last 10 used crypto address](#last-10-used-crypto-address)        
        * [Withdraw crypto](#withdraw-crypto)
        * [Transfer convert between currencies](#transfer-convert-between-currencies)
        * [Commit or Rollback Withdraw crypto](#withdraw-crypto-commit-or-rollback)
        * [Estimate withdraw fee](#estimate-withdraw-fee)
        * [Check if crypto address belongs to current account](#check-if-crypto-address-belongs-to-current-account)
        * [Transfer between trading and account](#transfer-money-between-trading-and-bank-account)
        * [Transfer money to another user by email or username](#transfer-money-to-another-user-by-email-or-username)        
        * [Transactions history](#get-transactions-history)    
* [Sub-accounts](#sub-accounts)
    * [Get sub-accounts list](#get-sub-account-list)
    * [Freeze sub-account](#freeze-sub-account)
    * [Activate sub-account](#activate-sub-account)
    * [Transfer funds](#transfer-funds)
    * [Get withdrawal settings](#get-withdrawal-settings)
    * [Change withdrawal settings](#change-withdrawal-settings)
    * [Get sub-account balance](#get-sub-account-balance)
    * [Get sub-account crypro address](#get-sub-account-crypto address)
* [Socket API Reference](#socket-api-reference)
    * [Market Data](#socket-market-data)
    * [Authentication](#socket-authentication-session)
    * [Trading](#socket-trading)
    * [Margin trading](#socket-margin-trading)   
    * [Errors](#errors) 
    * [Guide & Code samples](#guide-&-code-samples) 
    
# DEVELOPMENT GUIDE

## API Explorer

You can explore the API using [SwaggerUI](https://hitbtc.com/api/2/explore/) including methods requiring authorization.

## API URLs

| REST                              | Streaming | Streaming Trading |
|:---------------------------------|:-----|:-----|
| https://hitbtc.com/api/2      | wss://hitbtc.com/api/2/ws | wss://hitbtc.com/api/2/ws/trading |


### Demo environment (sandbox):

| REST                              | Streaming | Streaming Trading |
|:---------------------------------|:-----|:-----|
| https://api.demo.hitbtc.com/api/2      | wss://api.demo.hitbtc.com/api/2/ws | wss://api.demo.hitbtc.com/api/2/ws/trading |


## DateTime Format

All timestamps are returned in ISO 8601 format (UTC). <br>Example: "2017-04-03T10:20:49.315Z"

## Number Format

All finance data, e.g. price, quantity, fee, etc., should be arbitrary precision numbers and have a string representation. <br>Example: "10.2000058"

## Pagination

Parameters:

| Parameter | Description |
|-----------|-------------|
| limit | Number of results per call <br>Accepted range: 0 - 1000 <br>Default value: 100|
| offset | Number of results offset <br>Default value: 0|
| sort | Sort direction <br>Accepted values: `ASC` (ascending order), `DESC` (descending order) <br>Default value: `DESC` |
| by | Defines filter type <br>Accepted values: `id`, `timestamp` |
| from | Interval initial value (optional parameter) <br>If filter by `timestamp` is used, then parameter type is `datetime`, otherwise `object id` |
| till | Interval end value (optional parameter) <br>If filter by `timestamp`  is used, then parameter type is `datetime`, otherwise `object id` |

# RATE LIMITING

The following Rate Limits are applied:

 * For the Market data, the limit is 100 requests per second for one IP;
 * For Trading, the limit is 300 requests per second for one user;
 * For other requests, including Trading history, the limit is 10 requests per second for one user.

Significantly exceeding the Rate Limits can lead to suspension.

# BEST PRACTICES

The HitBTC API development team strives to bring the best trading experience to API users. This manual contains a set of best practices for using the API as efficiently as possible.

## HTTP Persistent Connection
The underlying TCP connection is kept active for multiple requests/responses. Subsequent requests will result in reduced latency as the TCP handshaking process is no longer required.

If you use the HTTP 1.0 client, please ensure it supports the Keep-Alive directive and submit the ''Connection: Keep-Alive'' header with your request.

Keep-Alive is a part of the HTTP 1.1 protocol and enabled by default on compliant clients. However, you will have to ensure your implementation does not set other values as the connection header.

## Retrieving and updating account state

Use the Streaming API for real-time updates of your orders, trades and any transaction changes.

# REST API Reference

## HTTP Status codes

 * 200 OK. Successful request
 * 400 Bad Request. Returns JSON with the error message
 * 401 Unauthorised. Authorisation is required or has been failed
 * 403 Forbidden. Action is forbidden for API key
 * 429 Too Many Requests. Your connection has been rate limited
 * 500 Internal Server. Internal Server Error
 * 503 Service Unavailable. Service is down for maintenance
 * 504 Gateway Timeout. Request timeout expired


## Error response

All error responses have error `code` and human readable `message` fields. Some errors contain an additional `description` field.

**Example of error response:**

```json
{
  "error": {
    "code": 20001,
    "message": "Insufficient funds",
    "description": "Check that the funds are sufficient, given commissions"
  }
}
```

# Market data

## Currencies
### Get a list of all currencies or specified currencies

```shell
curl "https://api.hitbtc.com/api/2/public/currency"  
```

> The above command returns JSON structured like this:

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
      "transferEnabled": true,
      "delisted": false,
      "payoutFee": "0.00958",
      "payoutMinimalAmount": "0.00958",
      "precisionPayout": 10,
      "precisionTransfer": 10
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
      "transferEnabled": true,
      "delisted": false,
      "payoutFee": "0.001",
      "payoutMinimalAmount": "0.00958",
      "precisionPayout": 20,
      "precisionTransfer": 15
   }
]
```

`GET /api/2/public/currency`

Return the actual list of available currencies, tokens, etc.

**You can optionally use comma-separated list of currencies. If it is not provided, null or empty, the request returns all currencies.**

Requires no API key Access Rights.

Parameter:

| Name | Type | Description |
|:---|:---:|:---|
| currencies | String | Comma-separated list of currency codes. Optional parameter|

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Currency identifier (code), for example, ''BTC'' <br>Note: description will simply use `currency` in the future. |
| fullName | String | Currency full name, for example, ''Bitcoin'' |
| crypto | Boolean | Determines whether currency belongs to blockchain  |
| payinEnabled | Boolean | Determines whether it is allowed to generate addresses for a deposit |
| payinPaymentId | Boolean | Determines whether it is required to provide additional information other than the address for deposit |
| payinConfirmations | Number | Count of blocks confirmations, which are needed for deposit |
| payoutEnabled | Boolean | Determines whether withdraw is allowed |
| payoutIsPaymentId | Boolean | Determines whether providing of additional information for withdraw is allowed |
| transferEnabled | Boolean | Determines whether transfer between trading account and bank account is allowed (may be disabled on maintenance) |
| delisted | Boolean | `true` if currency is delisted (deposit and trading are stopped) |
| payoutFee | Number | Default withdraw fee |
| payoutMinimalAmount | String | Minimum withdraw amount |
| precisionPayout | Number | Currency precision for payout (number of digits after the decimal point) |
| precisionTransfer | Number | Currency precision for transfer (number of digits after the decimal point) |

### Get a certain currency

```shell
curl "https://api.hitbtc.com/api/2/public/currency/ETH"  
```

> The above command returns JSON structured like this:

```json
{
  "id": "ETH",
  "fullName": "Ethereum",
  "crypto": true,
  "payinEnabled": true,
  "payinPaymentId": false,
  "payinConfirmations": 20,
  "payoutEnabled": true,
  "payoutIsPaymentId": false,
  "transferEnabled": true,
  "delisted": false,
  "payoutFee": "0.042800000000",
  "payoutMinimalAmount": "0.00958",
  "precisionPayout": 10,
  "precisionTransfer": 10
}
```

`GET /api/2/public/currency/{currency}`

Returns the data for a certain currency.

Requires no API key Access Rights.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Currency identifier (code), for example, ''BTC'' <br>Note: description will simply use `currency` in the future. |
| fullName | String | Currency full name, for example, ''Bitcoin'' |
| crypto | Boolean | Determines whether currency belongs to blockchain  |
| payinEnabled | Boolean | Determines whether it is allowed to generate addresses for a deposit |
| payinPaymentId | Boolean | Determines whether it is required to provide additional information other than the address for deposit |
| payinConfirmations | Number | Count of blocks confirmations, which are needed for deposit |
| payoutEnabled | Boolean | Determines whether withdraw is allowed |
| payoutIsPaymentId | Boolean | Determines whether providing of additional information for withdraw is allowed |
| transferEnabled | Boolean | Determines whether transfer between trading account and bank account is allowed (may be disabled on maintenance) |
| delisted | Boolean | `true` if currency is delisted (deposit and trading are stopped) |
| payoutFee | Number | Default withdraw fee |
| payoutMinimalAmount | String | Minimum withdraw amount |
| precisionPayout | Number | Currency precision for payout (number of digits after the decimal point) |
| precisionTransfer | Number | Currency precision for transfer (number of digits after the decimal point) |



## Symbols
### Get a list of all symbols or specified symbols

```shell
curl "https://api.hitbtc.com/api/2/public/symbol"  
```

> The above command returns JSON structured like this:

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

`GET /api/2/public/symbol`

Return the actual list of currency symbols (currency pairs) traded on exchange.
The first listed currency of a symbol is called the **base currency**, and the second currency is called the **quote currency**.
The currency pair indicates how much of the quote currency is needed to purchase one unit of the base currency.
[Read more](http://www.investopedia.com/terms/c/currencypair.asp)

**You can optionally use comma-separated list of symbols. If it is not provided, null or empty, the request returns all symbols.**

Requires no API key Access Rights.

Parameter:

| Name | Type | Description |
|:---|:---:|:---|
| symbols | String | Comma-separated list of symbol codes. Optional parameter|

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Symbol (currency pair) identifier, for example, ''ETHBTC'' <br>Note: description will simply use `symbol` in the future. |
| baseCurrency | String | Name (code) of base currency, for example, ''ETH''|
| quoteCurrency | String | Name of quote currency |
| quantityIncrement | Number | Symbol quantity should be divided by this value with no remainder |
| tickSize | Number | Symbol price should be divided by this value with no remainder |
| takeLiquidityRate | Number | Default fee rate |
| provideLiquidityRate | Number | Default fee rate for market making trades |
| feeCurrency | String | Value of charged fee  |

### Get a certain symbol

```shell
curl "https://api.hitbtc.com/api/2/public/symbol/ETHBTC"  
```
> The above command returns JSON structured like this:

```json    
{
  "id": "ETHBTC",
  "baseCurrency": "ETH",
  "quoteCurrency": "BTC",
  "quantityIncrement": "0.0001",
  "tickSize": "0.000001",
  "takeLiquidityRate": "0.002",
  "provideLiquidityRate": "0.001",
  "feeCurrency": "BTC"
}
```

`GET /api/2/public/symbol/{symbol}`

Returns the data for a certain symbol. 

Requires no API key Access Rights.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Symbol (currency pair) identifier, for example, ''ETHBTC'' <br>Note: description will simply use `symbol` in the future. |
| baseCurrency | String | Name (code) of base currency, for example, ''ETH''|
| quoteCurrency | String | Name of quote currency |
| quantityIncrement | Number | Symbol quantity should be divided by this value with no remainder |
| tickSize | Number | Symbol price should be divided by this value with no remainder |
| takeLiquidityRate | Number | Default fee rate |
| provideLiquidityRate | Number | Default fee rate for market making trades |
| feeCurrency | String | Value of charged fee  |


## Tickers
### Get tickers for all symbols or for specified symbols

```shell
curl "https://api.hitbtc.com/api/2/public/ticker"  
```

> The above command returns JSON structured like this:


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

`GET /api/2/public/ticker`

Returns ticker information.

**You can optionally use comma-separated list of symbols. If it is not provided, null or empty, the request returns tickers for all symbols.**

Requires no API key Access Rights.

Parameter:

| Name | Type | Description |
|:---|:---:|:---|
| symbols | String | Comma-separated list of symbol codes. Optional parameter|

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| ask | Number or null | Best ask price. Can return 'null' if no data |
| bid | Number or null | Best bid price. Can return 'null' if no data  |
| last | Number or null | Last trade price. Can return 'null' if no data  |
| open | Number or null | Last trade price 24 hours ago. Can return 'null' if no data  |
| low | Number | Lowest trade price within 24 hours |
| high | Number | Highest trade price within 24 hours |
| volume | Number | Total trading amount within 24 hours in base currency |
| volumeQuote | Number | Total trading amount within 24 hours in quote currency |
| timestamp | Datetime | Last update or refresh ticker timestamp |
| symbol | String | Symbol name |

### Get ticker for a certain symbol

```shell
curl "https://api.hitbtc.com/api/2/public/ticker/ETHBTC"  
```

> The above command returns JSON structured like this:

```json
{
  "symbol": "ETHBTC",
  "ask": "0.020572",
  "bid": "0.020566",
  "last": "0.020574",
  "open": "0.020913",
  "low": "0.020388",
  "high": "0.021084",
  "volume": "138444.3666",
  "volumeQuote": "2853.6874972480",
  "timestamp": "2020-04-02T17:52:36.731Z"
}
```

`GET /api/2/public/ticker/{symbol}`

Returns the ticker for a certain symbol. 

Requires no API key Access Rights.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| ask | Number or null | Best ask price. Can return 'null' if no data |
| bid | Number or null | Best bid price. Can return 'null' if no data  |
| last | Number or null | Last trade price. Can return 'null' if no data  |
| open | Number or null | Last trade price 24 hours ago. Can return 'null' if no data  |
| low | Number | Lowest trade price within 24 hours |
| high | Number | Highest trade price within 24 hours |
| volume | Number | Total trading amount within 24 hours in base currency |
| volumeQuote | Number | Total trading amount within 24 hours in quote currency |
| timestamp | Datetime | Last update or refresh ticker timestamp |
| symbol | String | Symbol name |


## Trades
### Get trades for all symbols or for specified symbols
```shell
curl "https://api.hitbtc.com/api/2/public/trades"  
```

> The above command returns JSON structured like this:

```json
{
   "BTCUSD":[
      {
         "id":782135488,
         "price":"9793.94",
         "quantity":"0.21469",
         "side":"sell",
         "timestamp":"2020-02-24T12:54:41.972Z"
      }
   ],
   "ETHBTC":[
      {
         "id":782135414,
         "price":"0.027668",
         "quantity":"0.069",
         "side":"buy",
         "timestamp":"2020-02-24T12:54:32.843Z"
      }
   ]
}
```

`GET /api/2/public/trades`

Returns trades information for a symbol with a symbol Id.

**You can optionally use comma-separated list of symbols. If it is not provided, null or empty, the request returns trades for all symbols.**

Requires no API key Access Rights.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| sort | String | Sort direction<br>Accepted values: `ASC`, `DESC` <br>Default value: `DESC`  |
| from | Datetime or Number | Interval initial value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value |
| till | Datetime or Number | Interval end value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value |
| limit | Number | Default value: 100<br>Max value: 1000 |
| offset | Number | Default value: 0<br>Max value: 100000 |
| symbols | String | Comma-separated list of symbol codes. Optional parameter|

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | Number | Trade identifier |
| price | Number | Trade price |
| quantity | Number | Trade quantity |
| side | String | Trade side<br>Accepted values: `sell` or `buy` |
| timestamp | Datetime | Trade timestamp |

### Get trades for a certain symbol

```shell
curl "https://api.hitbtc.com/api/2/public/trades/ETHBTC?sort=DESC"  
```

> The above command returns JSON structured like this:

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

`GET /api/2/public/trades/{symbol}`

Returns trades information for a symbol with a symbol Id.

Requires no API key Access Rights.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| sort | String | Sort direction<br>Accepted values: `ASC`, `DESC` <br>Default value: `DESC`  |
| by | String | Defines filter type <br>Accepted values: `id`, `timestamp` <br>Default value: `timestamp` |
| from | Datetime or Number | Interval initial value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value |
| till | Datetime or Number | Interval end value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value |
| limit | Number | Default value: 100<br>Max value: 1000 |
| offset | Number | Default value: 0<br>Max value: 100000 |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | Number | Trade identifier |
| price | Number | Trade price |
| quantity | Number | Trade quantity |
| side | String | Trade side<br>Accepted values: `sell` or `buy` |
| timestamp | Datetime | Trade timestamp |


## Order Book

### Get Order Book for all symbols or for specified symbols

```shell
curl "https://api.hitbtc.com/api/2/public/orderbook"  
```

> The above command returns JSON structured like this:

```json
{
  "BTCUSD": {
    "symbol": "BTCUSD",
    "ask": [
      {
        "price": "9777.51",
        "size": "4.50579"
      },
      {
        "price": "9777.52",
        "size": "5.79832"
      }
    ],
    "bid": [
      {
        "price": "9777.5",
        "size": "0.00002"
      },
      {
        "price": "9776.26",
        "size": "0.0001"
      }
    ],
    "timestamp": "2020-02-11T11:18:03.857366871Z"
  },
  "ETHBTC": {
    "symbol": "ETHBTC",
    "ask": [
      {
        "price": "0.022626",
        "size": "0.0057"
      },
      {
        "price": "0.022628",
        "size": "1.4259"
      }
    ],
    "bid": [
      {
        "price": "0.022624",
        "size": "0.5748"
      },
      {
        "price": "0.022623",
        "size": "26.5"
      }
    ],
    "timestamp": "2020-02-11T11:18:03.790858502Z"
  }
}
```

`GET /api/2/public/orderbook`

An Order Book is an electronic list of buy and sell orders for a specific symbol, structured by price level.

**You can optionally use comma-separated list of symbols. If it is not provided, null or empty, the request returns an Order Book for all symbols.**

Requires no API key Access Rights.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| limit | Number | Limit of Order Book levels<br>Default value: 100 <br>Set 0 to view full list of Order Book levels. |
| symbols | String | Comma-separated list of symbol codes. Optional parameter|

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| ask | Array | Ask side array of levels |
| bid | Array | Bid side array of levels |
| size | Number | Total volume of orders with the specified price |
| price | Number | Price level |


### Get Order Book for a certain symbol
```shell
curl "https://api.hitbtc.com/api/2/public/orderbook/ETHBTC?volume=0.5"  
```

> The above command returns JSON structured like this:

```json
{
  "ask": [
    {
      "price": "9779.68",
      "size": "2.497"
    }
  ],
  "bid": [
    {
      "price": "9779.67",
      "size": "0.03719"
    },
    {
      "price": "9779.29",
      "size": "0.171"
    },
    {
      "price": "9779.27",
      "size": "0.171"
    },
    {
      "price": "9779.21",
      "size": "0.171"
    }
  ],
  "timestamp": "2020-02-11T11:30:38.597950917Z",
  "askAveragePrice": "9779.68",
  "bidAveragePrice": "9779.29"
}
```


`GET /api/2/public/orderbook/{symbol}`

The request returns an Order Book for a certain symbol.

Requires no API key Access Rights.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| limit | Number | Limit of Order Book levels<br>Default value: 100 <br>Set 0 to view full list of Order Book levels. |
| volume | Number | Desired volume for market depth search |

Please note that if the `Volume` is specified, the `Limit` will be ignored,
`askAveragePrice` and `bidAveragePrice` are returned in response.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| ask | Array | Ask side array of levels |
| bid | Array | Bid side array of levels |
| size | Number | Total volume of orders with the specified price |
| price | Number | Price level |
| askAveragePrice | Number | Average aggregated price for the ask side orders |
| bidAveragePrice | Number | Average aggregated price for the bid side orders |


## Candles
### Get candles for all symbols or for specified symbols

```shell
curl "https://api.hitbtc.com/api/2/public/candles"  
```

> The above command returns JSON structured like this:

```json
{
   "BTCUSD":[
      {
         "timestamp":"2013-12-27T08:23:00.000Z",
         "open":"777",
         "close":"777",
         "min":"777",
         "max":"777",
         "volume":"0.1",
         "volumeQuote":"77.7"
      }
   ],
   "ETHBTC":[
      {
         "timestamp":"2015-08-20T19:01:00.000Z",
         "open":"0.006",
         "close":"0.006",
         "min":"0.006",
         "max":"0.006",
         "volume":"0.003",
         "volumeQuote":"0.000018"
      }
   ]
}
```

`GET /api/2/public/candles`

Candles are used for the representation of a specific symbol as an [OHLC](https://www.investopedia.com/terms/o/ohlcchart.asp) chart.

**You can optionally use comma-separated list of symbols. If it is not provided, null or empty, the request returns candles for all symbols.**

Requires no API key Access Rights.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| period | String | Accepted values: `M1` (one minute), `M3`, `M5`, `M15`, `M30`, `H1` (one hour), `H4`, `D1` (one day), `D7`, `1M` (one month) <br>Default value: `M30` (30 minutes) |
| sort | String | Sort direction <br>Accepted values: `ASC`, `DESC` <br>Default value: `ASC` |
| from | Datetime | Interval initial value (optional parameter) |
| till | Datetime | Interval end value (optional parameter) |
| limit | Number | Limit of candles<br>Default value: 100<br>Max value: 1000 |
| offset | Number | Default value: 0<br>Max value: 100000 |
| symbols | String | Comma-separated list of symbol codes. Optional parameter|


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| timestamp | Datetime | Candle timestamp |
| open | Number | Open price |
| close | Number | Closing price |
| min | Number | Lowest price for the period |
| max | Number | Highest price for the period |
| volume | Number | Volume in base currency |
| volumeQuote | Number | Volume in quote currency |

<aside class="notice">The result contains candles with non-zero volume only (no trades = no candles).</aside>

### Get candles for a certain symbol

```shell
curl "https://api.hitbtc.com/api/2/public/candles/ETHBTC"  
```

> The above command returns JSON structured like this:

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

`GET /api/2/public/candles/{symbol}`

The request returns candles for a certain symbol.

Requires no API key Access Rights.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| period | String | Accepted values: `M1` (one minute), `M3`, `M5`, `M15`, `M30`, `H1` (one hour), `H4`, `D1` (one day), `D7`, `1M` (one month) <br>Default value: `M30` (30 minutes) |
| sort | String | Sort direction <br>Accepted values: `ASC`, `DESC` <br>Default value: `ASC` |
| from | Datetime | Interval initial value (optional parameter) |
| till | Datetime | Interval end value (optional parameter) |
| limit | Number | Limit of candles<br>Default value: 100<br>Max value: 1000 |
| offset | Number | Default value: 0<br>Max value: 100000 |


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| timestamp | Datetime | Candle timestamp |
| open | Number | Open price |
| close | Number | Closing price |
| min | Number | Lowest price for the period |
| max | Number | Highest price for the period |
| volume | Number | Volume in base currency |
| volumeQuote | Number | Volume in quote currency |

<aside class="notice">The result contains candles with non-zero volume only (no trades = no candles).</aside>


# Authentication

```shell
curl -u "publicKey:secretKey" https://api.hitbtc.com/api/2/trading/balance
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")

```

```javascript
const fetch = require('node-fetch');

const credentials = Buffer.from('publicKey' + ':' + 'secretKey').toString('base64');

fetch('https://api.hitbtc.com/api/2/trading/balance', {
    method: 'GET',
    headers: {
        'Authorization': 'Basic ' + credentials
    }
});

```

Public market data are available without authentication. Authentication is required for other requests.

You should create API keys on the [API Setting](https://hitbtc.com/settings/api-keys) page. You can create multiple API keys with different access rights for your applications.

Use Basic Authentication to access the REST API.

# Trading

## Trading Balance

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/trading/balance"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/trading/balance').json()
print(b)
```

> The above command returns JSON structured like this:

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

`GET /api/2/trading/balance`

Returns the user's trading balance.

Requires the "Orderbook, History, Trading balance" API key Access Right.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency code |
| available | Number | Amount available for trading or transfer to main account |
| reserved | Number | Amount reserved for active orders or incomplete transfers to main account |



## Order model

Order model consists of:

| Name | Type | Description |
|:---|:---:|:---|    
| id | Number | Order unique identifier as assigned by exchange |
| clientOrderId | String | Order unique identifier as assigned by trader. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| symbol | String| Trading symbol name |
| side | String | Trade side <br>Accepted values: `sell` or `buy`|
| status | String | Order state <br>Accepted values: `new`, `suspended`, `partiallyFilled`, `filled`, `canceled`, `expired` |
| type | String | Accepted values: `limit`, `market`, `stopLimit`, `stopMarket` |
| timeInForce | String | Time in Force is a special instruction used when placing a trade to indicate how long an order will remain active before it is executed or expired. <br> **GTC** - ''Good-Till-Cancelled'' order won't be closed until it is filled. <br> **IOC** - ''Immediate-Or-Cancel'' order must be executed immediately. Any part of an IOC order that cannot be filled immediately will be cancelled.<br> **FOK** - ''Fill-Or-Kill'' is a type of ''Time in Force'' designation used in securities trading that instructs a brokerage to execute a transaction immediately and completely or not execute it at all. <br> **Day** - keeps the order active until the end of the trading day (UTC). <br> **GTD** - ''Good-Till-Date''. The date is specified in `expireTime`. |
| quantity | Number | Order quantity |
| price | Number | Order price |
| avgPrice | Number |  Average execution price, only for history orders |
| cumQuantity | Number | Cumulative executed quantity |
| createdAt | Datetime | Order creation date |
| updatedAt | Datetime | Date of order's last update |
| stopPrice | Number | Required for stop-limit and stop-market orders |
| postOnly | Boolean| A post-only order is an order that does not remove liquidity. If your post-only order causes a match with a pre-existing order as a taker, then the order will be cancelled. |
| expireTime | Datetime | Date of order expiration for `timeInForce` = `GTD` |     
| tradesReport | Array of trades | Optional list of trades |

```json
{
  "id": 828680665,
  "clientOrderId": "f4307c6e507e49019907c917b6d7a084",
  "symbol": "ETHBTC",
  "side": "sell",
  "status": "partiallyFilled",
  "type": "limit",
  "timeInForce": "GTC",
  "quantity": "13.942",
  "price": "0.011384",
  "cumQuantity": "5.240",
  "createdAt": "2017-01-16T14:18:47.321Z",
  "updatedAt": "2017-01-19T15:23:54.876Z",
  "postOnly": false
}
```

## Get active orders

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/order"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/order').json()
print(b)
```

> The above command returns JSON structured like this:

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
    "postOnly": false,
    "createdAt": "2017-05-12T17:17:57.437Z",
    "updatedAt": "2017-05-12T17:18:08.610Z"
  }
]
```

`GET /api/2/order`

Return array of active orders.

Requires the "Place/cancel orders" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter active orders by symbol |

Response: Array of orders



## Get active order by clientOrderId

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/order/c1837634ef81472a9cd13c81e7b91401"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/order/c1837634ef81472a9cd13c81e7b91401').json()
print(b)
```

> The above command returns JSON structured like this:

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
    "postOnly": false,
    "createdAt": "2017-05-12T17:17:57.437Z",
    "updatedAt": "2017-05-12T17:18:08.610Z"
}
```

`GET /api/2/order/{clientOrderId}`

Requires the "Place/cancel orders" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| wait | Number | Time in milliseconds (optional parameter) <br>Max value: 60000 <br>Default value: none <br>While using long polling request: if order is filled, cancelled or expired order info will be returned instantly. For other order statuses, actual order info will be returned after specified wait time. |

Response: order


## Create new order

```shell
curl -X PUT -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/order/d8574207d9e3b16a4a5511753eeef175" \
    -d 'symbol=ETHBTC&side=sell&quantity=0.063&price=0.046016'
```

```python
    import requests
    session = requests.session()
    session.auth = ("publicKey", "secretKey")
    orderData = {'symbol':'ethbtc', 'side': 'sell', 'quantity': '0.063', 'price': '0.046016' }
    r = session.post('https://api.hitbtc.com/api/2/order', data = orderData)        
    print(r.json())

```

> The above command returns JSON structured like this:

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
        "postOnly": false,
        "createdAt": "2017-05-15T17:01:05.092Z",
        "updatedAt": "2017-05-15T17:01:05.092Z"
    }
```    

> Error response example:

```json
{
  "error": {
    "code": 20001,
    "message": "Insufficient funds",
    "description": "Check that the funds are sufficient, given commissions"
  }
}
```    

`POST /api/2/order`        
`PUT /api/2/order/{clientOrderId}`        

Requires the "Place/cancel orders" API key Access Right.

**Price accuracy and quantity**

Symbol config contains the `tickSize` parameter which means that `price` should be divided by `tickSize` with no remainder. <br>`quantity` should be divided by `quantityIncrement` with no remainder. <br>By default, if `strictValidate` is not enabled, the Server rounds half down the `price` and `quantity` for `tickSize` and `quantityIncrement`.

Example of ETHBTC: `tickSize` = '0.000001', then price '0.046016' is valid, '0.0460165' is invalid.    

**Fees**

Charged fee is determined in symbol's `feeCurrency`. Maker-taker fees offer a transaction rebate `provideLiquidityRate` to those who provide liquidity (the market maker), while charging customers who take that liquidity `takeLiquidityRate`.

To create buy orders, you must have sufficient balance including fees.<br>`Available balance > price * quantity * (1 + takeLiquidityRate)`                                    

**Order result status**

For orders with `timeInForce` = `IOC` or `FOK`, the REST API returns final order state: `filled` or `expired`.

If order can be instantly executed, then the REST API returns a status of `filled` or `partiallyFilled` in the order's info.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Optional parameter.<br>If it is skipped, it will be generated by the Server. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| symbol| String| Trading symbol |
| side | String | Trade side <br>Accepted values: `sell` or `buy` |
| type | String | Optional parameter <br>Accepted values: `limit`, `market`, `stopLimit`, `stopMarket`<br> Default value: `limit`  |
| timeInForce | String | Optional parameter<br>Accepted values: `GTC`, `IOC`, `FOK`, `Day`, `GTD` <br>Default value: `GTC`  |
| quantity | Number | Order quantity |
| price | Number | Order price. Required for limit order types |
| stopPrice | Number | Required for stop-limit and stop-market orders |
| expireTime | Datetime | Required for orders with `timeInForce` = `GTD` |    
| strictValidate | Boolean | Price and quantity will be checked for incrementation within the symbol’s tick size and quantity step.. See the symbol's `tickSize` and `quantityIncrement`. |
| postOnly | Boolean|  If your post-only order causes a match with a pre-existing order as a taker, then the order will be cancelled. |

Response: order

## Cancel orders

`DELETE /api/2/order`

Cancel all active orders, or all active orders for a specified symbol.

Requires the "Place/cancel orders" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter active orders by symbol |


Response: Array of orders

## Cancel order by clientOrderId

```shell
curl -X DELETE -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/order/d8574207d9e3b16a4a5511753eeef175"    
```

> The above command returns JSON structured like this:

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
  "postOnly": false,
  "createdAt": "2017-05-15T17:01:05.092Z",
  "updatedAt": "2017-05-15T18:08:57.226Z"
}
```

> Example of **Order not found** error response:

```json
{
  "error": {
    "code": 20002,
    "message": "Order not found",
    "description": ""
  }
}
```

`DELETE /api/2/order/{clientOrderId}`

Requires the "Place/cancel orders" API key Access Right.

Response: order


## Get trading commission

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/trading/fee/ETHBTC"    
```

> The above command returns JSON structured like this:

```json
{
  "takeLiquidityRate": "0.001",
  "provideLiquidityRate": "-0.0001"
}
```

`GET /api/2/trading/fee/{symbol}`

Get personal trading commission rate.

Requires the "Place/cancel orders" API key Access Right.

<% if to_show_margin_trading -%>
# Margin Trading

Margin Account should be understood as Isolated Margin Account regarding this API description.
<% if read_env() == 'HIT' -%>
Learn how margin trading works at our support center [https://support.hitbtc.com/en/support/solutions/articles/63000225029-what-is-margin-trading](https://support.hitbtc.com/en/support/solutions/articles/63000225029-what-is-margin-trading)
<% end -%>

## Isolated Margin Account Model

>Isolated Margin Account without position:

```json
{
    "symbol": "BTCUSD",
    "leverage": "5.00",
    "marginBalance": "50.0000000000000",
    "marginBalanceOrders": 0,
    "marginBalanceRequired": 0,
    "createdAt": "2020-06-21T14:29:48.882Z",
    "updatedAt": "2020-06-21T14:29:48.882Z"
}
```

>Isolated Margin Account with position:

```json
{
    "symbol": "BTCUSD",
    "leverage": "5.00",
    "marginBalance": "999.9841334080000",
    "marginBalanceOrders": "0",
    "marginBalanceRequired": "1.6056991104000",
    "createdAt": "2020-06-21T14:33:34.723Z",
    "updatedAt": "2020-06-21T14:33:46.149Z",
    "position": {
        "id": 298724,
        "symbol": "BTCUSD",
        "quantity": "-0.00100",
        "pnl": "0",
        "priceEntry": "7933.30",
        "priceMarginCall": "887772.25",
        "priceLiquidation": "914625.61",
        "createdAt": "2020-06-21T14:33:34.723Z",
        "updatedAt": "2020-06-21T14:33:46.149Z"
    }
}
```

Isolated Margin Account model consists of:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String| Trading symbol. Where base currency is the currency of funds reserved on the trading account for positions and quote currency is the currency of funds reserved on a Isolated Margin Account (e.g. "BTCUSD").|
| leverage | Number | Margin leverage. The ratio of the trader's own funds to funds borrowed from the platform. |
| marginBalance | Number | Amount of currency, reserved for margin purpose. |
| marginBalanceOrders | String | Amount of currency, reserved for margin orders. |
| marginBalanceRequired | String | Amount of currency, reserved for margin position close. |
| createdAt | Datetime | Account creation date and time. |
| updatedAt | Datetime | Account last update date and time. |
| position | Position | Optional parameter.<br> Open positions of the Isolated Margin Account. |


## Position Model

```json
{
    "id": 298724,
    "symbol": "BTCUSD",
    "quantity": "-0.00100",
    "pnl": "0",
    "priceEntry": "7933.30",
    "priceMarginCall": "887772.25",
    "priceLiquidation": "914625.61",
    "createdAt": "2020-06-21T14:33:34.723Z",
    "updatedAt": "2020-06-21T14:33:46.149Z"
}
```

Position model consists of:

| Name | Type | Description |
|:---|:---:|:---|
| id | Number| Position identifier. |
| symbol | String| Trading symbol. |
| quantity | Number | Position quantity. |
| pnl | Number | Unrealized profit and loss valued in currency. |
| priceEntry | Number | The price of the first transaction that opened a position. |
| priceMarginCall | Number | The market price of the margin call. |
| priceLiquidation | Number | The market price of force close. |
| createdAt | Datetime | Position creation date and time. |
| updatedAt | Datetime | Position last update date and time. |


## Get Isolated Margin Accounts

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/account"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/margin/account').json()
print(b)
```

`GET /api/2/margin/account`

Returns user Isolated Margin Account details.

Requires the "Place/cancel orders" API key Access Right.


## Retrieve All Margin

```shell
curl -X DELETE -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/account"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.delete('https://api.hitbtc.com/api/2/margin/account').json()
print(b)
```

`DELETE /api/2/margin/account`

Closes all positions and then closes all Isolated Margin Accounts. This will completely free up any balance reserved for margin purposes.

Returns list of the closed Isolated Margin Accounts.

Requires the "Place/cancel orders" API key Access Right.


## Get Isolated Margin Account

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/account/BTCUSD"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/margin/account/BTCUSD').json()
print(b)
```

`GET /api/2/margin/account/{symbol}`

Returns Isolated Margin Account details by symbol.

Requires the "Place/cancel orders" API key Access Right.


## Create/Update Isolated Margin Account

```shell
curl -X PUT -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/account/BTCUSD" \
     -d "marginBalance=123.44"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.put('https://api.hitbtc.com/api/2/margin/account/BTCUSD',
                json={"marginBalance":"123.4455", "strictValidate": True}).json()
print(b)
```

```json
{
    "symbol": "BTCUSD",
    "leverage": "5.00",
    "marginBalance": "50.0000000000000",
    "marginBalanceOrders": "0",
    "marginBalanceRequired": "0",
    "createdAt": "2020-06-21T14:29:48.882Z",
    "updatedAt": "2020-06-21T14:29:48.882Z"
}
```

`PUT /api/2/margin/account/{symbol}`

Creates or updates margin account. Setting margin balance to zero will lead to closing margin account and retrieval all formerly reserved funds to the trading account. 

Returns margin account details.

Requires the "Place/cancel orders" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol| String| Trading symbol. Where base currency is the currency of funds reserved on the trading account for positions and quote currency is the currency of funds reserved on a margin account (e.g. "BTCUSD"). |
| marginBalance | Number | Amount of currency reserved. |
| strictValidate | Boolean | The value indicating whether the `marginBalance` going to be checked for correct non exponential format and currency precision. |


## Close Isolated Margin Account

```shell
curl -X DELETE -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/account/BTCUSD"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.delete('https://api.hitbtc.com/api/2/margin/account/BTCUSD').json()
print(b)
```

`DELETE /api/2/margin/account/{symbol}`

Closes Isolated Margin Account by symbol. This will completely free up any balance reserved for margin purposes, and all open positions will be closed at market price.


Returns closed Isolated Margin Account details.

Requires the "Place/cancel orders" API key Access Right.


## Get Margin Positions

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/position"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/margin/position').json()
print(b)
```

```json
[
    {
        "id": 298724,
        "symbol": "BTCUSD",
        "quantity": "-0.00100",
        "pnl": "0",
        "priceEntry": "7933.30",
        "priceMarginCall": "887772.25",
        "priceLiquidation": "914625.61",
        "createdAt": "2020-06-21T14:33:34.723Z",
        "updatedAt": "2020-06-21T14:33:46.149Z"
    }
]
```

`GET /api/2/margin/position`

Returns a list of open positions.

Requires the "Place/cancel orders" API key Access Right.


## Close Margin Positions

```shell
curl -X DELETE -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/position"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.delete('https://api.hitbtc.com/api/2/margin/position').json()
print(b)
```

`DELETE /api/2/margin/position`

Closes all open positions.

Returns a list of the successfully closed margin positions.

Requires the "Place/cancel orders" API key Access Right.


## Get Margin Position

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/position/BTCUSD"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/margin/position/BTCUSD').json()
print(b)
```

`GET /api/2/margin/position/{symbol}`

Returns opened position for the requested symbol.

Requires the "Place/cancel orders" API key Access Right.


## Close Margin Position

```shell
curl -X DELETE -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/position/BTCUSD"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.delete('https://api.hitbtc.com/api/2/margin/position/BTCUSD',
                   json={"price": "9800.50", "strictValidate": True}).json()
print(b)
```

`DELETE /api/2/margin/position/{symbol}`

Closes open position by symbol.

Returns a list of the successfully closed margin positions.

Requires the "Place/cancel orders" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol| String| Trading symbol. |
| price | Number | Optional parameter.<br>If a price is defined, then close order would be an limit order with the specified price, instead, close order would be a market order with the market price. |
| strictValidate | Boolean | Optional parameter.<br>The value indicating whether the price going to be checked for incrementation within the symbol’s tick size step. See the symbol's `tickSize`.<br>Default value: false |


## Get Active Margin Orders

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/order"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/margin/order').json()
print(b)
```

>The above command returns JSON structured like this:

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
    "postOnly": false,
    "createdAt": "2017-05-12T17:17:57.437Z",
    "updatedAt": "2017-05-12T17:18:08.610Z"
  }
]
```

`GET /api/2/margin/order`

Returns an array of the active margin orders.

Requires the "Place/cancel orders" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter.<br>Parameter to filter active orders by symbol. |


## Get Active Margin Order

```shell
curl -X GET -u "publicKey:secretKey" \
     "https://api.hitbtc.com/api/2/margin/order/c1837634ef81472a9cd13c81e7b91401"
```

```python
import requests
session = requests.session()
session.auth = ("publicKey", "secretKey")
b = session.get('https://api.hitbtc.com/api/2/margin/order/c1837634ef81472a9cd13c81e7b91401').json()
print(b)
```

> The above command returns JSON structured like this:

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
    "postOnly": false,
    "createdAt": "2017-05-12T17:17:57.437Z",
    "updatedAt": "2017-05-12T17:18:08.610Z"
}
```

`GET /api/2/margin/order/{clientOrderId}`

Returns an active order by `clientOrderId`.

Requires the "Place/cancel orders" API key Access Right.


## Create/Update Margin Order

```shell
curl -X PUT -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/margin/order/d8574207d9e3b16a4a5511753eeef175" \
    -d 'symbol=ETHBTC&side=sell&quantity=0.063&price=0.046016'
```

```python
    import requests
    session = requests.session()
    session.auth = ("publicKey", "secretKey")
    orderData = {'symbol':'ethbtc', 'side': 'sell', 'quantity': '0.063', 'price': '0.046016' }
    r = session.post('https://api.hitbtc.com/api/2/margin/order', data = orderData)
    print(r.json())

```

> The above command returns JSON structured like this:

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
        "postOnly": false,
        "createdAt": "2017-05-15T17:01:05.092Z",
        "updatedAt": "2017-05-15T17:01:05.092Z"
    }
```

> Error response example:

```json
{
  "error": {
    "code": 20001,
    "message": "Insufficient funds",
    "description": "Check that the funds are sufficient, given commissions"
  }
}
```

`POST /api/2/margin/order`

`PUT /api/2/margin/order/{clientOrderId}`

Requires the Isolated Margin Account for the order symbol.

Requires the "Place/cancel orders" API key Access Right.

**Price accuracy and quantity**

Symbol config contains the `tickSize` parameter which means that `price` should be divided by `tickSize` with no remainder. <br>`quantity` should be divided by `quantityIncrement` with no remainder. <br>By default, if `strictValidate` is not enabled, the Server rounds half down the `price` and `quantity` for `tickSize` and `quantityIncrement`.

Example of ETHBTC: `tickSize` = '0.000001', then price '0.046016' is valid, '0.0460165' is invalid.

**Fees**

Charged fee is determined in symbol's `feeCurrency`. Maker-taker fees offer a transaction rebate `provideLiquidityRate` to those who provide liquidity (the market maker), while charging customers who take that liquidity `takeLiquidityRate`.

To create buy orders, you must have sufficient balance including fees.<br>`Available balance > price * quantity * (1 + takeLiquidityRate)`

**Order result status**

For orders with `timeInForce` = `IOC` or `FOK`, the REST API returns final order state: `filled` or `expired`.

If order can be instantly executed, then the REST API returns a status of `filled` or `partiallyFilled` in the order's info.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Optional parameter.<br>If it is skipped, it will be generated by the Server. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| symbol| String| Trading symbol. |
| side | String | Trade side.<br>Accepted values: `sell` or `buy` |
| type | String | Optional parameter <br>Accepted values: `limit`, `market`, `stopLimit`, `stopMarket`<br> Default value: `limit`  |
| timeInForce | String | Optional parameter.<br>Accepted values: `GTC`, `IOC`, `FOK`, `Day`, `GTD` <br>Default value: `GTC`  |
| quantity | Number | Order quantity. |
| price | Number | Order price. Required for limit order types. |
| stopPrice | Number | Required for stop-limit and stop-market orders. |
| expireTime | Datetime | Required for orders with `timeInForce` = `GTD`. |
| strictValidate | Boolean | Price and quantity will be checked for incrementation within the symbol’s tick size and quantity step.. See the symbol's `tickSize` and `quantityIncrement`. |
| postOnly | Boolean|  If your post-only order causes a match with a pre-existing order as a taker, then the order will be cancelled. |

Response: order


## Cancel Margin Orders

`DELETE /api/2/margin/order`

Cancels all active margin orders, or all active margin orders for the specified symbol.

Returns a list of cancelled margin orders.

Requires the "Place/cancel orders" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter.<br>Parameter to filter active margin orders by symbol. |


## Cancel Margin Order

```shell
curl -X DELETE -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/margin/order/d8574207d9e3b16a4a5511753eeef175"
```

>The above command returns JSON structured like this:

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
  "postOnly": false,
  "createdAt": "2017-05-15T17:01:05.092Z",
  "updatedAt": "2017-05-15T18:08:57.226Z"
}
```

> Example of **Order not found** error response:

```json
{
  "error": {
    "code": 20002,
    "message": "Order not found",
    "description": ""
  }
}
```

`DELETE /api/2/margin/order/{clientOrderId}`

Returns the successfully cancelled margin order.

Requires the "Place/cancel orders" API key Access Right.

<% end -%>


# Trading history

<aside class="notice">
Please note that trading history may be updated with a delay up to 30 seconds, with a mean delay of around 1 second.
</aside>

## Orders history

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/history/order?symbol=ETHBTC"    
```

```json
[
  {
    "id": 828680665,
    "clientOrderId": "f4307c6e507e49019907c917b6d7a084",
    "symbol": "ETHBTC",
    "side": "sell",
    "status": "partiallyFilled",
    "type": "limit",
    "timeInForce": "GTC",
    "quantity": "13.942",
    "price": "0.011384",
    "avgPrice": "0.055487",
    "cumQuantity": "5.240",
    "createdAt": "2017-01-16T14:18:47.321Z",
    "updatedAt": "2017-01-19T15:23:54.876Z"
  },
  {
    "id": 828680667,
    "clientOrderId": "f4307c6e507e49019907c917b6d7a084",
    "symbol": "ETHBTC",
    "side": "sell",
    "status": "partiallyFilled",
    "type": "limit",
    "timeInForce": "GTC",
    "quantity": "13.942",
    "price": "0.011384",
    "avgPrice": "0.045000",
    "cumQuantity": "5.240",
    "createdAt": "2017-01-16T14:18:50.321Z",
    "updatedAt": "2017-01-19T15:23:56.876Z"
  }
]
```

`GET /api/2/history/order`

All orders older than 24 hours without trades are deleted.

Requires the "Orderbook, History, Trading balance" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter orders by symbol. |
| clientOrderId | String | If set, other parameters will be ignored,  including `limit` and `offset`. |
| from | Datetime | Interval initial value (optional parameter) |
| till | Datetime | Interval end value (optional parameter) |
| limit | Number | Default value: 100<br>Max value: 1000 |
| offset | Number | Default value: 0<br>Max value: 100000 |


Response: Array of orders

## Trades history

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/history/trades?symbol=ETHBTC"    
```

```json
[
  {
    "id": 9535486,
    "orderId": 816088377,
    "clientOrderId": "f8dbaab336d44d5ba3ff578098a68454",
    "symbol": "ETHBTC",
    "side": "sell",
    "quantity": "0.061",
    "price": "0.045487",
    "fee": "0.000002775",
    "timestamp": "2017-05-17T12:32:57.848Z"
  },
  {
    "id": 9535437,
    "orderId": 816088021,
    "clientOrderId": "27b9bfc068b44194b1f453c7af511ed6",
    "symbol": "ETHBTC",
    "side": "buy",
    "quantity": "0.038",
    "price": "0.046000",
    "fee": "-0.000000174",
    "timestamp": "2017-05-17T12:30:57.848Z"
  }
]
```

`GET /api/2/history/trades`

Get the user's trading history.

Requires the "Orderbook, History, Trading balance" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter active orders by symbol |
| sort | String | Sort direction <br>Accepted values: `DESC`, `ASC` <br>Default value: `DESC` |
| by | String | Defines filter type <br>Accepted values: `timestamp`, `id`. Default value: `id` |
| from | Datetime or Number | Interval initial value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value. |
| till | Datetime or Number | Interval end value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value. |
| limit | Number | Default value: 100<br>Max value: 1000 |
| offset | Number | Default value: 0<br>Max value: 100000 |
| margin | String | Filtering of margin orders <br>Accepted values: `include`, `only`, 'ignore' <br>Default value: `include` |


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | Number | Trade unique identifier as assigned by exchange |
| orderId | Number | Order unique identifier as assigned by exchange |
| clientOrderId | String | Order unique identifier as assigned by trader |
| symbol| String| Trading symbol |
| side | String | Trade side<br>Accepted values: `sell` or `buy` |
| quantity | Number | Trade quantity |
| price | Number | Trade price |
| fee | Number | Trade commission <br>Can be negative (''rebate'' - reward paid to a trader). See fee currency in the symbol config. |
| liquidation | Boolean | Optional parameter. Liquidation trade flag for margin trades |
| timestamp | Datetime | Trade timestamp |



## Trades by order


```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/history/order/816088021/trades"    
```

```json
[
  {
    "id": 828680665,
    "orderId": 816088021,
    "clientOrderId": "f4307c6e507e49019907c917b6d7a084",
    "symbol": "ETHBTC",
    "side": "sell",
    "status": "partiallyFilled",
    "type": "limit",
    "timeInForce": "GTC",
    "price": "0.011384",
    "quantity": "13.942",
    "postOnly": false,
    "cumQuantity": "5.240",
    "createdAt": "2017-01-16T14:18:47.321Z",
    "updatedAt": "2017-01-19T15:23:54.876Z"
  }
]

```

`GET /api/2/history/order/{orderId}/trades`

Returns the user's trading order with a specified orderId.

Requires the "Orderbook, History, Trading balance" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| orderId | Number | Order id |

> `orderId` is required in cause of `clientOrderId` can be non-unique during long period.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | Number | Trade unique identifier as assigned by exchange |
| orderId | Number | Order unique identifier as assigned by exchange |
| clientOrderId | String | Order unique identifier as assigned by trader |
| symbol| String| Trading symbol |
| side | String | Trade side<br>Accepted values: `sell` or `buy` |
| quantity | Number | Trade quantity |
| price | Number | Trade price |
| fee | Number | Trade commission<br>Can be negative (''rebate'' - reward paid to a trader). See fee currency in the symbol config. |
| liquidation | Boolean | Optional parameter. Liquidation trade flag for margin trades |
| timestamp | Datetime | Trade timestamp |

# Account management

## Account balance


```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/balance"    
```

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

`GET /api/2/account/balance`

Returns the user's account balance.

Requires the "Payment information" API key Access Right.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency code |
| available | Number | Amount available for withdraw or transfer to trading account |
| reserved | Number | Amount reserved for incomplete transactions |


## Deposit crypto address

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/crypto/address/NXT"    
```

```json
{
  "address": "NXT-G22U-BYF7-H8D9-3J27W",
  "publicKey": "f79779a3a0c7acc75a62afe8125de53106c6a19c1ebdf92a3598676e58773df0"
}
```

`GET /api/2/account/crypto/address/{currency}` - Get current address

Requires the "Payment information" API key Access Right.

`POST /api/2/account/crypto/address/{currency}` - Create new address

Requires the "Payment information" API key Access Right.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| address | String | Address for deposit |
| paymentId | String | Optional parameter <br>If this parameter is set, it is **required** for deposit. |
| publicKey | String | Optional parameter <br>If this parameter is set, it is **required** for deposit. |

## Last 10 deposit crypto address

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/crypto/addresses/NXT"    
```

```json
[
  {
    "address": "NXT-G22U-BYF7-H8D9-3J27W",
    "publicKey": "f79779a3a0c7acc75a62afe8125de53106c6a19c1ebdf92a3598676e58773df0"
  }
]
```

`GET /api/2/account/crypto/addresses/{currency}` - Get last 10 deposit addresses for currency

Requires the "Payment information" API key Access Right.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| address | String | Address for deposit |
| paymentId | String | Optional <br>If this field is presented, it is **required** for deposit. |
| publicKey | String | Optional <br>If this field is presented, it is **required** for deposit. |

## Last 10 used crypto address

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/crypto/used-addresses/NXT"
```

```json
[
  {
    "address": "NXT-G22U-BYF7-H8D9-3J27W",
    "paymentId": "f79779a3a0c7acc75a62afe8125de53106c6a19c1ebdf92a3598676e58773df0"
  }
]
```

`GET /api/2/account/crypto/used-addresses/{currency}` - Get last 10 unique addresses used for withdraw by currency

Note: used long ago addresses may be omitted, even if they are among last 10 unique addresses

Requires the "Payment information" API key Access Right.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| address | String | Address identifier |
| paymentId | String | Optional parameter |

## Withdraw crypto

```shell
curl -X POST -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/crypto/withdraw" \
    -d 'currency=BTC&amount=0.001&address=example_address&autoCommit=false'
```

```json
{
  "id": "d2ce578f-647d-4fa0-b1aa-4a27e5ee597b"
}
```

`POST /api/2/account/crypto/withdraw`

Requires the "Withdraw cryptocurrencies" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency code |
| amount | Number | The amount that will be sent to the specified address |
| address | String | Address identifier |
| paymentId | String | Optional parameter |
| includeFee | Boolean | Default value: `false` <br>If `true` is set, then total spent value will include fees. |
| autoCommit | Boolean | Default value: `true` <br>If `false` is set, then you should commit or rollback transaction in an hour. Used in two phase commit schema. |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Transaction unique identifier as assigned by exchange |

## Transfer convert between currencies

```shell
curl -X POST -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/crypto/transfer-convert" \
    -d 'fromCurrency=USDT20&toCurrency=usd&amount=0.001'
```

```json
{
  "result": ["d2ce578f-647d-4fa0-b1aa-4a27e5ee597b", "d2ce57hf-6g7d-4ka0-b8aa-4a27e5ee5i7b"]
}
```

`POST /api/2/account/crypto/transfer-convert`

Requires the "Payment information" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| fromCurrency | String | Currency code |
| toCurrency | String | Currency code |
| amount | Number | The amount that will be sent to the specified address |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| result | Array | Transaction unique identifiers as assigned by exchange |

## Withdraw crypto commit or rollback

```shell
curl -X PUT -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/crypto/withdraw/d2ce578f-647d-4fa0-b1aa-4a27e5ee597b"
```

```json
{
  "result": true
}
```

`PUT /api/2/account/crypto/withdraw/{id}` - Commit

`DELETE /api/2/account/crypto/withdraw/{id}` - Rollback

Requires the "Withdraw cryptocurrencies" API key Access Right.

> Both methods are idempotent

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| result | Boolean | `True`, if request is completed |


## Estimate withdraw fee

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/crypto/estimate-withdraw?currency=BTC&amount=0.01"
```

```json
{
  "fee": "0.0008"
}
```

`GET /api/2/account/crypto/estimate-withdraw`

Requires the "Withdraw cryptocurrencies" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency code |
| amount | Number | Expected withdraw amount |

Responses:

| Name | Type | Description |
|:---- |:---- |:----------- |
| fee | Number | |


## Check if crypto address belongs to current account

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/crypto/is-mine/1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2"
```

```json
{
  "result": true
}
```

`GET /api/2/account/crypto/is-mine/{address}`

Requires the "Payment information" API key Access Right.

Responses:

| Name | Type | Description |
|:---- |:---- |:----------- |
| result | Boolean | `True`, if address belongs to current account |


## Transfer money between trading account and bank account

```shell
curl -X POST -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/transfer" \
    -d "currency=eth&amount=0.01&type=bankToExchange"
```

```json
{
  "id": "d2ce578f-647d-4fa0-b1aa-4a27e5ee597b"
}
```

`POST /api/2/account/transfer`

Requires the "Payment information" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency code |
| amount | Number | The amount that will be transferred between balances |
| type | String | Accepted values: `bankToExchange`,  `exchangeToBank` |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Transaction unique identifier as assigned by exchange |


## Transfer money to another user by email or username

```shell
curl -X POST -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/transfer/internal" \
    -d "by=email&identifier=user@example.com&currency=BTC&amount=0.001"
```

```json
{
  "result": "fd3088da-31a6-428a-b9b6-c482673ff0f2"
}
```

`POST /api/2/account/transfer/internal`

Requires the "Payment information" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency code |
| amount | Number | The amount that will be transferred |
| by | String | Accepted values: `email`,  `username` |
| identifier | String | Identifier value. Either email or username |

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| result | String | Transaction unique identifier as assigned by exchange |


## Get transactions history

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/account/transactions?currency=ETH&sort=DESC"
```

```json
[
  {
    "id": "6a2fb54d-7466-490c-b3a6-95d8c882f7f7",
    "index": 20400458,
    "currency": "ETH",
    "amount": "38.616700000000000000000000",
    "fee": "0.000880000000000000000000",
    "address": "0xfaEF4bE10dDF50B68c220c9ab19381e20B8EEB2B",        
    "hash": "eece4c17994798939cea9f6a72ee12faa55a7ce44860cfb95c7ed71c89522fe8",
    "status": "pending",
    "type": "payout",
    "createdAt": "2017-05-18T18:05:36.957Z",
    "updatedAt": "2017-05-18T19:21:05.370Z"
  }
]
```

`GET /api/2/account/transactions`

`GET /api/2/account/transactions/{id}` - get transaction by transaction id

Requires the "Payment information" API key Access Right.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| currency | String | Currency code |
| sort | String | Sort direction<br>Accepted values: `DESC`, `ASC`<br>Default value: `DESC` |
| by | String | Defines filter type <br>Accepted values: `timestamp` or `index`<br>Default value `timestamp` |
| from | Datetime or Number | Interval initial value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value. |
| till | Datetime or Number | Interval end value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value. |
| limit | Number | Default value: 100<br>Max value: 1000 |
| offset | Number | Default value: 0<br>Max value: 100000 |
| showSenders | Boolean | Default value: false<br>Show senders addresses for payins |


Responses:

| Name | Type | Description |
|:---|:---:|:---|
| id | String | Transaction unique identifier as assigned by exchange |
| index | Number | Internal index value that represents when the entry was updated |
| currency | String | Currency code |
| amount | Number | Amount of traded currency |
| fee | Number | Payment commission value |
| address | String | Address identifier |
| paymentId | String | Optional parameter |
| hash | String | Transaction hash |
| status | String | Accepted values: `created`, `pending`, `failed`, `success` |
| type | String | Accepted values: `payout` (crypto withdraw transaction), `payin` (crypto deposit transaction), `deposit`, `withdraw`, `bankToExchange`, `exchangeToBank` |
| subType | String | Optional parameter. Currently accepted values: `swap` (swap between currenices), `offchain` (offchain transaction). Warning: possible subTypes list may be extended in future |
| senders | Array of String | senders for this payin transaction<br>shown only for payins and showSenders = true |
| offchainId | Array of String | Transaction id of external system |
| confirmations | Number | Optional parameter. Current count of confirmations for transaction in network |
| errorCode | String | Possible payout error reason: `INVALID_ADDRESS`, `INVALID_PAYMENT_ID`, `BAD_PRECISION` |
| createdAt | Datetime | Transaction creation date |
| updatedAt | Datetime | Date of transactions's last update  |

<% if to_show_sub_acc -%>
# Sub-accounts

## Get sub-accounts list

```shell
curl -X GET -u "publicKey:secretKey" "https://api.hitbtc.com/api/2/sub-acc"
```

> The above command returns JSON structured like this:

```json
{
  "result": [
    {"id": 7988900, "email": "user+1@example.com", "status": "active"},
    {"id": 7988906, "email": "user+2@example.com", "status": "active"},
    {"id": 7988909, "email": "user+3@example.com", "status": "disable"},
    {"id": 7988918, "email": "user+4@example.com", "status": "disable"},
    {"id": 7988921, "email": "user+5@example.com", "status": "disable"},
    {"id": 7989017, "email": "user+6@example.com", "status": "new"},
    {"id": 8001467, "email": "user+8@example.com", "status": "new"}
  ]
}
```

> Error response example:

> Failed authorization:

```json
{
  "error": {
    "code": 1002,
    "message": "Authorization is required or has been failed"
  }
}
```

> Empty sub-account's list:

```json
{
  "result": []
}
```

`GET /api/2/sub-acc`

Returns list of sub-accounts per a super account.

Requires no API key Access Rights.

Responses:

|Name     |Type    |Description                                                               |
|:--------|:------:|:-------------------------------------------------------------------------|
|id       |Number  |Unique identifier of a sub-account.                                       |
|email    |String  |Email address of a sub-account.                                           |
|status   |String  |User status of a sub-account. Accepted values: `new`, `active`, `disable`.|

## Freeze sub-account

```shell
curl -X POST -u "publicKey:secretKey" "https://api.hitbtc.com/api/2/sub-acc/freeze" \
    -d "ids=7988909"
```

> The above command returns JSON structured like this:

```json
{
  "result": true
}
```

> Error response example:

> Sub-accounts are already frozen or disabled:

```json
{
  "error": {
    "code": 500,
    "message": "Internal Server Error",
    "description": "account 7988921 already freeze or disabled"
  }
}
```


`POST /api/2/sub-acc​/freeze`

Freezes sub-accounts listed. It implies that the Sub-accounts frozen wouldn't be able to:

- login;
- withdraw funds;
- trade;
- complete pending orders;
- use API keys.

For any sub-account listed, all orders will be canceled and all funds will be transferred form the Trading balance.

Requires no API key Access Rights.

Parameters:

|Name     |Type    |Description                                                               |
|:--------|:------:|:-------------------------------------------------------------------------|
|ids      |Array   |Sub-accounts' userIds separated by commas (,). Those could be obtained by `GET /api/2/sub-acc` request.|

Responses:

|Name     |Type    |Description                                                               |
|:--------|:------:|:-------------------------------------------------------------------------|
|result   |Boolean |Value indicating, whether sub-accounts were successfully frozen.          |

## Activate sub-account


```shell
curl -X POST -u "publicKey:secretKey" "https://api.hitbtc.com/api/2/sub-acc/activate" \
    -d 'ids=7988909,7988918'
```

> The above command returns JSON structured like this:

```json
{
  "result": true
}
```

> Error response example:

> Sub-accounts are disabled, and their functionality can't be restored through activation:

```json
{
  "result": false
}
```

> Failed authorization:

```json
{
  "error": {
    "code": 1002,
    "message": "Authorization is required or has been failed"
  }
}
```

> Wrong input data format:

```json
{
  "error": {
    "code": 10001,
    "message": "Validation error",
    "description": "Param 'ids' required"
  }
}
```

> Sub-accounts listed don't exist:

```json
{
  "error": {
    "code": 500,
    "message": "Internal Server Error",
    "description": "Cannot find sub account 7988920"
  }
}

```


`POST /api/2/sub-acc​/activate`

Activates sub-accounts listed. It would make sub-accounts active after being frozen.

Requires no API key Access Rights.

Parameters:

|Name|Type |Description                                                                                     |
|:---|:---:|:-----------------------------------------------------------------------------------------------|
|ids |Array|Sub-accounts' userIds separated by commas (,). Those could be obtained by `GET /api/2/sub-acc` request.|

Responses:

|Name  |Type   |Description                                                        |
|:-----|:-----:|:------------------------------------------------------------------|
|result|Boolean|Value indicating, whether sub-accounts were successfully activated.|             

## Transfer funds


```shell
curl -X POST -u "publicKey:secretKey" "https://api.hitbtc.com/api/2/sub-acc/transfer" \
    -d "subAccountId=7988900&amount=1&currency=BTC&type=deposit"
```

> The above command returns JSON structured like this:

```json
{
  "result": "ae37e806-0191-45fc-8c49-18137274772c"
}

```

> Error response example:

> Insufficient permissions:

```json
{
  "error": {
    "code": 1003,
    "message": "Action is forbidden for this API key"
  }
}

```

> Sub-account is frozen or disabled:

```json
{
  "error": {
    "code": 500,
    "message": "Internal Server Error",
    "description": "Bad Request"
  }
}
```

> Insufficient funds:

```json
{
  "error": {
    "code": 20001,
    "message": "Internal Server Error",
    "description": "Bad Request"
  }
}
```

`POST /api/2/sub-acc​/transfer`

Transfers funds from the master account to sub-account or from sub-account to the master account.

Requires the "Withdraw cryptocurrencies" API key Access Right.


Parameters:

|Name        |Type  |Description                                                 |
|:-----------|:----:|:-----------------------------------------------------------|
|subAccountId|Number|Identifier of a sub-account to deposit/withdraw funds.   |
|amount      |Number|Amount of funds to transferred.                             |
|currency    |String|Name (code) of base currency, for example, "BTC".           |
|type        |String|Type of transaction. Accepted values: `deposit`, `withdraw`.|

Responses:

|Name     |Type  |Description                                                               |
|:--------|:----:|:-------------------------------------------------------------------------|
|result   |String|Identifier of the transaction resulting.                               |


## Get withdrawal settings

```shell
curl -X GET -u "publicKey:secretKey" "https://api.hitbtc.com/api/2/sub-acc/acl?subAccountIds=7988906"
```

>The above command returns JSON structured like this:

```json
{
    "result": [
      {
        "subAccountId": 7988906,
        "isPayoutEnabled": true,
        "description": ""
      }
    ]
}
```

>Error response example:

>Insufficient permissions:

```json
{
  "error": {
    "code": 1003,
    "message": "Action is forbidden for this API key"
  }
}

```

>Sub-account is frozen or disabled:

```json
{
  "result": []
}

```

`GET /api/2/sub-acc/acl`

Returns a list of withdrawal settings for sub-accounts listed.

Requires the "Payment information" API key Access Right.

Parameters:

|Name         |Type |Description                                                                                     |
|:------------|:---:|:-----------------------------------------------------------------------------------------------|
|subAccountIds|Array|Sub-accounts' userIds separated by commas (,). Those could be obtained by `GET /api/2/sub-acc` request.|

Responses:

|Name           |Type   |Description                                                              |
|:--------------|:-----:|:------------------------------------------------------------------------|
|subAccountId   |Number |Unique identifier of a sub-account.                                      |
|isPayoutEnabled|Boolean|Value indicating, whether withdrawals are enabled (true) or not (false). |
|description    |String |Textual description. Normally left empty.                                |

## Change withdrawal settings

```shell
curl -X PUT -u "publicKey:secretKey" "https://api.hitbtc.com/api/2/sub-acc/acl/7988900"
    -d "isPayoutEnabled=false&description=spam"
```

> The above command returns JSON structured like this:

```json
{
  "result": [
    {
      "id": 7988900,
      "isPayoutEnabled": true,
      "description": "spam"
    }
  ]
}
```

>Error response example:

>Sub-account is frozen or disabled:

```json
{
  "error": {
    "code": 10021,
    "message": "user disabled"
  }
}

```

>Insufficient permissions:

```json
{
  "error": {
    "code": 1003,
    "message": "Action is forbidden for this API key"
  }
}

```

`PUT /api/2/sub-acc/acl​/{subAccountUserId}`

Disables or enables withdrawals for a sub-account.

Requires the "Payment information" API key Access Right.

Parameters:

|Name           |Type.  |Description                                             |
|:--------------|:-----:|:-------------------------------------------------------|
|isPayoutEnabled|Boolean|Value indicating the desired state of withdrawals.      |
|description    |String |Textual description.                                    |

Responses:

|Name           |Type   |Description                                                              |
|:--------------|:-----:|:------------------------------------------------------------------------|
|subAccountId   |Number |Unique identifier of a sub-account.                                      |
|isPayoutEnabled|Boolean|Value indicating, whether withdrawals are enabled (true) or not (false). It must be equal to the value in a request.|
|description    |String |Textual description. It duplicates the value in a request.               |


## Get sub-account balance

```shell
curl -X GET -u "publicKey:secretKey" "https://api.hitbtc.com/api/2/sub-acc/balance/7988900"
```

>The above command returns JSON structured like this:

```json
{
  "result": {
    "main": [
      {
        "currency": "1ST",
        "available": "0.0",
        "reserved": "0.0"
      }
    ],
    "trading": [
      {
        "currency": "1ST",
        "available": "0",
        "reserved": "0"
      }
    ]
  }
}

```

>Error response example:

>Insufficient permissions:

```json
{
  "error": {
    "code": 1003,
    "message": "Action is forbidden for this API key"
  }
}
```

`GET /api/2/sub-acc​/balance​/{subAccountUserID}`

Returns balance values by sub-account ID specified. Report will include main account and Trading balances for each currency. It is functional with no regard to the state of a sub-account.

Requires the "Payment information" API key Access Right.


## Get sub-account crypto address

```shell
curl -X GET -u "publicKey:secretKey" \
    "https://api.hitbtc.com/api/2/sub-acc/deposit-address/7988900/NXT"
```

>The above command returns JSON structured like this:

```json
{
    "result": {
        "address": "NXT-G22U-BYF7-H8D9-3J27W",
        "publicKey": "f79779a3a0c7acc75a62afe8125de53106c6a19c1ebdf92a3598676e58773df0"
    }
}
```

`GET /api/2/sub-acc/deposit-address/{subAccountUserId}/{currency}`

Returns sub-account crypto address for currency.

Requires the "Payment information" API key Access Right.

Responses:

| Name | Type | Description |
|:---|:---:|:---|
| address | String | Address for deposit |
| paymentId | String | Optional parameter <br>If this parameter is set, it is **required** for deposit. |
| publicKey | String | Optional parameter <br>If this parameter is set, it is **required** for deposit. |


## Errors

|Error code|HTTP Status Code|Message                             |Note                                                          |
|:---------|:--------------:|:-----------------------------------|:-------------------------------------------------------------|
|10021     |400             |user disabled                       |Sub-account is frozen or disabled                             |
|500       |500             |Internal Server Error               |Cannot find sub account `subAccountId`                        |

Following errors are conventional:

|Error code|HTTP Status Code|Message                             |Note                                                          |
|:---------|:--------------:|:-----------------------------------|:-------------------------------------------------------------|
|10001     |400             |Validation error                    |Param 'ids' required.                                         |
|1003      |403             |Action is forbidden for this API key|Check Access Rights for API key.                              |          
|20001     |400             |Insufficient funds                  |Insufficient funds for creating order or any account operation|
|1002      |401             |Authorization failed                |                                                              |
|504       |504             |Gateway Timeout                     |Check the result of your request later                        |
|503       |503             |Service Unavailable                 |Try it again later                                            |
|1001      |401             |Authorization required              |                                                              |
|2001      |400             |Symbol not found                    |                                                              |
|2002      |400             |Currency not found                  |                                                              |
|600       |400             |Action is not allowed               |                                                              |
<% end -%>

# Socket API Reference

Use JSON-RPC 2.0 over WebSocket connection as transport.

## Request object

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

An RPC call is represented by sending a Request object to a Server.

The Request object has the following members:

 * `method` - a String containing the name of the method to be invoked.
 * `params` - a Structured value that holds the parameter values to be used during the invocation of the method.
 * `id` - An identifier established by the Client that MUST contain a String, Number, or NULL value if included. If it is not included it is assumed to be a notification. The value SHOULD NOT be Null.

## Notification

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

A Notification is a Request object without an `id` member. A Request object (a Notification) signifies the lack of the Client's interest in the corresponding Response object. Therefore no Response objects need to be returned to the Client.

The Notification object has the following members:

 * `method` - a string containing the name of the method to be invoked.
 * `params` - a structured value holding the parameter values to be used during the method invocation.

## Response object

When RPC call is made, the Server MUST reply with a Response, except for Notifications cases.

Server responses for each request. Response on success subscription is `true`. Example:

```json
{
  "jsonrpc": "2.0",
  "result": true,
  "id": 123
}
```
> Response error

```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": 2001,
    "message": "Symbol not found",
    "description": "Try get /api/2/public/symbol, to get list of all available symbols."
  },
  "id": 123
}

```

The Response is represented as a single JSON Object, with the following members:

 * `result` - this member is REQUIRED on success. This member MUST NOT exist if there was an error during method invocation. The value of this member is determined by the method invoked on the Server.
 * `error` - this member is REQUIRED on error. This member MUST NOT exist if there was no error triggered during method invocation. The value for this member MUST be an Object as defined in the [Error response](#error_response) section.


# Socket Market Data

The parameters of requests, responses, and errors correspond to REST, but have different usage procedures. <br>
First, you should subscribe for the required data. Then the Server will send a full snapshot of the data. After that, the Server sends an update notification.


## Get Currencies

```shell
wscat -c wss://api.hitbtc.com/api/2/ws

{
  "method": "getCurrency",
  "params": {
    "currency": "ETH"
  },
  "id": 123
}
```

> Response

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
    "transferEnabled": true,
    "delisted": false,
    "payoutFee": "0.001",
    "payoutMinimalAmount": "0.00958",
    "precisionPayout": 10,
    "precisionTransfer": 10
  },
  "id": 123
}
```

Request methods: **`getCurrencies`**,**`getCurrency`**

> Request:

```json
{
  "method": "getCurrency",
  "params": {
    "currency": "ETH"
  },
  "id": 123
}
```

## Get Symbols

```shell
wscat -c wss://api.hitbtc.com/api/2/ws

{
  "method": "getSymbol",
  "params": {
    "symbol": "ETHBTC"
  },
  "id": 123
}
```

> Response

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

Request methods: **`getSymbols`**,**`getSymbol`**


## Subscribe to Ticker

```shell
wscat -c wss://api.hitbtc.com/api/2/ws

{
  "method": "subscribeTicker",
  "params": {
    "symbol": "ETHBTC"
  },
  "id": 123
}
```
> Notification ticker

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

Request methods: **`subscribeTicker`**, **`unsubscribeTicker`**

Notification method: **`ticker`**


## Subscribe to Order Book

```shell
wscat -c wss://api.hitbtc.com/api/2/ws

{
  "method": "subscribeOrderbook",
  "params": {
    "symbol": "ETHBTC"
  },
  "id": 123
}
```

> Notification snapshot

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
      }
    ],
    "symbol": "ETHBTC",
    "sequence": 8073827,    
    "timestamp": "2018-11-19T05:00:28.193Z"
  }
}
```

> Notification update

```json
{
  "jsonrpc": "2.0",
  "method": "updateOrderbook",
  "params": {    
    "ask": [
      {
        "price": "0.054590",
        "size": "0.000"
      },
      {
        "price": "0.054591",
        "size": "0.000"
      }
    ],
    "bid": [
      {
        "price": "0.054504",
         "size": "0.000"
      }
    ],
    "symbol": "ETHBTC",
    "sequence": 8073830,
    "timestamp": "2018-11-19T05:00:28.700Z"
  }
}
```

Request methods: **`subscribeOrderbook`**, **`unsubscribeOrderbook`**


**Notification snapshot**

Message contains a full snapshot of the Order Book.<br>
<br>`sequence` - same as **`updateOrderbook`** sequence field
Notification method: **`snapshotOrderbook`**


**Notification update**

Message contains incremental changes.
<br>`size` **= 0** means that level has been deleted.
<br>`sequence` is monotonically increased for each update, each symbol has its own sequence.
<br>Notification method: **`updateOrderbook`**


## Subscribe to Trades

```shell
wscat -c wss://api.hitbtc.com/api/2/ws

{
  "method": "subscribeTrades",
  "params": {
    "symbol": "ETHBTC",
    "limit": 100
  },
  "id": 123
}
```

Request methods: **`subscribeTrades`**, **`unsubscribeTrades`**


> Notification snapshot

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


Notification method: **`snapshotTrades`**

> Notification update

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

Notification method: **`updateTrades`**


## Get Trades

```shell
wscat -c wss://api.hitbtc.com/api/2/ws

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
> Response

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

Request method: **`getTrades`**


Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String | Optional parameter to filter active orders by symbol |
| sort | String | Sort direction<br>Accepted values: `DESC` or `ASC`<br>Default value `DESC` |
| by | String | Defines filter type <br>Accepted values: `timestamp` or `id`<br>Default value: `timestamp`|
| from | Datetime or Number | Interval initial value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value. |
| till | Datetime or Number | Interval end value (optional parameter) <br>If sorting by `timestamp` is used, then `Datetime`, otherwise `Number` of index value. |
| limit | Number | Default value: 100<br>Max value: 1000 |
| offset | Number | Default value: 0<br>Max value: 100000 |



## Subscribe to Candles

Request methods: **`subscribeCandles`**, **`unsubscribeCandles`**


```shell
wscat -c wss://api.hitbtc.com/api/2/ws

{
  "method": "subscribeCandles",
  "params": {
    "symbol": "ETHBTC",
    "period": "M30",
    "limit": 100
  },
  "id": 123
}     
```

> Notification snapshot

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

Notification method: **`snapshotCandles`**


> Notification update

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

Notification method: **`updateCandles`**


# Socket Session Authentication

> Example with BASIC algo:

```shell

wscat -c wss://api.hitbtc.com/api/2/ws

{
  "method": "login",
  "params": {
    "algo": "BASIC",
    "pKey": "3ef4a9f8c8bf04bd8f09884b98403eae",
    "sKey": "2deb570ab58fd553a4ed3ee249fd2d51"
  }
}
```    

> Example with HS256 algo:

```shell

wscat -c wss://api.hitbtc.com/api/2/ws

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

<aside class="notice">    
You should authenticate session once before request requires authorisation.
</aside>    

Request method: **`login`**

Params:

| Name | Type | Description |
|:---|:---:|:---|
| algo | String | Encryption algorithm<br>Accepted values: `BASIC` or `HS256` |
| pKey | String | API public key |
| sKey | String | API secret key, required on `BASIC` algo |
| nonce | String | Random string, required on `HS256` algo |
| signature | String | HMAC SHA256 sign nonce with API secret key, required on `HS256` algo |



# Socket Trading

Trade via socket has some major differences compared to REST:

 * Quickness. The time to place a new order is a bit higher than network latency.
 * The Server notifies you of any order updates.
 * FIFO. Your requests are executed on a First In First Out basis.

## Subscribe to reports


```shell
wscat -c wss://api.hitbtc.com/api/2/ws

{
  "method": "subscribeReports",
  "params": {}
}
```

> Notification with currently active orders

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
      "postOnly": false,
      "createdAt": "2017-10-20T12:17:12.245Z",
      "updatedAt": "2017-10-20T12:17:12.245Z",
      "reportType": "status"
    }
  ]
}
```

Method: **`subscribeReports`**


<aside class="notice">
A notification with active orders is sent after subscription or on any service maintainance.
</aside>

Method: **`activeOrders`**


> Notification report

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
    "postOnly": false,
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

Method: **`report`**

Params:

| Name | Type | Description |
|:---|:---:|:---|    
| id | Number | Order unique identifier as assigned by exchange |
| clientOrderId | String | Order unique identifier as  assigned by trader |
| symbol| String| Trading symbol |
| side | String | Trade side<br>Accepted values: `sell`, `buy` |
| status | String | Accepted values: `new`, `suspended`, `partiallyFilled`, `filled`, `canceled`, `expired` |
| type | String | Accepted values: `limit`, `market`, `stopLimit`, `stopMarket` |
| timeInForce | String | Time in Force is a special instruction used when placing a trade to indicate how long an order will remain active before it is executed or expired. <br> **GTC** - ''Good-Till-Cancel'' order won't close until it is filled. <br> **IOC** - ''Immediate-Or-Cancel'' order must be executed immediately. Any part of an IOC order that cannot be filled immediately will be cancelled. <br> **FOK** - ''Fill-Or-Kill'' is a type of ''Time in Force'' designation used in securities trading that instructs a brokerage to execute a transaction immediately and completely or not execute it at all. <br> **Day** - keeps the order active until the end of the trading day (UTC). <br> **GTD** - ''Good-Till-Date''. The date is specified in `expireTime`. |
| quantity | Number | Order quantity |
| price | Number | Order price |
| cumQuantity | Number | Cumulative executed quantity |
| postOnly | Boolean | A post-only order is an order that does not remove liquidity. If your post-only order causes a match with a pre-existing order as a taker, then the order will be cancelled. |
| createdAt | Datetime | Report creation date |
| updatedAt | Datetime | Date of the report's last update |
| stopPrice | Number | Required for stop-limit and stop-market orders |
| expireTime | Datetime | Date of order expiration for `timeInForce` = `GTD` |   
| **reportType** | String | Accepted values: `status`, `new`, `canceled`, `expired`, `suspended`, `trade`, `replaced` |
| tradeId | Number | Trade identifier. Required for `reportType` = `trade` |   
| tradeQuantity | Number | Quantity of trade. Required for `reportType` = `trade`  |   
| tradePrice | Number | Trade price. Required for `reportType` = `trade` |   
| tradeFee | Number | Fee paid for trade. Required for `reportType` = `trade` |   
| originalRequestClientOrderId | String | Identifier of replaced order |   


## Place new order

> Request:

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

> Success response:

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
    "postOnly": false,
    "createdAt": "2017-10-20T12:29:43.166Z",
    "updatedAt": "2017-10-20T12:29:43.166Z",
    "reportType": "new"
  },
  "id": 123
}
```

> Example error response:

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

Method: **`newOrder`**

Params:

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Required parameter. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| symbol| String| Trading symbol |
| side | String | Trade side. Accepted values: `sell`, `buy` |
| type | String | Optional parameter <br>Accepted values: `limit`, `market`, `stopLimit`, `stopMarket` <br>Default value: `limit` |
| timeInForce | String | Optional parameter<br>Accepted values: `GTC`, `IOC`, `FOK`, `Day`, `GTD`<br>Default value: `GTC` |
| quantity | Number | Order quantity |
| price | Number | Order price. Required for limit types |
| stopPrice | Number | Required for stop-limit and stop-market orders |
| expireTime | Datetime | Required for `timeInForce` = `GTD` |    
| strictValidate | Boolean | Price and quantity will be checked for incrementation within the symbol's tick size and quantity step. See the symbol's `tickSize` and `quantityIncrement` |
| postOnly | Boolean| A post-only order is an order that does not remove liquidity. If your post-only order causes a match with a pre-existing order as a taker, then order will be cancelled. |



<aside class="notice">
A `report` notification will arrive before request result.
</aside>

## Cancel order

> Request:

```json
{
  "method": "cancelOrder",
  "params": {
    "clientOrderId": "57d5525562c945448e3cbd559bd068c4"
  },
  "id": 123
}
```

> Response:

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
    "postOnly": false,
    "createdAt": "2017-10-20T12:29:43.166Z",
    "updatedAt": "2017-10-20T12:31:26.174Z",
    "reportType": "canceled"
  },
  "id": 123
}
```

Method: **`cancelOrder`**


## Cancel/Replace order

> Request:

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

> Response:

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
    "postOnly": false,
    "createdAt": "2017-10-20T12:47:07.942Z",
    "updatedAt": "2017-10-20T12:50:34.488Z",
    "reportType": "replaced",
    "originalRequestClientOrderId": "9cbe79cb6f864b71a811402a48d4b5b1"
  },
  "id": 123
}
```

The Cancel/Replace request is used to change the parameters of an existing order and to change the quantity or price attribute of an open order.

Do not use this request to cancel the quantity remaining in an outstanding order. Use the Cancel request message for this purpose.

It is stipulated that a newly entered order cancels a prior order that has been entered, but not yet executed.

Method: **`cancelReplaceOrder`**

Params:

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Replaced order (required parameter)|
| requestClientId | String | `clientOrderId` for new order (required parameter) <br>Uniqueness must be guaranteed within a single trading day, including all active orders. |
| quantity | Number | New order quantity |
| price | Number | New order price |
| strictValidate | Boolean | Price and quantity will be checked for the incrementation within tick size and quantity step. See symbol's `tickSize` and `quantityIncrement` |

## Get active orders


> Request:

```json
{
  "method": "getOrders",
  "params": {},
  "id": 123
}
```

> Response:

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
      "postOnly": false,
      "createdAt": "2017-10-20T12:47:07.942Z",
      "updatedAt": "2017-10-20T12:50:34.488Z",
      "reportType": "replaced",
      "originalRequestClientOrderId": "9cbe79cb6f864b71a811402a48d4b5b1"
    }
  ],
  "id": 123
}
```

Method: **getOrders**


## Get trading balance

> Request:

```json
{
  "method": "getTradingBalance",
  "params": {},
  "id": 123
}
```

> Response:

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

Method: **`getTradingBalance`**


<% if to_show_margin_trading -%>

# Socket Margin Trading

## Description

API provided the following tools to manage margin trading:

* subscribe to the reports about:
  * isolated margin account changes;
  * margin orders.
* setup isolated margin account &mdash; reserve amount for margin trading per symbol;
* operate with margin orders: 
  * observe orders;
  * place a new order; 
  * cancel an order;
  * alter an order;
  * cancel all orders.
* close positions.


## Subscribe to Reports
Method: **`marginSubscribe`**

Income methods: **`marginOrders`**, **`marginAccounts`**


> Subsription result:

```json
{"jsonrpc":"2.0","result":true,"id": 1234}
```


> Notification. Margin orders list:

```json
{
  "jsonrpc": "2.0",
  "method": "marginOrders",
  "params": [
    {
      "id": 262759024150,
      "clientOrderId": "6e5a2520dfe14d1bbb07b162909bc282",
      "symbol": "BTCUSD",
      "side": "buy",
      "status": "new",
      "type": "limit",
      "timeInForce": "GTC",
      "price": "100.00",
      "quantity": "0.00001",
      "postOnly": false,
      "cumQuantity": "0",
      "createdAt": "2020-07-04T09:49:02.704Z",
      "updatedAt": "2020-07-04T09:49:02.704Z",
      "reportType": "status"
    }
  ]
}
```

> Notification. Isolated Margin Accounts list:

```json
{
  "jsonrpc": "2.0",
  "method": "marginAccounts",
  "params": [
    {
      "symbol": "BTCUSD",
      "leverage": "10.00",
      "marginBalance": "3.000000000000",
      "marginBalanceOrders": "0.001052521050",
      "marginBalanceRequired": "0",
      "createdAt": "2020-07-04T09:45:47.256Z",
      "updatedAt": "2020-07-04T10:10:35.512Z",
      "reportType": "status",
      "reportReason": "status"
    }
  ]
}
```


## Notification Account Report

>Notification. Isolated Margin Account report:

```json
{
  "jsonrpc": "2.0",
  "method": "marginAccountReport",
  "params": {
    "symbol": "BTCUSD",
    "leverage": "10.00",
    "marginBalance": "2.999773378500",
    "marginBalanceOrders": "0",
    "marginBalanceRequired": "0.009314143650",
    "createdAt": "2020-07-04T12:11:33.665Z",
    "updatedAt": "2020-07-04T12:11:34.424Z",
    "position": {
      "id": 79,
      "symbol": "BTCUSD",
      "quantity": "-0.00001",
      "pnl": "0",
      "priceEntry": "9064.86",
      "priceMarginCall": "284612.92",
      "priceLiquidation": "293626.79",
      "createdAt": "2020-07-04T12:11:33.665Z",
      "updatedAt": "2020-07-04T12:11:34.424Z"
    },
    "reportType": "updated",
    "reportReason": "openTrade"
  }
}
```

Method: **`marginAccountReport`**

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol | String| Trading symbol. Where base currency is the currency of funds reserved on the trading account for positions and quote currency is the currency of funds reserved on a margin account (e.g. "BTCUSD"). |
| leverage | Number | Margin leverage. The ratio of the trader's own funds to funds borrowed from the platform. |
| marginBalance | Number | Amount of currency, reserved for margin purpose. |
| marginBalanceOrders | String | Amount of currency, reserved for margin orders. |
| marginBalanceRequired | String | Amount of currency, reserved for margin position close. |
| createdAt | Datetime | Account creation date and time. |
| updatedAt | Datetime | Account last update date and time. |
| position | Position | Optional parameter.<br> Open positions of the margin account. |
| reportType | String | <br>Accepted values:<br>- `status`: status of margin account requested e.g. subscription report with a list of current margin accounts;<br>- `created`: new margin account created or position e.g. new margin account requested or flipTrade occurs with position;<br>- `updated`: margin account or position updated;<br>- `closed`: margin account closed. |
| reportReason | String |  Values:<br> - `status`;<br>- `created`;<br>- `updated`;<br>- `marginChanged`;<br>- `openTrade`;<br>- `closeTrade`;<br>- `flipTrade`;<br>- `closed`;<br>- `reopened`;<br>- `liquidated`;<br>- `interestTaken`. |

Report type values:

| Status  | Description |
|---------|:------------|
| status  | Status of margin account requested e.g. get accounts or subscription report with a list of current margin accounts. |
| created | A new margin account created, e.g. new margin account has been created or recreated as a result of flip trade occurs with the position. |
| updated | Margin account or position has been updated. |
| closed  | Margin account has been closed. |

Report reason values:

| Status  | Description |
|---------|:------------|
| status | Response in account information request. |
| created | Margin account has been created. |
| updated | Margin account has been changed as a result of any order report e.g. new, canceled, filled and so on. |
| marginChanged | Margin account has been changed as a result of any requested change of the marginBalance. |
| openTrade | Opening trade has been executed. |
| closeTrade | Closing trade has been executed. |
| flipTrade | The position has been flipped as a result of opposite order execution with a quantity exceeding the position quantity (quantity has been changing sign). |
| closed | The position quantity has been set to 0 (zero) as a result of closing trade. |
| liquidated | The position has been liquidated. |
| interestTaken | The interest rate has been paid. |

## Notification Orders Report

>Notification. Margin orders report:

```json
{
  "jsonrpc": "2.0",
  "method": "marginOrderReport",
  "params": {
    "id": 262797201083,
    "clientOrderId": "93e5573062dd4a82a7fe8d5f91210a8c",
    "symbol": "BTCUSD",
    "side": "sell",
    "status": "filled",
    "type": "limit",
    "timeInForce": "GTC",
    "price": "9000.00",
    "quantity": "0.00001",
    "postOnly": false,
    "cumQuantity": "0.00001",
    "createdAt": "2020-07-04T12:11:34.424Z",
    "updatedAt": "2020-07-04T12:11:34.424Z",
    "reportType": "trade",
    "tradeId": 884145205,
    "tradeQuantity": "0.00001",
    "tradePrice": "9064.86",
    "tradeFee": "0.000226621500",
    "tradePositionId":117
  }
}
```

Method: **`marginOrderReport`**

Parameters:

| Name | Type | Description |
|:---|:---:|:---|    
| id | Number | Order unique identifier as assigned by exchange. |
| clientOrderId | String | Order unique identifier as  assigned by trader. |
| symbol| String| Trading symbol. |
| side | String | Trade side<br>Accepted values: `sell`, `buy` |
| status | String | Accepted values: `new`, `suspended`, `partiallyFilled`, `filled`, `canceled`, `expired` |
| type | String | Accepted values: `limit`, `market`, `stopLimit`, `stopMarket` |
| timeInForce | String | Time in Force is a special instruction used when placing a trade to indicate how long an order will remain active before it is executed or expired. <br> **GTC** - ''Good-Till-Cancel'' order won't close until it is filled. <br> **IOC** - ''Immediate-Or-Cancel'' order must be executed immediately. Any part of an IOC order that cannot be filled immediately will be cancelled. <br> **FOK** - ''Fill-Or-Kill'' is a type of ''Time in Force'' designation used in securities trading that instructs a brokerage to execute a transaction immediately and completely or not execute it at all. <br> **Day** - keeps the order active until the end of the trading day (UTC). <br> **GTD** - ''Good-Till-Date''. The date is specified in `expireTime`. |
| quantity | Number | Order quantity. |
| price | Number | Order price. |
| cumQuantity | Number | Cumulative executed quantity. |
| postOnly | Boolean | A post-only order is an order that does not remove liquidity. If your post-only order causes a match with a pre-existing order as a taker, then the order will be cancelled. |
| createdAt | Datetime | Report creation date. |
| updatedAt | Datetime | Date of the report's last update. |
| stopPrice | Number | Required for stop-limit and stop-market orders. |
| expireTime | Datetime | Date of order expiration for `timeInForce` = `GTD` |   
| reportType | String | Accepted values:<br>- `status`;- `new`;<br>- `canceled`;<br>- `expired`;<br>- `suspended`;<br>- `trade`;<br>- `replaced`. |
| tradeId | Number | Trade identifier. Required for `reportType` = `trade` |   
| tradeQuantity | Number | Quantity of trade. Required for `reportType` = `trade`  |   
| tradePrice | Number | Trade price. Required for `reportType` = `trade` |   
| tradeFee | Number | Fee paid for trade. Required for `reportType` = `trade` |   
| tradePositionId | Number | Position identifier of the trade |
| originalRequestClientOrderId | String | Identifier of replaced order. |


## Get Isolated Margin Accounts

>Request:

```json
{
  "method": "marginAccountsGet",
  "params": {},
  "id": 123
}
```

>Response: 

```json
{
  "jsonrpc": "2.0",
  "result": [
    {
      "symbol": "BTCUSD",
      "leverage": "10.00",
      "marginBalance": "3.000000000000",
      "marginBalanceOrders": "0",
      "marginBalanceRequired": "0",
      "createdAt": "2020-07-04T11:19:28.444Z",
      "updatedAt": "2020-07-04T11:19:28.444Z",
      "reportType": "status",
      "reportReason": "status"
    }
  ],
  "id": 123
}
```

Method: **`marginAccountsGet`**

Returns a list of Isolated Margin Accounts with details about open positions.


## Setup Isolated Margin Account

>Request:

```json
{
  "method": "marginAccountSet",
  "jsonrpc": "2.0",
  "params": {
    "marginBalance": "3.000",
    "symbol": "BTCUSD"
  },
  "id": 1234
}
```

>Response:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "symbol": "BTCUSD",
    "leverage": "10.00",
    "marginBalance": "3.000000000000",
    "marginBalanceOrders": "0",
    "marginBalanceRequired": "0",
    "createdAt": "2020-07-04T11:19:28.444Z",
    "updatedAt": "2020-07-04T11:19:28.444Z",
    "reportType": "created",
    "reportReason": "created"
  },
  "id": 1234
}
```

>Notification:

```json
{
  "jsonrpc": "2.0",
  "method": "marginAccountReport",
  "params": {
    "symbol": "BTCUSD",
    "leverage": "10.00",
    "marginBalance": "3.000000000000",
    "marginBalanceOrders": "0",
    "marginBalanceRequired": "0",
    "createdAt": "2020-07-04T11:19:28.444Z",
    "updatedAt": "2020-07-04T11:19:28.444Z",
    "reportType": "created",
    "reportReason": "created"
  }
}
```

Method: **`marginAccountSet`**

Adds a margin for the specified symbol.
Creates or updates Isolated Margin Account.
Setting margin balance to zero will lead to closing Isolated Margin Account and retrieval all formerly reserved funds to the trading account.


Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| symbol| String| Trading symbol. Where base currency is the currency of funds reserved on the trading account for positions and quote currency is the currency of funds reserved on a margin account (e.g. "BTCUSD"). |
| marginBalance | Number | Amount of currency reserved. |
| strictValidate | Boolean | The value indicating whether the `marginBalance` going to be checked for correct non exponential format and currency precision. |

## Close Position

>Request:

```json
{
  "method": "marginPositionClose",
  "jsonrpc": "2.0",
  "params": {
    "symbol": "BTCUSD"
  },
  "id": 1243
}
```

>Notification:

```json
{
  "jsonrpc": "2.0",
  "method": "marginAccountReport",
  "params": {
    "symbol": "BTCUSD",
    "leverage": "10.00",
    "marginBalance": "2.999546017750",
    "marginBalanceOrders": "0",
    "marginBalanceRequired": "0",
    "createdAt": "2020-07-04T12:20:22.629Z",
    "updatedAt": "2020-07-04T12:20:23.481Z",
    "reportType": "closed",
    "reportReason": "closed"
  }
}
```

>Response:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "symbol": "BTCUSD",
    "leverage": "10.00",
    "marginBalance": "2.999773460000",
    "marginBalanceOrders": "0",
    "marginBalanceRequired": "0.014295621250",
    "createdAt": "2020-07-04T12:20:22.629Z",
    "updatedAt": "2020-07-04T12:20:23.481Z",
    "position": {
      "id": 81,
      "symbol": "BTCUSD",
      "quantity": "-0.00001",
      "pnl": "0",
      "priceEntry": "9061.60",
      "priceMarginCall": "284609.92",
      "priceLiquidation": "293623.70",
      "createdAt": "2020-07-04T12:20:22.629Z",
      "updatedAt": "2020-07-04T12:20:23.481Z"
    },
    "reportType": "updated",
    "reportReason": "updated"
  },
  "id": 1243
}
```

Method: **`marginPositionClose`**

Closes a position for the specified symbol. This will result in cancelling all open orders within the position.


## Get Margin Orders

>Request:

```json
{
  "method": "marginOrdersGet",
  "jsonrpc": "2.0",
  "params": {},
  "id": 1240
}
```

>Response:

```json
{
  "jsonrpc": "2.0",
  "result": [
    {
      "id": 262795082247,
      "clientOrderId": "72532cca006d406585d7cf267a0d9a36",
      "symbol": "BTCUSD",
      "side": "buy",
      "status": "new",
      "type": "limit",
      "timeInForce": "GTC",
      "price": "100.01",
      "quantity": "0.00001",
      "postOnly": false,
      "cumQuantity": "0",
      "createdAt": "2020-07-04T12:06:31.021Z",
      "updatedAt": "2020-07-04T12:06:31.145Z",
      "reportType": "status"
    }
  ],
  "id": 1240
}
```

Method: **`marginOrdersGet`**

Returns all active margin orders.


## Cancel Margin Orders

>Request:

```json
{
  "method": "marginOrdersCancel",
  "params": {},
  "id": 1270
}
```

>Response:

```json
{
  "jsonrpc": "2.0",
  "result": [
    {
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
    "postOnly": false,
    "createdAt": "2017-10-20T12:29:43.166Z",
    "updatedAt": "2017-10-20T12:31:26.174Z",
    "reportType": "canceled"
    }
  ],
  "id": 1270
}
```

Method: **`marginOrdersCancel`**

Cancel all active margin orders.


## Place New Margin Order

>Request:

```json
{
  "method": "marginOrderNew",
  "jsonrpc": "2.0",
  "params": {
    "symbol": "BTCUSD",
    "side": "buy",
    "quantity": "0.25",
    "price": "100.00",
    "clientOrderId": "2d42e072e4f34846954509c955303e27"
  },
  "id": 1238
}
```

>Notification. Order report:

```json
{
  "jsonrpc": "2.0",
  "method": "marginOrderReport",
  "params": {
    "id": 262805375143,
    "clientOrderId": "2d42e072e4f34846954509c955303e27",
    "symbol": "BTCUSD",
    "side": "buy",
    "status": "new",
    "type": "limit",
    "timeInForce": "GTC",
    "price": "100.00",
    "quantity": "0.25000",
    "postOnly": false,
    "cumQuantity": "0",
    "createdAt": "2020-07-04T12:34:43.423Z",
    "updatedAt": "2020-07-04T12:34:43.423Z",
    "reportType": "new"
  }
}
```

>Response:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "id": 262805375143,
    "clientOrderId": "2d42e072e4f34846954509c955303e27",
    "symbol": "BTCUSD",
    "side": "buy",
    "status": "new",
    "type": "limit",
    "timeInForce": "GTC",
    "price": "100.00",
    "quantity": "0.25000",
    "postOnly": false,
    "cumQuantity": "0",
    "createdAt": "2020-07-04T12:34:43.423Z",
    "updatedAt": "2020-07-04T12:34:43.423Z",
    "reportType": "new"
  },
  "id": 1238
}
```

> Notification. Account report:

```json
{
  "jsonrpc": "2.0",
  "method": "marginAccountReport",
  "params": {
    "symbol": "BTCUSD",
    "leverage": "10.00",
    "marginBalance": "3.000000000000",
    "marginBalanceOrders": "2.631250000000",
    "marginBalanceRequired": "0",
    "createdAt": "2020-07-04T12:34:43.157Z",
    "updatedAt": "2020-07-04T12:34:43.423Z",
    "reportType": "updated",
    "reportReason": "updated"
  }
}
```

Method: **`marginOrderNew`**

Places a new margin order.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Required parameter. Uniqueness must be guaranteed within a single trading day, including all active orders. |
| symbol| String| Trading symbol |
| side | String | Trade side. Accepted values: `sell`, `buy` |
| type | String | Optional parameter <br>Accepted values: `limit`, `market`, `stopLimit`, `stopMarket` <br>Default value: `limit` |
| timeInForce | String | Optional parameter<br>Accepted values: `GTC`, `IOC`, `FOK`, `Day`, `GTD`<br>Default value: `GTC` |
| quantity | Number | Order quantity |
| price | Number | Order price. Required for limit types |
| stopPrice | Number | Required for stop-limit and stop-market orders |
| expireTime | Datetime | Required for `timeInForce` = `GTD` |    
| strictValidate | Boolean | Price and quantity will be checked for incrementation within the symbol's tick size and quantity step. See the symbol's `tickSize` and `quantityIncrement` |
| postOnly | Boolean| A post-only order is an order that does not remove liquidity. If your post-only order causes a match with a pre-existing order as a taker, then order will be cancelled. |


## Cancel Margin Order

>Request:

```json
{
  "method": "marginOrderCancel",
  "jsonrpc": "2.0",
  "params": {
    "clientOrderId": "b7580a84e82a4390815942e666625e51"
  },
  "id": 1240
}
```

>Notification. Order report:

```json
{
  "jsonrpc": "2.0",
  "method": "marginOrderReport",
  "params": {
    "id": 262793426955,
    "clientOrderId": "b7580a84e82a4390815942e666625e51",
    "symbol": "BTCUSD",
    "side": "buy",
    "status": "canceled",
    "type": "limit",
    "timeInForce": "GTC",
    "price": "100.01",
    "quantity": "0.00001",
    "postOnly": false,
    "cumQuantity": "0",
    "createdAt": "2020-07-04T12:01:25.259Z",
    "updatedAt": "2020-07-04T12:01:25.499Z",
    "reportType": "canceled"
  }
}
```

>Response:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "id": 262793426955,
    "clientOrderId": "b7580a84e82a4390815942e666625e51",
    "symbol": "BTCUSD",
    "side": "buy",
    "status": "canceled",
    "type": "limit",
    "timeInForce": "GTC",
    "price": "100.01",
    "quantity": "0.00001",
    "postOnly": false,
    "cumQuantity": "0",
    "createdAt": "2020-07-04T12:01:25.259Z",
    "updatedAt": "2020-07-04T12:01:25.499Z",
    "reportType": "canceled"
  },
  "id": 1240
}
```

>Notification. Account report:

```json
{
  "jsonrpc": "2.0",
  "method": "marginAccountReport",
  "params": {
    "symbol": "BTCUSD",
    "leverage": "10.00",
    "marginBalance": "3.000000000000",
    "marginBalanceOrders": "0",
    "marginBalanceRequired": "0",
    "createdAt": "2020-07-04T12:01:25.003Z",
    "updatedAt": "2020-07-04T12:01:25.499Z",
    "reportType": "updated",
    "reportReason": "updated"
  }
}
```

Method: **`marginOrderCancel`**

Cancels an active margin order.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Required parameter.<br> Uniqueness must be guaranteed within a single trading day, including all active orders. |


## Cancel/Replace Margin Order

>Request:

```json
{
  "method": "marginOrderCancelReplace",
  "jsonrpc": "2.0",
  "params": {
    "quantity": "0.00001",
    "price": "100.01",
    "clientOrderId": "8198a83590da46d0ac2bc04e936bb490",
    "requestClientId": "c81bbe24569e42d282a72d250f286865"
  },
  "id": 1239
}
```

>Notification. Order report:

```json
{
  "jsonrpc": "2.0",
  "method": "marginOrderReport",
  "params": {
    "id": 262791897983,
    "clientOrderId": "c81bbe24569e42d282a72d250f286865",
    "symbol": "BTCUSD",
    "side": "buy",
    "status": "new",
    "type": "limit",
    "timeInForce": "GTC",
    "price": "100.01",
    "quantity": "0.00001",
    "postOnly": false,
    "cumQuantity": "0",
    "createdAt": "2020-07-04T11:55:36.782Z",
    "updatedAt": "2020-07-04T11:55:36.902Z",
    "reportType": "replaced",
    "originalRequestClientOrderId": "8198a83590da46d0ac2bc04e936bb490"
  }
}
```

>Response:

```json
{
  "jsonrpc": "2.0",
  "result": {
    "id": 262791897983,
    "clientOrderId": "c81bbe24569e42d282a72d250f286865",
    "symbol": "BTCUSD",
    "side": "buy",
    "status": "new",
    "type": "limit",
    "timeInForce": "GTC",
    "price": "100.01",
    "quantity": "0.00001",
    "postOnly": false,
    "cumQuantity": "0",
    "createdAt": "2020-07-04T11:55:36.782Z",
    "updatedAt": "2020-07-04T11:55:36.902Z",
    "reportType": "replaced",
    "originalRequestClientOrderId": "8198a83590da46d0ac2bc04e936bb490"
  },
  "id": 1239
}
```

>Notification. Account report:

```json
{
  "jsonrpc": "2.0",
  "method": "marginAccountReport",
  "params": {
    "symbol": "BTCUSD",
    "leverage": "10.00",
    "marginBalance": "3.000000000000",
    "marginBalanceOrders": "0.000105260525",
    "marginBalanceRequired": "0",
    "createdAt": "2020-07-04T11:55:36.489Z",
    "updatedAt": "2020-07-04T11:55:36.902Z",
    "reportType": "updated",
    "reportReason": "updated"
  }
}
```

Method **`marginOrderCancelReplace`**

Changes the parameters of an existing margin order.

Parameters:

| Name | Type | Description |
|:---|:---:|:---|
| clientOrderId | String | Required parameter.<br>Replaced order. |
| requestClientId | String | Required parameter.<br>`clientOrderId` for new order.<br>Uniqueness must be guaranteed within a single trading day, including all active orders. |
| quantity | Number | New order quantity. |
| price | Number | New order price. |
| strictValidate | Boolean | Price and quantity will be checked for the incrementation within tick size and quantity step. See symbol's `tickSize` and `quantityIncrement`. |

<% end -%>

# Errors

The HitBTC API uses the following error codes:


| Error code | 	HTTP Status Code | Message | Note |
|------------|-------------------|:--------|:-----|
| 403   | 401 | Action is forbidden for account | |
| 429   | 429 | Too many requests | Action is being rate limited for account |
| 500   | 500 | Internal Server Error | |
| 503   | 503 | Service Unavailable | Try it again later |
| 504   | 504 | Gateway Timeout | Check the result of your request later |
| 1001  | 401 | Authorisation required | |
| 1002  | 401 | Authorisation required or has been failed | Check that authorization data provided |
| 1003  | 403 | Action forbidden for this API key | Check permissions for API key |
| 1004  | 401 | Unsupported authorisation method | Use Basic authentication |
| 2001  | 400 | Symbol not found |  |
| 2002  | 400 | Currency not found |  |
| 2010  | 400 | Quantity not a valid number |  |
| 2011  | 400 | Quantity too low |  |
| 2012  | 400 | Bad quantity | More details in description |
| 2020  | 400 | Price not a valid number |  |
| 2021  | 400 | Price too low |  |
| 2022  | 400 | Bad price | More details in description |
| 20001 | 400 | Insufficient funds | Insufficient funds for creating order or any account operation |
| 20002 | 400 | Order not found | Attempt to get active order that  not existing, filled, canceled or expired. Attempt to cancel not existing order. Attempt to cancel already filled or expired order. |
| 20003 | 400 | Limit exceeded | Withdrawal limit exceeded |
| 20004 | 400 | Transaction not found | Requested transaction not found |
| 20005 | 400 | Payout not found |  |
| 20006 | 400 | Payout already committed |  |
| 20007 | 400 | Payout already rolled back |  |
| 20008 | 400 | Duplicate clientOrderId |  |
| 20009 | 400 | Price and quantity not changed |  |
| 20010 | 400 | Exchange temporary closed | Exchange market temporary closed on symbol |
| 20014 | 400 | Offchain for this payout is unavailable | Address do not belong to exchange or belongs to same user or unavailable for currency |
| 20032 | 400 | Margin account or position not found | Create margin account before place orders or account for requested symbol not found |
| 20033 | 400 | Position not changed | Existed marginBalance equal requested value |
| 20034 | 400 | Position in close only state |  |
| 20040 | 400 | Margin trading forbidden |  |
| 20080 | 400 | Internal order execution deadline exceeded | Order not placed. |
| 10001 | 400 | Validation error | Input not valid, see more in `message` field |
| 10021 | 400 | User disabled | Sub-Account API. User cant be changed |

# Guide & Code Samples

<% if read_env() == 'HIT' -%>
## API Sample Code & Libraries

 * Python rest example [example_rest.py](https://github.com/hitbtc-com/hitbtc-api/blob/master/example_rest.py)
 * Js [WebSocket API 2.0 Client](https://gist.github.com/hitbtc-com/fc738c1b926d9d7aa7e3bd5247f792a1)
 * Python [WebSocket API 2.0 Client](https://github.com/Crypto-toolbox/hitbtc)

<% end -%>
## Fetch all public trades

Fetch all trades history for a symbol and keep it updated.

General idea: the fetch trades are sorted by `ASC` from latest `id`.

```python
import requests
import time
session = requests.session()


def get_trades(symbol_code, from_id):
    return session.get("https://api.hitbtc.com/api/2/public/trades/%s?sort=ASC&by=id&from=%d&limit=1000" % (symbol_code, from_id)).json()


symbol = 'ETHDAI'
from_trade_id = 1

while True:
    trades = get_trades(symbol, from_trade_id)
    if len(trades) > 0:
        from_trade_id = trades[-1]['id'] + 1
        print('Loaded trades till', trades[-1]['id'], trades[-1]['timestamp'])
        if len(trades) < 1000:
            time.sleep(5)
    else:
        time.sleep(30)
```

## Get trades updated

**First version**

The fastest way is use the socket API. Subscribe for reports (method `subscribeReports`) and handle `reportType` == `trade`.

**Second version**

Fetch trades as in the example above but with small changes.

```python
import requests
import time
session = requests.session()
session.auth = ('3ef4a9f8c8bf04bd8f09884b98403eae', '2deb570ab58fd553a4ed3ee249fd2d51')

def get_my_trades(from_id):
    return session.get("https://api.hitbtc.com/api/2/history/trades?sort=ASC&by=id&limit=100&from=%d" % (from_id)).json()

def get_recent_my_trades():
    return session.get("https://api.hitbtc.com/api/2/history/trades?sort=DESC&by=id&limit=10").json()

from_trade_id = 0

while True:
    if from_trade_id == 0:
        recent_trades = get_recent_my_trades()
        if len(recent_trades) > 0:
            from_trade_id = recent_trades[0]['id'] + 1
            print('Last recent trade', recent_trades[0])
        else:
            print('You do not have any trades')
    else:
        trades = get_my_trades(from_trade_id)
        if len(trades) > 0:
            from_trade_id = trades[-1]['id'] + 1
            print('new trades', trades)
        else:
            print('No new trades')
    time.sleep(30)
```  

## Getting new transactions

The code in the example allows you to get all of your new deposits, withdrawals and transfers.

The **index** is a 64-bit integer, that increases on any transaction update. To get all new updates of transactions you will need to request transactions with an index value which is greater than the latest one.

For example, if you have created a withdrawal, then you get an update with a transaction hash and committed status after receiving the required number of confirmations.


```python
import requests
import time
session = requests.session()
session.auth = ('3ef4a9f8c8bf04bd8f09884b98403eae', '2deb570ab58fd553a4ed3ee249fd2d51')

def get_my_transactions(from_index):
    return session.get("https://api.hitbtc.com/api/2/account/transactions?sort=ASC&by=index&limit=100&from=%d" % (from_index)).json()

def get_recent_my_transactions():
    return session.get("https://api.hitbtc.com/api/2/account/transactions?sort=DESC&by=index&limit=100").json()

from_index = 0

while True:
    if from_index == 0:
        recent_transactions = get_recent_my_transactions()
        if len(recent_transactions) > 0:
            from_index = recent_transactions[0]['index'] + 1
            print('Last recent trade', recent_transactions[0])
        else:
            print('You do not have any transactions')
    else:
        transactions = get_my_transactions(from_index)
        if len(transactions) > 0:
            from_index = transactions[-1]['index'] + 1
            print('New and Updated transactions', transactions)
        else:
            print('No new updates')
    time.sleep(60)
```
