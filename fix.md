# Hitbtc FIX specification (version 1.0.5)

## Basics 

* The FIX gateway is accessible with an OpenVPN connection.
* There are two interfaces: **FIX Market data** and **FIX Trading**.
* **FIX Trading** requires a valid `SenderCompId`, `login` and `password` specified in the `Logon` message.
* The protocol is based on FIX protocol 4.4 (http://www.fixprotocol.org/FIXimate3.0). Refer to FIX 4.4 documentation if there is no tag information specified.
* The FIX gateway supports subset of messages and tags listed in this document.
* Quantity is represented in lots (e.g. 1 lot of BTCUSD is 0.01 BTC)
* Price is represented in natural value (e.g. 550$ for BTC)

### Symbols

| Symbol | Lot size | Price step |
| --- | --- | --- |
| BTCUSD | 0.01 BTC | 0.01 |
| BTCEUR | 0.01 BTC | 0.01 |
| LTCBTC2 | 0.1 LTC | 0.00001 |
| LTCUSD | 0.1 LTC | 0.001 |
| LTCEUR | 0.1 LTC | 0.001 |
| DSHBTC | 1 DSH | 0.00000001 |
| ETHBTC | 0.001 ETH | 0.000001 |
| QCNBTC | 0.01 QCN | 0.000001 |
| FCNBTC | 0.01 FCN | 0.000001 |
| LSKBTC | 1 LSK    | 0.0000001 |
| LSKEUR | 1 LSK    | 0.0001 |
| STEEMBTC | 0.001 STEEM | 0.00001 |
| STEEMEUR | 0.001 STEEM | 0.0001 |
| SBDBTC | 0.001 SBD | 0.00001 |
| DASHBTC | 0.001 DASH | 0.000001 |
| XEMBTC | 1 XEM | 0.00000001 |
| XEMEUR | 1 XEM | 0.0000001 |
| EMCBTC | 0.1 EMC | 0.00000001 |
| EMCEUR | 0.01 EMC | 0.00001 |
| SCBTC | 100 SC | 0.000000001 |
| SCUSD | 1100 SC | 0.000001 |

### Document usage

* All tags listed should be treated as required by default. Optional tags will be marked.
* Tags that are not listed are not supported
* Tags are implemented according to FIX protocol 4.4 (http://www.fixprotocol.org/FIXimate3.0). If some values are not supported the complete list of values is specified.

### End points

There are two FIX end-points that use different network addresses (ipv4:port):

* **FIX Trading** end-point: interface for trading (placing new orders, order status requests, execution reports, order cancellation)
* **FIX Market data** end-point: interface for market data subscription

Client should use proper endpoint. Behavior in case of receiving wrong message is not specified. Client could possibly be disconnected or receive Reject messages in case of using an incorrect endpoint.

### Disaster scenarios

In case of severe fault the FIX session could be forcibly reset. 
In this case Client will receive `ResetSeqNo` flag from server and should synchronize its state by **MassStatusRequest**.

### Effective time

There are restrictions on internal order placement timeouts. In case of fault Client will receive Reject with the specific reason "Too late to enter". 

### Order statuses

There are two pseudo-states that could be returned only by sending **OrderStatusRequest** or **OrderMassStatusRequest**: 

  * **Pending New** 
  * **Pending Cancel**.

Please note:

  * First **ExecutionReport** will be **New** or **Rejected**
  * Not-filled **FOK** and **IOC** orders become **Expired** 

Consider the following diagram:

![Order states](http://www.plantuml.com/plantuml/png/RP313e8m44Jl-nLxDkK7F7W1CSP4ZF56F3Hb3zM2sfO8V--A59NqjlCoCxjjkJXZagnmJqyen_b85rAUAW2c0rbtTssfmYLkYrJanGvhxHmvhAMzafzykJQ6Sq4UfFLQ6jFFU2eRHGE1cIKMuwtKaMgzlZLH__zroES9t9moWPdieu5fFqS-CrfwjEGyG7ZyOEGVWT0Uz4_FMtwxHl82 "Order states")

### Session layer
FIX session has no persistence.

## FIX Trading

Supports the following application-level messages:

From Client to FIX gateway:

  * **NewOrderSingle [type 'D']**
  * **OrderCancelRequest [type 'F']**
  * **OrderStatusRequest [type 'H']**
  * **OrderMassStatusRequest [type 'AF']**

From FIX gateway to Client: 

  * **OrderCancelReject [type '9']**
  * **ExecutionReport [type '8']**

Client will receive **ExecutionReports** for orders that are placed with the same `SenderCompID`.

## FIX Market data

Supports the following application-level messages:

From Client to FIX gateway: 

  * **MarketDataRequest [type 'V']**

From FIX gateway to Client: 

  * **MarketDataRequestReject [type 'Y']**
  * **MarketDataSnapshotFullRefresh [type 'W']**
  * **MarketDataIncrementalRefresh [type 'X']**

## Message Format

### StandardHeader

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 8 | BeginString | FIX.4.4 | |
| 9 | BodyLength | | | 
| 35 | MsgType | 0 - Heartbeat <br> 1 - Test Request <br> 2 - Resend Request <br> 5 - Logout <br> A - Logon <br> D - NewOrderSingle <br>  F - Order Cancel Request <br> H - Order Status Request <br> V - Market Data Request <br> AF - Mass Status Request <br> AN - Request For Positions | |
| 49 | SenderCompID | Should be in the **login_<number>** format; only one connection can be established with the same `SenderCompID` | |
| 56 | TargetCompID | ME | |
| 34 | MsgSeqNum | | |
| 43 | PossDupFlag | | |
| 52 | SendingTime | @Snt | | |

### StandardTrailer

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 10 | CheckSum | | | |

### Heartbeat [type '0']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 112 | TestReqID | | this Heartbeat is a response to TestRequest |

### TestRequest [type '0']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 112 | TestReqID | | | |

### ResendRequest [type '2']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 7 | BeginSeqNo| | |
| 16 | EndSeqNo| | | |

### Reject [type '3']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 45 | RefSeqNum | | |
| 371| RefTagID| | |
| 58| Text | | optional |
 	 	 	 
### SequenceReset [type '4']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 123 | GapFillFlag | N (default), Y | |
| 371 | RefTagID| | |
| 123 | GapFillFlag| | |
| 36 | NewSeqNo | | | |

### Logout [type '5']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 58 | Text| optional | could contain a reason for server-initiated logout |

### Logon [type 'A']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 98 | EncryptMethod | 0 | |
| 108 | HeartBtInt  | | |
| 553 | Username | | |
| 554 | Password | | |
|  | ResetSeqNumFlag | optional, default=N | |
| 10001 | CancelOnDisconnect | Y or N, optional, default:N  | cancel all orders for a specific SenderCompID on disconnect |

### Instrument block

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 55 | Symbol | e.g. "BTCUSD | | |

### Parties block

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 453 | 	NoPartyIDs | 1 | | |
| 448 | 	PartyID | user id | | |

### MarketDataRequestReject [type 'Y']


| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 262 | MDReqID  | | |
| 281 | MDReqRejReason | 0 | |
| 58 | Text | optional | could contain a reason |

### MarketDataSnapshotFullRefresh [type 'W']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 262 | MDReqID  | | |
| | Instrument | (component) | |
| 268 | NoMDEntries | (repeating group)	| |
| 269 | > MDEntryType | 0 - Bid <br> 1 - Offer | |
| 270 | > MDEntryPx | | |
| 271 | > MDEntrySize | | | |

### MarketDataIncrementalRefresh [type 'X']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 262 | MDReqID | | |
| 268 | NoMDEntries | (repeating group) | |
| 279 | >MDUpdateAction	| 0 - New | in case of level removal MDEntrySize will be 0 |
| 269 | >MDEntryType | 0 - Bid <br> 1 - Offer <br> 2 - Trade | |
| | > Instrument | (component) | |
| 270 | > MDEntryPx | | | 
| 271 | > MDEntrySize | | | |

### MarketDataRequest [type 'V']

Subscribe:

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 262 | MDReqID | | |
| 263 | SubscriptionRequestType | 0 - Snapshot <br> 1 - Snapshot and updates| |
| 264 | MarketDepth | 0 - Full Book <br> 1 - Top Of Book | |
| 267 | NoMDEntryTypes | (repeating group) | |
| 269 | >MDEntryType | 0 - Bid <br> 1 - Offer <br> 2 - Trade | |
| 146 | NoRelatedSym | (repeating group) | |	 
| | > Instrument | (component) | | |

Unsubscribe:

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 262 | MDReqID | | |
| 263 | SubscriptionRequestType | 2 - Disable previous Snapshot + Update Request (Unsubscribe) | | |

### ExecutionReport [type '8']


| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 37 | OrderID | (Repeating group), size = 1 | |	 
| 11 | ClOrdID | <= 32 characters | |
| 41 | OrigClOrdID | | filled for **ExecutionReport** initiated by **OrderCancelRequest** | 
| 790 | OrdStatusReqID | | ExecType=I |
| 17 | ExecID | | |
| 150 | ExecType | 0 - New <br> 4 - Canceled <br> 8 - Rejected <br> C - Expired <br> F - Trade (partial fill or fill) <br> I = Order Status | |
| 39 | OrdStatus | 0 - New <br> 1 - Partially filled <br> 2 - Filled <br> 4 - Canceled <br> 8 - Rejected <br> C - Expired <br> 6 - Pending Cancel <br> A - Pending New | Pending Cancel and Pending New could be sent if ExecType=I |
| 103 | OrdRejReason | 1 - Unknown symbol <br> 2 - Exchange closed <br> 4 - Too late to enter <br> 5 - Unknown Order <br> 6 - Duplicate Order (e.g. dupe ClOrdID (11)) <br> 11 - Unsupported order characteristic <br> 15 - Unknown account(s) <br> 99 - Other | if Rejected |
| 1 | Account | | |
| | Instrument |(component)| |
| 54 | Side | 1 - Buy <br> 2 - Sell | |
| 60 | TransactTime | | |
| 38 | OrderQty	| | |
| 40 | OrdType | 1 - Market <br> 2 - Limit | |
| 44 | Price | | for limit orders |
| 32 | LastQty | | for Trades | 
| 31 | LastPx | | for Trades | 
| 851 | LastLiquidityIn | | for Trades |
| 151 | LeavesQty | | |
| 14 | CumQty | | |
| 6 | AvgPx | | |
| 584 | MassStatusReqID | | Mass status request identifier. Present if Order_Mass_Status_Request (35=AF) is used. |
| 912 | LastRptRequested | N - There is at least one more execution report <br> Y = This is last execution report | Present if Order_Mass_Status_Request (35=AF) is used |

### NewOrderSingle [type 'D']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 11 | ClOrdID | @ID | |
| | Parties | (component), size=1 | |
| 1 | Account | | | 	 	 
| | Instrument | (component) | |
| 54  | Side | 1 - Buy <br> 2 - Sell | |
| 60 | TransactTime | | |
| 38 | OrderQty | | |
| 40 | OrdType | 1 - Market <br> 2 - Limit | |
| 44 | Price | | |
| 59 | TimeInForce | 0 - Day (or session) <br> 1 - Good Till Cancel (GTC) <br> 2 = Immediate or Cancel (IOC) <br> 4 = Fill or Kill (FOK) <br> 6 - Good Till Date (GTD) | |
| 126 | ExpireTime | UTCTimestamp | for GTD orders | 
| 18 | ExecInst	| optional, 'o' from FIX5.0 (cancel on disconnect)| |
| 10001	| CancelOnDisconnect | optional <br> Y <br> N (default) | |

### OrderCancelReject [type '9']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 37 | OrderID | If CxlRejReason="Unknown order", specifies "NONE" |
| 11 | ClOrdID 	| | |
| 41 | OrigClOrdID | | |
| 39 | OrdStatus | If CxlRejReason = "Unknown Order", specifies Rejected | |
| 434 | CxlRejResponseTo | | |
| 102 | CxlRejReason | | | |

### OrderCancelRequest [type 'F']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 41 | OrigClOrdID | | |
| 11 | ClOrdID | | | 	 	 
| | Instrument | (component) | |
| 1 | Account | | | 	 
| 54 | Side | | |
| 60 | TransactTime | | | |

### OrderStatusRequest [type 'H']

Behavior:

  * Server sends order status as **ExecutionReport**
  * **Account** and **Side** are required 
  * If order is not found the server doesn't send any response.

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 11 | ClOrdID | @ID | |
| 790 | OrdStatusReqID | @StatReqID | |	 
| 1 | Account | | |
| | Instrument | (component) | |
| 54 | Side | 1 - Buy <br> 2 - Sell | | |

### OrderMassStatusRequest [type 'AF']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 584 | MassStatusReqID	| | |
| 585 | MassStatusReqType | 7 - Status for all orders | |
| 1 | Account | (required) |  |

### RequestForPositions [type 'AN']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 710 | PosReqID | | |
| 724 | PosReqType | 0 - Positions | |
| 263 | SubscriptionRequestType	| 0 - Snapshot| |	
| 453 | NoPartyIDs | size=0 | |
| 1 | Account | in account_ format | |
| 581 | AccountType | 8 = Joint Backoffice Account (JBO) | |
| 15 | Currency	| | currency code |
| 715 | ClearingBusinessDate | YYYYMMDD	| ignored, use now date |
| 60 | TransactTime | YYYYMMDD-HH:MM:SS	| ignored, use now |

### PositionReport [type 'AP']

| Tag | Name | Values | Comments |
| ---- | ---- | ---- | ---- |
| 721 | PosMaintRptID | 0 | |
| 710 | PosReqID | | |
| 728 | PosReqResult | 0 - Valid request <br> 1 - Invalid or unsupported request | |
| 715 | ClearingBusinessDate | 19810711	| no info |
| 453 | NoPartyIDs | (repeating group) size=0 | |
| 1 | Account | | |
| 581 | AccountType | 8 = Joint Backoffice Account (JBO) | |
| 15 | Currency	| | |
| 720 | SettlPrice | 0.0 | no info |
| 731 | SettlPriceType | 1 = Final | |
| 734 | PriorSettlPrice	 | 0.0	| no info |
| 702 | NoPositions | (repeating group) size=0 | |
| 753 | NoPosAmt | (repeating group) size=1 | |
| 707 | > PosAmtType | CASH | a|
| 708 | > PosAmt | | account balance or 0 in case of error |
| 58 | Text | optional <br> "15(Currency)" <br> "1(Account)" <br> "710(PosReqID)" | contains incorrect tag in case of wrong message |
