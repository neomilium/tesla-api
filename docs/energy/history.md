# Energy History

These endpoints for retreiving the data about the energy generated or stored by the products at a site, such as the power generated by Solar panels and the energy stored in a Powerwall.

The statistics can either be instantaneous power generation/storage in watts at 15 minute intervals for a day, or cumulative energy in kWh generated/stored in a day, week, month, or year. Depending on your equipment, some statistics may be unsupported and always return 0.

## GET `/api/1/energy_sites/{site_id}/history`

Retrieves either the power generation/storage (watts) for the previous day, or the cumulative energy generation/storage (kWh) for a specified recent period.

### Request parameters

| Field    | Type             | Example                           | Description                                                                                                                             |
| :------- | :--------------- | :-------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------- |
| `kind`   | String, required | `power` or `energy`               | `power` selects power statistics, in watts, at 15-minute intervals. `energy` selects energy statistics, in kWh, for a specified period. |
| `period` | String           | `day`, `week`, `month`, or `year` | The time span for energy statistics to cover.                                                                                           |

When `kind=power`, the `period` parameter is not required, and is ignored if present. When `kind=energy`, the `period` parameter is required and the value determines the time span of the retrieved data items:

| Value   | Statistics returned                                                                                                                    |
| :------ | :------------------------------------------------------------------------------------------------------------------------------------- |
| `day`   | Total kWh for yesterday, and for today up to the current time.                                                                         |
| `week`  | Total kWh for each of the past 7 days, not including today.                                                                            |
| `month` | Total kWh for each of the past 4 weeks, not including the current week.<br>Weeks are Sunday to Saturday; timestamps are for Saturdays. |
| `year`  | Total kWh for each of the past 12 months, not including the current month.                                                             |

### Response when `kind=power`

```json
{
  "response": {
    "serial_number": "313dbc37-555c-45b1-83aa-62a4ef9ff7ac",
    "time_series": [
      {
        "timestamp": "2022-05-13T00:00:00-07:00",
        "solar_power": 0,
        "battery_power": 0,
        "grid_power": 0,
        "grid_services_power": 0,
        "generator_power": 0
      }
    ]
  }
}
```

The `solar_power` value is the power being generated, in watts, at the given timestamp. To save space, only the first entry is shown in the `time_series` array. In a real response there are many entries in the array: one entry for each 15 minutes, starting at the beginning (midnight) of the previous day and ending at the current time and day. The entries cover 24 - 48 hours, depending on when the request is made. Depending on your equipment, some nighttime samples may be omitted if they are all zeroes.

### Response when `kind=energy`

```json
{
  "response": {
    "serial_number": "313dbc37-555c-45b1-83aa-62a4ef9ff7ac",
    "period": "week",
    "time_series": [
      {
        "timestamp": "2022-05-09T01:00:00-07:00",
        "solar_energy_exported": 18630,
        "generator_energy_exported": 0,
        "grid_energy_imported": 0,
        "grid_services_energy_imported": 0,
        "grid_services_energy_exported": 0,
        "grid_energy_exported_from_solar": 0,
        "grid_energy_exported_from_generator": 0,
        "grid_energy_exported_from_battery": 0,
        "battery_energy_exported": 0,
        "battery_energy_imported_from_grid": 0,
        "battery_energy_imported_from_solar": 0,
        "battery_energy_imported_from_generator": 0,
        "consumer_energy_imported_from_grid": 0,
        "consumer_energy_imported_from_solar": 0,
        "consumer_energy_imported_from_battery": 0,
        "consumer_energy_imported_from_generator": 0
      }
    ]
  }
}
```

The `solar_energy_exported` value is the energy generated by the solar panels, in kWh. To save space, only the first entry is shown in the `time_series` array. In a real response there are multiple entries in the array, depending on the value of the `period` parameter. For example, if the `period` value is `week`, there are 7 entries, each one representing the energy generated for one day.

## GET `/api/1/energy_sites/{site_id}/calendar_history`

Retrieves either the power generation/storage (watts) at 15-minute intervals for a given day, or the energy generation/storage (kWh) for a specified period.

### Request parameters

| Field      | Type                | Example                                       | Description                                                                                                                             |
| :--------- | :------------------ | :-------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------- |
| `kind`     | String, required    | `power` or `energy`                           | `power` selects power statistics, in watts, at 15-minute intervals. `energy` selects energy statistics, in kWh, for a specified period. |
| `end_date` | Date/time, required | `2022-04-29T14:15:00-07:00`                   | Specifies the last day for which statistics are retrieved.                                                                              |
| `period`   | String              | `day`, `week`, `month`, `year`, or `lifetime` | The time span for energy statistics to cover.                                                                                           |

The format for the `end_date` parameter value is "yyyy-mm-ddThh:mm:ss-hh:mm". Specify your local time zone offset for best clarity. Universal time is accepted in the format "yyyy-mm-ddThh:mm:ssZ", but the time is converted to your local time zone, which could also change the date.

When `kind=power`, the `period` parameter is not required, and is ignored if present. When `kind=energy`, the `period` parameter is required and the value determines the time span of the retrieved data items:

| Value      | Statistics returned                                                                        |
| :--------- | :----------------------------------------------------------------------------------------- |
| `day`      | Total kWh for the day specified by `end_date`.                                             |
| `week`     | Total kWh for each day from the previous Monday to, and including, `end_date`.             |
| `month`    | Total kWh for each day from the first of the month to, and including,`end_date`.           |
| `year`     | Total kWh for each calendar month from the previous January to, and including, `end_date`. |
| `lifetime` | Total kWh for each calendar year from installation to, and including, `end_date`.          |

The handling of partial periods is a bit inconsistent at this time. If `end_date` specifies a time in the middle of the day, then statistics for a partial day are reported when `period=day`, but statistics for the entire day are reported when `period=week` or `period=month`. For `period=year` or `period=lifetime`, an `end_date` in mid-month or mid-year returns statistics for a partial month or partial year.

### Response when `kind=power`

```json
{
  "response": {
    "serial_number": "313dbc37-555c-45b1-83aa-62a4ef9ff7ac",
    "time_zone_offset": -420,
    "time_series": [
      {
        "timestamp": "2022-04-29T00:00:00-07:00",
        "solar_power": 0,
        "battery_power": 0,
        "grid_power": 0,
        "grid_services_power": 0,
        "generator_power": 0
      }
    ]
  }
}
```

The `solar_power` value is the power being generated, in watts, at the given timestamp. To save space, only the first entry is shown in the `time_series` array. In a real response, there is one entry for each 15 minutes, starting at the beginning (midnight) of the specified date and ending at the specified time. To retrieve a full day's records, specify an ending time after sunset, or 23:45:00. Don't use 00:00:00 for the time, or you'll retrieve an empty list.

### Response when `kind=energy`

```json
{
  "response": {
    "serial_number": "313dbc37-555c-45b1-83aa-62a4ef9ff7ac",
    "period": "year",
    "time_zone_offset": -420,
    "time_series": [
      {
        "timestamp": "2021-01-01T01:00:00-07:00",
        "solar_energy_exported": 152680,
        "generator_energy_exported": 0,
        "grid_energy_imported": 0,
        "grid_services_energy_imported": 0,
        "grid_services_energy_exported": 0,
        "grid_energy_exported_from_solar": 0,
        "grid_energy_exported_from_generator": 0,
        "grid_energy_exported_from_battery": 0,
        "battery_energy_exported": 0,
        "battery_energy_imported_from_grid": 0,
        "battery_energy_imported_from_solar": 0,
        "battery_energy_imported_from_generator": 0,
        "consumer_energy_imported_from_grid": 0,
        "consumer_energy_imported_from_solar": 0,
        "consumer_energy_imported_from_battery": 0,
        "consumer_energy_imported_from_generator": 0
      }
    ]
  }
}
```

`solar_energy_exported` is the energy generated by the solar panels, in kWh. To save space only the first entry in the `time_series` array is shown. In a real response, the number of entries in the array depends on the `period` and `end_date` values. Timestamps in the response represent the start of a period; e.g., monthly statistics for February show a date of February 1st in the timestamp, and daily statistics for February 15th show a timestamp of 1:00 AM on February 15th.