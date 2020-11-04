# Implementing Wallet Payment Methods

This section provides a step-by-step for implementing Apple Pay and Google Pay.

## Apple Pay

Apple Pay enables secure, simple checkouts in your app or on your website.

Your customer will tap the Apple Pay button in the app or on the website, selects the payment card they want to use and uses the Touch-ID to complete the transaction.

1. Your App communicates with the your server and creates a transaction ID
2. Your App obtains the encrypted transaction payload (The tokenized card data "DPAN", Cryptogram, and transaction details) from Apple's Pass Kit Framework
3. Your App sends the encrypted transaction payload to our /payments API using the Apple Pay SDK
4. Our payments platform decrypts the encrypted transaction payload and processes the transaction
5. Our /paymennts API responds back to the Merchant App (through the SDK) with either an approval or decline

To accept Apple Pay, you'll need us to generate a Certificate Signing Request. Please reach out to your local Integration Support team for getting your account enabled. Our Integration team will generate the CSR and share it with you along with the Merchant Identifier.

To accept Apple Pay, you'll need to follow these steps:

### 1. Generate Payment Processing Certificate from Apple Portal

- Login to apple 'Account' in developer.apple.com
- In-order to generate certificate you must have a paid apple developer account or an organization account. New users must follow the prompts to set up a developer account
  - Select Certificates, Identifiers and Profile :
    - Certificates, Identifiers and Profile 
      - Select Identifier's and in the dropdown in the upper-right corner select merchant id
        - Enter a unique merchant id, and upload a valid CSR in .pem format
- Once successfully uploaded, apple provides the payment processing certificate and the merchant id will come up in the certificate section.
- Download payment processing certificate and install in your computer by double clicking it.
- The certificate should show up in the 'Key chain access' of your Macbook.

### 2. Upload and register the apple development certificate for your machine

- Request a new certificate from your keychain access
- Request certificate
- Follow the prompt and request the certificate to be saved on file
- In the 'Certificate' section in the apple portal, click on the '+' and follow the prompt to request apple developer certificate for 'IOS' development
- Upload the requested certificate
- Download and install the apple pay developer certificate
- Your machine is now setup for programming IOS app using Xcode

### 3. Set-up Provision Profile for the application

- In the 'Certificate, Identifier and Profile' section in the apple portal navigate to profile.
- Click the '+' sign to create a new profile.
- Choose, 'IOS' development and follow the prompts
- In the capabilities select 'Apple Pay' and 'In-App Purchases'
- You can also register the test devices here using, uuid. (Xcode also does this when the app is built on the phone)
- Download the provision profile on to your mac

### 4. Set-up Project in Xcode

- Select new xcode project with a single view or import an existing project.
- Register the app using the app-id in the apple portal under Identifier->appId
- Go to Xcode->Preferences,->accounts and install the downloaded provision profile, your profile is now linked to your Xcode.
- Click on your project, go to Signing & Capabilities select 'Automatically manage signing'.
- '+ capabilities' add apple pay.
- Under 'Apple Pay' click '+' and add the merchant id's registered in the portal, which in turn will be added to the entitlements file.
- Now the Xcode is set-up for coding.

### 5. In the SDK enter the URL, api key and api secret and build the app:

- Merchant id: Enter any valid merchant id registered in the apple portal. This gives the capability for a single user to use multiple merchant id's
- Amount: Enter the amount of the transaction
- Transaction type: Select PreAuth or Sale.
- Apple Pay Button: Click this to produce payment sheet and fingerprint authentication for the transaction.

### 6. Example Payload

Once the user authenticates the transaction the apple returns the payment token, then the SDK generates the following payload. The sample below is a `walletPaymentMethod` object within a `WalletSaleTransaction` requestType POST call to the /payments API.

```json yaml
{

   "walletPaymentMethod":{

      "walletType":"EncryptedApplePayWalletPaymentMethod",

      "encryptedApplePay":{

         "data":"hbreWcQg980mUoUCfuCoripnHO210lvtizOFLV6PTw1DjooSwik778bH/qgK2pKelDTiiC8eXeiSwSIfrTPp6tq9x8Xo2H0KYAHCjLaJtoDdnjXm8QtC3m8MlcKAyYKp4hOW6tcPmy5rKVCKr1RFCDwjWd9zfVmp/au8hzZQtTYvnlje9t36xNy057eKmA1Bl1r9MFPxicTudVesSYMoAPS4IS+IlYiZzCPHzSLYLvFNiLFzP77qq7B6HSZ3dAZm244v8ep9EQdZVb1xzYdr6U+F5n1W+prS/fnL4+PVdiJK1Gn2qhiveyQX1XopLEQSbMDaW0wYhfDP9XM/+EDMLaXIKRiCtFry9nkbQZDjr2ti91KOAvzQf7XFbV+O8i60BSlI4/QRmLdKHmk/m0rDgQAoYLgUZ5xjKzXpJR9iW6RWuNYyaf9XdD8s2eB9aBQ=",

         "header":{

            "applicationDataHash":"94ee059335e587e501cc4bf90613e0814f00a7b08bc7c648fd865a2af6a22cc2",

            "ephemeralPublicKey":"MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEvR+anQg6pElOsCnC3HIeNoEs2XMHQwxuy9plV1MfRRtIiHnQ6MyOS+1FQ7WZR2bVAnHFhPFaM9RYe7/bynvVvg==",

            "publicKeyHash":"KRsyW0NauLpN8OwKr+yeu4jl6APbgW05/TYo5eGW0bQ=",

            "transactionId":"31323334353637"

         },

         "signature":"MIAGCSqGSIb3DQEHAqCAMIACAQExDzANBglghkgBZQMEAgEFADCABgkqhkiG9w0BBwEAAKCAMIIB0zCCAXkCAQEwCQYHKoZIzj0EATB2MQswCQYDVQQGEwJVUzELMAkGA1UECAwCTkoxFDASBgNVBAcMC0plcnNleSBDaXR5MRMwEQYDVQQKDApGaXJzdCBEYXRhMRIwEAYDVQQLDAlGaXJzdCBBUEkxGzAZBgNVBAMMEmQxZHZ0bDEwMDAuMWRjLmNvbTAeFw0xNTA3MjMxNjQxMDNaFw0xOTA3MjIxNjQxMDNaMHYxCzAJBgNVBAYTAlVTMQswCQYDVQQIDAJOSjEUMBIGA1UEBwwLSmVyc2V5IENpdHkxEzARBgNVBAoMCkZpcnN0IERhdGExEjAQBgNVBAsMCUZpcnN0IEFQSTEbMBkGA1UEAwwSZDFkdnRsMTAwMC4xZGMuY29tMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAErnHhPM18HFbOomJMUiLiPL7nrJuWvfPy0Gg3xsX3m8q0oWhTs1QcQDTT+TR3yh4sDRPqXnsTUwcvbrCOzdUEeTAJBgcqhkjOPQQBA0kAMEYCIQDrC1z2JTx1jZPvllpnkxPEzBGk9BhTCkEB58j/Cv+sXQIhAKGongoz++3tJroo1GxnwvzK/Qmc4P1K2lHoh9biZeNhAAAxggFSMIIBTgIBATB7MHYxCzAJBgNVBAYTAlVTMQswCQYDVQQIDAJOSjEUMBIGA1UEBwwLSmVyc2V5IENpdHkxEzARBgNVBAoMCkZpcnN0IERhdGExEjAQBgNVBAsMCUZpcnN0IEFQSTEbMBkGA1UEAwwSZDFkdnRsMTAwMC4xZGMuY29tAgEBMA0GCWCGSAFlAwQCAQUAoGkwGAYJKoZIhvcNAQkDMQsGCSqGSIb3DQEHATAcBgkqhkiG9w0BCQUxDxcNMTkwNjA3MTg0MTIxWjAvBgkqhkiG9w0BCQQxIgQg0PLaZU4YWZqtP9t/ygv9XIS/5ngU6FlGjpvyK6VFXVMwCgYIKoZIzj0EAwIERjBEAiBTNmQEPyc3aMm4Mwa0riD3dNdSc9aAhslj65Us8b3aKwIgNSc/y+CWpsr8qDln0fZK6ZD/LWPMxofQedlPy7Q6gY8AAAAAAAA=",

         "version":"EC_v1",

         "applicationData":"VEVTVA==",

         "merchantId":"merchant.com.fapi.tcoe.applepay"

      }

   },

   "transactionAmount":{

      "total":"12.99",

      "currency":"USD"

   }

}
```

## Google Pay

Google Payment happens after your customer taps the "Google Pay" button, then selects a payment method and shipping address.

If the purchase originates from a third-party site

1. Your server issues a credential request with the Merchant ID and Processor Name as First Data to Google
2. Google returns response with encrypted payment credentials signed with the First Data key to the merchant server
3. You send the encrypted payload to our /payments API
4. We decrypt and validate the payload, then process the transaction and respond back to you with either an approval or decline response

If the purchase originates from a Google site:

1. Google initiates a purchase request to you after the consumer confirms the order
2. Your server issues a request with your Merchant ID and Processor Name as First Data to Google
3. Google returns response with encrypted payment credentials signed with the First Data key to your server
4. You send the encrypted payload to our /payments API
5. We decrypt and validate the payload and process the transaction and respond back to you with either an approval or decline response

Google Pay enables developers to add payment processing to Android-compatible apps and on Chrome on Android. These APIs allow consumers to pay with any credit card they have stored in their Google account, including any cards they may have previously set up on their Android Pay digital wallet. 

Implementing Google Pay

### 1. Merchant Boarding

Please reach out to your local Integration Support team for getting your account enabled. The Integration team will share a Sample Application to generate Google Encrypted payloads.

### 2. Prerequisites to build and run the sample Application

Developers wishing to use the Fiserv Google Pay sample application will need the following software and hardware:

- Google Play Services version 18.0.0 
- A physical device or an emulator to use for developing and testing. Google Play services can only be installed on an emulator with an AVD that runs Google APIs platform based on Android 4.4 or higher.
- The latest version of Android Studio. This includes:
  - The latest version of the AndroidSDK, including the SDK Tools component. The SDK is available from the
  - Android SDK Manager
  - JavaJRE(JDKfordevelopment) as per Android SDK requirements

Your project should be able to compile against Android 4.4 (KITKAT) or higher.

For more details, please refer https://developers.google.com/pay/api/android/guides/setup

### 3. Changes to be made in the Application 

The following parameters need to be defined 

- [ ] Merchant ID
- [ ] Merchant Token
- [ ] APIKey
- [ ] APISecret

Define the Fiserv Object Parameters: Parameters must be updated in the following files: 

- Constants.java
- EnvData.java

Update the Constants.java file with the Merchant ID and Gateway Tokenization parameters. Note that:

- The Merchant ID will be shared by the Integration team and
- The Gateway Tokenization parameter defaults to ‘firstdata’. 

In the EnvData.java file, set the following environment variables, which will be shared by the Integration Team:

- APIKey
- Token 
- APISecret 
 
envMap.put("CERT", new EnvPropertiesImpl( "CERT",

 "https://cert.api.firstdata.com/gateway/v1/payments",

 "---------", "----------", "-----------"));

gatewayMerchantId and the APIGEE credentials will be provided by the Integration Team

### 4. Credit Card for Testing

Note that for testing purposes; the credit card information used in the app must be attached to an active account. The standard test cards will not be validated by Google and will fail in processing, our integration team will provide google pay-specific test cards.

### 5. Execute Authorize and Purchase Request

The Authorization parameter, required as part of the Header for a Fiserv API transaction, needs to be created. Construct the data param by appending the following parameters in the order shown:

- apikey – the developer’s API key
- nonce – A secure random number
- timestamp – Epoch timestamp in milliseconds
- token – the Merchant Token 
- payload – The actual body content passed as the POST request 

### 6. Google Pay APK Installation to Device

- Once the downloaded code for the sample app is built successfully in Android Studio, build the APK and install it on your device. 
- Once the APK is installed, select the Open option to access the application. 
- The sample application should now be installed on the test device (nonPROD environment), and the response from a payment processed in the First Data nonPROD environment/Google test environment available

### 7. Example Payload

Sample Google Pay Request within the `walletPaymentMethod` object for a `WalletPreAuthTransaction`:

```json yaml
{

  "requestType": "WalletPreAuthTransaction",

  "walletPaymentMethod": {

    "walletType": "EncryptedGooglePayWalletPaymentMethod",

    "encryptedGooglePay": {

      "data": {

        "encryptedMessage": "8nxjB9mr2tWZeDRQRcGN91UUnb7AioGp3oRo8kmQ6lyvJZiqD7PJlbRCYElNqUmr6Z8zK7b2gO9MKOjpnTCqH0qAe2vuIlwNXB60M2Lh7Qfl3bVgWzwF/FfFcenVW381hoItYi8AjWnWoydz1XMTEv2qhqUG03mEnRXdMyDyk6KKZXoW8Qc0p1F1thbxxu8weU8CZbZsWGGTjB42cilIqLVbribcOAG8Oas1AcgefFsu2hwp4gdSuOg7wmeSV7XKsGQzzVy85qtjuqET2XYzJE3K/Wh9QKkhu5P9Ms5s1+Smr2IjRyidqQa88SxQplrVoo9+PvT0bxFcMspBmO3pLkuaZSUBy++dL2fefcxNJvGCFfFhdxW9DojuuQxgpeu7RAQUsGLyFmr/4ZfBxt882xTmpX9MRx5CAudl9qUgBfKdwWwMX35qSbDTm1ju5XXzNh94VebjD3bB9Zj8qgbmUOr/+6OQLhoFJyBCXgx3EEH8hBwNVFrss/SLwQvFhZh62eO6lOtnmbOtP1yTDDVqGDBfai5SwAmM+KTcc9SGv/xDC+cWe8ck+aCBkG4HoRPapUVMZ3JIgV7yzTsVLJE\\u003d",

        "ephemeralPublicKey": "BGH3fRFdoAobYrAlxnZOCYzkH84Cna92IZxtgsU36CMDaqSaDYb9/LsY8qw+vMtlBnwsUg/YVMOeeKp+qDkOWb4\\u003d",

        "tag": "nvmOUNpnOTZULLhMxT/hWCHzH/4f7gGpfvQgwl3p8ng\\u003d"

      },

      "signature": "MEUCIFWTRWUZAOM5nfJC79FtJm56olnbwG4H5uWWxAUWAquiAiEA24j/BcOroeISsdJzYsyoVi8wzu4tnmKw+jdsGfuvPko=",

      "version": "ECv1"

    }

  },

  "transactionAmount": {

    "total": "1200",

    "currency": "USD"

  }

}
```