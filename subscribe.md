# WebSocket

**Endpoint URL**

**`wss://data.wcex.com`**

## Subscribe

To receive WebSocket events, you must first send a `subscribe` message with the  channels you wish to subscribe to.

```javascript
{
  "type": "subscribe",
  "channels": [
    "exchange:quotes:XT-BTC",
    "exchange:book:XT-BTC",
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

* `key` Your API key
* `sig` The signed message for the `GET /ws-auth` request, with no body. See [Signing a Request](authentication.md#signing-a-request).
* `timestamp` The current time in milliseconds since UNIX epoch

```javascript
{
  "type": "subscribe",
  "channels": [
    "trading:quotes:EUR-USD",
    "trading:positions",
    "exchange:orders"
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

WCX will send a `heartbeat` event every five seconds so you'll know your WebSocket connection is active. Hearbeats also include sequence numbers which you can use to check that no messages were missed.

If you miss one or more heartbeats, or your sequence numbers contain gaps, your connection may be unreliable. We recommend that you disconnect and reconnect.

### Exchange

#### Public

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

#### Authenticated

**`exchange:orders`**

When subscribed to this channel, you can receive `open`, `match`, `cancel`, and `done` events.

An **`open`** event means that your order is now visible on the book with a non-zero amount. Open orders remain on the book until they are filled or cancelled.

```javascript
{
    "event": "open",
    "sequence": 44,
    "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
    "product": "XT-BTC",
    "size": "0.77942322",
    "filled": "0.2451223",
    "price": "0.05132",
    "side": "buy",
    "type": "limit",
    "timestamp": 1511482279492
}
```

A **`match`** event indicates that a trade occurred concerning one of your orders. `side` represents the taker side: a `buy` side match is an up-tick and a `sell` side match is a down-tick.

```javascript
{
    "event": "match",
    "sequence": 66,
    "product": "XT-BTC",
    "order_id_taker": "7ec97c53-75b2-4e9b-bdfa-6eca818f8c8d",
    "order_id_maker": "5d818156-fc9a-4bfe-b0e3-1ef3f3f5d799",
    "size": "11.774650",
    "price": "0.04771",
    "side": "buy",
    "timestamp": 1511434619393
}
```

A **`cancel`** event indicates that your cancel request has been processed by the matching engine.

```javascript
{
    "event": "cancel",
    "sequence": 812,
    "id": "31bf3a89-2a0a-40b0-9327-ec0a5cc3ce3c",
    "product": "XT-BTC",
    "size": "1.22393981",
    "price": "0.03131",
    "side": "buy",
    "type": "limit",
    "timestamp": 1511434617391
}
```

A **`done`** event indicates that your order has been fully filled and is no longer visible on the book.

```javascript
{
    "event": "done",
    "sequence": 91,
    "id": "bf2b704c-010a-48ca-93fb-d0193f24420a",
    "product": "XT-BTC",
    "price": "0.05132",
    "side": "buy",
    "timestamp": 1511434699393
}
```

**`exchange:balances`**

A `balance` event is sent whenever one or more of your balances changes. If the event received contains a `diff` field, the value is an increment or decrement, not the new balance.

```javascript
{
    "event": "balance",
    "sequence": 22,
    "data": {
        "XT-BTC": "3445.66760000"
    }
}
```

### Trading

#### Public

* trading:candles:\[product\]:\[resolution\]
* trading:quotes:\[product\]

#### Authenticated

* trading:orders
* trading:positions
* trading:balances

