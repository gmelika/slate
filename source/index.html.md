---
title: Partner API Reference

language_tabs:
  - shell

toc_footers:
  - <a href='mailto:support@sfox.com'>Sign Up for a Partner Account</a>
  - <a href='mailto:support@sfox.com'>Need help? Email us</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the SFOX API! The API allows you to offer buy/sell services to your customers through the SFOX platform.

SFOX uses partner IDs for tracking purposes only.

<aside class="notice">
You must replace "partner id" with your partner id provided to you by SFOX.  "partner name" is used for analytics purposes, please use your company name in all requests
</aside>

# Sandbox

A public sandbox is available for testing against our API. The sandbox provides all of the functionality of the production envirnoment while not executing real trades.

API keys are separate from production.

## Sandbox URLs

When testing your API connectivity, make sure to use the following URLs.

**Partner API**  
**https://api.staging.sfox.com**

**Quotes API**  
**https://quotes.staging.sfox.com**


# Creating a Customer Account

To create a customer account on SFOXs:

1. [Create an account](#signup) for the Customer
2. API returns an `account token` which is used for api call to identify the target account
3. Collect KYC information and [sends it to SFOX](#verify-account)
4. If required, [upload](#upload-required-verification-documents) customer documents as requested by SFOX
5. [Add a payment method](#add-payment-method) to the account
6. Once the customer and the payment method are verified, the customer is allowed to buy/sell crypto currencies with SFOX

## Signup

> To create an account:

```shell
# With shell, you can just pass the correct header with each request
curl "https://api.staging.sfox.com/v2/partner/<partner name>/account" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Content-type: application/json" \
  -d '{
    "username":"user@domain.com",
    "password":"password123",
    "user_data": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImRldmVsb3BlcnNAc2ZveC5jb20iLCJwaG9uZV9udW1iZXIiOiIrMTMxMDU1NTExMTEifQ.SCeI3WAV2rLds5nV89EBtgxGjAV-iCdf-0dNdIEVNUo65hoQKJ6aCgNktzc7Ka57qcWgK6qWDMkAEy9ZM7XzZQ"
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
```

This api is the first step with any new user interaction with the system.
It create an account on SFOX for the customer.  Once the call is successful the API returns an `account token` which can be
used for further actions on the account.  The `account token` should be encrypted and stored securely.  [Read more](#account-fields) about the fields returned by this api.

### Request Parameters

Parameter | Description
--------- | -----------
username | this must be the same as the user's email
password | this is used to allow the user to recover account information
user_data | a [jwt](https://jwt.io/) object - signed by the partner's key - that containes the verified `phone_number` and `email` of the user

### Account Fields

Parameter | Description
--------- | -----------
token | account token to be used in all further communications regarding account.  There is no way for the partner to retrieve this token at a future time.  Partner is responsible for securily storing this token.
**verification_status** |
<ul><li>level</li><li>required_docs</li></ul> | <ul><li>see [verification levels](#verification-levels) for further information</li><li>see [required docs](#required-docs) for more info</li></ul>
can_buy | whether the user is permitted to buy from SFOX
can_sell | whether the user is permitted to sell to SFOX
limits | this describes both the user's total limits, and their available limits (after taking into account what they've used up already)

## Get Account Info

> To view an account:

```shell
# With shell, you can just pass the correct header with each request
curl "https://api.staging.sfox.com/v2/partner/<partner name>/account" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
  -H "Content-type: application/json"
```

> The result of creating an account:

```json
{
  "account": {
    "id": "user123",
    "verification_status": {
      "level": "unverified"
    },
    "can_buy": false,
    "can_sell": false,
    "limits": {
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
```

This api will return account information for the provided `account token`

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

## Verify Account

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/account/verify" \
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
  "identity": {
    "type": "driver_license",
    "number": "b1234567",
    "state": "CA",
    "country": "US"
  }
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
```

To verify an account, please provide identifying information on the user for KYC purposes.  Before a user can buy/sell currencies they need to be fully
verified.  This api allows the user to provide Personally Indetifiable Information, which we use to verify their identity.
If the information matches without issues, their verification level will be marked `verified`.  This call returns the same [account](#account-fields)
data structure as described before.

### Request Parameters

Level | Description
--------- | -----------
firstname | The legal first name of the customer.
middlename | The legal middle name of the customer.
lastname | The legal last name of the customer.
street1 | The primary street address of the customer.
street2 | The second address line typically used for apartment or suite numbers.
city | The city name of the customer.
state | The state name of the customer.
zipcode | The zip code of the customer. Also known as a postal code.
country | The country of the customer. Should be of the ISO code form.  Currently this limited to `US`
birth_day | The integer day of birth of the person.
birth_month | The integer month of birth of the person.
birth_year | The integer year of birth of the person.
identity | This is an object containing the following fields: <br/> <ul><li>type: see [Identity Types](#identity-types)</li><li>number: The number associated with the form of identification used.</li><li>country: The country associated with the form of identification used.  Currently this must match the address country. otherwise the verification will fail</li></ul>


### Identity Types

Level | Description
--------- | -----------
drivers_license | Driver's license 
passport | Passport
ssn | Social Security Number


### Verification levels

Level | Description
--------- | -----------
unverified|the user has not attempted verification yet.  this is the default level for brand new accounts
verified|the user has been verified and no further action is needed
pending|the user needs further verification by SFOX, however no further action is needed by the user
needs_documents|SFOX needs further documentation from the user which need to be [uploaded](#upload-required-verification-documents) using our api.  The list of documents required will be returned in the [required_docs](#required-docs) property. 

If the api, however, returns a list of [required documents](#required-docs) then the user has to submit these documents for further consideration.

### Testing

In the sandbox envirnoment you can test each one of these levels on demand.  To do that simply override the `street2` field with one of the following values to get the desired result

street2 | Result
--------- | -----------
testing-docs-id|the user will be required to upload proof of id
testing-docs-address|the user will be required to upload proof of address
testing-docs-all|the user will be required to upload both proof of id and proof of address
testing-user-block|the user will be marked as blocked and will not be allowed to buy/sell


## Required Docs

Value | Description
--------- | -----------
ssn|a document showing proof of the social security number
id|a scan of the person's state issued identification or driver's license bearing the person's photo, full name and date of birth
address|a scan of recent utility bill or bank account statement showing the person's address
passport|a scan of the pages that include the passport number, person's photo, full name and date of birth.

## Upload Required Verification Documents

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/uploads/sign" \
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

`POST https://api.staging.sfox.com/v1/upload/sign`

# Payment Methods

## Add Payment Method

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/payment-methods" \
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
        "nickname": "checking 1",
        "type": "checking"
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

### Request Fields

Field | Description
---------  | -----------
type | currently this is limited to `ach` only
ach | an object describing the bank account to use for ACH transactions. See [ACH Account](#ach-account)

### ACH Account

Field | Description
---------  | -----------
routing_number | the bank's routing (ABA) number
account_number | the customer's account number with the bank
name1 | the full name on the account
name2 | if this is a joint account, the secondary name on the account
nickname | a name for the customer to identify this account on SFOX
type | one of: `checking`, or `savings`

### Payment Status

Status | Description
---------  | -----------
pending|payment method requires verification.  User will get 2 deposits in their account and need to provide the amounts to activate the account.  Use the [payment verification](#verify-payment-method) api to finish adding the payment method
active|payment method is ready to be used
inactive|payment method is not valid and cannot be used

### HTTP Request

`POST https://api.staging.sfox.com/v2/partner/<partner name>/payment-methods`

## Verify Payment Method

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/payment-methods/verify" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
  -H "Content-type: application/json" \
  -d '{
    "payment_method_id": "payment123",
    "amount1": 0.12,
    "amount2": 0.34
}'
```

> Returns the payment method status:

```json
{
  "payment_method_id": "payment123",
  "status": "active"
}
```

Use Verify Payment Method for payment methods that are in the "pending" state.  These payment methods, like ACH account, require the user to enter the 2 amounts that will be sent to their account.  When the amounts match, the payment method will be activated

<aside class="notice">
In our sandbox envirnoment the verification amounts will always be <b>0.01</b> and <b>0.02</b>
</aside>

### Request Fields

Field | Description
---------  | -----------
amount1 | one of the deposit amounts initiated by SFOX (order is not important) 
amount2 | the other deposit amount initiated by SFOX

### HTTP Request

`POST https://api.staging.sfox.com/v2/partner/<partner name>/payment-methods/verify`

## Get Payment Methods

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/payment-methods"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
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

`GET https://api.staging.sfox.com/v2/partner/<partner name>/account/payment-methods`

# Trade

## Request a Quote

```shell
curl "https://quotes.staging.sfox.com/v1/partner/<partner name>/quote/<action>" \
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Content-type: application/json" \
  -d '{
    "action": "buy",
    "base_currency": "btc",
    "quote_currency": "usd",
    "amount": "3000",
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
    "expires_on": 1257894000000,
    "fee_amount": "75",
    "fee_currency": "usd"
}
```

This api allows you to request a quote from SFOX for certain amount.  This is used for both buying/selling currencies.  The api also allows you to request
a quote in both the base and quote currencies.

### HTTP Request

`POST https://quotes.staging.sfox.com/v1/partner/<partner name>/quote/<action>`

### Request Parameters

Parameter | Description
--------- | -----------
action | "buy" or "sell"
base currency | the base currency of the requested pair.  In the case of "btcusd", "btc" is the base currency
quote currency | the quote currency of the requested pair.  In the case of "btcusd", "usd" is the quote currency
amount | the amount requested
amount currency | the currency of the amount requested. In the case of "btcusd", it can be "btc" or "usd"

### Response Fields

Parameter | Description
--------- | -----------
action | "buy" or "sell"
quote_currency | the quote currency (in the case of `btcusd` this will be `usd`)
quote_amount | the quote amount
base_currency | the base currency (in the case of `btcusd` this will be `btc`)
base_amount | the base amount
expires_on | expiration time of the quote, expressed as the number of seconds elapsed since January 1, 1970 UTC
fee_amount | this is the fee charged, which has already been included in the quote_amount
fee_currency | the currency of the fee charged
quote_id | the unique id of this quote

### Examples

#### A quote to buy $20 worth of bitcoins

`POST https://quotes.staging.sfox.com/v2/partner/sfox/quote/buy/btc/usd/20/usd`

#### A quote to buy 2 bitcoins

`POST https://quotes.staging.sfox.com/v2/partner/sfox/quote/buy/btc/usd/2/btc`

#### A quote to sell 2.12345678 bitcoins

`POST https://quotes.staging.sfox.com/v2/partner/sfox/quote/sell/btc/usd/2.12345678/btc`

## Get Quote Details

```shell
curl "https://quotes.staging.sfox.com/v1/quote/<quote id>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
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
    "expires_on": 1257894000000,
    "fee_amount": "75",
    "fee_currency": "usd"
}
```

This api allows you to request a quote from SFOX for certain amount.  This is used for both buying/selling currencies.  The api also allows you to request
a quote in both the base and quote currencies.

### HTTP Request

`POST https://quotes.staging.sfox.com/v1/partner/<partner name>/quote/<action>`


### Examples

#### A quote to buy $20 worth of bitcoins

`POST https://api.staging.sfox.com/v2/partner/sfox/quote/buy/btc/usd/20/usd`

#### A quote to buy 2 bitcoins

`POST https://api.staging.sfox.com/v2/partner/sfox/quote/buy/btc/usd/2/btc`

#### A quote to sell 2.12345678 bitcoins

`POST https://api.staging.sfox.com/v2/partner/sfox/quote/sell/btc/usd/2.12345678/btc`

## Buy

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/transaction"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
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
  "transaction_id": "e8ae8ed9-e7f4-43b3-8d87-417c1f19b0f9",
  "status": "pending"
}
```

This api call will initiate the buy transaction by withdrawing `amount` from the quote specified.  Once the funds are available, the transaction's status will change to completed, and the bitcoin will be delivered once available.

### HTTP Request

`POST https://api.staging.sfox.com/v2/partner/<partner name>/transaction`

### Request Parameters

Parameter | Description
--------- | -----------
destination.type | the type of destination to receive the funds. For a buy transaction this is always `address`
destination.address | the bitcoin address to send the purchased bitcoins to
payment_method_id | the funding source for the transaction (if this is a buy transaction). default: the primary funding source on the account
quote_id | the quote_id which is the basis of this transaction

### Response Fields

Parameter | Description
--------- | -----------
transaction_id | the id for this transaction
status | possible values: `pending`, `completed`, `failed`, `rejected`

## Sell

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/transaction"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
  -H "Content-type: application/json" \
  -d '{
    "quote_id": "5e15e347-b7d2-4b4f-8721-854fa9bb4a99",
    "destination": {
      "type": "payment_method",
      "payment_method_id": "payment123"
    }
  }'
```

> The result will be

```json
{
  "transaction_id": "8bc2172a-5b5f-11e6-b9e0-14109fd9ceb9",
  "address": "3EyTupDgqm5ETjwTn29QPWCkmTCoEv1WbT",
  "status": "pending"
}
```

This api call will initiate the sell transaction by providing a deposit address for the user to transfer bitcoins to.  Once the bitcoin is transfered and confirmed,
the funds will be delivered to the `payment_method_id` specified.  

### HTTP Request

`POST https://api.staging.sfox.com/v2/partner/<partner name>/transaction/<transaction id>`

### Request Parameters

Parameter | Description
--------- | -----------
destination.type | the type of destination to receive the funds. For a sell transaction this has to be `payment_method`
destination.payment_method_id | the payment method id to receive the funds from a bitcoin sale
quote_id | the quote_id which is the basis of this transaction

### Response Fields

Parameter | Description
--------- | -----------
transaction_id | the same transaction_id used in the request
address | the address the user needs to send the bitcoins to complete this transaction
status | one of: `pending`, `failed`, `rejected`, `ready`, `completed` 

# Transactions

## Get Transactions

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/transaction"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
  -H "Content-type: application/json"
```

> The result will be

```json
[
  {
    "action": "buy",
    "transaction_id": "579afee4-5b5f-11e6-8e25-14109fd9ceb9",
    "amount": "100",
    "amount_currency": "usd",
    "status": "pending"
  },
  {
    "action": "buy",
    "transaction_id": "579afee4-5b5f-11e6-8e25-14109fd9ceb9",
    "amount": "100",
    "amount_currency": "usd",
    "quote_id": "07c89dda-5b60-11e6-87d5-14109fd9ceb9",
    "base_currency": "btc",
    "quote_currency": "usd",
    "base_amount": "0.16666667",
    "quote_amount": "100",
    "fee_amount": "1",
    "fee_currency": "usd",
    "status": "completed"
  },
  {
    "action": "sell",
    "transaction_id": "b30ef9d2-5b60-11e6-8ac6-14109fd9ceb9",
    "amount": "1",
    "amount_currency": "btc",
    "quote_id": "b8b9d168-5b60-11e6-8c1d-14109fd9ceb9",
    "base_currency": "btc",
    "quote_currency": "usd",
    "base_amount": "1",
    "quote_amount": "590",
    "fee_amount": "6",
    "fee_currency": "usd",
    "status": "completed"
  }
]
```

Use this api to get a list of all transaction.  

### HTTP Request

`GET https://api.staging.sfox.com/v2/partner/<partner name>/transaction/<transaction id>`

### Response Fields

This api will return an array of [transaction details](#transaction-fields) sorted from newest to oldest

## Get Transaction Details

```shell
curl "https://api.staging.sfox.com/v2/partner/<partner name>/transaction/<transaction id>"
  -H "X-SFOX-PARTNER-ID: <partner id>" \
  -H "Authorization: Bearer <account token>" \
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

`GET https://api.staging.sfox.com/v2/partner/<partner name>/transaction/<transaction id>`

### Transaction Fields

Parameter | Description
--------- | -----------
transaction_id | same as the provided transaction_id
amount | the amount of the transaction
amount_currency | the currency of the amount specified
quote_id | the quote_id used to complete this transaction (if any) 
base_currency | the base currency of the quote
base_amount | the base amount of the quote
quote_currency | the quote currency of the quote used
quote_amount | the quote amount
fee_amount | the fee charged for this transaction
fee_currency | the currency of the fee charged
status | one of: `pending`, `failed`, `rejected`, `ready`, `completed` 

