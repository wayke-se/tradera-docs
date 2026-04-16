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

Emitted once when a branch activates the Tradera integration. Provides the branch identity so Tradera can bootstrap its dealer mapping.

```json
{
  "branch_id": "61fcbe56-2a07-4fd9-8850-02b070a1470c",
  "display_name": "Wayke Stockholm"
}
```

| Field          | Type   | Required | Description                      |
|----------------|--------|----------|----------------------------------|
| `branch_id`    | string | yes      | UUID of the Wayke branch         |
| `display_name` | string | no       | Human-readable name of the branch |

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
