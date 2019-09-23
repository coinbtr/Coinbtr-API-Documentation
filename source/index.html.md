---
title: Tauros API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://tauros.io/signup'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Tauros provides a simple and practical REST API to help you to automatically perform nearly all actions you can do in our platform [Tauros](https://tauros.io).

## General considerations

Before making API calls consider the following:

* All requests use the `application/json` content type and go over `https`.
* The base url is `https://api.tauros.io/api/v1`.
* All requests are `GET` and `POST` requests methods and responses come in a default response json object with the result in the `data` field.
* Check the `success` flag to ensure that your API call succeeded.
* If something goes wrong look at the `msg` field. There you will find the error description.

## HTTP API Responses

Tauros REST API calls will return a JSON Object.

* A typical successful API call will response a JSON  object that looks like:

`{
  "success": true,
  "msg": null,
  "data": {
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

In order to use our platform through API calls you must enable `Developer Mode` in your profile section.
Once enabled, you can create and configure as many API keys as you need. You can configure each API key with its own level of permissions.

<aside class="notice">
API key is always needed for accessing private endpoints.
</aside>

<!-- ## Login without 2FA
```shell
curl -X POST https://api.tauros.io/api/v1/login/ \
-H 'Content-Type: application/json' \
-d '{"email": "example@mail.com", "password": "secure_pass"}'
```
API key can be obtained by log in to tauros if not 2FA enabled. But withdrawal permission is disabled.

> The API call will response this:

```json
{
    "success": true,
    "data": {
      "token": "3feeac2b57a40c68cc1643b84f848588cb272f7f",
      "email": "example@mail.com",
    }
}
``` -->

## Login with 2FA deactivated
```shell
curl -X POST https://api.tauros.io/api/v2/auth/signin/ \
-H 'Content-Type: application/json' \
-d '{"email": "example@mail.com", "password": "secure_pass"}'
```
Login in this endpoint returns a Json Web Token (JWT) that expires in 30 minutes, therefore `refresh-token` needs to be executed periodically.

> The API call will response this:

```json
{
    "success": true,
    "data": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
      "two_factor":false
    }
}
```

## Login with 2FA activated
```shell
# First step
curl -X POST https://api.tauros.io/api/v2/auth/signin/ \
-H 'Content-Type: application/json' \
-d '{"email": "example@mail.com", "password": "secure_pass"}'
```
Login when the account has two factor authentication enabled is performed in two steps.
The first step requires `email` and `password` and the endpoint will return a temporal `token` that must be provided in the second step along with your 2FA `code`.  

> The API call will response this:

```json
{
    "success": true,
    "msg": null,
    "payload": {
      "token": "cc4be0e5597b0d325765afda9a2d23afe7e0017j",
      "two_factor":true
    }
}
```
```shell
# Second step
curl -X POST https://api.tauros.io/api/v2/auth/verify-tfa/ \
-H 'Content-Type: application/json' \
-d '{"tempToken": "cc4be0e5597b0d325765afda9a2d23afe7e0017j", "code": "123456"}'
```

> The API call will response this:

```json
{
    "success": true,
    "msg": null,
    "payload": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
      "two_factor":true
    }
}
```

## Refresh JWT
```sh
$ curl -X POST https://api.tauros.io/api/v2/auth/refresh-jwt/ \
  -H 'Content-Type: application/json' \
  -d '{"token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"}'
```
Json Web Token (JWT) is configured to expire in 30 minutes, after that time, the token is not valid anymore. Therefore refreshing the token needs to be executed periodically if using JWT for authenticating API calls.

> The API call will response this:

```json
{
    "success": true,
    "msg": null,
    "payload": {
      "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c",
      "two_factor":true
    }
}
```

# Currencies [PUBLIC]
<aside class="notice">
You can access the following endpoints freely, API KEY is not required.
</aside>
## List available currencies
```shell
curl -X GET "https://api.tauros.io/api/v1/data/coins/"
```
> The API call will response this:

```json
{
  "success": true,
  "msg": "",
  "data": {
    "crypto": [
      {
        "coin": "XEM",
        "min_withdraw": 15.0,
        "fee_withdraw": 5.0,
        "confirmations_required": 10
      },
      {
        "coin": "DASH",
        "min_withdraw": 0.004,
        "fee_withdraw": 0.002,
        "confirmations_required": 6
      },
      {
        "coin": "ZEC",
        "min_withdraw": 0.085,
        "fee_withdraw": 0.05,
        "confirmations_required": 25
      },
      {
        "coin": "LTC",
        "min_withdraw": 0.002,
        "fee_withdraw": 0.003,
        "confirmations_required": 6
      },
      {
        "coin": "BTC",
        "min_withdraw": 0.0006,
        "fee_withdraw": 0.0003,
        "confirmations_required": 4
      },
      {
        "coin": "BCH",
        "min_withdraw": 0.002,
        "fee_withdraw": 0.001,
        "confirmations_required": 6
      },
      {
        "coin": "XLM",
        "min_withdraw": 20.0,
        "fee_withdraw": 0.1,
        "confirmations_required": 10
      }
    ],
    "fiat": [
      {
        "coin": "MXN",
        "min_withdraw": 20.0,
        "fee_withdraw": 0.0,
        "country": "Mexico"
      }
    ]
  }
}
```
This endpoint returns all available currencies in Tauros, cryptocurrencies as well as fiat currencies.

### HTTP Request
`GET /data/coins/`

### Query Parameters
None

# Wallet Operations
<!-- ====================================================================================================== -->

## Get Deposit Address

```shell
TAUROS_API_KEY='your_api_key'
COIN=btc

curl -X GET "https://api.tauros.io/api/v1/data/getdepositaddress/?coin=$COIN" \
-H "Authorization: Token $TAUROS_API_KEY"
```
This API call will bring you a deposit address for funding your cryptocurrency wallet.

### HTTP Request
`GET /data/getdepositaddress/`

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

```shell
TAUROS_API_KEY="your_api_key"
ADDRESS="2N9JiEUYgRwKAw6FfnUca54VUeaSYSL9qqG"
COIN="btc"
AMOUNT="0.001"

curl -X POST "https://api.tauros.io/api/v1/data/withdraw/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{\"coin\": \"$COIN\", \"address\": \"$ADDRESS\", \"amount\": \"$AMOUNT\"}"
```
This API call allows you to send cryptocurrency to a given destination address.

### HTTP Request
`POST /data/whithdraw/`

> The API response will look like this:

```json
{
  "success": true,
  "msg": null,
  "data": {
    "coin":"BTC",
    "txid": "a4b50c3f7fb5dd9273f5be69661b79eed61570421f76ec903ad914d39980549e",
    "status": "signed",
    "date": "2018-07-02T04:57:55.942Z"
  }
}
```
### Body Parameters

| Parameter | Type | Required | Coins | Description |
|---|---|---|---|---|
| coin | String | Yes | All | Cryptocurrency symbol (e.g. 'btc'). |
| address | String | Yes | All | Destination address. |
| amount | Float | Yes | All | Amount to send. |
| message | String | No | XEM | Message attached to the transaction. |
| memoId | Integer | No | XLM | Memo id attached to the transaction. |
| memoText | String | No | XLM | Memo text attached to the transaction. |

<aside class="notice">
Make sure your API key has permission to perform this action.
</aside>


## btr pay®

```shell
TAUROS_API_KEY="your_api_key"
EMAIL="jhon@mail.com"
COIN="mxn"
AMOUNT="100" # 100 MXN

curl -X POST "https://api.tauros.io/api/v1/data/withdraw/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{\"coin\": \"$COIN\", \"email\": \"$EMAIL\", \"amount\": \"$AMOUNT\"}"
```
*btr pay®* allows you to send funds to another user registered in Tauros.
Funds are transferred instantly with 0 commission fee.

### HTTP Request
`POST /data/transfer/`

> The API response will look like this:

```json
{
  "success":true,
  "msg": "100.00 MXN sent to jhon@mail.com.",
  "data": {
    "amount": 100.00,
    "coin": "MXN"
  }
}
```
### Body Parameters

| Parameter | Type | Required | Coins | Description |
|---|---|---|---|---|
| coin | String | Yes | All | Coin symbol (e.g. 'btc'). |
| email | String | Yes | All | Receiver email. |
| amount | Float | Yes | All | Amount to send. |

## Mexican Pesos Withdraw (SPEI)
```shell
TAUROS_API_KEY="your_api_key"
CLABE="002123456789012345"
RECIPIENT="RAMON SANCHEZ CRUZ"
COIN="mxn"
AMOUNT="100" # 100 MXN

curl -X POST "https://api.tauros.io/api/v1/data/transfer/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{\"coin\": \"$COIN\", \"clabe\": \"$CLABE\", \"amount\": \"$AMOUNT\", \"recipient\": \"$RECIPIENT\"}"
```
> The API response will look like this:

```json
{
  "success": true,
  "msg": "Withdrawal created. Your withdrawal will processed in the next 60 minutes",
}
```
This API call is used to withdraw MXN to a given CLABE.

### HTTP Request
`POST /data/fiatwithdraw/`

### Body Parameters
| Parameter | Type | Required | Coins | Description |
|---|---|---|---|---|
| coin | String | Yes | MXN | Fiat coin symbol. Currently `MXN` is only supported |
| clabe | String | Yes | MXN | 18 digits recipient's CLABE (*Clave Bancaria Estandarizada*). |
| amount | Float | Yes | MXN | Amount to send. |
| recipient | String | Yes | MXN | Recipient's full name. |

## List Balances
```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/data/listbalances/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY"
```
This API call is used to retrieve your wallets balances, including their deposit addresses. There are three type of balances in tauros: `available`, `pending` and `frozen`.

* `available`: Funds you can spend.

* `pending`: Funds that will be added in your account due to an incoming transfer.

* `frozen`: Frozen funds due to a limit order you have previously placed.


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
`GET /data/listbalances/`

### Body Parameters
None

## Get Balance
Alternatively you can request your balance for a specific coin.

```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/data/getbalance/?coin=btc" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY"
```

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
```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/data/transfershistory/?coin=btc&type=deposits" \   
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY"
```

> The API response will look like this:

```json
{
  "success": true,
  "msg": null,
  "data": {
    "transfers": [{
      "sender": "None",
      "receiver": "jhon@mail.com",
      "coin": "btc",
      "coin_name": "Bitcoin",
      "txId": "bd8d8dc229ccc4a6f06a41e138eae6e10e44b5deec2ff8bbcc26c0b07fb6a466",
      "confirmed": true,
      "created_at": "2019-03-19T20: 11: 37.073628Z",
      "confirmed_at": "2019-03-19T20: 51: 38.808905Z",
      "is_innerTransfer": false,
      "address": "3K4ijxakSQ86Nk26JZuHZa2m7tHjJ3YeSb",
      "explorer_link": "https://www.blockchain.com/es/btc/tx/bd8d8dc229ccc4a6f06a41e138eae6e10e44b5deec2ff8bbcc26c0b07fb6a466",
      "amount": 0.00033699,
      "fee_amount": 0.0,
      "total_amount": 0.00033699,
      "type": "deposit"
    }]
  }
}
```
This API call is used to retrieve your withdraws and deposits history. These can be filtered by type, which can be `withdrawals` or `deposits`, and/or by `coin`.

### HTTP Request
`GET /data/transfershistory/`

### Body Parameters

| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| coin | String | No | Coin symbol (e.g. `btc`). |
| type | String | No |type of transfer, wich can be `withdrawals` or `deposits`. |

#Trading Operations
<!-- ====================================================================================================== -->
<aside class="notice">
Make sure that your API key has permissions to perform these actions.
</aside>

## Place a new order
```shell
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
```
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
`POST /trading/placeorder/`

### Body Parameters

| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | Yes | Market where your order will be placed (e.g. `btc-mxn`). |
| amount | String | Yes | Amount of coins to trade. |
| side | String | Yes | `buy` or `sell`. |
| type | String | Yes | `market` or `limit`. |
| price | String | Yes | Order price at which you wish to exchange your coins. |

## List my open orders
```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/trading/myopenorders/" \
-H "Authorization: Token $TAUROS_API_KEY"
```

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
`GET /trading/myopenorders/`

### Query Parameters

| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | No | List your open orders filtering by `market` (e.g. `btc-mxn`). |

## Close an open order

```shell
TAUROS_API_KEY='your_api_key'
ORDER_ID=3453

curl -X POST "https://api.tauros.io/api/v1/trading/closeorder/" \
-H "Content-Type: application/json" \
-H "Authorization: Token $TAUROS_API_KEY" \
-d "{\"id\": $ORDER_ID}"
```

> The API response will look like this:

```json
{
  "success":true,
  "msg":"Order closed"
}
```
This API call allows you to close an open order that you have previously placed. If your order has already been partially filled, that is to say, partially bought or sold, the exchanged funds will not be restored. Full-filled orders will be automatically closed.

### HTTP Request
`POST /trading/closeorder/`

### Body Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| id | Integer | Yes | Order id |

## List my trading history
```shell
TAUROS_API_KEY='your_api_key'

curl -X GET "https://api.tauros.io/api/v1/trading/history/" \
-H "Authorization: Token $TAUROS_API_KEY"
```

> The API response will look like this:

```json
{
  "success":true,
  "msg":null,
  "data":{
    "all":[{
      "market":"XEM-BTC",
      "side":"SELL",
      "amount_paid":4.0,
      "amount_received":6.82e-05,
      "price":1.705e-05,
      "fee_amount":0.0,
      "created_at":"2019-01-07T20:11:22.820012Z",
      "left_coin":"XEM",
      "right_coin":"BTC",
      "filled_as":"maker",
      "closed_at":"2019-01-07T20:11:56.457616Z"
    },
    {
      "market":"XEM-BTC",
      "side":"SELL",
      "amount_paid":4.0,
      "amount_received":6.82e-05,
      "price":1.705e-05,
      "fee_amount":0.0,
      "created_at":"2019-01-04T04:06:53.187436Z",
      "left_coin":"XEM",
      "right_coin":"BTC",
      "filled_as":"maker",
      "closed_at":"2019-01-07T20:11:56.383562Z"
    }],
    "taker":[],
    "maker":[{
      "market":"XEM-BTC",
      "side":"SELL",
      "amount_paid":4.0,
      "amount_received":6.82e-05,
      "price":1.705e-05,
      "fee_amount":0.0,
      "created_at":"2019-01-04T04:06:53.187436Z",
      "left_coin":"XEM",
      "right_coin":"BTC",
      "filled_as":"maker",
      "closed_at":"2019-01-07T20:11:56.383562Z"
    },
    {
      "market":"XEM-BTC",
      "side":"SELL",
      "amount_paid":4.0,
      "amount_received":6.82e-05,
      "price":1.705e-05,
      "fee_amount":0.0,
      "created_at":"2019-01-07T20:11:22.820012Z",
      "left_coin":"XEM",
      "right_coin":"BTC",
      "filled_as":"maker",
      "closed_at":"2019-01-07T20:11:56.457616Z"
    }]
  }
}
```
This endpoint retrieves your trading history.

The `all` field contains maker and taker history ordered by most recent. Filtering by `market` is optional.

### HTTP Request
`GET /trading/history/`

### Query Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | No | List your trades filtering by `market` (e.g. `btc-mxn`). |


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
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "BTC-MXN",
      "base": "MXN",
      "quote": "BTC"
    },
    {
      "id": 2,
      "name": "LTC-MXN",
      "base": "MXN",
      "quote": "LTC"
    },
    {
      "id": 3,
      "name": "DASH-MXN",
      "base": "MXN",
      "quote": "DASH"
    },
    {
      "id": 4,
      "name": "DASH-BTC",
      "base": "BTC",
      "quote": "DASH"
    },
    {
      "id": 3,
      "name": "LTC-BTC",
      "base": "BTC",
      "quote": "LTC"
    },
    {
      "id": 5,
      "name": "ZEC-BTC",
      "base": "BTC",
      "quote": "ZEC"
    }
  ]
}

```
This API call returns all existing markets (also known as "books") in Tauros.

### HTTP Request
`GET /trading/markets/`

### Query Parameters
| Parameter | Type | Required |  Description |
|---|---|---|---|---|
| market | String | No | Market name (e.g. `btc-mxn`). |

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
`GET /trading/orders/`

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
`GET /trading/trades/`

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
    "market": "LTC-BTC",
    "taker": "0.00095000",
    "maker": "0.00075000",
    "user_level": 1,
    "lower_amount": 0.0,
    "upper_amount": 9.99999999
  },
  {
    "market": "LTC-BTC",
    "taker": "0.00090000",
    "maker": "0.00070000",
    "user_level": 1,
    "lower_amount": 10.0,
    "upper_amount": 19.99999999
  },
  {
    "market": "LTC-BTC",
    "taker": "0.00085000",
    "maker": "0.00065000",
    "user_level": 1,
    "lower_amount": 20.0,
    "upper_amount": 49.99999999
  },
  {
    "market": "LTC-BTC",
    "taker": "0.00075000",
    "maker": "0.00060000",
    "user_level": 1,
    "lower_amount": 50.0,
    "upper_amount": 499.99999999
  },
  {
    "market": "LTC-BTC",
    "taker": "0.00070000",
    "maker": "0.00055000",
    "user_level": 1,
    "lower_amount": 500.0,
    "upper_amount": 1799.99999999
  },
  {
    "market": "LTC-BTC",
    "taker": "0.00065000",
    "maker": "0.00050000",
    "user_level": 1,
    "lower_amount": 1800.0,
    "upper_amount": 10000.0
  }
]
```
Returns the trading fees. See [Trading fees](https://tauros.io/fees).

### HTTP Request
`GET /trading/fees/`

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
let io = require('socket.io-client');

const JWT = 'YOUR_JSON_WEB_TOKEN';


// connect to your websocket
const socket = io.connect(`wss://private-ws.coinbtr.com?token=${JWT}`);

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

* `private-ws.coinbtr.com` for production environment.
* `private-ws-staging.coinbtr.com` for staging environment.

We recommend to use [socket.io](https://socket.io) if using JavaScript. Installation:

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
const socket = io.connect('wss://ws.coinbtr.com');

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

* `ws.coinbtr.com` for production environment.
* `ws-staging.coinbtr.com` for staging environment.

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
