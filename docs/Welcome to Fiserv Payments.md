# Welcome to Fiserv Payments!

In this section we'll tell you about our payments services, and guide you through integrating to and using the different capabilities within our payments products.

Our payments solutions enable you to integrate them into a seamless customer experience, and enable you to accept your customers' preferred payment methods. 

See below for the most popular options:

![Payment Method Icons](https://raw.githubusercontent.com/Fiserv-Developer/payments-api/2.0.0-docs/assets/images/icons.jpg)

Beyond accepting customer payments, our payments services include Tokenisation, 3DSecure 1 & 2, Pre-Authorisation functionality, creating and managing Recurring Payments and Payment URLs.

# Payment Solutions

## Hosted Payment Page

Our Hosted Payment Page called 'Connect' allows you to redirect your customer to our payment page when they are checking out. 

Our Hosted Page then manages the customer redirections that are required in the checkout process of many payment methods, or the complex authentication mechanisms (3DSecure) card payments now require. 

Using secure hosted pages can reduce the burden of compliance with the Data Security Standard of the Payment Card Industry (PCI DSS). If you want to lighten the PCI DSS load, but still make use of our extended capabilities, you can still use our RESTful APIs to access features where no direct consumer interaction is required and no sensitive cardholder data is getting processed.

Additionally, if you want to remove the complexity of enabling authenticated payments via the 3DSecure API, we suggest you use the Connect solution for all sale transactions, then use the REST APIs for all other use cases. 

## Payment APIs

If you want to build your own UI and manage payments within your checkout flow natively within your own site or application, use our RESTful APIs. For this, you'll need to have the relevant PCI Compliance capabilities to process raw card data, but you can own the end to end customer experience.

## Payment links you can send to customers

You can also request a Payment URL via our REST APIs, then send that to your customer. You customer then clicks the link you've sent them, which takes them to our Hosted Payment Page to complete the payment.

