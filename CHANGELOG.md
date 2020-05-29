### 3.0.0 (30 may 2020)
 - separated WebSocket endpoint for high performance trading

### 2.4.0 (8 november 2018)
 - provide post-only orders option (If your post-only order would cause a match with a pre-existing order as a taker, then order will be canceled. Useful for market makers)

### 2.3.0 (25 september 2018)
 - provide get candles history by time period

### 2.2.1 (5 april 2018)
 - add deposit address into account transaction history

### 2.2.0 (22 match 2018)
 - add rate limiting
 
### 2.1.2 (19 february 2018)
 - add default withdraw fee for currency
 - add delisted attribute for currency

### 2.1.1 (8 february 2018) 
 - fix timeout on place stopLimit order
 - change default TimeInForce for market orders for `IOK`, for limit orders default `GTC`
 - fix on history orders: filter by symbol, add `price` on a response  
 - allow set symbol in query parameter for cancel orders
 - fix bug with default filter by the timestamp on public trades
  
### 2.1.0
 - Release SocketAPIv2

### 2.0.0
 - Release APIv2
