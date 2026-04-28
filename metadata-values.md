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
