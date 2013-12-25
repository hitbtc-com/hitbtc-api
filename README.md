hitbtc API - streaming market data and trading 
================================

### General considerations

Streaming API is based on [WebSocket protocol](http://en.wikipedia.org/wiki/WebSocket).
All messages are encoded as JSON.

The following symbols are traded on hitbtc exchange.

| Symbol	| Lot size | Price step
| --- | --- | --- |
| BTCUSD |	0.01 BTC |	0.01 |
| BTCEUR |	0.01 BTC |	0.01 |
| EURUSD | 1 EUR |	0.0001 |

Size values in messages are represented in lots.

### Market data end-point

URL: [ws://api.hitbtc.com](ws://api.hitbtc.com)

Once client connects to this URL the session is started. 

The server broadcasts the following types of messages:

* [MarketDataSnapshotFullRefresh](#MarketDataSnapshotFullRefresh) message contains a full snapshot of the order book.
* [MarketDataIncrementalRefresh](#MarketDataIncrementalRefresh) message contains incremental changes

Some recommendations to consider:

* The application could receive the first snapshot and maintain the order book by applying incremental updates
* It's recommended to invalidate a state of the application periodically using snapshots
* It's recommended to check sequence numbers and to drop updates with non-increasing sequence numbers

<a name="MarketDataSnapshotFullRefresh"/>
##### MarketDataSnapshotFullRefresh

MarketDataSnapshotFullRefresh message contains a full snapshot of the order book.

Example message:
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

| Field | Description |
| --- | --- |
| seqNo | monotone increasing number, each symbol has an own sequencemonotone |
| timestamp | millisecond timestamp UTC |
| symbol | |
| exchangeStatus | `on` or `off`, `off` means the trading is suspended |
| ask, bid | sorted arrays of price levels in the order book; full snapshot (all price levels) is provided |

<a name="MarketDataIncrementalRefresh"/>
##### MarketDataIncrementalRefresh

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

| Field | Description |
| --- | --- |
| seqNo	| monotone increasing number, each symbol has an own sequence
| timestamp |	millisecond timestamp UTC |
| symbol	| |
| exchangeStatus |  `on` or `off`, `off` means the trading is suspended |
| ask, bid, trade | an array of changes in the order book; <br> `size` means new size, `size`=0 means price level has been removed |

### Trading end-point

Trading endpoint requires login and all messages from client should be signed.

Messages:

| Type | Side |
| --- | --- | --- |
| [Login](#Login) | Client -> Server |
| [NewOrder](#NewOrder) | Client -> Server |
| [OrderCancel](#OrderCancel) | Client -> Server  |
| [ExecutionReport](#ExecutionReport) | Server -> Client |
| [CancelReject](#CancelReject) | Server -> Client |

##### API keys and message signatures
 in the following manner:

```json
{
    "apikey": "23857823952452",
    "signature": "83578345789348578934567",
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
| nonce | should monotonous in the same connection |
| signature | hmac-sha512(binary representation of the message) |

<a name="Login"/>
##### Login 

Example:
```json
{
	"apikey": "23857823952452",
	"signature": "83578345789348578934567",
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
##### NewOrder

Parameters:

| Parameter	| Description | Type / Enum |
| --- | --- | --- |
| clientOrderId | should be unique | |
| symbol | | |
| side | order side | `buy`, `sell` |
| quantity | quantity in lots | integer |
| type | order type	| only `limit` orders are currently supported |
| price	| price (in currency) | decimal, consider price steps |
| timeInForce | time in force | `GTC` - Good-Til-Canceled <br>`IOK` - Immediate-Or-Cancel<br>`FOK` - Fill-Or-Kill |

<a name="OrderCancel"/>
##### OrderCancel

TBD

<a name="ExecutionReport"/>
##### ExecutionReport

TBD

<a name="CancelReject"/>
##### CancelReject

TBD

### Useful tools

Chrome extension Simple WebSocket Client (https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo)

