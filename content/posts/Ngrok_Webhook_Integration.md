---
title: "Testing Webhooks Locally with Ngrok"
slug: "how-to-use-ngrok-webhook-integration"
date: 2022-03-12
updated: 2025-07-19
tags:
  - webhook
  - ngrok
---

When developing applications that consume webhooks, one common challenge is testing the integration on your local machine. Webhooks require a publicly accessible, SSL-certified URL to receive data from external services like Shopify, Sanity.io, or Mailchimp. This is where [Ngrok](https://ngrok.com) becomes an indispensable tool.

Ngrok creates a secure tunnel from the public internet directly to your `localhost`, allowing you to expose your local development server and test webhook payloads in real-time without deploying your application.

## Step-by-Step Guide to Using Ngrok for Webhooks

### 1. Install Ngrok

First, install Ngrok on your machine. If you're on a Mac, you can use Homebrew:

```shell
brew install ngrok/ngrok/ngrok
```
For other operating systems, please refer to the official [Ngrok installation guide](https://ngrok.com/download).

### 2. Authenticate Your Ngrok Account

While not strictly required for basic use, signing up for a free Ngrok account and adding your authtoken provides access to more features, such as longer tunnel durations and custom subdomains.

```shell
ngrok authtoken <YOUR_AUTH_TOKEN>
```

![Ngrok Dashboard](/images/ngrok-dashboard.png)

### 3. Start the Ngrok Tunnel

With your local server running, start an Ngrok tunnel and point it to the port your application is listening on. For example, if your server is on port 3000, run:

```shell
ngrok http 3000
```

Ngrok will generate a unique, public HTTPS URL that forwards all traffic to your local port 3000.

### 4. Configure Your Webhook Provider

Copy the secure forwarding URL from your terminal and paste it into the webhook configuration field of the service you're testing (e.g., Shopify, Stripe, GitHub).

![Shopify Webhook Configuration](/images/shopify-webhook.png)

### 5. Trigger and Receive the Webhook

Once configured, trigger the event in the external service (e.g., update a product in Shopify). The service will send a webhook payload to your Ngrok URL, which will be securely tunneled to your local application.

![SSH Tunnel Local](/images/ngrok-ssh-tunnel.png)

### 6. Inspect the Payload

Ngrok provides a powerful web interface for inspecting all traffic. To see the details of incoming requests, including headers and the JSON payload, navigate to `http://127.0.0.1:4040` in your browser.

![Ngrok Localhost](/images/ngrok-localhost.png)

## Conclusion

With the webhook payload successfully received and inspected, you can now confidently develop and debug your application's logic, mapping the incoming JSON data to your local database or other services. Ngrok simplifies the webhook development lifecycle, enabling rapid testing and iteration in your local environment.