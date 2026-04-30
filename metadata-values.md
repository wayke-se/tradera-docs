# Metadata Values (SE)

Reference for possible values on metadata fields published from Wayke to Tradera.

## Fields published on `CarFields`

JSON field names as they appear on the Tradera Kafka contract:

| Field | Possible values |
|---|---|
| `brand` | free text |
| `model` | free text |
| `body_type` | free text |
| `model_year` | integer |
| `chassis_type` | closed set, see below |
| `engineBaseType` | see below |
| `fuels` | closed set, see below |
| `transmission` | see below |
| `color` | closed set, see below |
| `equipments` | array of free-text strings |
| `registration_number`, `vin`, `mileage_km` | direct |
| `brandSpecific.model` | free text, OEM model grouping |
| `brandSpecific.variant` | free text, OEM variant grouping |
| `brandSpecific.badge` | free text, OEM sales badge |
| `brandSpecific.terms` | array of free-text strings (drivetrain / body / trim tags) |

## chassis_type

| Emitted |
|---|
| Coupé |
| Cabriolet |
| Halvkombi |
| Lätta nyttofordon |
| Minibuss |
| Pickup |
| Mikrobil |
| Sedan |
| Kombi |
| SUV |
| Buss |
| Lastbil |
| Motorcykel |
| Moped |
| Snöskoter |
| Campingbil |

## engineBaseType

High-level powertrain type. Free-text Swedish string. Observed values:

| Emitted |
|---|
| Elektrisk |
| Förbränningsmotor |
| Laddhybrid |

Full enumeration not maintained in the repo.

## color

| Emitted |
|---|
| Röd |
| Grön |
| Silver |
| Blå |
| Svart |
| Grå |
| Vit |
| Lila |
| Rosa |
| Brun |
| Guld |
| Gul |
| Orange |
| Turkos |
| Beige |
| Violet |
| Flerfärgad |

## fuels

Array of strings.

| Emitted |
|---|
| Bensin |
| Diesel |
| El |
| Etanol |
| Gas |
| Gasol |
| Hybrid |
| Laddhybrid |
| Naturgas |
| Vätgas |

## transmission

| Emitted |
|---|
| Automat |
| Manuell |

Format templates applied:

| Variant | Template |
|---|---|
| automatic | `{gears}-stegad {value}` |
| manual | `{gears}-växlad {value}` |
| default | `{value}` |

## brandSpecific

Brand-curated taxonomy nested inside `car_fields.brandSpecific`. Sourced from the federated `Vehicle.brandSpecific` block in item-enrichment-service. The whole object is omitted when the vehicle has no brand-specific data; individual fields are omitted when empty.

```json
"brandSpecific": {
  "model": "A4",
  "variant": "40",
  "badge": "40 TDI quattro 2.0",
  "terms": ["quattro", "Avant"]
}
```

| Field     | Type     | Description                                                                       |
|-----------|----------|-----------------------------------------------------------------------------------|
| `model`   | string   | OEM model grouping (e.g. `"A4"`, `"5-Serie"`, `"XC60"`)                           |
| `variant` | string   | OEM variant under the model (e.g. `"40"`, `"540"`)                                |
| `badge`   | string   | Specific sales badge / configuration (e.g. `"40 TDI quattro 2.0"`)                |
| `terms`   | string[] | Free-form taxonomy terms — drivetrain, body, trim, etc. (e.g. `["quattro", "Avant"]`) |
