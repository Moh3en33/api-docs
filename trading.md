# Trading

**Base Endpoint URL**

**`https://api.trade8.to/trading`**

## Authenticated Endpoints

### Place a New Order

**`https://api.trade8.to/trading/trade/new`**

#### Request (POST)

| Body Parameter | Type | Description |
| :--- | :--- | :--- |
| product | `string` | A valid product. |
| side | `string` | Can be `buy` or `sell`. |
| type | `string` | Can be `market` or `pending`. |
| leverage | `number` | Leverage to use. |
| amount | `integer` | Amount in contracts (1 contract = 0.001 BTC). |
| base_currency | `string` | Must be `BTC`. |
| price (optional) | `string` | Required only if type is pending. |
| take_profit (optional) | `string` | The trade's take-profit price. |
| stop_loss (optional) | `string` | The trade's stop-loss price. |

A trade with type `market` will result in an open position.

A trade with type `pending` will result in a pending order and behaves as either a limit or stop order depending on its price \(it works in both directions\).

For demo trading, add `?demo=true` to your request.

#### Response

A trade accepted by the engine will be assigned a trade `id` which will appear in the response.

```javascript
{
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
```

### Cancel Order

**`https://api.trade8.to/trading/trade/cancel`**

#### Request (POST)

| Body Parameter | Type | Description |
| :--- | :--- | :--- |
| id | `string` | The ID of the order to cancel. |

For demo trading, add `?demo=true` to your request.

#### Response

If successful, the response will contain the ID of the cancelled order.

If an order could not be cancelled \(if it has been already filled, does not exist, etc.\), an error is returned.

```javascript
{
    "id": "9d335cce-d581-45f6-8efc-bdab3d61c6e2"
}
```

### Update Order

**`https://api.trade8.to/trading/trade/update`**

Updates an order or position.

#### Request (POST)

| Body Parameter | Type | Description |
| :--- | :--- | :--- |
| id | `string` | The ID of the order to update. |
| price (optional) | `number` | New price of the order. |
| take_profit (optional) | `number` | New take-profit price for the order. |
| stop_loss (optional) | `number` | New stop-loss price for the order. |

For pending orders, `price`, `take_profit`, and `stop_loss` can be updated.

For open positions, `take_profit` and `stop_loss` can be updated.

To remove a `take_profit` or `stop_loss`, simply set it to `0`.

For demo trading, add `?demo=true` to your request.

#### Response

```javascript
{
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
```

### Close Position

**`https://api.trade8.to/trading/trade/close`**

Closes a position at market price. Sells close at the ask, buys at the bid.

#### Request (POST)

| Body Parameter | Type | Description |
| :--- | :--- | :--- |
| id | `string` | The ID of the position to close. |

For demo trading, add `?demo=true` to your request.

#### Response

```javascript
{
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
```

### Split Position

**`https://api.trade8.to/trading/trade/split`**

Splits a position into two positions that can be managed separately.

#### Request (POST)

| Body Parameter | Type | Description |
| :--- | :--- | :--- |
| id | `string` | The ID of the position to close. |
| ratio | `string` | A ratio of percents. E.g. `40:60`. |

For demo trading, add `?demo=true` to your request.

#### Response

```javascript
{
	success: true,
	data: {
		parent_id: "bf2b704c-010a-48ca-93fb-d0193f24420a",
		first_position: {...},
		second_position: {...}
	}
}
```

### Get Orders

**`https://api.trade8.to/trading/orders`**

Lists your pending orders, sorted by submission time (most recent first).

#### Request

| Query Parameter | Type | Description |
| :--- | :--- | :--- |
| limit (optional) | `integer` | Number of orders to return (max is 100). |

You can optionally pass a `limit` query parameter to limit the number of orders returned. By default, 100 items are returned.

E.g. `/orders?limit=50`.

For demo trading, add `?demo=true` to your request.

#### Keeping track of your orders

We recommend listening to streaming WebSocket order events to maintain an up-to-date view of your open orders. It's faster and in many cases more accurate than polling this endpoint, because an order's state may change between the time you make an HTTP request and receive a response.

Polling this endpoint is highly discouraged.

#### Response

An array of your pending orders.

```javascript
[
    {
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
]
```

### Get Positions

**`https://api.trade8.to/trading/positions`**

Lists your open positions, sorted by opening time.

#### Request

| Query Parameter | Type | Description |
| :--- | :--- | :--- |
| limit (optional) | `integer` | Number of positions to return (max is 100). |

For demo trading, add `?demo=true` to your request.

#### Response

```javascript
[
    {
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
        "opened_at": 1511483479492
    }
]
```

### Get History

**`https://api.trade8.to/trading/history/:product?`**

List your recent trades (closed positions), ordered by close time.

#### Request

| Path Parameter | Type | Description |
| :--- | :--- | :--- |
| product (optional) | `string` | Limit trades returned to this product. |

| Query Parameter | Type | Description |
| :--- | :--- | :--- |
| timestamp (optional) | `integer` | Return trades that occurred before this time. |
| limit (optional) | `integer` | Number of trades to return (max is 100). |

Omitting `product` will return trades across all products.

E.g. `/history?limit=50&timestamp=1511481127561`

For demo trading, add `?demo=true` to your request.

#### Keeping track of your trades

We recommend listening to streaming WebSocket match events to maintain an up-to-date view of your closed positions. It's faster and more accurate than polling this endpoint.

Polling this endpoint is highly discouraged.

#### Response

An array of your recent trades.

```javascript
[
    {
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
        "opened_at" 1511483479492,
        "closed_at" 1511483979492
    }
]
```

### Get Balances

**`https://api.trade8.to/trading/balances/:asset?`**

Lists your balances.

#### Request

| Path Parameter | Type | Description |
| :--- | :--- | :--- |
| asset (optional) | `string` | Limit balances returned to this asset. |

Omitting `asset` will return all balances on your account. Only non-zero balances, or assets for which you have made at least one deposit will be returned.

For demo trading, add `?demo=true` to your request.

#### Response

```javascript
{
    "BTC": "11.88900211"
}
```

### Get Transactions

**`https://api.trade8.to/trading/transactions/:type`**

Lists your deposits and withdrawals, sorted by most recent first.

#### Request

| Path Parameter | Type | Description |
| :--- | :--- | :--- |
| type (optional) | `string` | Can be `deposit` or `withdraw`. |

| Query Parameter | Type | Description |
| :--- | :--- | :--- |
| asset (optional) | `string` | Limits transactions returned to this asset. |
| limit (optional) | `string` | Number of transactions to return (max 100). |
| timestamp (optional) | `string` | Return transactions that occurred before this time. |

Omitting type will return all transactions on your account.

E.g. `/transactions?asset=BTC&limit=50&timestamp=1511481127561`

#### Response

```javascript
[
    {
        "asset": "BTC",
        "amount": "0.2000000",
        "type": "deposit",
        "address": "2MzeJNyp47Cugv85aq361AwBmaC4kxHofuu",
        "txid": "8a0a2cd49b3bbcabc5a4066783e3aad47ea745a849450a865977b668ce514ec2",
        "timestamp": 1511482279491,
        "confirmed_at": 1511482379491
    }
]
```

### Get Deposit Address

**`https://api.trade8.to/trading/deposit/:asset`**

Returns a deposit address for the provided asset.

#### Request

| Path Parameter | Type | Description |
| :--- | :--- | :--- |
| asset | `string` | A valid asset. |

#### Response

```javascript
{
    "address": "2MzeJNyp47Cugv85aq361AwBmaC4kxHofuu",
    "asset": "BTC",
    "confirmations_needed": 1
}
```

## Public Endpoints

### Get Products

**`https://api.trade8.to/trading/products/:product?`**

Lists products available to trade.

#### Request

| Path Parameter | Type | Description |
| :--- | :--- | :--- |
| product (optional) | `string` | Limits results returned to this product. |

#### Response

```javascript
{
    "forex": [
        [
            "EUR-USD", // product
            "300", // max leverage
            "0.00001", // tick
            "-0.0001", // decay rate
            {
                "BTC": "600000" // max size
            }
            "EUR/USD" // name
        ],
    ...
}
```

### Get Quotes

**`https://api.trade8.to/trading/quotes/:product(s)`**

Returns quotes for the provided product(s).

#### Request

| Path Parameter | Type | Description |
| :--- | :--- | :--- |
| products | `string` | One or more comma-separated products. |

E.g. `/quotes/EUR-USD,AAPL,SNAP,ETH-USD`

Polling this endpoint is highly discouraged in favor of subscribing to WebSocket events.

E.g. `/book/XT-BTC?limit=50`.

Polling this endpoint is discouraged in favor of subscribing to WebSocket events.

#### Response

```javascript
{
        "EUR-USD": [
                "1.12442", // mid price
                "1.12439", // bid price
                "1.12445", // ask price
                151143431932 // timestamp
        ],
        ...
}
```

### Get Candles

**`https://api.trade8.to/trading/candles/:product/:resolution`**

Lists historical candles for a product. Candles returned are grouped by `resolution`.

#### Request

| Path Parameter | Type | Description |
| :--- | :--- | :--- |
| product | `string` | A valid product. |
| resolution | `string` | Can be 1, 5, 15, 60, 240, or 1D. |

| Query Parameter | Type | Description |
| :--- | :--- | :--- |
| signal (optional) | `string` | Can be `mid`, `bid`, or `ask`. Defaults to `mid`. |
| start (optional) | `number` | The time after which to fetch candles, in seconds. |
| end (optional) | `number` | The time before which to fetch candles, in seconds. |
| limit (optional) | `string` | Number of candles to return (max 150). |

E.g. `/candles/EUR-USD/60?signal=bid&start=1511480127&end=1511480129` fetches 1h bid candles on EUR/BTC between the start and end timestamps.

#### Response

Each item in the array returned represents a candle with granularity `resolution`:

* **timestamp** \(in seconds\)
* **open** \(first trade in this interval\)
* **high** \(highest trade in this interval\)
* **low** \(lowest trade in this interval\)
* **close** \(last trade in this interval\)

```javascript
[
    [
        "1511481600",
        "0.051323",
        "0.051323",
        "0.05132",
        "0.05132"
    ],
    [
        "1511467200",
        "0.051323",
        "0.051323",
        "0.051323",
        "0.051323"
    ],
    [
        "1511251200",
        "0.044683",
        "0.053744",
        "0.042048",
        "0.045321"
    ]
]
```
