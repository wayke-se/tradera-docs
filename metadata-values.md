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

## Fields in the extended vehicle specification

Fields of the [extended vehicle specification](kafka.md#extended-vehicle-specification). Closed sets are enumerations, the rest are free text or numbers.

| Field | Possible values |
|---|---|
| `registration_date` | `yyyy-MM-dd` |
| `drivetrain` | closed set, see below |
| `engine_power_hp`, `engine_torque_nm`, `engine_displacement_cc`, `number_of_gears` | integer |
| `engine_displacement_litres`, `acceleration_0_100_s` | decimal |
| `fuel_consumption_combined_l_100km`, `electricity_consumption_combined_kwh_100km` | decimal |
| `fuel_consumption_standard`, `co2_standard` | `WLTP` or `NEDC` |
| `co2_combined_g_km`, `electric_range_km` | integer |
| `euro_class` | see below for observed values |
| `energy_class` | single letter `A`–`G`, see below |
| `electric_vehicle_type` | closed set, see below |
| `battery_capacity_gross_kwh`, `battery_capacity_net_kwh` | decimal |
| `length_mm`, `width_mm`, `height_mm`, `seats`, `doors` | integer |
| `segment` | closed set, see below |
| `trunk_volume_rear_l`, `trunk_volume_front_l` | integer |
| `curb_weight_kg`, `max_load_kg`, `max_trailer_weight_braked_kg` | integer |
| `euro_ncap_rating` | integer `1`–`5` |
| `euro_ncap_test_year` | integer, four-digit year |
| `imported` | boolean |
| `number_of_owners`, `annual_tax_sek`, `manufacturer_warranty_years` | integer |

## drivetrain

| Emitted |
|---|
| Framhjulsdrift |
| Bakhjulsdrift |
| Fyrhjulsdrift |

Omitted when only the coarser "Tvåhjulsdrift" (two-wheel drive) level is known; that value is never emitted.

## euro_class

Free text as registered. Observed values:

| Emitted |
|---|
| Euro 2 |
| Euro 4 |
| Euro 5 |
| Euro 6 |

Older vehicles may carry other Euro levels.

## energy_class

Class letter. Observed values: `A`, `B`, `C`, `D`, `E`, `G`; the full range is `A`–`G`.

## electric_vehicle_type

| Emitted | Meaning |
|---|---|
| Elbil | Battery electric vehicle |
| Laddhybrid | Plug-in hybrid |
| Range Extender (REX) | Electric vehicle with a range-extender engine (e.g. BMW i3 REX) |
| Hybrid | Non-plug-in (full) hybrid |

Omitted for pure combustion vehicles and mild hybrids. Battery, range and electricity-consumption fields are sent only for `Elbil`, `Laddhybrid` and `Range Extender (REX)`.

## segment

Size class. Observed values:

| Emitted |
|---|
| Småbil |
| Liten Familjebil |
| Liten SUV |
| Liten MPV |
| Mellanstor Familjebil |
| Mellanstor SUV |
| Mellanstor premiumbil |
| Stor Familjebil |
| Stor SUV |
| Stor MPV |
| Premiumbil |
| Sportbil |
| Minibuss |
| Skåpbil |
| Pickup |
| Transportchassi |

Full enumeration not maintained in the repo; other classes may appear for less common vehicle types.

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
