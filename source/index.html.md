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

SFOX uses partner IDs for tracking purposes only.

<aside class="notice">
You must replace "partner id" with your partner id provided to you by SFOX.
</aside>

# Creating a Customer Account

To create a customer account on SFOXs:

1. [Create an account](#signup) for the Customer
2. API returns an `account token` which is used for api call to identify the target account
3. Collect KYC information and [sends it to SFOX](#verify-account)
4. If required, [upload](#upload-required-verification-documents) customer documents as requested by SFOX
5. [Add a payment method](#add-payment-method) to the account
6. Once the customer and the payment method are verified, the customer is allowed to buy/sell crypto currencies with SFOX

## Signup

This api is the first step with any new user interaction with the system.
It create an account on SFOX for the customer.  Once the call is successful the API returns an `account token` which can be
used for further actions on the account.  The `account token` should be encrypted and stored securely.  [Read more](#account-fields) about the fields returned by this api.

### Request Parameters

Parameter | Description
--------- | -----------
email | the email used by the customer
username | this must be the same as the user's email
password | this is used to allow the user to recover account information

### Account Fields

Parameter | Description
--------- | -----------
token | account token to be used in all further communications regarding account.  There is no way for the partner to retrieve this token at a future time.  Partner is responsible for securily storing this token.
**verification_status** |
<ul><li>level</li><li>required_docs</li></ul> | <ul><li>see [verification levels](#verification-levels) for further information</li><li>see [required docs](#required-docs) for more info</li></ul>
can_buy | whether the user is permitted to buy from SFOX
can_sell | whether the user is permitted to sell to SFOX
limits | this describes both the user's total limits, and their available limits (after taking into account what they've used up already)

> To create an account:

```shell
# With shell, you can just pass the correct header with each request
curl "https://api.sfox.com/v2/partner/<partner name>/account" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Content-type: application/json" \
  -d '{
    "username":"user@domain.com",
    "email":"user@domain.com",
    "password":"password123"
  }'
```

> The result of creating an account:

```json
{
  "token": "<account token>",
  "account": {
    "id": "user123",
    "verification_status": {
      "level": "unverified"
    },
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

To verify an account, please provide identifying information on the user for KYC purposes.  Before a user can buy/sell currencies they need to be fully
verified.  This api allows the user to provide Personally Indetifiable Information, which we use to verify their identity.
If the information matches without issues, their verification level will be marked `verified`.  This call returns the same [account](#account-fields)
data structure as described before.

### Verification levels

Level | Description
--------- | -----------
verified|the user has been verified and no further action is needed
pending|the user needs further verification by SFOX, however no further action is needed by the user
needs_documents|SFOX needs further documentation from the user which need to be [uploaded](#upload-required-verification-documents) using our api.  The list of documents required will be returned in the [required_docs](#required-docs) property. 

If the api, however, returns a list of [required documents](#required-docs) then the user has to submit these documents for further consideration.

## Required Docs

Value | Description
--------- | -----------
ssn|a document showing proof of the social security number
id|a scan of the person's state issued identification or driver's license bearing the person's photo, full name and date of birth
address|a scan of recent utility bill or bank account statement showing the person's address
passport|a scan of the pages that include the passport number, person's photo, full name and date of birth.

## Upload Required Verification Documents

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
3. Please note that S3 expects the files to be uploaded as a PUT request. [read more](http://docs.aws.amazon.com/AmazonS3/latest/dev/PresignedUrlUploadObject.html)

### HTTP Request

`POST https://api.sfox.com/v1/upload/sign`

# Payment Methods

## Add Payment Method

```shell
curl "https://api.sfox.com/v2/account/<account id>/paymentmethod" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>:" \
  -H "Content-type: application/json" \
  -d '{
	"type": "ach",
	"ach": {
    "currency": "usd",
		"routing_number": "0123456789",
		"account_number": "0001112345667",
		"name1": "john doe",
		"name2": "jane doe",
		"nickname": "checking 1"
	}
}'
```

> Add payment method returns the payment id and status:

```json
{
  "payment_id": "payment123",
  "status": "pending"
}
```

To add a payment method to the account.  The payment method will be used when buying and selling currency.  Currently, this api only supports "ach" as a payment type.  

### Payment Status

Status | Description
---------  | -----------
pending|payment method requires verification.  User will get 2 deposits in their account and need to provide the amounts to activate the account.  Use the [payment verification](#verify-payment-method) api to finish adding the payment method
active|payment method is ready to be used
inactive|payment method is not valid and cannot be used

### HTTP Request

`POST https://api.sfox.com/v2/account/<account id>/paymentmethod`

## Verify Payment Method

```shell
curl "https://api.sfox.com/v2/account/<account id>/paymentmethod/<payment method id>/verify" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -d '{
	"amount1": 0.12,
	"amount2": 0.34
}'
```

> Returns the payment method status:

```json
{
  "payment_id": "payment123",
  "status": "active"
}
```

Use Verify Payment Method for payment methods that are in the "pending" state.  These payment methods, like ACH account, require the user to enter the 2 amounts that will be sent to their account.  When the amounts match, the payment method will be activated

### HTTP Request

`POST https://api.sfox.com/v2/account/<account id>/paymentmethod/<payment method id>/verify`

## Get Payment Methods

```shell
curl "https://api.sfox.com/v2/account/<account id>/paymentmethod"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json"
```

> Example result from Get Payment Methods:

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
curl "https://quotes.sfox.com/v1/partner/<partner id>/quote/<action>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -d '{
    "action": "buy",
    "base_currency": "btc",
    "quote_currency": "usd",
    "amount": "50",
    "amount_currency": "usd"
  }'
```

> The quote returned will have the following format

```json
{
	"quote_id": "a5098dd0-4cb2-4256-9cb5e871fbe672d1",
	"quote_amount": "3000",
	"quote_currency": "usd",
	"base_amount": "5",
	"base_currency": "btc",
  "expires_at": 1257894000000,
	"fee": "75",
	"fee_currency": "usd"
}
```

This api allows you to request a quote from SFOX for certain amount.  This is used for both buying/selling currencies.  The api also allows you to request
a quote in both the base and quote currencies.

### HTTP Request

`POST https://quotes.sfox.com/v1/partner/<partner id>/quote/<action>`

### Request Parameters

Parameter | Description
--------- | -----------
action | "buy" or "sell"

### Quote Fields

Parameter | Description
--------- | -----------
action | "buy" or "sell"
quote_amount | the amount offered by SFOX
quote_currency | the currency of the quoted amount
base_currency | the currency of the requested quote amount
base_amount | the amount requested in the quote
expires_at | expiration time of the quote, expressed as the number of seconds elapsed since January 1, 1970 UTC
fee | the fee that will be charged on top of the quoted amount
fee_currency | the currency of the fee charged
quote_id | the unique id of this quote

### Examples

#### A quote to buy $20 worth of bitcoins

`POST https://api.sfox.com/v2/partner/sfox/quote/buy/btc/usd/20/usd`

#### A quote to buy 2 bitcoins

`POST https://api.sfox.com/v2/partner/sfox/quote/buy/btc/usd/2/btc`

#### A quote to sell 2.12345678 bitcoins

`POST https://api.sfox.com/v2/partner/sfox/quote/sell/btc/usd/2.12345678/btc`

## Get Quote Details

```shell
curl "https://quotes.sfox.com/v1/quote/<quote id>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json"
```

> The quote returned will have the following format.  If the quote has expired or the quote id is invalid
then you will get a `404` back

```json
{
	"quote_id": "a5098dd0-4cb2-4256-9cb5e871fbe672d1",
  "action": "buy",
	"quote_amount": "3000",
	"quote_currency": "usd",
	"base_amount": "5",
	"base_currency": "btc",
  "expires_at": 1257894000000,
	"fee": "75",
	"fee_currency": "usd"
}
```

This api allows you to request a quote from SFOX for certain amount.  This is used for both buying/selling currencies.  The api also allows you to request
a quote in both the base and quote currencies.

### HTTP Request

`POST https://quotes.sfox.com/v1/partner/<partner id>/quote/<action>`

### Request Parameters

Parameter | Description
--------- | -----------
action | "buy" or "sell"
base currency | the base currency of the requested pair.  In the case of "btcusd", "btc" is the base currency
quote currency | the quote currency of the requested pair.  In the case of "btcusd", "usd" is the quote currency
amount | the amount requested
amount currency | the currency of the amount requested. In the case of "btcusd", it can be "btc" or "usd"

### Examples

#### A quote to buy $20 worth of bitcoins

`POST https://api.sfox.com/v2/partner/sfox/quote/buy/btc/usd/20/usd`

#### A quote to buy 2 bitcoins

`POST https://api.sfox.com/v2/partner/sfox/quote/buy/btc/usd/2/btc`

#### A quote to sell 2.12345678 bitcoins

`POST https://api.sfox.com/v2/partner/sfox/quote/sell/btc/usd/2.12345678/btc`

## Initiate Buy

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -d '{
    "action": "buy",
    "destination": {
      "type": "address",
      "address": "1EyTupDgqm5ETjwTn29QPWCkmTCoEv1WbT"
    },
    "amount": "100",
    "amount_currency": "usd",
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

This api call will initiate the buy transaction by withdrawing `amount` from the payment source specified.  Once the funds are available, the transaction's status will change to ready.  The partner needs to call the [confirm buy](#confirm-buy) api to complete the purchase.  

### HTTP Request

`POST https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction`

### Request Parameters

Parameter | Description
--------- | -----------
destination.type | the type of destination to receive the funds. For a buy order this is always `payment_method`
destination.address | the bitcoin address to send the purchased bitcoins to
payment_method_id | the funding source for the transaction (if this is a buy transaction)
amount_currency | the name of the fiat currency the user wishes to use
amount | the amount that'll be used for the purchase transaction 

## Confirm Buy

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction/<transaction id>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -X PATCH \
  -d '{
    "quote_id": "b3d7ce70-4d91-11e6-bfc6-14109fd9ceb9",
    "transaction_id": "transaction123"
  }'
```

> The quote returned will have the following format

```json
{
	"transaction_id": "d0acf552-4d91-11e6-82df-14109fd9ceb9",
	"status": "completed"
}
```

Once the transaction is ready, the user must accept a quote and confirm the transaction with the provided `quote_id`.  The amount
specified in the Buy transaction must match the quote_amount + fees in the quote, otherwise the transaction will not confirm.

### HTTP Request

`PATCH https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction/<transaction id>`

## Initiate Sell

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -d '{
    "action": "sell",
    "destination": {
      "type": "payment_method",
      "payment_method_id": "payment123"
    },
    "currency": "btc"
  }'
```

> The result will be

```json
{
  "transaction_id": "transaction123",
  "address": "3EyTupDgqm5ETjwTn29QPWCkmTCoEv1WbT",
  "status": "pending"
}
```

This api call will initiate the sell transaction by providing a deposit address for the user to transfer bitcoins to.  Once the funds are available,
and confirmed, the partner needs to call the [confirm sell](#confirm-sell) api to complete the sell transaction.  

### HTTP Request

`POST https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction/<transaction id>`

### Request Parameters

Parameter | Description
--------- | -----------
action | this is always `sell`
destination.type | the type of destination to receive the funds. One of: payment_method, or address
destination.payment_method_id | the payment method id to receive the funds from a bitcoin sale
currency | the crypto-currency the user is selling

## Confirm Sell

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction/<transaction id>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json" \
  -X PATCH \
  -d '{
    "quote_id": "b3d7ce70-4d91-11e6-bfc6-14109fd9ceb9",
    "transaction_id": "transaction123"
  }'
```

> The quote returned will have the following format

```json
{
	"transaction_id": "d0acf552-4d91-11e6-82df-14109fd9ceb9",
	"status": "completed"
}
```

Parameter | Description
--------- | -----------
action | this is `sell`
quote_id | the id of the quote to be used to complete the sell transaction.  The base amount of the quote must match that of the transaction otherwise this will fail
transaction_id | the transaction which is being confirmed

### HTTP Request

`PATCH https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction/<transaction id>`

## Get Transaction Details

```shell
curl "https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction/<transaction id>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <api_key>" \
  -H "Content-type: application/json"
```

> The result will be

```json
{
  "transaction_id": "transaction123",
  "amount": "100",
  "amount_currency": "usd",
  "status": "pending"
}
```

Use this api to get the status of a previously initiated transaction.  

### HTTP Request

`GET https://api.sfox.com/v2/partner/<partner id>/<account id>/transaction/<transaction id>`

### Response Fields

Parameter | Description
--------- | -----------
transaction_id | same as the provided transaction_id
amount | the amount of the transaction
amount_currency | the currency of the amount specified
status | one of: `pending`, `failed`, `rejected`, `ready`, `completed` 

