---
title: "Integrating E-Commerce into Your Site with Shopify's Storefront API"
slug: "shopify-storefront-api-integration-with-existing-site"
date: 2022-02-22
updated: 2025-07-19
tags:
  - shopify
  - headless-commerce
  - api
---

If you're looking to add e-commerce functionality to an existing website or application, Shopify offers a powerful and developer-friendly solution. With tools like the Storefront and Admin APIs (both with GraphQL endpoints), webhooks, and Shopify Payments, you can build a custom shopping experience without starting from scratch.

This approach, often called "headless commerce," allows you to use your existing frontend while Shopify handles the complex backend logic.

### Why Go Headless with Shopify?

-   **Decoupled Architecture**: You maintain full control over your website's look and feel while Shopify manages payments, shipping, discounts, and inventory.
-   **Seamless Integration**: Shopify is designed to connect with other business systems like ERPs, CRMs, or a CMS, making it a flexible hub for your commerce operations.
-   **Developer Resources**: To accelerate development, Shopify provides official SDKs and starter kits for various languages and frameworks.

## Understanding the Storefront API

The **Storefront API** is the core of Shopify's headless offering. It provides a public-facing GraphQL endpoint that lets you fetch product data, create carts, and manage checkouts directly from your client-side application. This is what enables you to build custom e-commerce sites, mobile apps, or even integrate purchasing into video games using the [Shopify Unity Buy SDK](https://www.shopify.com/partners/blog/using-shopify-unity-buy-sdk).

Because the Storefront API is public, requests can be safely made from a user's browser. This contrasts with the **Admin API**, which is designed for backend use only and provides read/write access to your store's data, including products, orders, and shipping details.

## Using the Shopify Buy SDK

To simplify the integration process, Shopify provides the **Buy SDK**. This SDK acts as a wrapper around the Storefront API, providing convenient methods for common tasks like fetching products and collections, managing line items, and applying discounts. It essentially initializes a GraphQL client with the necessary credentials and queries, so you can focus on building your user interface.

## The Checkout Process

The headless checkout process is straightforward:

1.  Your application uses the Storefront API (or Buy SDK) to create a cart session and add products to it.
2.  Shopify generates a unique checkout URL for that session.
3.  When the customer is ready to pay, you redirect them to this URL.

This takes the customer to Shopify's secure, PCI-compliant checkout page, where they can enter their payment and shipping information. Shopify handles the entire transaction, so you don't have to worry about processing sensitive financial data.

## Example: NetSuite Integration

The ability to integrate with other business systems is a major advantage. In my organization, we use NetSuite as our ERP. When an order is placed through our Shopify-powered site, a webhook triggers a process that pushes the order details into NetSuite for our accounting team.

Conversely, when the finance team marks an order as paid in NetSuite (for manual payments), an automated process uses a GraphQL mutation via the Admin API to sync the updated order status back to Shopify. This creates a seamless flow of information between our commerce platform and our core business system.
