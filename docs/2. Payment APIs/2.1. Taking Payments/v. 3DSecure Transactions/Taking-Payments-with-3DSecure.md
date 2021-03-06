# Implementing 3DSecure

## Introduction

When using our payments API as the 3DSecure provider, the authentication is performed in-line with the existing transaction flow.  The process starts by performing a typical authorization or sale request with a desire to perform 3DSecure authentication in the request.

The authorization is then placed into a `WAITING` status until the authentication process is completed.  During authentication, the merchant may be required to update the original transaction request one or more times in order to move the process flow forward.  

At the end of the authentication process, the original transaction is updated with the authentication results and the authorization is completed.

The sequence diagrams below map to the steps in the text that follows. The first diagram is for the frictionless flow. This means the issuer does not require the cardholder to authenticate.
 
![Frictionless 3DSecure Flow](https://raw.githubusercontent.com/Fiserv-Developer/payments-api/2.0.0-docs/assets/images/3DS-Frictionless.png) 

The next diagram shows the flow when your customer has to authenticate, which means their issuer has requested they provide additional authentication details.

![Challenge 3DSecure Flow](https://raw.githubusercontent.com/Fiserv-Developer/payments-api/2.0.0-docs/assets/images/3DS-Challenge.png) 

## Step by step: How to implement 3DSecure using our API

### Frictionless Flow

#### Step 1 – Collect cardholder payment details

First, collect the card payment information (credit card number, expiration date, security code) from your customer. This can be done using the embedded form or hosted payment page.

#### Step 2 – Initiate a payment

Use either the payment card or the payment token to initiate a Primary Payment Transaction.

You can instruct the payment to use 3DSecure if you want to enforce it. The relevant RequestTypes for 3DSecure authentication are as follows:
	
- `PaymentCardPreAuthTransaction`
- `PaymentCardSaleTransaction`
- `PaymentTokenPreAuthTransaction`
- `PaymentTokenSaleTransaction`

This message needs to include the `authenticationRequest` object in the transaction request message and includes the following values:

Attribute | Description 
---------|----------
authenticationType | Default to `Secure3D21AuthenticationRequest` - this is the default 3DS 2.1 request
termURL | Indicates the callback URL where the results of the authentication process should be posted by the ACS server (this is the Access Control Server that executes the cardholder authentication).
methodNotifictionURL | If you wish to be notified about the `3DSMethod` form display completion, you need to also submit this optional element in your transaction request. The URL should be uniquely identifiable, so when there is a notification received on this URL, you should be able to map it with the corresponding transaction. This eliminates any dependency on the `Secure3dTransId` which you will receive with the `3DSMethod` form response. An easy way to ensure correct transaction mapping is to pass a transaction reference as a query string. For example: https://www.mywebshop.com/process3dSecureMethodNotificationtransactionReferenceNumber=ffffffff-ba0b-539f-8000-016b2343ad7e
challengeIndicator | In case you would like to influence which authentication flow should be used, you can submit this optional element with one of the values listed below. In case Challenge Indicator is not sent within your transaction request, the Gateway will populate the value “01” – No preference. 
challengeWindowSize | If you want to define the size of the challenge window displayed to your customers during the authentication process, you can submit this optional element with one of the values listed below.


#### Challenge indicator available values for 3DS protocol version 2.1 are:

Challenge Indicator | Description 
---------|----------
01 | No preference (You have no preference whether a challenge should be performed. This is the default value).
02 | No challenge requested (You prefer that no challenge should be performed – this means you are willing to accept liability for the transaction)
03 | Challenge requested: 3DS Requestor Preference (You prefer that a challenge should be performed – this should be set for high risk or high value transactions)
04 | Challenge requested: Mandate (There are local or regional mandates that mean that a challenge must be performed – this is only relevant in exceptional circumstances – please contact your Fiserv representative)


#### challengeWindowSize options:

Challenge Window Code | Description 
---------|----------
01 | 250 x 400 
02 | 390 x 400 
03 | 500 x 600 
04 | 600 x 400 
05 | Full screen

The payment schemes recommended using the value `05 - Full screen` only for browser-based flows. Using full screen mode in app-based flows where the authentication of the cardholder happens on a smartphone or tablet might cause time-outs and trigger an error on the issuer/ACS side.

It is highly recommended to also include also Billing and Shipping details in your transaction request to lower the risk of authentication declines. To do this, ensure you populate the objects in any of the sale or preauth requestType payloads.

The following JSON document represents an example of a Sale transaction request with minimal set of elements:

``` json
{
  "requestType": "PaymentCardSaleTransaction",
    "transactionAmount": {
      "total": "122.04",
      "currency": "USD"
      },
    "paymentMethod": {
      "paymentCard": {
        "number": "403587XXXXXX4977",
        "securityCode": "977",
        "expiryDate": {
        "month": "12",
        "year": "24"
      }   
    }
  },
  “authenticationRequest”: {
    "authenticationType": "Secure3D21AuthenticationRequest",
    "termURL": "https://www.mywebshop.com/process3dSecure",
    "methodNotificationURL": "https://www.mywebshop.com/process3dSecureMethodNotification?  transactionReferenceNumber=ffffffff-ba0b-539f-8000-016b2343ad7e",
    "challengeIndicator": "01",
    "challengeWindowSize": "01"
  }
}
```
<!-- theme: info -->
> ### Issuer 3DS Support
> 
> Not all Issuers support the collection of browser data using the 3DS Method Form.  In those cases, no data will be posted to the methodNotificationURL, and the flow should continue by posting a status of `EXPECTED_BUT_NOT_RECEIVED` – see below.

### Step 3 – 3DSecure Authentication Response

Our response will include a `3DSMethod` element, which generates a hidden iframe that helps to collect the browser data for the issuers. This information adds to the overall consumer profile and helps in identifying potentially fraudulent transactions. It also increases the probability of a frictionless, successful transaction.

You will need to include the 3DSMethod in your website as hidden iframe. No user interface screen is presented to the cardholder. 

At this point, a verification request takes place to determine if the 3DSecure system is functional and the cardholder is enrolled for 3DSecure.  If the 3DSecure system is not functioning or if the cardholder is not enrolled, the transaction will process normally and be approved or declined by the processing network.

In the above case the transaction status will appear like so: 

```javascript
transactionStatus = `APPROVED` || `DECLINED`
```

If the cardholder is verified to be enrolled in the 3DSecure program, then an `authenticationResponse` object will be included in the transaction response.

While awaiting the response the transaction will have the following transaction status:

```javascript
transactionStatus = WAITING
```

The `authenticationResponse` object will contain the following values:

Attribute | Description 
---------|----------
type | 3D_SECURE
version | 2.1
secure3DMethod/methodForm | HTML form data with hidden iFrame used to collect the web browser data for the Issuer.
secure3DMethod/secure3dTransId | A unique identifier for the transaction provided by the Issuer ACS server.

The following JSON document represents an example of a response:

```json 
{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0c80a3403e2c2def0-d-ea-28805-6810951-2",
  "ipgTransactionId": "838916029301",
  "transactionType": "SALE",
  "transactionTime": 1518811817,
  "approvedAmount": {
  "total": 122.04,
  "currency": "USD"
},
  "transactionStatus": "WAITING",
  “authenticationResponse”: {
    "type": "3D_SECURE",
    "version": "2.1",
    "secure3dMethod": {
      "methodForm": "&lt;!DOCTYPE iframe SYSTEM "about:legacy-compat"&gt;
      &lt;iframe id="tdsMmethodTgtFrame" name="tdsMmethodTgtFrame"
      style="width: 1px; height: 1px; display: none;" src="javascript:false;"
      xmlns="http://www.w3.org/1999/xhtml"&gt;
      &lt;!--.--&gt; &lt;/iframe&gt;&lt;form id="tdsMmethodForm"
      name="tdsMmethodForm"
      action=https://localhost.modirum.com:8543/dstests/ACSEmu2
      method="post"
      target="tdsMmethodTgtFrame" xmlns="http://www.w3.org/1999/xhtml"&gt;
      &lt;input type="hidden" name="3DSMethodData"
      value="eyAidGhyZWVEU1NlcnZlclRyYW5zSUQiIDogIjAwMDAwMDAwLTU2NzYtNTY2My
      04MDAwLTAwMDAw    &amp;#10;MDAwNDFhOSIsICJ0aHJlZURTTWV0aG9kTm90aWZpY2F0aW9
      uVVJMIiA6ICJodHRwczovL2xvY2Fs&amp;#10;aG9zdC5tb2RpcnVtLmNvbTo4NTQzL21kcGF5bXBpL
      01lcmNoYW50U2VydmVyP21uPVkmdHhpZD0x
      &amp;#10;NjgwOSZkaWdlc3Q9aSUyQnhhUEF5NWFOcVJRbllqNmozbWFDZlFJbTdFdjJYTm
      kwNnh6YmZNJTJG&amp;#10;R3MlM0QiIH0"/&gt; &lt;input type="hidden"
      name="threeDSMethodData"            
      value="eyAidGhyZWVEU1NlcnZlclRyYW5zSUQiIDogIjAwMDAwMDAwLTU2NzYtNTY2
      My04MDAwLTAwMDA
      w&amp;#10;MDAwNDFhOSIsICJ0aHJlZURTTWV0aG9kTm90aWZpY2F0aW9uVVJMIiA
      6ICJodHRwczovL2xvY 2Fs&amp;#10;aG9zdC5tb2RpcnVtLmNvbTo4NTQzL21kcGF5bXBpL01lcm
      NoYW50U2VydmVyP21uPVkmdHhpZD0x&amp;#10;NjgwOSZkaWdlc3Q9aSUyQnhhUEF5NWFOcV
      JRbllqNmozbWFDZlFJbTdFdjJYTmkwNnh6YmZNJTJG&amp;#10;R3MlM0QiIH0"/&gt;
      &lt;/form&gt;&lt;script type="text/javascript" 
      xmlns="http://www.w3.org/1999/xhtml"&gt;
      document.getElementById("tdsMmethodForm").submit(); &lt;/script&gt;",
      "secure3dTransId": "3ac7caa7-aa42-2663-791b-2ac05a542c4a"
    }
  }
}
```

### Step 4 – 3DS Method Notification Request & Response

The 3DSecure `methodForm` is used to provide details of the cardholder environment to the Issuer Access Control Server (ACS). This is an optional step. The `methodForm` contains the HTML for a hidden iFrame which is to be included in a merchant web page.  This will force the information to be automatically posted to the ACS server via Fiserv. The HTML information is a self-contained HTML block that does not need to be modified or posted, as it will be taken care of automatically when the page in which it is inserted is rendered.  Alternatively, this can be created on a page which never becomes visible to the merchant.

If received properly, the response data will be posted to the URL provided in the original `methodNotificationURL` field and the posted message will contain a `threeDSServerTransID` field containing the unique ACS transaction id associated with the original request.  Note that the payload for this response will contain a single element called `threeDSMethodData`.  That element will contain a base64 encoded JSON response that contains the `threeDSServerTransID` field.

Example:
```html
<form name="frm" method="POST" action="{value from methodNotificationURL}">
  <input type="hidden" name="threeDSMethodData" value="eyJ0aHJlZURTU2VydmVyVHJhbnNJRCI6IjNhYzdjYWE3LWFhNDItMjY2My03OTFiLTJhYzA1YTU0MmM0YSJ9">
</form>
```

Decoded threeDSMethodData: 
```javascript
{"threeDSServerTransID":"3ac7caa7-aa42-2663-791b-2ac05a542c4a"}
```

<!-- theme: info -->
> ### Keep a copy of `threeDSServerTransID`
>
> The threeDSServerTransID is not required for any further 3DS processing. However, it is recommended that the merchant save this value for reference to the ACS server in the future if necessary.

The merchant should wait a minimum of 10 seconds for the above POST operation to complete and then determine the `methodNotificationStatus` as follows:

Status | Description 
---------|----------
RECEIVED | You have submitted the element `methodNotificationURL` in the initial Sale transaction request and have received the notification from ACS within 10 seconds, you will receive HTTP POST message from ACS, which will contain  a unique transaction identifier represented by `secure3dTransId`
EXPECTED_BUT_NOT_RECEIVED | You have submitted the element `methodNotificationURL` in the initial Sale transaction request and have not received the notification from ACS within 10 seconds
NOT_EXPECTED | You have NOT submitted the element `methodNotificationURL` in the initial Sale transaction request

### Step 5(F) – Request to continue 3DS Authentication

Once the 3DS Method call has been completed, you need to notify the gateway that the authentication process can continue by submitting the `methodNotificationStatus` element with the values based on corresponding conditions from the 3DSecure Method Form above.  This is done by performing a PATCH operation on the original transaction.

The merchant may also include the optional cardholder billing address and the security code at this time.

The following JSON document represents an example of a request to be sent after `3DSMethod` form display:

```json
{
  "authenticationType": "Secure3D21AuthenticationUpdateRequest",
  "storeId": "12345500000",
  "billingAddress": {
  "company": "Test Company",
  "address1": "5565 Glenridge Conn",
  "address2": "Suite 123",
  "city": "Atlanta",
  "region": "Georgia",
  "postalCode": "30342",
  "country": "USA"
},
  "securityCode": "123",
  "methodNotificationStatus": "RECEIVED"
}
```

### Step 6(F) – Final 3DS Response

When it is determined that a frictionless flow can be performed (i.e. the customer has been fully authenticated by their bank without the need for further information), the transaction authorization is processed and the 3DSecure process is completed.

The transaction response contains a `Secure3DResponse` object and the transaction is either approved or declined.

```javscript
transactionStatus = APPROVED or DECLINED
```

The `Secure3DResponse` object will contain the following field: `responseCode3dSecure`

The following JSON document represents an example of a response you receive from the API indicating, that the authorisation has been successful:

```json
{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0c80a3403e2c2def0-d-ea-28805-6810951-2",
  "ipgTransactionId": "838916029301",
  "transactionType": "SALE",
  "transactionTime": 1518811817,
  "approvedAmount": {
    "total": 122.04,
    "currency": "USD"
    },
  "transactionStatus": "APPROVED",
  "schemeTransactionId": "019078743804756",
  "processor": {
    "responseCode": "00",
    "responseMessage": "APPROVED",
    "authorizationCode": "OK7118"
    },
    "secure3dResponse": {
      "responseCode3dSecure": "1"
    }
  }
} 
```

## 3DSecure Challenge Flow

The challenge flow is triggered, when the transaction is not considered as low risk or when the Issuer requires additional authentication by the cardholder. The whole process starts with an initial Authorisation or Sale transaction request through the step where 3DS Method is displayed, as described in Steps 1 through 4 above.

### Step 5(C) – Request to continue 3DS Authentication

Once the 3DS Method call has been completed, you need to notify the gateway that the authentication process can continue by submitting the `methodNotificationStatus` element with the values based on corresponding conditions from the 3DSecure Method Form above.  This is done by performing a PATCH operation on the original transaction.

The merchant may also include the optional cardholder billing address and the security code at this time.

The following JSON document represents an example of a request to be sent after 3DSMethod form display:

```json 
{
  "authenticationType": "Secure3D21AuthenticationUpdateRequest",
  "storeId": "12345500000",
  "billingAddress": {
  "company": "Test Company",
  "address1": "5565 Glenridge Conn",
  "address2": "Suite 123",
  "city": "Atlanta",
  "region": "Georgia",
  "postalCode": "30342",
  "country": "USA"
},
  "securityCode": "123",
  "methodNotificationStatus": "RECEIVED"
}
```

### Step 6(C) – Fiserv responds to continue 3DS Authentication

For the challenge flow, the transaction status will be returned as follows:

```javascript
transactionStatus = "WAITING"
```

The response will contain an `authenticationResponse` object with the following fields:

Field | Description
---------|----------
type | `3D_SECURE` 
version | `2.1` 
acsURL | The URL to which the `cReq` and `sessionData` values should be posted for the cardholder challenge to take place.
termURL | The URL where the results of the authentication will be posted.
cReq | An encoded request message returned from the ACS server.
sessionData | An encoded list of session parameters to be used for authentication.  Note that this value may not always be provided.

The following JSON document represents an example of a response:

```json 
{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0c80a3403e2c2def0-d-ea-28805-6810951-2",
  "ipgTransactionId": "838916029301",
  "transactionType": "SALE",
  "transactionTime": 1518811817,
  "approvedAmount": {
  "total": 122.04,
  "currency": "USD"
},
  "transactionStatus": "WAITING",
  "authenticationResponse": {
    "type": "3D_SECURE",
    "version": "2.1",
    "params": {
      "acsURL": "https://3ds-acs.test.modirum.com/mdpayacs/pareq",
      "termURL": "https://www.mywebshop.com/process3dSecure/",
      "cReq": "ewogICAiYWNzVHJhbCIgOiA...wMDAtMDAwMDAwMDA0MWE5Igp9",
      "sessiondata": "50F2156E03083CA665BCB4.."
      }
    }
  }
}
```

### Step 7(C) – Cardholder Challenge Indication

In the next step you need to POST data to the indicated `acsURL` usually implemented as an auto-submit form. This needs to be implemented within your website. The cardholders will be redirected to the ACS and presented with the UI to collect the authentication details - for example enter one-time-password or perform authentication using their banking app (out-of-band authentication). After the authentication is completed, the consumer is redirected back to your webpage.

The merchant posts the `cReq` value and the `sessionData` value to the URL specified in the `acsURL` field.

> ### Field name mapping
>
> This information is posted using the following field names:
`CReq` = The entire base64 encoded `cReq` Challenge Request message as obtained above.
`threeDSSessionData` = The entire base64 encoded `sessionData` message as obtained above.

Example:

```html
<form name="frm" method="POST" action="https://3ds-acs.test.modirum.com/mdpayacs/pareq ">
  <input type=”hidden” name=”CReq” value=” ewogICAiYWNzVHJhbCIgOiA...wMDAtMDAwMDAwMDA0MWE5Igp9”>
  <input type=”hidden” name=”threeDSSessionData” value=”50F2156E03083CA665BCB4..”>
</form>
```

When the authentication is completed, an authentication response will be posted to the URL specified in the `termURL` field.

<!-- theme: info -->
> Field name mapping
>
> This information is posted using the following field names:
`CRes` = The base64 encoded Challenge Response message from the Issuer ACS server.
`threeDSSessionData` = The base64 encoded session data from the Issuer ACS server.

Example:

```html
<form name="frm" method="POST" action=" https://www.mywebshop.com/process3dSecure">
  <input type="hidden" name="CRes" value="ewogICAiYWNzUmVmZX…Fuc1N0YXR…IKfQ==">
  <input type=”hidden” name=”threeDSSessionData” value=”50F2156E03083CA665BCB4..”>
</form>
```

### Step 8(C) - Send request to complete transaction

After you receive the data from the ACS you need to submit the values to us in the `cRes` element together with the reference to the original transaction.

The merchant sends a PATCH request to the original transaction and includes the following values:

Field | Description
---------|----------
 authenticationType | `Secure3D21AuthenticationUpdateRequest`
 acsResponse/cRes | The `CRes` data posted to the `termURL` by the ACS server.

The merchant may also include the optional cardholder billing address and the security code at this time.

The following JSON document represents an example of a request with `cRes` element:

```json 
{
  "authenticationType": "Secure3D21AuthenticationUpdateRequest",
  "storeId": "12345500000",
  "billingAddress": {
    "company": "Test Company",
    "address1": "5565 Glenridge Conn",
    "address2": "Suite 123"
    "city": "Atlanta",
    "region": "Georgia",
    "postalCode": "30342",
    "country": "USA"
  },
  "securityCode": "123",
  "acsResponse": {
    "cRes": "ewogICAiYWNzUmVmZX…Fuc1N0YXR…IKfQ=="
  }
}
```

### Step 9(C) – Final response of 3DS 

Since this transaction was initiated as a `Sale`, the authorization is performed as part of this final step if the authentication was successful.

The transaction response contains a `Secure3DResponse` object and the transaction is either approved or declined.

The `Secure3DRespnse` object will contain the following field: `responseCode3dSecure`

The following JSON document represents an example of a response you receive indicating that the authorization has been successful:

```json
{
  "clientRequestId": "30dd879c-ee2f-11db-8314-0800200c9a66",
  "apiTraceId": "rrt-0c80a3403e2c2def0-d-ea-28805-6810951-2",
  "ipgTransactionId": "838916029301",
  "transactionType": "SALE",
  "transactionTime": 1518811817,
  "approvedAmount": {
    "total": 122.04,
    "currency": "USD"
  },
  "transactionStatus": "APPROVED",
  "schemeTransactionId": "019078743804756",
  "processor": {
    "responseCode": "00",
    "responseMessage": "APPROVED",
    "authorizationCode": "OK7118"
  },
  "secure3dResponse": {
    "responseCode3dSecure": "1"
  }
}
```

### Fallback to 3DS 1.0 Protocol 

For cases, where Issuers do not support 2.x 3DS protocol version yet, the gateway provides an option to “downgrade” to 3DS 1.0 authentication instead.

<!-- theme: warning -->

> ### 3DS Fallback
>
> The fallback flow is the same as the challenge flow above, except that the values returned by the ACS server are different, resulting in a different set of values to be sent on the final request.

