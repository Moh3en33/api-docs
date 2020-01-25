# WebSocket

**Endpoint URL**

**`wss://data.trade8.to`**

## Subscribe

To receive WebSocket events, you must first send a `subscribe` message with the  channels you wish to subscribe to.

```javascript
{
  "type": "subscribe",
  "channels": [
    "trading:quotes:AAPL",
    "trading:quotes:EUR-USD"
  ]
}
```

You can unsubscribe in a similar fashion, by sending an `unsubscribe`message.

```javascript
{
  "type": "unsubscribe"
}
```

## Sequence Numbers

Events received through the WebSocket contain a `sequence` number. This is an integer value that will increase by 1 for each event sent for the duration of your connection.

You can ensure that you're receiving WebSocket messages in the expected order, and without dropped messages, by keeping track of the sequence numbers.

If you observe out of order sequence numbers, or dropped sequence numbers, you may need to disconnect and reconnect.

Each WebSocket connection keeps track of sequence numbers independently, so if you have multiple WebSocket connections open, keep in mind to track sequence numbers separately.

## Rate Limits

WebSocket `subscribe` and `unsubscribe` messages are rate limited to 1 per 5 seconds per IP.

If you're being rate-limited, you'll receive a response with type `error`.

## Authentication

To receive events from authenticated channels, you must authenticate when sending the initial `subscribe` message.

Authentication is similar to REST request signing and authentication. Pass in an `auth` field in your `subscribe` message containing:

* `key` Your API key \(you can use any API key on your Trading account\)
* `sig` The signed message for the `GET /ws-auth` request, with no body. See [Signing a Request](authentication.md#signing-a-request).
* `timestamp` The current time in milliseconds since UNIX epoch

```javascript
{
  "type": "subscribe",
  "channels": [
    "trading:quotes:EUR-USD",
    "trading:quotes:BTC-USD",
    "trading:positions",
  ],
  "auth": {
    "key": "8b6bcdc6-5ef8-463b-833f-3681dc2c90d3",
    "sig": "5a1073b62e4dd62e4dd56244c89941...",
    "timestamp": 1511027890939
  }
}
```

## Channels & Events

**`heartbeat`**

Trade8 will send a `heartbeat` event every five seconds so you'll know your WebSocket connection is active. Hearbeats also include sequence numbers which you can use to check that no messages were missed.

If you miss one or more heartbeats, or your sequence numbers contain gaps, your connection may be unreliable. We recommend that you disconnect and reconnect.

## Trading

### Public

**`trading:quotes:[product]`**

The `quote` event provides real-time price updates.

```javascript
{
    "event": "quote",
    "sequence": 889,
    "data": [
        "EUR-USD", // product
        "1.12442", // mid price
        "1.12439", // bid price
        "1.12445", // ask price
        "1.12411", // mid price 24 hours ago
        151143431932 // timestamp
    ]
}
```

**`trading:candles:[product]:[resolution]`**

The `candle` event provides real-time candle updates.

```javascript
{
    "event": "candle",
    "resolution": "1",
    "product": "BTC-USD",
    "data": {
      "t": "1553623740", // timestamp in seconds
      "m": ["3916.6","3916.6","3916.5","3916.5"], // mid OHLC
      "b": ["3916.3","3916.3","3916.2","3916.2"], // bid OHLC
      "a": ["3917.0","3917.0","3916.8","3916.8"] // ask OHLC
    }
}
```

### Authenticated

For demo trading, channels and events have a `_demo` suffix. E.g. `trading:orders_demo`, `order_demo:new`, `position_demo:close`, `trading:balances_demo`, etc.

**`trading:orders`**

When subscribed to this channel, you can receive `order:new`, `order:cancel`, `order:activate`, and `order:update` events.

An **`order:new`** event indicates that your order has been submitted and received by the trading engine.

```javascript
{
    "event": "order:new",
    "sequence": 827,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "EUR-USD",
        "price": "1.26635",
        "side": "buy",
        "leverage": "300",
        "amount": 1000000,
        "margin": "3.33333333",
        "base_size": "1000",
        "base_currency": "BTC",
        "type": "market",
        "liquidation_price": "1.26354",
        "decay_rate": "0.0002",
        "created_at": 1511482279492
    }
}
```

An **`order:cancel`** event indicates that your order has been cancelled.

```javascript
{
    "event": "order:cancel",
    "sequence": 332,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "EUR-USD"
    }
}
```

An **`order:activate`** event indicates that your order is now an open position. This event is sent in parallel with **`position:new`**.

```javascript
{
    "event": "order:activate",
    "sequence": 93,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "EUR-USD"
    }
}
```

An **`order:update`** event contains information relating to an order update. An order's price, margin, leverage, take-profit, and stop-loss can be updated. The event's data filed contains the full updated order object.

```javascript
{
    "event": "order:update",
    "sequence": 998,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "EUR-USD",
        "price": "1.26635",
        "side": "buy",
        "leverage": "300",
        "amount": 1000000,
        "margin": "3.33333333",
        "base_size": "1000",
        "base_currency": "BTC",
        "type": "market",
        "liquidation_price": "1.26354",
        "take_profit": "1.28898",
        "stop_loss": "1.26477",
        "decay_rate": "0.0002",
        "created_at": 1511482279492,
        "updated_at": 1511482476135
    }
}
```

**`trading:positions`**

When subscribed to this channel, you can receive `position:new`, `position:close`, `position:split`, and `position:update` events.

A **`position:new`** event indicates that your position has been received by the trading engine and is now open.

```javascript
{
    "event": "position:new",
    "sequence": 124,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "EUR-USD",
        "price": "1.26635",
        "side": "buy",
        "leverage": "300",
        "amount": 1000000,
        "margin": "3.33333333",
        "base_size": "1000",
        "base_currency": "BTC",
        "type": "market",
        "liquidation_price": "1.26354",
        "decay_rate": "0.0002",
        "created_at": 1511482279492,
        "opened_at": 1511482876413
    }
}
```

A **`position:close`** event indicates that your position was closed. A position can be closed manually or due to a trigger such as a take-profit or a liquidation.

```javascript
{
    "event": "position:close",
    "sequence": 1434,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "EUR-USD",
        "price": "1.26635",
        "side": "buy",
        "leverage": "300",
        "amount": 1000000,
        "margin": "3.33333333",
        "base_size": "1000",
        "base_currency": "BTC",
        "type": "market",
        "liquidation_price": "1.26354",
        "decay_rate": "0.0002",
        "close_price": "1.26995",
        "pnl": "2.84281590",
        "pnl_percent": "85.2845",
        "reason": "manual",
        "decay": "0",
        "created_at": 1511482279492,
        "opened_at": 1511482876413,
        "closed_at": 1511483816212
    }
}
```

A **`position:split`** event indicates that your position was split into two positions.

```javascript
{
    "event": "position:split",
    "sequence": 398,
    "data": {
        "parent_id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "first_position": { ... }
        "second_position": { ... }
    }
}
```

An **`position:update`** event contains information relating to a position update. You can update a position's margin, leverage, take-profit, and stop-loss. The event's data filed contains the full updated position object.

```javascript
{
    "event": "position:update",
    "sequence": 135,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "EUR-USD",
        "price": "1.26635",
        "side": "buy",
        "leverage": "300",
        "amount": 1000000,
        "margin": "3.33333333",
        "base_size": "1000",
        "base_currency": "BTC",
        "type": "market",
        "liquidation_price": "1.26354",
        "take_profit": "1.28898",
        "stop_loss": "1.26477",
        "decay_rate": "0.0002",
        "created_at": 1511482279492,
        "opened_at": 1511482876413,
        "updated_at": 1511486476135
    }
}
```

**`trading:balances`**

A `balance` event is sent whenever one or more of your balances changes.

```javascript
{
    "event": "balance",
    "sequence": 233,
    "data": {
        "BTC": "33.99388733"
    }
}
```

## XT Exchange

**Important Note: the channels below cover our [XT Exchange](https://exchange.trade8.to), not our main trading platform. If you're looking for those, please see the Trading section above.**

### Public

**`exchange:book:[product]`**

A `book` event provides real-time order book updates. It contains an `updates` array that contains arrays representing price level deltas.

Each delta array contains:

* price
* size
* order count at this level. '0' means the price level was removed
* bid \(0\) or ask \(1\)

This event is batched and sent every 500 milliseconds at most.

```javascript
{
    "event": "book",
    "sequence": 991,
    "product": "XT-BTC",
    "updates": [
        ["0.04771", "1.44543", 1, 0],
        ["0.05892", "2.99112", 2, 1],
    ]
}
```

**`exchange:trades:[product]`**

A `trade` event indicates that a trade occurred on the market. `side`represents the taker side: a `buy` side match is an up-tick and a `sell` side match is a down-tick.

```javascript
{
    "event": "trade",
    "sequence": 439,
    "product": "XT-BTC",
    "order_id_taker": "7ec97c53-75b2-4e9b-bdfa-6eca818f8c8d",
    "order_id_maker": "5d818156-fc9a-4bfe-b0e3-1ef3f3f5d799",
    "size": "11.774650",
    "price": "0.04771",
    "side": "buy",
    "timestamp": 151143461939
}
```

**`exchange:quotes:[product]`**

The `quote` event provides real-time price updates.

```javascript
{
    "event": "quote",
    "sequence": 889,
    "data": [
        "XT-BTC", // product
        "0.0454", // last price
        "buy", // last tick
        "888377.1283",  // 24h volume
        "0.0355", // price 24h ago
        151143431932 // timestamp
    ]
}
```

### Authenticated

**`exchange:orders`**

When subscribed to this channel, you can receive `open`, `match`, `cancel`, and `done` events.

An **`open`** event means that your order is now visible on the book with a non-zero amount. Open orders remain on the book until they are filled or cancelled.

```javascript
{
    "event": "open",
    "sequence": 44,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "XT-BTC",
        "size": "0.77942322",
        "filled": "0.2451223",
        "price": "0.05132",
        "side": "buy",
        "type": "limit",
        "timestamp": 1511482279492
    }
}
```

A **`match`** event indicates that a trade occurred concerning one of your orders. `side` represents the taker side: a `buy` side match is an up-tick and a `sell` side match is a down-tick.

```javascript
{
    "event": "match",
    "sequence": 66,
    "data": {
        "product": "XT-BTC",
        "order_id_taker": "7ec97c53-75b2-4e9b-bdfa-6eca818f8c8d",
        "order_id_maker": "5d818156-fc9a-4bfe-b0e3-1ef3f3f5d799",
        "size": "11.774650",
        "price": "0.04771",
        "side": "buy",
        "timestamp": 1511434619393
    }
}
```

A **`cancel`** event indicates that your cancel request has been processed by the matching engine.

```javascript
{
    "event": "cancel",
    "sequence": 812,
    "data": {
       "id": "31bf3a89-2a0a-40b0-9327-ec0a5cc3ce3c",
        "product": "XT-BTC",
        "size": "1.22393981",
        "price": "0.03131",
        "side": "buy",
        "type": "limit",
        "timestamp": 1511434617391
    }
}
```

A **`done`** event indicates that your order has been fully filled and is no longer visible on the book.

```javascript
{
    "event": "done",
    "sequence": 91,
    "data": {
        "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "product": "XT-BTC",
        "price": "0.05132",
        "side": "buy",
        "timestamp": 1511434699393
    }
}
```

**`exchange:balances`**

A `balance` event is sent whenever one or more of your balances changes. If the event received contains a `diff` field, the value is an increment or decrement, not the new balance.

```javascript
{
    "event": "balance",
    "sequence": 22,
    "data": {
        "XT": "3445.66760000"
    }
}
```

