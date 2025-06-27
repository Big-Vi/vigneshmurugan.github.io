---
title: "Shopify storefront API integration with existing site"
slug: "shopify-storefront-api-integration-with-existing-site"
date: 2022-02-22
tags:
  - shopify
---

If you're thinking of adding e-commerce to your existing application, turn to Shopify as it's offering great developer-friendly features such as Storefront API & Admin API with GraphQL endpoint, Webhooks, and Shopify payment(available for limited countries).

If you're thinking of adding e-commerce to your existing application, turn to Shopify as it's offering great developer-friendly features such as Storefront API & Admin API with GraphQL endpoint, Webhooks, and Shopify payment(available for limited countries). 

- When working with Shopify Headless, you're merely displaying the products on the existing website, and Shopify does all the heavy lifting(Payment, Shipping, Discount, and so on).
- Integrating with other external systems such as ERP, CRM or CMS is easy with Shopify.
- Shopify created a GIT repo for various languages and frameworks to kick-start with.

## Storefront API

Storefront API offers a GraphQL endpoint that enables developers to build headless e-commerce sites, and mobile apps & even sell products via video games([Shopify unity buy SDK](https://www.shopify.com/partners/blog/using-shopify-unity-buy-sdk)).

Storefront API is a public-facing API, as opposed to Admin API, which means requests to the Storefront API can be made from the client side.

Request to the Admin API can be sent only via the back end of the app. Using Admin API, developers can read and write to products, inventory, orders, shipping & more.

## Shopify buy SDK

"Buy SDK" is a wrapper around functionalities like fetching products, collections, line items, discounts, and so on. It basically initializes the GraphQL client with the query to make a request to Shopify with the credentials supplied.

## Checkout process

The checkout process starts with creating a Shopify cart session from your website which then creates Shopify checkout URL. Products can be added to the cart session.

The checkout URL takes the customers to the Shopify checkout page. 

## Netsuite integration

As I mentioned above that integrating with the business system is a breeze with Shopify. In my organization, we use NetSuite ERP. So all the orders created in Shopify need to be pushed into NetSuite for the accounts team to handle. NetSuite sends an automatic invoice to the customer If they chose to pay their order manually and then sync the order status back to Shopify via GraphQL mutation.
