# Analytics Events

Wayke exposes an HTTP ingress for browser-side analytics events. Tradera's consumer-facing pages (search, vehicle detail, etc.) should `POST` JSON events to this ingress whenever a user interacts with Wayke-sourced inventory. Events are forwarded to Wayke's analytics pipeline (Google Pub/Sub + Elasticsearch) and power dashboards, conversion reporting, and exposure billing.

## Endpoints

| Environment | Market | URL                                      |
|-------------|--------|------------------------------------------|
| Production  | SE     | `https://analytics.wayke.se/`            |
| Test        | SE     | `https://test-analytics.wayketech.se/`   |

Use the SE endpoint for Swedish inventory and the NO endpoint for Norwegian inventory.

## Request

| Property        | Value                                              |
|-----------------|----------------------------------------------------|
| Method          | `POST`                                             |
| Path            | `/`                                                |
| Content-Type    | `application/json`                                 |
| Body            | JSON **array** of one or more event objects        |
| Max body size   | 10 MiB                                             |
| CORS            | Enabled — the endpoint can be called directly from the browser |
| Success response| `204 No Content` (empty body)                      |

Events should be batched rather than sent one-per-request. Wayke's own web client debounces events for ~1.5 seconds and flushes the accumulated batch as a single array. A similar strategy is recommended.

## Event envelope

Every element in the array is a flat JSON object. Common top-level fields:

| Field              | Type              | Required     | Description                                                                 |
|--------------------|-------------------|--------------|-----------------------------------------------------------------------------|
| `eventType`        | string            | yes          | One of the values listed under [Event types](#event-types)                  |
| `source`           | string            | yes          | Origin of the event, e.g. `"tradera"`                                       |
| `eventTimestamp`   | string (ISO 8601) | yes          | Client-side timestamp when the event occurred                               |
| `url`              | string            | no           | Full page URL at the time of the event                                      |
| `pageType`         | string            | no           | Page context, e.g. `"search"`, `"carDetail"`, `"startPage"`                 |
| `item`             | object            | conditional  | Required for item-scoped events (`view`, `exposure`, `productClick`, `microConversion`, `shownInterest`) |
| `searchData`       | object            | conditional  | Required for `search` events; contains `hitCount`, `sortOption`, response stats |
| `information`      | object            | no           | Free-form per-event metadata. `information.email` is stripped server-side   |
| `conversionType`   | string            | conditional  | Required for `microConversion`, e.g. `"imagesClick"`, `"insuranceInterest"`, `"share"` |
| `list`             | string            | no           | List context the event occurred in, e.g. `"search-results"`                 |                                             |
| `userData`         | object            | no           | Session / user context — see [Privacy](#privacy)                            |
| `urlParameters`    | string            | no           | Raw query string; server parses it into individual fields                   |

The `item` object typically carries: `id`, `name`, `slug`, `branches` (array of branch ids), `position.location.{lat,lon}`, `modelYear`, `properties.{seats,co2}`, and media/price info as available.

The `userData` object typically carries session metadata: `sessionId`, `unAuthId`, `userId`, `login` (boolean), `userType` (e.g. `"private"`, `"dealer"`, `"N/A"`), `cid`.

## Event types

Events are routed into three delivery buckets based on `eventType`. Bucket affects latency expectations (realtime fans out immediately; deferred buckets are batched downstream).

**Realtime** (default — any `eventType` not listed below is treated as realtime):

- `conversion`
- `hasRegistered`
- `subscribed`
- Any custom `eventType` not recognised by the pipeline

**Deferred — interaction**:

- `pageview`
- `navigation`
- `search`
- `view`
- `microConversion`
- `shownInterest`
- `productClick`
- `carDetailNavigation`

**Deferred — exposure**:

- `exposure`
- `newsItemExposure`
- `accessoryExposure`

## Privacy

The following fields are **stripped server-side** before any event is persisted or forwarded:

- `userData.email`
- `userData.givenName`, `userData.familyName`, `userData.name`
- `userData.phone`
- `userData.address`, `userData.zipCode`, `userData.city`, `userData.country`
- `userData.latitude`, `userData.longitude`
- `userData.birthDate`, `userData.gender`, `userData.title`
- `userData.imageUrl`
- `userData.wantsEmailNotificationOnSubscriptionChange`
- `userData.wantsEmailNotificationOnConversationChange`
- `userData.wantsSmsNotificationOnAssignmentChange`
- `information.email`

This is a safety net. Callers should still avoid sending personal data in the first place — in particular, do not include names, email addresses, or phone numbers in the `information` object.

## Examples

### `pageview`

```json
[
  {
    "eventType": "pageview",
    "source": "tradera",
    "url": "https://www.tradera.com/wayke/search?brand=Volvo",
    "eventTimestamp": "2026-04-20T09:12:34.000Z",
    "pageType": "search",
    "userData": {
      "sessionId": "9d3e4b30-5ad2-4d4e-8e0a-7d7f7a1a9e1e",
      "unAuthId": "unauth-8a1c3d2b-...",
      "login": false,
      "userType": "N/A"
    }
  }
]
```

### `search`

```json
[
  {
    "eventType": "search",
    "source": "tradera",
    "url": "https://www.tradera.com/wayke/search?brand=Volvo&sort=priceAsc",
    "eventTimestamp": "2026-04-20T09:12:36.120Z",
    "pageType": "search",
    "urlParameters": "brand=Volvo&sort=priceAsc",
    "searchData": {
      "hitCount": 342,
      "sortOption": "priceAsc",
      "responseTime": 87
    },
    "userData": {
      "sessionId": "9d3e4b30-5ad2-4d4e-8e0a-7d7f7a1a9e1e",
      "unAuthId": "unauth-8a1c3d2b-...",
      "login": false,
      "userType": "N/A"
    }
  }
]
```

### `microConversion`

```json
[
  {
    "eventType": "microConversion",
    "source": "tradera",
    "url": "https://www.tradera.com/wayke/vehicle/61fcbe56-2a07-4fd9-8850-02b070a1470c",
    "eventTimestamp": "2026-04-20T09:14:02.480Z",
    "pageType": "carDetail",
    "conversionType": "imagesClick",
    "list": "carDetail",
    "item": {
      "id": "61fcbe56-2a07-4fd9-8850-02b070a1470c",
      "name": "Volvo XC60 T6 Recharge",
      "slug": "volvo-xc60-t6-recharge",
      "branches": ["b1fcbe56-2a07-4fd9-8850-02b070a14700"],
      "modelYear": 2024,
      "properties": { "seats": 5 }
    },
    "userData": {
      "sessionId": "9d3e4b30-5ad2-4d4e-8e0a-7d7f7a1a9e1e",
      "unAuthId": "unauth-8a1c3d2b-...",
      "login": false,
      "userType": "N/A"
    }
  }
]
```

### Batched POST with curl

```bash
curl -i -X POST https://test-analytics.wayketech.se/ \
  -H 'Content-Type: application/json' \
  -d '[
    {
      "eventType": "pageview",
      "source": "tradera",
      "url": "https://www.tradera.com/wayke/search",
      "eventTimestamp": "2026-04-20T09:12:34.000Z"
    },
    {
      "eventType": "productClick",
      "source": "tradera",
      "url": "https://www.tradera.com/wayke/search",
      "eventTimestamp": "2026-04-20T09:12:40.000Z",
      "list": "search-results",
      "item": { "id": "61fcbe56-2a07-4fd9-8850-02b070a1470c" }
    }
  ]'
```

Expected response:

```
HTTP/1.1 204 No Content
```
