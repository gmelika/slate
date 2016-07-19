---
title: Partner API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='www.sfox.com/partners'>Sign Up for a Partner Account</a>
  - <a href='mailto:support@sfox.com'>Need help? Email us</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the SFOX API! The API allows you to offer buy/sell services to your customers through the SFOX platform.

> To create an account, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "https://api.sfox.com/v2/partner/<partner name>/<path>" \
  -H "X-SFOX-PARTNER-ID: <partner id>"
```

SFOX uses partner IDs only for tracking purposes.

<aside class="notice">
You must replace "partner id" with your partner id provided by sfox.
</aside>

# Creating a Customer Account

This api allows you to create an account on SFOX for the customer.  This api will return an API token associated with this account which will allow you to submit requests on behalf of the customer.

> To create an account, use this code:

```shell
# With shell, you can just pass the correct header with each request
curl "https://api.sfox.com/v2/partner/<partner name>/account" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Content-type: application/json" \
  -d "username=user@domain.com" \
  -d "password=password123"
```

> The result of the call will be something like this:

```json
{
  "token": "<account token>",
  "account": {
    "id": "user123",
    "verification_status": {
      "level": "unverified"
    }
    "can_buy": false,
    "can_sell": false,
    "limits": {
      "usd": {
        "available": {
          "buy": 5000,
          "sell": 5000
        },
        "total": {
          "buy": 5000,
          "sell": 5000
        }
      }
    }
  }
}
```

# KYC

## Verify Account

```shell
curl "https://api.sfox.com/v2/partner/<partner name>/account/<account id>/verify" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
  -H "Content-type: application/json" \
  -d '{
  "firstname": "name1",
  "middlename": "t",
  "lastname": "name2",
  "street1": "street1",
  "street2": "street2",
  "city": "city",
  "state": "state",
  "zipcode": "zip",
  "country": "US",
  "birth_day": 1,
  "birth_month": 1,
  "birth_year": 1990,
  "ssn": 123456789,
  "identity": {
    "type": "driver_license",
    "number": "b1234567",
    "state": "CA",
    "country": "US"
  },
  "phone_number": "+12345678900"
}'

```

> The result of the calls is the updated account info:

```json
{
	"account": {
		"id": "<account id>",
		"verification_status": {
			"level": "needs_documents",
			"required_docs": ["address", "ssn"]
		},
		"can_buy": true,
		"can_sell": true,
		"limits": {
			"usd": {
				"available": {
					"buy": 5000,
					"sell": 5000
				},
				"total": {
					"buy": 10000,
					"sell": 10000
				}
			}
		}
	}
}
```

This api allows you to submit personal information on the user for KYC purposes.  Users are not allowed to buy/sell currencies without having to submit this information and getting their account to a verified status.

After you submit the information, the api will return a verification status with possible next steps for the user.

If the api returns a list of [required documents](#required-docs) then the user has to submit these documents for further consideration.

## Upload Required Documents

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/account/<account id>/upload/sign" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
  -H "Content-type: application/json" \
  -d `{
    "type": "address",
    "filename": "utility-bill-201606.jpg",
    "mime_type": "image/jpg"
  }`
```

> The result of the call is the signed_request to upload the file to:

```json
{
  "signed_url": "https://sfox-kyc.s3.amazonaws.com/31-individual-address-utility-bill-201606-ada7189dc4c7.JPG?AWSAccessKeyId=AKIAI4JAQOPLZJH3WRMA&Content-Type=image%2Fjpeg&Expires=1468909986&Signature=aqU5abWqt0fmE5GTjQUGh82%2BpzU%3D&x-amz-acl=private""
}
```

User documents are uploaded directly to S3 from the client browser.  The process is as follows:  

1. client requests a signed url from SFOX
2. client uploads file to the url returned by #1

### HTTP Request

`POST https://api.sfox.com/v1/upload/sign`

# Payment Methods

## Add Payment Method

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/account/<account id>/paymentmethod/<currency>" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>:" \
  -H "Content-type: application/json" \
  -d '{
	"type": "ach",
	"ach": {
		"routing_number": "0123456789",
		"account_number": "0001112345667",
		"name1": "john doe",
		"name2": "jane doe",
		"nickname": "checking 1"
	}
}'
```

> The above api will return the payment id and status:

```json
{
  "payment_id": "payment123",
  "status": "pending"
}
```

This api is used to add a payment method to the account.  Currently, this api only supports "ach" as a payment type.  

### Payment Status

Statys | Description
---------  | -----------
pending|payment method requires verification. use the [payment verification](#verify-payment-method) api to finish adding the payment method
active|payment method is ready to be used

### HTTP Request

`POST https://api.sfox.com/v2/partner/<partner id>/account/<account id>/paymentmethod/<currency>`

## Verify Payment Method

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/account/<account id>/paymentmethod/<payment method id>/verify" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -d '{
	"amount1": 0.12,
	"amount2": 0.34
}'
```

> This api will return the payment method status:

```json
{
  "payment_id": "payment123",
  "status": "active"
}
```

Use this api for payment methods that are in the "pending" state.  These payment methods, like ACH account, require the user to enter the 2 amounts that will be sent to their account.  When the amounts match, the payment method will be activated

### HTTP Request

`POST https://api.sfox.com/v2/partner/<partner id>/account/<account id>/paymentmethod/<payment method id>/verify`

## Get Payment Methods

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/account/<account id>/paymentmethod"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json"
```

> The above command returns JSON structured like this:

```json
[
	{
		"type": "ach",
		"status": "active",
		"routing_number": "**89",
		"account_number": "**67",
		"nickname": "checking 1",
    "currency": "usd"
	}
]
```

Returns a list of payment methods on the account

### HTTP Request

`POST https://api.sfox.com/v2/partner/<partner id>/account/<account id>/paymentmethod`

# Quotes

## Request a Quote

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/quote/<action>/<base currency>/<quote currency>/<amount>/<amount currency>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json"
```

> The quote returned will have the following format

```json
{
	"quote_id": "a5098dd0-4cb2-4256-9cb5e871fbe672d1",
	"quote_amount": "3000",
	"quote_currency": "usd",
	"base_amount": "5",
	"base_currency": "btc",
	"ttl": 3000,
	"fee": "75",
	"fee_currency": "usd"
}
```

This api allows you to request a quote from SFOX for certain amount.  This is used for both buying/selling currencies.  The api also allows you to request
a quote in both the base and quote currencies.

### HTTP Request

`GET https://api.sfox.com/v2/partner/<partner id>/quote/<action>/<base currency>/<quote currency>/<amount>/<amount currency>`

### Request Parameters

Parameter | Description
--------- | -----------
action | "buy" or "sell"
base currency | the base currency of the requested pair.  In the case of "btcusd", "btc" is the base currency
quote currency | the quote currency of the requested pair.  In the case of "btcusd", "usd" is the quote currency
amount | the amount requested
amount currency | the currency of the amount requested

### Examples

#### A quote to buy $20 worth of bitcoins

`GET https://api.sfox.com/v2/partner/sfox/quote/buy/btc/usd/20/usd`

#### A quote to buy 2 bitcoins

`GET https://api.sfox.com/v2/partner/sfox/quote/buy/btc/usd/2/btc`

#### A quote to sell 2.12345678 bitcoins

`GET https://api.sfox.com/v2/partner/sfox/quote/sell/btc/usd/2.12345678/btc`

## Buy/Sell Bitcoin

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -d '{
    "quote_id": "a5098dd0-4cb2-4256-9cb5e871fbe672d1",
    "destination": {
      "type": "address",
      "address": "1EyTupDgqm5ETjwTn29QPWCkmTCoEv1WbT"
    },
    "payment_method_id": "payment123"
  }'
```

> The result will be

```json
{
  "transaction_id": "transaction123",
  "status": "pending"
}
```

This api call executes on a buy or sell [quote](#request-a-quote) that was previously obtained.  The status indicates whether the transactions
executed successfully, or if it is pending funding of the account.  

### Flow for Pending transactions

If the status is "pending" then it is very likely that the account will be funded after the quote has expired.  In that case, you can use the [get a quote](#request-a-quote)
api call to refresh the quote associated with this transaction.  Once the account is funded, and the quote is updated, then you can call the [update transactions]()
api to execute on the new quote.

### HTTP Request

`POST https://api.sfox.com/v1/orders/buy`

### Query Parameters

Parameter | Description
--------- | -----------
quote_id | the quote to use for this transaction
destination.type | the type of destination to receive the funds. One of: payment_method, or bitcoin
destination.address | the bitcoin address to send the bitcoins to (if a buy transaction) 
destination.payment_method_id | the payment method id to receive the funds from a bitcoin sale
payment_method_id | the funding source for the transaction (if this is a buy transaction)

## Update Transaction

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction/<transaction id>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -X PATCH \
  -d '{
    "quote_id": "b3d7ce70-4d91-11e6-bfc6-14109fd9ceb9",
  }'
```

> The quote returned will have the following format

```json
{
	"transaction_id": "d0acf552-4d91-11e6-82df-14109fd9ceb9",
	"status": "completed"
}
```

This api allows you to request an updated quote from SFOX for certain amount.  This is used for both buying/selling currencies.  The api also allows you to request
a quote in both the base and quote currencies.

### HTTP Request

`GET https://api.sfox.com/v2/partner/<partner id>/quote/<action>/<base currency>/<quote currency>/<amount>/<amount currency>`


## Required Docs

Value | Description
--------- | -----------
ssn|a document showing proof of the social security number
dl|a scan of the person's driver's license
address|a scan of recent utility bill or bank account statement showing the person's address

