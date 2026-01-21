# Vendor API Documentation

This document provides details on all available endpoints for accessing live data.

All requests require an **API key** via the `X-API-Key` header.


## What’s New

Added support for **incremental sync** using a new `created_since` parameter on all endpoints (`/contacts`, `/barcodes`, `/locations/villages`, `/beneficiaries`).  

- Use `created_since` to fetch **only records created after a specific timestamp**, avoiding repeated full dataset downloads.  
- Combined with the existing `after_id` cursor, this ensures **safe pagination even when multiple rows share the same timestamp** (common due to batch imports).  
- Responses now include a `meta` object containing `last_timestamp` and `last_cursor` to simplify resuming syncs.  

## Base URL

```
https://insight.delagua.org/wp-json/vendor-api/v1/
```


## Authentication

All requests must include the HTTP header:

```
X-API-Key: <your_api_key>
```

If the key is invalid or missing, the API will return **HTTP 401 Unauthorized**.


## Pagination & Incremental Sync

Endpoints support:
- Cursor-based pagination via after_id
- Incremental sync via created_since

| Parameter       | Type   | Required | Description                                                                                                                      |
| --------------- | ------ | -------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `after_id`      | int    | No       | ID of the last record from the previous page (default: 0)                                                                        |
| `created_since` | string | No       | ISO-8601 timestamp. Only records **created after this timestamp** will be returned. Default: `1970-01-01 00:00:00` for full sync |
| `limit`         | int    | No       | Max number of rows per request (default 500, max 1000)                                                                           |
| `country`       | string | Yes      | Country code: `SL`, `RW`, or `GAM`                                                                                               |

Response metadata:
- meta.count – Number of rows returned
- meta.has_more – Boolean, true if more rows are available
- meta.last_timestamp – Timestamp of the last record returned
- meta.last_cursor – ID of the last record returned (use as after_id for next request)


# Endpoints

## 1. `/contacts`

**Description:** Returns a list of field staff / contacts.

**Request Parameters:**

| Parameter | Type   | Required | Description                  |
|-----------|--------|----------|------------------------------|
| country   | string | Yes      | Country code: `SL`, `RW`, `GAM` |
| after_id  | int    | No       | Last DAID from previous page |
| limit     | int    | No       | Max rows to return (default 100) |

**Example Request:**

```bash
curl -X GET \
  'https://insight.delagua.org/wp-json/vendor-api/v1/contacts?country=SL&after_id=0&created_since=2026-01-20%2012:00:00&limit=100' \
  -H 'X-API-Key: your_api_key_here'
```

**Example Response:**

```json
{
  "meta": {
    "count": 2,
    "has_more": true,
    "last_timestamp": "2026-01-20 12:10:00",
    "last_cursor": 102
  },
  "data": [
    { "DAID": 101, "FullName": "John Doe", "RegionID": 5, "ProjectID": 3699, "Timestamp": "2026-01-20 08:50:59" },
    { "DAID": 102, "FullName": "Jane Smith", "RegionID": 5, "ProjectID": 3699, "Timestamp": "2026-01-20 08:50:59" }
  ]
}
```


## 2. `/barcodes`

**Description:** Returns barcodes scanned, associated investors, and destination regions.

**Request Parameters:**

| Parameter     | Type   | Required | Description                                |
| ------------- | ------ | -------- | ------------------------------------------ |
| country       | string | Yes      | Country code: `SL`, `RW`, `GAM`            |
| after_id      | int    | No       | Last barcode ID from previous page         |
| created_since | string | No       | Fetch records created after this timestamp |
| limit         | int    | No       | Max rows to return (default 500)           |

**Example Request:**

```bash
curl -X GET \
  'https://insight.delagua.org/wp-json/vendor-api/v1/barcodes?country=GAM&after_id=0&created_since=2026-01-20%2012:00:00&limit=100' \
  -H 'X-API-Key: your_api_key_here'
```

**Example Response:**

```json
{
  "meta": {
    "count": 2,
    "has_more": true,
    "last_timestamp": "2026-01-20 12:12:00",
    "last_cursor": 2
  },
  "data": [
    { "ID": 1, "Barcode": "BC12345", "Investor": "Investor A", "RegionName": "Western", "TID": 5001, "DestID": 12, "ProjectID": 3699, "Timestamp": "2026-01-20 08:50:59" },
    { "ID": 2, "Barcode": "BC12346", "Investor": "Investor B", "RegionName": "Northern", "TID": 5002, "DestID": 15, "ProjectID": 3699, "Timestamp": "2026-01-20 08:50:59" }
  ]
}
```


## 3. `/locations/villages`

**Description:** Returns village locations with hierarchical boundaries.

**Request Parameters:**

| Parameter     | Type   | Required | Description                                |
| ------------- | ------ | -------- | ------------------------------------------ |
| country       | string | Yes      | Country code: `SL`, `RW`, `GAM`            |
| after_id      | int    | No       | Last VillageID from previous page          |
| created_since | string | No       | Fetch records created after this timestamp |
| limit         | int    | No       | Max rows to return (default 50)            |

**Example Request:**

```bash
curl -X GET \
  'https://insight.delagua.org/wp-json/vendor-api/v1/locations/villages?country=SL&after_id=0&created_since=2026-01-20%2012:00:00&limit=50' \
  -H 'X-API-Key: your_api_key_here'
```

**Example Response (Sierra Leone):**

```json
{
  "meta": {
    "count": 1,
    "has_more": true,
    "last_timestamp": "2026-01-20 12:15:00",
    "last_cursor": 1023
  },
  "data": [
    {
      "VillageID": 1023,
      "Country": "Sierra Leone",
      "Province": null,
      "Region": "Eastern",
      "District": "Kenema",
      "Chiefdom": "Kandu Leema",
      "Sector": null,
      "Cell": null,
      "Village": "Sample Village",
      "Timestamp": "2025-01-24 09:31:01"
    }
  ]
}
```

**Note:** The fields returned vary by country. Unused fields will be `null`.


## 4. `/beneficiaries`

**Description:** Returns beneficiaries with hierarchical location data.

**Request Parameters:**

| Parameter     | Type   | Required | Description                                |
| ------------- | ------ | -------- | ------------------------------------------ |
| country       | string | Yes      | Country code: `SL`, `RW`, `GAM`            |
| after_id      | int    | No       | Last HHID from previous page               |
| created_since | string | No       | Fetch records created after this timestamp |
| limit         | int    | No       | Max rows to return (default 500)           |

**Example Request:**

```bash
curl -X GET \
  'https://insight.delagua.org/wp-json/vendor-api/v1/beneficiaries?country=SL&after_id=0&created_since=2026-01-20%2012:00:00&limit=100' \
  -H 'X-API-Key: your_api_key_here'
```

**Example Response (Sierra Leone):**

```json
{
  "meta": {
    "count": 1,
    "has_more": true,
    "last_timestamp": "2026-01-20 12:20:00",
    "last_cursor": 1001
  },
  "data": [
    {
      "HHID": 1001,
      "UID": 5555,
      "Name": "Jane Doe",
      "Phone": "+2327000000",
      "StoneFire": 1,
      "ReceivedStove": 1,
      "VillageID": 1023,
      "Region": "Eastern",
      "District": "Kenema",
      "Chiefdom": "Kandu Leema",
      "Province": null,
      "Sector": null,
      "Cell": null,
      "Village": "Sample Village",
      "WorkRegionGID": 1234,
      "CountryID": 18022,
      "ProjectID": 3699,
      "Timestamp": "2025-01-24 09:31:01"
    }
  ]
}
```

**Notes:**

* All countries return the **same set of fields**; unused fields are `null`.
* `CountryID` values:

  * Sierra Leone: 18022
  * Rwanda: 1
  * The Gambia: 40000


## Country Hierarchy Reference

| Country | Hierarchy Fields Returned                 |
| ------- | ----------------------------------------- |
| SL      | Region, District, Chiefdom, Village       |
| RW      | Province, District, Sector, Cell, Village |
| GAM     | Region, District, Village                 |


## General Notes

1. All endpoints are read-only.
2. Large datasets are paginated via `after_id` and `limit`.
3. `created_since` can be used to fetch only new records since the last sync.
4. Responses include `meta.last_timestamp` and `meta.last_cursor` for safe incremental syncs.
5. Ensure `X-API-Key` header is included in every request.
6. JSON structure is consistent across countries; only unused fields are `null`.


