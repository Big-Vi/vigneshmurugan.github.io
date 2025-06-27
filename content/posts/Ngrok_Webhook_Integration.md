---
title: "How to use ngrok - webhook integration"
slug: "how-to-use-ngrok-webhook-integration"
date: 2022-03-12
tags:
  - webhook
  - ngrok
---

Ngrok allows you to expose your localhost via a secure tunnel to the internet. One of the features of Ngrok is testing the webhook payload in your local environment.

## What is Ngrok?

[Ngrok](https://ngrok.com) allows you to expose your localhost via a secure tunnel to the internet. One of the features of Ngrok is testing the webhook payload in your local environment.

## What is a webhook?

Webhook is sending real-time information(as JSON or XML) to a specified URL(External system) whenever some event happens. It's more prevalent these days with the latest platforms such as Shopify, Sanity.io & Mailchimp.

For security reasons, Webhook payloads are only exposed to SSL-certified URLs. This is where Ngrok comes into help.

## How does it work?

1 - Install Ngrok on your local machine. For Mac run the below command in your terminal:

```shell
brew install ngrok/ngrok/ngrok
```

2 - I suggest signing up for Ngrok so that account-only features can be used. 

```shell
ngrok authtoken <token>
```

![image](https://cdn.sanity.io/images/bz8z0oa1/production/1f0dac2b38f6982dded78adab7140f9f4f5cc749-2890x1638.png?w=650)

3 - Start your Ngrok tunnel. 

```shell
ngrok http 3000
```

4 - Copy the secure URL from the terminal and paste it into the webhook URL field from which the payload comes. In my case, I'm testing it with Shopify.

![image](https://cdn.sanity.io/images/bz8z0oa1/production/94edc111b69736cb2dfb2abfa4c841866bfd93c4-2918x1334.png?w=650)

5 - Once the product from Shopify gets updated, it sends the payload to your specified Ngrok URL which tunnels the payload into your local environment.

![image](https://cdn.sanity.io/images/bz8z0oa1/production/7a2b0c7743022a3334a5ec0d478abe63bacf9b9f-1666x583.png?w=650)

6 - To inspect your payload, go to "http://127.0.0.1:4040/inspect/http" from your browser.

![image](https://cdn.sanity.io/images/bz8z0oa1/production/4789a78cec296e5c6db83744246d1b40221aa434-2916x1636.png?w=650)

🎉 JSON payload can be mapped to your local database.