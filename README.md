# Vendor API Documentation

This document provides details on all available endpoints for accessing live data.  
All requests require an **API key** via the `X-API-Key` header.


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


## Pagination

Endpoints support **cursor-based pagination**:

- `after_id` (optional): ID of the last record from the previous page (default: 0)
- `limit` (optional): Maximum number of rows to return (default: 500, max: 1000)

**Response fields**:

- `next_cursor`: ID to use in `after_id` for the next page  
- `has_more`: Boolean indicating if more records exist



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
  'https://insight.delagua.org/wp-json/vendor-api/v1/contacts?country=SL&after_id=0&limit=100' \
  -H 'X-API-Key: your_api_key_here'
```

**Example Response:**

```json
{
  "data": [
    { "DAID": 101, "FullName": "John Doe", "RegionID": 5, "ProjectID": 3699 },
    { "DAID": 102, "FullName": "Jane Smith", "RegionID": 5, "ProjectID": 3699 }
  ],
  "next_cursor": 102,
  "has_more": true
}
```


## 2. `/barcodes`

**Description:** Returns barcodes scanned, associated investors, and destination regions.

**Request Parameters:**

| Parameter | Type   | Required | Description                      |
| --------- | ------ | -------- | -------------------------------- |
| country   | string | Yes      | Country code: `SL`, `RW`, `GAM`  |
| after_id  | int    | No       | Last ID from previous page      |
| limit     | int    | No       | Max rows to return (default 100) |

**Example Request:**

```bash
curl -X GET \
  'https://insight.delagua.org/wp-json/vendor-api/v1/barcodes?country=GAM&after_id=0&limit=100' \
  -H 'X-API-Key: your_api_key_here'
```

**Example Response:**

```json
{
  "data": [
    { "ID": "1", "Barcode": "BC12345", "Investor": "Investor A", "RegionName": "Western", "TID": 5001, "DestID": 12, "ProjectID": 3699 },
    { "ID": "2", "Barcode": "BC12346", "Investor": "Investor B", "RegionName": "Northern", "TID": 5002, "DestID": 15, "ProjectID": 3699 }
  ],
  "next_cursor": 2,
  "has_more": true
}
```


## 3. `/locations/villages`

**Description:** Returns village locations with hierarchical boundaries.

**Request Parameters:**

| Parameter | Type   | Required | Description                       |
| --------- | ------ | -------- | --------------------------------- |
| country   | string | Yes      | Country code: `SL`, `RW`, `GAM`   |
| after_id  | int    | No       | Last VillageID from previous page |
| limit     | int    | No       | Max rows to return (default 50)   |

**Example Request:**

```bash
curl -X GET \
  'https://insight.delagua.org/wp-json/vendor-api/v1/locations/villages?country=SL&after_id=0&limit=50' \
  -H 'X-API-Key: your_api_key_here'
```

**Example Response (Sierra Leone):**

```json
{
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
      "Village": "Sample Village"
    }
  ],
  "next_cursor": 1023,
  "has_more": true
}
```

**Note:** The fields returned vary by country. Unused fields will be `null`.


## 4. `/beneficiaries`

**Description:** Returns beneficiaries with hierarchical location data.

**Request Parameters:**

| Parameter | Type   | Required | Description                       |
| --------- | ------ | -------- | --------------------------------- |
| country   | string | Yes      | Country code: `SL`, `RW`, `GAM`   |
| after_id  | int    | No       | Last HHID from previous page |
| limit     | int    | No       | Max rows to return (default 500)  |

**Example Request:**

```bash
curl -X GET \
  'https://insight.delagua.org/wp-json/vendor-api/v1/beneficiaries?country=SL&after_id=0&limit=100' \
  -H 'X-API-Key: your_api_key_here'
```

**Example Response (Sierra Leone):**

```json
{
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
      "ProjectID": 3699
    }
  ],
  "next_cursor": 5555,
  "has_more": true
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

1. All endpoints are **read-only**.
2. Large datasets are paginated via `after_id` and `limit`.
3. Responses include `next_cursor` and `has_more` for easy paging.
4. Ensure `X-API-Key` header is included in every request.
5. JSON structure is **consistent across countries**; only unused fields are `null`.
