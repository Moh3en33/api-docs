# Authentication

## Getting your API Key

You can generate an API key and secret from your [Account Settings](https://trade8.to/settings). Use them to sign requests to authenticated endpoints.

## Signing a Request

Requests to private REST API endpoints must contain these headers:

* `WCX-APIKEY` Your API key
* `WCX-SIG` Your request signature \(see below\)
* `WCX-TIMESTAMP` A current timestamp, in milliseconds since UNIX epoch

To sign a request, first create the message to sign, which is the concatenated string:

`timestamp + method + path + body`

where:

* `timestamp` is the millisecond timestamp, the same one you pass to the `WCX-TIMESTAMP` header
* `method` is the UPPER CASE HTTP method used to make the request \(`GET`, `POST`, `PUT`, or `DELETE`\)
* `path` is the request path, e.g. `/trade/new` or `/trade/new?demo=true` \(omit the `/trading` or `/exchange`\)
* `body` is the JSON-stringified body sent in the request. This generally applies to `POST` requests and can be omitted for requests without a body.

Generate a SHA-256 HMAC of this string using your API secret, then Base64-encode the output to get `WCX-SIG`.

```javascript
import crypto from 'crypto'
import request from 'request'

// Sending a New Trade request

const API_SECRET = "6a0ef...";

const TIMESTAMP = Date.now();

const body_json = {
	product: "EUR-USD",
	side: "buy",
	type: "market",
	leverage: 10,
	amount: 50000,
	base_currency: "BTC"
};

const body = JSON.stringify(body_json);

const METHOD = "POST";

const message_to_sign = TIMESTAMP + METHOD + "/trade/new" + body;

const hmac = crypto.createHmac('sha256', API_SECRET);

const WCX_SIG = hmac.update(message_to_sign).digest('base64');

request({
	method: METHOD,
	url: "https://api.trade8.to/trading/trade/new",
	json: body_json,
	headers: {
		"WCX-APIKEY": "de53e16e-...",
		"WCX-TIMESTAMP": TIMESTAMP,
		"WCX-SIG": WCX_SIG
	}
}, function(err, res, body) {
	console.log('Result:', err, body);
});

```

### Timestamp

`WCX-TIMESTAMP` is the current timestamp, in milliseconds since UNIX epoch. Providing it helps prevent man-in-the-middle attacks and provides request replay protection.

It has to be within 30 seconds of server time for the request to be valid. You can check the server time using the [/time](./#get-server-time) endpoint.

