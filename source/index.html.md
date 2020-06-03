---
title: Tauros API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - python
  - javascript
  - php

toc_footers:
  - <a href='https://tauros.io/signup'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Tauros provides a simple and practical REST API to help you to automatically perform nearly all actions you can do in the [Tauros](https://tauros.io) web app.

## General considerations

Before to start developing on the Tauros API consider the following:

* Create and account in the staging Tauros web site `https://staging.tauros.io. Staging environment uses testnet coins.
* Enable the Developer Mode in your profile section to create an API Key and a API Secret.
* The API base url is `https://api.staging.tauros.io/api/` for staging environment and `https://api.tauros.io/api/` for production.
* All private enpoints requests require message signing.
* Check the success`flag to ensure that your API call succeeded.
* All requests use the `application/json` content type and go over `https`.
* If something goes wrong look at the `msg` field. There you will find the error description.

## HTTP API Responses

Tauros REST API calls will return a JSON Object.

* A typical successful API call will response a JSON  object that looks like:

`{
  "success": true,
  "msg": null,
  "payload": {
    RELATED_DATA_HERE
    }
  }`

* An unsuccessful API call will response a JSON  object that looks like:

`{
  "success": false,
  "msg": ERROR_DESCRIPTION
}`

# Authentication

## Create an API Key

In order to use our platform through API calls you must first enable `Developer Mode` in your profile section.
Once enabled, you can create and configure as many API Keys as you need. API keys can be configurated with its own level of permissions.


## API Key Permissions

You can restrict the functionality of API keys. Before creating a key, you must choose what permissions you would like the key to have. The permissions are:

* `trading` - Allows a key perform trading operations (read, place or delete orders).
* `withdraw` - Allows a key to withdraw funds (**Enable with caution**).
* `reading` - Allows a key to read account data.
* `check_ips` - Designates wheter the client IP must be validated.
* `ips` - List of trusted IP addresses (by default is empty). Checking is done only if `check_ips` permission is enabled.
* `enabled` - Designates wheter the key is enabled to be used.

## Creating a Request

All private requests must include the following headers:

* `'Authorization: Bearer API_KEY'` - Authorization header that includes the API Key.
* `'Taur-Nonce: NONCE'` - A number that uniquely identifies each call to the API (see What is a nonce?).
* `'Taur-Signature: SIGNATURE'`- The base64-encoded signature (see Signing a Message)

All request bodies should have content type `application/json` and be valid JSON.

## What is a nonce?

A nonce is a number that uniquely identifies each API request. A nonce is required for all requests to the private endpoints. It must be included in the `Taur-Nonce` header.

Our nonce is implemented as a counter that must be unique and must increase with each call to the API. For example, assuming a starting nonce of 0, valid subsequent nonce values would be 1, 2, 3, and so on.


## Signing a Message
```shell
API_URL=https://api.staging.tauros.io
API_KEY="TAUROS_API_KEY"
API_SECRET="TAUROS_API_SECRET"

NONCE=$(date +%s)

METHOD=POST

REQUEST_PATH="/api/v1/trading/placeorder/"

DATA='{
    "market": "BTC-MXN",
    "amount": "0.001",
    "side": "SELL",
    "type": "LIMIT",
    "price": "250000"
}'

URL_ENCODE_DATA="price=250000&amount=0.001&type=LIMIT&side=SELL&market=BTC-MXN"

MESSAGE=$NONCE$METHOD$REQUEST_PATH$URL_ENCODE_DATA

MESSAGE_SHA256=$(echo "$MESSAGE" | sha256sum)

API_SECRET_DECODED=$(echo $API_SECRET | base64 -d)

SIGNATURE=$(echo -n $MESSAGE_SHA256 | openssl dgst -binary -sha512 -hmac $API_SECRET_DECODED | base64 )

curl -X $METHOD $API_URL$REQUEST_PATH \
  -H "Content-Type: application/json" \
  -H "Taur-Nonce: $NONCE" \
  -H "Taur-Signature: $SIGNATURE" \
  -H "Authorization: Bearer $API_KEY" \
  -d "$DATA";
```


```python
import requests
import json
import time
import hmac
import hashlib
import base64
from urllib.parse import urlencode

api_url = 'https://api.staging.tauros.io'
api_key = 'TAUROS_API_KEY'
api_secret = 'TAUROS_API_SECRET'

path = '/api/v1/trading/placeorder/'

data = {
    "market": "BTC-MXN",
    "amount": "0.001",
    "side": "SELL",
    "type": "LIMIT",
    "price": "250000",
}
method = 'post'
nonce = str(int(1000*time.time()))
request_data = urlencode(data)
message = str(nonce) + method.upper() + path + request_data
api_sha256 = hashlib.sha256(message.encode()).digest()
api_hmac = hmac.new(base64.b64decode(api_secret), api_sha256, hashlib.sha512)
api_signature = base64.b64encode(api_hmac.digest())
signature = api_signature.decode()

headers = {
    'Authorization': 'Bearer {}'.format(api_key),
    'Taur-Signature': signature,
    'Taur-Nonce': nonce,
    'Content-Type': 'application/json',
}

server_res = requests.request(
    method=method,
    url=api_url + path,
    headers=headers,
    data=json.dumps(data),
)

server_res.status_code # 200 ok
server_res.json() # {...}
```

```javascript
const crypto = require('crypto');
const fetch = require('node-fetch');

const URL_API = 'https://staging.api.tauros.io';

const api_key = 'TAUROS_API_KEY';
const api_secret = 'TAUROS_API_SECRET';

let path = '/api/v1/trading/placeorder/';

let method = 'POST';

let data = {
    market: "BTC-MXN",
    amount: "0.001",
    side: "SELL",
    type: "LIMIT",
    price: "250000"
};

let nonce = Date.now() / 1000;
nonce = nonce.toString().replace(".", "");

// make signature
let body = new URLSearchParams(data).toString();

let message = nonce + method.toUpperCase() + path + body;

let api_sha256 = crypto.createHash('sha256').update(message).digest();

// create a sha512 hmac with the secret
let hmac = crypto.createHmac('sha512', Buffer.from(api_secret, 'base64'));

let signature = hmac.update(api_sha256).digest('base64');

let headers = {
  'Content-Type': 'application/json',
  'Authorization': 'Bearer ' + api_key,
  'Taur-Nonce': nonce,
  'Taur-Signature': signature
};

let request = {
    method: method,
    headers: headers,
    body: JSON.stringify(data)
};


fetch(URL_API + path, request)
.then(res => res.ok ? res.json() : {status_code: res.status, message: res.statusText})
.then(json => {
  console.log(json);
})
.catch(err => {
  console.log(err.message);
});
```

```php
<?php

<?php

$apiUrl = 'http://coinbtr:8000';
$apiKey = 'fe4fd5f1e056e4c7aa71bf73f626e855078ec2b8';
$apiSecret = 'b0b1cb82db2782375446ce4f337a7ed20f8093f8705712b7f20a6ae94c5c1869';

$path = '/api/v1/enable-developer-mode/';

$data = array(
   "password" => "hola1425"
);

$method = 'post';
$microTime = explode(' ', microtime());
$nonce = $microTime[1] . str_pad(substr($microTime[0], 2, 6), 6, '0');

$requestData = http_build_query($data);
$message = $nonce . strtoupper($method) . $path . $requestData;
$apiSha256 = hash('sha256', utf8_encode($message), true);
$apiHmac = hash_hmac('sha512', $apiSha256, base64_decode($apiSecret), true);
$signature = base64_encode($apiHmac);

$headers = array(
    "Content-Type: application/json",
    "Taur-Signature: " . $signature,
    "Taur-Nonce: " . $nonce,
    "Authorization: Bearer " . $apiKey
);

$options = array(
    CURLOPT_POST => true,
    CURLOPT_FOLLOWLOCATION => false,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => $headers,
    CURLOPT_POSTFIELDS => json_encode($data, JSON_FORCE_OBJECT),
);

$handler = curl_init($apiUrl . $path);
curl_setopt_array($handler, $options);
$result = curl_exec($handler);
$status = curl_getinfo($handler, CURLINFO_HTTP_CODE);


if ($result === false) {
    die(curl_error($handler));
}
var_dump($status);
echo "<br>";
var_dump($result);

curl_close($handler);
```

The `Taur-Signature` header (message signature) is generated using HMAC-SHA512 of SHA256(`nonce` + `method` + `requestPath` + `body`) and base64 decoded secret API Key. Where + means string concatenation.

* The `nonce` value is the same as the `Taur-Nonce` header.
* The `body` is the request body string or omitted if there is no request body (for GET requests, mainly).
* The `method` should be UPPER CASE.

### Example API clients
Below are sample API client code libraries that can be used when writing your own API client.

### Python
[https://github.com/coinbtr/tauros-api-python](https://github.com/coinbtr/tauros-api-python)
### Node.js
[https://github.com/coinbtr/tauros-api-nodejs](https://github.com/coinbtr/tauros-api-nodejs)
### Php
[https://github.com/coinbtr/tauros-api-php](https://github.com/coinbtr/tauros-api-php)

# Currencies [PUBLIC]
<aside class="notice">
You can access the following endpoints freely, API KEY is not required.
</aside>
## List available currencies
```shell
curl -X GET "https://api.tauros.io/api/v2/coins/"
```
> The API call will response this:

```json
{
  "success": true,
  "payload": {
    "cryto": [
      {
        "coin": "BTC",
        "coin_name": "Bitcoin",
        "min_withdraw": "0.00050000",
        "fee_withdraw": 0.0002,
        "confirmations_required": 4,
        "coin_icon": "https://static.coinbtr.com/media/coins/BTC_9ItLsUF.png",
        "available_to_deposit": true,
        "available_to_withdraw": true
      },
      {
        "coin": "BEMB",
        "coin_name": "Bitcoin Embassy Bar",
        "min_withdraw": "5.00000000",
        "fee_withdraw": 1,
        "confirmations_required": 10,
        "coin_icon": "https://static.coinbtr.com/media/coins/BEMB_Po8mN3B.png",
        "available_to_deposit": false,
        "available_to_withdraw": true
      },
      {
        "coin": "ZEC",
        "coin_name": "ZCash",
        "min_withdraw": "0.08500000",
        "fee_withdraw": 0.05,
        "confirmations_required": 25,
        "coin_icon": "https://static.coinbtr.com/media/coins/ZEC_hecsnl7.png",
        "available_to_deposit": true,
        "available_to_withdraw": false
      },
      {
        "coin": "LTC",
        "coin_name": "Litecoin",
        "min_withdraw": "0.00500000",
        "fee_withdraw": 0.0035,
        "confirmations_required": 6,
        "coin_icon": "https://static.coinbtr.com/media/coins/LTC_O4Ktdtr.png",
        "available_to_deposit": true,
        "available_to_withdraw": true
      },
      {
        "coin": "XLM",
        "coin_name": "Stellar",
        "min_withdraw": "2.00000000",
        "fee_withdraw": 1,
        "confirmations_required": 10,
        "coin_icon": "https://static.coinbtr.com/media/coins/XLM_UKjRoth.png",
        "available_to_deposit": true,
        "available_to_withdraw": true
      },
      {
        "coin": "BCH",
        "coin_name": "Bitcoin Cash",
        "min_withdraw": "0.00200000",
        "fee_withdraw": 0.001,
        "confirmations_required": 6,
        "coin_icon": "https://static.coinbtr.com/media/coins/BCH_IlW6auL.png",
        "available_to_deposit": true,
        "available_to_withdraw": true
      },
      {
        "coin": "DASH",
        "coin_name": "Dash",
        "min_withdraw": "0.00400000",
        "fee_withdraw": 0.002,
        "confirmations_required": 6,
        "coin_icon": "https://static.coinbtr.com/media/coins/DASH_7yHBf0P.png",
        "available_to_deposit": true,
        "available_to_withdraw": true
      }
    ],
    "fiat": [
      {
        "coin": "MXN",
        "coin_name": "Pesos Mexicanos",
        "min_withdraw": "10.00000000",
        "fee_withdraw": "0.0000",
        "country": "Mexico",
        "coin_icon": "https://static.coinbtr.com/media/coins/MXN_LmmSZ71.png"
      }
    ]
  }
}
```
This endpoint returns all available currencies in Tauros, cryptocurrencies as well as fiat currencies.

### HTTP Request
`GET /v2/coins/`

### Query Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|---|
| coin | String | No | Cryptocurrency symbol (e.g. 'btc'). |

# Wallet Operations


## Get Deposit Address

<!-- ```shell
TAUROS_API_KEY='your_api_key'
COIN=btc

curl -X GET "https://api.tauros.io/api/v1/data/getdepositaddress/?coin=$COIN" \
-H "Authorization: Token $TAUROS_API_KEY"
``` -->
This API call will bring you a deposit address for funding your cryptocurrency wallet.

### HTTP Request
`GET /v1/data/getdepositaddress/`

> The API call will response this:

```json
{
  "success": true,
  "msg": null,
  "data": {
    "coin":"btc",
    "address": "2NFfxvXpAWjKng7enFougtvtxxCJ2hQEMo4",
  }
}
```
### Body Parameters

| Parameter | Type | Required | Coins | Description |
|---|---|---|---|---|
| coin | String | Yes | All | Cryptocurrency symbol (e.g. 'btc'). |

<aside class="warning">
Extra information is required for certain coins like XEM (Nem) or XLM (Stellar) that need to be included in your transaction for detecting that you're the owner of the funds.
</aside>
* XEM requires a `message` field.
* XLM requires whether a `memoId` or `memoText`.

Most of exchanges or wallets allows you to include this information in your transaction.

## Cryptocurrency Withdraw

<!-- ```shell
TAUROS_API_KEY="your_api_key"
ADDRESS="2N9JiEUYgRwKAw6FfnUca54VUeaSYSL9qqG"
COIN="btc"
AMOUNT="0.001"

curl -X POST "https://api.tauros.io/api/v1/data/withdraw/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{\"coin\": \"$COIN\", \"address\": \"$ADDRESS\", \"amount\": \"$AMOUNT\"}"
``` -->
This API call allows you to send cryptocurrency to a given destination address.

### HTTP Request
`POST /v3/wallets/crypto-withdraw/`

> The API response will look like this:

```json
{
  "success":true,
  "payload":{
    "txid":"ea3b3df941b229c0d30a7c0e825f4507354aec31ebc082ff0c199e476822c04d",
    "coin":"BTC",
    "isInnerTransfer":false,
    "amount_sent":0.0007,
    "receiver":"2NAzzENWzPk7iJBe8XscDGLJgrr3zjxdcgW",
    "explorer_link":"https://live.blockcypher.com/btc-testnet/tx/"
  }
}
```
### Body Parameters

| Parameter | Type | Required | Coins | Description |
|---|---|---|---|---|
| nip | String | Yes | All | Transactional NIP (e.g. '1234'). |
| coin | String | Yes | All | Cryptocurrency symbol (e.g. 'btc'). |
| address | String | Yes | All | Destination address. |
| amount | Float | Yes | All | Amount to send. |
| memoId | Integer | No | XLM | Memo id attached to the transaction. |
| memoText | String | No | XLM | Memo text attached to the transaction. |

<aside class="notice">
Make sure your API key has permission to perform this action.
</aside>


## Tauros Transfer®

<!-- ```shell
TAUROS_API_KEY="your_api_key"
EMAIL="jhon@mail.com"
COIN="mxn"
AMOUNT="100" # 100 MXN

curl -X POST "https://api.tauros.io/api/v1/data/withdraw/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{\"coin\": \"$COIN\", \"email\": \"$EMAIL\", \"amount\": \"$AMOUNT\"}"
``` -->
*Tauros Transfer®* allows you to transfer funds to another user registered in Tauros.
Funds are transferred instantly with 0 commission fee.

### HTTP Request
`POST /v3/wallets/inner-transfer/`

> The API response will look like this:

```json
{
  "success":true,
  "msg":"0.001 BTC sent to jhon@mail.com.",
  "payload":{
    "amount_sent":0.001,
    "coin":"BTC",
    "fee_amount":0
  }
}
```
### Body Parameters

| Parameter | Type | Required | Coins | Description |
|---|---|---|---|---|
| nip | String | Yes | All | Transactional NIP (e.g. '1234'). |
| coin | String | Yes | All | Coin symbol (e.g. 'btc'). |
| email | String | Yes | All | Receiver email. |
| amount | Float | Yes | All | Amount to send. |

## Mexican Pesos Withdraw (SPEI)
<!-- ```shell
TAUROS_API_KEY="your_api_key"
CLABE="002123456789012345"
RECIPIENT="RAMON SANCHEZ CRUZ"
COIN="mxn"
AMOUNT="100" # 100 MXN

curl -X POST "https://api.tauros.io/api/v1/data/transfer/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{\"coin\": \"$COIN\", \"clabe\": \"$CLABE\", \"amount\": \"$AMOUNT\", \"recipient\": \"$RECIPIENT\"}"
``` -->
> The API response will look like this:

```json
{
  "success": true,
  "msg": "Withdrawal created. Your withdrawal will processed in the next 60 minutes",
}
```
This API call is used to withdraw MXN to a given CLABE.

### HTTP Request
`POST /v3/wallets/mxn-withdraw/`

### Body Parameters
| Parameter | Type | Required | Description |
|---|---|---|---|---|
| nip | String | Yes | Transactional NIP (e.g. '1234') |
| clabe | String | Yes | 18 digits recipient's CLABE (*Clave Bancaria Estandarizada*). |
| amount | Float | Yes | Two decimal presition amount to send. |
| recipient | String | Yes | Recipient's full name (e.g. José Arcadio Buendía). |
| text_ref | String | No | Payment text reference, also known as "*Concepto*" (e.g. Pizzas). |
| numeric_ref | Integer | No | Seven digits numerical reference, also known as "*Referencia numerica*" (e.g. 1234567). |

## List Balances
<!-- ```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/data/listbalances/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY"
``` -->
This API call is used to retrieve your wallets balances, including their deposit addresses (if they have been previously generated). There are three type of balances in tauros: `available`, `pending` and `frozen`.

* `available`: Funds you can spend.

* `pending`: Funds that will be added in your account due to an incoming transfer.

* `frozen`: Frozen funds that will be removed due to and outgoing withdrawal.

* `in_orders`: Funds that are compromised in a limit order that you have previously placed.


### HTTP Request
> The API response will look like this:

```json
{
	"success": true,
	"message": null,
	"data": [{
			"coin": "LTC",
			"balance": 0.00000000,
			"available": 0.00000000,
			"pending": 0.00000000,
			"depositAddress": "DLxcEt3AatMyr2NTatzjsfHNoB9NT62HiF"
		}, {
			"coin": "BTC",
			"balance": 14.21549076,
			"available": 14.21549076,
			"pending": 0.00000000,
			"depositAddress": "1Mrcdr6715hjda34pdXuLqXcju6qgwHA31"
		}
	]
}
```
`GET /v1/data/listbalances/`

### Body Parameters
None

## Get Balance
Alternatively you can request your balance for a specific coin.

<!-- ```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/data/getbalance/?coin=btc" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY"
``` -->

> The API response will look like this:

```json
{
  "success": true,
  "msg": null,
  "data": {
    "coin": "btc",
    "coin_name": "Bitcoin",
    "balances": {
      "available": 0.00033191,
      "frozen": 0.00134,
      "pending": 0.0
    }
  }
}
```

### HTTP Request
`GET /data/getbalance/`

### Body Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| coin | String | No | Coin symbol (e.g. `btc`). |

## List transfers
<!-- ```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/data/transfershistory/?coin=btc&type=deposits" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY"
``` -->

> The API response will look like this:

```json
{
  "count":3,
  "next":"http://api.staging.tauros.io/api/v2/wallets/transfershistory/?limit=3&offset=8",
  "previous":null,
  "results": [
    {
      "sender": null,
      "receiver": "jhon@mail.com",
      "coin": "LTC",
      "coin_name": "Test Litecoin",
      "amount": "2",
      "txId": "93c5e96b149692938d8d0db37e6773f0c2a95e1f2de69efde68b87b92971737b",
      "confirmed": true,
      "created_at": "2019-07-04T10: 23: 03.008961-05: 00",
      "confirmed_at": "2019-07-04T10: 24: 28.156336-05: 00",
      "is_innerTransfer": false,
      "address": "QRca86Fp5wLA5rD87iUfu5rAWTTQndssLp",
      "explorer_link": "https://chain.so/tx/LTCTEST/93c5e96b149692938d8d0db37e6773f0c2a95e1f2de69efde68b87b92971737b",
      "fee_amount": "0",
      "total_amount": 2.0,
      "type": "deposit",
      "description":  null,
      "operation": "transfer",
      "coin_icon": "https://staging-static.coinbtr.com/media/coins/LTC.png",
      "status": "DONE",
      "id": 195
    },
    {
      "sender": "jhon@mail.com",
      "receiver": null,
      "coin": "XLM",
      "coin_name": "Test Stellar",
      "amount": "1.1234567",
      "txId": "cd9976f0c34d457891675d58133f1ef83e06c7dc7566cfbf5114457947453755",
      "confirmed": true,
      "created_at": "2019-07-04T12: 09: 12.681467-05: 00",
      "confirmed_at": "2019-07-04T12: 10: 14-05: 00",
      "is_innerTransfer": false,
      "address": "GCWDFWVXGPBHBTJNT5XRPW2WU2XEKLKG7G4N5LW4PPJEURMSVBBEVOI5",
      "explorer_link": "https://testnet.steexp.com/tx/cd9976f0c34d457891675d58133f1ef83e06c7dc7566cfbf5114457947453755",
      "fee_amount": "0",
      "total_amount": 1.1234567,
      "type": "withdrawal",
      "description":  null,
      "operation": "transfer",
      "coin_icon": "https://staging-static.coinbtr.com/media/coins/XLM.png",
      "status": "DONE",
      "id": 201
    }
  ]
}
```
This API call is used to retrieve your withdraws and deposits history. Both crypto and fiat transfers will be listed here. Tauros Transfers will have the `is_innerTransfer` flag as `true`.

### HTTP Request
`GET /v2/wallets/transfershistory/`

### Query Parameters

| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| limit | Integer | No | Indicates the maximum number of items to return. |
| offset | Integer | No | Indicates the starting position of the query in relation to the complete set of unpaginated items. |
| coin | String | No | Allows to filter by coin (e.g. `btc`). |
| type | String | No | Filters by type of transfer, wich can be `withdrawals` or `deposits`. |

#Trading Operations
<!-- ====================================================================================================== -->
<aside class="notice">
Make sure that your API key has permissions to perform these actions.
</aside>

## Place a new order
<!-- ```shell
TAUROS_API_KEY='your_api_key'
MKT="btc-mxn"
SIDE="sell"
TYPE="limit"
AMOUNT="0.001" # 0.001 BTC
PRICE="100000" # Means 1 BTC = $100,000.00 MXN

curl -X POST "https://api.tauros.io/api/v1/trading/placeorder/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{ \"market\": \"$MKT\", \"amount\": \"$AMOUNT\", \"type\":\"$TYPE\", \"side\": \"$SIDE\", \"price\": \"$PRICE\"}"
``` -->
> The API response will look like this:

```json
{
  "success": true,
  "msg": "Order successfully placed in orderbook",
  "data": {
    "id": 123456790
  }
}
```
You can place two types of orders: `limit` and `market`. Orders can be placed only if your wallet has enough funds. Once an order is placed, your wallet funds will be frozen. If you cancel your order, the associated funds will be restored. If you cancel an open order that has been partially filled the exchanged funds will not be restored.
### HTTP Request
`POST /v1/trading/placeorder/`

### Body Parameters

| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | Yes | Market where your order will be placed (e.g. `btc-mxn`). |
| amount | String or Float | Yes | Amount of coins to trade. |
| side | String | Yes | `buy` or `sell`. |
| price | String or Float| Yes* | Order price at which you wish to exchange your coins. |
| type | String | No | `market` or `limit`. Default is `limit`. |
| is_amount_value | Boolean | No | Designates whether the provided `amount` is the order amount or order value. |

* Required only for limit orders
## List my open orders
<!-- ```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/trading/myopenorders/" \
-H "Authorization: Token $TAUROS_API_KEY"
``` -->

> The API response will look like this:

```json
{
  "success":true,
  "msg":null,
  "data":[{
    "market":"LTC-BTC",
    "side":"BUY",
    "amount":0.00033699,
    "initial_amount":0.00033699,
    "filled":0.0,
    "value":0.00000508,
    "initial_value":0.00000508,
    "price":0.01508999,
    "fee_decimal":0.00075,
    "fee_percent":0.075,
    "fee_amount_paid":0.0,
    "created_at":"2019-03-25T18:24:35.862824Z",
    "is_open":true,
    "amount_received":0.0,
    "amount_paid":0.0,
    "left_coin":"LTC",
    "right_coin":"BTC",
    "order_id":13
  },
  {
    "market":"DASH-BTC",
    "side":"SELL",
    "amount":0.014,
    "initial_amount":0.014,
    "filled":0.0,
    "value":0.00032204,
    "initial_value":0.00032204,
    "price":0.023003,
    "fee_decimal":0.0,
    "fee_percent":0.0,
    "fee_amount_paid":0.0,
    "created_at":"2019-03-27T18:50:52.119514Z",
    "is_open":true,
    "amount_received":0.0,
    "amount_paid":0.0,
    "left_coin":"DASH",
    "right_coin":"BTC",
    "order_id":14
  }]
}
```
This endpoint returns your open orders and their status.
### HTTP Request
`GET /v1/trading/myopenorders/`

### Query Parameters

| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | No | List your open orders filtering by `market` (e.g. `btc-mxn`). |

## Close an open order

<!-- ```shell
TAUROS_API_KEY='your_api_key'
ORDER_ID=3453

curl -X POST "https://api.tauros.io/api/v1/trading/closeorder/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{\"id\": $ORDER_ID}"
``` -->

> The API response will look like this:

```json
{
  "success":true,
  "msg":"Order closed"
}
```
This API call allows you to close an open order that you have previously placed. If your order has already been partially filled, that is to say, partially bought or sold, the exchanged funds will not be restored. Full-filled orders will be automatically closed.

### HTTP Request
`POST /v1/trading/closeorder/`

### Body Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| id | Integer | Yes | Order id |

## List my trading history
<!-- ```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/trading/history/" \
-H "Authorization: Token $TAUROS_API_KEY"
``` -->

> The API response will look like this:

```json
{
  "count": 3,
  "next": null,
  "previous": null,
  "results": [
    {
      "market": "BTC-MXN",
      "amount_paid": 100.0,
      "amount_received": 0.00051077,
      "price": "195000.00",
      "fee_amount": 2.05e-06,
      "side": "BUY",
      "fee": "0.00400000",
      "created_at": "2020-06-03T07:33:18.233149-05:00"
    },
    {
      "market": "BTC-MXN",
      "amount_paid": 0.0001,
      "amount_received": 510.77,
      "price": "195000.00",
      "fee_amount": 2.05,
      "side": "SELL",
      "fee": "0.00200000",
      "created_at": "2020-06-03T07:33:18.233149-05:00"
    },
    {
      "market": "BTC-MXN",
      "amount_paid": 125.0,
      "amount_received": 1.992e-05,
      "price": "160000.00",
      "fee_amount": 8e-08,
      "side": "BUY",
      "fee": "0.00200000",
      "created_at": "2020-06-02T17:59:27.426156-05:00"
    },
    {
      "market": "BTC-MXN",
      "amount_paid": 0.0005,
      "amount_received": 2553.84,
      "price": "195000.00",
      "fee_amount": 10.26,
      "side": "SELL",
      "fee": "0.00200000",
      "created_at": "2020-06-02T12:02:59.894952-05:00"
    }
  ]
}
```
This endpoint retrieves your trading history.

### HTTP Request
`GET /v2/trading/my-trades/<market>/`

### Query Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| limit | Integer | No | Indicates the maximum number of items to return. |
| offset | Integer | No | Indicates the starting position of the query in relation to the complete set of unpaginated items. |


#Market Data [PUBLIC]
The following API calls retrieve information related to markets.

<aside class="notice">
You can access the following endpoints freely, API KEY is not required.
</aside>

## List Markets (Books)
```shell
curl -X GET "https://api.tauros.io/api/v1/trading/markets/"
```
> The API response will look like this:

```json
{
  "count": 7,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 9,
      "name": "LTC-BTC",
      "base_coin_id": 2,
      "min_amount": 10000,
      "max_amount": 1000000000000,
      "min_value": 2500,
      "max_value": 150000000000,
      "min_price": "0.00100000",
      "max_price": "80000.00000000",
      "is_open": false,
      "quote_coin": 3,
      "base_coin_type": 1
    },
    {
      "id": 15,
      "name": "BEMB-MXN",
      "base_coin_id": 7,
      "min_amount": 500,
      "max_amount": 20000000,
      "min_value": 100,
      "max_value": 20000000,
      "min_price": "0.01000000",
      "max_price": "100.00000000",
      "is_open": true,
      "quote_coin": 8,
      "base_coin_type": 2
    },
    {
      "id": 13,
      "name": "DASH-MXN",
      "base_coin_id": 7,
      "min_amount": 50000,
      "max_amount": 10000000000000,
      "min_value": 100,
      "max_value": 100000000,
      "min_price": "100.00000000",
      "max_price": "50000.00000000",
      "is_open": true,
      "quote_coin": 7,
      "base_coin_type": 2
    },
    {
      "id": 12,
      "name": "XLM-MXN",
      "base_coin_id": 7,
      "min_amount": 10000,
      "max_amount": 10000000000000,
      "min_value": 100,
      "max_value": 100000000,
      "min_price": "0.01000000",
      "max_price": "100.00000000",
      "is_open": true,
      "quote_coin": 4,
      "base_coin_type": 2
    },
    {
      "id": 11,
      "name": "BCH-MXN",
      "base_coin_id": 7,
      "min_amount": 10000,
      "max_amount": 10000000000000,
      "min_value": 100,
      "max_value": 1000000000,
      "min_price": "100.00000000",
      "max_price": "200000.00000000",
      "is_open": true,
      "quote_coin": 6,
      "base_coin_type": 2
    },
    {
      "id": 10,
      "name": "LTC-MXN",
      "base_coin_id": 7,
      "min_amount": 10000,
      "max_amount": 100000000000,
      "min_value": 100,
      "max_value": 100000000,
      "min_price": "100.00000000",
      "max_price": "10000000.00000000",
      "is_open": true,
      "quote_coin": 3,
      "base_coin_type": 2
    },
    {
      "id": 6,
      "name": "BTC-MXN",
      "base_coin_id": 7,
      "min_amount": 1000,
      "max_amount": 50000000000,
      "min_value": 100,
      "max_value": 1000000000,
      "min_price": "500.00000000",
      "max_price": "16000000.00000000",
      "is_open": true,
      "quote_coin": 2,
      "base_coin_type": 2
    }
  ]
}

```
This API call returns all existing markets (also known as "books") in Tauros.

### HTTP Request
`GET /v2/trading/markets/`

### Query Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| limit | Integer | No | Indicates the maximum number of items to return. |
| offset | Integer | No | Indicates the starting position of the query in relation to the complete set of unpaginated items. |

## List Market Orders (Orderbook)

```shell
MKT="btc-mxn"

curl -X GET "https://api.tauros.io/api/v1/trading/orders/?market=$MKT"
```
> The API response will look like this:

```json

  "success": true,
  "msg": null,
  "data": {
    "market": "BTC-MXN",
    "asks": [
      {
        "market": "BTC-MXN",
        "side": "SELL",
        "initial_amount": 0.16743,
        "amount": 0.16743,
        "filled": 0.0,
        "value": 12724.68,
        "initial_value": 12724.68,
        "price": 76000.0,
        "left_coin": "BTC",
        "right_coin": "MXN",
        "created_at": "2019-03-27T20:40:14.686939Z"
      },
      {
        "market": "BTC-MXN",
        "side": "SELL",
        "initial_amount": 0.252654,
        "amount": 0.252654,
        "filled": 0.0,
        "value": 19226.96,
        "initial_value": 19226.96,
        "price": 76100.0,
        "left_coin": "BTC",
        "right_coin": "MXN",
        "created_at": "2019-03-27T20:40:33.609201Z"
      }
    ],
    "bids": [
      {
        "market": "BTC-MXN",
        "side": "BUY",
        "initial_amount": 0.00269,
        "amount": 0.00269,
        "filled": 0.0,
        "value": 200.4,
        "initial_value": 200.4,
        "price": 74500.0,
        "left_coin": "BTC",
        "right_coin": "MXN",
        "created_at": "2019-03-27T20:42:09.944029Z"
      },
      {
        "market": "BTC-MXN",
        "side": "BUY",
        "initial_amount": 0.002,
        "amount": 0.002,
        "filled": 0.0,
        "value": 148.0,
        "initial_value": 148.0,
        "price": 74000.0,
        "left_coin": "BTC",
        "right_coin": "MXN",
        "created_at": "2019-03-27T20:41:09.127862Z"
      }
    ]
  }
}
```
This endpoint retrieves the 50 best sell and buy orders (also known as "orderbook") for a given market.

### HTTP Request
`GET /v1/trading/orders/`

### Query Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | Yes | Market name (e.g. `btc-mxn`). |

## List last trades
```shell
MKT="ltc-btc"

curl -X GET "https://api.tauros.io/api/v1/trading/trades/?market=$MKT"
```
> The API response will look like this:

```json
{
  "success": true,
  "msg": null,
  "data": [
    {
      "created_at": "2019-02-20T03:20:53.549954Z",
      "price": "0.01276979",
      "amount": 1.0,
      "value": 0.01278
    },
    {
      "created_at": "2019-02-20T03:20:25.106036Z",
      "price": "0.01276979",
      "amount": 1.0,
      "value": 0.01278
    },
    {
      "created_at": "2019-02-20T03:20:05.758377Z",
      "price": "0.01276979",
      "amount": 1.0,
      "value": 0.01278
    },
    {
      "created_at": "2019-02-20T03:19:41.953355Z",
      "price": "0.01273489",
      "amount": 1.0,
      "value": 0.01277
    },
    {
      "created_at": "2019-02-20T03:07:43.574070Z",
      "price": "0.01270000",
      "amount": 1.0,
      "value": 0.01276979
    },
    {
      "created_at": "2019-02-20T03:05:56.810445Z",
      "price": "0.00426700",
      "amount": 0.1,
      "value": 0.00127697
    }
  ]
}
```
This endpoint returns the last 50 trades for a given market.

### HTTP Request
`GET /v1/trading/trades/`

### Query Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | No | Market name (e.g. `btc-mxn`). |

## List trading fees
```shell
curl -X GET "https://api.tauros.io/api/v1/trading/fees/?market=ltc-btc&user_level=1"
```
> The API response will look like this:

```json
[
  {
    "base_coin": "MXN",
    "taker": "0.00400000",
    "maker": "0.00200000",
    "user_level": 1,
    "upper_amount": 999999.99
  },
  {
    "base_coin": "MXN",
    "taker": "0.00350000",
    "maker": "0.00160000",
    "user_level": 1,
    "upper_amount": 2999999.99
  },
  {
    "base_coin": "BTC",
    "taker": "0.00100000",
    "maker": "0.00020000",
    "user_level": 1,
    "upper_amount": 4.99999999
  },
  {
    "base_coin": "MXN",
    "taker": "0.00300000",
    "maker": "0.00120000",
    "user_level": 1,
    "upper_amount": 9999999.99
  },
  {
    "base_coin": "MXN",
    "taker": "0.00250000",
    "maker": "0.00100000",
    "user_level": 1,
    "upper_amount": 10000000
  },
  {
    "base_coin": "BTC",
    "taker": "0.00075000",
    "maker": "0.00015000",
    "user_level": 1,
    "upper_amount": 14.99999999
  },
  {
    "base_coin": "BTC",
    "taker": "0.00050000",
    "maker": "0.00010000",
    "user_level": 1,
    "upper_amount": 49.99999999
  },
  {
    "base_coin": "BTC",
    "taker": "0.00020000",
    "maker": "0.00007500",
    "user_level": 1,
    "upper_amount": 50
  }
]
```
Returns the trading fees. See [Trading fees](https://tauros.io/fees).

### HTTP Request
`GET /v1/trading/fees/`

### Query Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | No | Market name (e.g. `btc-mxn`). |
| user_level | Integer | No | User level (e.g. 1). By default all users are level 1|

#Websocket

## Subscribe
```javascript
import io from 'socket.io-client';
// or
const io = require('socket.io-client');

const ApiKey = 'YOUR_API_KEY';


// connect to your websocket
const socket = io.connect(`wss://private-ws.tauros.io?token=${ApiKey}`);

socket.on('connect', () => {
  console.log("connected");
});

// Manage notifications
socket.on('notification', (notification) => {
  console.log(notification);
  /*
  Place your code here
  */
});

```


Tauros uses Web-Sockets in order to notify real-time events that occur in your account. The websocket endpoints are the following:

* `private-ws.tauros.io` for production environment.
* `private-ws-staging.tauros.io` for staging environment.

You need to use a [socket.io] client implementation (https://socket.io) if using JavaScript. Installation:

If using yarn:

`yarn add socket.io-client`

If using npm:

`npm install --save socket.io-client`

or CDN:

`<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/1.4.5/socket.io.min.js"></script>`

<aside class="notice">
Note that the private websocket requires your JWT for authentication.
</aside>

## Notifications

> Messages on this channel look like this:

```javascript
// Notification standard:
{
  "title": "NOTIFICATION_TITLE",
  "description": "NOTIFICATION_DESCRIPTION"
  "type": 'TYPE_OF_NOTIFICATION',
  "date": '2019-02-20T03:05:56.810445Z',
  "object": OBJECT // Order, Trade or Transfer object
}
```

Notifications are sent if some of the following events occur.

Types of notifications:

* Order placed (`OP`)
* Order filled (`OF`)
* Order closed (`OC`) ([by the user](#close-an-open-order))
* New trade (`TD`)
* New deposit or withdrawal (`TR`)

The `order`, `trade` or `transfer` object is included in the `object` field.

```javascript
// Order filled  Notification example
{
  "title": "Order filled",
  "description": "Your BUY order has been partially filled"
  "type": 'OF',
  "date": '2019-02-20T03:05:56.810445Z',
  "object": {
    "amount": "0.0005",
    "amount_paid": "100",
    "amount_received": "0.0004995",
    "closed_at": null,
    "created_at": "2019-09-23 21:27:06.083978+00:00",
    "fee_amount_paid": "0.5",
    "fee_decimal": "0.00100000",
    "fee_percent": "0.10000000",
    "filled": "0.0005",
    "id": 107487,
    "initial_amount": "0.0010",
    "initial_value": "200",
    "is_open": true,
    "left_coin": "BTC",
    "market": "BTC-MXN",
    "price": "200000.00",
    "right_coin": "MXN",
    "side": "BUY",
    "value": "100"
  }
}
```

#Websocket [PUBLIC]


```javascript
import io from 'socket.io-client';
// or
let io = require('socket.io-client');


// connect to the websocket
const socket = io.connect('wss://ws.tauros.io');

// Define a market you want to subscribe
const market = 'btc-mxn';

socket.on('connect', () => {
  // Subscribe to the market
  socket.emit('subscribe', market);
});


// Read and manage market data as you want
socket.on('message', (msg) => {
  console.log(msg);
  /*
  Place your code here
  */
});

// For unsubscribing
socket.emit('unsubscribe', market);

```


Tauros implements Web-Sockets in order to provide real-time market data from our trading engine. The websocket public endpoints are the following:

* `ws.tauros.io` for production environment.
* `ws-staging.tauros.io` for staging environment.

We recommend to use [socket.io](https://socket.io) JavaScript. Installation:

If using yarn:

`yarn add socket.io-client`

If using npm:

`npm install --save socket.io-client`

or CDN:

`<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/1.4.5/socket.io.min.js"></script>`


## Order-book Channel
> Messages on this channel look like this:

```json
{
  "channel": "orderbook",
  "type": "BTC-MXN",
  "data": {
    "asks": [
      {
        "a": "0.026832",
        "p": "75000.00",
        "v": "2012.4",
        "t": 1553747507445
      },
      {
        "a": "0.031011",
        "p": "75050.00",
        "v": "2327.37",
        "t": 1553747562361
      }
    ],
    "bids": [
      {
        "a": "0.00531345",
        "v": "397.18",
        "p": "74750.00",
        "t": 1553747963921
      },
      {
        "a": "0.0034",
        "v": "253.3",
        "p": "74500.00",
        "t": 1553747925659
      }
    ]
  }
}
```
The orderbook channel provides the top ten SELL and BUY limit orders for the orderbook (or market) specified.

### JSON message returned

`asks` field contains **SELL** orders.

`bids` field contains **BUY** orders.

The `data` field contains an array with `asks` and `bids` orders in the following form:

| Field name | Type | Description |
|---|---|---|---|---|
| a | String | Order amount. |
| p | String | Order price. |
| v | String | Order value. |
| t | Integer | UTC timestamp with milliseconds precision. |

## Trades Channel
> Messages on this channel look like this:

```json
{
  "channel": "trades",
  "type": "BTC-MXN",
  "data": [
    {
      "a": "0.0034",
      "v": "253.3",
      "p": "74500.00",
      "s": 1,
      "t": 1553747925659
    },
    {
      "a": "0.031011",
      "p": "75050.00",
      "v": "2327.37",
      "s": 0,
      "t": 1553747925659
    },
  ]
}
```
The trades channel provides the last ten trades done for the specified market.

### JSON message returned

The `data` field contains an array with the last trades in the following form:

| Field name | Type | Description |
|---|---|---|---|---|
| a | String | Order amount. |
| p | String | Order price. |
| v | String | Order value. |
| s | Integer | Taker side. 1 indicates **BUY**, 0 indicates **SELL**. |
| t | Integer | UTC timestamp with milliseconds precision. |

#Webhooks
## What is a webhook

Webhooks may be setup to programmatically receive callbacks from Tauros. Webhook notifications are triggered when the specified event occurs, such as an order filled (OF) or a taker trade (TD), among others. You can setup and configure a webhook in the [Tauros Developer Section](https://tauros.io/develop).

Tauros servers will make a POST http request to the specified URL with a JSON payload, and expect a HTTP 200 OK.

## Type of notifications

> Messages on this channel look like this:

```javascript
// Notification standard:
{
  "title": "NOTIFICATION_TITLE",
  "description": "NOTIFICATION_DESCRIPTION"
  "type": 'TYPE_OF_NOTIFICATION',
  "date": '2019-02-20T03:05:56.810445Z',
  "object": OBJECT // Order, Trade or Transfer object
}
```

Webhook can be configurated to notify if some of the following events occur.

Types of notifications:

* Order placed (`OP`)
* Order filled (`OF`)
* Order closed (`OC`) ([by the user](#close-an-open-order))
* New trade (`TD`)
* New deposit or withdrawal (`TR`)

The `order`, `trade` or `transfer` object is included in the `object` field.

```javascript
// Order partially filled notification sample
{
  "title": "Order filled",
  "description": "Your BUY order has been partially filled"
  "type": 'OF',
  "date": '2019-02-20T03:05:56.810445Z',
  "object": {
    "amount": "0.0005",
    "amount_paid": "100",
    "amount_received": "0.0004995",
    "closed_at": null,
    "created_at": "2019-09-23 21:27:06.083978+00:00",
    "fee_amount_paid": "0.5",
    "fee_decimal": "0.00100000",
    "fee_percent": "0.10000000",
    "filled": "0.0005",
    "id": 107487,
    "initial_amount": "0.0010",
    "initial_value": "200",
    "is_open": true,
    "left_coin": "BTC",
    "market": "BTC-MXN",
    "price": "200000.00",
    "right_coin": "MXN",
    "side": "BUY",
    "value": "100"
  }
}
```

```javascript
// Withdrawal notification sample
{
  "title": "New BTC withdrawal",
   "description": "You have sent 0.001 BTC via Tauros Transfer",
   "type": "TR",
   "date": "2020-02-23 19:34:15.004078+00:00",
   "object": {
      "sender": "example1@mail.com",
      "receiver": "emaple2@mail.com",
      "coin": "BTC",
      "coin_name": "Test Bitcoin",
      "amount": "0.001",
      "txId": null,
      "confirmed": true,
      "confirmed_at": "2020-02-23 19:34:14.901017+00:00",
      "is_innerTransfer": true,
      "address": "",
      "explorer_link": null,
      "fee_amount": "0",
      "total_amount": "0.001",
      "type": "withdrawal",
      "description": "My part of papa johns pizza",
      "id": 827
  }
}
```

```javascript
// Taker trade notification sample
{"title": "New trade",
 "description": "New trade in BTC-MXN orderbook",
 "type": "TD",
 "date": "2020-02-23 19:38:49.679603+00:00",
 "object": {
    "market": "BTC-MXN",
    "side": "BUY",
    "amount_paid": "1990",
    "amount_received": "0.00996",
    "price": "199000.00",
    "fee_amount": "0.00004",
    "created_at": "2020-02-23 19:38:49.161057+00:00",
    "left_coin": "BTC",
    "right_coin": "MXN",
    "filled_as": "taker",
    "closed_at": "2020-02-23 19:38:49.161057+00:00"
  }
}
```
