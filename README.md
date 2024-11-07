# Gateway SDK Doc

## Overview

This documentation provides a detailed guide for integrating the Payment Gateway SDK into your application, offering both basic and advanced use cases. This guide covers configuration, API usage, error handling, and additional functionality to help you implement a seamless payment experience in your app.

## Prerequisites

Before starting the integration, ensure you meet the following requirements:

- **API Key**: You need a valid API key, which can be obtained through registration in the dev portal.
- **Secure Communication**: All communication with the payment gateway must occur over HTTPS to ensure data security.
- **Languages Supported**: The SDK is available for **JavaScript (Node.js)**, **PHP**, and **Python**.
- **Rate Limiting**: The API has rate limits in place. If these limits are exceeded, a `429 Too Many Requests` error will be returned.

## SDK Installation

To integrate the Payment Gateway SDK into your project, follow these steps:

### 1. Install SDK via Package Manager

#### JavaScript (Node.js)
```bash
npm install payment-gateway-sdk --save
```

#### PHP (Composer)
```bash
composer require payment-gateway/sdk:^2.0
```

#### Python (pip)
```bash
pip install payment-gateway-sdk==2.0.0
```

### 2. Manual Installation

If you prefer manual installation, you can download the SDK from our GitHub repository and include it in your project. Be sure to include all necessary dependencies and follow the setup instructions in the README file.

## Configuration

Once the SDK is installed, you need to configure it with your API Key and set the environment (either **sandbox** or **production**). Below are the steps for each language.

### Setting Up API Keys

Store your API key securely in environment variables to avoid exposing it in your codebase. Here is how to do it for each language:

#### JavaScript (Node.js)
```javascript
const PaymentGateway = require('payment-gateway-sdk');

const gateway = new PaymentGateway({
    apiKey: process.env.PAYMENT_GATEWAY_API_KEY,
    environment: 'production', // Options: 'production', 'sandbox'
    timeout: 5000, // Timeout in milliseconds for requests
});
```

#### PHP
```php
require 'vendor/autoload.php';

$gateway = new PaymentGateway([
    'apiKey' => getenv('PAYMENT_GATEWAY_API_KEY'),
    'environment' => 'production', // Options: 'production', 'sandbox'
    'timeout' => 5000, // Timeout in milliseconds
]);
```

#### Python
```python
import os
from payment_gateway_sdk import PaymentGateway

gateway = PaymentGateway(
    api_key=os.getenv("PAYMENT_GATEWAY_API_KEY"),
    environment="production",  # Options: 'production', 'sandbox'
    timeout=5000  # Timeout in milliseconds
)
```

## Payment Workflow

### 1. Creating a Payment

Use the `createPayment` method to initiate a payment. You need to provide parameters such as the amount, currency, description, and a redirect URL.

#### Parameters:
- **amount**: The amount to be charged in the smallest unit of the currency (e.g., cent or groš).
- **currency**: The ISO 4217 currency code (e.g., PLN for Polish Zloty).
- **description**: A description of the payment (e.g., "Payment for Order #12345").
- **redirectUrl**: The URL where the user will be redirected after the payment is processed.

#### Example - JavaScript
```javascript
gateway.createPayment({
    amount: 2000,  // Amount in the smallest unit (e.g., 2000 = 20.00 PLN)
    currency: 'PLN',  // Currency code for Polish Zloty
    description: 'Payment for Order #12345',
    redirectUrl: 'https://your-website.com/payment-success',
    metadata: {
        order_id: '12345',
        customer_email: 'customer@example.com'
    }
}).then(response => {
    console.log("Payment URL:", response.paymentUrl); // URL for redirect to the payment gateway
}).catch(error => {
    console.error("Error:", error.message);
});
```

#### Example - PHP
```php
$response = $gateway->createPayment([
    'amount' => 2000,  // Amount in smallest unit (e.g., 2000 = 20.00 PLN)
    'currency' => 'PLN',
    'description' => 'Payment for Order #12345',
    'redirectUrl' => 'https://your-website.com/payment-success',
    'metadata' => [
        'order_id' => '12345',
        'customer_email' => 'customer@example.com'
    ]
]);

echo "Payment URL: " . $response->paymentUrl;  // URL for redirecting to payment gateway
```

#### Example - Python
```python
response = gateway.create_payment(
    amount=2000,  # Amount in smallest unit (e.g., 2000 = 20.00 PLN)
    currency="PLN",
    description="Payment for Order #12345",
    redirect_url="https://your-website.com/payment-success"
)

print(f"Payment URL: {response.payment_url}")  # URL for redirect to the payment gateway
```

### 2. Checking Payment Status

After initiating a payment, you can check its status using the `checkPaymentStatus` method.

#### Parameters:
- **paymentId**: The unique identifier returned by the `createPayment` method.

#### Example - JavaScript
```javascript
gateway.checkPaymentStatus('PAYMENT_ID')
    .then(response => {
        console.log("Payment Status:", response.status);  // e.g., "paid", "pending", "failed"
    });
```

#### Example - PHP
```php
$response = $gateway->checkPaymentStatus('PAYMENT_ID');
echo "Payment Status: " . $response->status;  // e.g., "paid", "pending", "failed"
```

#### Example - Python
```python
status = gateway.check_payment_status(payment_id="PAYMENT_ID")
print(f"Payment Status: {status}")  # e.g., "paid", "pending", "failed"
```

### Handling Webhooks

You can receive notifications about payment status updates through webhooks. Use the `parseNotification` method to process incoming webhook data.

#### Example - PHP
```php
$payload = file_get_contents('php://input');
$notification = $gateway->parseNotification($payload);

if ($notification->status === 'paid') {
    // Process successful payment, such as updating the order status
}
```

#### Example - Python
```python
from flask import request

@app.route('/payment-notification', methods=['POST'])
def payment_notification():
    notification = gateway.parse_notification(request.data)
    
    if notification['status'] == 'paid':
        # Process successful payment
        pass
    
    return '', 200
```

### Refunds

Refunding a payment is simple with the `refundPayment` method. To initiate a refund, you need to provide the payment ID and the refund amount.

#### Example - JavaScript
```javascript
gateway.refundPayment('PAYMENT_ID', 2000)  // Refund amount in smallest unit (e.g., 2000 = 20.00 PLN)
    .then(response => {
        console.log("Refund Status:", response.status);  // "success" or "failed"
    });
```

#### Example - PHP
```php
$response = $gateway->refundPayment('PAYMENT_ID', 2000);  // Refund amount in smallest unit (e.g., 2000 = 20.00 PLN)
echo "Refund Status: " . $response->status;  // "success" or "failed"
```

#### Example - Python
```python
refund_response = gateway.refund_payment(payment_id="PAYMENT_ID", amount=2000)
if refund_response.success:
    print("Refund successful.")
else:
    print("Refund failed:", refund_response.error_message)
```

## Error Handling and Debugging

Handling errors gracefully is essential for maintaining a smooth user experience. The SDK provides detailed error messages for common failure scenarios.

### Retries and Timeout Management

In case of temporary network issues, the SDK automatically retries failed requests up to three times with an increasing delay. You can adjust the retry configuration if necessary.

#### Example - JavaScript
```javascript
gateway.createPayment({
    amount: 1000,
    currency: 'PLN',
    description: 'Order #987654',
    redirectUrl: 'https://your-website.com/payment-success'
}).then(response => {
    console.log("Payment URL:", response.paymentUrl);
}).catch(error => {
    if (error.retryable) {
        console.log("Retrying...");
        setTimeout(() => gateway.createPayment({/* retry parameters */}), 3000);
    } else {
        console.error("Non-retryable error:", error.message);
    }
});
```

## Conclusion

This guide provides a detailed explanation of how to integrate the Payment Gateway SDK into your application using PLN
