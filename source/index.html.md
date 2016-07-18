---
title: API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='/account/api'>Sign Up for a Developer Key</a>
  - <a href='mailto:support@sfox.com'>Need help? Email us</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the SFOX API! The API allows you to connect your application to SFOX, execute trades, and deposit and withdraw currencies.

# Authentication

> To authorize, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -u "<api token>:"
```

> Make sure to replace <api_key> with your API key, and don't forget the colon.

SFOX uses API keys to grant access. You can create a new SFOX API key at our [developer portal](http://sfox.com/#/account/api).

The API key should be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer <api_key>`

<aside class="notice">
You must replace `api_key` with your personal API key.
</aside>

# Price Data

## Get Best Price

```shell
curl "https://api.sfox.com/v1/offer/buy?amount=1"
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

This will return the price you need to specify for a limir order to execute fully. Please note that price fluctuates very quickly and this price is based on the data available at that moment.  If you want to trade now, we recommend using Buy Now and Sell Now calls and not limit orders.

To get the sell price simply change "buy" to "sell" in the url.

Use the "price" returned to you as the price in limit order you're placing.

We also return VWAP - Volume weighted average price.  "vwap" is the expected price you will pay for the entire order.  Even though the price is 383.21 in this example, you will most likely pay $383 for the entire order.  These prices are not guaranteed as the market is always moving.  

### HTTP Request

`GET https://api.sfox.com/v1/offer/buy?quantity`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
quantity||The number of bitcoin you will be trading


## Get Orderbook

```shell
curl "https://api.sfox.com/v1/markets/orderbook"
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

### HTTP Request

`GET https://api.sfox.com/v1/markets/orderbook`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
NONE

# User

## Get Account Balance

```shell
curl "https://api.sfox.com/v1/user/balance"
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

Use this endpoint to access your account balance.  It returns an array of objects, each of which has details for a single currency.  

You will get Balance and Available balance.  Balance is your total balance for this currency.  Available, on the other hand, is what is available to you to trade and/or withdraw.  The difference is amount that is reserved either in an open trade or pending a withdrawal request.

### HTTP Request

`GET https://api.sfox.com/v1/balance`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
NONE

## Request an ACH deposit

```shell
curl "https://api.sfox.com/v1/user/withdraw"
  -H "Authorization: <api_key>"
  -d "amount=1"
```

> The above command returns JSON structured like this:

```json
{
  "success": true
}
```

You can transfer funds from your bank account to SFOX using ACH.  You have to setup your bank account by going to [Accounts / Deposits](https://api.sfox.com/#/account/deposit).  Once you have setup your bank account, you can use this call to initiate the transfer request.

<aside class="notice">If the request fails, the json result will include an error field with the reason.</aside>

### HTTP Request

`POST https://api.sfox.com/v1/user/deposit`

### Form Parameters

Parameter | Description
--------- | -----------
amount | The amount you wish to deposit from your bank account


## Withdraw Funds

```shell
curl "https://api.sfox.com/v1/user/withdraw"
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

Submits a coin or currency withdrawal request to SFOX.  Your funds must be available before requesting withdrawal.  For fiat currency withdrawal, your bank account must be setup prior to making the withdrawal request.  You can setup your bank account by going to [Accounts / Deposits](https://api.sfox.com/#/account/deposit).

<aside class="notice">If the request fails, the json result will include an error field with the reason.</aside>

### HTTP Request

`POST https://api.sfox.com/v1/user/withdraw`

### Form Parameters

Parameter | Description
--------- | -----------
amount | The amount you wish to withdraw
currency | Currency is one of: usd or btc
address | if currency == btc this field has to be a valid mainnet bitcoin address. Otherwise leave it out or empty


# Orders

## Buy Now

```shell
curl "https://api.sfox.com/v1/orders/buy"
  -u "<api_key>:"
  -d "quantity=1"
```

> The above command returns the same JSON object as the Order Status API, and it is structured like this:

```json
{
  "id": 666,
  "quantity": 1,
  "price": 10,
  "o_action": "Buy",
  "pair": "btcusd",
  "type": "Limit",
  "vwap": 0,
  "filled": 0,
  "status": "Started"
}
```

This is a market order request to buy certain quantity of bitcoin.  Since this is a "Buy Now" order, it will execute immediately.

### HTTP Request

`POST https://api.sfox.com/v1/orders/buy`

### Query Parameters

Parameter | Description
--------- | -----------
quantity | the amount of bitcoin you wish to buy


## Buy Bitcoin

```shell
curl "https://api.sfox.com/v1/orders/buy"
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
  "pair": "btcusd",
  "type": "Limit",
  "vwap": 0,
  "filled": 0,
  "status": "Started"
}
```

This endpoint initiates a buy order for bitcoin for the specified amount with the specified limit price.  If the price field is omitted, then the order will be treated as a market order and will execute immediately.  If there is an issue with the request you will get the reason in the "error" field.

### HTTP Request

`POST https://api.sfox.com/v1/orders/buy`

### Query Parameters

Parameter | Description
--------- | -----------
quantity | the amount of bitcoin you wish to buy
price | the max price you are willing to pay.  The executed price will always be less than or equal to this price if the market conditions allow it, otherwise the order will not execute.
algorithm_id | the [algorithm id](#algorithm-ids) you wish to use to execute the order (default: 200) 
client_order_id | this is an optional field that will hold a user specified id for reference
currency_pair | the currency pair you wish to trade (default: btcusd)


## Sell Now

```shell
curl "https://api.sfox.com/v1/orders/sell"
  -u "<api_key>:"
  -d "quantity=1"
```

> The above command returns the same JSON object as the Order Status API, and it is structured like this:

```json
{
  "id": 667,
  "quantity": 1,
  "price": 10,
  "o_action": "Sell",
  "pair": "btcusd",
  "type": "Limit",
  "vwap": 0,
  "filled": 0,
  "status": "Started"
}
```
This is a market order request to sell certain quantity of bitcoin.  Since this is a "Sell Now" order, it will execute immediately.


### HTTP Request

`POST https://api.sfox.com/v1/orders/sell`

### Query Parameters

Parameter | Description
--------- | -----------
quantity | the amount of bitcoin you wish to buy

## Sell Bitcoin

```shell
curl "https://api.sfox.com/v1/orders/sell"
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
  "pair": "btcusd",
  "type": "Limit",
  "vwap": 0,
  "filled": 0,
  "status": "Started"
}
```

This endpoint initiates a sell order for bitcoin for the specified amount with the specified limit price.  If the price field is omitted, then the order will be treated as a market order and will execute immediately.  If there is an issue with the request you will get the reason in the "error" field.

### HTTP Request

`POST https://api.sfox.com/v1/orders/sell`

### Query Parameters

Parameter | Description
--------- | -----------
quantity | the amount of bitcoin you wish to buy
price | the min price you are willing to accept.  The executed price will always be higher than or equal to this price if the market conditions allow it, otherwise the order will not execute.
algorithm_id | the [algorithm id](#algorithm-ids) you wish to use to execute the order (default: 200) 
client_order_id | this is an optional field that will hold a user specified id for reference
currency_pair | the currency pair you wish to trade (default: btcusd)




## Get Order Status

```shell
curl "https://api.sfox.com/v1/order/<order_id>"
  -u "<api_key>:"
```

> The above command returns a JSON structured like this:

```json
{
  "id": 666,
  "quantity": 1,
  "price": 10,
  "o_action": "Buy",
  "pair": "btcusd",
  "type": "Limit",
  "vwap": 0,
  "filled": 0,
  "status": "Started"
}
```

This endpoint returns the status of the order specified by the <order_id> url parameter

### HTTP Request

`GET https://api.sfox.com/v1/order/<order_id>`

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
curl "https://api.sfox.com/v1/orders"
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
    "pair": "btcusd",
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
    "pair": "btcusd",
    "type": "Limit",
    "vwap": 0,
    "filled": 0,
    "status": "Started"
  }
]
```

This endpoint returns an array of statuses for all active orders.

### HTTP Request

`GET https://api.sfox.com/v1/orders`

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
curl "https://api.sfox.com/v1/order/<order_id>"
  -u "<api_key>:"
  -X DELETE
```

> The above command does not return anything.  To check on the cancellation status of the order you will need to poll the get order status api.

This endpoint will start cancelling the order specified.

### HTTP Request

`DELETE https://api.sfox.com/v1/order/<order_id>`

## Algorithm IDs
ID | Description
--------- | -----------
200 | Smart Routing
301 | Goliath
302 | Tortoise
303 | Hare
304 | Stop-Loss
305 | Polar Bear
306 | Sniper
307 | TWAP

