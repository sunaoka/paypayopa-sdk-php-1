# Paypay SDK - PHP

[![License](https://img.shields.io/:license-apache-orange.svg)](https://opensource.org/licenses/Apache-2.0)
[![Packagist Version](https://img.shields.io/packagist/v/paypayopa/php-sdk)](https://packagist.org/packages/paypayopa/php-sdk)
[![Build Status](https://travis-ci.org/paypay/paypayopa-sdk-php.svg?branch=master)](https://travis-ci.org/paypay/paypayopa-sdk-php)
[![Coverage Status](https://coveralls.io/repos/github/paypay/paypayopa-sdk-php/badge.svg)](https://coveralls.io/github/paypay/paypayopa-sdk-php)
[![Maintainability](https://api.codeclimate.com/v1/badges/7f020ad8816dc9f64f6f/maintainability)](https://codeclimate.com/github/paypay/paypayopa-sdk-php/maintainability)
[![Black Duck Security Risk](https://copilot.blackducksoftware.com/github/repos/paypay/paypayopa-sdk-php/branches/master/badge-risk.svg)](https://copilot.blackducksoftware.com/github/repos/paypay/paypayopa-sdk-php/branches/master)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fpaypay%2Fpaypayopa-sdk-php.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fpaypay%2Fpaypayopa-sdk-php?ref=badge_shield)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=paypay_paypayopa-sdk-php&metric=alert_status)](https://sonarcloud.io/dashboard?id=paypay_paypayopa-sdk-php)
[![Packagist Downloads](https://img.shields.io/packagist/dm/paypayopa/php-sdk)](https://packagist.org/packages/paypayopa/php-sdk)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/4d981c534bab4f839b5492962f7f0a58)](https://app.codacy.com/gh/paypay/paypayopa-sdk-php?utm_source=github.com&utm_medium=referral&utm_content=paypay/paypayopa-sdk-php&utm_campaign=Badge_Grade_Dashboard)
[![BCH compliance](https://bettercodehub.com/edge/badge/paypay/paypayopa-sdk-php?branch=master)](https://bettercodehub.com/)

PHP Class for interacting with the Paypay API
This is the quickest way to integrate PayPay payment services, primarily meant for merchants who wish to perform interactions with the Paypay API programatically.
With PayPay's OPA SDK, you can build a custom payment checkout process to suit your unique business needs and branding guidelines.

## Integrating with PayPay's Open Payment API (OPA)

## Prerequisites

Before integrating with the SDK, run through this checklist:

* Understand the payment flow
* Sign up for a PayPay developer/merchant account
* Generate the API keys from the Developer Panel. Use the sandbox API keys to test out the integration

## Minimum required software requirements

To use the Paypay OPA PHP SDK you need:
- A server compute environment (local machines, docker containers, VPS or dedicated servers, cloud infrastructure etc. )
- A web server to serve your API responses and html documents.
- PHP version 7.x interpreter to execute your backend code.
- Composer to manage your dependencies(recommended) or a release version from this repo to manually maintain your dependencies. 

## HMAC Signature Verification

Signature verification is a mandatory step to ensure that the callback is sent by PayPay and the payment is received from an authentic source.

### Generate a Signature

The PayPay signature, returned to you on successful payment, can be generated by your system and verified as follows:

* Start by hashing the body and content-type with MD5 algorithm
* - Note : If there is no request body, for instance, the HTTP GET method case, no need of generating MD5. Instead hash value is set as "empty".
* The value of authHeader is passed in HttpHeader. AUTHORIZATION. The authHeader will decode back to the data added and with the HTTP request object and based on data available for api-key in the system, we will recreate the SHA256("key", requestParams) which gives macData. This macData is verified against the value passed in the header.

For the complete step-by-step explanation refer the link [here](https://www.paypay.ne.jp/opa/doc/v1.0/webcashier#tag/Api-Authentication)

### Composer

To install the bindings via [Composer](http://getcomposer.org/), run the following command in your shell :

``` sh
composer require paypayopa/php-sdk
```

## Getting Started

You need to setup your key and secret using the following:

``` php
include('PATH_TO_SDK_FOLDER/Client.php');

$client = new Client([
    'API_KEY' => 'YOUR_API_KEY',
    'API_SECRET'=>'YOUR_API_SECRET',
	'MERCHANT_ID'=>'YOUR_MERCHANT_ID'
]);

```
### [Note:] Setter chaining in request payload classes
In the examples below methods are written one after the other for the sake of your understanding. However you can save a few keystrokes by chaining multiple setter functions like so:
```php
use PayPay\OpenPaymentAPI\Models\CreateQrCodePayload;
$cqcp = new CreateQrCodePayload();
$cqcp->setMerchantPaymentId('Test123')->setRequestedAt()->setCodeType();
```

## Dynamic QR Code

### Create a dynamic QR Code to receive payments.

``` php
use PayPay\OpenPaymentAPI\Models\CreateQrCodePayload;
use PayPay\OpenPaymentAPI\Models\OrderItem;
/*
.....initialize SDK
*/
// setup payment object
$CQCPayload = new CreateQrCodePayload();

// Set merchant pay identifier
$CQCPayload->setMerchantPaymentId("YOUR_TRANSACTION_ID");

// Log time of request
$CQCPayload->setRequestedAt();
// Indicate you want QR Code
$CQCPayload->setCodeType("ORDER_QR");

// Provide order details for invoicing
$OrderItems = [];
$OrderItems[] = (new OrderItem())
    ->setName('Cake')
    ->setQuantity(1)
    ->setUnitPrice('amount' => 20, 'currency' => 'JPY']);
$CQCPayload->setOrderItems($OrderItems);

// Save Cart totals
$amount = [
    "amount" => 1,
    "currency" => "JPY"
];
$CQCPayload->setAmount($amount);
// Configure redirects
$CQCPayload->setRedirectType('WEB_LINK');
$CQCPayload->setRedirectUrl($_SERVER['SERVER_NAME']);

// Get data for QR code
$response = $client->code->createQRCode($CQCPayload);

$data = $response['data'];

```

    For a list of params refer to the API guide :
    https://www.paypay.ne.jp/opa/doc/v1.0/dynamicqrcode#operation/createQRCode

### Delete a particular Dynamic QR Code

``` php
/*
.....initialize SDK
*/

$response =  $client->code->deleteQRCode('ID_OF_CODE');
$data = $response['data'];
```

### Fetch a particular QR CODE payment detail

``` php
/*
.....initialize SDK
*/

$response =  $client->code->getPaymentDetails('MERCHANT_PAYMENT_ID');
$data = $response['data'];
```

### Cancel a payment

``` php
/*
.....initialize SDK
*/

$response =  $client->code->cancelPayment('MERCHANT_PAYMENT_ID');
$data = $response['data'];
```

### Get User Authorization URL

``` php

/*
.....initialize SDK
*/

use PayPay\OpenPaymentAPI\Models\AccountLinkPayload;
$payload = new AccountLinkPayload();
$payload
    ->setScopes(["direct_debit"])
    ->setRedirectUrl("https://merchant.domain/test/callback")
    ->setReferenceId(uniqid("TEST123"));
$resp = $client->user->createAccountLinkQrCode($payload);
$url=$resp['data']['linkQRCodeURL'];
echo $url.'   ';
$nonce = $payload->getNonce();
/*
.... store nonce for later integrity checks in session or DB
*/
```

### Decode user authorization from token

The PayPay authorization system will redirect user back to your site with a JWT token in the `responseToken` URL parameter. 
``` php
/*
.....initialize SDK
*/
$token = $_GET['responseToken'];
$authorization = $client->user->decodeUserAuth($token);
/*
...fetch stored nonce for integrity check
*/
$userAuthorizationId = false;
if ($authorization['result']==='succeeded' && $authorization['nonce']===$fetchedNonce){
    $userAuthorizationId = $authoriresponseTokenzation['userAuthorizationId'] 
}
```

### Capture payment details

``` php
use PayPay\OpenPaymentAPI\Models\CapturePaymentAuthPayload;
/*
.....initialize SDK
*/
// setup payment object
$CAPayload = new CapturePaymentAuthPayload();
$CAPayload->setMerchantPaymentId("YOUR_TRANSACTION_ID");
$amount = [
    "amount" => 1,
    "currency" => "JPY"
];
$CAPayload->setAmount($amount);

$CAPayload->setMerchantCaptureId("MERCHANT_CAPTURE_ID")
$CAPayload->setRequestedAt();
$CAPayload->setOrderDescription("ORDER_DESCRIPTION")
$CAPayload->setCodeType("ORDER_QR");
$response = $client->payment->capturePaymentAuth($CAPayload);

$data = $response['data'];

```

    For a list of params refer to the API guide :
    https://www.paypay.ne.jp/opa/doc/v1.0/dynamicqrcode#operation/capturePaymentAuth

### Fetch a particular Direct Debit payment detail

``` php
/*
.....initialize SDK
*/

$response =  $client->payments->getPaymentDetails('MERCHANT_PAYMENT_ID');
$data = $response['data'];
```
### Revert payment

``` php
use PayPay\OpenPaymentAPI\Models\RevertAuthPayload;
/*
.....initialize SDK
*/
// setup payment object
$RAPayload = new RevertAuthPayload();
$RAPayload->setMerchantRevertId("UNIQUE_REVERT_ID");
$RAPayload->setPaymentId("MERCHANT_PAYMENT_ID");
$RAPayload->setRequestedAt();
$RAPayload->setReason("REASON_FOR_REFUND");
     
 $response = $client->payment->revertAuth($RAPayload)
```

    For a list of params refer to the API guide :
    https://www.paypay.ne.jp/opa/doc/v1.0/dynamicqrcode#operation/revertAuth

### Refund payment

``` php
use PayPay\OpenPaymentAPI\Models\RefundPaymentPayload;
/*
.....initialize SDK
*/

// setup payment object
$RPPayload = new RefundPaymentPayload();
$RPPayload->setMerchantRefundId('MERCHANT_REFUND_ID');
$RPPayload->setMerchantPaymentId('MERCHANT_PAYMENT_ID');
$amount = [
    "amount" => 1,
    "currency" => "JPY"
];
$RPPayload->setAmount($amount);
$RPPayload->setRequestedAt();
$RPPayload->setReason("Refunds test");
$response = $client->refund->refundPayment($RPPayload);
$data = $response['data'];
```

    For a list of params refer to the API guide :
    https://www.paypay.ne.jp/opa/doc/v1.0/dynamicqrcode#operation/refundPayment

### Fetch refund status and details

``` php
/*
.....initialize SDK
*/
$response=$client->refund->getRefundDetails('UNIQUE_REFUND_ID');
$data = $response['data'];
```

    For a list of params refer to the API guide :
    https://www.paypay.ne.jp/opa/doc/v1.0/dynamicqrcode#operation/getRefundDetails


## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fpaypay%2Fpaypayopa-sdk-php.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fpaypay%2Fpaypayopa-sdk-php?ref=badge_large)
