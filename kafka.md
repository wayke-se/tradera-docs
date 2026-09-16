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
| `display_name`        | string   | yes¹     | Human-readable name of the branch                        |
| `legal_name`          | string   | no       | Registered legal name of the dealership                  |
| `organization_number` | string   | yes¹     | Swedish organisation number                              |
| `market`              | string   | yes      | Market code: `SE`, `NO`, or `FI`                         |
| `telephone`           | string   | yes¹     | Main telephone number                                    |
| `email`               | string   | yes¹     | Main contact email address                               |
| `home_page`           | string   | no       | URL to the branch's website                              |
| `logo`                | string   | no       | URL to the branch logo image                             |
| `description`         | string   | no       | Free-text description of the branch                      |
| `address`             | object   | yes¹     | Physical address of the branch (see below)               |
| `opening_hours`       | object[] | no       | Weekly opening hours entries (see below)                 |
| `is_mrf_dealer`       | bool     | no       | Whether the branch is an MRF-certified dealer            |
| `reseller_for`        | string[] | no       | Brands the branch is an authorised reseller for          |

¹ Enforced by Wayke at activation time — see [Branch activation prerequisites](#branch-activation-prerequisites) below.

#### `address`

| Field       | Type   | Required | Description                              |
|-------------|--------|----------|------------------------------------------|
| `street`    | string | yes¹     | Street name and number                   |
| `zip`       | string | yes¹     | Postal code                              |
| `city`      | string | yes¹     | City name                                |
| `county`    | string | no       | County or region name                    |
| `latitude`  | float  | no       | Geographic latitude (WGS 84)             |
| `longitude` | float  | no       | Geographic longitude (WGS 84)            |

¹ Enforced by Wayke at activation time — see [Branch activation prerequisites](#branch-activation-prerequisites) below.

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

#### Branch activation prerequisites

Wayke refuses to activate the Tradera integration for a branch until the following fields are populated in the branch's organization profile. Every `branch.updated` event Tradera receives is therefore guaranteed to carry non-empty values for these fields:

| Field                 | Notes                                                                |
|-----------------------|----------------------------------------------------------------------|
| `display_name`        | dealer name shown to buyers                                          |
| `organization_number` | Swedish organisation number                                          |
| `telephone`           | main telephone number                                                |
| `email`               | main contact email                                                   |
| `address.street`      | the `address` block is always present, with these three sub-fields populated |
| `address.zip`         |                                                                      |
| `address.city`        |                                                                      |

All other fields listed in the schema (`legal_name`, `home_page`, `logo`, `description`, `address.county`, `address.latitude`, `address.longitude`, `opening_hours`, `is_mrf_dealer`, `reseller_for`) remain optional and may be absent or empty.

If any of the prerequisite fields are missing when a dealer attempts to enable Tradera in Wayke, the operator gets an error listing exactly which fields need to be populated; no events flow to Tradera until the data is corrected.

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
    "equipments": ["Navigation", "Dragkrok", "Panoramatak"],
    "brandSpecific": {
      "model": "XC60",
      "variant": "T6",
      "badge": "T6 AWD Inscription",
      "terms": ["AWD", "Inscription"]
    },
    "registration_date": "2021-03-12",
    "drivetrain": "Fyrhjulsdrift",
    "engine_power_hp": 340,
    "engine_torque_nm": 400,
    "engine_displacement_cc": 1969,
    "engine_displacement_litres": 2.0,
    "number_of_gears": 8,
    "acceleration_0_100_s": 5.9,
    "fuel_consumption_combined_l_100km": 2.1,
    "fuel_consumption_standard": "WLTP",
    "co2_combined_g_km": 48,
    "co2_standard": "WLTP",
    "euro_class": "Euro 6",
    "energy_class": "A",
    "electric_vehicle_type": "Laddhybrid",
    "battery_capacity_gross_kwh": 11.6,
    "battery_capacity_net_kwh": 9.1,
    "electricity_consumption_combined_kwh_100km": 18.2,
    "electric_range_km": 53,
    "length_mm": 4708,
    "width_mm": 1902,
    "height_mm": 1658,
    "seats": 5,
    "doors": 5,
    "segment": "Mellanstor SUV",
    "trunk_volume_rear_l": 468,
    "curb_weight_kg": 2131,
    "max_load_kg": 529,
    "max_trailer_weight_braked_kg": 2250,
    "euro_ncap_rating": 5,
    "euro_ncap_test_year": 2017,
    "number_of_owners": 2,
    "imported": false,
    "annual_tax_sek": 360,
    "manufacturer_warranty_years": 2
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
| `brandSpecific`       | object   | no       | OEM-curated taxonomy (see below)                 |

##### `brandSpecific`

Brand-curated values used to group listings under the right OEM model/variant page on Tradera. The block is omitted entirely when the federated vehicle has no brand-specific data; individual fields are omitted when empty.

```json
"brandSpecific": {
  "model": "A4",
  "variant": "40",
  "badge": "40 TDI quattro 2.0",
  "terms": ["quattro", "Avant"]
}
```

| Field     | Type     | Required | Description                                                                |
|-----------|----------|----------|----------------------------------------------------------------------------|
| `model`   | string   | no       | Brand-curated model name (e.g. `"A4"`, `"5-Serie"`, `"XC60"`)              |
| `variant` | string   | no       | Brand-curated variant grouping under the model (e.g. `"40"`, `"540"`)      |
| `badge`   | string   | no       | Specific sales badge / configuration (e.g. `"40 TDI quattro 2.0"`)         |
| `terms`   | string[] | no       | Free-form taxonomy terms (drivetrain, body, trim, etc., e.g. `["quattro", "Avant"]`) |

> Field names use camelCase (`brandSpecific`) to match the agreed contract — same exception `engineBaseType` already makes; the rest of `car_fields` is snake_case.

##### Extended vehicle specification

All fields below live directly on `car_fields`, next to the existing ones. Every field is optional and is omitted when Wayke has no value for the vehicle; a missing field means "unknown", never zero or false.

**Motor och prestanda / Engine and performance**

| Field                        | Type    | Required | Description                                                                 |
|------------------------------|---------|----------|-----------------------------------------------------------------------------|
| `registration_date`          | string  | no       | Date of first registration, `yyyy-MM-dd`                                    |
| `drivetrain`                 | string  | no       | `Framhjulsdrift`, `Bakhjulsdrift` or `Fyrhjulsdrift`                        |
| `engine_power_hp`            | int32   | no       | Engine power in hk (metric horsepower). Total system output for hybrids and EVs |
| `engine_torque_nm`           | int32   | no       | Torque in Nm. Total system torque for hybrids and EVs                       |
| `engine_displacement_cc`     | int32   | no       | Exact displacement in cm³, e.g. `1969`. Omitted for pure EVs               |
| `engine_displacement_litres` | float   | no       | Nominal marketing displacement in litres, e.g. `2.0`. Omitted for pure EVs |
| `number_of_gears`            | int32   | no       | Number of forward gears. Usually absent for pure EVs                        |
| `acceleration_0_100_s`       | float   | no       | 0–100 km/h in seconds, e.g. `5.9`                                           |

**Förbrukning och miljö / Consumption and environment**

| Field                               | Type   | Required | Description                                                                                  |
|-------------------------------------|--------|----------|----------------------------------------------------------------------------------------------|
| `fuel_consumption_combined_l_100km` | float  | no       | Combined fuel consumption in l/100 km. Omitted for pure EVs                                  |
| `fuel_consumption_standard`         | string | no       | `WLTP` or `NEDC`: the test cycle `fuel_consumption_combined_l_100km` was measured under. Present exactly when that field is |
| `co2_combined_g_km`                 | int32  | no       | Combined CO₂ emissions in g/km                                                               |
| `co2_standard`                      | string | no       | `WLTP` or `NEDC`: the test cycle `co2_combined_g_km` was measured under. Present exactly when that field is |
| `euro_class`                        | string | no       | Euro emission standard as registered, e.g. `"Euro 6"`                                       |
| `energy_class`                      | string | no       | Class letter `A`–`G`                                                                          |

WLTP is preferred; NEDC is used only when no WLTP figure exists for the vehicle (typically vehicles registered before 2018). The `*_standard` companion says which one applies.

**El och laddhybrid / Electric and plug-in hybrid**

| Field                                        | Type   | Required | Description                                                                    |
|----------------------------------------------|--------|----------|--------------------------------------------------------------------------------|
| `electric_vehicle_type`                      | string | no       | `Elbil` (BEV), `Laddhybrid` (PHEV), `Range Extender (REX)` or `Hybrid` (non-plug-in). Omitted for pure combustion vehicles and mild hybrids |
| `battery_capacity_gross_kwh`                 | float  | no       | Gross battery capacity in kWh                                                  |
| `battery_capacity_net_kwh`                   | float  | no       | Net / usable battery capacity in kWh                                           |
| `electricity_consumption_combined_kwh_100km` | float  | no       | Combined electricity consumption in kWh/100 km (WLTP)                          |
| `electric_range_km`                          | int32  | no       | Electric range in km (WLTP)                                                    |

The four battery/range/consumption fields are emitted for chargeable vehicles only (`electric_vehicle_type` = `Elbil`, `Laddhybrid` or `Range Extender (REX)`). A non-plug-in hybrid has no WLTP electric range and its small buffer battery is not comparable with a traction battery, so these fields are omitted for it.

**Mått och utrymme / Dimensions and space**

| Field                 | Type   | Required | Description                                                                              |
|-----------------------|--------|----------|------------------------------------------------------------------------------------------|
| `length_mm`           | int32  | no       | Overall length in mm                                                                     |
| `width_mm`            | int32  | no       | Overall width in mm                                                                      |
| `height_mm`           | int32  | no       | Overall height in mm                                                                     |
| `seats`               | int32  | no       | Number of seats                                                                          |
| `doors`               | int32  | no       | Number of doors                                                                          |
| `segment`             | string | no       | Size class / segment, free text, e.g. `"Stor SUV"`. Closed set, see [metadata values](metadata-values.md#segment) |
| `trunk_volume_rear_l` | int32  | no       | Rear luggage compartment in litres                                                       |
| `trunk_volume_front_l`| int32  | no       | Front luggage compartment ("frunk") in litres. Only present for vehicles that have one   |

**Vikter / Weights**

| Field                          | Type  | Required | Description                                                                 |
|--------------------------------|-------|----------|-----------------------------------------------------------------------------|
| `curb_weight_kg`               | int32 | no       | Registered curb weight (tjänstevikt) in kg                                  |
| `max_load_kg`                  | int32 | no       | Registered maximum load (maxlast) in kg                                     |
| `max_trailer_weight_braked_kg` | int32 | no       | Maximum braked trailer weight in kg, the registered figure at 12 % incline  |

**Säkerhet / Safety**

| Field                 | Type  | Required | Description                                                  |
|-----------------------|-------|----------|--------------------------------------------------------------|
| `euro_ncap_rating`    | int32 | no       | Euro NCAP overall rating, `1`–`5` stars                      |
| `euro_ncap_test_year` | int32 | no       | Year of the Euro NCAP test the rating comes from             |

**Administrativt / Administrative**

| Field              | Type  | Required | Description                                                                                             |
|--------------------|-------|----------|---------------------------------------------------------------------------------------------------------|
| `number_of_owners` | int32 | no       | Number of registered owners in the Swedish registry, including the current one                          |
| `imported`         | bool  | no       | Whether the vehicle was imported to Sweden as a used vehicle                                            |
| `annual_tax_sek`   | int32 | no       | Annual vehicle tax in SEK                                                                               |

**Garanti / Warranty**

| Field                         | Type  | Required | Description                                                                                       |
|-------------------------------|-------|----------|---------------------------------------------------------------------------------------------------|
| `manufacturer_warranty_years` | int32 | no       | Length in years of the manufacturer's new-car warranty for this model. Not adjusted for the vehicle's age |

**Rules that apply to every field above**

- A numeric field is omitted when its value would be zero and zero is not meaningful. `co2_combined_g_km` keeps zero, since 0 g/km is the real figure for an electric vehicle.
- Every field carries exactly one value; a text field is never a list or a joined string.
- Units are fixed by the field name.
- Strings such as `drivetrain`, `euro_class`, `energy_class` and `segment` are Swedish. The possible values are listed in [metadata values](metadata-values.md).

##### Not available

| Field                      | Note                                                                                         |
|----------------------------|----------------------------------------------------------------------------------------------|
| Max laddeffekt DC (kW)     | Not available in Wayke's vehicle data. Can be added if a data source is agreed later.        |
| I trafik (ja/nej)          | Not sent. The registration status Wayke holds is not current enough at listing time. Can be added when a live registry source is in place. |

## Lifecycle

1. **Branch activates Tradera** -> `branch.updated` event is published with branch identity.
2. **Ad is published or updated** -> `ad.updated` event with `status: "published"` and full vehicle data.
3. **Ad is unpublished** (sold, removed, or branch deactivates) -> `ad.updated` event with `status: "unpublished"`.

When a branch deactivates the Tradera integration, one `ad.updated` event with `status: "unpublished"` is emitted for each ad that was published at the time of deactivation.

## Notes

- All events for a given branch share the same partition key (`subject: branchId`), guaranteeing strict per-branch ordering.
- The `price_sek` field is a decimal string to avoid floating-point precision issues.
- The `engineBaseType` and `brandSpecific` fields use camelCase per the agreed contract; all other fields use snake_case.
