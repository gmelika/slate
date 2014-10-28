---
title: API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='/#/account/api'>Sign Up for a Developer Key</a>
  - <a href='mailto:support@sfox.com'>Need help? Email us</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the SFOX API! The SFOX API allows you to connect your application to our platform execute trades, deposit and withdraw without going through the web interface.

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -u "<api token>:"
```

> Make sure to replace `<api_key>` with your API key, and don't forget the colon.

SFOX uses API keys to allow access to the API. You can register a new SFOX API key at our [developer portal](http://sfox.com/#/account/api).

SFOX expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer <api_key>`

<aside class="notice">
You must replace `<api_key>` with your personal API key.
</aside>

# Price Data

## Get Best Price

```shell
curl "https://www.sfox.com/v1/offer/buy?amount=1"
```

> The result of the calls is something like this:

```json
{
  "quantity":1,
  "vwap":383.00,
  "price":383.21,
  "fees":0.95,
  "total":382.26
}
```

This will return the limit price you need to specify for this order to execute fully. Please note that price fluctuates very quickly and this price is based on the data available at that moment.

To get the sell price simply change "buy" to "sell" in the url.

You can use the "price" returned to you in the limit order you're placing.  "vwap" on the other hand is the expected average execution price. Even though the price is 383.21 in this example, you should expect to pay $383 (vwap * quantity)

## Get Orderbook

```shell
curl "https://www.sfox.com/v1/markets/orderbook"
```

> The result of the calls is an array of bids and asks:

```json
{
  "bids":[
    [383.26,0.53,"bitstamp"],
    [383.2,1.02069829,"bitstamp"],
    [383.18,0.03914609,"bitstamp"],
    [381.01,1.97630598,"bitfinex"],
    [380.93,0.705,"bitfinex"]
  ],
  "asks":[
    [381.06,0.01,"bitfinex"],
    [381.1,5,"bitfinex"],
    [383.74,1.307,"bitstamp"]
  ]
}
```

This will return the blended orderbook of all the available exchanges.

# User

## Get Account Balance

```shell
curl "https://www.sfox.com/v1/user/balance"
  -u "<api_key>:"
```

> The above command returns JSON structured like this:

```json
[
  {
    "currency":"btc",
    "balance":0.627,
    "available":0
  },
  {
    "currency":"usd",
    "balance":0.25161318,
    "available":0.23161321
  }
]
```

This endpoint retrieves all the balances for your account.  It returns an array of objects, each of which has details for a single currency.  As this example return object is demonstrating you can have different values for balance and available.  Balance is your total balance for this currency.  Available, on the other hand, is what is available to you to trade and/or withdraw.  The difference is reserved; either in an open trade or a withdrawal request.

### HTTP Request

`GET https://www.sfox.com/v1/balance`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
NONE

## Withdraw Funds

```shell
curl "https://www.sfox.com/v1/user/withdraw"
  -H "Authorization: <api_key>"
  -d "amount=1"
  -d "address="
  -d "currency=usd"
```

> The above command returns JSON structured like this:

```json
{
  "success": true
}
```

This endpoint submits a withdrawal request to SFOX.

<aside class="notice">If the request fails, the json result will include an error field with the reason.</aside>

### HTTP Request

`POST https://www.sfox.com/v1/user/withdraw`

### Form Parameters

Parameter | Description
--------- | -----------
amount | The amount you wish to withdraw
currency | Currency is one of: usd or btc
address | if currency == btc this field has to be a valid mainnet bitcoin address. Otherwise leave it out or empty

## Request an ACH deposit

```shell
curl "https://www.sfox.com/v1/user/withdraw"
  -H "Authorization: <api_key>"
  -d "amount=1"
```

> The above command returns JSON structured like this:

```json
{
  "success": true
}
```

This endpoint submits an ach deposit request to SFOX.

<aside class="notice">If the request fails, the json result will include an error field with the reason.</aside>

### HTTP Request

`POST https://www.sfox.com/v1/user/deposit`

### Form Parameters

Parameter | Description
--------- | -----------
amount | The amount you wish to deposit from your bank account

# Orders

## Buy Bitcoin

```shell
curl "https://www.sfox.com/v1/orders/buy"
  -u "<api_key>:"
  -d "quantity=1"
  -d "price=10"
```

> The above command returns the same JSON object as the Order Status API, and it is structured like this:

```json
{
  "id": 666,
  "quantity": 1,
  "price": 10,
  "o_action": "Buy",
  "pair": "BTCUSD",
  "type": "Limit",
  "vwap": 0,
  "filled": 0,
  "status": "Started"
}
```

This endpoint initiates a buy order for bitcoin for the specified amount with the specified limit price.  If the price field is omitted, then the order will be treated as a market order and will execute immediately.  If there is an issue with the request you will get the reason in the "error" field.

### HTTP Request

`POST https://www.sfox.com/v1/orders/buy`

### Query Parameters

Parameter | Description
--------- | -----------
quantity | the amount of bitcoin you wish to buy
price | the max price you are willing to pay.  The executed price will always be less than or equal to this price if the market conditions allow it, otherwise the order will not execute.




## Sell Bitcoin

```shell
curl "https://www.sfox.com/v1/orders/sell"
  -u "<api_key>:"
  -d "quantity=1"
  -d "price=10"
```

> The above command returns the same JSON object as the Order Status API, and it is structured like this:

```json
{
  "id": 667,
  "quantity": 1,
  "price": 10,
  "o_action": "Sell",
  "pair": "BTCUSD",
  "type": "Limit",
  "vwap": 0,
  "filled": 0,
  "status": "Started"
}
```

This endpoint initiates a sell order for bitcoin for the specified amount with the specified limit price.  If the price field is omitted, then the order will be treated as a market order and will execute immediately.  If there is an issue with the request you will get the reason in the "error" field.

### HTTP Request

`POST https://www.sfox.com/v1/orders/sell`

### Query Parameters

Parameter | Description
--------- | -----------
quantity | the amount of bitcoin you wish to buy
price | the min price you are willing to accept.  The executed price will always be higher than or equal to this price if the market conditions allow it, otherwise the order will not execute.




## Get Order Status

```shell
curl "https://www.sfox.com/v1/order/<order_id>"
  -u "<api_key>:"
```

> The above command returns a JSON structured like this:

```json
{
  "id": 666,
  "quantity": 1,
  "price": 10,
  "o_action": "Buy",
  "pair": "BTCUSD",
  "type": "Limit",
  "vwap": 0,
  "filled": 0,
  "status": "Started"
}
```

This endpoint returns the status of the order specified by the <order_id> url parameter

### HTTP Request

`GET https://www.sfox.com/v1/order/<order_id>`

### Possible "status" values

Value | Description
--------- | -----------
Started | The order is open on the marketplace waiting for fills
Cancel pending | The order is in the process of being cancelled
Canceled | The order was successfully canceled
Filled | The order was filled
Done | The order was completed successfully




## Get Active Orders

```shell
curl "https://www.sfox.com/v1/orders"
  -u "<api_key>:"
```

> The above command returns an array of Order Status JSON objects structured like this:

```json
[
  {
    "id": 666,
    "quantity": 1,
    "price": 10,
    "o_action": "Buy",
    "pair": "BTCUSD",
    "type": "Limit",
    "vwap": 0,
    "filled": 0,
    "status": "Started"
  },
  {
    "id": 667,
    "quantity": 1,
    "price": 10,
    "o_action": "Sell",
    "pair": "BTCUSD",
    "type": "Limit",
    "vwap": 0,
    "filled": 0,
    "status": "Started"
  }
]
```

This endpoint returns an array of statuses for all active orders.

### HTTP Request

`GET https://www.sfox.com/v1/orders`

### Possible "status" values

Value | Description
--------- | -----------
Started | The order is open on the marketplace waiting for fills
Cancel pending | The order is in the process of being cancelled
Canceled | The order was successfully canceled
Filled | The order was filled
Done | The order was completed successfully




## Cancel Order

```shell
curl "https://www.sfox.com/v1/order/<order_id>"
  -u "<api_key>:"
  -X DELETE
```

> The above command does not return anything.  To check on the cancellation status of the order you will need to poll the get order status api.

This endpoint will start cancelling the order specified.

### HTTP Request

`DELETE https://www.sfox.com/v1/order/<order_id>`

