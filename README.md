# Real Time Bayut Data API

[![Platform](https://img.shields.io/badge/Platform-RapidAPI-blue)](https://rapidapi.com/)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-success)]()
[![Uptime](https://img.shields.io/badge/Uptime-99.9%25-brightgreen)]()
[![Response Time](https://img.shields.io/badge/Latency-<400ms-blueviolet)]()

Real-time access to Bayut property data - Dubai and UAE listings, off-plan projects, agents, agencies, developers, and transaction history. No scraping, no proxies, clean JSON.

**RapidAPI:** https://rapidapi.com/happyendpoint/api/bayut14/
**Docs:** https://bayutapi.dev
**Website:** https://happyendpoint.com/apis/bayut

---

## Table of Contents

- [Overview](#overview)
- [Why use this API](#why-use-this-api)
- [Quick Start](#quick-start)
- [Authentication](#authentication)
- [Base URL](#base-url)
- [Endpoints](#endpoints)
  - [Health Check](#health-check)
  - [Location - Autocomplete](#location---autocomplete)
  - [Property Search](#property-search)
  - [Property Details](#property-details)
  - [Off-Plan Projects](#off-plan-projects)
  - [Agent Search](#agent-search)
  - [Agent Search by Name](#agent-search-by-name)
  - [Agent Details](#agent-details)
  - [Agent Properties](#agent-properties)
  - [Agency Search](#agency-search)
  - [Agency Search by Name](#agency-search-by-name)
  - [Agency Details](#agency-details)
  - [Agency Agents](#agency-agents)
  - [Agency Properties](#agency-properties)
  - [Developer Search by Name](#developer-search-by-name)
  - [Transaction History](#transaction-history)
  - [Amenities Search](#amenities-search)
- [Response Format](#response-format)
- [Error Handling](#error-handling)
- [Code Examples](#code-examples)
- [Common Location IDs](#common-location-ids)
- [Use Cases](#use-cases)
- [Pricing and Plans](#pricing-and-plans)
- [Bulk Data](#bulk-data)
- [FAQ](#faq)

---

## Overview

Bayut is the largest real estate portal in the UAE. It has hundreds of thousands of active listings across Dubai, Abu Dhabi, Sharjah, and the rest of the Emirates - covering properties for sale and rent, off-plan developments, agents, and agencies.

This API gives you programmatic access to that data. It is built and maintained by [Happy Endpoint](https://happyendpoint.com) and hosted on RapidAPI.

**What you can access:**

- Property listings for sale and rent across all UAE
- Off-plan and new development projects
- Full property details - photos, floor plans, amenities, agent info, coordinates
- Real estate agent profiles and their listings
- Real estate agency profiles and their listings
- Property developer search
- Historical transaction data (up to 24 months)
- Location search and autocomplete
- Amenity search for advanced filtering

**Key specs:**

| Spec | Value |
|---|---|
| Protocol | HTTPS / REST |
| Response format | JSON |
| Average response time | < 400ms |
| Coverage | UAE-wide (Dubai, Abu Dhabi, Sharjah, and more) |
| Authentication | API key via RapidAPI |
| Proxy required | No |
| Last updated | March 2026 |

---

## Why use this API

Bayut does not offer a public API. The only way to get their data without this API is to scrape it yourself - which means dealing with Cloudflare, rotating proxies, frequent HTML changes, and terms of service violations.

This API handles all of that for you. You get clean, structured JSON data without any of the infrastructure overhead.

**Compared to scraping:**

| | Scraping | This API |
|---|---|---|
| Setup time | Days to weeks | 15 minutes |
| Maintenance | Constant - breaks on site updates | None |
| Reliability | Unpredictable | 99.9% uptime |
| Speed | 5-30 seconds per page | < 400ms |
| Proxy costs | $200-500/month | Not needed |
| Legal risk | Violates ToS | None |
| Data quality | Inconsistent | Validated, structured JSON |

---

## Quick Start

### 1. Get your API key

Subscribe on RapidAPI - free plan available:
https://rapidapi.com/happyendpoint/api/bayut14/

### 2. Make your first request

```bash
curl --request GET \
  --url 'https://bayut14.p.rapidapi.com/health' \
  --header 'x-rapidapi-host: bayut14.p.rapidapi.com' \
  --header 'x-rapidapi-key: YOUR_API_KEY'
```

Response:
```json
{ "success": true }
```

### 3. Search for properties

```bash
curl --request GET \
  --url 'https://bayut14.p.rapidapi.com/search-property?purpose=for-sale&location_ids=5003&property_type=apartments&rooms=1&page=1' \
  --header 'x-rapidapi-host: bayut14.p.rapidapi.com' \
  --header 'x-rapidapi-key: YOUR_API_KEY'
```

`5003` is the location ID for Dubai Marina. Use `/autocomplete` to find IDs for other areas.

---

## Authentication

All requests require two headers:

```
x-rapidapi-key: YOUR_API_KEY
x-rapidapi-host: bayut14.p.rapidapi.com
```

Get your key by subscribing on RapidAPI: https://rapidapi.com/happyendpoint/api/bayut14/

---

## Base URL

```
https://bayut14.p.rapidapi.com
```

---

## Endpoints

### Health Check

```
GET /health
```

Simple health check to confirm the API is up.

**Response:**
```json
{ "success": true }
```

---

### Location - Autocomplete

```
GET /autocomplete
```

Search for locations across Dubai and the UAE. Returns location names, IDs, and slugs. The `externalID` field is what you pass as `location_ids` in all other endpoints.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Location search string. Min 1 character. Example: `dubai marina`, `palm jumeirah`, `downtown` |
| `langs` | string | No | Comma-separated language codes. Options: `en`, `ar`, `ru`, `zh`. Default: `en` |

**Example request:**
```bash
GET /autocomplete?query=dubai+marina&langs=en
```

**Example response:**
```json
{
  "success": true,
  "data": {
    "locations": [
      {
        "id": 36,
        "externalID": "5003",
        "name": { "en": "Dubai Marina" },
        "slug": { "en": "/dubai/dubai-marina" },
        "level": 2,
        "type": "neighborhood",
        "path": "Dubai > Dubai Marina",
        "adCount": 12450
      }
    ],
    "total": 1
  }
}
```

**Location levels:**
- Level 0 - UAE (whole country)
- Level 1 - Emirate (Dubai, Abu Dhabi, Sharjah, etc.)
- Level 2 - Neighbourhood (Dubai Marina, Downtown Dubai, etc.)

---

### Property Search

```
GET /search-property
```

The main search endpoint. Search properties for sale or rent across the UAE with a wide range of filters.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `purpose` | enum | Yes | `for-sale` or `for-rent` |
| `location_ids` | string | No | Comma-separated location externalIDs from `/autocomplete`. Example: `5003,5460` |
| `property_type` | string | No | See property types below |
| `rooms` | string | No | Comma-separated bedroom counts. `0` = studio. Example: `0,1,2` |
| `baths` | string | No | Comma-separated bathroom counts. Example: `1,2` |
| `price_min` | integer | No | Minimum price in AED |
| `price_max` | integer | No | Maximum price in AED |
| `area_min` | number | No | Minimum area in sqft |
| `area_max` | number | No | Maximum area in sqft |
| `completion_status` | enum | No | `completed`, `under-construction`, or `any` |
| `is_furnished` | enum | No | `furnished` or `unfurnished` |
| `amenities` | string | No | Comma-separated amenity names from `/amenities-search`. Example: `Swimming Pool,Gym` |
| `sort_order` | enum | No | `popular` (default), `latest`, `verified`, `trubroker_first`, `lowest_price`, `highest_price` |
| `rent_frequency` | enum | No | For rent only: `yearly` (default), `monthly`, `weekly`, `daily` |
| `has_video` | boolean | No | Filter to properties with video tours |
| `has_360_tour` | boolean | No | Filter to properties with 360 virtual tours |
| `has_floorplan` | boolean | No | Filter to properties with floor plans |
| `agent_ids` | string | No | Comma-separated agent IDs to filter by specific agents |
| `agency_ids` | string | No | Comma-separated agency externalIDs |
| `developer_ids` | string | No | Comma-separated developer IDs |
| `page` | integer | No | Page number, starts at 1. Returns 24 results per page. Default: 1 |
| `langs` | string | No | Language codes: `en`, `ar`, `ru`, `zh`. Default: `en` |

**Property types:**

Residential: `apartments`, `townhouses`, `penthouse`, `villas`, `villa-compound`, `hotel-apartments`, `residential-plots`, `residential-floors`, `residential-building`

Commercial: `offices`, `warehouses`, `commercial-villas`, `commercial-plots`, `commercial-buildings`, `industrial-land`, `showrooms`, `shops`, `labour-camps`, `bulk-units`, `commercial-floors`, `factories`, `mixed-use-land`

Use `residential` or `commercial` to search all sub-types at once.

**Example request:**
```bash
GET /search-property?purpose=for-sale&location_ids=5003&property_type=apartments&rooms=1,2&price_max=2000000&sort_order=popular&page=1
```

**Example response:**
```json
{
  "success": true,
  "data": {
    "properties": [
      {
        "externalID": "10283980",
        "title": { "en": "2BR Apartment in Dubai Marina" },
        "price": 1500000,
        "rentFrequency": null,
        "rooms": 2,
        "baths": 3,
        "area": 120.5,
        "purpose": "for-sale",
        "completionStatus": "completed",
        "furnishingStatus": "furnished",
        "isVerified": true,
        "referenceNumber": "REF-12345",
        "contactName": "John Smith",
        "coverPhoto": { "url": "https://..." },
        "photoCount": 15,
        "videoCount": 1,
        "location": [
          { "id": 1, "name": { "en": "UAE" }, "level": 0 },
          { "id": 2, "name": { "en": "Dubai" }, "level": 1 },
          { "id": 36, "name": { "en": "Dubai Marina" }, "level": 2 }
        ],
        "agency": { "name": "Example Realty", "externalID": "10212" },
        "ownerAgent": { "name": "John Smith", "externalID": "2518657" },
        "amenities": [
          { "text": "Swimming Pool" },
          { "text": "Gym" },
          { "text": "Parking" }
        ],
        "geography": { "lat": 25.0819, "lng": 55.1367 },
        "createdAt": 1709632442,
        "updatedAt": 1711407610
      }
    ],
    "total": 1250,
    "page": 1,
    "totalPages": 53,
    "hitsPerPage": 24
  }
}
```

---

### Property Details

```
GET /property-details
```

Get full details for a single property by its `externalID`. Returns everything - all photos, floor plans, full amenity list, agent contact details, geographic coordinates, and more.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `external_id` | string | Yes | Property externalID from `/search-property` response |
| `langs` | string | No | Language code. Default: `en` |

**Example request:**
```bash
GET /property-details?external_id=13495633&langs=en
```

---

### Off-Plan Projects

```
GET /search-new-projects
```

Search for off-plan and new development projects in Dubai and the UAE. These are properties under construction or recently launched by developers.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `location_ids` | string | No | Comma-separated location externalIDs |
| `property_type` | string | No | Same options as `/search-property`. Default: `residential` |
| `developer_ids` | string | No | Filter by developer ID from `/developer-search-by-name` |
| `completion_percentage` | enum | No | `any` (default), `0-25`, `25-50`, `50-75`, `75-100` |
| `pre_handover_payment` | integer | No | Max pre-handover payment percentage (0-100). Filters deals where you pay at most this % before handover |
| `price_min` | integer | No | Minimum price in AED |
| `price_max` | integer | No | Maximum price in AED |
| `area_min` | number | No | Minimum area in sqft |
| `area_max` | number | No | Maximum area in sqft |
| `rooms` | string | No | Comma-separated bedroom counts |
| `baths` | string | No | Comma-separated bathroom counts |
| `sort_order` | enum | No | `popular`, `latest`, `lowest_price`, `highest_price` |
| `page` | integer | No | Page number. Default: 1 |
| `langs` | string | No | Language codes. Default: `en` |

**Example request:**
```bash
GET /search-new-projects?location_ids=5003&completion_percentage=0-25&pre_handover_payment=40&page=1
```

---

### Agent Search

```
GET /agent-search
```

Search for real estate agents by location, purpose, and category. Returns 40 agents per page.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `location_ids` | string | No | Comma-separated location externalIDs |
| `purpose` | enum | No | `for-sale` (default) or `for-rent` |
| `category` | enum | No | `residential` (default) or `commercial` |
| `completion_status` | enum | No | `any` (default), `completed`, `under-construction` |
| `page` | integer | No | Page number. Returns 40 per page. Default: 1 |
| `langs` | string | No | Language code. Default: `en` |

**Example request:**
```bash
GET /agent-search?location_ids=5003&purpose=for-sale&category=residential&page=1
```

---

### Agent Search by Name

```
GET /agent-search-by-name
```

Search agents by name. Returns 20 agents per page.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Agent name to search for. Example: `john`, `anna`, `vipul` |
| `page` | integer | No | Page number. Default: 1 |
| `langs` | string | No | Language code. Default: `en` |

**Example request:**
```bash
GET /agent-search-by-name?query=anna&page=1
```

---

### Agent Details

```
GET /agent-details
```

Get full profile for a specific agent - contact info, languages, service areas, TruBroker badge status, and listing statistics.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `agent_id` | string | Yes | Agent externalID from `/agent-search` or `/agent-search-by-name` |
| `langs` | string | No | Language codes. Default: `en` |

**Example request:**
```bash
GET /agent-details?agent_id=2518657&langs=en
```

---

### Agent Properties

```
GET /agent-properties
```

Get all listings by a specific agent. Returns 25 properties per page.

**Important:** This endpoint uses `ownerID`, not `externalID`. Get the `ownerID` from the `/agent-details` response - they are different fields.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `owner_id` | string | Yes | Agent ownerID from `/agent-details` response |
| `page` | integer | No | Page number. Default: 1 |
| `langs` | string | No | Language codes. Default: `en` |

**Example request:**
```bash
GET /agent-properties?owner_id=2243594&page=1
```

---

### Agency Search

```
GET /agency-search
```

Search for real estate agencies by location. Returns agencies with active listings in the specified area. Returns 40 agencies per page.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `location_id` | string | Yes | Single location externalID. Use `1` for all of Dubai, `3` for all of Abu Dhabi |
| `page` | integer | No | Page number. Default: 1 |
| `langs` | string | No | Language codes. Default: `en` |

**Example request:**
```bash
GET /agency-search?location_id=1&page=1
```

---

### Agency Search by Name

```
GET /agency-search-by-name
```

Search agencies by name. Returns 80 results per page.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Agency name. Example: `emaar`, `metropolitan`, `white` |
| `page` | integer | No | Page number. Default: 1 |
| `langs` | string | No | Language code. Default: `en` |

**Example request:**
```bash
GET /agency-search-by-name?query=emaar&page=1
```

---

### Agency Details

```
GET /agency-details
```

Get full profile for a specific agency - contact info, listing statistics, and branding.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `agency_id` | string | Yes | Agency ID from `/agency-search` response |

**Example request:**
```bash
GET /agency-details?agency_id=8566
```

---

### Agency Agents

```
GET /agency-agents
```

Get all agents working at a specific agency.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `agency_id` | string | Yes | Agency ID from `/agency-search` or `/agency-details` |
| `page` | number | No | Page number. Default: 1 |

**Example request:**
```bash
GET /agency-agents?agency_id=10212&page=1
```

---

### Agency Properties

```
GET /agency-properties
```

Get all listings by a specific agency. Returns 25 properties per page.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `agency_external_id` | string | Yes | Agency externalID from `/agency-search` or `/agency-search-by-name` |
| `page` | integer | No | Page number. Default: 1 |
| `langs` | string | No | Language codes. Default: `en` |

**Example request:**
```bash
GET /agency-properties?agency_external_id=106317&page=1
```

---

### Developer Search by Name

```
GET /developer-search-by-name
```

Search property developers by name. Returns developer IDs you can use in `/search-new-projects` to filter off-plan projects by developer. Returns 80 results per page.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | Yes | Developer name. Example: `emaar`, `damac`, `sobha`, `nakheel` |
| `page` | integer | No | Page number. Default: 1 |
| `langs` | string | No | Language code. Default: `en` |

**Example request:**
```bash
GET /developer-search-by-name?query=emaar&page=1
```

---

### Transaction History

```
GET /transactions
```

Get historical property transaction data for Dubai and the UAE. Covers both sale and rental transactions. One of the most useful endpoints for market research and investment analysis.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `purpose` | enum | Yes | `for-sale` or `for-rent` |
| `location_ids` | string | No | Comma-separated location externalIDs |
| `category_ids` | string | No | `residential`, `commercial`, `apartments`, `villas`, `townhouses`, `offices`, etc. Or use numeric IDs: `1` (residential), `2` (commercial), `4` (apartments), `3` (villas) |
| `completion_status` | enum | No | `any` (default), `completed`, `under-construction` |
| `time_period` | enum | No | `1m`, `3m`, `6m`, `12m` (default), `24m` |
| `beds` | string | No | Comma-separated bedroom counts. `0` = studio |
| `price_min` | integer | No | Minimum price in AED |
| `price_max` | integer | No | Maximum price in AED |
| `area_min` | number | No | Minimum area in sqft |
| `area_max` | number | No | Maximum area in sqft |
| `sort` | enum | No | `date_desc` (default), `date_asc`, `price_desc`, `price_asc`, `area_desc`, `area_asc` |
| `page` | integer | No | Page number. Returns 20 per page. Default: 1 |

**Example request:**
```bash
GET /transactions?purpose=for-sale&location_ids=5003&category_ids=apartments&time_period=12m&sort=date_desc&page=1
```

---

### Amenities Search

```
GET /amenities-search
```

Search available amenity names and see how many properties have each amenity. Use the returned `value` field as the `amenities` parameter in `/search-property`.

**Parameters:**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | No | Amenity search string. Example: `pool`, `gym`, `parking` |

**Example request:**
```bash
GET /amenities-search?query=pool
```

**Example response:**
```json
{
  "success": true,
  "data": {
    "amenities": [
      { "value": "Swimming Pool", "count": 45230, "highlighted": "<em>Swimming</em> Pool" },
      { "value": "Rooftop Pool", "count": 3100, "highlighted": "Rooftop Pool" },
      { "value": "Infinity Pool", "count": 1850, "highlighted": "Infinity Pool" }
    ],
    "total": 3
  }
}
```

---

## Response Format

All endpoints return a consistent JSON envelope:

```json
{
  "success": true,
  "data": { ... }
}
```

List endpoints include pagination info:

```json
{
  "success": true,
  "data": {
    "properties": [ ... ],
    "total": 1250,
    "page": 1,
    "totalPages": 53,
    "hitsPerPage": 24
  }
}
```

**Notes on the data:**

- Area is returned in **square metres (sqm)**. To convert to sqft: `sqft = sqm * 10.764`
- Prices are in **AED (UAE Dirham)**
- Timestamps (`createdAt`, `updatedAt`) are **Unix timestamps**
- Text fields like `title` and `description` are objects keyed by language code: `{ "en": "...", "ar": "..." }`
- Some fields may be `null` - always check before accessing nested properties

---

## Error Handling

All errors follow this format:

```json
{
  "success": false,
  "message": "Invalid request: Missing required parameter 'purpose'",
  "error": "VALIDATION_ERROR"
}
```

| HTTP Status | Error Code | When it happens |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Missing required parameter or invalid value |
| 404 | `API_ERROR` | Property, agent, or agency not found |
| 500 | `INTERNAL_ERROR` | Server-side error |
| 502 | `API_ERROR` | Upstream service temporarily unavailable |

**Handling rate limits (429):**

If you hit the rate limit for your plan, you will get a 429 response. Implement exponential backoff:

```javascript
async function fetchWithRetry(url, options, maxRetries = 3) {
  for (let attempt = 0; attempt < maxRetries; attempt++) {
    const response = await fetch(url, options);
    if (response.status !== 429) return response;
    const wait = Math.pow(2, attempt) * 1000; // 1s, 2s, 4s
    await new Promise(resolve => setTimeout(resolve, wait));
  }
  throw new Error("Max retries exceeded");
}
```

---

## Code Examples

### cURL

**Search apartments for sale in Dubai Marina:**
```bash
curl --request GET \
  --url 'https://bayut14.p.rapidapi.com/search-property?purpose=for-sale&location_ids=5003&property_type=apartments&rooms=1,2&price_max=2000000&page=1' \
  --header 'x-rapidapi-host: bayut14.p.rapidapi.com' \
  --header 'x-rapidapi-key: YOUR_API_KEY'
```

**Get location ID for an area:**
```bash
curl --request GET \
  --url 'https://bayut14.p.rapidapi.com/autocomplete?query=downtown+dubai&langs=en' \
  --header 'x-rapidapi-host: bayut14.p.rapidapi.com' \
  --header 'x-rapidapi-key: YOUR_API_KEY'
```

**Get 12 months of transaction history:**
```bash
curl --request GET \
  --url 'https://bayut14.p.rapidapi.com/transactions?purpose=for-sale&location_ids=5003&time_period=12m&category_ids=apartments' \
  --header 'x-rapidapi-host: bayut14.p.rapidapi.com' \
  --header 'x-rapidapi-key: YOUR_API_KEY'
```

---

### Python

```python
import requests

HEADERS = {
    "x-rapidapi-host": "bayut14.p.rapidapi.com",
    "x-rapidapi-key": "YOUR_API_KEY"
}

# Step 1 - find location ID
r = requests.get(
    "https://bayut14.p.rapidapi.com/autocomplete",
    params={"query": "dubai marina", "langs": "en"},
    headers=HEADERS
)
location_id = r.json()["data"]["locations"][0]["externalID"]
# "5003"

# Step 2 - search properties
r = requests.get(
    "https://bayut14.p.rapidapi.com/search-property",
    params={
        "purpose": "for-sale",
        "location_ids": location_id,
        "property_type": "apartments",
        "rooms": "1",
        "price_max": 1500000,
        "sort_order": "popular",
        "page": 1
    },
    headers=HEADERS
)
data = r.json()["data"]
print(f"{data['total']} properties found")

for prop in data["properties"][:3]:
    title = prop["title"]["en"]
    price = prop["price"]
    rooms = prop["rooms"]
    area_sqm = prop["area"]
    area_sqft = round(area_sqm * 10.764)
    print(f"{title} - AED {price:,} - {rooms} bed - {area_sqft} sqft")
```

---

### JavaScript / Node.js

```javascript
const API_KEY = "YOUR_API_KEY";
const HEADERS = {
  "x-rapidapi-host": "bayut14.p.rapidapi.com",
  "x-rapidapi-key": API_KEY
};

async function searchProperties(locationId, purpose = "for-sale") {
  const params = new URLSearchParams({
    purpose,
    location_ids: locationId,
    property_type: "apartments",
    rooms: "1",
    sort_order: "popular",
    page: "1"
  });

  const response = await fetch(
    `https://bayut14.p.rapidapi.com/search-property?${params}`,
    { headers: HEADERS }
  );

  const json = await response.json();
  return json.data;
}

// Get location ID first
async function getLocationId(query) {
  const response = await fetch(
    `https://bayut14.p.rapidapi.com/autocomplete?query=${encodeURIComponent(query)}&langs=en`,
    { headers: HEADERS }
  );
  const json = await response.json();
  return json.data.locations[0]?.externalID;
}

const locationId = await getLocationId("dubai marina");
const data = await searchProperties(locationId);
console.log(`${data.total} properties found`);
```

---

### PHP

```php
<?php

$apiKey = "YOUR_API_KEY";
$headers = [
    "x-rapidapi-host: bayut14.p.rapidapi.com",
    "x-rapidapi-key: $apiKey"
];

function bayutRequest($path, $params, $headers) {
    $url = "https://bayut14.p.rapidapi.com" . $path . "?" . http_build_query($params);
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
    $response = curl_exec($ch);
    curl_close($ch);
    return json_decode($response, true);
}

// Get location ID
$location = bayutRequest("/autocomplete", ["query" => "dubai marina", "langs" => "en"], $headers);
$locationId = $location["data"]["locations"][0]["externalID"];

// Search properties
$result = bayutRequest("/search-property", [
    "purpose" => "for-sale",
    "location_ids" => $locationId,
    "property_type" => "apartments",
    "rooms" => "1",
    "page" => 1
], $headers);

echo "Found " . $result["data"]["total"] . " properties\n";
```

---

### Next.js (App Router)

Keep your API key server-side. Never expose it in client components.

```javascript
// lib/bayut.js
export async function searchProperties(params) {
  const url = new URL("https://bayut14.p.rapidapi.com/search-property");
  Object.entries(params).forEach(([k, v]) => {
    if (v != null) url.searchParams.set(k, v);
  });

  const res = await fetch(url.toString(), {
    headers: {
      "x-rapidapi-host": "bayut14.p.rapidapi.com",
      "x-rapidapi-key": process.env.RAPIDAPI_KEY  // server-side only
    },
    next: { revalidate: 300 }  // cache for 5 minutes
  });

  if (!res.ok) throw new Error(`API error: ${res.status}`);
  return res.json();
}

// app/properties/page.jsx (Server Component)
import { searchProperties } from "@/lib/bayut";

export default async function PropertiesPage({ searchParams }) {
  const data = await searchProperties({
    purpose: searchParams.purpose || "for-sale",
    location_ids: searchParams.locationId,
    property_type: searchParams.type,
    page: searchParams.page || 1
  });

  return (
    <main>
      <h1>{data.data.total.toLocaleString()} Properties</h1>
      {data.data.properties.map(p => (
        <div key={p.externalID}>
          <h2>{p.title?.en}</h2>
          <p>AED {p.price?.toLocaleString()}</p>
        </div>
      ))}
    </main>
  );
}
```

---

## Common Location IDs

Use `/autocomplete` to find IDs for any area. Here are the most commonly used ones:

| Area | externalID | Notes |
|---|---|---|
| UAE (whole country) | 1 | Top-level, returns all UAE |
| Dubai (whole emirate) | 1 | Level 1 |
| Abu Dhabi (whole emirate) | 3 | Level 1 |
| Sharjah | 4 | Level 1 |
| Dubai Marina | 5003 | High-rise waterfront area |
| Downtown Dubai | 6901 | Burj Khalifa area |
| Palm Jumeirah | 5002 | Iconic palm-shaped island |
| Business Bay | 5460 | Mixed-use, near Downtown |
| Jumeirah Village Circle (JVC) | 6388 | Affordable, high yield |
| Jumeirah Lake Towers (JLT) | 5006 | Near Marina, established |
| Dubai Hills Estate | 11621 | Master-planned, Emaar |
| Jumeirah | 5001 | Beachside, villa community |
| Arabian Ranches | 6020 | Villa community |
| DIFC | 5462 | Financial district |
| Dubai Silicon Oasis | 6019 | Tech hub |
| Mirdif | 5008 | Family-friendly, affordable |

---

## Use Cases

### Property search portal

Build a Bayut-like search experience for a niche market - luxury properties, off-plan only, a specific nationality of buyers, or a specific area.

Core endpoints: `/autocomplete`, `/search-property`, `/property-details`, `/search-new-projects`

### Investment analytics dashboard

Pull transaction history and current listings to calculate rental yields, price trends, and area comparisons. Useful for investors, brokers, and analysts.

Core endpoints: `/transactions`, `/search-property` (for-sale + for-rent), `/search-new-projects`

**Rental yield formula:**
```
Gross Yield (%) = (Annual Rent / Purchase Price) x 100
```

### Real estate agent CRM

Sync agent listings automatically, track competitor activity, and generate market reports.

Core endpoints: `/agent-search`, `/agent-details`, `/agent-properties`, `/agency-search`

### Off-plan investment tracker

Aggregate all active off-plan projects in Dubai, track completion progress, and compare developers.

Core endpoints: `/search-new-projects`, `/developer-search-by-name`, `/transactions`

### Price alert system

Monitor listings and notify users when prices drop or new listings appear matching their criteria.

Core endpoints: `/search-property` (run on a schedule, compare against stored data)

### Neighbourhood comparison tool

Compare two Dubai areas side by side - average prices, transaction volume, available amenities, agent density.

Core endpoints: `/autocomplete`, `/search-property`, `/transactions`, `/amenities-search`

### Market research and academic analysis

Use transaction data for hedonic pricing models, rental yield studies, and market trend analysis.

Core endpoints: `/transactions`, `/search-property`, `/search-new-projects`

---

## Pricing and Plans

The API is available on RapidAPI with multiple plans:

- **Free plan** - limited requests per month, enough to prototype and test
- **Paid plans** - higher rate limits for production use

Subscribe here: https://rapidapi.com/happyendpoint/api/bayut14/

---

## Bulk Data

If you need large-scale historical data beyond what the real-time API provides, we sell pre-compiled datasets:

- 100,000+ Bayut records (listings + transactions)
- CSV and JSON formats
- Suitable for ML model training, econometric analysis, and large-scale research

Contact: happyendpointhq@gmail.com
Website: https://happyendpoint.com/apis/bayut

---

## FAQ

**Do I need proxies to use this API?**

No. The API handles everything on the backend. You just make standard HTTP requests with your API key.

**How often is the data updated?**

The API fetches data in real time on each request. There is no stale cache - you always get current listings.

**What is the area unit?**

Area is returned in square metres (sqm). To convert to square feet: `sqft = sqm * 10.764`. Dubai's market typically quotes prices in sqft, so you will want to convert for display.

**What languages are supported?**

English (`en`), Arabic (`ar`), Russian (`ru`), and Chinese (`zh`). Pass `langs=en,ar` to get both English and Arabic names in the same response.

**What is the difference between `externalID` and `ownerID` for agents?**

`externalID` is the agent's profile ID - use it with `/agent-details`. `ownerID` is a separate ID that links the agent to their listings - use it with `/agent-properties`. You get the `ownerID` from the `/agent-details` response.

**Can I filter properties by specific amenities?**

Yes. First call `/amenities-search` to get the exact amenity names, then pass them as a comma-separated list to the `amenities` parameter in `/search-property`. Example: `amenities=Swimming Pool,Gym,Parking`.

**What is TruBroker / TruCheck?**

TruBroker is Bayut's verified agent badge. TruCheck is their verified listing badge. You can filter for verified listings using `sort_order=verified` or `sort_order=trubroker_first`.

**Is there a rate limit?**

Yes, rate limits depend on your RapidAPI subscription plan. If you hit the limit, you will get a 429 response. Implement exponential backoff in your code.

**Can I get data for Abu Dhabi and other emirates?**

Yes. The API covers the whole UAE. Use `/autocomplete` to find location IDs for Abu Dhabi, Sharjah, Ajman, and other emirates.

**Do you offer a Postman collection?**

Yes - see the [bayut-api-postman-collection](https://github.com/yourusername/bayut-api-postman-collection) repo.

---

## Related Resources

- JavaScript / Next.js examples: [bayut-api-javascript-nextjs](https://github.com/happyendpointhq/bayut-api-javascript-nextjs)
- Postman collection: [bayut-api-postman-collection](https://github.com/happyendpointhq/bayut-api-postman-collection)

---

## Links

- API on RapidAPI: https://rapidapi.com/happyendpoint/api/bayut14/
- API landing page: https://happyendpoint.com/apis/bayut
- Full documentation: https://bayutapi.dev
- Happy Endpoint: https://happyendpoint.com
- All APIs: https://rapidapi.com/user/happyendpoint
- Twitter: https://x.com/happyendpointhq
- Medium: https://medium.com/@happyendpointhq
- Email: happyendpointhq@gmail.com

---

## License

MIT
