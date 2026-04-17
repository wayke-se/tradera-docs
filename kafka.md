# Kafka Event Contract

Wayke publishes vehicle ad events to a Kafka topic that Tradera consumes. All events follow the [CloudEvents v1.0](https://cloudevents.io/) specification.

## Topic

`ads.tradera.v0`

## Envelope

Every message is a CloudEvents envelope:

| Field         | Value                                              |
|---------------|----------------------------------------------------|
| `specversion` | `1.0.0`                                            |
| `source`      | `ads-service`                                      |
| `subject`     | `{branchId}` (UUID, used as partition key)         |
| `id`          | unique UUID per event                              |
| `time`        | RFC 3339 timestamp                                 |
| `type`        | one of the event types below                       |
| `data`        | JSON payload (see schemas below)                   |

## Event Types

### `wayke.ads.channels.tradera.branch.updated`

Emitted once when a branch activates the Tradera integration. Provides the branch identity and dealer profile so Tradera can bootstrap its dealer mapping.

```json
{
  "branch_id": "61fcbe56-2a07-4fd9-8850-02b070a1470c",
  "display_name": "Wayke Stockholm",
  "legal_name": "Wayke Sverige AB",
  "organization_number": "556123-4567",
  "market": "SE",
  "telephone": "+46 8 123 45 67",
  "email": "stockholm@wayke.se",
  "home_page": "https://wayke.se/stockholm",
  "logo": "https://img.wayke.se/branches/61fcbe56/logo.png",
  "description": "Auktoriserad Volvo- och Polestar-handlare i centrala Stockholm.",
  "address": {
    "street": "Sveavägen 100",
    "zip": "113 50",
    "city": "Stockholm",
    "county": "Stockholm",
    "latitude": 59.3426,
    "longitude": 18.0579
  },
  "opening_hours": [
    { "day_of_week": "MONDAY",   "open_from": "09:00", "open_to": "18:00" },
    { "day_of_week": "TUESDAY",  "open_from": "09:00", "open_to": "18:00" },
    { "day_of_week": "SATURDAY", "open_from": "10:00", "open_to": "15:00" }
  ],
  "is_mrf_dealer": true,
  "reseller_for": ["Volvo", "Polestar"]
}
```

| Field                 | Type     | Required | Description                                              |
|-----------------------|----------|----------|----------------------------------------------------------|
| `branch_id`           | string   | yes      | UUID of the Wayke branch                                 |
| `display_name`        | string   | no       | Human-readable name of the branch                        |
| `legal_name`          | string   | no       | Registered legal name of the dealership                  |
| `organization_number` | string   | no       | Swedish organisation number                              |
| `market`              | string   | no       | Market code: `SE`, `NO`, or `FI`                         |
| `telephone`           | string   | no       | Main telephone number                                    |
| `email`               | string   | no       | Main contact email address                               |
| `home_page`           | string   | no       | URL to the branch's website                              |
| `logo`                | string   | no       | URL to the branch logo image                             |
| `description`         | string   | no       | Free-text description of the branch                      |
| `address`             | object   | no       | Physical address of the branch (see below)               |
| `opening_hours`       | object[] | no       | Weekly opening hours entries (see below)                 |
| `is_mrf_dealer`       | bool     | no       | Whether the branch is an MRF-certified dealer            |
| `reseller_for`        | string[] | no       | Brands the branch is an authorised reseller for          |

#### `address`

| Field       | Type   | Required | Description                              |
|-------------|--------|----------|------------------------------------------|
| `street`    | string | no       | Street name and number                   |
| `zip`       | string | no       | Postal code                              |
| `city`      | string | no       | City name                                |
| `county`    | string | no       | County or region name                    |
| `latitude`  | float  | no       | Geographic latitude (WGS 84)             |
| `longitude` | float  | no       | Geographic longitude (WGS 84)            |

#### `opening_hours[]`

| Field        | Type   | Required | Description                                                              |
|--------------|--------|----------|--------------------------------------------------------------------------|
| `day_of_week`| string | yes      | Day of the week: `MONDAY`, `TUESDAY`, `WEDNESDAY`, `THURSDAY`, `FRIDAY`, `SATURDAY`, or `SUNDAY` |
| `open_from`  | string | no       | Opening time in `HH:MM` format (24-hour). Omitted when the branch is closed that day. |
| `open_to`    | string | no       | Closing time in `HH:MM` format (24-hour). Omitted when the branch is closed that day. |

**Conventions:**

- All field names use snake_case, consistent with the rest of this contract.
- `address`, `opening_hours`, and `reseller_for` are omitted entirely from the payload when empty — Tradera should treat a missing block as "no data" rather than an empty collection.
- `market` is one of `SE` | `NO` | `FI`. Today only SE branches flow through Tradera, but the enum is defined for future expansion.
- `day_of_week` is one of `MONDAY` | `TUESDAY` | `WEDNESDAY` | `THURSDAY` | `FRIDAY` | `SATURDAY` | `SUNDAY`.

### `wayke.ads.channels.tradera.ad.updated`

Emitted whenever a vehicle ad is published, updated, or unpublished. This is the primary event for keeping Tradera listings in sync.

```json
{
  "source_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "dealer_code": "ABC123",
  "image_urls": [
    "https://img.wayke.se/resize/1200x900/abc123.jpg",
    "https://img.wayke.se/resize/1200x900/def456.jpg"
  ],
  "price_sek": "249900.00",
  "status": "published",
  "description": "Well-maintained family car with full service history.",
  "url": "https://www.dealer-site.se/vehicles/volvo-xc60",
  "car_fields": {
    "registration_number": "ABC123",
    "vin": "YV1XZ91B6M1234567",
    "model_year": 2021,
    "mileage_km": 45000,
    "body_type": "SUV",
    "brand": "Volvo",
    "model": "XC60",
    "title": "Volvo XC60 T6 AWD Inscription",
    "fuels": ["Bensin"],
    "engineBaseType": "Hybrid",
    "transmission": "Automat",
    "color": "Svart",
    "equipments": ["Navigation", "Dragkrok", "Panoramatak"]
  }
}
```

| Field          | Type     | Required | Description                                                |
|----------------|----------|----------|------------------------------------------------------------|
| `source_id`    | string   | yes      | Unique identifier for the ad (Wayke ad ID)                 |
| `dealer_code`  | string   | yes      | Dealer code for the branch                                 |
| `image_urls`   | string[] | no       | Ordered list of image URLs                                 |
| `price_sek`    | string   | no       | Price in SEK as a decimal string (e.g. `"249900.00"`)      |
| `status`       | string   | yes      | `"published"` or `"unpublished"`                           |
| `description`  | string   | no       | Ad description text                                        |
| `url`          | string   | no       | Deep link to the vehicle on the dealer's website           |
| `car_fields`   | object   | yes      | Vehicle details (see below)                                |

#### `car_fields`

| Field                 | Type     | Required | Description                                      |
|-----------------------|----------|----------|--------------------------------------------------|
| `registration_number` | string   | no       | Swedish registration number                      |
| `vin`                 | string   | no       | Vehicle Identification Number                    |
| `model_year`          | int32    | no       | Model year                                       |
| `mileage_km`          | int64    | no       | Odometer reading in kilometers                   |
| `body_type`           | string   | no       | Body type (e.g. SUV, Sedan, Kombi)               |
| `brand`               | string   | no       | Manufacturer / brand                             |
| `model`               | string   | no       | Model name                                       |
| `title`               | string   | no       | Full ad title                                    |
| `fuels`               | string[] | no       | Fuel types (e.g. Bensin, Diesel, El)             |
| `engineBaseType`      | string   | no       | Engine base type (e.g. Hybrid, BEV, ICE)         |
| `transmission`        | string   | no       | Transmission type (e.g. Automat, Manuell)        |
| `color`               | string   | no       | Exterior color                                   |
| `equipments`          | string[] | no       | List of equipment / features                     |

## Lifecycle

1. **Branch activates Tradera** -> `branch.updated` event is published with branch identity.
2. **Ad is published or updated** -> `ad.updated` event with `status: "published"` and full vehicle data.
3. **Ad is unpublished** (sold, removed, or branch deactivates) -> `ad.updated` event with `status: "unpublished"`.

When a branch deactivates the Tradera integration, one `ad.updated` event with `status: "unpublished"` is emitted for each ad that was published at the time of deactivation.

## Notes

- All events for a given branch share the same partition key (`subject: branchId`), guaranteeing strict per-branch ordering.
- The `price_sek` field is a decimal string to avoid floating-point precision issues.
- The `engineBaseType` field uses camelCase per the agreed contract; all other fields use snake_case.
