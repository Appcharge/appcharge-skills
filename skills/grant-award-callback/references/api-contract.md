# Grant Award Callback — API contract

Docs: https://docs.appcharge.com/api-reference/checkout/awards/grant-award-callback

Appcharge calls your server after payment finalization or free-offer collection. A valid `publisherPurchaseId` confirms fulfillment.

## Request

`POST /{your-grant-award-path}`

### Body (`PlayerOrderReportRequest`)

| Field | Type | Notes |
|-------|------|-------|
| `orderId` | string | Dashboard order ID |
| `purchaseId` | string | Purchase ID |
| `appChargePaymentId` | string | Payment ID |
| `purchaseDateAndTimeUtc` | string (date-time) | UTC; may be empty for some free orders |
| `playerId` | string | From player authentication |
| `bundleName` | string | Offer name |
| `bundleId` | string | Offer ID |
| `sku` | string | Publisher offer SKU |
| `products` | array | `{ amount, sku, name }` |
| `priceInDollar` | int32 | USD cents before tax |
| `priceInCents` | int32 | Local currency minor units |
| `subTotal` | int32 | Pre-tax in local minor units |
| `tax` | int32 | Tax in local minor units |
| `taxRate` | float | Percentage |
| `taxInDollar` | int32 | Tax in USD cents |
| `currency` | string | ISO 4217 |
| `currencyExchangeCost` | float | USD cents |
| `action` | enum | `purchase` \| `bonus` |
| `actionStatus` | enum | `completed` |
| `originalPriceInDollar` | int32 | USD cents |
| `paymentPriceInDollar` | int32 | USD cents |
| `paymentMethod` | string | e.g. `card`; empty for free |
| `countryCode2` | string | ISO 3166-1 alpha-2 |
| `priceTotalInDollar` | int32 | USD cents incl. tax |
| `playerEmail` | string | |
| `sessionMetadata` | string | Opaque string (often JSON) from auth |
| `receiptId` | string | |
| `estimatedPublisherNetAmount` | float | USD cents |
| `estimatedAppchargeFee` | float | USD cents |
| `pricePointMetadata` | int32 | Optional USD cents |
| `createdByIp` | string | |
| `zipCode` | string \| null | US/Canada |
| `awardFlow` | enum | `auto` \| `manual_retry` |
| `promoCodeName` | string | Optional |
| `discount` | float | Optional |
| `discountRatePoints` | number/string | Optional |
| `storeMetadata` | object | `productsSequenceIndex`, `offerType` (`PopUp`, `Bundle`, `SpecialOffer`, `RollingOffer`, `CheckoutLink`), optional `utms` |

## Responses

| Status | Body |
|--------|------|
| 200 | `{ "publisherPurchaseId": "<id>" }` **required** |
| 400 | `{ "publisherErrorMessage": "<reason>" }` |
| 500 | `{ "publisherErrorMessage": "<reason>" }` |

Missing or invalid `publisherPurchaseId` marks the award unsuccessful in Appcharge.
