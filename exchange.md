# Exchange

**Endpoint URL**

`https://api.wcex.com/exchange`

## Authenticated Endpoints

{% api-method method="post" host="https://api.wcex.com/exchange" path="/order/new" %}
{% api-method-summary %}
Place a New Order
{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="product" type="string" required=true %}
A valid product.
{% endapi-method-parameter %}

{% api-method-parameter name="side" type="string" required=true %}
Can be `buy` or `sell.`
{% endapi-method-parameter %}

{% api-method-parameter name="type" type="string" required=true %}
Must be `limit`.
{% endapi-method-parameter %}

{% api-method-parameter name="size" type="string" required=true %}
Amount to buy or sell.
{% endapi-method-parameter %}

{% api-method-parameter name="price" type="string" required=true %}
Price of your order.
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}
Order successfully submitted.
{% endapi-method-response-example-description %}

```javascript
{
    "id": "9d335cce-d581-45f6-8efc-bdab3d61c6e2",
    "product": "XT-BTC",
    "type": "limit",
    "side": "buy",
    "size": "10.00000000",
    "price": "0.04",
    "filled": "2.23000000",
    "timestamp": 1511467005839
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

Currently, only limit orders are supported. To have your order execute immediately \(as a market order\), simply set a price deep in the opposite book.

### Order Lifecycle

Your account must have sufficient balance to successfully submit an order.

If any part of the order results in adding liquidity to the book, it will be marked as **open**.

An order that is completely filled and off the book will be marked as **done**.

### Response

An order accepted by the matching engine will be assigned an order `id`which will appear in the response.

`filled` specifies the amount that was matched as a result of your order.

`timestamp` is the time your order was received by the matching engine.

{% api-method method="post" host="https://api.wcex.com/exchange" path="/order/cancel" %}
{% api-method-summary %}
Cancel Order
{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="id" type="string" required=true %}
The ID of the order to cancel.
{% endapi-method-parameter %}

{% api-method-parameter name="product" type="string" required=false %}
The order's product.
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "id": "9d335cce-d581-45f6-8efc-bdab3d61c6e2"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

Providing `product` is optional, but it's recommended because it greatly speeds up the cancel operation by skipping the id-to-product lookup.

### Response

If successful, the response will contain the ID of the cancelled order.

If an order could not be cancelled \(if it has been already filled, does not exist, etc.\), an error is returned.

{% api-method method="get" host="https://api.wcex.com/exchange" path="/orders" %}
{% api-method-summary %}
Get Orders
{% endapi-method-summary %}

{% api-method-description %}
Lists your open orders, sorted by submission time \(most recent first\).
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-query-parameters %}
{% api-method-parameter name="limit" type="number" required=false %}
Number of orders to return \(max is 100\).
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
[
    {
        "id": "b20265e7-8d28-476a-8758-c6113ad98c0b",
        "product": "XT-BTC",
        "size": "2.34977543",
        "filled": "0.00000000",
        "price": "0.053435",
        "side": "sell",
        "type": "limit",
        "timestamp": 1511252206104
    },
    {
        "id": "b361e636-46cd-4a34-a532-d454c9e7b4b6",
        "product": "XT-BTC",
        "size": "4.52120565",
        "filled": "0.00000000",
        "price": "0.053147",
        "side": "buy",
        "type": "limit",
        "timestamp": 1511252184729
    }
]
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

You can optionally pass a `limit` query parameter to limit the number of orders returned. By default, 100 items are returned.

E.g. `/orders?limit=50`.

### Keeping track of your orders

We recommend listening to streaming WebSocket order events to maintain an up-to-date view of your open orders. It's faster and in many cases more accurate than polling this endpoint, because an order's state may change between the time you make an HTTP request and receive a response.

Polling this endpoint is highly discouraged.

### Response

An array of your open orders.

{% api-method method="get" host="https://api.wcex.com/exchange" path="/fills/:product?" %}
{% api-method-summary %}
Get Fills
{% endapi-method-summary %}

{% api-method-description %}
List your recent fills, ordered by fill time \(most recent first\).
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="product" type="string" required=false %}
Limit fills returned to this product.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-query-parameters %}
{% api-method-parameter name="timestamp" type="number" required=false %}
Return fills that occurred before this time.
{% endapi-method-parameter %}

{% api-method-parameter name="limit" type="number" required=false %}
Number of fills to return \(max is 100\).
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
[
    { 
        "order_id_taker": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "order_id_maker": "31bf3a89-2a0a-40b0-9327-ec0a5cc3ce3c",
        "product": "XT-BTC",
        "size": "0.77942322",
        "price": "0.05132",
        "side": "buy",
        "timestamp": 1511482279491
    },
    {
        "order_id_taker": "c6023aca-333a-44d9-98fe-279e279f3a86",
        "order_id_maker": "7f66a16c-0b47-4bd6-8add-6b2c33366082",
        "product": "XT-BTC",
        "size": "0.97422208",
        "price": "0.051323",
        "side": "buy",
        "timestamp": 1511482127568
    }
]
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

Omitting `product` will return fills across all products.

E.g. `/fills?limit=50&timestamp=1511481127561`

### Keeping track of your fills

We recommend listening to streaming WebSocket match events to maintain an up-to-date view of your fills. It's faster and more accurate than polling this endpoint.

Polling this endpoint is highly discouraged.

### Response

An array of your recent fills.

{% api-method method="get" host="https://api.wcex.com/exchange" path="/balances/:asset?" %}
{% api-method-summary %}
Get Balances
{% endapi-method-summary %}

{% api-method-description %}
Lists your balances.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="asset" type="string" required=false %}
Limits balances returned to this asset.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "BTC": "11.88900211",
    "XT": "2218.99900000"
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

Omitting `asset` will return all balances on your account. Only non-zero balances, or assets for which you have made at least one deposit will be returned.

{% api-method method="get" host="https://api.wcex.com/exchange" path="/transactions/:type" %}
{% api-method-summary %}
Get Transactions
{% endapi-method-summary %}

{% api-method-description %}
Lists your deposits and withdrawals, sorted by most recent first.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="type" type="string" required=false %}
Can be `deposit` or `withdraw`.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-query-parameters %}
{% api-method-parameter name="asset" type="string" required=false %}
Limits transactions returned to this asset.
{% endapi-method-parameter %}

{% api-method-parameter name="limit" type="string" required=false %}
Number of transactions to return \(max 100\).
{% endapi-method-parameter %}

{% api-method-parameter name="timestamp" type="string" required=false %}
Return transactions that occurred before this time.
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

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
    },
    {
        "asset": "XT",
        "amount": "200.00990000",
        "type": "withdraw",
        "address": "0xa93fae849449d2f9ddca15b861fcb329d39ad47c",
        "txid": "0xaaec962c014ab72eec8d4abb58326dca1c5c8a9425cd851ef8b9c390f2ad050e",
        "timestamp": 1511482279491,
        "confirmed_at": 1511482379491
    }
]
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

Omitting type will return all transactions on your account.

E.g. `/transactions?asset=XT&limit=50&timestamp=1511481127561`

{% api-method method="get" host="https://api.wcex.com/exchange" path="/deposit/:asset" %}
{% api-method-summary %}
Get Deposit Address
{% endapi-method-summary %}

{% api-method-description %}
Returns a deposit address for the provided asset.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="asset" type="string" required=true %}
A valid asset.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "address": "2MzeJNyp47Cugv85aq361AwBmaC4kxHofuu",
    "asset": "BTC",
    "confirmations_needed": 2
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

## Public Endpoints

{% api-method method="get" host="https://api.wcex.com/exchange" path="/products/:product?" %}
{% api-method-summary %}
Get Products
{% endapi-method-summary %}

{% api-method-description %}
Lists products available to trade.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="product" type="string" required=false %}
Limits results returned to this products.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
[
    [
        "XT-BTC", // product
        "0.001", // base tick
        "0.00001", // quote tick
        "0.01" // minimum size
    ]
]
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="https://api.wcex.com/exchange" path="/quotes/:product\(s\)" %}
{% api-method-summary %}
Get Quotes
{% endapi-method-summary %}

{% api-method-description %}
Returns quotes for the provided products\(s\).
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="product\(s\)" type="string" required=true %}
One or more comma-separated products.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
        "XT-BTC": [
                "0.0400", // last price
                "buy", // last tick
                "1223322.9983", // 24h volume
                "0.0344", // price 24 hours ago
                1534104719613 // timestamp
        ]
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

E.g. `/quotes/XT-BTC,XT-ETH`

Polling this endpoint is highly discouraged in favor of subscribing to WebSocket events.

{% api-method method="get" host="https://api.wcex.com/exchange" path="/book/:product" %}
{% api-method-summary %}
Get Book
{% endapi-method-summary %}

{% api-method-description %}
Returns list of open bids and asks for a product.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="product" type="string" required=true %}
A valid product.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-query-parameters %}
{% api-method-parameter name="limit" type="string" required=false %}
Number of levels to return \(max 100\).
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "bids":[
        ["0.04", "5.00000000", 1], // price, size, order count
        ["0.039465", "4.24022501", 1]
    ],
    "asks":[
        ["0.05132", "9.22057678", 1],
        ["0.051323", "3.52460275", 1]
    ]
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

E.g. `/book/XT-BTC?limit=50`.

Polling this endpoint is discouraged in favor of subscribing to WebSocket events.

{% api-method method="get" host="https://api.wcex.com/exchange" path="/trades/:product" %}
{% api-method-summary %}
Get Trade History
{% endapi-method-summary %}

{% api-method-description %}
Lists the latest trades for the provided product.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="product" type="string" required=true %}
A valid product.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-query-parameters %}
{% api-method-parameter name="timestamp" type="number" required=false %}
Return trades that occurred before this time,
{% endapi-method-parameter %}

{% api-method-parameter name="limit" type="string" required=false %}
Number of trades to return \(max 100\).
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
[
    {
        "product": "XT-BTC",
        "order_id_taker": "bf2b704c-010a-48ca-93fb-d0193f24420a",
        "order_id_maker": "31bf3a89-2a0a-40b0-9327-ec0a5cc3ce3c",
        "size": "1.77942322",
        "price": "0.05132",
        "side": "buy",
        "timestamp": 1511482279492
    },
    {
        "product": "XT-BTC",
        "order_id_taker": "c6023aca-333a-44d9-98fe-279e279f3a86",
        "order_id_maker": "7f66a16c-0b47-4bd6-8add-6b2c33366082",
        "size": "3.97422208",
        "price": "0.051323",
        "side": "buy",
        "timestamp": 1511482127569
    }
]
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

E.g. `/trades/XT-BTC?limit=50&timestamp=1511481127561`.

Polling this endpoint is discouraged in favor of subscribing to WebSocket events.

### Response

`side` indicates the taker order side. A `buy` indicates an up-tick and a `sell` indicates a down-tick.

{% api-method method="get" host="https://api.wcex.com/exchange" path="/candles/:product/:resolution" %}
{% api-method-summary %}
Get Candles
{% endapi-method-summary %}

{% api-method-description %}
Lists historical candles for a product. Candles returned are grouped by `resolution`.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="product" type="string" required=true %}
A valid product.
{% endapi-method-parameter %}

{% api-method-parameter name="resolution" type="string" required=true %}
Can be 1, 5, 15, 60, 240, or 1D.
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}

{% api-method-query-parameters %}
{% api-method-parameter name="start" type="number" required=false %}
The time after which to fetch candles, in ms.
{% endapi-method-parameter %}

{% api-method-parameter name="end" type="number" required=false %}
The time before which to fetch candles, in ms.
{% endapi-method-parameter %}

{% api-method-parameter name="limit" type="number" required=false %}
Number of candles to return \(max 150\).
{% endapi-method-parameter %}
{% endapi-method-query-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
[
    [
        "1511481600",
        "0.051323",
        "0.051323",
        "0.05132",
        "0.05132",
        "1009.753645"
    ],
    [
        "1511467200",
        "0.051323",
        "0.051323",
        "0.051323",
        "0.051323",
        "3201.906964"
    ],
    [
        "1511251200",
        "0.044683",
        "0.053744",
        "0.042048",
        "0.045321",
        "3310.527428"
    ]
]
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

E.g. `/candles/XT-BTC/60?start=1511480127561&end=1511480129513` fetches 1h candles on XT/BTC between the start and end timestamps.

### Response

Each item in the array returned represents a candle with granularity `resolution`:

* **timestamp** \(in seconds\)
* **open** \(first trade in this interval\)
* **high** \(highest trade in this interval\)
* **low** \(lowest trade in this interval\)
* **close** \(last trade in this interval\)
* **volume** \(volume of trading during this interval\)

`volume` is returned as string to preserve float number precision. If the candle is active, `close` is the last price.

