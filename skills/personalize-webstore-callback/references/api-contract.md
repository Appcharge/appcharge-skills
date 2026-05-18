# Personalize Web Store Callback — API contract

Docs: https://docs.appcharge.com/api-reference/webstore/personalization/personalize-webstore-callback

Appcharge calls on login, after purchase, and every ~5 minutes after last sync. Response drives store UI, offers, balances, and segments.

## Request

`POST /{your-personalize-webstore-path}`

### Body

| Field | Type | Notes |
|-------|------|-------|
| `playerId` | string | Same as `publisherPlayerId` from authenticate callback |

## Response (200)

Required: `status`, `sessionMetadata`.

| Field | Type | Notes |
|-------|------|-------|
| `version` | integer | Use `2` |
| `status` | enum | `valid` \| `invalid` |
| `sessionMetadata` | object | Returned later in Grant Award when enabled |
| `logo` | string | Asset Library ID |
| `profileFrameId` | string | |
| `playerLevelName` | string | |
| `bannerExternalId` | string | |
| `playerLevel` | object | `{ assetId, text?, endsIn? }` |
| `playerLevelBanners` | array | `{ assetId, designId, text?, endsIn? }` |
| `offersOrder` | enum | `publisherOrder` \| `priceHighToLow` \| `priceLowToHigh` (default `priceLowToHigh`) |
| `sectionsOrder` | string[] | Section IDs order |
| `segments` | string[] | Segmentation tags |
| `focus` | object | `{ publisherBundleId }` |
| `attributes` | object | Custom key-value for Dashboard filters |
| `storeTheme` | object | `bgImageMobile`, `bgImageDesktop`, `logo`, `profileFrameImage`, `bannerImage`, `playerLevelImage` |
| `balances` | array | `{ publisherProductId, quantity }` |
| `offers` | array | See below |

### `offers[]` (required per offer: `publisherOfferId`, `productsSequence`)

| Field | Notes |
|-------|-------|
| `publisherOfferId` | Publisher offer id |
| `endsIn` | Epoch ms countdown |
| `offerDescriptionOverride` | |
| `offerDesignOverride` | `offerDesignSubtitleTextOverride`, `offerDesignId`, `offerBackgroundImageOverride` |
| `priceDiscount` | `{ priceBeforeDiscount?, discount, type: "percentage" }` — not for Rolling Offers at root |
| `productSale` | Rolling-offer root rules differ; see docs |
| `badges` | `{ publisherBadgeId, position, ribbonTextOverride? }[]` |
| `productsSequence` | Rolling/progress-bar sequences |

### `productsSequence[]`

| Field | Notes |
|-------|-------|
| `index` | int ≥ 1; mission order for Progress Bar |
| `products` | `{ publisherProductId, quantity, priority?: Main\|Sub, traits?, rarityProductInfo? }[]` |
| `priceDiscount` | For Rolling Offers at sequence level |
| `productSale` | For Rolling Offers at sequence level |
| `badges` | For Rolling Offers at sequence level |
| `progressBarPoints` | `{ publisherBarId, points }[]` |

Start with a minimal valid payload (`status: valid`, `version: 2`, `sessionMetadata`, empty or sample `offers`/`balances`) and expand from live catalog/segmentation services.
