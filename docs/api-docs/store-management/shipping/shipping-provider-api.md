# Become a Shipping Provider

<!-- this paragraph is general, mostly for a shipping provider -->
Shipping providers can offer shipping services and rates to BigCommerce merchants and shoppers. Each provider must build a BigCommerce app that quotes rates on demand using our API endpoints. BigCommerce retrieves and displays the provider's shipping options and rates. Merchants can see these rates in the control panel, and shoppers can see them at checkout. 

<!-- benefits to whom? this audience is unclear -->
The benefits of using a shipping provider include the following: 
- Drop-shippers can set their own rates
- Merchants can retrieve rates from custom shipping tables
- Merchants can retrieve rates from in-house calculation services
- Shoppers can create a combination of in-store pickup and shipping options
- 
<!-- the target audience for this article is a provider's developer -->
This article is a guide to creating and registering a shipping provider app for BigCommerce stores. For a demo app, see the [Shipping Provider example app](https://github.com/bigcommerce/sample-shipping-provider). 

## Prerequisites

* Get familiar with the [Introduction to Building Apps](/api-docs/apps/guide/intro) article for building [single-click apps](/api-docs/apps/guide/types#single-click).

## Shipping app overview

Once BigCommerce approves and registers your app, merchants can install the app on their stores and connect to your shipping service. They can enable your carrier in one or more shipping zones, and set different options for each zone your carrier services. BigCommerce queries your app to retrieve service options and rates on demand. The following figures illustrate the workflow:

![Shipping Provider Overview](https://storage.googleapis.com/bigcommerce-production-dev-center/images/shipping-provider-figure.png)

![Shipping App Overview](https://storage.googleapis.com/bigcommerce-production-dev-center/images/ship%20prov%20api.png 'Shipping Provider API')

### Single-carrier versus multi-carrier shipping providers

When you [sign up](#sign-up) your shipping provider, you can specify whether it's a single-carrier or multi-carrier provider. 

Single carriers offer services from one carrier; for example, USPS. Multi carriers offer service options from more than one carrier; for example, USPS, DHL, and Canada Post. In both cases, BigCommerce considers the provider itself to be the carrier. In multi-carrier apps, the registered provider/carrier supplies quotes from downstream carriers. 

Which carrier option you select affects how your quotes display to shoppers at checkout. The name of a single carrier provider appears next to the quote description in the shopper's list of shipping options, whereas the name of a multi carrier provider does not. The following images illustrate the difference:

![Single-carrier quote example](https://storage.googleapis.com/bigcommerce-production-dev-center/images/Single%20Carrier%20Example.png 'Single-carrier quote')

![Multi-carrier quote example](https://storage.googleapis.com/bigcommerce-production-dev-center/images/Multi%20Carrier%20Example.png 'Multi-carrier quote')

## Develop the app

To offer shipping services, you must build a BigCommerce [single-click app](/api-docs/apps/guide/types#single-click). This allows you to create [app API credentials](/api-docs/getting-started/authentication/authenticating-bigcommerce-apis#app-api-credentials) and promote your solution in the BigCommerce app marketplace. 

For more info, see the [Apps Quick Start](/api-docs/apps/quick-start) article and the [Introduction to Building Apps](/api-docs/apps/guide/intro) article.

### Your app ID

When you create an app, BigCommerce assigns it an ID. You need the app ID when you [sign up](#sign-up) as a shipping provider. To get your app ID, [create a draft app](/api-docs/apps/guide/development#registering-a-draft-app) in the [Dev Portal](https://devtools.bigcommerce.com/), then complete the information requested on the [Technical tab](/api-docs/apps/guide/publishing#add-technical-information). After you save the app, the Dev Portal control panel navigates to a URL that includes your app's unique ID. 

![App ID](https://s3.amazonaws.com/user-content.stoplight.io/6012/1552664114224 "App ID")

### Your service URLs

Your app must expose public URLs to receive the requests BigCommerce sends. These URLs can be any valid HTTPS URLs that use port `443`; for example, `https://your_app.example.com/rate`. Replace `your_app.example.com` and `rate` with your own host and path. Provide BigCommerce with the following URLs when you [sign up](#sign-up):

| URL | Required | API Reference | Description |
|:----|:---------|:--------------|:------------|
| Quote URL | yes | [Request shipping rates](/api-reference/providers/shipping-provider-api/shipping-provider/requestshippingrates) | A URL that accepts rate requests from BigCommerce and responds with shipping quotes. |
| Check Connection Options URL | optional | [Validate connection options](/api-reference/providers/shipping-provider-api/shipping-provider/validateconnectionoptions) | A URL to check connection options. BigCommerce will send requests with a merchant’s connection settings. You can look up a merchant's credentials in your database, call a downstream service, etc. |

 

### Request and response bodies

BigCommerce sends data to your service URLs in JSON request bodies, and expects JSON responses. To see how BigCommerce formats requests and how you must format responses, see the following Shipping Provider API endpoint references:

- [Request shipping rates](/api-reference/providers/shipping-provider-api/shipping-provider/requestshippingrates) reference 

- [Validate connection options](/api-reference/providers/shipping-provider-api/shipping-provider/validateconnectionoptions) reference

### Error handling

To handle errors, include human-readable error messages in your responses. Use the `messages` array to return one or more messages; flag error messages by specifying type `ERROR`. The following are example error responses for the [Request shipping rates](/api-reference/providers/shipping-provider-api/shipping-provider/requestshippingrates) and [Validate connection options](/api-reference/providers/shipping-provider-api/shipping-provider/validateconnectionoptions) endpoints: 

<!--
type: tab
title: Request Shipping Rates
-->

```json title="Example response with error message" lineNumbers
{
  "quote_id": "example id",
  "messages": [
    {
      "text": "Invalid connection options",
      "type": "ERROR"
    }
  ],
  "carrier_quotes": []
}
```

<!--
type: tab
title: Validate Connection Options
-->

```json title="Example response with error message" lineNumbers
{
  "valid": false,
  "messages": [
    {
      "text": "Your account ID is invalid",
      "type": "ERROR"
    }
  ]
}
```

<!-- type: tab-end  -->

### Configuration fields

Configuration fields are optional connection options or shipping settings options for your carrier. 

Connection options are set for the entire store and include sensitive authentication information.

Shipping settings options can be set for each shipping zone and include packaging type and more. The following figures show how options appear in the merchant control panel:

![FedEx Settings](https://storage.googleapis.com/bigcommerce-production-dev-center/images/FedEx%20Settings.png 'Setting options')

![FedEx Connection Settings](https://storage.googleapis.com/bigcommerce-production-dev-center/images/FedEx%20Connection%20Settings.png 'Connection options')


#### How configuration fields are used
 
- API users and merchants use connection options when they [connect your app to a store](#how-your-app-will-be-connected-to-a-store). They use settings options when defining shipping methods for your carrier. 
- BigCommerce will send connection options when we need you to [validate connection options](#validate-connection-options) and [provide shipping rates](#provide-shipping-rates-to-bigcommerce). BigCommerce also sends settings options when we need you to provide shipping rates. 

#### Types of configuration options
These are the types of configuration options that we allow:
- Text
- Checkbox 
- Select
- Multi-select
- Password

Specify the following for each configuration option when you [sign up](#sign-up): 
- **Label**: Text that merchants see when they connect in the control panel
- **Code**: Unique code for the option. Use snake case.
- **Required**: Whether the option is required 
- **Type**: Type of option

For select and multi-select types, specify values that you want available.

Below are examples of what you can specify for various configuration types:

<!-- 
type: tab
title: Text 
-->

```text title="Example shipping settings option" lineNumbers 
- Label: Display Name
- Required: true
- Type: text
```

<!-- 
type: tab
title: Checkbox 
-->

```text title="Example connection option" lineNumbers 
- Label: Shipping Rate
- Required: false
- Type: Checkbox
- Values: Enable Expedited Shipping
```

<!-- 
type: tab
title: Select 
-->

```text title="Example connection option" lineNumbers 
- Label: Packaging Type
- Required: false
- Type: select
- Values: Anti-corrosive, Plastics, Cardboard
```

<!-- 
type: tab
title: Multi-select
-->

```text title="Example connection option" lineNumbers 
- Label: Packaging Method
- Required: false
- Type: Multi-select
- Values: Extra primary packaging, Extra secondary packaging, Extra tertiary packaging
```

<!-- 
type: tab
title: Password 
--> 

```text title="Example connection option" lineNumbers 
- Label: Password
- Required: true
- Type: password
```

<!-- type: tab-end -->


## Sign up
After you develop the app, sign up to be a shipping provider. Submit your app to BigCommerce and BigCommerce will register your app as a shipping provider. 

### What you should provide BigCommerce
Send an email to [ShippingProviderAPI@bigcommerce.com](mailto:shippingproviderapi@bigcommerce.com) that includes the following information:

- Name of app
- [Your app ID](#your-app-id)
- Your email
- A description of the app. The description is displayed to merchants in the control panel.
- Logo: A 70x70 pixel logo that represents the shipping carrier app. Optional if you keep your app in a draft state.
- [Your service URLs](#your-service-urls) 
- [Your carrier status: single versus multi-carrier](#single-carrier-versus-multi-carrier-shipping-providers)
- [Configuration fields](#configuration-fields) (optional): For a list of items you need to provide, see [types of configuration options](#types-of-configuration-options).
- Countries Available (optional): A list of countries where merchants can use your carrier. Merchants who have a store origin address outside this list will not be able to use your carrier. 
  
  In most cases, this list should be as broad as possible. For example, if your carrier operates worldwide, make it available worldwide. You can limit the countries further than what your shipping carrier offers. If you don't provide the countries available, your carrier will be available worldwide (i.e. for every shipping origin). 

After submitting your app, you will receive a carrier ID. Both single-carrier and multi-carrier shipping providers receive one `carrier_id`.

### What you should document for API users

We recommend that you document your `carrier_id` and configuration option `code`s. API users will need your carrier ID and connection options to [connect your carrier to their store](#how-your-app-will-be-connected-to-a-store). API users will need your settings options to define a shipping method for your carrier. 

## How your app will be connected to a store

Once a store owner installs your app, merchants and API users can connect your carrier to a store. 

### How merchants will use your app
A merchant can select connection options for your carrier in the store control panel. The UI displays the `label` for your connection options, as shown in the following figure. 

![Connect Carrier via UI](https://storage.googleapis.com/bigcommerce-production-dev-center/images/connection%20options%20example.png 'Connection options in Shipping Manager') 

A merchant can then define and enable a shipping method for your carrier in one or more shipping zones. 

### How API users will use your app 

API users can connect your carrier by using the [Create a carrier connection](/api-reference/store-management/shipping-api/shipping-carrier/postshippingcarrierconnection) endpoint. They will send your `carrier_id` that you received after sign-up. As shown, they will specify the `code` for each connection option in the `connection` object:

```http title="Example POST request with X-Auth-Token header" lineNumbers
POST https://api.bigcommerce.com/stores/{{STORE_HASH}}/v2/shipping/carrier/connection
X-Auth-Token: {{ACCESS_TOKEN}}
Content-Type: application/json
Accept: application/json

{	
  "carrier_id" : "endicia",
  "connection": {
    "account_id" : "example_id",
    "pass_phrase" : "example_passphrase"
  }
}
```

API users can then define and enable a shipping method for your carrier in one or more shipping zones by using the [Create a shipping method](/api-reference/store-management/shipping-api/shipping-method/createashippingmethod) endpoint. In the request, API users will send values for your app's settings options which help determine the rates that your app sends to BigCommerce when BigCommerce requests a quote. For more info on how API users will use your carrier, see the [Use a Real-Time Carrier](/api-docs/store-management/shipping/use-real-time-carrier) article.

<!-- theme:info  -->
> #### Removing service  
> If a merchant uninstalls your app from the store, the merchant removes all shipping methods and connection info for your carrier(s) from the store. BigCommerce will no longer be able to make quote requests and receive shipping quotes from your carrier.

## API requests to your app

### Validate connection options  

When a merchant tries to [connect your carrier](#how-your-app-will-be-connected-to-a-store), BigCommerce sends you a request to check their connection options. Your response should state if the credentials are valid and explain what is wrong. For more info, see the [Validate connection options](/api-reference/providers/shipping-provider-api/shipping-provider/validateconnectionoptions) endpoint.

<!--
type: tab
title: Request
-->

```http title="Example POST request with with X-Auth-Token header" lineNumbers
POST https://your_app.example.com/check_connection_options
X-Auth-Token: {{ACCESS_TOKEN}}
Content-Type: application/json
Accept: application/json

{
  "connection_options": {
    "account_id": "example"
  }
}
```

<!--
type: tab
title: Response
--> 

```json title="Example POST response" lineNumbers
{
  "valid": false,
  "messages": [
    {
      "text": "Your account ID is invalid",
      "type": "ERROR"
    }
  ]
}
```
<!-- type: tab-end  -->

<!-- theme: info -->
> #### Credential validation
> It is best practice to authenticate merchants. However, if you did not provide a Check Connection Options URL, BigCommerce assumes a merchant's credentials are valid as long as they pass type checks. 

You can also authenticate merchants when BigCommerce requests rates.

### Provide shipping rates to BigCommerce

When BigCommerce needs shipping rates, BigCommerce checks its internal cache for valid entries. If a valid cache entry does not exist, BigCommerce makes a request to the [Quote URL](#your-service-urls) that you provided. The request includes the `code` for any [connection and zone settings options](#configuration-fields). Your carrier must then respond with shipping quote(s). For more info, see the [Request shipping rates](/api-reference/providers/shipping-provider-api/shipping-provider/requestshippingrates) endpoint.


<!--
type: tab
title: Request
--> 

```http title="Example POST request with X-Auth-Token header" lineNumbers
POST https://your_app.example.com/rate
X-Auth-Token: {{ACCESS_TOKEN}}
Content-Type: application/json
Accept: application/json

{
  "base_options": {
    "origin": {
      "street_1": "685 MARKET ST",
      "street_2": "",
      "zip": "94105",
      "city": "SAN FRANCISCO",
      "state_iso2": "CA",
      "country_iso2": "US",
      "address_type": "commercial"
    },
    "destination": {
      "street_1": "",
      "street_2": "",
      "zip": "94103",
      "city": "",
      "state_iso2": "CA",
      "country_iso2": "US",
      "address_type": "residential"
    },
    "items": [
      {
        "sku": "SKU-100",
        "variant_id": "1",
        "product_id": "1",
        "name": "Shirt",
        "length": {
          "units": "in",
          "value": 1
        },
        "width": {
          "units": "in",
          "value": 1
        },
        "height": {
          "units": "in",
          "value": 1
        },
        "weight": {
          "units": "oz",
          "value": 1
        },
        "discounted_price": {
          "currency": "USD",
          "amount": "10"
        },
        "declared_value": {
          "currency": "USD",
          "amount": "10"
        },
        "quantity": 1,
        "attributes": []
      }
    ],
    "customer": {
      "customer_groups": [
        {
          "customer_group_id": 5,
          "customer_group_name": "Retail"
        }
      ],
      "customer_id": 6
    },
    "store_id": "ru7t7fv9",
    "request_context": {
      "reference_values": [
        {
          "name": "cart_id",
          "value": "8"
        }
      ]
    }
  },
  "connection_options": {
    "key": "userKey",
    "account_number": "userAccountNumber"
  },
  "zone_options": {
    "show_transit_time": true
  },
  "rate_options": []
}
```
<!--
type: tab
title: Response
--> 

```json title="Example POST response" lineNumbers
{
  "quote_id": "example_quote",
  "messages": [],
  "carrier_quotes": [
    {
      "carrier_info": {
        "code": "usps_pitney_bowes",
        "display_name": "USPS"
      },
      "quotes": [
        {
          "code": "",
          "rate_id": "9vcV1JfckPJZW2pjeNXcKP5y",
          "display_name": "USPS Priority Mail",
          "cost": {
            "currency": "USD",
            "amount": 6.35
          },
          "transit_time": {
            "units": "BUSINESS_DAYS",
            "duration": 1
          },
          "dispatch_date": "2018-08-29T00:00:00-05:00"
        },
        {
          "code": "",
          "rate_id": "EakTRTvck2XYGVAQw9Mza8WW",
          "display_name": "USPS Priority Mail Express",
          "cost": {
            "currency": "USD",
            "amount": 22.98
          },
          "transit_time": {
            "units": "BUSINESS_DAYS",
            "duration": 1
          },
          "dispatch_date": "2018-08-29T00:00:00-05:00"
        }
      ]
    },
    {
      "carrier_info": {
        "code": "fedex",
        "display_name": "FedEx"
      },
      "quotes": [
        {
          "code": "GND",
          "rate_id": "JnQ2MPqkAMX9cBsw0jyt551R",
          "display_name": "FedEx Ground",
          "cost": {
            "currency": "USD",
            "amount": 8.53
          },
          "transit_time": {
            "units": "BUSINESS_DAYS",
            "duration": 1
          },
          "dispatch_date": "2018-09-05T11:00:00-05:00"
        },
        {
          "code": "2DA",
          "rate_id": "QwygEz9XjZx1bT9rfDZsVxSy",
          "display_name": "FedEx 2 Day",
          "cost": {
            "currency": "USD",
            "amount": 10.47
          },
          "transit_time": {
            "units": "BUSINESS_DAYS",
            "duration": 2
          },
          "dispatch_date": "2018-09-05T11:00:00-05:00"
        }
      ]
    }
  ]
}
```

<!-- type: tab-end  -->

If no shipping quotes are available, your carrier will send a response with the following format:

```json title="Example POST response" lineNumbers
{
  "quote_id": "example_quote",
  "messages": [],
  "carrier_quotes": []
}
```

<!-- theme: info -->
> #### Shipping quote sort order
> BigCommerce will display the shipping quotes that you return from lowest to highest price.

#### Product metadata in rate requests

When requesting rates, BigCommerce passes product metadata via product and variant metafields. This is useful if your service depends on fields that aren't included with products or variants by default. 

The metafields you receive from BigCommerce requests have the following characteristics:   

- Are product or variant metafields (category, brand, or other metafields cannot be passed in rate requests) 
- Have a metafield `permission_set` of `read`, `write`, `read_and_sf_access`, and `write_and_sf_access`.
- Have a metafield `namespace` that matches this format: `shipping_carrier_<carrier_id>` (for example, `shipping_carrier_72`)

  You receive the `carrier_id` when you [sign up](#sign-up) as a shipping provider.

For more information on product and variant metafields, see the following Catalog V3 API endpoints:

- Product Metafields, e.g. [Get all product metafields](/api-reference/store-management/catalog/product-metafields/getproductmetafieldsbyproductid)
- Product Variant Metafields, e.g. [Get all product variant metafields](/api-reference/store-management/catalog/product-variants-metafields/getvariantmetafieldsbyproductidandvariantid)


## Definitions

| Name | Description or Reference |
|:-----|:-------------------------|
| Check Connection Options URL | An optional URL for a shipping carrier resource that accepts check requests containing the connection options provided by a user when connecting the carrier and indicates whether or not those settings are valid. You provide this URL when you [sign up](#sign-up).|
| Configuration Fields | Connection and settings options. Merchants and API users use these fields to connect your carrier to their store and define shipping methods for your carrier in a zone. For more info, see [Configuration fields](#configuration-fields). |
| Connection Options | Optional fields that merchants and API users can use to connect your carrier to a store, including keys and passwords. |
| Settings Options | Optional fields that merchants and API users can use to specify your real-time shipping method, including available rates, packaging types, and packing methods. 
| Shipping Carrier | A service that facilitates delivery, such as UPS and FedEx. |
| Shipping Provider | A shipping solution that provides shipping rates to BigCommerce. A shipping provider can provide rates for one or more carriers. For more info, see [Single-carrier versus multi-carrier shipping providers](#single-carrier-versus-multi-carrier-shipping-providers). |
| Shipping Quote | An estimation of the cost to ship a set of items from an origin to a destination.|
| Shipping Zone | Describes a set of destination addresses and the applicable shipping settings, such as handling fees and available shipping methods.|
| Shipping Origin | The location from which goods are shipped. This origin determines which shipping carriers are available for the merchant to configure in the control panel.|
| Quote URL | A URL you provide when you [sign up](#sign-up) that accepts quote requests from BigCommerce and responds with shipping quotes. |

## FAQ

**Can I publish more than one app at a time?**

No, you can only publish one app at a time. The others can be for use as testing or as private apps.

## Resources

### Articles
- [Introduction to Building Apps](/api-docs/getting-started/building-apps-bigcommerce/building-apps)
- [Single-click apps](/api-docs/apps/guide/types#single-click)
- [App Store Approval Requirements](/api-docs/apps/guide/requirements)

### References

- [Shipping Providers](/api-reference/providers/shipping-provider-api)
- [Shipping Zones](/api-reference/store-management/shipping-api/shipping-zones)
- [Shipping Carriers](/api-reference/store-management/shipping-api/shipping-carrier)
- [Shipping Methods](/api-reference/store-management/shipping-api/shipping-method)

### Webhooks

- [Webhooks](/api-docs/store-management/webhooks/webhook-events#shipment)
